## 初始化 Spring Boot 单体项目
根目录下直接放置代码和配置文件，不使用单独的子模块。

初始化项目
```
gradle init --type basic --dsl kotlin --project-name basic --java-version 21
```

编辑配置文件 `build.gradle.kts`：
```kotlin
plugins {  
    id("application")  
    id("org.springframework.boot").version("4.0.0")  
    id("io.spring.dependency-management").version("1.1.7")  
}  
  
repositories {  
    // Use Maven Central for resolving dependencies.  
    mavenCentral()  
}  
  
dependencies {  
    // 引入 spring-boot-starter-web 依赖  
    implementation("org.springframework.boot:spring-boot-starter-web")  
}  

// 配置打包任务的选项
tasks.bootJar {  
    // 设置打包的 jar 包名称  
    archiveBaseName.set("basic")  
    // 设置打包的 jar 包不带版本号  
    archiveVersion.set("")   // 关键  
    // 设置打 jar 包的时候先执行单元测试  
    dependsOn(tasks.test)  
}
```

新建目录：
```
mkdir -p src/main/java/com/jacky/basic
```

创建主类：
```java
package com.jacky.basic;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RequestMapping;  
import org.springframework.web.bind.annotation.RequestParam;  
import org.springframework.web.bind.annotation.RestController;  
  
@RestController  
@RequestMapping("app")  
@SpringBootApplication  
public class BasicApp {  
    public static void main(String[] args) {  
        SpringApplication.run(BasicApp.class, args);  
    }  
  
    @GetMapping("hello")  
    public String hello(@RequestParam String name) {  
        return String.format("hello, %s!", name);  
    }  
}
```

启动测试：
```
./gradlew bootRun --args="--spring.application.name='basic app' --server.port=8080"
curl localhost:8080/app/hello?name=basic
```

打包测试：
```
./gradlew bootJar
cd build/lib
java -jar basic.jar
curl localhost:8080/app/hello?name=basic
```

## 初始化 Spring Boot 微服务项目
初始化父项目
```
gradle init --type basic --dsl kotlin --project-name ms --java-version 21
```

编辑父项目的 settings.gradle.kts 配置文件
```kotlin
rootProject.name = "ms"

include("api", "service")
```

创建子模块的目录
```
mkdir api service
```
