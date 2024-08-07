<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 结构体 II：更多有趣的结构体操作

[i[`struct` 关键字]<]

原来你可以用`struct`做的操作远不止我们所讨论的内容，但这些内容只是杂七杂八的。所以我们将它们放在这一章节里。

如果你已经熟悉了`struct`的基础知识，那么你可以在这里扩展你的知识。

## 嵌套`struct`和数组的初始化器

[i[`struct` 关键字-->初始化器]<]

记得我们之前怎么样[初始化结构体成员](#struct-initializers)吗？

``` {.c}
struct foo x = {.a=12, .b=3.14};
```

原来我们在这些初始化器中有更多的功能，比我们最初分享的更加强大。令人兴奋！

首先，如果你有一个嵌套子结构如下所示，你可以按照变量名称一直往下初始化该子结构的成员：

``` {.c}
struct foo x = {.a.b.c=12};
```

让我们看一个例子：

``` {.c .numberLines}
#include <stdio.h>

struct cabin_information {
    int window_count;
    int o2level;
};

struct spaceship {
    char *manufacturer;
    struct cabin_information ci;
};

int main(void)
{
    struct spaceship s = {
        .manufacturer="General Products",
        .ci.window_count = 8,   // <-- 嵌套初始化器！
        .ci.o2level = 21
    };

    printf("%s: %d 座位, %d%% 氧气\n",
        s.manufacturer, s.ci.window_count, s.ci.o2level);
}
```

看看第16-17行！那里我们在定义`s`，我们的`struct spaceship`中初始化了`struct cabin_information`的成员。

以下是相同初始化器的另一种选项--这一次我们将做一些更加标准的看起来，但任何方法都可以工作：

``` {.c .numberLines startFrom="15"}
    struct spaceship s = {
        .manufacturer="General Products",
        .ci={
            .window_count = 8,
            .o2level = 21
        }
    };
```

现在，如果上面的信息还不够惊人，我们还可以在其中混合使用数组初始化器。

让我们改变一下，将乘客信息数组放进去，我们也可以看看初始化器是如何工作的。

``` {.c .numberLines}
#include <stdio.h>

struct passenger {
    char *name;
    int covid_vaccinated; // 布尔型
};

#define MAX_PASSENGERS 8

struct spaceship {
    char *manufacturer;
    struct passenger passenger[MAX_PASSENGERS];
};

int main(void)
{
    struct spaceship s = {
        .manufacturer="General Products",
        .passenger = {
            // 逐个字段初始化
            [0].name = "格里德利，刘易斯",
            [0].covid_vaccinated = 0,

            // 或一次性初始化
            [7] = {.name="布朗，提拉", .covid_vaccinated=1},
        }
    };

    printf("%s飞船的乘客信息：\n", s.manufacturer);

    for (int i = 0; i < MAX_PASSENGERS; i++)
        if (s.passenger[i].name != NULL)
            printf("    %s（%s接种疫苗）\n",
                s.passenger[i].name,
                s.passenger[i].covid_vaccinated? "已": "未 ");
}
```

[i[`struct` keyword-->initializers]>]

## 匿名结构体

[i[`struct` keyword-->anonymous]<]

这些是“没有名称的`struct`”。我们在[`typedef`](#typedef-struct)部分也提到过这些，但这里稍作解释。

这是一个常规的`struct`：

``` {.c}
struct animal {
    char *name;
    int leg_count, speed;
};
```

这是匿名`struct`的等效形式：

``` {.c}
struct {              // <-- 没有名称！
    char *name;
    int leg_count, speed;
};
```

好的。所以我们有一个`struct`，但它没有名字，所以我们之后没法使用它？看起来相当没有意义。

诚然，在那个例子中，是这样的。但我们仍然可以以几种方式利用它。

一种很少见，但由于匿名的`struct`表示一种类型，我们可以在它之后添加一些变量名并使用它们。

``` {.c}
struct {                   // <-- 没有名字！
    char *name;
    int leg_count, speed;
} a, b, c;                 // 这个结构类型的3个变量

a.name = "antelope";
c.leg_count = 4;           // 例如
```

但那仍然不是很有用。

更常见的是使用具有`typedef`的匿名`struct`，以便我们可以稍后使用它（例如将变量传递给函数）。

``` {.c}
typedef struct {                   // <-- 没有名字！
    char *name;
    int leg_count, speed;
} animal;                          // 新类型：animal

animal a, b, c;

a.name = "antelope";
c.leg_count = 4;           // 例如
```

就个人而言，我不太使用匿名`struct`。我认为在声明中将整个`struct animal`看到变量名之前会更令人愉悦。

但那只是我的看法而已。

## 自指向的`struct`

对于任何类似图形的数据结构，能够有指向连接节点/顶点的指针是很有用的。但这意味着在节点的定义中，你需要一个指向节点的指针。这是先有鸡还是先有蛋的问题！

但事实证明，在C中完全没有问题地做到这一点。

例如，这是一个链表节点：

``` {.c}
struct node {
    int data;
    struct node *next;
};
```

需要注意的是`next`是一个指针。这就是整个构建过程的基础。即使编译器还不知道整个`struct node`的具体样子，所有指针的大小都是相同的。

这里有一个简单的链表程序来测试它：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

struct node {
    int data;
    struct node *next;
};

int main(void)
{
    struct node *head;

    // 通过一种巧妙的方式建立一个链表 (11)->(22)->(33)
    head = malloc(sizeof(struct node));
    head->data = 11;
    head->next = malloc(sizeof(struct node));
    head->next->data = 22;
    head->next->next = malloc(sizeof(struct node));
    head->next->next->data = 33;
    head->next->next->next = NULL;

    // 遍历链表
    for (struct node *cur = head; cur != NULL; cur = cur->next) {
        printf("%d\n", cur->data);
    }
}
```

运行以上代码会输出：

``` {.default}
11
22
33
```

结构体数组成员

回到那个遥远的年代，当人们用木头雕刻C代码时，有些人认为如果能够在`struct`末尾分配可变长度的数组会很方便。 

需要澄清的是，本节的第一部分是旧的做法，之后我们会介绍一种新的方式。

例如，也许你可以定义一个用于保存字符串以及字符串长度的`struct`。它会包含一个长度和一个保存数据的数组。也许像这样：
``` {.c}
struct len_string {
    int length;
    char data[8];
};
```

但这将`8`硬编码为字符串的最大长度，并不是很好。如果我们做一些_聪明_的事情，只需在结构体末尾`malloc()`一些额外的空间，然后让数据溢出到这个空间中会怎么样呢？

让我们这样做，然后在其上再分配另外40个字节：

``` {.c}
struct len_string *s = malloc(sizeof *s + 40);
```

因为`data`是`struct`的最后一个字段，如果我们溢出该字段，它将流向我们已经分配的空间！因此，这个技巧只有在短数组是`struct`中的_最后_字段时才有效。

``` {.c}
// 复制超过8个字节！

strcpy(s->data, "Hello, world!");  // 不会崩溃。可能。
```

实际上，有一个常见的编译器解决方法可以做到这一点，其中你会在最后分配一个零长度的数组：

``` {.c}
struct len_string {
    int length;
    char data[0];
};
```

然后你分配的每个额外字节都可以在这个字符串中使用。

因为`data`是`struct`的最后一个字段，如果我们溢出该字段，它将流向我们已经分配的空间！

``` {.c}
// 复制超过8个字节！

strcpy(s->data, "Hello, world!");  // 不会崩溃。可能。
```

但是，当然，实际上访问超出该数组末尾的数据是未定义行为！在现代时代，我们不再沦为这种野蛮行径。

幸运的是，我们仍然可以在C99及更高版本中获得相同效果，不过现在是合法的。

让我们只需将上面的定义更改为数组没有大小^[严格来说我们说它有一个_不完全类型_]：

``` {.c}
struct len_string {
    int length;
    char data[];
};
```

同样，只有当灵活数组成员是`struct`中的_最后_字段时，此方法才有效。

然后，我们可以通过比 `struct len_string` 更大的空间进行 `malloc()`，为这些字符串分配所需的所有空间，就像在下面的示例中所做的那样，从一个C字符串创建一个新的 `struct len_string`：

``` {.c}
struct len_string *len_string_from_c_string(char *s)
{
    int len = strlen(s);

    // 为我们通常所需的字节数额外分配 "len" 个字节
    struct len_string *ls = malloc(sizeof *ls + len);

    ls->length = len;

    // 将字符串复制到这些额外的字节中
    memcpy(ls->data, s, len);

    return ls;
}
```

## 填充字节 {#struct-padding-bytes}

需要注意的是，C语言可以根据需要在 `struct` 中添加填充字节。不能保证它们会直接相邻存储在内存中^[尽管一些编译器有选项可以强制这种情况发生---搜索 `__attribute__((packed))` 以了解如何在GCC中实现此操作]。

让我们看一下这个程序。我们输出两个数字。一个是各个字段类型的 `sizeof` 之和，另一个是整个 `struct` 的 `sizeof`。

人们可能期望它们是相同的。总大小应该等于各部分大小之和，对吧？

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    int a;
    char b;
    int c;
    char d;
};

int main(void)
{
    printf("%zu\n", sizeof(int) + sizeof(char) + sizeof(int) + sizeof(char));
    printf("%zu\n", sizeof(struct foo));
}
```

但是在我系统上，这输出：

``` {.default}
10
16
```

它们不相同！编译器添加了6个字节的填充以提高性能。也许您的编译器输出结果不同，但除非您强制，否则不能确定是否有填充。

## `offsetof`

前一节中，我们看到编译器可以在结构体的内部任意注入填充字节。

如果我们需要知道填充字节的位置怎么办？我们可以使用在 `<stddef.h>` 中定义的 `offsetof` 进行测量。

让我们修改上面的代码以打印 `struct` 中各个字段的偏移量：

``` {.c .numberLines}
#include <stdio.h>
#include <stddef.h>

struct foo {
    int a;
    char b;
    int c;
    char d;
};

int main(void)
{
    printf("%zu\n", offsetof(struct foo, a));
    printf("%zu\n", offsetof(struct foo, b));
    printf("%zu\n", offsetof(struct foo, c));
    printf("%zu\n", offsetof(struct foo, d));
}
```

对我来说，这会输出：

``` {.default}
0
4
8
12
```

这表明我们为每个字段使用了 4 个字节。这有点奇怪，因为 `char` 只有 1 个字节，对吧？编译器在每个 `char` 后放置了 3 个填充字节，以确保所有字段的长度都是 4 个字节。这样在我的 CPU 上可能会运行得更快。

有一种略微滥用的东西，有点类似于面向对象编程，你可以用 `struct` 做。

由于指向 `struct` 的指针与指向 `struct` 第一个元素的指针相同，因此你可以自由地将指向 `struct` 的指针转换为指向第一个元素的指针。

这意味着我们可以设置这样一种情况：

``` {.c}
struct parent {
    int a, b;
};

struct child {
    struct parent super;  // 必须是第一个
    int c, d;
};
```

然后，我们可以将指向 `struct child` 的指针传递给一个期望两者中的任一个或者指向 `struct parent` 的函数！

因为 `struct parent super` 是 `struct child` 中的第一项，任何 `struct child` 的指针与那个 `super` 字段的指针是相同的^[顺便提一下，`super` 不是关键字，我只是借用了一些面向对象编程的术语]。

让我们举个例子。我们会像上面那样创建 `struct`，但是然后我们将把指向 `struct child` 的指针传递给需要一个指向 `struct parent` 的函数... 它仍然可以工作。

``` {.c .numberLines}
#include <stdio.h>

struct parent {
    int a, b;
};

struct child {
    struct parent super;  // 必须是第一个
    int c, d;
};

// 将参数设置为 `void*`，这样我们可以传入任何类型的参数
// （即 struct parent 或 struct child）
void print_parent(void *p)
{
    // 预期一个 struct parent--但结构体 child 也可以工作
    // 因为指针指向了第一个字段中的 struct parent：
    struct parent *self = p;

    printf("Parent: %d, %d\n", self->a, self->b);
}

void print_child(struct child *self)
{
    printf("Child: %d, %d\n", self->c, self->d);
}

int main(void)
{
    struct child c = {.super.a=1, .super.b=2, .c=3, .d=4};

    print_child(&c);
    print_parent(&c);  // 即使是 struct child，仍然可以工作！
}
```

看看我们在`main()`的最后一行做了什么？我们调用了`print_parent()`，但传递了一个`struct child*`作为参数！尽管`print_parent()`需要参数指向`struct parent`，但我们这样做是可以的，因为`struct child`中的第一个字段就是`struct parent`。

再次说明，这样做是因为指向`struct`的指针与指向该`struct`的第一个字段的指针具有相同的值。

这一切都依赖于规范中的这一部分：

> **§6.7.2.1¶15** [...] 指向结构对象的指针经过适当转换指向其初始成员[...]，反之亦然。

以及

> **§6.5¶7** 对象的存储值只能通过具有以下类型之一的lvalue表达式进行访问：
>
> * 与对象的有效类型兼容的类型
> * [...]

以及我对“适当转换”意味着“转换为初始成员的有效类型”的假设。

## 位域

在我看来，这些很少使用，但你可能会偶尔在一些较低级的应用程序中看到它们，特别是将位打包到较大空间中的应用程序中。

让我们看一些代码来演示一个用例：

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    unsigned int a;
    unsigned int b;
    unsigned int c;
    unsigned int d;
};

int main(void)
{
    printf("%zu\n", sizeof(struct foo));
}
```

对我来说，这会打印`16`。这是有道理的，因为在我的系统上，`unsigned`是4字节。

但是，如果我们知道要存储在`a`和`b`中的所有值可以在5位中存储，而`c`和`d`中的值可以在3位中存储，那么只需要总共16位。如果我们只使用16位，为什么为它们保留128位呢？

好吧，我们可以告诉C，请尽量尝试将这些值打包在一起。我们可以指定值可以占用的最大位数（从1到包含类型的大小）。

我们通过在字段名后面放置一个冒号，然后跟着位数来做到这一点。

``` {.c .numberLines startFrom="3"}
struct foo {
    unsigned int a:5;   // a:5 表示这个字段可以占用 5 位
    unsigned int b:5;   // b:5 表示这个字段可以占用 5 位
    unsigned int c:3;   // c:3 表示这个字段可以占用 3 位
    unsigned int d:3;   // d:3 表示这个字段可以占用 3 位
};
```

现在当我询问C我的`struct foo`有多大时，它告诉我是4！它原来是16字节，但现在只有4字节。它已经“打包”这4个值进入4个字节，实现了四倍的内存节省。

当然，这种权衡在于，5位字段只能存储0-31之间的值，而3位字段只能存储0-7之间的值。但生活终归是充满妥协的。

### 非相邻位域

一个需要注意的地方：C只会合并**相邻**的位域。如果它们被非位域打断，你将无法实现节省：

``` {.c}
struct foo {            // sizeof(struct foo) == 16 （可能因系统而异）
    unsigned int a:1;   // 因为a不在c旁边，所以
    unsigned int b;     // 现在这里有一个额外的整数
    unsigned int c:1;   // 由于a和c不相邻，它们各自占用一个整数
    unsigned int d;     // 现在这里有一个额外的整数
};
```

在这个例子中，由于`a`不与`c`相邻，它们分别被“打包”在自己的整数中。

因此，`a`、`b`、`c`和`d`每个都有一个整数。由于我的整数是4字节，所以总共是16字节。

通过快速重新排列可将空间节省从16字节降至12字节（可能因系统而异）：

``` {.c}
struct foo {            // sizeof(struct foo) == 12 （可能因系统而异）
    unsigned int a:1;
    unsigned int c:1;
    unsigned int b;
    unsigned int d;
};
```

现在，由于`a`和`c`相邻，编译器将它们合并为单个整数。

所以我们有一个用于包含 `a` 和 `c` 的 `int`，以及分别用于 `b` 和 `d` 的一个 `int`。总共是3个 `int`，或者说12个字节。

把所有的位域放在一起，让编译器将它们合并。

### 有符号或无符号 `int`

如果只声明一个位域为 `int`，不同的编译器会把它视为 `signed` 或 `unsigned`。就像对待 `char` 的情况一样。

在使用位域时要明确有符号性。

### 未命名的位域

在某些特定情况下，您可能需要出于硬件原因保留一些位，但在代码中不需要使用它们。

例如，假设您有一个字节，其中顶部2位有意义，底部1位有意义，但中间的5位对您没有用^[假设采用8位 `char`，即 `CHAR_BIT == 8`]。

我们可以这样做：

``` {.c}
struct foo {
    unsigned char a:2;
    unsigned char dummy:5;
    unsigned char b:1;
};
```

这样做有效——在我们的代码中，我们使用 `a` 和 `b`，但从不使用 `dummy`。它只是用来吃掉5位，以确保 `a` 和 `b` 位于字节中“必需”的（通过这个矫揉造作的示例）位置。

C 允许我们一种方法来整理：_未命名的位域_。在这种情况下，你可以只是省略名称（`dummy`），C 对于同样的效果很满意：

``` {.c}
struct foo {
    unsigned char a:2;
    unsigned char :5;   // <-- 未命名的位域！
    unsigned char b:1;
};

### 零宽度未命名的位域

这里有一些更加深奥的知识…… 假设您在将位打包到 `unsigned int` 中，并且需要一些相邻的位域打包到下一个 `unsigned int`。 

也就是说，如果你这样做：
``` {.c}
struct foo {
    unsigned int a:1;
    unsigned int b:2;
    unsigned int c:3;
    unsigned int d:4;
};
```

编译器将所有这些内容打包到一个单独的 `unsigned int` 中。但是如果您需要将 `a` 和 `b` 放在一个 `int` 中，将 `c` 和 `d` 放在另一个 `int` 中，该怎么办呢？

有一个解决方法：在您希望编译器重新开始在不同的 `int` 中打包位的地方放置一个宽度为 `0` 的未命名位域：

``` {.c}
struct foo {
    unsigned int a:1;
    unsigned int b:2;
    unsigned int :0;   // <--零宽度的未命名位域
    unsigned int c:3;
    unsigned int d:4;
};
```

这类似于文字处理器中的显式分页符。您在告诉编译器，"停止在这个 `unsigned int` 中打包位，开始在下一个 `unsigned int` 中打包位。"

通过在那个位置添加零宽度的未命名位域，编译器将会将 `a` 和 `b` 放在一个 `unsigned int` 中，将 `c` 和 `d` 放在另一个 `unsigned int` 中。总共两个，对于我的系统来说大小为 8 字节（每个 `unsigned int` 都是 4 字节）。

[i[`struct` keyword-->bit fields]>]

## 联合

[i[`union` keyword]<]

这些基本上与 `struct` 相同，但字段在内存中重叠。`union` 的大小仅仅足够容纳最大的字段，并且一次只能使用一个字段。

这是一种在不同类型的数据之间重复使用相同内存空间的方法。

声明它们的方式与 `struct` 相同，只是将 `struct` 换成了 `union`。看下面这个示例：

``` {.c}
union foo {
    int a, b, c, d, e, f;
    float g, h;
    char i, j, k, l;
};
```

现在，这是很多字段。如果这是一个 `struct`，我的系统会告诉我需要 36 字节才能容纳所有字段。

但这是一个 `union`，因此所有这些字段在相同的内存区域中重叠。最大的是 `int`（或 `float`），在我的系统上占据 4 个字节。事实上，如果我要求 `union foo` 的 `sizeof`，它会告诉我是 4！

权衡之处在于您只能同时移植使用其中一个字段。然而...

### 联合和类型强制转换

您可以非移植地向一个 `union` 字段写入数据，然后从另一个字段读取数据！

这样做被称为[类型强制转换](#type-punning)，您会在真正清楚自己在做什么的情况下使用它，通常是在某种低级编程中。

因为 union 的成员共享同一内存空间，向一个成员写入数据必然会影响到其他成员。如果您从一个成员读取了另一个成员写入的数据，就会得到一些奇怪的效果。

``` {.c .numberLines}
#include <stdio.h>

union foo {
    float b;  // 浮点数
    short a;  // 短整数
};

int main(void)
{
    union foo x;

    x.b = 3.14159;

    printf("%f\n", x.b);  // 3.14159，没问题

    printf("%d\n", x.a);  // 那么这个会怎样呢？
}
```

在我的系统上，这会输出：

```
3.141590
4048
```

因为在底层，浮点数 `3.14159` 的对象表示与短整数 `4048` 的对象表示相同。在我的系统上是这样。您的结果可能会有所不同。

---

### 指针与 `union`

如果您有一个指向 `union` 的指针，您可以将该指针转换为该 `union` 中字段的任何类型，并以这种方式获得相应的值。

在这个例子中，我们看到 `union` 中有 `int` 和 `float` 类型。我们获取 `union` 的指针，但将它们转换为 `int*` 和 `float*` 类型（转换会消除编译器警告）。然后，如果我们对这些指针进行解引用，我们会发现它们具有我们直接存储在 `union` 中的值。

``` {.c .numberLines}
#include <stdio.h>

union foo {
    int a, b, c, d, e, f; // 整数
    float g, h;  // 浮点数
    char i, j, k, l;  // 字符
};

int main(void)
{
    union foo x;

    int *foo_int_p = (int *)&x;
    float *foo_float_p = (float *)&x;

```c
    x.a = 12;
    printf("%d\n", x.a);           // 12
    printf("%d\n", *foo_int_p);    // 12, 再次

    x.g = 3.141592;
    printf("%f\n", x.g);           // 3.141592
    printf("%f\n", *foo_float_p);  // 3.141592, 再次
}
```

反过来也是一样的。如果我们有一个指向`union`内部类型的指针，我们可以将其转换为指向`union`的指针并访问其成员。

```c
union foo x;
int *foo_int_p = (int *)&x;             // 指向 int 字段的指针
union foo *p = (union foo *)foo_int_p;  // 回到指向 union 的指针

p->a = 12;  // 这一行和...
x.a = 12;   // 这一行是一样的。
```

所有这些只是让你知道，在底层，`union`中的所有这些值都从内存中的同一位置开始，并且这与整个`union`的位置相同。

### `union` 中的公共初始序列

如果在`union`中有`struct`的组合，并且所有这些`struct`以一个_公共初始序列_开始，那么可以从任何`union`成员中访问该序列的成员是有效的。

什么?

这里有两个具有公共初始序列的`struct`：

```c
struct a {
	int x;     //
	float y;   // 公共初始序列

	char *p;
};

struct b {
	int x;     //
	float y;   // 公共初始序列

	double *p;
	short z;
};
```

你看到了吗? 它们以`int`开头，后面跟着`float`---这就是公共初始序列。`struct`序列中的成员必须是兼容的类型。我们看到 `x` 和 `y`，它们分别是`int`和`float`。

现在让我们构建这些结构的一个`union`：

```c
union foo {
	struct a sa;
	struct b sb;
};
```

这条规则告诉我们，我们可以保证共同初始序列的成员在代码中是可以互换的。也就是说：

* `f.sa.x` 等同于 `f.sb.x`。

以及

* `f.sa.y` 等同于 `f.sb.y`。

因为字段 `x` 和 `y` 都在共同初始序列中。

此外，共同初始序列中的成员的名称并不重要---重要的是类型要相同。

总的来说，这使我们可以安全地在 `union` 中的 `struct` 之间添加一些共享信息。可能最好的例子是使用一个字段来确定 `union` 中当前“正在使用”的所有 `struct` 的类型。

也就是说，如果我们不被允许这样做，并且将 `union` 传递给某个函数，那么该函数如何知道应该查看 `union` 中的哪个成员呢？

看一下这些 `struct`。注意共同的初始序列：

``` {.c .numberLines}
#include <stdio.h>

struct common {
	int type;   // 共同的初始序列
};

struct antelope {
	int type;   // 共同的初始序列

	int loudness;
};

struct octopus {
	int type;   // 共同的初始序列

	int sea_creature;
	float intelligence;
};

```

现在让我们把它们放入一个 `union` 中：

``` {.c .numberLines startFrom="20"}
union animal {
	struct common common;
	struct antelope antelope;
	struct octopus octopus;
};

```

同时，请容许我为演示添加这两个 `#define`：

``` {.c .numberLines startFrom="26"}
#define ANTELOPE 1
#define OCTOPUS  2

```

到目前为止，这里没有发生任何特殊的事情。`type` 字段似乎完全没用。

但现在让我们创建一个打印 `union animal` 的通用函数。它必须某种方式能够识别正在查看 `struct antelope` 还是 `struct octopus`。

由于共同初始序列的神奇，它可以在特定的 `union animal x` 中的任何位置查找动物类型：

``` {.c}
int type = x.common.type;    \\ 或者...
int type = x.antelope.type;  \\ 或者...
int type = x.octopus.type;
```

所有这些引用的是内存中的相同值。

而且，正如你可能已经猜到的，`struct common` 存在是为了使代码可以在不提及特定动物的情况下查看类型。

让我们看一下打印 `union animal` 的代码：

``` {.c .numberLines startFrom="29"}
void print_animal(union animal *x)
{
	switch (x->common.type) {
		case ANTELOPE:
			printf("Antelope: loudness=%d\n", x->antelope.loudness);
			break;

		case OCTOPUS:
			printf("Octopus : sea_creature=%d\n", x->octopus.sea_creature);
			printf("          intelligence=%f\n", x->octopus.intelligence);
			break;
		
		default:
			printf("Unknown animal type\n");
	}

}

int main(void)
{
	union animal a = {.antelope.type=ANTELOPE, .antelope.loudness=12};
	union animal b = {.octopus.type=OCTOPUS, .octopus.sea_creature=1,
	                                   .octopus.intelligence=12.8};

	print_animal(&a);
	print_animal(&b);
}
```

看看第29行，我们只是传递 `union` 进去 -- 我们并不知道其中使用了哪种类型的动物 `struct`。

但没关系！因为在第31行，我们检查类型以查看它是羚羊还是章鱼。然后我们可以查看适当的 `struct` 来获取成员。

使用只有 `struct`s 绝对可以实现相同的效果，但如果你希望获得 `union` 的节省内存的效果，可以使用这种方式。

[i[`union` 关键字-->共同初始序列]>]

## Unions 和 未命名的 Structs

[i[`union` 关键字-->和未命名 `struct`s]<]

你知道你可以有一个未命名的 `struct`，就像这样：

``` {.c}
struct {
    int x, y;
} s;
```

这定义了一个变量`s`，它是一个匿名的`struct`类型（因为`struct`没有带名标签），它有成员`x`和`y`。

所以下面这样的操作是有效的：

``` {.c}
s.x = 34;
s.y = 90;

printf("%d %d\n", s.x, s.y);
```

原来你可以像预期的那样在`union`中放弃那些没有名字的`struct`：

``` {.c}
union foo {
    struct {       // 没有名字！
        int x, y;
    } a;

    struct {       // 没有名字！
        int z, w;
    } b;
};
```

然后像平常一样访问它们：

``` {.c}
union foo f;

f.a.x = 1;
f.a.y = 2;
f.b.z = 3;
f.b.w = 4;
```

没问题！

## 传递和返回`struct`和`union`

你可以通过值（而不是指针）将`struct`或`union`传递给函数---将一个对象的副本传递给参数，就像通常的赋值一样。

你也可以从函数返回`struct`或`union`，它也是通过值返回的。

``` {.c .numberLines}
#include <stdio.h>

struct foo {
    int x, y;
};

struct foo f(void)
{
    return (struct foo){.x=34, .y=90};
}

int main(void)
{
    struct foo a = f();  // 复制被做了

    printf("%d %d\n", a.x, a.y);
}
```

有趣的事实：如果你这样做，你可以直接在函数调用后使用`.`操作符：

``` {.c .numberLines startFrom="16"}
    printf("%d %d\n", f().x, f().y);
```

（当然，这个例子会两次调用函数，效率较低。）

对于返回`struct`和`union`的指针同样适用---在这种情况下确保使用`->`箭头操作符。

[‘union’关键字-->传递和返回]
[‘struct’关键字-->传递和返回]
[‘union’关键字]
[‘struct’关键字]