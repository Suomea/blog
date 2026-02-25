## 文件上传实时处理
Spring boot 默认的 MultipartFile 会将文件全部接收缓存在本地路径，然后再进入 Controller 进行处理。这时如果文件过大本地路径放不下的话就会失败。

使用 Apache Commons FileUpload [Streaming](https://commons.apache.org/proper/commons-fileupload/streaming.html) 进行处理，但是需要配置。
```yaml
spring:  
  servlet:  
    multipart:  
      enabled: false
```

## 设置微服务不注册到 Eureka
设置微服务不注册到 Eureka 注册中心，但是能够调用注册中心微服务的接口。
```yaml
eureka:  
  client:  
    register-with-eureka: false  # 不注册自身  
    fetch-registry: true         # 仍然获取注册表信息
```


## Spring Boot 设置 deubg 日志
全局 debug：
```yml
debug: true
```

应用配置文件配置 logger 级别为 debug：
```yml
logging:  
  level:  
    com.jacky.slog.App: debug
```

日志配置文件配置 logger 级别为 debug：
```xml
<logger name="com.jacky.slog.App" level="DEBUG"/>
```

Tips：应用配置文件的配置会覆盖日志配置文件的配置。

Spring Boot 会扫描 jar 包当前目录 config 子目录下的应用配置文件，优先级高于 jar 包内的应用配置文件。可以将日志配置文件也放在 config 子目录下，然后在应用配置文件中设置日志配置文件的路径。
```yml
logging:  
  config: file:config/logback-spring.xml
```