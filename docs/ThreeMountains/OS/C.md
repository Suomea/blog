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

C 语言程序可以看成由一些列外部对象构成，这些外部对象可能是变量或函数。外部变量定义在函数之外，可以在函数中使用。而函数本身就是“外部的”。默认情况下，外部变量具有下列性质：通过一个名字对外部变量的所有引用实际上都是引用同一个对象。

### 静态变量
外部变量可以使用 static 进行修饰，此时就不再是外部变量了，而是静态变量。静态变量仅供其所在的源文件中的函数使用。
```c
static int global_var = 10;
```

static 也可以声明函数，如果把函数声明为 static 类型，则改函数名只能对该函数所在的文件可见。
```c
static void helperFund() {
	// some code
}
```

static 也可以声明内部变量，即在函数内部声明的变量。与自动变量不同的是，如果使用 static 修饰内部变量，不管函数是否调用，它一直存在，而不会随着所在函数的调用和退出而存在和消失。
```c
void myFunc() {
    static int count = 0;
    count++;
    printf("Count: %d\n", count);
}
```
### 寄存器变量
`register` 声明高速编译器，它所声明的变量在程序中使用频率较高。其思想是，将 register 变量放在机器的寄存器中，这样可以使程序更小、执行速度更快。然而，现代编译器通常会忽略这种建议，因为它们能够更好地决定变量应该存储在何处，以获得最佳性能。

register 声明只适用于自动变量和函数参数的形式。

## 数据类型

基本数据类型有：char，short int，int，long int，float，double，long double。

char 类型占用一个字节。

short int 和 long int 类型的声明中，int 可以省略。int 通常代表机器中整数的自然长度。short 通常为 16 位，long 通常为 32 位，int 可以为 16 位或者 32 位。编译器可以根据硬件特性自主选择合适的长度，但是遵循下列限制：short 和 int 类型至少为 16 位，long 类型至少为 32 位，并且 short 不得长于 int，int 不得长于 long。

`signed` 和 `unsigned` 可以修饰 char 或者任何整型，比如 signed char 类型的变量取值范围为 -128~127，而 unsigned char 取值范围为 0~255。

`float`、`double` 和 `long double` 的长度也取决具体的实现。

有关基本类型的取值范围，整形可以在 `<limits.h>` 中获取，浮点型可以在 `<float.h>` 文件中获取。
```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

int main() {
    // signed types
    printf("signed char min     = %d\n", SCHAR_MIN);
    printf("signed char max     = %d\n", SCHAR_MAX);
    printf("signed short min    = %d\n", SHRT_MIN);
    printf("signed short max    = %d\n", SHRT_MAX);
    printf("signed int min      = %d\n", INT_MIN);
    printf("signed int max      = %d\n", INT_MAX);
    printf("signed long min     = %ld\n", LONG_MIN);
    printf("signed long max     = %ld\n", LONG_MAX);

    // unsigned types
    printf("unsigned char max     = %u\n", UCHAR_MAX);
    printf("unsigned short max    = %u\n", USHRT_MAX);
    printf("unsigned int max      = %u\n", UINT_MAX);
    printf("unsigned long max     = %lu\n", ULONG_MAX);

    printf("float min: %E\n", FLT_MIN);
    printf("float max: %E\n", FLT_MAX);
    printf("double min: %E\n", DBL_MIN);
    printf("double max: %E\n", DBL_MAX);
    printf("long double max: %E\n", LDBL_MAX);
    printf("long double max: %E\n", LDBL_MAX);
}
```

### 初始化
在不进行显示初始化的情况下，外部变量和静态变量都将被初始化为 0，自动变量和寄存器变量的初始值没有定义（没有意义）。

对于结构体变量，如果没有显示初始化，则内部的变量则会根据类型自动初始化默认值。
## 函数
一个设计得当的函数可以把程序中不需要了解的具体操作细节隐藏起来，从而使整个程序结构更加清晰，并降低程序的修改难度。

### 查找文本
编写一个程序，将输入中包含特定“模式”或字符串的各行打印出来。例如，在下面文本中查找包含字符串 “ould” 的行：
```
Ah Love! could you and I with Fate conspire
To grasp this sorry Scheme of Things entire,
Would not we shatter it to bits -- and then
Re-mould it nearer to the Heart's Desire
```

程序执行后输出下列结果：
```
Ah Love! could you and I with Fate conspire 
Would not we shatter it to bits -- and then 
Re-mould it nearer to the Heart's Desire!
```

代码如下：
```c
#include <stdio.h>

#define MAXLINE 100

char pattern[] = "ould";

int get_line(char*, int);

int strindex(char*, char*);

int main() {

    char line[MAXLINE];

    while(get_line(line, MAXLINE) > 0) {
        if (strindex(line, pattern) >= 0) {
            printf("%s", line);
        }
    }
}

int get_line(char *line, int lim) {
    int c;
    int i = 0;

    while(--lim > 0 && (c = getchar()) != '\n' && c != EOF) {
        line[i++] = c;
    }

    if (c == '\n') {
        line[i++] = c;
    }

    line[i] = '\0';
    return i;
}

int strindex(char *line, char *p) {
    int i, j, k;

    for(i = 0; line[i] != '\0'; i ++) {
        j = i;
        k = 0;

        while (p[k] != '\0' && line[j] == p[k]) {
            j ++;
            k ++;
        }

        if (k > 0 && p[k] == '\0') {
            return i;
        }
    }
    return -1;
}
```

### 计算后缀表达式的值
中缀表达式：`(1 + 2) * 3` 对应的后缀表达式为：`1 2 + 3 *` 。后缀表达式也成为逆波兰表达式。

`calc.h` 代码为头文件，定义宏并且声明函数。
```c
// calc.h
#define NUMBER '0'

void push(double);

double pop();

int getop(char []);

int getch();

void ungetch(int);
```

后缀表达式的求值逻辑：由左向右逐个读取后缀表达式的元素，如果读取的元素为操作数则将操作数压入栈中，如果读取的元素为操作符，则取出栈顶的两个操作数，进行计算并将计算结果入栈。  
```c
// main.c
#include <stdio.h>
#include <stdlib.h>
#include "calc.h"

#define MAXOP 100

int main() {
    int type;
    double op2;
    char s[MAXOP];

    while((type = getop(s)) != EOF) {
        switch (type) {
        case NUMBER:
            push(atof(s));
            break;
        case '+':
            push(pop() + pop());
            break;
        case '*':
            push(pop() * pop());
            break;
        case '-':
            op2 = pop();
            push(pop() - op2);
            break;
        case '/':
            op2 = pop();
            if (op2 != 0.0) {
                push(pop() / op2);
            } else {
                printf("error: zero divisor\n");
            }
            break;
        case '\n':
            printf("\t%.8g\n", pop());
            break;
        default:
            printf("error: unknown command %s\n", s);
            break;
        }
    }
    return 0;
}
```

`stack.c` 封装栈的基本操作。
```c
stack.c
#include <stdio.h>
#include "calc.h"

#define MAXVAL 100
c
int sp = 0;

double val[MAXVAL];

void push(double f) {
    if(sp < MAXVAL) {
        val[sp ++] = f;
    } else {
        printf("error: stack full, can't push %g\n", f);
    }
}

double pop() {
    if(sp > 0) {
        return val[--sp];
    } else {
        printf("error: stack empty\n");
        return 0.0;
    }
}
```

`getop.c` 代码的作用是解析后缀表达式中的每个元素。
```c
// getop.c
#include <stdio.h>
#include <ctype.h>
#include "calc.h"

int getch();
void ungetch(int);

int getop(char s[]) {
    int i, c;

    // 获取第一个操作字符，忽略前置的空格和 TAB
    while((s[0] = c = getch()) == ' ' || c == '\t') {
        ;
    }
    s[1] = '\0';

    // 判断字符如果是不属于数字直接返回
    if (!isdigit(c) && c != '.' && c != '-') {
        return c;
    }

    i = 0;
    if (c == '-') {
        if (isdigit(c = getch()) || c == '.') {
            s[++i] = c;
        } else {
            if (c != EOF) {
                ungetch(c);
            }
            return '-';
        }   
    }
    
    // 如果是多位数
    if (isdigit(c)) {
        while(isdigit(s[++i] = c = getch())) {
            ;
        }
    }
    // 如果遇到小数位继续获取后续的小数部分
    if (c == '.') {
        /* code */
        while(isdigit(s[++i] = c = getch())) {
            ;
        }
    }

    s[i] = '\0';

    // 缓存多读取的下一个元素
    if (c != EOF) {
        ungetch(c);
    }
    return NUMBER;
}
```

`getch.c` 代码的作用是缓存读取的字符，因为 `getop.c` 为了获取一个元素，往往要多读取一个字符来判断当前元素是不是结束，所以要将多读取的字符缓存起来。
```c
// getch.c
#include <stdio.h>
#define BUFSIZE 100

char buf[BUFSIZE];
int bufp = 0;

int getch() {
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c) {
    if (bufp >= BUFSIZE) {c
        printf("ungetch: too many characters\n");
    } else {
        buf[bufp ++] = c;
    }
    
}
```
## 预处理
### 文件包含
`#include <>` 用来引用标准库头文件，这些头文件通常已经安装在系统的标准目录中。编译器会在这个标准目录中查找头文件并将其包含到代码中。  
 
`#include ""` 用来引用自定义头文件，这些头文件通常是用户自己编写的。编译器会在当前文件所在目录中查找头文件并将其包含进代码中。

### 宏替换
最简单的宏替换，即定义一个名字和替换文本，后续出现的所有名字都会被文本替换。
```c
#define name text
```

`#define` 指令定义的名字的作用域从其定义点开始，到被编译的源文件的末尾处结束。宏定义也可以使用前面出现的宏定义。

替换只对记号进行，对括在引号的字符串不起作用。例如，如果 YES 是一个通过 `#define` 指令定义过的名字，则在 printf("YES") 或 YESMAN 中将不执行替换。

替换可以是任意文本的，例如，为无限循环定义了一个新名字 forever。
```
#define forever for(;;)
```

宏定义也可以带参数，但是要注意表达式副作用的或者计算顺序的问题。

下面的宏定义就存在表达式的副作用问题，如果调用 `max(i++, j++)` 就会暴露缺陷。
```c
#define max(A,B) ((A) > (B) ? (A) : (B))
```

下面的宏定义就存在计算顺序问题，如果调用 `squrare(z + 1)` 就会暴露缺陷。
```c
#define squrare(x) x * x
```

可以通过 `#undef` 指令取消名字的宏定义。

## 指针
指针是一种保存变量地址的变量。可以把指针看着一种特殊类型的变量，为指针类型，并且值只能是地址。

一元运算符 & 可用于取一个对象的地址。
```c
p = &c;
```

上述表达式把变量 c 的地址赋值给变量 p，称 p 为“指向” c 的指针。

地址运算符 & 只能应用于内存中的对象，即变量与数组。它不能作用于表达式、常量或 `register` 类型的变量。
```c
int x = 1, y = 2, z[10];
int *ip;
ip = &x;    // ip now points x
y = *ip;    // y is now 1
*ip = 0;    // x is now 0
ip = &z[0];    // ip now points to z[0]
```

### 指针与函数
由于 C 语言是以传值的方式将参数值传递给被调用函数。因此，被调用函数不能直接修改主调函数中变量的值。

如下的 swap 函数就是错误的，它仅仅交换了 a 和 b 的副本的值。
```c
void swap(int x, int y) {
	int tmp = x;
	x = y;
	y = tmp;
}
```

正确的定义如下：
```c
void swap(int *x, int *y) {
	int tmp = *x;
	*x = *y;
	*i = tmp;
}
```
### 指针与数组
在 C 语言中指针与数组的关系十分密切。一般来说，用指针编写的程序比用数组下标编写的程序执行速度更快，但另一方面，用指针实现的程序理解起来稍微困难一些。

定义个长度为 10 的数组 a，`a[i]` 表示数组的第 `i` 个元素。
```c
int a[10];
```

定义一个指向整型对象的指针 `pa`，并且将指针 `pa` 指向数组 a 的第 0 个元素。
```c
int *pa;
pa = &a[0];
```

如果 `pa` 指向数组中的某个特定元素，那么，根据指针运算的定义，`pa + 1` 将指向下一个元素。因此，如果 pa 指向 `a[0]`，那么 `*(pa + 1)` 引用的是素组元素 `a[1]` 的内容，`pa + i` 是数组元素 `a[i]` 元素的地址，`*(pa + i)` 引用的是数组元素 `a[i]` 的内容。无论数组 a 中元素的类型或者数组的长度是什么，以上结论都成立。

根据定义，数组类型的变量或表达式的值是该数组第 0 个元素的地址。下面两条赋值语句等价。
```c
pa = &a[0];
pa = a;
```

对数组元素 `a[i]` 的引用也可以写成 `*(a + i)` 这种形式，因此 `&a[i]` 和 `a + i` 的含义也是相同的。响应地，如果 pa 是个指针，那么，在表达式中也可以在它后面加下表。`pa[i]` 与 `*(pa + i)` 是等价的。

但是，数组名和指针有一个不同之处，指针是一个变量，因此，在 C 语言中语句 `pa = a` 和 `pa ++` 都是合法的。但数组名不是变量，因此，类似与 `a = pa` 和 `a++` 形式的语句都是非法的。

当把数组名传递给一个函数时，实际上传递的是该数组第一个元素的地址，即指针。因此在函数定义中，形参 `char s[]` 和 `char *s` 是等价的。
```c
#include <stdio.h>

int str_len(char *s);

int main() {
    char s1[] = {'a', 'b', 'd', '\0'};
    char *s2 = "hello world!";
    
    printf("s1 len: %d\n", str_len(s1 + 1));    // 2
    printf("s2 len: %d\n", str_len(s2));    // 12
}

int str_len(char *s) {
    int n;
    for(n = 0; *s != '\0'; s ++) {
        n ++;
    }
    return n;
}
```

由于数组的传递实际上传递的是指针，因此在函数内部使用 sizeof 是获取不到数组的长度。因此在下面的代码中 `sizeof line` 和 `sizeof(char *)` 是等价的。
```c
int get_line(char line[]) {
    printf("get_line line len: %d\n", sizeof line);
    printf("get_line line len: %d\n", sizeof(char *));
    
    return 0;
}
```

!!! note "指针的初始化"
	同其它类型的变量一样，指针也可以初始化。通常针对指针有意义的初始化值只能是 0 或者表示地址的表达式。C 语言保证，0 永远不是有效的数据地址，因此，返回值 0 可用来表示发生了异常事件。
	指针与整数之间不能互相转换，但是 0 是唯一的例外：常量 0 可以赋值给指针，指针也可以和常量 0 进行比较。程序中常用符号 `NULL` 代替常量 0，符号常量 NULL 定义在标准头文件 `<stddef.h>` 中。

在某些情况下指针可以及进行比较运算，例如，如果指针 p 和 q 指向同一个数组的成员，那么它们之间就可以进行类似与 `==`、`!=`  的关系比较运算。但是，指向不同数组的元素的指针之间的算术或比较运算没有意义。
如果 p 指向的元素的位置在 q 指向的元素的位置之前，那么下面的表达式为真。
```c
p < q
```

指针可以和整数进行相加或相减运算，例如 `p + n` 表示 p 当前指向的对象之后第 n 个对象的地址。无论指针 p 指向的对象是何种类型，上述结论都成立。

指针的相减运算也是有意义的：如果 p 和 q 指向相同数组中的元素，且 p < q，那么 `q - p + 1` 就是位于 p 和 q 指向的元素之间元素的数目。

有效的指针运算包括相同类型指针之间的赋值运算；指针同整数时间的加减法运算；指向相同数组中元素的两个指针间的减法或比较运算；将指针赋值为 0 或指针与 0 之间的比较运算。其它所有形式的指针运算都是非法的。
### 字符指针与函数
字符串常量是一个字符数组，例如，`"I am a string"` 在字符串的内部表示中，字符数组以空字符 `\0` 结尾，所以，可以通过检查空字符找到字符数组的结尾。字符串常量占据的存储单元数也因此比双引号内的字符数大 1。

字符串常量一个常见的用法是作为函数参数，例如，下面代码实际上是通过字符指针访问该字符串的。
```c
printf("hello world\n");
```

下面两个语句的定义之间有很大的差别：
```c
char amessage[] = "now is the time";
char *pmessage = "now is the time";
```

`amessage` 是一个仅仅足以存放初始化字符串以及空字符 `\0` 的一维数组。数组中的字符可以修改，但是 amessage 始终指向同一存储位置。
`pmessage` 是一个指针，其初始值指向一个字符串常量，之后它可以被修改指向其它地址，**但如果试图修改字符串的内容，其结果是没有定义的。**

下面是使用指针实现的字符串复制函数，提供将字符串指针 `t` 指向的字符串复制到 `s`。
```c
#include <stdio.h>

void str_copy1(char*, char*);
void str_copy2(char*, char*);
void str_copy3(char*, char*);

int main() {
    
    char s1[] = "hello world!";
    char t1[] = "nihao";
    
    str_copy1(s1, t1);
    printf("%s\n", s1);  // nihao
    
    char s2[] = "how much!";
    char t2[] = "duoshao";
    
    str_copy2(s2, t2);
    printf("%s\n", s2); // duoshao
    
    char s3[] = "thank you!";
    char t3[] = "tks";
    
    str_copy3(s3, t3);
    printf("%s\n", s3); // tks
}

void str_copy1(char *s, char *t) {
    int i = 0;
    
    while((s[i] = t[i]) != '\0') {
        i ++;
    }
}

void str_copy2(char *s, char *t) {
    while((*s = *t) != '\0') {
        s ++;
        t ++;
    }
}

void str_copy3(char *s, char *t) {
    while((*s ++ = *t ++) != '\0') {    // 此处也可以省略和空字符 '\0' 的比较，因为空字符的值为 0 即 false
        ;
    }
}

void strcpy(char *s, char * t) {
    while(*s ++ = *t ++) {
        ;
    }
}
```

下面是字符串比较函数，比较字符串 s 和 t，并且根据 s 按照字典顺序小于、等于或大于 t 的结果分别返回负整数、0 或正整数。
```c
#include <stdio.h>

int strcmp1(char *, char *);
int strcmp2(char *, char *);

int main()
{
    char str1[] = "a!a";   
    char *str2 = "a!";
    
    printf("strcmp1 result: %d\n", strcmp1(str1, str2));
    printf("strcmp2 result: %d\n", strcmp2(str1, str2));
}

int strcmp1(char *s, char *t)
{
    int i = 0;
    while (s[i] == t[i])
    {
        if (s[i] == '\0')
        {
            return 0;
        }
        i++;
    }

    return s[i] - t[i];
}

int strcmp2(char *s, char *t)
{
    while (*s == *t)
    {
        if (*s == '\0')
        {
            return 0;
        }
        s++, t++;
    }
    return *s - *t;
}
```

### 二维数组

```c
#include <stdio.h>

int main() {
    int day_of_year(int, int, int);
    void month_day(int, int, int*, int *);
    
    printf("2024-03-10: %d\n", day_of_year(2024, 3, 10)); //2024-03-10: 70
    
    
    int month = 0;
    int day = 0;
    month_day(2024, 70, &month, & day);
    printf("2024 70, month: %d, day: %d\n", month, day); // 2024 70, month: 3, day: 10

}

static char daytab[2][13] = {
    {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
    {0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
};

int day_of_year(int year, int month, int day) {
    int i, leap;
    leap = year % 4 == 0 && year % 100 != 0 || year % 400 ==0;
    for(i = 1; i < month; i ++) {
        day += daytab[leap][i];
    }
    return day;
}

void month_day(int year, int yearday, int *pmonth, int *pday) {
    int i, leap;
    leap = year % 4 == 0 && year % 100 != 0 || year % 400 ==0;
    for(i = 1; yearday > daytab[leap][i]; i ++) {
        yearday -= daytab[leap][i];
    }
    *pmonth = i;
    *pday = yearday;
}
```

在 C 语言中，二维数组是一种特殊的一维数组，它的每一个元素也是一个数组。可以表示为
	`daytab[i][j]`    // `[row][col]`

如果将二维数据作文参数传递给函数 f，那么 f 的声明可以写成如下形式。
```c
f(int daytab[2][13]) {}

f(int daytab[][13]) {}

f(int (*daytab)[13]) {}
```

需要注意的是 `[]` 的优先级高于 `*`。所有下面两种声明方式是有区别的：
- `int (*daytab)[13]` 表示 daytab 是一个指针，指向包含 13 个整型元素的数组。指向数组的指针。
- `int *daytab[13]`  表示 daytab 是一个数组，数组内包含 13 个元素，其中每一个元素都是一个指向 int 类型的指针。指针数组。

`int a[10][20]`
`int *b[10]`
这里 a 是一个真正的二维数组，分配了 200 个 int 长度的存储空间。
而 b 仅仅分配了 10 个指针。

main 函数打印参数。
```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    // int i;
    
    // for(int i = 1; i < argc; i ++) {
    //     printf("%s%s", argv[i], (i < argc - 1) ? " " : "");
    // }
    // printf("\n");
    // return 0;
    // ---------------------------------------------------------
    // while(--argc > 0) {
    //     printf("%s%s", *++argv, (argc > 1) ? " ": "");
    // }
    // printf("\n");
    // return 0;
    // ---------------------------------------------------------
    while(--argc > 0) {
        printf((argc > 1) ? "%s " : "%s", *++argv);
    }
    printf("\n");
    return 0;
}
```

目前所有遇到的指针的形式
```c
#include <stdio.h>

int main(int argc, char **argv) {
    
    while(argc-- > 0) {
        printf((argc > 0 ? "%s " : "%s\n"), *(argv++));
    }
    
    char *pmsg = "hello";
    char amsg[] = {'h', 'e', 'l', 'l', 'o', '\0'};
    printf("pmsg: %s, amsg: %s\n", pmsg, amsg);        // pmsg: hello, amsg: hello
    printf("pmsg: %c, amsg: %c\n", *pmsg, *amsg);      // pmsg: h, amsg: h
    printf("pmsg: %c, amsg: %c\n", pmsg[0], amsg[0]);  // pmsg: h, amsg: h
    
    char *p[] = {
        "hello",
        "nihao"
    };
    char a[][6] = {
        {'h', 'e', 'l', 'l', 'o', '\0'},
        {'n', 'i', 'h', 'a', 'o', '\0'}
    };
    printf("a: %s, p: %s\n", *(p+1), a[1]);     // a: nihao, p: nihao
    printf("a: %c, p: %c\n", **(p+1), *a[1]);   // a: n, p: n
    printf("a: %c, p: %c\n", *p[1], a[1][0]);   // a: n, p: n
}
```

运行输出结果
```shell
# ./a.out hello nihao
./a.out hello nihao
pmsg: hello, amsg: hello
pmsg: h, amsg: h
pmsg: h, amsg: h
a: nihao, p: nihao
a: n, p: n
a: n, p: n
```

### 指向函数的指针

### 复杂的指针声明

## 结构体
一些简单的使用示例：
```c
#include <stdio.h>
#define XMAX 100
#define YMAX 100

struct point
{
    int x;
    int y;
};

struct rect
{
    struct point pt1;
    struct point pt2;
};

int main()
{
    struct rect screen;
    struct point middle;
    struct point makepoint(int, int);

    screen.pt1 = makepoint(0, 0);
    screen.pt2 = makepoint(XMAX, YMAX);
    middle = makepoint((screen.pt1.x + screen.pt2.x) / 2,
                       (screen.pt1.y + screen.pt2.y) / 2);
                       
    printf("middle x: %d, y: %d\n", middle.x, middle.y);
}

struct point makepoint(int x, int y)
{
    struct point temp;
    temp.x = x;
    temp.y = y;
    return temp;
};

struct point addpoint(struct point p1, struct point p2)
{
    p1.x += p2.x;
    p1.y += p2.y;
    return p1;
}

int ptinrect(struct point p, struct rect r)
{
    return p.x >= r.pt1.x && p.x < r.pt2.x && p.y>= r.pt1.y && p.y < r.pt2.y;
}
```

如果传递给函数的结构很大，使用指针方式比复制整个结构的效率要高。C 语言为值传递。
```
struct point *pp;
```
pp 指向一个 point 结构，`*pp` 即为该结构，`(*pp).x` 和 `(*pp).y` 则是结构成员。`pp->x` 和 `pp->y` 也是结构成员。
```c
#include <stdio.h>

struct point {
    int x;
    int y;
} origin, *pp;

int main() {
    origin.x = 100;
    origin.y = 100;
    
    pp = &origin;
    printf("origin is (%d, %d)\n", (*pp).x, (*pp).y);
    printf("origin is (%d, %d)\n", pp->x, pp->y);
}
```

下面四个表达式是等价的。
```
struct rect r, *rp = &r;

r.pt1.x;
rp->pt1.x;
(r.pt1).x;
(rp->pt1).x;
```

在所有的运算符中结构运算符 `.` 和 `->`、用于函数调用的“()” 以及用于下标的“[]” 的优先级最高。
```
struct {
	int len;
	char *str;
} *p;

++p.len // 将增加 len 的值
*p->str  // 读取的是 str 所指的对象的值
*p->str++ // 先读取 str 所指的对象的值，然后再将 str 加 1，不是 str 的值加 1
(*p->str)++ // 先读取 str 所指的对象的值，然后再将 str 所指向的值加 1
*p++->str // 先读取 str 指向的对象的值，然后再将 p 加 1
```

## 输入输出
下面的程序从标准输入中一个一个读取字符，然后将字符一个一个输出到标准输出。
```c
#include <stdio.h>

int main() {
    int c;

    while((c = getchar()) != EOF) 
        printf("%c", c);

    return 0;
}
```

`./a.out` 直接等待输入，直到遇到换行或者 EOF（Ctrl+D）。
`./a.out < filename` 使用重定向输出文件的内容。注意 `< filename` 并不包含在 argv 的命令行参数中。
`echo "hello world" | ./a.out` 使用管道输出上一个程序的输出。注意 `echo "hello world" |` 并不包含在 argv 的命令行参数中。


## 习题答案
### 1-1
在你自己的系统中运行“hello, world”程序。再有意去掉程序中的部分 内容，看看会得到什么出错信息。
解：
加上 `\n` 字符表示换行。
```c
// p1-1.c
#include <stdio.h>

int main() {
    printf("hello world!\n");
}
```

### 1-2
做个实验，当 printf 函数的参数字符串中包含\c（其中 c 是上面的转义 字符序列中未曾列出的某一个字符）时，观察一下会出现什么情况。
解：
转义字符 `\c` 是未定义的，其行为是未定义的。`\7` 则是一声蜂鸣。
```
// p1-2.c
#include <stdio.h>

int main() {
    printf("\chello world!\n"); // chello world
    printf("\7hello world!\n"); // hello world
}
```

### 1-3
修改温度转换程序，使之能在转换表的顶部打印一个标题。
华氏温度转换为摄氏温度和的转换公式如下： `℃=(5/9)(℉-32)`。源程序：
```c
#include <stdio.h>
#define UPPER 300
#define STEP 20

int main() {
    for(int i = 0; i <= UPPER; i += STEP) {
	    printf("%3d\t%7.2f\n", i, (i - 32) * 5.0 / 9);
    }
}
```
解：
```c
// p1-3.c
#include <stdio.h>
#define UPPER 300
#define STEP 20

int main() {
    printf("Fahr\tCelsius\n");
    for(int i = 0; i <= UPPER; i += STEP) {
        printf("%3d\t%7.2f\n", i, (i - 32) * 5.0 / 9);
    }
}
```

###  1-4
编写一个程序打印摄氏温度转换为相应华氏温度的转换表。
解：
摄氏温度转换为华氏温度的公式：`℉=℃(9/5)+32`。
```c
// p1-4.c
#include <stdio.h>
#define UPPER 300
#define STEP 20

int main() {
    printf("Celsius\t   Fahr\n");
    for(int i = 0; i <= UPPER; i += STEP) {
        printf("%7d\t%7.0f\n", i, i * 9.0 / 5 + 32);
    }
}
```

### 1-5
修改温度转换程序，要求以逆序（即按照从 300 度到 0 度的顺序）打印温度转换表。
解：
```c
// p1-5.c
#include <stdio.h>
#define UPPER 300
#define STEP 20

int main() {
    for(int i = UPPER; i >= 0; i-=STEP) {
        printf("%3d\t%7.2f\n", i, (i - 32) * 5.0 / 9);
    }
}
```


### 1-6
验证表达式 getchar() != EOF 的值是 0 还是 1。
解：
表达式 `c = getchar() != EOF` 与表达式 `c = (getchar() != EOF)` 是等价的。
```c
// p1-6.c
#include <stdio.h>

int main() {
    int c;
    while((c = getchar()) != EOF) {
        purchar(c);
    }
    printf("%d - at EOF\n", c);  // ctrl + D 结束程序时打印 0 - at EOF
}
```

### 1-7
编写一个打印 EOF 值的程序。
解：
`EOF` 在 `stdio.h` 中被定义，一般定义为 `-1`，`#define EOF (-1)`。
```c
// p1-7.c
#include <stdio.h>

int main() {
    printf("%d\n", EOF);
}
```

### 1-8
编写一个统计空格、制表符与换行符个数的程序。
解：
```c
// p1-8.c
#include <stdio.h>

int main() {
    int ws = 0, tab = 0, nl = 0;

    int c;
    while((c = getchar()) != EOF) {
        switch(c) {
            case '\n':
                nl ++;
                break;
            case '\t':
                tab ++;
                break;
            case ' ':
                ws ++;
                break;
            default:
                break;
        }
    }

    printf("ws: %d, tab: %d, nl: %d\n", ws, tab, nl);
}
```

### 1-9
编写一个将输入复制到输出的程序，并将其中连续的多个空格用一个空格代替。
解：
```c
// p1-9.c
#include <stdio.h>

int main() {
    int c;
    int ws = 0;
    
    while((c = getchar()) != EOF) {
        if(c == ' ') {
            if(ws == 0) {
                putchar(c);
                ws = 1;
            }
        } else {
            putchar(c);
            ws = 0;
        }
    }
}
```

### 1-10
编写一个将输入复制到输出的程序，并将其中的制表符替换为\t，把回退符 替换为\b，把反斜杠替按为\\。这样可以将制表符和回退符以可见的方式显示出来。
解：
```c
// p1-10.c
#include <stdio.h>

int main() {
    int c;
    
    while((c = getchar()) != EOF) {
        if(c == '\t') {
            printf("\\t");
        } else if(c == '\b') {
            printf("\\b");
        } else if(c == '\\') {
            printf("\\\\");
        } else {
            putchar(c);
        }
    }
}
```

### 1-11 
你准备如何测试单词计数程序？如果程序中存在某种错误，那么什么样的输 入最可能发现这类错误呢？
首先一个程序统计字符、行和单词的数量，假定单词的定义为不包括空格、制表符或换行符的字符序列。
```c
#include <stdio.h>

#define IN 1
#define OUT 0

int main() {
    int c, nc, nw, nl, state;
    
    nc = nw = nl = 0;
    state = OUT;
    while((c = getchar()) != EOF) {
        nc ++;
        if(c == '\n') {
            nl ++;
        } 
        if(c == '\t' || c == ' ' || c == '\n') {
            state = OUT;
        } else if(state == OUT) {
            state = IN;
            nw ++;
        }
    }
    printf("nc: %d nw: %d nl: %d\n", nc, nw, nl);
}
```
解：
测试用例的选取问题，除一般字符串外，考虑边界值、非法值、极端值的情况。
1. 没有输入。
2. 全部空格。

### 1-12 
编写一个程序，以每行一个单词的形式打印其输入。
解：
```c
// p1-12.c
#include <stdio.h>

#define IN 1
#define OUT 0

int main() {
    int c, state;
    
    state = OUT;
    while((c = getchar()) != EOF) {
        if(c == '\t' || c == ' ' || c == '\n') {
            if(state == IN) {
                state = OUT;
                putchar('\n');
            }
        } else {
            state = IN;
            putchar(c);
        }
    }
}
```

### 1-13
编写一个程序，打印输入中单词长度的直方图。水平方向的直方图比较容易绘制，垂直方向的直方图则要困难些。
解：
```c
// p1-13.c
#include <stdio.h>

#define IN 1
#define OUT 0
#define MAXWORD 20

int wordLen[MAXWORD];

int
main()
{
  int state = OUT;
  int len = 0;
  int overflow = 0;

  // 单词计数逻辑
  int c;
  while ((c = getchar()) != EOF) {
    if (c == '\n' || c == '\t' || c == ' ') {
      if (state == IN) {
        if (len <= MAXWORD) {
          wordLen[len - 1]++;
        } else {
          overflow++;
        }
      }
      len = 0;
      state = OUT;
    } else {
      if (state == OUT) {
        state = IN;
        len++;
      } else {
        len++;
      }
    }
  }

  if (state == IN) {
    if (len <= MAXWORD) {
      wordLen[len - 1]++;
    } else {
      overflow++;
    }
  }

  // 打印水平方向
  for (int i = 0; i < sizeof wordLen / sizeof(int); i++) {
    printf("%2d : ", i + 1);
    for (int j = 0; j < wordLen[i]; j++) {
      putchar('*');
    }
    printf(" %d\n", wordLen[i]);
  }

  if (overflow > 0) {
    printf("There ard %d words > %d\n", overflow, MAXWORD);
  }

  // 打印垂直方向
  int maxvalue = 0;
  for (int i = 0; i < MAXWORD; i++) {
    if (wordLen[i] > maxvalue) {
      maxvalue = wordLen[i];
    }
  }

  for (int i = maxvalue + 1; i > 0; i--) {
    for (int j = 0; j < MAXWORD; j++) {
      if (wordLen[j] == i - 1) {
        printf("%3d", i - 1);
      } else if (wordLen[j] >= i) {
        printf("  *");
      } else {
        printf("  \\");
      }
    }
    putchar('\n');
  }

  for (int i = 0; i < MAXWORD; i++) {
    printf("%3d", i + 1);
  }

  putchar('\n');
  if (overflow > 0) {
    printf("There ard %d words > %d\n", overflow, MAXWORD);
  }
}

```

### 1-14 
编写一个程序，打印输入中各个字符出现频度的直方图。
解：
和 1-13 不同的是，指定了图表的最大长度为 15。
```c
// p1-14.c
#include <stdio.h>
#include <ctype.h>

#define MAXHIST 15
#define MAXCHAR 120

int main() {
    int c;
    int len;
    int maxvalue;
    int cc[MAXCHAR];

    for(int i = 0; i < MAXCHAR; i ++) {
        cc[i] = 0;
    }

    while((c = getchar()) != EOF) {
        if(c < MAXCHAR) {
            ++cc[c];
        }
    }

    maxvalue = 0;
    for(int i = 0; i < MAXCHAR; i ++) {
        if(cc[i] > maxvalue) {
            maxvalue = cc[i];
        }
    }

    for(int i = 1; i < MAXCHAR; i ++) {
        if(isprint(i)) {
            printf("%5d - %c - %5d : ", i, i, cc[i]);
        } else {
            printf("%5d -    - %5d : ", i, i, cc[i]);
        }

        if(cc[i] > 0) {
            if((len = cc[i] * MAXHIST / maxvalue) <= 0){
                len = 1;
            }
        } else {
            len = 0;
        }
        while(len > 0) {
            putchar('*');
            --len;
        }
        putchar('\n');
    }
}
```

### 1-15 
重新编写 1.2 节中的温度转换程序，使用函数实现温度转换计算。
解：
```c
// p1-15.c
#include <stdio.h>

float ceisius(float);

int main() {
    for(float i = 0; i < 300; i += 20) {
        printf("%3.0f %6.1f\n", i, ceisius(i));
    }
}

float ceisius(float fahr) {
    return (5.0 / 9) * (fahr - 32);
}
```

### 1-16 
修改打印最长文本行的程序的主程序 main，使之可以打印任意长度的输入行的长度，并尽可能多地打印文本。
解：
1. 假设最多打印 99 个字符。
```c
// p1-16.c
#include <stdio.h>

#define MAXLINE 100

int get_line(char *, int);

void copy(char *, char *);

int main() {
    
    char longest[MAXLINE];
    char line[MAXLINE];
    
    int max = 0;
    int len;
    
    while((len = get_line(line, MAXLINE)) > 0) {
        if(len > max) {
            copy(line, longest);
            max = len;
        }
    }
    
    printf("longest line len: %d, content: %s", max, longest);
}

void copy(char from[], char to[]) {
    int i = 0;
    while((to[i] = from[i]) != '\0') {
        i ++;
    }
}
```

```c
// get_line.c
#include <stdio.h>

int get_line(char *line, int lim) {
    int c;
    int i = 0, l = 0;
    while((c = getchar()) != EOF && c != '\n') {
        if(i < lim - 2) {
            line[i++] = c;
        }
        
        l ++;
    } 
    
    if(c == '\n') {
        line[i++] = '\n';
        l ++;
    }
    
    line[i] = '\0';
    
    return l;
}
```

### 1-17 
编写一个程序，打印长度大于 80 个字符的所有输入行。
解：
```c

#include <stdio.h>

#define MAXLINE 10
#define PLEN 15

int get_line(char *, int);

int main() {
    int len;
    char line[MAXLINE];
    
    while((len = get_line(line, MAXLINE)) > 0) {
        if(len > PLEN) {
            printf("%s", line);
        }
    }
}

int get_line(char line[], int lim) {
    int c;
    int l = 0, i = 0;
    while((c = getchar()) != EOF && c != '\n') {
        if(l < lim - 2) {
            line[i++] = c;
        }
        l ++;
    }
    
    if(c == '\n') {
        line[i ++] = c;
        l ++;
    }
    
    line[i] = '\0';
    return l;
}
```

### 1-18 
编写一个程序，删除每个输入行末尾的空格及制表符，并删除完全是空格的行。
解：
```c
// p1-18.c
#include <stdio.h>
#include <string.h>

#define MAXLINE 100

int get_line(char *, int);
void remove_end_blank(char *, int);

int main() {
    char line[MAXLINE];
    
    int len;
    while((len = get_line(line, MAXLINE)) > 0) {
        printf("str len: %d, str: %s", strlen(line), line);
        remove_end_blank(line, MAXLINE);
        if(line[0] != '\n') {
            printf("str len: %d, str: %s", strlen(line), line);
        }
    }   
}

void remove_end_blank(char line[], int max) {
    int i;
    for(i = 0; i < max && line[i] != '\0'; i ++);
    
    for(i = i - 1; i >= 0 && (line[i] == '\t' || line[i] == ' ' || line[i] == '\n'); i--);
    
    line[i + 1] = '\n';
    line[i + 2] = '\0';
}

int get_line(char line[], int max) {
    int c;
    int i = 0, l = 0;
    
    
    while((c = getchar()) != EOF && c != '\n') {
        if(i < max - 2) {
            line[i++] = c;
        }
        
        l++;
    }
    
    if(c == '\n') {
        line[i++] = '\n';
        l++;
    }
    
    line[i] = '\0';
    return l;
}
```

### 1-19 
编写函数 reverse(s)，将字符串 s 中的字符顺序颠倒过来。使用该函数 编写一个程序，每次颠倒一个输入行中的字符顺序。
解：
```c
// p1-19.c
#include <stdio.h>
#include <string.h>
#define MAXLINE 100

int get_line(char *, int);

void revese(char *, int);

int main() {
    char line[MAXLINE];
    while(get_line(line, MAXLINE) > 0) {
        printf("str len: %d, str: %s\n", strlen(line), line);
        revese(line, MAXLINE);
        if(line[0] != '\n') {
            printf("str len: %d, str: %s", strlen(line), line);
        }
    }
}

void revese(char line[], int max) {
    int i;
    for(i = 0; i < max && (line[i] != '\n') && line[i] != '\0'; i ++) {
        printf("%c - %d\n", line[i], i);
    }
    
    int right = i - 1, left = 0;
    while(left < right) {
        int tmp = line[left];
        line[left] = line[right];
        line[right] = tmp;
        
        left ++;ccc 
        right --;
    }
}

int get_line(char line[], int max) {
    int c;
    int i = 0, l = 0;
    
    while((c = getchar()) != EOF && c != '\n') {
        if(i < max - 2) {
            line[i++] = c;
        }
        l++;
    }
    
    if(c == '\n') {
        line[i] = '\n';
        l ++;
    }
    
    line[i] = '\0';
    return l;
}
```
### 1-23 
编写一个删除 C 语言程序中所有的注释语句。要正确处理带引号的字符串与 字符常量。在 C 语言中，注释不允许嵌套。  
解：  
要处理几种情况：   

- `// xxx` 直到行结束，或者文件结束。  
- `\* *\`  多行注释。  
- `'xxx // xxx \' xx  '` 引号内的内容原样输出。  
- `"xxx // xxx \" xx "`  引号内的内容原样输出。  

```c
// p1-23.c
#include <stdio.h>

void echo_quote(int);

int main() {
    printf("hello \" world!\n");
    printf("// helloworld /* nihaos */ \n");
    int c;
    // hello world
    /*
        hello world // 
        ' '
        "hello world"
    */
    int d;
    while((c = getchar()) != EOF) {
        if(c == '/') {
            if((d = getchar()) == '*') {
                c = getchar();
                d = getchar();
                while(c != '*' || d != '/') {
                    c = d;
                    d = getchar();
                }
            } else if(d == '/') {
                while((c = getchar()) != EOF && c != '\n')
                    ;
                if(c == '\n') {
                    putchar(c);
                }
            } else {
                putchar(c);
                putchar(d);
            }
        }  else if(c == '\'' || c == '"') {
            echo_quote(c);
        } else {
            putchar(c);
        }
    }
}

void echo_quote(int c) {
    int d;
    putchar(c);
    
    while((d = getchar()) != c) {
        putchar(d);
        if(d == '\\') {
            putchar(getchar());
        }
    }
    
    putchar(d);
}
```

