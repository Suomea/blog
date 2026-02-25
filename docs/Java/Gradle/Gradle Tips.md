## 配置 Gradle 单独的 JDK 环境
全局配置文件 `~/.gradle/gradle.properties`，项目配置文件 `project_dir/gradle.properties`：
```properties
org.gradle.java.home=D:\\App\\jdk-21.0.7+6
```

## 配置控制台显示详细的任务信息
```
命令行增加参数
--console=verbose

或者在项目配置文件增加配置
org.gradle.console=verbose
```

## 增量构建和缓存
模拟执行任务，但是实际不执行，常用来查看任务的执行顺序：
```
./gradlew :app:bootJar -m
```

增量构建
简单理解就是 Gradle 会缓存任务的执行结果，如果重新执行任务但是任务的输出没有变化的话，直接复用上次的执行结果。

构建缓存
默认是启用的，在项目的配置的 Gradle 配置文件中
```properties
org.gradle.caching=true
```

可以先模拟执行任务看一下任务的依赖关系 `-m`，然后在具体执行任务的时候使用 `--console=verbose` 输出每个任务的标签，查看是否重新执行还是缓存的。

注意，增量构建和构建缓存是两个不同的东西：
```
# 增量构建
> Task :app:compileJava UP-TO-DATE
# 输入未变，跳过执行

# 构建缓存
> Task :app:compileJava FROM-CACHE
# 从缓存获取输出
```

## 配置 lombok 依赖
```kotlin
compileOnly("org.projectlombok:lombok")  
annotationProcessor("org.projectlombok:lombok")
```

## Gradle 不同类型的项目结构
初始化项目，**basic**
```shell
gradle init --type basic --dsl kotlin --project-name basic --java-version 21
```

项目结构
```
├── build.gradle.kts
├── gradle
│   ├── libs.versions.toml
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
└── settings.gradle.kts
```

初始化项目，**java-library**
```shell
gradle init --type java-library --dsl kotlin --project-name java-lib --package com.jacky.lib --java-version 21
```

项目结构，在 basic 的基础上多出
```
├── lib
│   ├── build.gradle.kts
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── jacky
│       │   │           └── lib
│       │   │               └── Library.java
│       │   └── resources
│       └── test
│           ├── java
│           │   └── com
│           │       └── jacky
│           │           └── lib
│           │               └── LibraryTest.java
│           └── resources
```

初始化项目，**java-application**
```shell
gradle init --type java-application --dsl kotlin --project-name java-app --package com.jacky.app --java-version 21
```

项目结构，在 basic 的基础上多出
```
├── app
│   ├── build.gradle.kts
│   └── src
│       ├── main
│       │   ├── java
│       │   │   └── com
│       │   │       └── jacky
│       │   │           └── app
│       │   │               └── App.java
│       │   └── resources
│       └── test
│           ├── java
│           │   └── com
│           │       └── jacky
│           │           └── app
│           │               └── AppTest.java
│           └── resources
```
