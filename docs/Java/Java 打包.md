## jar
`jar` 命令是 Java 提供的，用于创建、更新、查看和提取 JAR 文件。

创建 JAR 文件
```
jar cvfm my.jar META-INF/MANIFEST.MF -C classes . -C resources .

c: 创建新的 JAR 文件。
v: 在标准输出中生成详细输出。
f: 指定 JAR 文件的名称。
m: 指定 MANIFEST.MF 文件，该文件包含了 JAR 文件的一些元数据。
-C: 切换到指定的目录，[.] 表示将该目录中的所有文件添加到 JAR 中。
```

查看 JAR 文件
```
jar tf my.jar

t: 列出 JAR 文件中的内容。
```

提取 JAR 文件
```
jar xf my.jar

x: 从 JAR 文件中提取文件。
```

更新 JAR 文件
```
jar uf my.jar META-INF/MANIFEST.MF
jar uf my.jar -C classes .

u: 添加文件到 JAR，如果文件已经存，则更新。
```
## Java 打包
目录树
```
.
├── lib
│   ├── logback-classic-1.5.18.jar
│   ├── logback-core-1.5.18.jar
│   └── slf4j-api-2.0.17.jar
├── META-INF
│   └── MANIFEST.MF
├── resources
│   └── logback.xml
└── src
    └── com
        └── jacky
            └── Main.java
```

MANIFEST.MF 文件示例：
```
Main-Class: com.jacky.Main  
Class-Path: lib/logback-classic-1.5.18.jar lib/logback-core-1.5.18.jar lib/slf4j-api-2.0.17.jar

```
`Class-Path` 如果有多个依赖需要手动指定每一个，不能使用通配符，多个使用空格隔开。**注意最后一行是空行。** 
`Class-Path` 指定的类路径，项目目录是 jar 包所在的目录。

编译和执行
```
javac -cp "lib/*" -d classes src/com/jacky/Main.java
java -cp "lib/*:classes" com.jacky.App
```

编译、打包和执行
```
javac -cp "lib/*" -d classes -encoding UTF-8 src/com/jacky/*.java
jar cvfm my.jar META-INF/MANIFEST.MF -C classes/ . -C resources/ .
java -jar my.jar
```
## Spring Boot 打包
Spring Boot 配置打包，如果项目继承了 spring-boot-starter-parent，无需额外配置 repackage，简化 pom.xml。
```xml
    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

一个好用的反编译脚本
需要下载 cfr jar 包，如果使用 jd-gui 进行反编译的话内部类处理不了，使用 cfr 能够处理内部类的反编译。
```bash
#!/bin/bash

# 指定搜索目录
search_dir="/Users/jacky/Desktop"
# 输出目录
output_dir="decompiled_classes"
mkdir -p "$output_dir"

# 使用 find 命令递归查找 .class 文件并反编译
find "$search_dir" -type f -name "*.class" | while read -r class_file; do
    # 获取相对路径
    relative_path="${class_file#$search_dir/}"
    # 创建对应的目录结构
    mkdir -p "$output_dir/$(dirname "$relative_path")"
    # 使用 CFR 反编译器示例，或替换为其他反编译器
    java -jar cfr.jar "$class_file" > "$output_dir/${relative_path%.class}.java"
done
```

可能会生成内部类的文件，删除命令。
```
find /Users/jacky/Desktop -type f -name '*$*.java' -exec rm -rf {} \;
```

反编译 jar 文件
```
java -jar cfr.jar /path/to/your/file.jar --outputdir /path/to/output/directory
```