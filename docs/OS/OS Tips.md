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

GCC 编译器驱动程序读取源程序文件 `hello.c`，并把它翻译成一个可执行目标文件 `hello`。这个翻译过程可分为四个阶段，如下图所示。
执行这四个阶段的程序（预处理器、编译器、汇编器和链接器）一起构成了编译系统（compilation system）。
![](../pictures/OS Tips-1.png)

### 预处理阶段
预处理器 cpp 根据以字符 `#` 开头的命令，修改原始的 C 程序。比如 hello.c 中的第一行的 `#include <stdio.h>` 告诉预处理器读取系统
头文件 `stdio.h` 的内容，并把它直接插入程序文本中。结果就得到另一个 C 程序，通常以 `.i` 作为文件的扩展名。

    只进行至预处理阶段的命令，如果不加 `-o` 选项的话直接输出到控制台而不是 `hello.i`。
    ```shell
    gcc -E hello.c -o hello.i
    ```

### 编译阶段
编译器 ccl 将文本文件翻译成文本文件 `hello.s`，它包含一个汇编程序。

    只进行至编译阶段的命令，此时 `-o` 是可选的，默认会生成一个 `hello.s` 文件。
    ```shell
    gcc -S hello.c|hello.i [-o hello.s]
    ```
### 汇编阶段
接下来，汇编器 as 将 `hello.s` 翻译成机器语言指令，把这些指令打包成一种叫做可重定位目标程序（relocatable object program）的格式，
并将结果保存在文件 `hello.o` 中。

    只进行至汇编阶段的命令，此时 `-o` 选项是可选的，默认会生成一个` hello.o` 文件。
    ```shell
    gcc -c hello.c|hello.i|hello.s [-o hello.o]
    ```

### 链接阶段
`hello` 程序调用了 `printf` 函数，它是标准 C 库中的函数。`printf` 函数存在于一个名为 `printf.o` 的单独的预编译好了的目标文件中，而这个
文件必须以某种方式合并到我们的 `hello.o` 程序中。链接器 `ld` 就负责处理这种合并，结果就是得到 `hello` 文件，它是一个可执行目标文件，可以被加载到内存中，
由系统执行。

    生成可执行文件的命令，如果不加 `-o` 选项的话默认生成的是 `a.out` 文件（assembly output，即汇编输出）。
    ```shell
    gcc hello.c|hello.i|hello.s|hello.o -o hello
    ```

## 开发系统引导

### 头文件
头文件用来提供对常量的定义和对系统函数及库函数调用的声明。对 C 语言来说，这些头文件几乎总是位于 `/usr/include` 目录及其子目录中。  
```text

# ls  /usr/include/
···
ctype.h      gnu-versions.h    netash      protocols           stdio_ext.h    video
dirent.h     grp.h             netatalk    pthread.h           stdio.h        wait.h
dlfcn.h      gshadow.h         netax25     pty.h               stdlib.h       wchar.h
elf.h        iconv.h           netdb.h     pwd.h               string.h       wctype.h
endian.h     ifaddrs.h         neteconet   python3.9           strings.h      wordexp.h
···
```
!!! note "GCC 头文件默认的搜索路径"
      可以使用如下命令查看 GCC 在编译 C 语言时默认的头文件搜索路径。
      ```shell
      gcc -xc -E -v -
      ```

在调用 C 语言编译器时，可以使用 -I 选项来包含保存在子目录或非标准位置中的头文件。例如：
```shell
gcc -I/home/computer/c fred.c
```
它指示编译器不仅在标准位置，也在 `/home/computer/c` 目录中查找程序 `fred.c` 中包含的头文件。

如果我们想查找用于从程序中返回退出状态的 #define 定义的名字，可以在 /usr/include 目录下使用 grep 命令进行搜索。
```text
# grep EXIT_ *.h
···
stdlib.h:#define        EXIT_FAILURE    1       /* Failing exit status.  */
stdlib.h:#define        EXIT_SUCCESS    0       /* Successful exit status.  */

```

### 库文件
`库`是一组预先编译好的函数的集合，这些函数是按照可重用的原则编写的。它们通常有一组相互关联的函数组成以执行某项常见的任务。

标准系统库文件一般存储在 /lib 和 /usr/lib 目录中。C 语言编译器（确切的说是链接器，`ld`）需要知道搜索哪些库文件，因为在默认情况下，它只搜索
标准 C 语言库。所以仅仅是把库文件放在标准目录中是不够的，还需要使库文件遵循特定的命名规范并且在命令行中明确指定。

库文件的名字总是 `lib` 开头，随后的部分指明这是什么库（例如，m 代表数据学库）。文件名的后缀一般为 `.a` 或者 `.so`。  

- .a 代表传统的静态函数库。
- .so 代表共享函数库。
#### 静态库

#### 共享库



## 从可执行进行反汇编

QA:

1. GCC 生成的汇编和 NASM 编译得到的汇编有什么不同？
2. 什么叫做可重定位目标程序？
3. 什么是标准 C 库，还有什么 C 库？
4. 为什么 C 语言的 `main` 函数可以不写 `return` 语句？  
   如果 `main` 函数中不写 `return` 语句，则默认返回 0，表示程序正常执行退出了。但是为了保持好的习惯，还是应该加上 `return 0;` 语句。

参考文档：

1. 《Linux 程序设计·第四版》
2. 《Unix 环境高级编程·第三版》
3. 《操作系统导论》
4. 《汇编语言·第四版》