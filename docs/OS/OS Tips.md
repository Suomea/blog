---
comments: true
---

## C 语言到可执行文件的步骤

```c
#include <stdio.h>

int main() {
    printf("hello world!\n");
}
```
编译成可执行文件，然后执行：
```shell
# gcc hello.c -o hello
# ./hello
hello world!
```

GCC 编译器驱动程序读取源程序文件 hello.c，并把它翻译成一个可执行目标文件 hello。这个翻译过程可分为四个阶段，如下图所示。
执行这四个阶段的程序（预处理器、编译器、汇编器和链接器）一起构成了编译系统（compilation system）。
![](../pictures/OS Tips-1.png)

1. 预处理阶段。预处理器 cpp 根据以字符 # 开头的命令，修改原始的 C 程序。比如 hello.c 中的第一行的 #include <stdio.h> 告诉预处理器读取系统
头文件 stdio.h 的内容，并把它直接插入程序文本中。结果就得到另一个 C 程序，通常以 .i 作为文件的扩展名。

    只进行至预处理阶段的命令，如果不加 -o 选项的话直接输出到控制台而不是 hello.i。
    ```shell
    gcc -E hello.c -o hello.i
    ```

2. 编译阶段。编译器 ccl 将文本文件翻译成文本文件 hello.s，它包含一个汇编程序。

    只进行至编译阶段的命令，此时 -o 是可选的，默认会生成一个 hello.s 文件。
    ```shell
    gcc -S hello.c|hello.i [-o hello.s]
    ```

3. 汇编阶段。接下来，汇编器 as 将 hello.s 翻译成机器语言指令，把这些指令打包成一种叫做可重定位目标程序（relocatable object program）的格式，
并将结果保存在文件 hello.o 中。

    只进行至汇编阶段的命令，此时 -o 选项是可选的，默认会生成一个 hello.o 文件。
    ```shell
    gcc -c hello.c|hello.i|hello.s [-o hello.o]
    ```

4. 链接阶段。hello 程序调用了 printf 函数，它是标准 C 库中的函数。printf 函数存在于一个名为 printf.o 的单独的预编译好了的目标文件中，而这个
文件必须以某种方式合并到我们的 hello.o 程序中。链接器 ld 就负责处理这种合并，结果就是得到 hello 文件，它是一个可执行目标文件，可以被加载到内存中，
由系统执行。

    生成可执行文件的命令，如果不加 -o 选项的话默认生成的是 a.out 文件（assembly output，即汇编输出）。
    ```shell
    gcc hello.c|hello.i|hello.s|hello.o -o hello
    ```

## 从可执行进行反汇编

QA:

1. GCC 生成的汇编和 NASM 编译得到的汇编有什么不同？
2. 什么叫做可重定位目标程序？
3. 什么是标准 C 库，还有什么 C 库？