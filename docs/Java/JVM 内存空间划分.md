Java 虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域有各自的用途，以及创建和销毁时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。

由所有线程共享的区域：  

- Method Area 方法区。  
- Head 堆。

线程隔离的区域：  

- VM Stack 虚拟机栈。  
- Native Method Stack 本地方法栈。  
- Program Counter Register 程序计数器。

