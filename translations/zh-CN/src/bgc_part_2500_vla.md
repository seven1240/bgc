<!-- C语言指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 变长数组（VLAs）

C语言提供了一种方法让你声明一个数组，其大小在运行时确定。这为你提供了类似使用`malloc()`时动态确定大小的好处，但无需担心在之后需要调用`free()`释放内存。

现在，很多人不喜欢使用VLAs。比如，它们已经被禁止在Linux内核中使用。我们将 [稍后](#vla-general-issues) 深入探讨更多相关理由。

[`__STDC_NO_VLA__` 宏]<

这是语言的一个可选功能。宏`__STDC_NO_VLA__`设置为`1`表示VLAs _不存在_。（它们在C99中是强制的，然后从C11起变成了可选的。）

``` {.c}
#if __STDC_NO_VLA__ == 1
   #error 对不起，这个程序需要VLAs支持！
#endif
```

[`__STDC_NO_VLA__` 宏]>

但是由于GCC和Clang都没有定义这个宏，因此你可能无法从中受益太多。

让我们先通过一个示例深入了解，然后再来细看具体细节。

## 基础知识

一个常规数组是用常量大小声明的，就像这样：

``` {.c}
int v[10];
```

[变长数组-->定义]<

但使用VLAs，我们可以使用运行时确定的大小来设置数组，就像这样：

``` {.c}
int n = 10;
int v[n];
```

现在，这看起来和之前一样，而在很多方面也是一样的，但这样做给了你计算所需大小的灵活性，然后得到确切大小的数组。

让我们让用户输入数组的大小，然后将索引乘以10存储在每个数组元素中：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int n;
    char buf[32];

    printf("输入一个数字："); fflush(stdout);
    fgets(buf, sizeof buf, stdin);
    n = strtoul(buf, NULL, 10);

```c
int v[n];

for (int i = 0; i < n; i++)
    v[i] = i * 10;

for (int i = 0; i < n; i++)
    printf("v[%d] = %d\n", i, v[i]);
}
```

（在第7行，我有一个 `fflush()`，它应该强制输出这一行，即使我在末尾没有换行符。）

第10行是我们声明可变长度数组（VLA）的地方---一旦执行到该行，数组的大小将设置为那一刻的 `n` 值。数组长度后续无法更改。

你也可以在方括号中放置一个表达式，比如：

``` {.c}
int v[x * 100];
```

一些限制：

* 你不能在文件作用域中声明一个VLA，也不能在块作用域内使用`static`[^（这是因为 VLAs 通常在栈上分配，而 `static` 变量在堆上。而 VLAs 的整个理念在于它们将在函数末尾弹出时自动释放。）]。
* 你不能使用初始化列表来初始化数组。

此外，在数组大小中输入一个负值会引发未定义行为---在这个宇宙里至少是这样。

[i[Variable-length array-->defining]>]

## `sizeof`和VLAs

[i[`sizeof()`和可变长度数组-->和`sizeof()`]<]

我们习惯于`sizeof`给出任何特定对象（包括数组）的大小（以字节为单位）。VLAs也不例外。

主要区别在于，对于一个VLA，`sizeof`在_运行时_执行，而对于一个非可变大小的变量，它在_编译时_计算。

但用法是一样的。

你甚至可以用通常的数组技巧计算VLA中的元素个数：

``` {.c}
size_t num_elems = sizeof v / sizeof v[0];
```

上面一行有一个微妙但是正确的暗示：指针算术对于普通数组的工作方式就像你预期的那样。所以放心地尽情使用它：
```

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

类似于普通数组，您可以使用 `sizeof()` 和括号来获取将要声明的 VLA 的大小，而不实际声明一个：

``` {.c}
int x = 12;

printf("%zu\n", sizeof(int [x]));  // 在我的系统上打印出 48
```

## 多维 VLA

您可以创建各种一维或多维 VLA，其中一个或多个维度设置为一个变量

``` {.c}
int w = 10;
int h = 20;

int x[h][w];
int y[5][w];
int z[10][w][20];
```

同样，您可以像处理普通数组一样处理这些。

## 将一维 VLA 传递给函数

将一维 VLA 传递到函数中与传递普通数组没有太大不同。您可以直接这样做。

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
    int x[5];   // 普通数组

    int a = 5;
    int y[a];   // VLA

    for (int i = 0; i < a; i++)
        x[i] = y[i] = i + 1;

    printf("%d\n", sum(5, x));
    printf("%d\n", sum(a, y));
}
```

但除此之外还有更多。您还可以让 C 知道该数组是特定大小的 VLA，方法是首先将其传递给函数，然后在参数列表中给出该维度：

``` {.c}
int sum(int count, int v[count])
{
    // ...
}
```

顺便提一下，列出上面函数的原型有几种方法；其中一种涉及在 VLA 中不想明确命名值时使用 `*`。这只是表明该类型是 VLA，而不是常规指针。

VLA 的原型：

``` {.c}
void do_something(int count, int v[count]);  // 包含名称
void do_something(int, int v[*]);            // 不包含名称
```

再次强调，上面这种 `*` 只适用于原型---在函数本身中，你必须提供显式大小。

现在---_让我们多维度起来_！这里开始变得有趣了。

## 将多维 VLA 传递给函数

与上面处理二维 VLA 的第二种形式类似，但这次我们传递的是两个维度，并使用它们。

在以下示例中，我们构建了一个可变宽度和高度的乘法表矩阵，然后将其传递给一个函数打印出来。

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

### 部分多维 VLA

你可以固定一些维度，而另一些维度变量。假设我们的记录长度固定为 5 个元素，但我们不知道有多少条记录。

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
    // goat是一个包含10个整数的数组

    // 用数字的平方来初始化
    for (int i = 0; i < w; i++)
        x[i] = i*i;

    // 打印它们
    for (int i = 0; i < w; i++)
        printf("%d\n", x[i]);

    // 现在让我们改变w...

    w = 20;

    // 但是goat仍然是一个包含10个整数的数组，因为这是typedef执行时的w的值。
}
```

所以它表现得像一个大小固定的数组。

但是你仍然不能对它使用初始化列表。

## 跳坑

在使用VLA附近使用`goto`时要小心，因为很多事情是不合法的。

当你使用`longjmp()`时，有一种情况会导致VLA泄漏内存。

但是这两件事情我们会在各自的章节中介绍。

## 一般问题

VLAs 在 Linux 内核中被禁止有几个原因：

* 很多地方应该使用固定大小数组。
* VLA 的背后代码速度较慢（大多数人可能察觉不到，但在操作系统中会有影响）。
* 不是所有C编译器都同等支持 VLA。
* 栈大小有限，而VLA存放在栈中。如果某些代码意外（或恶意）传递一个大值给分配一个VLA的内核函数，会发生糟糕的事情。
```

其他网友指出，无法检测变长数组(VLA)分配失败的情况，遇到这种问题的程序很可能会崩溃。虽然固定大小的数组也存在同样的问题，但更有可能的是有人不小心创建了一个容量异常大的_VLA（变长数组），而不是无意中声明一个固定大小的，比如说，30兆字节的数组。