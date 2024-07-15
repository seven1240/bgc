# 变长数组（VLAs）

C提供了一种方法，允许您声明一个数组，其大小在运行时确定。这让您享受到动态运行时大小调整的好处，就像使用`malloc()`一样，但无需担心在之后释放内存。

现在，很多人不喜欢VLAs。例如，它们已经被Linux内核禁止使用。我们将在稍后深入探讨更多相关的原理[（点此查看）](#vla-general-issues)。

这是语言的一个可选功能。宏`__STDC_NO_VLA__`设置为`1`，如果VLAs不可用的话（在C99中是强制存在的，然后在C11中变成了可选的）。

``` {.c}
#if __STDC_NO_VLA__ == 1
   #error 对不起，此程序需要VLAs支持！
#endif
```

但由于GCC和Clang都没有定义这个宏，所以这个方法可能有限。

让我们首先通过一个示例来了解基础知识，然后再深入探讨细节。

## 基础知识

一个普通的数组声明大小是固定的，像这样：

``` {.c}
int v[10];
```

但使用VLAs，我们可以在运行时使用确定大小来设置数组，就像这样：

``` {.c}
int n = 10;
int v[n];
```

现在，看起来和之前一样，很多方面也确实如此，但这让你有了计算所需大小的灵活性，然后得到恰好那个大小的数组。

让我们要求用户输入数组大小，然后在每个数组元素中存储该索引乘以10的值：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int n;
    char buf[32];

    printf("输入一个数字："); fflush(stdout);
    fgets(buf, sizeof buf, stdin);
    n = strtoul(buf, NULL, 10);
```

```c
int v[n];

for (int i = 0; i < n; i++)
    v[i] = i * 10;

for (int i = 0; i < n; i++)
    printf("v[%d] = %d\n", i, v[i]);
}
```

（在第7行，我使用了`fflush()`来强制输出这一行，即使我最后没有换行符。）

在第10行我们声明了VLA——一旦执行超过那一行，数组的大小将设置为那个时刻的`n`。数组长度以后不能再更改。

你可以在括号里放一个表达式，比如：

``` {.c}
int v[x * 100];
```

一些限制：

* 你不能在文件作用域声明VLA，在块作用域中也不能定义`static`静态的变量^[这是因为VLA通常是在堆栈上分配的，而`static`变量是在堆上。而VLA的整个概念是它们将在函数结束时自动释放。]。
* 你不能使用初始化列表来初始化数组。

另外，在数组大小中输入负值会引起未定义的行为——至少在这个宇宙中是这样的。

[i[Variable-length array-->defining]>]

## `sizeof` 和 VLAs

[i[Variable-length array-->and `sizeof()`]<]

我们习惯于`sizeof`给出特定对象的大小（以字节为单位），包括数组。而对于VLA也不例外。

主要区别在于，在VLA上执行的`sizeof`是在_运行时_计算的，而对于非可变大小的变量，它在 _编译时_计算。

但用法是一样的。

你甚至可以用通常的数组技巧来计算VLA中的元素个数：

``` {.c}
size_t num_elems = sizeof v / sizeof v[0];
```

从上面一行可以暗示一个微妙而正确的结论：指针算术对于常规数组来说也像你预期的那样工作。所以放心使用，尽情享用吧：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int n = 5;
    int v[n];

    int *p = v;

    *(p+2) = 12;
    printf("%d\n", v[2]);  // 12

    p[3] = 34;
    printf("%d\n", v[3]);  // 34
}
```

跟普通数组一样，你可以用`sizeof()`和括号来获取一个可能的VLA的大小，而无需实际声明一个：

``` {.c}
int x = 12;

printf("%zu\n", sizeof(int [x]));  // 在我的系统上打印出48
```

[i[可变长度数组-->和`sizeof()`]>]

## 多维VLA

[i[可变长度数组-->多维]<]

你可以继续创建各种各样的VLA，其中一个或多个维度设置为一个变量

``` {.c}
int w = 10;
int h = 20;

int x[h][w];
int y[5][w];
int z[10][w][20];
```

同样，你可以像操作普通数组那样操作这些VLA。

[i[可变长度数组-->多维]>]

## 将一维VLA传递给函数

[i[可变长度数组-->传递给函数]<]

将单维VLA传递到函数中与传递普通数组没有什么不同。你直接就行。

``` {.c .numberLines}
#include <stdio.h>

int sum(int count, int *v)
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += v[i];

    return total;
}

int main(void)
{
    int x[5];   // 标准数组

    int a = 5;
    int y[a];   // VLA

    for (int i = 0; i < a; i++)
        x[i] = y[i] = i + 1;

    printf("%d\n", sum(5, x));
    printf("%d\n", sum(a, y));
}
```

但比这更有趣的是你还可以告诉C数组具体的VLA大小，方法是首先传入该大小，然后在参数列表中给出该维度：

``` {.c}
int sum(int count, int v[count])
{
    // ...
}
```

[i[可变长度数组-->在函数原型中]<]
[i[用*表示VLA函数原型]<]

顺便说一下，列出上述函数的原型有几种方法；其中一种涉及`*`，如果你不想特别命名VLA中的值的话。它只是表示该类型是VLA，而不是常规指针。

VLA原型:

``` {.c}
void do_something(int count, int v[count]);  // 带有名称
void do_something(int, int v[*]);            // 没有名称
```

再次强调，`*`只适用于原型---在函数内部，你必须放置显式大小。

[i[`*`适用于VLA函数原型]>]
[i[可变长度数组-->在函数原型中]>]

现在---_让我们变得多维_！这才是乐趣所在。

## 将多维VLA传递给函数

与我们上面做的一维VLA第二种形式一样，但这次我们传入两个维度并使用它们。

在以下示例中，我们构建一个宽度和高度可变的乘法表矩阵，然后将其传递给一个打印函数打印出来。

``` {.c .numberLines}
#include <stdio.h>

void print_matrix(int h, int w, int m[h][w])
{
    for (int row = 0; row < h; row++) {
        for (int col = 0; col < w; col++)
            printf("%2d ", m[row][col]);
        printf("\n");
    }
}

int main(void)
{
    int rows = 4;
    int cols = 7;

    int matrix[rows][cols];

    for (int row = 0; row < rows; row++)
        for (int col = 0; col < cols; col++)
            matrix[row][col] = row * col;

    print_matrix(rows, cols, matrix);
}
```

### 部分多维VLA

您可以将一些维度固定，一些维度变量。假设我们的记录长度固定为5个元素，但不知道有多少记录。

``` {.c .numberLines}
#include <stdio.h>

```c
void print_records(int count, int record[count][5])
{
    for (int i = 0; i < count; i++) {
        for (int j = 0; j < 5; j++)
            printf("%2d ", record[i][j]);
        printf("\n");
    }
}

int main(void)
{
    int rec_count = 3;
    int records[rec_count][5];

    // 用一些虚拟数据填充数组
    for (int i = 0; i < rec_count; i++)
        for (int j = 0; j < 5; j++)
            records[i][j] = (i+1)*(j+2);

    print_records(rec_count, records);
}
```

```c
    // goat 是一个包含 10 个整数的数组
    int goat[10];

    // 用数字的平方初始化
    for (int i = 0; i < w; i++)
        goat[i] = i * i;

    // 打印它们
    for (int i = 0; i < w; i++)
        printf("%d\n", goat[i]);

    // 现在让我们改变 w...

    w = 20;

    // 但是 goat 仍然是一个包含 10 个整数的数组，因为这是
    // 在执行 typedef 时 w 的值。

}
```

因此，它的作用就像一个固定大小的数组。

但你仍然不能在它上面使用初始化列表。

## 跳坑

在使用变长数组（VLA）附近使用 `goto` 时，你必须注意很多事情是不被允许的。

当使用 `longjmp()` 时，存在一种情况下，你可能会因为 VLA 而泄漏内存。

但这两件事情我们会在各自的章节中进行讨论。

## 普遍问题 {#vla-general-issues}

一些原因导致变长数组（VLA）被 Linux 内核禁止：

* 很多地方本应该使用固定大小的数组。
* VLA 的实现比较慢（大多数人可能察觉不到，但在操作系统中会有影响）。
* 不是所有的 C 编译器都对 VLA 的支持程度一样。
* 栈的大小是有限的，并且 VLA 放在栈上。如果某段代码不小心（或恶意地）向内核函数中传递一个大值来分配一个 VLA，可能会发生糟糕的情况。
```

```c
// 其他在线用户指出，无法检测动态分配数组（VLA）无法分配的失败，程序遇到这类问题很可能会崩溃。尽管固定大小数组也存在相同的问题，但更有可能的是有人会不小心创建一个“异常大小的VLA”，而不是不知何故地声明一个固定大小的，比如说30兆字节的数组。
```