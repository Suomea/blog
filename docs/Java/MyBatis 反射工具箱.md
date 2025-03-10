每个 `Reflector` 对象对应一个类,在 `Reflector` 中缓存了反射操作需要使用的类的元信息。

!!!note "字段和属性"
	在解析 `Reflector` 的源码之前，先区分一下“字段”和“属性”的区别。类中定义的成员变量称为字段，而属性只与方法有关。如果类存在 `getA()` 和 `setA(String)` 方法，就认为类中存在属性 `a`，和实际类中存不存在字段 `a` 无关。

Reflector 类中各个字段的含义。
```java

  // 对应的 Class 类型
  private final Class<?> type;

  // 可读属性的集合，存在相应的 getter 方法
  private final String[] readablePropertyNames;
  // 可写属性的集合，存在相应的 setter 方法
  private final String[] writablePropertyNames;

  // 缓存了属性名称和对应的 setter 方法，Invoker 是 Method 的封装
  private final Map<String, Invoker> setMethods = new HashMap<>();
  // 缓存了属性名称和对应的 getter 方法
  private final Map<String, Invoker> getMethods = new HashMap<>();

  // 缓存了属性名称和 setter 方法接收的参数类型
  private final Map<String, Class<?>> setTypes = new HashMap<>();
  // 缓存了属性名称和 getter 方法返回的参数类型
  private final Map<String, Class<?>> getTypes = new HashMap<>();

  // 默认的构造函数
  private Constructor<?> defaultConstructor;
```

使用方法示例：
```java
Reflector reflector = new Reflector(TestEntity.class);
```

`Reflector` 的构造函数
```java
  public Reflector(Class<?> clazz) {
    type = clazz; // 初始化 type 字段
    addDefaultConstructor(clazz); // 查找默认的构造函数🍕
    Method[] classMethods = getClassMethods(clazz); // 获取类的所有方法 🍔
    if (isRecord(type)) { // 判断是否是 record 类类型，用的少忽略
      addRecordGetMethods(classMethods);
    } else {
      addGetMethods(classMethods); // 处理所有的 getter 方法 🍟
      addSetMethods(classMethods); // 处理所有的 setter 方法
      addFields(clazz);
    }
    readablePropertyNames = getMethods.keySet().toArray(new String[0]);
    writablePropertyNames = setMethods.keySet().toArray(new String[0]);
    for (String propName : readablePropertyNames) {
      caseInsensitivePropertyMap.put(propName.toUpperCase(Locale.ENGLISH), propName);
    }
    for (String propName : writablePropertyNames) {
      caseInsensitivePropertyMap.put(propName.toUpperCase(Locale.ENGLISH), propName);
    }
  }
```

查找默认的构造函数🍕：
```
  // 查找默认的构造函数，如果存在参数列表为 0 的就赋值给 defaultConstructor
  // 否则 defaultConstructor 为 null
  private void addDefaultConstructor(Class<?> clazz) {
    Constructor<?>[] constructors = clazz.getDeclaredConstructors();
    Arrays.stream(constructors)
    .filter(constructor -> constructor.getParameterTypes().length == 0)
    .findAny()
    .ifPresent(constructor -> this.defaultConstructor = constructor);
  }
```

获取类的所有方法🍔：
```java
 // 获取类的所有方法，本身定义的以及父类父接口定义的
 private Method[] getClassMethods(Class<?> clazz) {
    // 缓存方法签名和 Method 对象
    Map<String, Method> uniqueMethods = new HashMap<>();
    Class<?> currentClass = clazz;
    // 由于语法规定，一个类只能继承一个类，所以这里循环向上便利直到 Object
    while (currentClass != null && currentClass != Object.class) {
      // getDeclaredMethods 获取当前类中定义的所有方法
      addUniqueMethods(uniqueMethods, currentClass.getDeclaredMethods());

      // we also need to look for interface methods -
      // because the class may be abstract
      // 如果是实现的接口，直接使用 getMethods 获取整个接口树的所有方法
      // 因为接口的方法都是 public 的，所以可以一次性获取
      Class<?>[] interfaces = currentClass.getInterfaces();
      for (Class<?> anInterface : interfaces) {
        addUniqueMethods(uniqueMethods, anInterface.getMethods());
      }

	  // 向上便利父类
      currentClass = currentClass.getSuperclass();
    }

    Collection<Method> methods = uniqueMethods.values();

    return methods.toArray(new Method[0]);
  }

  private void addUniqueMethods(Map<String, Method> uniqueMethods, Method[] methods) {
    for (Method currentMethod : methods) {
      if (!currentMethod.isBridge()) { // 忽略泛型桥接方法
        String signature = getSignature(currentMethod);
        // check to see if the method is already known
        // if it is known, then an extended class must have
        // overridden a method
        // 如果方法签名已经存在，则忽略
        if (!uniqueMethods.containsKey(signature)) {
          uniqueMethods.put(signature, currentMethod);
        }
      }
    }
  }

  // 获取方法签名：返回类型#方法名:参数1类型,参数2类型
  // 方法签名不包括返回值类型
  private String getSignature(Method method) {
    StringBuilder sb = new StringBuilder();
    Class<?> returnType = method.getReturnType();
    sb.append(returnType.getName()).append('#');
    sb.append(method.getName());
    Class<?>[] parameters = method.getParameterTypes();
    for (int i = 0; i < parameters.length; i++) {
      sb.append(i == 0 ? ':' : ',').append(parameters[i].getName());
    }
    return sb.toString();
  }
```

处理 getter 方法🍟：
```java
  private void addGetMethods(Method[] methods) {
    // 缓存重载的 getter 方法
    Map<String, List<Method>> conflictingGetters = new HashMap<>();
    Arrays.stream(methods)
    // 保留 getXXX 或者 isXXX 并且参数列表为空的 getter 方法
    .filter(m -> m.getParameterTypes().length == 0 && PropertyNamer.isGetter(m.getName()))
    // 将 getAbc 或者 isAbc 转换成 abc，并且缓存到 map 里面 
    .forEach(m -> addMethodConflict(conflictingGetters, PropertyNamer.methodToProperty(m.getName()), m));
    // 解决重载的 getter 方法
    resolveGetterConflicts(conflictingGetters);
  }
```