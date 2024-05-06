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


## 数据类型c

类型和长度

默认值