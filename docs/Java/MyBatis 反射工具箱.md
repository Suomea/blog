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

  private void resolveGetterConflicts(Map<String, List<Method>> conflictingGetters) {
    for (Entry<String, List<Method>> entry : conflictingGetters.entrySet()) {
      // 从冲突的属性方法中选出一个
      Method winner = null;
      String propName = entry.getKey();
      boolean isAmbiguous = false;
      for (Method candidate : entry.getValue()) {
        if (winner == null) {
          winner = candidate;
          continue;
        }
        Class<?> winnerType = winner.getReturnType();
        // 候选方法
        Class<?> candidateType = candidate.getReturnType();
        if (candidateType.equals(winnerType)) { 
          // 如果返回类型相同并且不是 boolean 返回类型的话就存在歧义
          // 比如 String getA() 和 String isA()
          if (!boolean.class.equals(candidateType)) {
            isAmbiguous = true;
            break;
          }
		  // 如果返回类型相同，并且都是 boolean 返回类型，则查找 is 开头的方法
		  // 比如 boolean getA() 和 boolean isA() 则使用 isA()
          if (candidate.getName().startsWith("is")) {
            winner = candidate;
          }
        } 
        // 如果返回类型不同，但是存在父子关系，则使用父类型方法
        else if (candidateType.isAssignableFrom(winnerType)) {
          // OK getter type is descendant
        } else if (winnerType.isAssignableFrom(candidateType)) {
          winner = candidate;
        } 
        // 存在属性相同，但是返回类型不一样，且返回类型不一致
        // 比如 String getA() 和 Integer isA()
        else {
          isAmbiguous = true;
          break;
        }
      }
      addGetMethod(propName, winner, isAmbiguous);
    }
  }

  private void addGetMethod(String name, Method method, boolean isAmbiguous) {
    MethodInvoker invoker = isAmbiguous ? new AmbiguousMethodInvoker(method, MessageFormat.format(
        "Illegal overloaded getter method with ambiguous type for property ''{0}'' in class ''{1}''. This breaks the JavaBeans specification and can cause unpredictable results.",
        name, method.getDeclaringClass().getName())) : new MethodInvoker(method);
    // 缓存属性和对应的 MethodInvoker
    getMethods.put(name, invoker);
    Type returnType = TypeParameterResolver.resolveReturnType(method, type);
    // 缓存属性和 getter 的返回类型
    getTypes.put(name, typeToClass(returnType));
  }
```

`MethodInvoker` 类的源码：
```java
public interface Invoker {  
  Object invoke(Object target, Object[] args) throws IllegalAccessException, InvocationTargetException;  
  
  Class<?> getType();  
}

public class MethodInvoker implements Invoker {

  private final Class<?> type;
  private final Method method;

  public MethodInvoker(Method method) {
    this.method = method;

	// setter 方法 type 存储参数类型
    if (method.getParameterTypes().length == 1) {
      type = method.getParameterTypes()[0];
    } 
    // getter 方法存储返回类型
    else {
      type = method.getReturnType();
    }
  }

  @Override
  public Object invoke(Object target, Object[] args) throws IllegalAccessException, InvocationTargetException {
    try {
      return method.invoke(target, args);
    } catch (IllegalAccessException e) {
      if (Reflector.canControlMemberAccessible()) {
        method.setAccessible(true);
        return method.invoke(target, args);
      }
      throw e;
    }
  }

  @Override
  public Class<?> getType() {
    return type;
  }
}
```

`ReflectorFactory` 工厂接口，只有一个实现类 `DefaultReflectorFactory`：
```java
public interface ReflectorFactory {
    boolean isClassCacheEnabled();

    void setClassCacheEnabled(boolean var1);

    Reflector findForClass(Class<?> var1);
}

public class DefaultReflectorFactory implements ReflectorFactory { 
    // 是否开启缓存
    private boolean classCacheEnabled = true;  
    // 缓存 Class 和 Reflector
    private final ConcurrentMap<Class<?>, Reflector> reflectorMap = new ConcurrentHashMap();  
  
    public DefaultReflectorFactory() {  
    }  
  
    public boolean isClassCacheEnabled() {  
        return this.classCacheEnabled;  
    }  
  
    public void setClassCacheEnabled(boolean classCacheEnabled) {  
        this.classCacheEnabled = classCacheEnabled;  
    }  
  
    public Reflector findForClass(Class<?> type) {  
        return this.classCacheEnabled ? 
        (Reflector)MapUtil.computeIfAbsent(this.reflectorMap, type, Reflector::new) : new Reflector(type);  
    }  
}

```