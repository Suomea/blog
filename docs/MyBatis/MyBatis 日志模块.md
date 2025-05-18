在 MyBatis 的日志模块中，定义了 Log 接口
```java
public interface Log {  
  
  boolean isDebugEnabled();  
  
  boolean isTraceEnabled();  
  
  void error(String s, Throwable e);  
  
  void error(String s);  
  
  void debug(String s);  
  
  void trace(String s);  
  
  void warn(String s);  
}
```

MyBatis 同时为 Log4j、Sl4j 等第三方日志组件提供了 Adapter，比如 Sl4j
```java
public class Slf4jImpl implements Log {  
  
  private Log log;  
  
  public Slf4jImpl(String clazz) {  
    Logger logger = LoggerFactory.getLogger(clazz);  
  
    if (logger instanceof LocationAwareLogger) {  
      try {  
        // check for slf4j >= 1.6 method signature  
        logger.getClass().getMethod("log", Marker.class, String.class, int.class, String.class, Object[].class,  
            Throwable.class);  
        log = new Slf4jLocationAwareLoggerImpl((LocationAwareLogger) logger);  
        return;  
      } catch (SecurityException | NoSuchMethodException e) {  
        // fail-back to Slf4jLoggerImpl  
      }  
    }  
  
    // Logger is not LocationAwareLogger or slf4j version < 1.6  
    log = new Slf4jLoggerImpl(logger);  
  }  
  
  @Override  
  public boolean isDebugEnabled() {  
    return log.isDebugEnabled();  
  }  
  
  @Override  
  public boolean isTraceEnabled() {  
    return log.isTraceEnabled();  
  }  
  
  @Override  
  public void error(String s, Throwable e) {  
    log.error(s, e);  
  }  
  
  @Override  
  public void error(String s) {  
    log.error(s);  
  }  
  
  @Override  
  public void debug(String s) {  
    log.debug(s);  
  }  
  
  @Override  
  public void trace(String s) {  
    log.trace(s);  
  }  
  
  @Override  
  public void warn(String s) {  
    log.warn(s);  
  }  
  
}
```

MyBatis 还提供了 LogFactory 工厂，来获取 Log 对象。
```java
public final class LogFactory {  
  
  /**  
   * Marker to be used by logging implementations that support markers.   */  public static final String MARKER = "MYBATIS";  
  
  private static final ReentrantLock lock = new ReentrantLock();  
  private static Constructor<? extends Log> logConstructor;  
  
  static {  
    tryImplementation(LogFactory::useSlf4jLogging);  
    tryImplementation(LogFactory::useCommonsLogging);  
    tryImplementation(LogFactory::useLog4J2Logging);  
    tryImplementation(LogFactory::useLog4JLogging);  
    tryImplementation(LogFactory::useJdkLogging);  
    tryImplementation(LogFactory::useNoLogging);  
  }
  
  private static void tryImplementation(Runnable runnable) {
    if (logConstructor == null) {
      try {
        runnable.run();
      } catch (Throwable t) {
        // ignore
      }
    }
  }

  public static Log getLog(Class<?> clazz) {
    return getLog(clazz.getName());
  }

  public static Log getLog(String logger) {
    try {
      return logConstructor.newInstance(logger);
    } catch (Throwable t) {
      throw new LogException("Error creating logger for logger " + logger + ".  Cause: " + t, t);
    }
  }
  
  ……

  public static void useCustomLogging(Class<? extends Log> clazz) {
    setImplementation(clazz);
  }

  public static void useSlf4jLogging() {
    setImplementation(org.apache.ibatis.logging.slf4j.Slf4jImpl.class);
  }

  private static void setImplementation(Class<? extends Log> implClass) {
    lock.lock();
    try {
      Constructor<? extends Log> candidate = implClass.getConstructor(String.class);
      Log log = candidate.newInstance(LogFactory.class.getName());
      if (log.isDebugEnabled()) {
        log.debug("Logging initialized using '" + implClass + "' adapter.");
      }
      logConstructor = candidate;
    } catch (Throwable t) {
      throw new LogException("Error setting Log implementation.  Cause: " + t, t);
    } finally {
      lock.unlock();
    }
  }
```

Logfactory 会依次尝试加载各个日志组件，加载是线程安全的的，使用 ReentrantLock 保证。

所以如果在应用启动阶段类路径下有 Sl4j 日志框架组件的话，那么 LogFactory 就会使用 Slf4jImpl 适配器来输出框架日志。

MyBatis 的链接、预处理和获取返回结果都有日志输出，但是只有在 DEBUG 级别启用时才会输出。ConnectionLogger 的代码片段如下：
```java
if ("prepareStatement".equals(method.getName()) || "prepareCall".equals(method.getName())) {  
  if (isDebugEnabled()) {  
    debug(" Preparing: " + removeExtraWhitespace((String) params[0]), true);  
  }  
  PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);  
  return PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);  
}
```

所以如果你使用 Sl4j + Logback 的话，想要输出 MyBatis 日志可以配置对应 Mapper 包的级别为 DEBUG：
```java
<logger name="com.jacky.mapper" level="DEBUG"/>
```

或者配置 MyBatis 的配置类，代码如下，
```java
public static SqlSessionFactory sqlSessionFactory() {  
    if (FACTORY_INSTANCE == null) {  
        synchronized (MyBatisConfig.class) {  
            if (FACTORY_INSTANCE == null) {  
				……              
                Configuration configuration = new Configuration(environment);  
                // 设置日志实现为标准输出  
                configuration.setLogImpl(org.apache.ibatis.logging.stdout.StdOutImpl.class);  
                // 如果 Mapper 接口都在同一个包下，可以使用包扫描  
                configuration.addMappers("com.jacky.mapper");  
                // 构建 SqlSessionFactory                
                FACTORY_INSTANCE =  new SqlSessionFactoryBuilder().build(configuration);  
            }  
        }  
    }  
    return FACTORY_INSTANCE;  
}
```

Configuration 的 setLogImpl 会设置 LogFactory 的日志实现类
```java
public void setLogImpl(Class<? extends Log> logImpl) {  
  if (logImpl != null) {  
    this.logImpl = logImpl;  
    LogFactory.useCustomLogging(this.logImpl);  
  }  
}
```

StdOutImpl 的 DEBUG 默认是开启的：
```java
public class StdOutImpl implements Log {  
  
  public StdOutImpl(String clazz) {  
    // Do Nothing  
  }  
  
  @Override  
  public boolean isDebugEnabled() {  
    return true;  
  }  
  
  @Override  
  public boolean isTraceEnabled() {  
    return true;  
  }  
  
  @Override  
  public void error(String s, Throwable e) {  
    System.err.println(s);  
    e.printStackTrace(System.err);  
  }  
  
  @Override  
  public void error(String s) {  
    System.err.println(s);  
  }  
  
  @Override  
  public void debug(String s) {  
    System.out.println(s);  
  }  
  
  @Override  
  public void trace(String s) {  
    System.out.println(s);  
  }  
  
  @Override  
  public void warn(String s) {  
    System.out.println(s);  
  }  
}
```

如果在项目的类路径下有 Sl4j + Logback，但同时又手动配置了 Configuration 的 logImpl 为 StdOutImpl，那么最终 MyBatis 使用的是 StdOutImpl。
前者是在类加载阶段设置的，static 代码片段；后者一般是运行时设置的，会覆盖掉 LogFactory 的 logConstructor 属性。MyBatis 内部进行日志打印时，一般是在运行时调用 LogFactory.getLog 方法获取。