
## Java 打包
目录树
```
├── META-INF
│   └── MANIFEST.MF
├── lib
│   └── commons-lang3-3.17.0.jar
└── src
    └── com
        └── otto
            └── Main.java

```

MANIFEST.MF 文件示例：
```
Manifest-Version: 1.0
Main-Class: com.otto.Main
Class-Path: lib/commons-lang3-3.17.0.jar
Created-By: 21.0.5 (Eclipse Adoptium)

```
`Class-Path` 如果有多个依赖需要手动指定每一个，不能使用通配符，多个使用空格隔开。**注意最后一行是空行。** 需要注意的是 `Class-Path` 指定的指定的依赖会打进 jar 包，如果不指定的话也是可以的，但是要在运行时指定 -cp。

Main.java 文件
```java
package com.otto;

import org.apache.commons.lang3.StringUtils;

public class Main {
    public static void main(String[] args) {
        System.out.println(StringUtils.equals("jacky", "suomea"));
    }
}
```

编译，-classpath 可以使用 -cp 缩写，能够使用通配符匹配文件。
```
javac -classpath .:lib/commons-lang3-3.17.0.jar -d . src/com/otto/Main.java
```
**`-d .`**：将编译后的 `.class` 文件输出到当前目录，按照包名生成对应的目录结构。

执行
```
java -classpath .:lib/commons-lang3-3.17.0.jar com.otto.Main
```

打包
```
 jar cvfm my-app.jar META-INF/MANIFEST.MF -C . com lib/
```
- **`cvfm`**：
	- `c`：创建 JAR 文件。
    - `v`：显示详细信息。
    - `f`：指定输出文件名。
    - `m`：指定使用的清单文件。
- **`-C .`**：切换到当前目录，包含 `com` 和 `lib/`。

检查 jar 包
```
jar tf my-app.jar
```

执行
```
java -jar my-app.jar
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