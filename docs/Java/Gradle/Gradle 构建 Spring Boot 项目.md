
```
gradle init --type java-application --dsl kotlin --test-framework junit-jupiter --package com.jacky.gradle --project-name gradle-demo --no-split-project --java-version 21
```
--no-split-project
作用：不创建多项目结构
效果：生成单项目结构，所有代码都在根目录下
替代选项：省略此参数会创建多项目结构（根项目 + app 子项目）

配置文件 `libs.versions.toml`，新增配置：
```toml
[versions]  
spring-boot = "4.0.0"  
dependency-management = "1.1.7"  
  
[plugins]  
# spring boot 插件
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }  
# spring boot 依赖管理插件
spring-dependency-management = { id = "io.spring.dependency-management", version.ref = "dependency-management" }
```

配置文件 `build.gradle.kts`，新增配置：
```
plugins {  
    alias(libs.plugins.spring.boot)  
	alias(libs.plugins.spring.dependency.management)  
}

dependencies {  
    // 引入 spring-boot-starter-web 依赖
    implementation("org.springframework.boot:spring-boot-starter-web")  
}

tasks.bootJar {  
	// 设置打包的 jar 包名称
    archiveBaseName.set("gradle-demo.jar")  
	// 设置打 jar 包的时候先执行单元测试
    dependsOn(tasks.test)  
}
```

查看任务列表
```
gradlew tasks     # 列出所有的可用任务
gradlew build     # 构建项目
gradlew run       # 运行应用程序
gradlew test      # 运行测试
```

