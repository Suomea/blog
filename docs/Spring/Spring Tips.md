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