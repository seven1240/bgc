# 不完整类型

您可能会感到惊讶，这段代码可以编译通过而没有错误：

``` {.c}
extern int a[];

int main(void)
{
    struct foo *x;
    union bar *y;
    enum baz *z;
}
```

我们从未给出 `a` 的大小。而且我们有指向从未似乎在任何地方声明过的 `foo`、`bar` 和 `baz` 的结构体指针。

我收到的唯一警告是 `x`、`y` 和 `z` 未被使用。

这些是不完整类型的示例。

不完整类型是指其大小（即从 `sizeof` 得到的大小）未知的类型。另一种理解方式是，这是一种您没有完成声明的类型。

你可以有指向不完整类型的指针，但你不能对其进行解引用或进行指针算术操作。而且你不能用 `sizeof` 计算它的大小。

那么你能做什么呢？

## 使用场景：自引用结构体

我只知道一个真正的用例：对具有自引用或互相依赖结构的 `struct` 或 `union` 进行前向引用。（在这些示例中，我将继续使用 `struct`，但它们同样适用于 `union`。）

让我们先来看一个经典例子。

但在此之前，请注意！当您声明一个 `struct` 时，`struct` 在达到结束大括号之前都是不完整的！

``` {.c}
struct antelope {              // 此处的 struct antelope 是不完整的
    int leg_count;             // 仍然不完整
    float stomach_fullness;    // 仍然不完整
    float top_speed;           // 仍然不完整
    char *nickname;            // 仍然不完整
};                             // 现在它是完整的了。
```

那又怎样？看起来还挺正常的。

但是如果我们在做一个链表呢？每个链表节点都需要引用另一个节点。但是如果我们甚至还没有完成声明节点的过程，那该怎么创建对另一个节点的引用呢？

C语言对未完整类型的容忍使这成为可能。我们无法声明一个节点，但是我们可以声明一个指向它的指针，即使它是不完整的！

``` {.c}
struct node {
    int val;
    struct node *next;  // struct node 是不完整的，但没问题！
};
```

尽管第3行的 `struct node` 是不完整的，我们仍然可以声明一个指向它的指针^[这是因为在C语言中，指针的大小不受指向的数据类型的影响。因此编译器在这一点上不需要知道 `struct node` 的大小；它只需要知道指针的大小。]。

如果我们有两个互相引用的不同 `struct`，我们也可以做同样的事情：

``` {.c}
struct a {
    struct b *x;  // 指向一个 `struct b`
};

struct b {
    struct a *x;  // 指向一个 `struct a`
};
```

如果没有不完整类型的宽松规则，我们将永远不能创建这一对结构。

## 不完整类型错误信息

你是否遇到了如下错误？

``` {.default}
invalid application of ‘sizeof’ to incomplete type

invalid use of undefined type

dereferencing pointer to incomplete type
```

最有可能的原因：你可能忘记了 `#include` 声明类型的头文件。

## 其他不完整类型

声明一个没有具体内容的 `struct` 或 `union` 会创建一个不完整类型，例如`struct foo;`。

`enums` 在闭括号之前都是不完整的。

`void` 是一个不完整类型。

用没有大小的 `extern` 声明的数组是不完整的，例如：

``` {.c}
extern int a[];
```

如果是非`extern`数组，后面没有尺寸并带有初始化器，直到初始化器的闭合括号结束前，它都是不完整的。

## 使用案例：头文件中的数组

在头文件中声明不完整的数组类型是有用的。在这些情况下，实际存储（完整数组声明的位置）应该在单个`.c`文件中。如果将其放在`.h`文件中，每次包含头文件时都会重复。

因此，您可以制作一个带有对数组的不完整类型引用的头文件，如下所示：

``` {.c .numberLines}
// 文件: bar.h

#ifndef BAR_H
#define BAR_H

extern int my_array[];  // 不完整类型

#endif
```

然后在`.c`文件中实际定义数组：

``` {.c .numberLines}
// 文件: bar.c

int my_array[1024];     // 完整类型！
```

然后，您可以从任意位置包含头文件，并且所有这些位置都将引用相同的基础`my_array`。

``` {.c .numberLines}
// 文件: foo.c

#include <stdio.h>
#include "bar.h"    // 包含对my_array的不完整类型

int main(void)
{
    my_array[0] = 10;

    printf("%d\n", my_array[0]);
}
```

在编译多个文件时，请记住向编译器指定所有`.c`文件，但不是`.h`文件，例如：

``` {.zsh}
gcc -o foo foo.c bar.c
```

## 完成不完整类型

如果您有一个不完整类型，您可以通过在同一范围内定义完整的`struct`、`union`、`enum`或数组来完成它。

``` {.c}
struct foo;        // 不完整类型

struct foo *p;     // 指针，没有问题

// struct foo f;   // 错误：不完整类型！

struct foo {
    int x, y, z;
};                 // 现在struct foo是完整的！

struct foo f;      // 成功！
```

请注意，虽然 `void` 是一个不完整类型，但无法对其进行完整定义。并不是有人会考虑这种奇怪的事情。但这解释了为什么可以这样做：

``` {.c}
void *p;             // OK: 指向不完整类型的指针
```

而不是这样做：

``` {.c}
void v;              // Error: 声明不完整类型的变量

printf("%d\n", *p);  // Error: 对不完整类型进行解引用
```

了解得越多，越有助于理解...