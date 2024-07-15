```c
# 手动内存分配

这是 C 与你已经了解的语言可能大不相同的一个重要领域：_手动内存管理_。

其他语言使用引用计数、垃圾回收或其他方法来确定何时为某些数据分配新内存，以及当没有变量引用它时何时释放它。

这很方便。不用操心，只需放弃对一个项目的所有引用，相信在某个时刻与其关联的内存将被释放。

但 C 并非完全如此。

当然，在 C 中，一些变量在进入作用域和离开作用域时会自动分配和释放。我们称这些为自动变量。它们是您平均普通的块范围“局部”变量。没问题。

但如果你希望某些东西的存在时间超过特定块怎么办？这就是手动内存管理发挥作用的地方。

你可以明确告诉 C 为你分配一定数量的字节，你可以随意使用。这些字节将保持分配，直到你显式释放该内存^[或者直到程序退出，在这种情况下，由它分配的所有内存将被释放。星号：有些系统允许您分配程序退出后仍然存在的内存，但这是系统相关的，超出了本指南的范围，你肯定不会无意中这样做。]。

重要的是释放你不再需要的内存！如果不释放，我们将其称为_内存泄漏_，你的进程将继续保留该内存直到退出。

_如果你手动分配了它，那么在完成后就必须手动释放它。_
```

所以我们要怎么做呢？我们将学习几个新的函数，并且利用`sizeof`运算符来帮助我们确定要分配多少字节。

在普通的C术语中，开发者说自动本地变量被分配在"堆栈上”，而手动分配的内存在"堆上"。规范文件没有涉及这两者中的任何一个，但所有的C开发人员都会明白你在谈论什么。

所有我们将在本章学习的函数都可以在`<stdlib.h>`中找到。

## 分配内存和释放内存，`malloc()`和`free()`

[`malloc()`函数]
`malloc()`函数接受要分配的字节数，然后返回指向该新分配内存块的`void`指针。

由于它是一个`void*`，你可以将其分配给任何你想要的指针类型...通常这个类型将以某种方式与你要分配的字节数对应。

[`sizeof`运算符-->与`malloc()`一起使用]
那么...我应该分配多少字节呢？我们可以使用`sizeof`来帮助确定。如果我们想要为一个`int`分配足够的空间，我们可以使用`sizeof(int)`，并将其传递给`malloc()`。

[`free()`函数]
当我们使用完内存后，我们可以调用`free()`以指示我们不再使用该内存，可以用于其他用途。作为参数，你传递给`malloc()`的相同指针（或其副本）。在你调用`free()`后使用内存区域是未定义行为。

让我们试试。我们将为一个`int`分配足够的内存，然后将某些内容存储在其中，然后打印出来。

``` {.c}
// 为单个整数分配空间（sizeof(int)字节大小）：

int *p = malloc(sizeof(int));

*p = 12;  // 在这里存储一些内容
```

```c
printf("%d\n", *p);  // 打印它：12

free(p);  // 完成内存释放

//*p = 3490;  // 错误：未定义行为！free()之后进行使用！

```
[i[`free()`函数]>]

在这个刻意制造的例子中，实际上并没有什么好处。我们可以使用自动`int`，也可以完成任务。但是，我们会看到通过这种方式分配内存的能力具有其优点，尤其是对于更复杂的数据结构。

[i[`sizeof`运算符-->与`malloc()`一起使用]<]

还有一件经常看到的事情是利用`sizeof`可以给出任何常量表达式的结果类型大小这一事实。因此，你也可以在那里放一个变量名，并使用它。以下是一个示例，就像之前的那个：

``` {.c}
int *p = malloc(sizeof *p);  // *p是一个int，所以和sizeof(int)一样
```
[i[`sizeof`运算符-->与`malloc()`一起使用]>]
[i[`malloc()`函数]>]

## 错误检查

[i[`malloc()`函数-->错误检查]<]
所有的分配函数都会返回一个指向新分配内存段的指针，或者如果由于某种原因无法分配内存，则返回`NULL`。

一些像Linux这样的操作系统可以被配置成`malloc()`即使内存不足时也不会返回`NULL`。但是尽管如此，您应该始终考虑到保护的编码方式。

``` {.c}
int *x;

x = malloc(sizeof(int) * 10);

if (x == NULL) {
    printf("分配10个整数时出错\n");
    // 在此处执行处理
}
```

下面是一个常见的模式，我们在同一行上进行赋值和条件判断操作：

``` {.c}
int *x;

if ((x = malloc(sizeof(int) * 10)) == NULL)
    printf("分配10个整数时出错\n");
    // 在此处执行处理
}
```
[i[`malloc()`函数-->错误检查]>]

## 为数组分配空间

```c
//malloc()函数-->和数组
我们已经看到如何为单个数据分配空间；现在要为一堆数据在数组中分配空间呢？

在C语言中，数组是相同类型数据在内存中连续排列的一堆数据。

我们可以分配一段连续的内存空间---我们已经知道如何做了。如果我们需要3490个字节的内存，我们可以直接申请：

```c
char *p = malloc(3490);  // 搞定
```

而且--没错！---这就是一个由3490个`char`（也就是字符串！）组成的数组，因为每个`char`占用1个字节。换句话说，`sizeof(char)`就是`1`。

注意：新分配的内存没有被初始化---里面装着垃圾值。如果需要的话，可以用`memset()`清零，或者看下面的`calloc()`。

但我们可以通过把我们需要的数据类型的大小乘以所需元素个数来得到总大小，然后通过指针或数组表示法来访问它们。举个例子！

// sizeof运算符-->配合malloc()
```c {.numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    // 为10个整数分配空间
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

关键在于`malloc()`那一行。如果我们知道每个`int`需要`sizeof(int)`个字节来存储，而且我们需要10个`int`，我们就可以直接分配适量的字节：

```c
sizeof(int) * 10
```

这个技巧对于每种类型都适用。只需要传给`sizeof`并乘以数组的大小。

// sizeof运算符-->配合malloc()
// malloc()函数-->和数组
```

## 一种替代方案：`calloc()`

[`calloc()` 函数]
这是另一个类似于 `malloc()` 的分配函数，有两个关键区别：

* 你不是传递一个参数，而是传递一个元素的大小和你想要分配的元素数量。这就像它是为了分配数组而设计的。
* 它会将内存清零。

你仍然使用 `free()` 来释放通过 `calloc()` 获得的内存。

这里是 `calloc()` 和 `malloc()` 的比较。

``` {.c}
// 使用 calloc() 为 10 个整数分配空间，并初始化为 0：
int *p = calloc(10, sizeof(int));

// 使用 malloc() 为 10 个整数分配空间，并初始化为 0：
int *q = malloc(10 * sizeof(int));
memset(q, 0, 10 * sizeof(int));   // 设置为 0
```

再次强调，除了 `malloc()` 默认不将内存清零之外，两者的结果是相同的。

[`calloc()` 函数]   

## 使用 `realloc()` 更改分配的大小

[`realloc()` 函数]  
如果你已经分配了 10 个整数，但后来你决定需要 20 个，你该怎么办呢？

其中一种选择是分配一些新空间，然后通过 `memcpy()` 将内存复制过去... 但有时候你其实不需要移动任何东西。有一个函数，足够聪明，可以在一切适当的情况下做出正确的事情：`realloc()`。

它接受一个指针，该指针指向之前由 `malloc()` 或 `calloc()` 分配的内存，还需要一个新的内存区域大小。

然后，它会增长或缩小该内存，并返回一个指针。有时它可能返回相同的指针（如果数据不必复制到其他地方），或者它可能返回一个不同的指针（如果数据必须复制）。

在调用 `realloc()` 时，请确保指定要分配的 _字节_ 数量，而不仅仅是数组元素的数量！也就是说：

``` {.c}
num_floats *= 2;
```

```c
np = realloc(p, num_floats * sizeof(float));  // 更好的写法!
```

让我们分配一个包含20个`float`元素的数组，然后改变主意，将其改为包含40个元素。

我们要将`realloc()`的返回值赋给另一个指针，只是为了确保它不是`NULL`。如果不是`NULL`，那么我们可以将其重新赋值给原始指针。（如果我们直接将返回值直接赋给原始指针，如果函数返回`NULL`，我们会丢失该指针，并且无法重新获取。）

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    // 为20个float分配空间
    float *p = malloc(sizeof *p * 20);  // sizeof *p 等同于 sizeof(float)

    // 给它们赋予0.0-1.0的小数值：
    for (int i = 0; i < 20; i++)
        p[i] = i / 20.0;

    // 等等！实际上让它成为包含40个元素的数组
    float *new_p = realloc(p, sizeof *p * 40);

    // 检查是否成功重新分配
    if (new_p == NULL) {
        printf("重新分配出错\n");
        return 1;
    }

    // 如果成功，我们可以重新赋值给p
    p = new_p;

    // 给新元素赋值范围在1.0-2.0之间
    for (int i = 20; i < 40; i++)
        p[i] = 1.0 + (i - 20) / 20.0;

    // 打印所有40个元素中的0.0-2.0的值：
    for (int i = 0; i < 40; i++)
        printf("%f\n", p[i]);

    // 释放空间
    free(p);
}
```

请注意，我们如何将`realloc()`的返回值重新赋给了相同的指针变量`p`，这种做法非常常见。

这里还有一件事，如果第7行看起来有点奇怪，有那个 `sizeof *p`，
记住 `sizeof` 是用于表达式类型的大小。
而 `*p` 的类型是 `float`，所以这行等同于 `sizeof(float)`。

```c
            char *new_buf = realloc(buf, bufsize);

            if (new_buf == NULL) {
                free(buf);   // On error, free and bail
                return NULL;
            }

            buf = new_buf;  // Successful realloc
        }

        buf[offset++] = c;  // Add the byte onto the buffer
    }

    // We hit newline or EOF...

    // If at EOF and we read no bytes, free the buffer and
    // return NULL to indicate we're at EOF:
    if (c == EOF && offset == 0) {
        free(buf);
        return NULL;
    }

    // Shrink to fit
    if (offset < bufsize - 1) {  // If we're short of the end
        char *new_buf = realloc(buf, offset + 1); // +1 for NUL terminator

        // If successful, point buf to new_buf;
        // otherwise we'll just leave buf where it is
        if (new_buf != NULL)
            buf = new_buf;
    }

    // Add the NUL terminator
    buf[offset] = '\0';

    return buf;
}

int main(void)
{
    FILE *fp = fopen("foo.txt", "r");

    char *line;

    while ((line = readline(fp)) != NULL) {
        printf("%s\n", line);
        free(line);
    }

    fclose(fp);
}
```

当扩展内存时，通常（尽管不是铁律）会每次加倍所需空间，以便尽量减少`realloc()`调用的次数。

最后需要注意的是，`readline()`函数返回一个指向用`malloc()`分配的缓冲区的指针。因此，调用方需要在使用完后显式调用`free()`释放那块内存。

### 使用带有`NULL`的`realloc()`

这里有个趣闻！这两行代码是等价的：

``` {.c}
char *p = malloc(3490);
char *p = realloc(NULL, 3490);
```

这在有某种分配循环的情况下可能很方便，你不需要对第一个`malloc()`进行特殊处理。

``` {.c}
int *p = NULL;
int length = 0;

while (!done) {
    // 分配10个额外的整数：
    length += 10;
    p = realloc(p, sizeof *p * length);

    // 进行了一些了不起的操作
    // ...
}
```

在这个示例中，我们不需要初始的`malloc()`，因为`p`一开始就是`NULL`。
[i[`realloc()`函数-->用`NULL`参数]>]

## 对齐内存分配

[i[内存对齐]<]
你可能不会需要使用这个。

我不想在这里深入讨论，但有一件事叫做_内存对齐_，它涉及到内存地址（指针值）是某个数字的倍数。

例如，一个系统可能要求16位值从内存地址开始，这些地址是2的倍数。或者64位值从内存地址开始，这些地址是2、4或8的倍数。这取决于CPU。

一些系统要求这种对齐方式以便快速访问内存，或者完全不能没有对齐方式。

如果你使用 `malloc()`、`calloc()` 或 `realloc()`，C会给你一个对于任何值都很好对齐的内存块，甚至对于`struct`也是适用的。在所有情况下都有效。

但有时候你可能知道某些数据可以以较小的边界对齐，或者出于某种原因必须以较大的边界对齐。我想这在嵌入式系统编程中更为常见。

[i[`aligned_alloc()`函数]<]
在这些情况下，你可以使用`aligned_alloc()`指定对齐方式。

对齐是大于零的2的整数次方，例如`2`、`4`、`8`、`16`等，你在所关心的字节数之前将其传递给`aligned_alloc()`。

``` {.c}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void)
{
    // Allocate 256 bytes aligned on a 64-byte boundary
    char *p = aligned_alloc(64, 256);  // 256 == 64 * 4

    // Copy a string in there and print it
    strcpy(p, "Hello, world!");
    printf("%s\n", p);

    // Free the space
    free(p);
}
```

**要注意的另一个限制是，你分配的字节数需要是对齐值的倍数。但这一点可能会改变。请查看 [fl[C缺陷报告460|http://www.open-std.org/jtc1/sc22/wg14/www/docs/summary.htm#dr_460]]**

我想在这里对 `realloc()` 和 `aligned_alloc()` 作一点说明。`realloc()` 没有任何对齐保证，所以如果需要获取一些对齐的重新分配空间，就必须用 `memcpy()` 的方式来做。

如果需要，这是一个非标准的 `aligned_realloc()` 函数：

``` {.c}
void *aligned_realloc(void *ptr, size_t old_size, size_t alignment, size_t size)
{
    char *new_ptr = aligned_alloc(alignment, size);

    if (new_ptr == NULL)
        return NULL;

    size_t copy_size = old_size < size ? old_size : size;  // 取较小值

    if (ptr != NULL)
        memcpy(new_ptr, ptr, copy_size);

    free(ptr);

    return new_ptr;
}
```

请注意，它总是复制数据，需要时间，而真正的 `realloc()` 如果可能的话就会避免复制。因此，这并不高效。尽量避免需要重新分配自定义对齐的数据。[i[内存对齐]>]
[i[手动内存管理]>]
```