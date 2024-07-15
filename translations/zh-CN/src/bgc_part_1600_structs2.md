<!-- Beej's指南：C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# `struct`s II：更多有趣的`struct`s

[i[`struct` 关键字]<]

原来`struct`s有很多功能，比我们讨论过的要多，但这些功能混杂在一起，所以我们将它们放在这一章节。

如果你对`struct`的基础知识很熟悉，那么可以在这里进一步完善你的知识。

## 嵌套`struct`和数组的初始化器

[i[`struct`关键字-->初始化器]<]

还记得你可以[按下面这样初始化结构体成员吗](#struct-initializers)？

``` {.c}
struct foo x = {.a=12, .b=3.14};
```

原来在这些初始化器中我们拥有比最初分享的更多的功能。令人兴奋！

首先，如果你有一个嵌套的子结构，像下面这样，你可以通过沿着变量名称进行初始化：

``` {.c}
struct foo x = {.a.b.c=12};
```

让我们看一个示例：

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

    printf("%s: %d 个座位，氧气含量 %d%%\n",
        s.manufacturer, s.ci.window_count, s.ci.o2level);
}
```

看看第16-17行！这里我们在`struct spaceship`的定义中初始化`struct cabin_information`的成员。

下面是相同初始化器的另一个选项---这次我们将使用更标准的形式，但两种方法都可以：

``` {.c .numberLines startFrom="15"}
    struct spaceship s = {
        .manufacturer="General Products",
        .ci={
            .window_count = 8,
            .o2level = 21
        }
    };
```

现在，就好像上面的信息还不够出色一样，我们还可以在里面混合使用数组初始化器。

让我们做出改变，将乘客信息数组加进去，我们可以看看初始化器是如何工作的。

``` {.c .numberLines}
#include <stdio.h>

struct passenger {
    char *name;
    int covid_vaccinated; // Boolean
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
            // 逐一初始化字段
            [0].name = "Gridley, Lewis",
            [0].covid_vaccinated = 0,

            // 或一次性全部初始化
            [7] = {.name="Brown, Teela", .covid_vaccinated=1},
        }
    };

    printf("乘坐 %s 的乘客信息：\n", s.manufacturer);

    for (int i = 0; i < MAX_PASSENGERS; i++)
        if (s.passenger[i].name != NULL)
            printf("    %s（%s接种疫苗）\n",
                s.passenger[i].name,
                s.passenger[i].covid_vaccinated? "": "未");
}
```

[i[`struct`关键字-->初始化器]>]

## 匿名`struct`

[i[`struct`关键字-->匿名]<]

这些是“没有名字的`struct`”。我们也在[`typedef`](#typedef-struct)部分提到过这些，但我们在这里再次提及。

这是一个常规的`struct`：

``` {.c}
struct animal {
    char *name;
    int leg_count, speed;
};
```

下面是匿名的等效形式：

``` {.c}
struct {              // <-- 没有名字！
    char *name;
    int leg_count, speed;
};
```

Okaaaaay. 所以我们有一个`struct`，但它没有名字，那我们之后怎么使用它呢？似乎没什么意义。

诚然，在那个例子中，确实是这样。但我们仍然可以以几种方式利用它。

其中一种很少见，但由于匿名的`struct`代表一种类型，我们可以在其后放一些变量名并使用它们。

``` {.c}
struct {                   // <-- 没有名字！
    char *name;
    int leg_count, speed;
} a, b, c;                 // 这种结构类型的3个变量

a.name = "antelope";
c.leg_count = 4;           // 举例
```

但这还是没什么用。

更常见的是使用带有`typedef`的匿名`struct`，这样我们可以在之后使用它（例如，将变量传递给函数）。

``` {.c}
typedef struct {                   // <-- 没有名字！
    char *name;
    int leg_count, speed;
} animal;                          // 新的类型：animal

animal a, b, c;

a.name = "antelope";
c.leg_count = 4;           // 举例
```

就个人而言，我不太使用匿名`struct`。我认为在声明中，在变量名之前看到整个`struct animal`更加舒适。

但这只是，像，我的看法啦。

[i[`struct` keyword-->anonymous]>]

## 自引用`struct`们

[i[`struct` keyword-->self-referential]<]

对于任何类似图形的数据结构，能够有指向连接节点/顶点的指针非常有用。但这意味着在节点的定义中，你需要有一个指向节点的指针。这有点像先有鸡还是先有蛋的问题！

但事实证明，在C中，这完全没问题。

例如，这是一个链表节点：

``` {.c}
struct node {
    int data;
    struct node *next;
};
```

重要的是要注意，`next` 是一个指针。这正是整个程序能够成功构建的原因。尽管编译器还不知道整个 `struct node` 的全部内容是什么样，但所有指针的大小都是相同的。

以下是一个简单的链表程序，用来测试它：

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

    // 粗略地设置一个链表 (11)->(22)->(33)
    head = malloc(sizeof(struct node));
    head->data = 11;
    head->next = malloc(sizeof(struct node));
    head->next->data = 22;
    head->next->next = malloc(sizeof(struct node));
    head->next->next->data = 33;
    head->next->next->next = NULL;

    // 遍历
    for (struct node *cur = head; cur != NULL; cur = cur->next) {
        printf("%d\n", cur->data);
    }
}
```

运行后会打印：

``` {.default}
11
22
33
```

[i[`struct` 关键字-->自引用]]>

## 灵活数组成员

[i[`struct` 关键字-->灵活数组成员]]>

在很久以前，当人们用木头雕刻 C 代码时，一些人认为如果他们能够为 `struct` 分配具有可变长度数组的空间那就很好玩了。

我想要明确指出，本节的第一部分是过去的做法，之后我们会介绍新的方法。

例如，也许你可以定义一个用于保存字符串及其长度的 `struct`。它将包含一个长度和一个用于保存数据的数组。可能是这样的结构：

``` {.c}

但是这个地方设定了字符串的最大长度是`8`，有点少了。要不我们来点 _聪明_ 的，直接在结构体后面`malloc()`几个额外的空间，让数据溢出到那些空间里怎么样？

让我们试试，再在上面分配另外`40`个字节：

``` {.c}
struct len_string *s = malloc(sizeof *s + 40);
```

因为`data`是结构体中的最后一个字段，如果我们溢出这个字段，它就会延伸到我们已经分配的空间里！因此，这个技巧只有在短数组是`struct`中的 _最后一个_ 字段时才有效。

``` {.c}
// 复制超过 8 个字节！

strcpy(s->data, "Hello, world!");  // 不会崩溃。大概吧。
```

实际上，以前有一个常见的编译器变通方法可以做到这一点，就是在结尾分配一个零长度的数组：

``` {.c}
struct len_string {
    int length;
    char data[0];
};
```

然后你分配的每个额外字节都可以用在那个字符串里。

因为`data`是`struct`的最后一个字段，如果我们溢出这个字段，它就会延伸到我们已经分配的空间里！

``` {.c}
// 复制超过 8 个字节！

strcpy(s->data, "Hello, world!");  // 不会崩溃。大概吧。
```

但当然，实际上访问数组末尾之外的数据是未定义行为！在这个现代时代，我们不再屈尊使用这种野蛮的方法。

幸运的是，我们在C99及更高版本中仍然可以实现相同的效果，而且现在是合法的。

让我们只需修改上面的定义，对数组不设定大小^[从技术上讲，我们说它有一个 _不完整类型_ 。]:

``` {.c}
struct len_string {
    int length;
    char data[];
};
```

同样，这只有在柔性数组成员是`struct`中的 _最后一个_ 字段时才有效。

然后我们可以通过比`struct len_string`更大的`malloc()`来为这些字符串分配所有我们想要的空间，就像在这个示例中所做的那样，从C字符串创建新的`struct len_string`：

``` {.c}
struct len_string *len_string_from_c_string(char *s)
{
    int len = strlen(s);

    // 分配比通常需要的"len"更多的字节
    struct len_string *ls = malloc(sizeof *ls + len);

    ls->length = len;

    // 将字符串复制到这些额外的字节中
    memcpy(ls->data, s, len);

    return ls;
}
```

## 填充字节 {#struct-padding-bytes}

C语言允许在`struct`内部或之后添加填充字节，具体位置由其自行决定。不能保证它们在内存中是直接相邻的^[尽管一些编译器有选项可以强制使其完成此操作---搜索`__attribute__((packed))`以了解如何在GCC中执行此操作]。

让我们看看这个程序。我们输出两个数值。一个是各个字段类型的`sizeof`之和。另一个是整个`struct`的`sizeof`。

人们可能指望它们是相同的。总大小应该是部分大小之和，对吧？

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

但是在我的系统上，这个输出是：

``` {.default}
10
16
```

它们并不相同！编译器添加了6个字节的填充以提高性能。也许您在您的编译器上得到了不同的输出，但是除非您强制执行，否则无法确定没有填充。

## `offsetof`

```c
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

尊贵的顾客，现在展示给您看：

``` {.default}
0
4
8
12
```

这表明我们对每个字段使用了4个字节。这有点奇怪，因为`char`只有1个字节，对吧？编译器在每个`char`后面放置3个填充字节，以便使所有字段都是4个字节长。这样做可能会在我的 CPU 上运行得更快。

<!--

6.7.2.1

15 在结构对象内部，非位域成员和位域所在的单元的地址按照它们声明的顺序递增。对结构对象的指针，经过适当转换，指向其初始成员（或者如果那个成员是位域，则指向该单位所在的位置），反之亦然。结构对象内部可能存在未命名的填充，但不在其开头。

6.2.7 兼容类型和复合类型

1 两种类型具有兼容类型如果它们的类型是相同的。

6.5

7 对象的存储值只能由与对象的有效类型兼容的lvalue表达式访问：

- 与对象的有效类型兼容的类型

-->
```

有一件有点像面向对象（OOP）的不太友好的事情，你可以用`struct`去完成。

因为`struct`的指针与`struct`的第一个元素的指针是相同的，你可以自由地将指向`struct`的指针转换为指向第一个元素的指针。

这意味着我们可以设置这样的情况：

``` {.c}
struct parent {
    int a, b;
};

struct child {
    struct parent super;  // 必须放在第一位
    int c, d;
};
```

然后我们可以将指向`struct child`的指针传递给一个期望接受`struct child`或`struct parent`指针的函数！

因为`struct parent super`是`struct child`中的第一项，任何`struct child`的指针都等同于指向`super`字段的指针^[‘super’实际上并不是关键字。我只是借用了一些OOP术语.]。

让我们举个例子。我们定义如上的`struct`，然后将`struct child`的指针传递给一个需要`struct parent`指针的函数… 它也会正常工作。

``` {.c .numberLines}
#include <stdio.h>

struct parent {
    int a, b;
};

struct child {
    struct parent super;  // 必须放在第一位
    int c, d;
};

// 将参数设为`void*`，这样我们就可以将任何类型传递进来
// （比如struct parent或struct child）
void print_parent(void *p)
{
    // 预期一个struct parent--但一个struct child 也会正常工作
    // 因为指针指向的是struct child中的第一个field，也就是struct parent：
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
    print_parent(&c);  // 尽管传入的是struct child，但也会正常运行！
}
```

查看我们在`main()`的最后一行做了什么？我们调用了`print_parent()`，但却传递了一个`struct child*`作为参数！尽管`print_parent()`需要参数指向一个`struct parent`，但我们却"糊弄过去"，因为`struct child`中的第一个字段是一个`struct parent`。

再次强调，这是因为`struct`的指针与指向该`struct`中第一个字段的指针具有相同的值。

所有这些都依赖于规范的这一部分：

> **§6.7.2.1¶15** [...] 一个结构体对象的指针，经过适当转换后，指向其初始成员[...]，反之亦然。

以及

> **§6.5¶7** 只能通过以下类型之一的lvalue表达式访问对象的存储值：
>
> * 与对象的有效类型兼容的类型
> * [...]

以及我假设，“适当转换”意味着“转换为初始成员的有效类型”。

## 位字段

在我看来，这些很少被使用，但你可能会时不时地看到它们，特别是在将位打包在更大空间中的低级应用中。

让我们看一段代码以演示一个用例：

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

对我来说，这将打印`16`。这是有道理的，因为在我的系统上，`unsigned`是4个字节。

但是，如果我们知道所有存储在`a`和`b`中的值都可以用5位来存储，而`c`和`d`中的值可以用3位来存储呢？那只需要总共16位。如果我们只使用16位，为什么要为它们保留128位？

``` {.c .numberLines startFrom="3"}
struct foo {
    unsigned int a:5;
    unsigned int b:5;
    unsigned int c:3;
    unsigned int d:3;
};
```

好了，我们可以告诉 C 请尽量尝试这些值进行打包。我们可以指定这些值可以占用的最大位数（从 1 到包含类型的大小为止）。

这可以通过在字段名后面加上冒号，然后是位宽来实现。

现在当我询问 C 我的 `struct foo` 有多大时，它告诉我是 4！原来是 16 字节，但现在只有 4 字节。它已经将这 4 个值压缩至 4 个字节，实现了四倍的内存节省。

当然，权衡之处在于，5 位字段只能存储 0-31 的值，而 3 位字段只能存储 0-7 的值。但生活终究是一场妥协。

### 非相邻位字段

一个注意点：C 只会合并**相邻**的位字段。如果它们被非位字段中断，你就得不到节省：

``` {.c}
struct foo {            // 对我来说，sizeof(struct foo) == 16
    unsigned int a:1;   // 因为 a 不相邻 c。
    unsigned int b;
    unsigned int c:1;
    unsigned int d;
};
```

在这个例子中，由于 `a` 不与 `c` 相邻，它们分别被“打包”在各自的 `int` 中。

因此，`a`、`b`、`c` 和 `d` 各自占用一个 `int`。因为我的 `int` 是 4 字节，总共就是 16 字节。

简单重新排列一下就可以从 16 字节节省到 12 字节（对我来说）：

``` {.c}
struct foo {            // 对我来说，sizeof(struct foo) == 12
    unsigned int a:1;
    unsigned int c:1;
    unsigned int b;
    unsigned int d;
};
```

现在，由于 `a` 与 `c` 相邻，编译器就会将它们合并成一个 `int`。

``` {.c}
// 所以我们有一个`int`来表示合并的`a`和`c`，以及一个`int`分别表示`b`和`d`。总共是3个`int`，或者说12个字节。

// 把所有的位域放在一起，让编译器将它们合并起来。

### 有符号还是无符号的`int`

如果你只声明一个位域为`int`，不同的编译器会将其视为`signed`或`unsigned`。就像`char`的情况一样。

在使用位域时，要明确指定有符号还是无符号。

### 无名位域

在某些特定情况下，您可能需要出于硬件原因保留一些位，但在代码中不需要使用它们。

例如，假设您有一个字节，其中最高的2位有意义，最后的1位有意义，但中间的5位您不使用^[假设是8位`char`s，即`CHAR_BIT == 8`]。

我们可以这样做：

``` {.c}
struct foo {
    unsigned char a:2;
    unsigned char dummy:5;
    unsigned char b:1;
};
```

这样就可以了---在我们的代码中，我们使用`a`和`b`，但从不使用`dummy`。它只是用来占用5位，以确保`a`和`b`在字节内"必需"的位置（按照这个假设的示例）。

C允许我们有一种方法来简化这个过程：_无名位域_。在这种情况下，您可以省略名称（`dummy`），同样在C中会产生相同的效果：

``` {.c}
struct foo {
    unsigned char a:2;
    unsigned char :5;   // <-- 无名位域！
    unsigned char b:1;
};
```

### 零宽度的无名位域

这里还有更多晦涩的部分... 比如说，如果您要把位打包到一个`unsigned int`中，并且需要一些相邻的位域装入下一个`unsigned int`中。

也就是说，如果您这样做：

``` {.c}
struct foo {
    unsigned int a:1;
    unsigned int b:2;
    unsigned int c:3;
    unsigned int d:4;
};
```

编译器将所有这些打包成一个`unsigned int`。但是如果你希望将`a`和`b`放在一个`int`中，而将`c`和`d`放在另一个`int`中呢？

有一个解决方法：在你希望编译器重新开始使用另一个`int`进行位打包的地方，加入一个宽度为`0`的无名位域：

``` {.c}
struct foo {
    unsigned int a:1;
    unsigned int b:2;
    unsigned int :0;   // <--零宽度的无名位域
    unsigned int c:3;
    unsigned int d:4;
};
```

这类似于文字处理器中的显式分页符。你在告诉编译器：“停止在这个`unsigned`中打包位，开始在下一个`unsigned`中打包。”

通过在这个位置添加零宽度的无名位域，编译器将`a`和`b`放在一个`unsigned int`中，将`c`和`d`放在另一个`unsigned int`中。总共两个，使用8字节（在我的系统上`unsigned int`每个为4字节）。

[i[`struct`关键字-->位域]>]

## 联合体

[i[`union`关键字]<]

这些基本上就像`struct`，只是字段在内存中重叠。`union`将只大到可以容纳最大的字段，并且你一次只能使用一个字段。

这是一种将相同内存空间用于不同数据类型的方法。

声明它们就像声明`struct`一样，只是用`union`。看看这个例子：

``` {.c}
union foo {
    int a, b, c, d, e, f;
    float g, h;
    char i, j, k, l;
};
```

现在，这是很多字段。如果这是一个`struct`，我的系统会告诉我需要36字节才能容纳所有数据。

但这是一个`union`，所以所有这些字段重叠在同一块内存中。最大的字段是`int`（或`float`），在我的系统上占用4字节。事实上，如果我询问`union foo`的`sizeof`，它会告诉我是4！

权衡之处在于你在任一时刻只能可移植地使用其中一个字段。然而...

### 联合体和类型转换 {#union-type-punning}

使用`union`，你可以在一个字段中写入值，然后从另一个字段中读取值，但是这样做并不可移植！

这种技术称为[flw[type punning|Type_punning]]，通常在进行一些低级编程时使用，只有当你非常了解自己在做什么的情况下才会使用。

因为联合体的成员共享同一块内存，向一个成员写入值必然会影响其他成员。如果你从一个成员中读取了其他成员中写入的值，就会得到一些奇怪的效果。

``` {.c .numberLines}
#include <stdio.h>

union foo {
    float b;
    short a;
};

int main(void)
{
    union foo x;

    x.b = 3.14159;

    printf("%f\n", x.b);  // 3.14159，可以理解

    printf("%d\n", x.a);  // 但这行呢？
}
```

在我的系统上，这段代码输出：

```
3.141590
4048
```

这是因为在内部，浮点数 `3.14159` 的对象表示和短整数 `4048` 的对象表示实际是一样的。至少在我的系统上是这样。你的系统可能会有不同的结果。

### 指向 `union` 的指针

如果你有一个指向 `union` 的指针，你可以将该指针强制转换为 `union` 中任何字段的类型，然后以此方式获取值。

在这个例子中，我们看到 `union` 中有 `int` 和 `float`。我们获得了指向 `union` 的指针，但我们将它们强制转换为 `int*` 和 `float*` 类型（强制转换可以消除编译器警告）。然后，如果我们对它们进行解引用，我们会发现它们存储了我们直接存储在 `union` 中的值。

```c
x.a = 12;
printf("%d\n", x.a);           // 12
printf("%d\n", *foo_int_p);    // 12, 再次

x.g = 3.141592;
printf("%f\n", x.g);           // 3.141592
printf("%f\n", *foo_float_p);  // 3.141592, 再次
}
```

相反地，如果我们有一个指向`union`内部类型的指针，我们可以将其转换为指向`union`的指针并访问其成员。

```c
union foo x;
int *foo_int_p = (int *)&x;             // 指向int字段的指针
union foo *p = (union foo *)foo_int_p;  // 再次指向union的指针

p->a = 12;  // 这一行和…
x.a = 12;   // 这一行是一样的。
```

所有这些只是让你知道，在幕后，`union`中的所有这些值都从内存中的同一位置开始，这与整个`union`的位置相同。

[i[`union`关键字-->指针到]>]

### `union`中的共同初始序列

[i[`union`关键字-->共同初始序列]<]

如果你有一个`union`的`structs`，并且所有这些`structs`都以_共同的初始序列_开始，那么可以从`union`的任何成员访问该序列的成员是有效的。

什么？

这里有两个具有共同初始序列的`structs`：

```c
struct a {
    int x;     //
    float y;   // 共同的初始序列

    char *p;
};

struct b {
    int x;     //
    float y;   // 共同的初始序列

    double *p;
    short z;
};
```

你看到了吗？它们以`int`开头，然后是`float`---这就是共同的初始序列。`structs`序列中的成员必须是兼容的类型。我们看到`x`和`y`，它们分别是`int`和`float`。

现在让我们建立这些的一个`union`：

```c
union foo {
    struct a sa;
    struct b sb;
};
```

这条规则告诉我们的是，在代码中，共同初始序列的成员是可以互换的。也就是说：

* `f.sa.x` 和 `f.sb.x` 是一样的。

以及

* `f.sa.y` 和 `f.sb.y` 是一样的。

因为字段 `x` 和 `y` 都在共同初始序列里。

此外，共同初始序列中成员的名称并不重要---唯一重要的是类型要相同。

总的来说，这让我们可以安全地在 `union` 中的 `struct` 间添加一些共享信息。其中最好的例子可能是使用一个字段来确定 `union` 中当前“正在使用”的所有 `struct` 中的哪种类型。

也就是说，假如我们没有这个规则，如果我们将 `union` 传递给某个函数，那函数怎么知道应该查看 `union` 的哪个成员呢？

看一下这些 `struct`。注意共同初始序列：

``` {.c .numberLines}
#include <stdio.h>

struct common {
	int type;   // 共同初始序列
};

struct antelope {
	int type;   // 共同初始序列

	int loudness;
};

struct octopus {
	int type;   // 共同初始序列

	int sea_creature;
	float intelligence;
};

```

现在让我们将它们放入一个 `union`：

``` {.c .numberLines startFrom="20"}
union animal {
	struct common common;
	struct antelope antelope;
	struct octopus octopus;
};

```

还有，请容许我加入这两个 `#define` 作为演示：

``` {.c .numberLines startFrom="26"}
#define ANTELOPE 1
#define OCTOPUS  2

```

到目前为止，这里没有发生特别的事情。看起来 `type` 字段完全没用。

但现在让我们编写一个通用函数来打印一个 `union animal`。它必须以某种方式能够知道它正在查看的是一个 `struct antelope` 还是一个 `struct octopus`。

因为有了共同初始序列的魔法, 可以在特定 `union animal x` 的任何地方查找动物类型:

``` {.c}
int type = x.common.type;    \\ 或者...
int type = x.antelope.type;  \\ 或者...
int type = x.octopus.type;
```

所有这些都指向内存中的同一个值。

而且, 你也许已经猜到了, `struct common` 存在是为了让代码可以在不提及特定动物的情况下检查类型。

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

看一下第29行，我们只是传入了 `union`，我们不知道它内部使用了哪种类型的动物 `struct`。

但没关系! 因为在第31行我们检查了类型, 看它是羚羊还是章鱼。然后我们可以查看正确的 `struct` 来获取成员。

通过使用只有 `struct` 的方式也可以实现相同的效果，但如果希望节省内存，就可以这样做。

``` {.c}
struct {
    int x, y;
} s;
```

这定义了一个变量`s`，它是一个无名结构体类型（因为结构体没有名字标签），具有成员`x`和`y`。

所以像这样是有效的：

``` {.c}
s.x = 34;
s.y = 90;

printf("%d %d\n", s.x, s.y);
```

原来你可以像在`union`中一样简单地丢弃无名`struct`：

``` {.c}
union foo {
    struct {       // 无名的！
        int x, y;
    } a;

    struct {       // 无名的！
        int z, w;
    } b;
};
```

然后可以正常访问它们：

``` {.c}
union foo f;

f.a.x = 1;
f.a.y = 2;
f.b.z = 3;
f.b.w = 4;
```

毫无问题！

## 传递和返回`struct`s和`union`s

可以通过值（而不是指针）将`struct`或`union`传递给函数---参数将像通常情况下一样通过赋值方式进行复制。

也可以从函数中返回`struct`或`union`，它也是通过值传递的。

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
    struct foo a = f();  // 复制被制作

    printf("%d %d\n", a.x, a.y);
}
```

有趣的事实：如果这样做，你可以直接在函数调用后使用`.`运算符：

``` {.c .numberLines startFrom="16"}
    printf("%d %d\n", f().x, f().y);
```

（当然，这个示例会两次调用函数，效率不高。）

相同的规则也适用于返回指向`struct`和`union`的指针---在这种情况下要确保使用`->`箭头运算符。

```c
// 将`union`关键字-->传递和返回
// 将`struct`关键字-->传递和返回
// `union`关键字
// `struct`关键字
```