Java 虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域有各自的用途，以及创建和销毁时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。

由所有线程共享的区域：  

- Method Area 方法区。  
- Head 堆。

线程隔离的区域：  

- VM Stack 虚拟机栈。  
- Native Method Stack 本地方法栈。  
- Program Counter Register 程序计数器。

## 方法区
方法区主要存储类的元数据、常量、静态变量即时编译后的字节码缓存等数据。

在 JDK 8 之前方法区的实现称为永久代（PermGen）从。 JDK 8 开始，永久代被移除，方法区的实现该为元空间（Metaspace），元空间使用本地内存（Native Memory），不再受 JVM 堆内存的限制。

字符串常量池在 JDK 7 之前也是在方法区的，但是从 JDK 7 及之后，字符串常量池被移到了堆中。这样可以充分利用 JVM 的垃圾回收机制，及时清理不再使用的字符串常量。

字符串常量池用于存储字符串常量如 `a = "hello"` 和通过 `new String("hello").intern();` 方法显示加入的字符串。
## 堆
堆主要存储对象实例和数组。

使用 jmap 监控堆内存的使用情况，默认输出的单位为 B。
```
# jmap -heap 170260
Heap Configuration:
   MinHeapFreeRatio         = 40 // 当堆的空闲内存低于 40% 时，JVM 会尝试扩展堆
   MaxHeapFreeRatio         = 70 // 当堆的空闲内存高于 70% 时，JVM 会尝试收缩堆
   MaxHeapSize              = 448790528 (428.0MB) // 堆的最大大小
   NewSize                  = 9764864 (9.3125MB) // 新生代的初始大小
   MaxNewSize               = 149553152 (142.625MB) // 新生代的最大大小
   OldSize                  = 19595264 (18.6875MB) // 老年代的初始大小
   NewRatio                 = 2 // 新生代与老年代的比例，老年代时新生代的 2 倍
   SurvivorRatio            = 8 // 新生代 Eden 和 Survivor 的比例，Eden 占区 8，Survivor 区占 2
   MetaspaceSize            = 21807104 (20.796875MB) // 元空间的初始大小
   CompressedClassSpaceSize = 1073741824 (1024.0MB) // 压缩类空间大小
   MaxMetaspaceSize         = 17592186044415 MB // 元空间的最大大小，理论上无限大（当然受限物理内存的限制）
   G1HeapRegionSize         = 0 (0.0MB) // G1 垃圾回收器的 Region 大小，为 0 表示未使用 G1 垃圾回收器

Heap Usage:
New Generation (Eden + 1 Survivor Space): // 新生代的大小，From Space 和 To Space 都属于 Survivor
   capacity = 8847360 (8.4375MB)
   used     = 817776 (0.7798919677734375MB)
   free     = 8029584 (7.6576080322265625MB)
   9.2431640625% used
Eden Space:
   capacity = 7929856 (7.5625MB)
   used     = 817776 (0.7798919677734375MB)
   free     = 7112080 (6.7826080322265625MB)
   10.312621061466942% used
From Space:
   capacity = 917504 (0.875MB)
   used     = 0 (0.0MB)
   free     = 917504 (0.875MB)
   0.0% used
To Space:
   capacity = 917504 (0.875MB)
   used     = 0 (0.0MB)
   free     = 917504 (0.875MB)
   0.0% used
tenured generation: // 老年代的大小
   capacity = 19595264 (18.6875MB)
   used     = 0 (0.0MB)
   free     = 19595264 (18.6875MB)
   0.0% used

1162 interned Strings occupying 80544 bytes. // 字符串常量池中有1163 个字符串，占用 80544 个字节
```

使用 jstac 监控堆内存的使用情况，默认输出单位为 KB。
```shell
# jstat -gc 170260
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
896.0  896.0   0.0    0.0    7744.0   798.6    19136.0      0.0     4480.0 779.1  384.0   76.0       0    0.000   0      0.000    0.000
```

| 字段     | 全程                                 | 含义                                          |
| ------ | ---------------------------------- | ------------------------------------------- |
| `S0C`  | Survivor 0 Capacity                | Survivor 0 区（From Survivor）的容量（单位：KB）。      |
| `S1C`  | Survivor 1 Capacity                | Survivor 1 区（To Survivor）的容量（单位：KB）。        |
| `S0U`  | Survivor 0 Utilization             | Survivor 0 区（From Survivor）已使用的内存（单位：KB）。   |
| `S1U`  | Survivor 1 Utilization             | Survivor 1 区（To Survivor）已使用的内存（单位：KB）。     |
| `EC`   | Eden Capacity                      | Eden 区的容量（单位：KB）。                           |
| `EU`   | Eden Utilization                   | Eden 区已使用的内存（单位：KB）。                        |
| `OC`   | Old Capacity                       | 老年代（Old Generation）的容量（单位：KB）。              |
| `OU`   | Old Utilization                    | 老年代（Old Generation）已使用的内存（单位：KB）。           |
| `MC`   | Metaspace Capacity                 | 元空间（Metaspace）的容量（单位：KB）。                   |
| `MU`   | Metaspace Utilization              | 元空间（Metaspace）已使用的内存（单位：KB）。                |
| `CCSC` | Compressed Class Space Capacity    | 压缩类空间（Compressed Class Space）的容量（单位：KB）。    |
| `CCSU` | Compressed Class Space Utilization | 压缩类空间（Compressed Class Space）已使用的内存（单位：KB）。 |
| `YGC`  | Young Generation GC Count          | 年轻代（Young Generation）垃圾回收的次数。               |
| `YGCT` | Young Generation GC Time           | 年轻代（Young Generation）垃圾回收的总时间（单位：秒）。        |
| `FGC`  | Full GC Count                      | Full GC（全局垃圾回收）的次数。                         |
| `FGCT` | Full GC Time                       | Full GC（全局垃圾回收）的总时间（单位：秒）。                  |
| `GCT`  | GC Time                            | 所有垃圾回收的总时间（单位：秒）。                           |