---
comments: false
---
## Reflector
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
```java
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
    // 由于语法规定，一个类只能继承一个类，所以这里循环向上遍历直到 Object
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

	  // 向上遍历父类
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
    // Class.getName() 不会包含泛型信息
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
    // 缓存属性对应的方法，key 为属性名，value 为方法列表
    // 使用列表缓存方法，是因为考虑到属性名称相同的方法（isA 和 getA）
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
        // 大部分情况下并不会存在 isA 和 getA 方法同时存在的情况
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

  public static Type resolveReturnType(Method method, Type srcType) {
    Type returnType = method.getGenericReturnType();
    Class<?> declaringClass = method.getDeclaringClass();
    return resolveType(returnType, srcType, declaringClass);
  }

  private static Type resolveType(Type type, Type srcType, Class<?> declaringClass) {
    if (type instanceof TypeVariable) {
      return resolveTypeVar((TypeVariable<?>) type, srcType, declaringClass);
    } 
    else if (type instanceof ParameterizedType) {
      return resolveParameterizedType((ParameterizedType) type, srcType, declaringClass);
    } 
    else if (type instanceof GenericArrayType) {
      return resolveGenericArrayType((GenericArrayType) type, srcType, declaringClass);
    } 
    else if (type instanceof WildcardType) {
      return resolveWildcardType((WildcardType) type, srcType, declaringClass);
    } 
    else {
      return type;
    }
  }
```

如果存在属性相同但是返回类型不同的情况，会构造一个 `AmbiguousMethodInvoker` 否则是 `MethodInvoker`：
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

public class AmbiguousMethodInvoker extends MethodInvoker {
  private final String exceptionMessage;

  public AmbiguousMethodInvoker(Method method, String exceptionMessage) {
    super(method);
    this.exceptionMessage = exceptionMessage;
  }

  @Override
  public Object invoke(Object target, Object[] args) throws IllegalAccessException, InvocationTargetException {
    throw new ReflectionException(exceptionMessage);
  }
}
```

## ReflectorFactory
工厂接口，只有一个实现类 `DefaultReflectorFactory`：
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

## ObjectFatory
接口主要定义了两个创建对象的方法：
```java
  <T> T create(Class<T> type);
  <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs);
```

DefaultObjectFactory 是该接口的唯一实现类：
```java
public class DefaultObjectFactory implements ObjectFactory, Serializable {

  private static final long serialVersionUID = -8855120656740914948L;

  @Override
  public <T> T create(Class<T> type) {
    return create(type, null, null);
  }

  @SuppressWarnings("unchecked")
  @Override
  public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    Class<?> classToCreate = resolveInterface(type);
    // we know types are assignable
    return (T) instantiateClass(classToCreate, constructorArgTypes, constructorArgs);
  }

  private <T> T instantiateClass(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    try {
      Constructor<T> constructor;
      if (constructorArgTypes == null || constructorArgs == null) {
        // 如果没有参数列表，直接调用默认的构造函数生成对象
        constructor = type.getDeclaredConstructor();
        try {
          return constructor.newInstance();
        } catch (IllegalAccessException e) {
          if (Reflector.canControlMemberAccessible()) {
            constructor.setAccessible(true);
            return constructor.newInstance();
          }
          throw e;
        }
      }
      // 如果传递了参数类型列表，则查找对应的构造函数生成对象
      constructor = type.getDeclaredConstructor(constructorArgTypes.toArray(new Class[0]));
      try {
        return constructor.newInstance(constructorArgs.toArray(new Object[0]));
      } catch (IllegalAccessException e) {
        if (Reflector.canControlMemberAccessible()) {
          constructor.setAccessible(true);
          return constructor.newInstance(constructorArgs.toArray(new Object[0]));
        }
        throw e;
      }
    } catch (Exception e) {
      String argTypes = Optional.ofNullable(constructorArgTypes).orElseGet(Collections::emptyList).stream()
          .map(Class::getSimpleName).collect(Collectors.joining(","));
      String argValues = Optional.ofNullable(constructorArgs).orElseGet(Collections::emptyList).stream()
          .map(String::valueOf).collect(Collectors.joining(","));
      throw new ReflectionException("Error instantiating " + type + " with invalid types (" + argTypes + ") or values ("
          + argValues + "). Cause: " + e, e);
    }
  }

  // 如果传递了这些接口类型，则处理一下使用默认的实现类
  protected Class<?> resolveInterface(Class<?> type) {
    Class<?> classToCreate;
    if (type == List.class || type == Collection.class || type == Iterable.class) {
      classToCreate = ArrayList.class;
    } else if (type == Map.class) {
      classToCreate = HashMap.class;
    } else if (type == SortedSet.class) { // issue #510 Collections Support
      classToCreate = TreeSet.class;
    } else if (type == Set.class) {
      classToCreate = HashSet.class;
    } else {
      classToCreate = type;
    }
    return classToCreate;
  }
}
```

## Property 工具集
### PropertyTokenizer
用来解析属性表达式，如 `user.name`，`user.orders[0].amount`，`userList[0].name` 等。
```java
public class PropertyTokenizer implements Iterator<PropertyTokenizer> {
  private String name;  // . 分割的第一部分，不包含索引
  private final String indexedName;  // . 分割的第一部分，包含索引
  private String index;  // 索引值
  private final String children; // . 分割的剩余部分 
  public PropertyTokenizer(String fullname) {
    int delim = fullname.indexOf('.');
    if (delim > -1) {
      name = fullname.substring(0, delim);
      children = fullname.substring(delim + 1);
    } else {
      name = fullname;
      children = null;
    }
    indexedName = name;
    delim = name.indexOf('[');
    if (delim > -1) {
      index = name.substring(delim + 1, name.length() - 1);
      name = name.substring(0, delim);
    }
  }
  
  @Override
  public boolean hasNext() {
    return children != null;
  }

  @Override
  public PropertyTokenizer next() {
    return new PropertyTokenizer(children);
  }
}
```

示例：
```java
String b = "user.orders[0].amount";  
  
PropertyTokenizer pt = new PropertyTokenizer(b);  
System.out.println("name: " + pt.getName() + ", indexedName: " + pt.getIndexedName() + ", index: " + pt.getIndex() + ", children: " + pt.getChildren());  
while (pt.hasNext()) {  
    pt = pt.next();  
    System.out.println("name: " + pt.getName() + ", indexedName: " + pt.getIndexedName() + ", index: " + pt.getIndex() + ", children: " + pt.getChildren());  
}

// 输出
// name: user, indexedName: user, index: null, children: orders[0].amount
// name: orders, indexedName: orders[0], index: 0, children: amount
// name: amount, indexedName: amount, index: null, children: null
```

### PropertyNamer
提供静态方法，完成方法名到属性名称的转换。
```java
public final class PropertyNamer {

  private PropertyNamer() {
    // Prevent Instantiation of Static Class
  }

  // 方法名称转换成属性名称
  public static String methodToProperty(String name) {
    if (name.startsWith("is")) {
      name = name.substring(2);
    } else if (name.startsWith("get") || name.startsWith("set")) {
      name = name.substring(3);
    } else {
      throw new ReflectionException(
          "Error parsing property name '" + name + "'.  Didn't start with 'is', 'get' or 'set'.");
    }

    if (name.length() == 1 || name.length() > 1 && !Character.isUpperCase(name.charAt(1))) {
      name = name.substring(0, 1).toLowerCase(Locale.ENGLISH) + name.substring(1);
    }

    return name;
  }

  public static boolean isProperty(String name) {
    return isGetter(name) || isSetter(name);
  }

  public static boolean isGetter(String name) {
    return name.startsWith("get") && name.length() > 3 || name.startsWith("is") && name.length() > 2;
  }

  public static boolean isSetter(String name) {
    return name.startsWith("set") && name.length() > 3;
  }
}
```

### PropertyCopier
主要实现对两个同类型对象之间的属性拷贝。
```java
public final class PropertyCopier {

  private PropertyCopier() {
    // Prevent Instantiation of Static Class
  }

  // 属性复制
  public static void copyBeanProperties(Class<?> type, Object sourceBean, Object destinationBean) {
    Class<?> parent = type;
    while (parent != null) {
      final Field[] fields = parent.getDeclaredFields();
      for (Field field : fields) {
        try {
          try {
            field.set(destinationBean, field.get(sourceBean));
          } catch (IllegalAccessException e) {
            if (!Reflector.canControlMemberAccessible()) {
              throw e;
            }
            field.setAccessible(true);
            field.set(destinationBean, field.get(sourceBean));
          }
        } catch (Exception e) { 
          // Nothing useful to do, will only fail on final fields, which will be ignored.
        }
      }
      parent = parent.getSuperclass();
    }
  }
}
```

## MetaClass
是 MyBatis 对类级别的元数据封装对处理，对外提供的工具类。成员变量和构造方法：
```java
  private final ReflectorFactory reflectorFactory;
  private final Reflector reflector;

  private MetaClass(Class<?> type, ReflectorFactory reflectorFactory) {
    this.reflectorFactory = reflectorFactory;
    this.reflector = reflectorFactory.findForClass(type);
  }

  public static MetaClass forClass(Class<?> type, ReflectorFactory reflectorFactory) {
    return new MetaClass(type, reflectorFactory);
  }
```

`findProperty(String)` 方法查找属性，需要注意的是不支持索引查找。
```java
public String findProperty(String name) {  
  StringBuilder prop = buildProperty(name, new StringBuilder());  
  return prop.length() > 0 ? prop.toString() : null;  
}

private StringBuilder buildProperty(String name, StringBuilder builder) {  
  PropertyTokenizer prop = new PropertyTokenizer(name);  // 解析属性名称
  if (prop.hasNext()) { // 如果是表达式则递归查找 
    String propertyName = reflector.findPropertyName(prop.getName());  // 先查找名称
    if (propertyName != null) {  
      builder.append(propertyName);  
      builder.append(".");  
      MetaClass metaProp = metaClassForProperty(propertyName);  
      // 解析 children 进行递归查找
      metaProp.buildProperty(prop.getChildren(), builder); 
    }  
  } else {  // 如果直接是属性名称，直接查找
    String propertyName = reflector.findPropertyName(name);  
    if (propertyName != null) {  
      builder.append(propertyName);  
    }  
  }  
  return builder;  
}

public MetaClass metaClassForProperty(String name) {  
  Class<?> propType = reflector.getGetterType(name);  
  return MetaClass.forClass(propType, reflectorFactory);  
}
```


hasGetter(String) hasSetter(String) 方法，判断是否存在对应的属性，支持索引递归查询。
```java
  public boolean hasSetter(String name) {
    PropertyTokenizer prop = new PropertyTokenizer(name);
    if (!prop.hasNext()) { // 递归出口
      return reflector.hasSetter(prop.getName());
    }
    if (reflector.hasSetter(prop.getName())) {
      MetaClass metaProp = metaClassForProperty(prop.getName());
      return metaProp.hasSetter(prop.getChildren());
    }
    return false;
  }

  public boolean hasGetter(String name) {
    PropertyTokenizer prop = new PropertyTokenizer(name);
    if (!prop.hasNext()) { // 递归出口
      return reflector.hasGetter(prop.getName());
    }
    if (reflector.hasGetter(prop.getName())) {
      MetaClass metaProp = metaClassForProperty(prop);
      return metaProp.hasGetter(prop.getChildren());
    }
    return false;
  }
  
  private MetaClass metaClassForProperty(PropertyTokenizer prop) {
    Class<?> propType = getGetterType(prop);
    return MetaClass.forClass(propType, reflectorFactory);
  }

  private Class<?> getGetterType(PropertyTokenizer prop) {
    // 获取属性类型
    Class<?> type = reflector.getGetterType(prop.getName());
    // 如果使用了索引并且是 Collection 子类
    if (prop.getIndex() != null && Collection.class.isAssignableFrom(type)) {
      // 属性的返回类型
      Type returnType = getGenericGetterType(prop.getName());
      if (returnType instanceof ParameterizedType) {
        Type[] actualTypeArguments = ((ParameterizedType) returnType).getActualTypeArguments();
        if (actualTypeArguments != null && actualTypeArguments.length == 1) {
          returnType = actualTypeArguments[0];
          if (returnType instanceof Class) {
            type = (Class<?>) returnType;
          } else if (returnType instanceof ParameterizedType) {
            type = (Class<?>) ((ParameterizedType) returnType).getRawType();
          }
        }
      }
    }
    return type;
  }

  private Type getGenericGetterType(String propertyName) {
    try {
      Invoker invoker = reflector.getGetInvoker(propertyName);
      if (invoker instanceof MethodInvoker) {
        Field declaredMethod = MethodInvoker.class.getDeclaredField("method");
        declaredMethod.setAccessible(true);
        Method method = (Method) declaredMethod.get(invoker);
        return TypeParameterResolver.resolveReturnType(method, reflector.getType());
      }
      if (invoker instanceof GetFieldInvoker) {
        Field declaredField = GetFieldInvoker.class.getDeclaredField("field");
        declaredField.setAccessible(true);
        Field field = (Field) declaredField.get(invoker);
        return TypeParameterResolver.resolveFieldType(field, reflector.getType());
      }
    } catch (NoSuchFieldException | IllegalAccessException e) {
      // Ignored
    }
    return null;
  }
```

## ObjectWrapper
```java
public interface ObjectWrapper {
  // 如果封装的是普通的 Bean 对象，则调用相应属性的 getter 方法
  // 如果封装的是集合类，则获取指定 key 或下标对应的 value 值
  Object get(PropertyTokenizer prop);

  // 如果封装的是普通的 Bean 对象，则调用相应属性的 setter 方法
  // 如果封装的是集合类，则设置指定 key 或下标对应的 value 值
  void set(PropertyTokenizer prop, Object value);

  // 查找属性表达式指定的属性
  String findProperty(String name, boolean useCamelCaseMapping);

  // 查找可读属性的名称集合
  String[] getGetterNames();

  // 查找可写属性的名称集合
  String[] getSetterNames();

  // 解析属性表达式指定属性 setter 方法的参数类型
  Class<?> getSetterType(String name);

  // 解析属性表达式指定输出 getter 方法的返回值类型
  Class<?> getGetterType(String name);

  // 判断属性表达式指定属性是否有 setter/getter 方法
  boolean hasSetter(String name);
  boolean hasGetter(String name);

  // 为属性表达式指定的属性创建相应的 MetaObject
  MetaObject instantiatePropertyValue(String name, PropertyTokenizer prop, ObjectFactory objectFactory);

  // 判断封装对象是否为 Collection 类型
  boolean isCollection();

  // 调用 Collection 对象的 add 方法
  void add(Object element);

  // 调用 Collection 对象的 addAll 方法
  <E> void addAll(List<E> element);
}
```

ObjectWrapperFactory 负责创建 ObjectWrapper 对象。

ObjectWrapper  

- BaseWrapper  
	- BeanWrapper  
	- MapWrapper  
- CollectionWrapper  

BaseWrapper 是一个实现了 ObjectWrapper 接口的抽象类，其中封装了 MetaObject 对象，并提供了三个常用的方法供子类使用：
```java
  protected Object resolveCollection(PropertyTokenizer prop, Object object) {
    if ("".equals(prop.getName())) {
      return object;
    }
    return metaObject.getValue(prop.getName());
  }

  // 解析属性表达式的索引信息，获取指定的项
  protected Object getCollectionValue(PropertyTokenizer prop, Object collection) {
    if (collection == null) {
      throw new ReflectionException("Cannot get the value '" + prop.getIndexedName() + "' because the property '"
          + prop.getName() + "' is null.");
    }
    if (collection instanceof Map) {
      return ((Map) collection).get(prop.getIndex());
    }
    int i = Integer.parseInt(prop.getIndex());
    if (collection instanceof List) {
      return ((List) collection).get(i);
    } else if (collection instanceof Object[]) {
      return ((Object[]) collection)[i];
    } else if (collection instanceof char[]) {
      return ((char[]) collection)[i];
    } else if (collection instanceof boolean[]) {
      return ((boolean[]) collection)[i];
    } else if (collection instanceof byte[]) {
      return ((byte[]) collection)[i];
    } else if (collection instanceof double[]) {
      return ((double[]) collection)[i];
    } else if (collection instanceof float[]) {
      return ((float[]) collection)[i];
    } else if (collection instanceof int[]) {
      return ((int[]) collection)[i];
    } else if (collection instanceof long[]) {
      return ((long[]) collection)[i];
    } else if (collection instanceof short[]) {
      return ((short[]) collection)[i];
    } else {
      throw new ReflectionException("Cannot get the value '" + prop.getIndexedName() + "' because the property '"
          + prop.getName() + "' is not Map, List or Array.");
    }
  }

  // 解析属性表达式的索引信息，设置指定的项
  protected void setCollectionValue(PropertyTokenizer prop, Object collection, Object value) {
    if (collection == null) {
      throw new ReflectionException("Cannot set the value '" + prop.getIndexedName() + "' because the property '"
          + prop.getName() + "' is null.");
    }
    if (collection instanceof Map) {
      ((Map) collection).put(prop.getIndex(), value);
    } else {
      int i = Integer.parseInt(prop.getIndex());
      if (collection instanceof List) {
        ((List) collection).set(i, value);
      } else if (collection instanceof Object[]) {
        ((Object[]) collection)[i] = value;
      } else if (collection instanceof char[]) {
        ((char[]) collection)[i] = (Character) value;
      } else if (collection instanceof boolean[]) {
        ((boolean[]) collection)[i] = (Boolean) value;
      } else if (collection instanceof byte[]) {
        ((byte[]) collection)[i] = (Byte) value;
      } else if (collection instanceof double[]) {
        ((double[]) collection)[i] = (Double) value;
      } else if (collection instanceof float[]) {
        ((float[]) collection)[i] = (Float) value;
      } else if (collection instanceof int[]) {
        ((int[]) collection)[i] = (Integer) value;
      } else if (collection instanceof long[]) {
        ((long[]) collection)[i] = (Long) value;
      } else if (collection instanceof short[]) {
        ((short[]) collection)[i] = (Short) value;
      } else {
        throw new ReflectionException("Cannot set the value '" + prop.getIndexedName() + "' because the property '"
            + prop.getName() + "' is not Map, List or Array.");
      }
    }
  }
```

BeanWrapper 继承了 BaseWrapper，封装了 JavaBean 以及该对象的 MetaClass 对象，还有从 BaseWrapper 继承而来的 MetaObject 对象。

BeanWrapper 的 get/set 会根据指定的表达式获取和设置相应的属性值。
```java
  @Override
  public Object get(PropertyTokenizer prop) {
    if (prop.hasNext()) {
      return getChildValue(prop);
    } else if (prop.getIndex() != null) {
      return getCollectionValue(prop, resolveCollection(prop, object));
    } else {
      return getBeanProperty(prop, object);
    }
  }

  protected Object getChildValue(PropertyTokenizer prop) {
    MetaObject metaValue = metaObject.metaObjectForProperty(prop.getIndexedName());
    if (metaValue == SystemMetaObject.NULL_META_OBJECT) {
      return null;
    }
    // 
    return metaValue.getValue(prop.getChildren());
  }

  private Object getBeanProperty(PropertyTokenizer prop, Object object) {
    try {
      Invoker method = metaClass.getGetInvoker(prop.getName());
      try {
        return method.invoke(object, NO_ARGUMENTS);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
    } catch (RuntimeException e) {
      throw e;
    } catch (Throwable t) {
      throw new ReflectionException(
          "Could not get property '" + prop.getName() + "' from " + object.getClass() + ".  Cause: " + t.toString(), t);
    }
  }
```

## MetaObject
ObjectWrapper 提供获取/设置对象中指定属性、检测 getter/setter 等常用功能。但是具体的属性解析还是能看到 MetaObject 身影，这些功能由 MetaObject 来负责。

MetaObject 的属性和构造函数：
```java
public class MetaObject {

  private final Object originalObject;
  private final ObjectWrapper objectWrapper;
  private final ObjectFactory objectFactory;
  private final ObjectWrapperFactory objectWrapperFactory;
  private final ReflectorFactory reflectorFactory;

  private MetaObject(Object object, ObjectFactory objectFactory, ObjectWrapperFactory objectWrapperFactory,
      ReflectorFactory reflectorFactory) {
    this.originalObject = object;
    this.objectFactory = objectFactory;
    this.objectWrapperFactory = objectWrapperFactory;
    this.reflectorFactory = reflectorFactory;

    if (object instanceof ObjectWrapper) {
      this.objectWrapper = (ObjectWrapper) object;
    } else if (objectWrapperFactory.hasWrapperFor(object)) {
      this.objectWrapper = objectWrapperFactory.getWrapperFor(this, object);
    } else if (object instanceof Map) {
      this.objectWrapper = new MapWrapper(this, (Map) object);
    } else if (object instanceof Collection) {
      this.objectWrapper = new CollectionWrapper(this, (Collection) object);
    } else {
      this.objectWrapper = new BeanWrapper(this, object);
    }
  }
}
```

MetaObject 和 ObjectWrapper 中关于类的方法都是直接调用 MetaClass 方法实现的。其它关于类的方法是和 ObjectWrapper 配合实现。