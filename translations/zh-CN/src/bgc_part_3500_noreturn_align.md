<!-- Beej's指南C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 函数修饰符，对齐修饰符/操作符

根据我的经验，这些东西用得不是太多，但为了完整起见，我们还是会介绍它们。

## 函数修饰符

[i[函数修饰符]<]

当你声明一个函数时，可以向编译器提供一些关于函数可能如何或将如何被使用的提示。这样可以激活或鼓励编译器做出一些优化。

### `inline` 用于速度---也许

[i[`inline` 函数修饰符]<]

你可以这样声明函数为inline：

``` {.c}
static inline int add(int x, int y) {
    return x + y;
}
```

这旨在鼓励编译器尽可能地提高该函数调用的速度。从历史上看，一种实现方式是_内联_，这意味着函数的主体会被嵌入到调用的地方。这样可以避免设置函数调用的所有开销，而以函数被复制到各个地方而非被重用的方式来增加代码大小。

似乎这就是故事的结束，但实际上并非如此。`inline` 有一系列规则，使得事情变得_有趣_。我甚至不确定是否完全理解所有规则，而且不同编译器的行为似乎有所不同。

简单回答是，在需要的文件中将 `inline` 函数定义为`static`。然后只在那个文件中使用。这样你就不用担心其余问题了。

但如果你想知道更多，这里有更多的_"有趣时光"_。

让我们尝试去掉 `static`。

``` {.c .numberLines}
#include <stdio.h>

inline int add(int x, int y)
{
    return x + y;
}

int main(void)
{
    printf("%d\n", add(1, 2));
}
```

`gcc` 在 `add()` 上给出一个链接器错误... 除非你使用了优化编译（可能）！

看，编译器可以选择内联或不内联，但如果选择不内联，你将一无所获。`gcc` 不会内联，除非你在进行优化编译。

解决这个问题的一种方法是在其他地方定义一个不是 `inline` 的外部链接版本的函数，当 `inline` 版本没有使用时，将会用到这个。但作为程序员的你不能确定哪个版本是便携的。如果两个都是可用的，不确定编译器会选择哪一个。对于 `gcc`，如果使用优化编译，会使用内联函数，否则会使用非内联函数。即使这两个函数的实现完全不同。疯狂！

另一种方法是将函数声明为 `extern inline`。这将尝试在本文件中进行内联，但同时也会创建一个带有外部链接的版本。 因此，`gcc` 将根据优化使用其中一个，但至少它们是同一个函数。

当然，如果你有另一个带有相同名称的 `inline` 函数的源文件；它将使用其自己的 `inline` 函数或具有外部链接的函数，这取决于优化。

但假设你正在做一个编译器正在内联该函数的编译。在这种情况下，你可以在定义中直接使用普通的 `inline`。但是，现在有额外的限制。

你不能引用任何 `static` 全局变量或调用任何 `static` 函数：

``` {.c}
static int b = 13;

inline int add(int x, int y)
{
    return x + y + b;  // 错误 -- 不能引用 b
}
```

并且你不能定义任何非 `const` 的 `static` 局部变量：

``` {.c}
inline int add(int x, int y)
{
    static int b = 13;  // 错误 -- 不能定义 static

    return x + y + b;
}
```

但是将它设置为`const` 是可以的：

``` {.c}
inline int add(int x, int y)
{
    static const int b = 13;  // 可以 -- 静态常量

    return x + y + b;
}
```

现在，你知道函数默认是`extern`的，因此我们应该能够从另一个文件中调用`add()`。你会希望是这样的，不是吗！

但是你却不能！如果它只是一个普通的`inline`，那么它类似于`static`：只在该文件中可见。

好的，那么如果你在这里加上一个`extern`会怎么样呢？现在我们又回到了之前讨论的将`inline`与具有外部链接的函数混合在一起的情况。

如果两者都可见，编译器可以选择使用哪一个。

让我们演示一下这种行为。我们将有两个文件，`foo.c` 和 `bar.c`。它们都调用`func()`，在`foo.c`中是`inline`函数，而在`bar.c`中是外部链接。

这里是带有`inline`的`foo.c`。

``` {.c .numberLines}
// foo.c

#include <stdio.h>

inline char *func(void)
{
    return "foo's function";
}

int main(void)
{
    printf("foo.c: %s\n", func());

    void bar(void);
    bar();
}
```

请注意，除非我们正在使用`gcc`进行优化构建。否则`func()`将消失，我们将收到链接器错误。当然，除非在其他地方定义了具有外部链接的版本。

而我们有。在`bar.c`中。

``` {.c .numberLines}
// bar.c

#include <stdio.h>

char *func(void)
{
    return "bar's function";
}

void bar(void)
{
    printf("bar.c: %s\n", func());
}

```

所以问题是，输出会是什么？

似乎当我们从`foo.c`调用`func()`时，应该打印"`foo's function`"。还有当从`bar.c`调用`func()`时，`func()`应该打印"`bar's function`"。

而且如果我使用`gcc`进行编译并进行优化^[你可以在命令行上使用`-O`执行此操作。]，它会使用内联函数，我们会得到预期的输出：

``` {.default}
foo.c: foo函数的功能
bar.c: bar函数的功能
```

太棒了！

但是如果我们在没有优化的情况下使用 `gcc` 进行编译，则会忽略内联函数，并使用从 `bar.c` 中的外部链接 `func()`！然后我们会得到这样的结果：

``` {.default}
foo.c: bar函数的功能
bar.c: bar函数的功能
```

简言之，规则非常复杂。我认为我正确描述的概率是30%。

[i[`inline` 函数说明符]>]

### `noreturn` 和 `_Noreturn` {#noreturn}

[i[`noreturn` 函数说明符]<]
[i[`_Noreturn` 函数说明符]<]

这告诉编译器某个特定函数永远不会返回给其调用者，即在函数返回之前程序将通过某种机制退出。

它允许编译器在函数调用周围进行一些优化。

它还允许您告诉其他开发人员，某些程序逻辑依赖于函数_不_返回。

您可能永远不需要使用这个，但在一些库调用中你会看到，比如[fl[`exit()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-exit]] 和 [fl[`abort()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-abort]]。

内建关键字是 `_Noreturn`，但如果它不会破坏您现有的代码，大家都会建议包含 `<stdnoreturn.h>` 并使用更易读的 `noreturn`。

如果指定为 `noreturn` 的函数实际上返回，那将是未定义行为。这是计算上的不诚实。

这里是一个正确使用 `noreturn` 的示例：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <stdnoreturn.h>

noreturn void foo(void) // 这个函数永远不应该返回！
{
    printf("Happy days\n");

    exit(1);            // 它不会返回，它在这里退出！
}

int main(void)
{
    foo();
}
```

如果编译器检测到一个 `noreturn` 函数可能会返回，它可能会给出一个有用的警告。

将 `foo()` 函数替换为以下内容：

``` {.c}
noreturn void foo(void)
{
    printf("违法行为\n");
}
```

会得到一个警告：

``` {.default}
foo.c:7:1: 警告: 声明为 'noreturn' 的函数不应该返回
```

## 对齐说明符和运算符

[i[对齐]<]

[flw[_对齐_|数据结构对齐]] 是关于对象可以存储在哪些地址倍数上的。你可以把它存储在任何地址吗？还是必须是可被 2 整除的起始地址？或者是 8？还是 16？

如果你正在编写像内存分配器这样与操作系统交互的低级程序，你可能需要考虑这一点。大多数开发人员在整个职业生涯中都不会使用C中的这些功能。

### `alignas` 和 `_Alignas`

[i[`alignas` 对齐说明符]<]
[i[`_Alignas` 对齐说明符]<]

这不是一个函数。相反，它是一个您可以在变量声明中使用的_对齐说明符_。

内置的说明符是 `_Alignas`，但头文件 `<stdalign.h>` 将其定义为 `alignas` 以获得更好的外观。

如果你希望你的 `char` 类型和 `int` 类型一样对齐，你可以在声明时像这样强制执行它：

``` {.c}
char alignas(int) c;
```

你也可以传递一个常量值或表达式来指定对齐方式。这必须是系统支持的内容，但规范未规定可以放入其中的值。通常，小的2的幂（1、2、4、8和16）是比较安全的选择。

``` {.c}
char alignas(8) c;   // 在8字节的边界上对齐
```

如果你希望按照系统使用的最大对齐方式进行对齐，
请包含 `<stddef.h>` 并使用类型 `max_align_t`，如下所示：

``` {.c}
char alignas(max_align_t) c;
```

你可以通过指定大于 `max_align_t` 的对齐方式来 _过度对齐_，但是能否允许这样的操作取决于系统。

[i[`alignas` 对齐说明符]>]
[i[`_Alignas` 对齐说明符]>]

### `alignof` 和 `_Alignof`

[i[`alignof` 运算符]<]
[i[`_Alignof` 运算符]<]

这个运算符会返回特定类型在这个系统上用于对齐的地址倍数。例如，也许 `char` 每 1 个地址对齐，而 `int` 每 4 个地址对齐。

内建运算符是 `_Alignof`，但头文件 `<stdalign.h>` 将其定义为 `alignof`，如果你想看起来更酷的话。

下面是一个程序，将打印出各种不同类型的对齐方式。再次强调这些将因系统而异。请注意类型 `max_align_t` 将给出系统使用的最大对齐方式。

``` {.c .numberLines}
#include <stdalign.h>
#include <stdio.h>     // for printf()
#include <stddef.h>    // for max_align_t

struct t {
    int a;
    char b;
    float c;
};

int main(void)
{
    printf("char       : %zu\n", alignof(char));
    printf("short      : %zu\n", alignof(short));
    printf("int        : %zu\n", alignof(int));
    printf("long       : %zu\n", alignof(long));
    printf("long long  : %zu\n", alignof(long long));
    printf("double     : %zu\n", alignof(double));
    printf("long double: %zu\n", alignof(long double));
    printf("struct t   : %zu\n", alignof(struct t));
    printf("max_align_t: %zu\n", alignof(max_align_t));
}
```

在我的系统上输出为：

``` {.default}
char       : 1
short      : 2
int        : 4
long       : 8
long long  : 8
double     : 8
long double: 16
struct t   : 16
max_align_t: 16
```

[i[`alignof` 运算符]>]
[i[`_Alignof` 运算符]>]

## `memalignment()` 函数

[i[`memalignment()` 函数]<]

C23 中的新功能！

(注意：我的编译器都还不支持这个函数，所以代码基本上没经过测试。)

如果你知道数据的类型，`alignof` 非常好用。但是如果你对数据的类型一无所知，只有一个指向数据的指针呢？

这又是怎么发生的呢？

当然是用我们的好朋友 `void*`。我们无法将其传递给 `alignof`，但是如果我们需要知道它所指向的东西的对齐方式呢？

如果我们即将使用这块内存来存储具有重要对齐需求的东西，那么我们可能需要知道这个信息。例如，原子和浮点类型在未对齐时通常表现不佳。

因此，有了这个函数，只要我们有指向数据的指针，甚至是 `void*`，就可以检查数据的对齐方式。

让我们快速测试一下，看看一个 `void*` 指针是否适合作为原子类型使用，并且在适合时，让我们定义一个变量来用作那种类型：

``` {.c}
void foo(void *p)
{
    if (memalignment(p) >= alignof(atomic int)) {
        atomic int *i = p;
        do_things(i);
    } else
        puts("This pointer is no good as an atomic int\n");

...
```

我怀疑你很少（甚至可能从未）需要使用这个函数，除非你在做一些低级别的操作。

[i[`memalignment()` 函数]>]

这就是对齐！ 

[i[对齐]>]