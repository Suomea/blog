## SLF4J + Logback

SLF4J（Simple Logging Facade for Java）是一个简单的日志门面，为各种日志框架提供了一个统一的抽象层。

Logback 是一个日志框架，是 SLF4J 的原生实现。这两个库是同一个作者。



## Logback 的架构
Logback 有三个包：

- logback-classic SLF4J 的实现，继承并扩展 logback-core 的功能。
- logback-core 核心的日志功能实现。
- logback-access 为 Servlet 容器提供 HTTP 访问日志的功能。

Logback 主要的组件：

- Logger 创建日志，决定是否记录。
- Appender 决定日志输出的目的地。一个 Logger 可以有多个 Appender。
- Layout 决定日志输出的格式。

## Logger 上下文
Logger 存在层级关系，ROOT Logger 是所有 Logger 的祖先，默认 DEBUG 日志级别。

如果 ExampleP 位于 com.package，ExampleS 位于 com.package.logger，那么 ExampleS 的 Logger 就是 ExcampleP Logger 的孩子。前提是这样初始化 `LoggerFactory.getLogger(Example[P|S].class);`。

如果一个 Logger 没有设置 LEVEL，那么默认继承父级 Logger 的 LEVEL，LEVEL 可以在配置文件和代码中配置，代码中的配置会覆盖配置文件的配置。 可以在配置文件中设置 additivity="false" 关闭继承。

参数化日志记录 log.debug("Current count is {}", count); 的一个关键优势在于：当日志级别不满足输出条件时，参数构造操作将被完全跳过。  
相比而言，传统的字符串拼接写法 log.debug("Current count is " + count); 无论日志级别如何，都会先执行字符串拼接操作，可能带来不必要的性能开销。

### 配置文件
输出 loback 自身处理配置时的调试信息
```
<configuration debug="true">
...
</configuration>
```

自动重新加载配置文件，默认 60 秒
```
<configuration scan="true" scanPeriod="15 seconds">
...
</configuration>
```

设置 service 包下面的全部输出 WARN 级别日志，mapper 包下面的全部数据 DEBUG 日志
```
<configuration">
   ...
   <logger name="com.app.service" level="WARN" /> 
   <logger name="com.app.mapper" level="DEBUG" /> 
   ...
</configuration>
```

变量替换，配置文件支持定义变量
```
<property name="LOG_DIR" value="/var/log/application" />
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${LOG_DIR}/tests.log</file>
    <append>true</append>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender>
```

变量不仅可以通过配置文件的 property 标签定义，也可以通过环境变量的方式定义
```
$ java -DLOG_DIR=/var/log/application LogbackTests
```

logback 处理变量的方式仅仅只是文本替换。

### Appender
Logger 传递 LoggingEvents 给 Appenders，Appenders 负责实际的日志记录工作。

ConsoleAppender

FileAppender
```xml
<configuration debug="true">
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoders are assigned the type
             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>tests.log</file>
        <append>true</append>
        <encoder>
            <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <logger name="com.baeldung.logback" level="INFO" /> 
    <logger name="com.baeldung.logback.tests" level="WARN"> 
        <appender-ref ref="FILE" /> 
    </logger> 

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```
FileAppender 的 file 标签配置了日志输出的文件名，append 设置日志是追加到日志文件还是覆盖日志文件。

Logger `com.baeldung.logback.tests` 的日志会输出到控制台和文件，因为子 Logger 会继承所有祖先的 Appender，可以在配置文件中配置通过设置 additivity="false" 关闭继承。

RollingFileAppender

通常日志输出文件要能够基于时间、文件大小、或者两者结合滚动生成新的文件，以防止单一的日志文件过大。
```xml
<property name="LOG_FILE" value="LogFile" />
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_FILE}.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- daily rollover -->
        <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>

        <!-- keep 30 days' worth of history capped at 3GB total size -->
        <maxHistory>30</maxHistory>
        <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender>
```
RollingFileAdapter 能够配置 rollingPolicy，上面配置使用 TimeBasedRollingPolicy。

自动删除 30 天前的文件，所有日志文件总大小不超过 3GB，当超过 3GB 时，删除最旧的文件。

Logback 还提供了 SizeAndTimeBasedRollingPolicy 和 FixedWindowRollingPolicy 两种滚动策略。
```xml
 <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 滚动文件命名模式：必须包含 %d 和 %i -->
            <fileNamePattern>${LOG_PATH}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- 每个日志文件最大大小 -->
            <maxFileSize>10MB</maxFileSize>
            <!-- 保留30天的历史日志 -->
            <maxHistory>30</maxHistory>
            <!-- 所有日志文件总大小上限 -->
            <totalSizeCap>3GB</totalSizeCap>
</rollingPolicy>
```

RollingFileAppendet 内置了基于文件名的文件压缩功能，如果文件名是 .gz 或 .zip 则使用 GZ 或 ZIP 压缩。

1.5.18 之后，Loback 还支持 xz 压缩，压缩比更高当然也会消耗更多的 CPU 和内存资源。
使用 xz 压缩需要日志文件后缀为 .xz，同时还要引入依赖，如果使用 .xz 但是缺少依赖的话，Loback 会使用 GZ 压缩。
```xml
<dependency>
    <groupId>org.tukaani</groupId>
    <artifactId>xz</artifactId>
    <version>1.10</version>
</dependency>
```

## Layout
Layout 主要是决定日志的格式。

默认的 PatternLayout 已经能够满足大多数的需求。主要了解以 `%` 开头的 [conversion words](https://logback.qos.ch/manual/layouts.html#conversionWord) 就行了。 