## Java 打包
目录树
```
├── META-INF
│   └── MANIFEST.MF
├── lib
│   └── fastjson2-2.0.57.jar
└── src
    └── com
        └── jacky
            └── Main.java
```

MANIFEST.MF 文件示例：
```
Main-Class: com.jacky.Main  
Class-Path: lib/fastjosn2-2.0.57.jar

```
`Class-Path` 如果有多个依赖需要手动指定每一个，不能使用通配符，多个使用空格隔开。**注意最后一行是空行。** 

Class-Path 是指定 java -jar 时的 classpath，相对于 jar 文件的目录而言的路径。注意，是 jar 包外部的路径。

Main.java 文件
```java
package com.jacky;  
  
import com.alibaba.fastjson2.JSONObject;  
  
public class Main {  
    public static void main(String[] args) {  
        JSONObject jsonObject = new JSONObject();  
        jsonObject.put("name", "jacky");  
  
        System.out.println(jsonObject.toJSONString());  
    }  
}
```

编译
```
javac -cp lib/* -encoding UTF-8 -d bin src/com/jacky/Main.java
```

执行
```
java -cp lib/*;bin com.jacky.Main
```

打包
```
jar -cfm my.jar META-INF/MANIFEST.MF -C bin com -C . lib
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