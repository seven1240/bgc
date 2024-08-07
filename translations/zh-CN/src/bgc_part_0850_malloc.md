<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 手动内存分配

手动内存管理

这是 C 与您已经了解的其他语言可能有所不同的一个重要领域：_手动内存管理_。

其他语言使用引用计数、垃圾回收或其他方法来确定何时为某些数据分配新内存，并在没有变量引用它时何时释放它。

这很好。能够不用担心这个问题，只需放弃对某个项目的所有引用，并相信在某个时刻与其相关联的内存将被释放。

但 C 不完全是这样的。

自动变量

当然，在 C 中，一些变量在进入作用域和离开作用域时会自动分配和释放。我们称这些为自动变量。它们是您平均普通的块作用域“局部”变量。没有问题。

但是如果您想要让某些东西比特定块存在时间更长怎么办？这就是手动内存管理发挥作用的地方。

您可以明确告诉 C 为您分配一定数量的字节，以便您随心所欲地使用。这些字节将保持分配状态，直到您明确释放该内存^[或者直到程序退出，此时由它分配的所有内存均被释放。星号：一些系统允许您分配在程序退出后仍然存在的内存，但这取决于系统，超出了本指南的范围，您肯定不会不小心做到这一点。]。

重要的是释放处理完的内存！如果不释放，我们称之为 _内存泄漏_，您的进程将继续保留该内存直到退出。

_如果您手动分配它，那么在使用完毕后必须手动释放它。_

这么做我们需要怎么样呢？我们将学习一些新函数，并利用 `sizeof` 运算符来帮助我们学习需要分配多少字节。

在通用的C术语中，开发人员称自动本地变量为在“栈上”分配的，手动分配的内存为“堆上”的。规范并未提及这些内容，但所有的C开发人员如果你提及这些概念，他们都会明白你在说什么。

本章中我们将要学习的所有函数都可以在 `<stdlib.h>` 中找到。

## 分配和释放内存，`malloc()` 和 `free()`

[`malloc()` 函数<] 
`malloc()` 函数接受要分配的字节数，返回一个指向新分配内存块的 `void` 指针。

因为它是一个 `void*`，你可以将它分配给任何你想要的指针类型... 通常，这会与你分配的字节数有某种关联。

[`sizeof` 运算符-->配合 `malloc()`<]
那么... 我应该分配多少字节呢？我们可以使用 `sizeof` 来帮助确定。如果我们想要为一个 `int` 分配足够的空间，可以使用 `sizeof(int)` 并将其传递给 `malloc()`。
[`sizeof` 运算符-->配合 `malloc()`>]

[`free()` 函数<]
当我们不再需要某块已分配的内存时，我们可以调用 `free()` 表示我们不再需要该内存，可以被用作其他用途。作为参数，你传递给 `malloc()` 获取的相同指针（或其副本）。在你释放内存后还使用该内存区域是未定义行为。

让我们试一试。我们将为一个 `int` 分配足够的内存，然后将数据存储在那里，然后打印出来。

``` {.c}
// 为一个 int 分配空间（大小为 sizeof(int) 字节）：

int *p = malloc(sizeof(int));

*p = 12;  // 将数据存储在那里
```

```c
printf("%d\n", *p);  // 打印出来：12

free(p);  // 释放这块内存

//*p = 3490;  // 错误：未定义行为！在调用free()后继续使用！
```

在这个编造的例子中，实际上没有任何好处。我们本可以使用一个自动变量`int`，效果也一样。但我们将看到使用这种方式分配内存的能力有其优点，尤其是在处理更复杂的数据结构时。

[i[`sizeof` 运算符-->结合 `malloc()`]<]
你经常会看到的另一种用法是利用`sizeof`可以获取任意常量表达式的结果类型的大小这一事实。因此，你也可以在里面放一个变量名，并使用它。下面是一个例子，与之前的类似：

```c
int *p = malloc(sizeof *p);  // *p 是一个 int，所以等同于 sizeof(int)
```
[i[`sizeof` 运算符-->结合 `malloc()`]>]
[i[`malloc()` 函数]>]

## 错误检查

[i[`malloc()` 函数-->错误检查]<]
所有的分配函数都会返回指向新分配的内存块的指针，或者如果由于某种原因无法分配内存则返回 `NULL`。

一些操作系统如Linux可以配置成即使内存用尽，`malloc()`也不会返回 `NULL`。但尽管如此，你始终应该以防护措施编写代码。

```c
int *x;

x = malloc(sizeof(int) * 10);

if (x == NULL) {
    printf("分配 10 个 int 时出错\n");
    // 在这里进行错误处理
}
```

下面是一个常见模式，我们在同一行上完成赋值和条件判断：

```c
int *x;

if ((x = malloc(sizeof(int) * 10)) == NULL)
    printf("分配 10 个 int 时出错\n");
    // 在这里进行错误处理
}
```
[i[`malloc()` 函数-->错误检查]>]

## 为数组分配空间
```

我们已经学会为单个项目分配空间；现在该考虑为数组中的一堆项目分配空间了。

在C语言中，数组是相同的东西在连续的内存区域中依次排列。

我们可以分配一个连续的内存区域---我们已经知道如何做到这一点。如果我们想要3490个字节的内存，我们可以直接请求：

``` {.c}
char *p = malloc(3490);  // 大功告成
```

而且---确实！---这是一个有3490个`char`的数组（也叫字符串！）因为每个`char`占用1个字节。换句话说，`sizeof(char)`是`1`。

注意: 新分配的内存没有进行初始化---里面充满了垃圾值。如果需要的话，可以用`memset()`来清除，或者查看下面的`calloc()`。

但我们可以将所需项目的大小乘以所需元素的数量，然后使用指针或数组表示法访问它们。举个例子！

[`sizeof`运算符-->与`malloc()`搭配使用]

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    // 为10个int分配空间
    int *p = malloc(sizeof(int) * 10);

    // 为它们赋值0-45：
    for (int i = 0; i < 10; i++)
        p[i] = i * 5;

    // 打印所有值 0, 5, 10, 15, ..., 40, 45
    for (int i = 0; i < 10; i++)
        printf("%d\n", p[i]);

    // 释放空间
    free(p);
}
```

关键在于`malloc()`这一行。如果我们知道每个`int`需要`sizeof(int)`个字节来存储，且我们需要10个`int`，我们可以直接分配这么多字节：

``` {.c}
sizeof(int) * 10
```

这个技巧适用于每种类型。只需将其传递给`sizeof`，并乘以数组的大小。

这是另一个类似于 `malloc()` 的分配函数，有两个关键差异：

* 不同于单个参数，你需要传递一个元素的大小，以及你想要分配的元素的数量。就像它是为了分配数组而设计的。
* 它将内存清零。

你仍然需要使用 `free()` 来释放通过 `calloc()` 获得的内存。

下面是 `calloc()` 和 `malloc()` 的比较。

``` {.c}
// 使用 calloc() 为 10 个 int 分配空间，初始值为 0：
int *p = calloc(10, sizeof(int));

// 使用 malloc() 为 10 个 int 分配空间，初始值为 0：
int *q = malloc(10 * sizeof(int));
memset(q, 0, 10 * sizeof(int));   // 设置为 0
```

同样，两者的结果相同，除了 `malloc()` 默认不清零内存。
[i[`calloc()` function]>]

## 通过 `realloc()` 改变已分配的大小

[i[`realloc()` function]<]
如果你已经分配了 10 个 `int`，但后来决定需要 20 个，你可以怎么做呢？

一种选项是分配一些新空间，然后用 `memcpy()` 将内存复制过去... 但有时候事实证明你根本不需要移动任何东西。有一个函数足够聪明，能在所有情况下做正确的事情：`realloc()`。

它接受一个指向先前通过 `malloc()` 或 `calloc()` 分配的内存的指针，以及内存区域的新大小。

然后，它会扩大或缩小该内存，并返回一个指向它的指针。
有时它可能会返回相同的指针（如果数据不需要在别处复制），或者可能会返回一个不同的指针（如果数据需要被复制）。

确保当调用 `realloc()` 时，你指定要分配的字节数，而不仅仅是数组元素的数量！也就是说：

``` {.c}
num_floats *= 2;
```

np = realloc(p, num_floats);  // 错误：需要字节数，而不是元素个数！

np = realloc(p, num_floats * sizeof(float));  // 更好！

让我们分配一个包含 20 个 float 的数组，然后改变主意，将其更改为包含 40 个数组。

我们将把 `realloc()` 的返回值赋值给另一个指针，以确保它不是 `NULL`。如果不是 `NULL`，然后我们可以重新分配给我们原来的指针。（如果我们将返回值直接分配给原来的指针，如果函数返回 `NULL`，我们将失去该指针，也没有办法获取它。）

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    // 为 20 个 float 分配空间
    float *p = malloc(sizeof *p * 20);  // sizeof *p 等同于 sizeof(float)

    // 为它们赋予 0.0-1.0 的分数值：
    for (int i = 0; i < 20; i++)
        p[i] = i / 20.0;

    // 不过！我们实际上要将这个数组扩展为包含 40 个元素
    float *new_p = realloc(p, sizeof *p * 40);

    // 检查是否成功重新分配
    if (new_p == NULL) {
        printf("realloc 错误\n");
        return 1;
    }

    // 如果成功了，我们可以简单地重新分配给 p
    p = new_p;

    // 并为新元素赋值范围在 1.0-2.0
    for (int i = 20; i < 40; i++)
        p[i] = 1.0 + (i - 20) / 20.0;

    // 打印所有值介于 0.0-2.0 的 40 个元素：
    for (int i = 0; i < 40; i++)
        printf("%f\n", p[i]);

    // 释放空间
    free(p);
}
```

请注意我们如何将 `realloc()` 的返回值取出，并重新分配给我们传入的同一个指针变量 `p`。这是很常见的做法。

如果第7行看起来有些奇怪，有一个 `sizeof *p` 在那里，记住 `sizeof` 是根据表达式的类型计算大小的。 `*p` 的类型是 `float`，所以这一行相当于 `sizeof(float)`。

### 读取任意长度的行

我想通过这个完整的示例演示两件事。

1. 使用 `realloc()` 来随着读入更多数据来扩展缓冲区。
2. 使用 `realloc()` 在完成读取后将缓冲区缩小到合适的大小。

我们在这里看到一个循环，调用 `fgetc()` 来不断附加到缓冲区，直到我们看到最后一个字符是换行符为止。

一旦找到换行符，它会将缓冲区缩小到合适的大小然后返回它。

```plaintext
//当像这样增长内存时，通常（尽管绝不是法律规定）每一步都会将需要的空间加倍，以最小化出现的realloc() 次数。

最后您可能会注意到readline()返回一个指向malloc()缓冲区的指针。因此，调用者需要在完成后显式释放该内存。

### `realloc()`带有`NULL`

这两行等效：

``` {.c}
char *p = malloc(3490);
char *p = realloc(NULL, 3490);
```

如果您有某种分配循环且不希望为第一个`malloc()`设置特殊情况，这可能会很方便。
```

``` {.c}
int *p = NULL;
int length = 0;

while (!done) {
    // 分配 10 个额外的整数：
    length += 10;
    p = realloc(p, sizeof *p * length);

    // 做一些了不起的事情
    // ...
}
```

在这个例子中，由于 `p` 初始值为 `NULL`，我们不需要初始的 `malloc()`。

## 对齐分配

[i[内存对齐]<]
你可能不需要使用这个。

我现在不想太偏离主题，但有一个叫做“内存对齐”的东西，它与内存地址（指针值）是某个特定数字的倍数有关。

例如，系统可能要求 16 位值从内存地址的倍数开始。或者要求 64 位值从内存地址的倍数开始，比如 2，4 或 8 的倍数。这取决于 CPU。

有些系统需要这种对齐方式以便快速访问内存，甚至有些系统必须这样才能访问内存。

现在，如果你使用 `malloc()`、`calloc()` 或 `realloc()`，C 将为你提供一个适合任何值的内存块，甚至是 `struct`。在所有情况下都适用。

但有时候你可能会知道某些数据可以在更小的边界上对齐，或者出于某种原因必须在更大的边界上对齐。我想在嵌入式系统编程中这更常见。

[i[`aligned_alloc()` 函数]<]
在这种情况下，你可以使用 `aligned_alloc()` 指定对齐方式。

对齐方式是大于零的整数的二次幂，比如 `2`、`4`、`8`、`16` 等等，你需要在所需字节数之前将这个值传递给 `aligned_alloc()`。

另一个限制是您分配的字节数需要是对齐的倍数。但这可能会发生改变。参见[fl[C缺陷报告460](http://www.open-std.org/jtc1/sc22/wg14/www/docs/summary.htm#dr_460)]

让我们做一个示例，在64字节边界上分配：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // 在64字节边界上分配256字节
    char *p = aligned_alloc(64, 256);  // 256 == 64 * 4

    // 复制一个字符串并打印
    strcpy(p, "你好，世界！");
    printf("%s\n", p);

    // 释放空间
    free(p);
}
```

我想在这里提一下关于 `realloc()` 和 `aligned_alloc()`。`realloc()` 没有任何对齐保证，所以如果您需要获得一些对齐的重新分配空间，您将不得不用 `memcpy()` 的方式去做。
[i[`aligned_alloc()` 函数]>]

这里是一个非标准的 `aligned_realloc()` 函数，如果你需要的话：

``` {.c}
void *aligned_realloc(void *ptr, size_t old_size, size_t alignment, size_t size)
{
    char *new_ptr = aligned_alloc(alignment, size);

    if (new_ptr == NULL)
        return NULL;

    size_t copy_size = old_size < size? old_size: size;  // 获取最小值

    if (ptr != NULL)
        memcpy(new_ptr, ptr, copy_size);

    free(ptr);

    return new_ptr;
}
```

请注意，它总是复制数据，会花费时间，而真正的 `realloc()` 在可以的情况下会避免复制。因此这并不高效。尽量避免需要重新分配自定义对齐的数据。
[i[内存对齐]>]
[i[手动内存管理]>]