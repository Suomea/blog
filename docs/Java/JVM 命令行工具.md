## Java 进程启动参数
Java 程序启动时可以通过命令行指定多种参数，分为两大类 JVM 参数和程序参数。

### 常用JVM 参数
标准参数  

- `-cp` 指定类路径。  
- `-D<property>=<value>` 设置系统属性。
 
非标准参数  

- `-Xms -Xmx` 设置堆的最小值和最大值。  
- `-Xss` 设置每个线程栈区的大小。
 
高级参数 

- `-XX:+UseG1GC` 启动 G1 垃圾回收器。  
- `-XX:+HeapDumpOnOutOfMemoryError` 在堆内存溢出时生成堆转储文件，默认存储在当前目录。  
- `-XX:HeapDumpPath=/tmp/dump.hprof` 指定堆转储文件的位置。

### 程序参数
传递给 main 方法的参数，存储在 main 方法的 args 数组中。注意，和 C 语言不一样，args 数组不包含程序名。

### 示例
```shell
java -Xmx128m -XX:+HeapDumpOnOutOfMemoryError -Dname=jacky com.otto.reflective.App arg1 arg2
```

## JPS
jsp 命令的用法
```
usage: jps [-q] [-mlvV] [<hostid>]

-q 只输出 Java 进程ID。

-m 输出程序和程序参数。
-l 输出全类名，如果是 jar 包输出 jar 路径。
-v 输出 JVM 参数。
-V 显示通过 .hotspotrc 文件或 -XX:Flags= 指定的 JVM 参数。
```

输出示例，记住默认使用 jsp -lm 就基本够用了。
```
% jps -q
3418

% jps -l
3418 com.otto.reflective.App

% jps -m
3418 App arg1 arg2

% jps -v
3418 App -Xmx128m -XX:+HeapDumpOnOutOfMemoryError -Dname=jacky

% jps -lvm
3418 com.otto.reflective.App arg1 arg2 -Xmx128m -XX:+HeapDumpOnOutOfMemoryError -Dname=jacky

% jps -lm
3418 com.otto.reflective.App arg1 arg2
```