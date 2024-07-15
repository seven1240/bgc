```markdown
<!-- Beej的C指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 函数Specifier进行、对齐Specifier和运算符

就我个人的经验来说，这些在实际中用途不是很大，但为了完整起见，我们会在这里进行介绍。

## 函数Specifier

[函数Specifier]

当你声明一个函数时，你可以给编译器一些关于函数如何或将要被使用的提示。这可以启用或鼓励编译器进行一些优化。

### `inline` 用于提速---或许

[`inline`函数Specifier]

你可以这样声明一个函数为inline：

``` {.c}
static inline int add(int x, int y) {
    return x + y;
}
```

这旨在鼓励编译器尽可能地使这个函数调用变快。并且，一个历史悠久的方法是_内联_，这意味着函数体会完整地被嵌入到调用位置。这将避免设置函数调用和拆卸的所有开销，代价是代码体积更大，因为函数被复制到各处而不是被重复使用。

貌似故事到这里该结束了，但事实并非如此。`inline`还带来一堆规则，使得情况变得_有趣_。我甚至不确定自己是否理解了全部规则，而且不同编译器的行为似乎各不相同。

简单的答案是在需要的文件中将`inline`函数定义为`static`。然后在那个文件中使用它。你就不必担心剩下的问题了。

但如果你想知道更多，这里有更多有趣的事情。

让我们尝试去掉`static`。

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
```

```c
/* `gcc`在`add()`上给出链接器错误... 除非你开启了优化（很可能）！
   看你，编译器可以选择内联或不选择，但如果它选择不内联，你就没有任何函数。GCC不会内联，除非你进行了优化构建。
   一个解决方法是在其他地方定义一个非`inline`外部链接版本的函数，当`inline`版本不被使用时，将会使用这个版本。但你作为程序员无法可移植地确定到底使用哪一个。
   如果两者都可用，编译器选择哪一个是未指定的。在GCC中，如果你使用优化编译，将会使用内联函数，否则会使用非内联函数。即使这两个函数的主体完全不同。古怪！
   另一种方法是将函数声明为`extern inline`。这将尝试在此文件中内联，但也会创建一个具有外部链接的版本。因此，GCC将根据优化使用其中一个，但至少它们是相同的函数。
   当然，除非你有另一个带有相同名称`inline`函数的源文件；它将使用其`inline`函数或具有外部链接的函数，具体取决于优化。
   但假设你正在进行一个编译器正在内联该函数的构建。在这种情况下，你可以在定义中直接使用plain `inline`。然而，现在有额外的限制。
   你不能引用任何`static`全局变量或调用任何`static`函数：

   static int b = 13;

   inline int add(int x, int y)
   {
       return x + y + b;  // 不好 -- 不能引用b
   }
*/

/* 你不能定义任何非`const` `static`局部变量：

   inline int add(int x, int y)
   {
       static int b = 13;  // 不好 -- 不能定义static

       return x + y + b;
   }
*/
```

但将其设为`const`就可以了：

``` {.c}
inline int add(int x, int y)
{
    static const int b = 13;  // OK -- 静态常量

    return x + y + b;
}
```

现在你知道函数默认是`extern`的，所以我们应该可以从另一个文件中调用`add()`。你会觉得，是吧！

但实际上是不行的！如果它只是一个普通的`inline`，它类似于`static`：它只能在当前文件中可见。

好了，那如果你在那儿扔上一个`extern`怎么样？现在我们又回到了之前讨论过将`inline`和具有外部链接的函数混合在一起的问题。

如果两者都可见，编译器可以选择使用哪个。

让我们演示一下这种行为。我们将有两个文件，`foo.c`和`bar.c`。它们都会调用`func()`，`func()`在`foo.c`中是`inline`的，而在`bar.c`中是外部链接的。

这是带有`inline`的`foo.c`。

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

请记住，除非我们使用`gcc`进行优化构建。否则`func()`将消失，而我们将会得到一个链接器错误。除非，在其他地方定义了具有外部链接的版本。

而我们的确在`bar.c`中定义了。

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

看起来当我们从`foo.c`调用`func()`时，它应该打印"`foo's function`"。而在`bar.c`中，`func()`应该打印"`bar's function`"。

如果我使用`gcc`进行优化编译^[你可以通过命令行上的`-O`来做到这一点。]，它将使用内联函数，并且我们会得到预期的结果：

``` {.default}
foo.c: foo的函数
bar.c: bar的函数
```

太棒了！

但是如果我们在`gcc`中没有进行优化编译，它会忽略内联函数，并使用`bar.c`中的外部链接`func()`！结果会是这样的：

``` {.default}
foo.c: bar的函数
bar.c: bar的函数
```

简而言之，这些规则看起来相当复杂。我自认为描述得正确的概率有 30%。

[i[`inline`函数说明符]>]

### `noreturn` 和 `_Noreturn` {#noreturn}

[i[`noreturn`函数说明符]<]
[i[`_Noreturn`函数说明符]<]

这告诉编译器特定函数不会返回到调用者，也就是在函数返回之前程序将通过某种机制退出。

这样做可以让编译器围绕函数调用进行一些优化。

它还可以让你告诉其他开发人员一些程序逻辑取决于函数_不_返回。

你可能永远不需要使用它，但在某些库调用中会看到它，比如
[fl[`exit()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-exit]]
和
[fl[`abort()`|https://beej.us/guide/bgclr/html/split/stdlib.html#man-abort]]。

内置关键字是`_Noreturn`，但如果不影响现有代码，大家都建议包含 `<stdnoreturn.h>`，并使用更易读的`noreturn`。

如果标记为`noreturn`的函数实际上返回，那将是未定义行为。这是计算行为不端，可耻的。

以下是正确使用`noreturn`的示例：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <stdnoreturn.h>

noreturn void foo(void) // 这个函数永远不应该返回！
{
    printf("Happy days\n");

    exit(1);            // 它并不返回--在这里退出了！
}

int main(void)
{
    foo();
}
```

如果编译器发现一个`noreturn`函数可能会返回，它会友善地发出警告。

将`foo()`函数替换为以下内容：

``` {.c}
noreturn void foo(void)
{
    printf("Breakin' the law\n");
}
```

会收到一个警告：

``` {.default}
foo.c:7:1: 警告：声明为'noreturn'的函数不应该返回
```

`noreturn`函数说明符

对齐说明符和运算符

对齐

数据结构的对齐是关于对象可存储的地址的倍数。您可以将其存储在任何地址吗？还是必须是可以被2整除的起始地址？或者是8？或16？

如果您在编写类似内存分配器这样底层的东西，需要与您的操作系统进行交互，您可能需要考虑这一点。大多数开发人员在整个职业生涯中都不会使用C中的这种功能。

`alignas` 和 `_Alignas`

`alignas`对齐说明符

`_Alignas`对齐说明符

这不是一个函数。而是一个您可以与变量声明一起使用的_对齐说明符_。

内置的说明符是`_Alignas`，但头文件`<stdalign.h>`将其定义为`alignas`以获得更好的可读性。

如果你希望你的`char`像`int`一样对齐，你可以在声明时强制执行这个操作：

``` {.c}
char alignas(int) c;
```

您还可以传递一个常量值或表达式以获取对齐。这必须是系统支持的内容，但规范并没有具体规定可以放入其中的值。小的2的幂（1、2、4、8和16）通常是安全保障。

``` {.c}
char alignas(8) c;   // 在8字节边界上对齐
```

如要按照系统中使用的最大对齐方式对齐，请包含`<stddef.h>`并使用`max_align_t`类型，如下所示：

``` {.c}
char alignas(max_align_t) c;
```

您可能通过指定大于`max_align_t`的对齐方式来进行_超对齐_，但是是否允许此类操作取决于系统。

### `alignof` 和 `_Alignof`

[i[`alignof` 运算符]<]
[i[`_Alignof` 运算符]<]

该运算符将返回特定类型在该系统上用于对齐的地址倍数。例如，也许 `char` 每个地址对齐一次，而 `int` 每隔 4 个地址对齐一次。

内置运算符是 `_Alignof`，但头文件 `<stdalign.h>` 将其定义为 `alignof`，如果您想要看起来更酷一些。

以下是一个将打印出各种不同类型的对齐方式的程序。再次强调，这些会因系统而异。请注意，类型`max_align_t`将给出系统使用的最大对齐方式。

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

在我的系统上的输出结果：

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

C23 新增！

（注意：我的编译器均不支持此函数，因此代码基本未经测试。）

`alignof` 对于已知数据类型来说很方便。但是如果对数据类型一无所知，只有数据指针怎么办？

这种情况怎么会发生？

当然是通过我们的老朋友 `void*`。我们无法将其传递给`alignof`，但如果我们需要知道其指向数据的对齐方式呢？

如果我们即将使用内存来做一些需要显著对齐的操作，我们可能会希望知道这一点。例如，原子和浮点类型如果未对齐可能会表现出异常行为。

因此，借助这个函数，只要我们有指向数据的指针，即使是 `void*`，我们就能检查数据的对齐方式。

让我们快速测试一下，看看一个 `void` 指针是否能够被很好地用作原子类型，如果可以的话，让我们获取一个变量将其用作该类型：

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

我猜你很少（甚至可能从未）需要使用此函数，除非你在做一些底层操作。

[i[`memalignment()` 函数]>]

这就是关于对齐的全部内容！

[i[对齐]>]