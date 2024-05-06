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
```

```