## `hello world`
先从一个入门的 `hello world!` 开始，代码：
```c
#include <stdio.h>

int main() {
    printf("hello world!\n");
}
```

运行
```
gcc p1.c
./a.out
```

## 华氏温度和摄氏度转换
`while` 循环实现：
```c
#include <stdio.h>

int main() {
    float fahr, celsius;
    float lower, upper, step;
    
    lower = 0;
    upper = 300;
    step = 20;
    
    fahr = lower;
    
    printf("%6s\t%9s\n", "Fahr/℉", "Celsius/℃");
    while (fahr <= upper)
    {
        celsius = 5 * (fahr - 32) / 9;
        printf("%6.0f\t%9.1f\n", fahr, celsius);
        fahr += step;
    }
    
}
```

`for` 循环实现：
```c
#include <stdio.h>

int main() {
    float fahr;
    
    printf("%6s\t%9s\n", "Fahr/℉", "Celsius/℃");
    for (fahr = 0; fahr <= 300; fahr += 20)
    {
        printf("%6.0f\t%9.1f\n", fahr, 5 * (fahr - 32) / 9);
    }
    
}
```

## 定义符号常量
类似 Java 中的常量，应该避免在程序中使用魔数，而是使用符号常量。

定义符号常量的语法，在该定义之后，程序中出现的 `name` 都会被替换为 `txt`。
```
#define name txt
```

示例：
```c
#include <stdio.h>

#define STEP 20
#define LOWER 0
#define UPPER 300

int main() {
    float fahr;
    
    printf("%6s\t%9s\n", "Fahr/℉", "Celsius/℃");
    for (fahr = LOWER; fahr <= UPPER; fahr += STEP)
    {
        printf("%6.0f\t%9.1f\n", fahr, 5 * (fahr - 32) / 9);
    }
    
}
```

## 将输入复制到输出

`getchar()` 函数从标准输入中读取一个字符，并在用户按下回车键时开始读取输入的字符。

`putchar(int c)` 函数将一个字符打印到标准输出。

`EOF` 是一个特殊值（注意，不是字符），标识输入结束。输入文件结束符的方式是按下Ctrl+D（在Unix/Linux系统下），或者Ctrl+Z（在Windows系统下）

```c
#include <stdio.h>

int main() {
    int c;
    while((c = getchar()) != EOF) {
        putchar(c);
    }
}
```

### 字符计数
`while` 实现字符计数：
```c
#include <stdio.h>

int main() {
    long nc;
    while (getchar() != EOF) {
        nc ++;
    }
    
    printf("%ld\n", nc);
    
}
```

`for` 实现行计数：
```c
#include <stdio.h>

int main() {
    long nl;
    int c;
    for (;(c = getchar()) != EOF;) {
        if(c == '\n') {
            nl ++;
        }
    }
    printf("%ld\n", nl);
}
```

### 单词计数
假设单词的分割符包括空格、制表符或换行符。代码示例：
```c
#include <stdio.h>

#define IN 1
#define OUT 0

int main() {
    int nw = 0, nl = 0, nc = 0;
    int c;
    int state = OUT;
    
    while((c = getchar()) != EOF) {
        nc ++;

        if (c == '\n') {
            nl ++;
        }
        
        if (c == ' ' || c == '\t' || c == '\n') {
            state = OUT;
        } else if(state == OUT) {
            state = IN;
            nw ++;
        }
    }
    printf("nc: %d, nw: %d, nl: %d\n", nc, nw, nl);
}
```


## 函数

### main 函数

  
在 C 语言中，`main` 函数可以接受两个参数：`argc` 和 `argv`。
- `argc`：这是一个整数，表示命令行参数的数量，包括程序的名称。即使没有提供额外的参数，`argc` 的值也至少为 1。

- `argv`：这是一个指向指针数组的指针，每个指针指向一个字符串，表示一个命令行参数。第一个参数是程序的名称，后续参数是传递给程序的参数。

其中 argc 和 argv 两个形参的参数类型和顺序是固定的，形参的名称则可以自定义。
```c
#include <stdio.h>

int main(int argc, char *argv[])
{

    printf("argument count: %d", argc);

    for(int i = 0; i < argc; i ++)
        printf("argument %d: %s", i, argv[i]);
}
```

运行结果：
```shell
# ./a.out hello world
argument count: 3
argument 0: ./a.out
argument 1: hello
argument 2: world
```

如果 `main` 函数没有定义 `return` 语句，则编译器会默认添加 `return 0;` 语句，返回 0 标识正常终止，非 0 标识出现异常情况或出错结束条件。

### 自定义函数
之前的代码使用的函数如 `printf`、`getchar` 和 `putchar` 都是库函数中提供的函数，自己实现一个求幂函数 `power(m, n)`。`power(m, n)` 函数用于计算求整数 m 的 n 次幂，其中 n 是正整数。

代码：
```c
#include <stdio.h>

int power(int m, int n);

int main() 
{
    for(int i = 0; i < 10; ++ i) 
        printf("%d %5d %7d\n", i, power(2, i), power(-3, i));
}
c
int power(int m, int n) {
    int i, p;

    p = 1;
    for (i = 0; i < n; i ++) {
        p = p * m;
    }
    
    return p;
}
```

在 main 函数之前有 power 函数的声明语句，表明 power 函数有两个 int 类型的参数，并返回一个 int 类型的值。
```c
int power(int m, int n);
```

这种声明成为函数原型，它必须与 power 函数定义和用法一致。函数原型中的参数名是可选的，也可以写成下面的形式：
```c
int power(int, int);
```


### 传值调用
在 C 语言中，所有的函数参数都是“通过值”传递的。

也就是说，传递给被调用函数的参数值存放在临时变量中，而不是存放在原来的变量中。因此在 C 语言中，被调用函数不能直接修改主调函数中变量的值，而只能修改其私有的临时副本的值。

## 字符数组
下面的程序读取一组文版，并把最长的文本行打印出来。
```c
#include <stdio.h>

#define MAXLINE 100

int get_line(char line[], int maxline);
void copy(char from[], char to[]);

int main() {
    int len;
    int max;
    
    char line[MAXLINE];
    char longest[MAXLINE];

    max = 0;
    while((len = get_line(line, MAXLINE)) > 0) {
        if (len > max) {
            max = len;
            copy(line, longest);
        }
    }

    if (max > 0) {
        printf("%s", longest);
    }

    return 0;
}

int get_line(char s[], int lim) {
    int c, i;
    
    for(i = 0; i < lim - 1 && (c = getchar()) != '\n' && c != EOF; i ++) {
        s[i] = c;
    }

    if (c == '\n') {
        s[i] = c;
        i ++;
    }

    s[i] = '\0';
    return i;
}

void copy(char from[], char to[]) {
    int i;

    for(i = 0; (to[i] = from[i]) != '\0'; i ++) {
        ;
    }
}
```

`get_line` 函数把字符 `\0` 即空字符插入到数组的末尾，以标记字符串的结束。所以在 C 语言中，声明 `"hello"` 字符串，在内存中会被存储为 `h` `e` `l` `l` `o` `\0`。

`printf` 函数中的格式规范 `%s` 规定，对应的参数必须是以这种形式标识的字符串。
```c
#include <stdio.h>

int main() {

    char tmp[] = {'a', 'b', 'c', '\0', 'd', 'e', 'f'};

    printf("%s\n", tmp);    // 打印 abc

    return 0;
}
```

另外 `copy` 函数的调用，传参也是值传递，传递的是数组的起始地址。

## 外部变量与作用域
上面的代码在函数内部声明的变量都称为”局部变量“，其它函数不能直接访问它们。函数中的每个局部变量只在函数调用时存在，在函数执行完毕退出时消失。

相对局部变量的是外部变量，外部变量是在所有函数中都可以访问的变量，而不必通过参数表。其次，外部变量在程序执行期间一直存在。

外部变量必须定义在所有函数之外，并且只能定义一次，定义后编译程序将为它们分配存储单元。在每个需要访问外部变量的函数中，必须声明相应的外部变量，以便对其类型进行说明。声明使用 `extern` 关键字进行显示声明，也可以通过上下文进行隐式声明。
```c
#include <stdio.h>

int global_var; // 定义 define

int main() {

    extern int global_var;  // 声明 declaration

    global_var = 100;

    printf("%d\n", global_var);
}
```

在源文件中，如果外部变量的定义出现在使用它的函数之前，那么在使用外部变量的函数中就可以省略 `extern` 声明。如果程序包含多个源文件，而某个外部变量定义在 file1，在 file2 和 file3 中使用就需要使用 `extern` 声明。当然一般使用头文件进行声明，并在每个源文件的开头使用 `#include` 语句把所要使用的头文件包含进来。
```c
#ifndef GLOBAL_VAR_FLAG
#define GLOBAL_VAR_FLAG
extern int global_var
#endif
```

!!! note "define 和 declaration"
	一般 `define` 定义，表示创建变量或分配存储单元；而 `declaration` 声明，指的是说明变量的性质，但不分配存储单元。
## 数据类型

基本数据类型有：char，short int，int，long int，float，double，long double。

char 类型占用一个字节。

short int 和 long int 类型的声明中，int 可以省略。int 通常代表机器中整数的自然长度。short 通常为 16 位，long 通常为 32 位，int 可以为 16 位或者 32 位。编译器可以根据硬件特性自主选择合适的长度，但是遵循下列限制：short 和 int 类型至少为 16 位，long 类型至少为 32 位，并且 short 不得长于 int，int 不得长于 long。

`signed` 和 `unsigned` 可以修饰 char 或者任何整型，比如 signed char 类型的变量取值范围为 -128~127，而 unsigned char 取值范围为 0~255。

float、double 和 long double 的长度也取决具体的实现。


默认值