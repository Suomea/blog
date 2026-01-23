引入依赖：
```
id("org.springframework.boot").version("4.0.0")  
id("io.spring.dependency-management").version("1.1.7")

implementation("org.springframework.boot:spring-boot-starter-actuator")  
implementation("io.micrometer:micrometer-registry-prometheus")
```

配置：
```
server:  
  port: 8080  
  
management:  
  server:  
    port: 8081
  endpoints:  
    web:  
      base-path: /actuator  
      exposure:  
        include: prometheus
```

Prometheus 配置：
```yml
scrape_configs:
  - job_name: "java-service"
    metrics_path: "/actuator/prometheus"
    scheme: "http"
    static_configs:
      - targets: ["localhost:8081"]
        labels:
          app: "op"

```