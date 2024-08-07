<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 信号处理

在我们开始之前，我建议你通常忽略整个章节，并使用你的操作系统（很可能）更优越的信号处理函数。类Unix系统有 `sigaction()` 函数家族，而Windows有... 不知道它有什么[^深层来看，显然它根本不支持类Unix风格的信号，它们被模拟在控制台应用上]。

搞定这个之后，什么是信号？

## 什么是信号？

_信号_ 是在各种外部事件发生时发出的。你的程序可以配置为在 _处理_ 信号时被中断，并且可选择在信号处理完毕后继续之前的工作。

可以将其视为一个在这些外部事件发生时自动调用的函数。

这些事件是什么？在你的系统上可能有很多，但在C规范中只有一些：

|信号|描述|
|-|-|
|`SIGABRT`|异常终止---当调用 `abort()` 时会发生。|
|`SIGFPE`|浮点异常。|
|`SIGILL`|非法指令。|
|`SIGINT`|中断---通常是按下 `CTRL-C` 导致的。|
|`SIGSEGV`|"段错误"：无效的内存访问。|
|`SIGTERM`|终止请求。|

你可以通过使用 `signal()` 函数来设置程序来忽略、处理或允许默认操作处理这些信号。

## 使用 `signal()` 处理信号

`signal()` 调用接受两个参数：所涉及的信号以及在引发该信号时采取的行动。

行动可以是以下三种之一：

* 一个指向处理程序函数的指针。
* `SIG_IGN` 宏用于忽略该信号。
* `SIG_DFL` 宏用于恢复该信号的默认处理程序。

让我们编写一个无法使用 `CTRL-C` 退出的程序。（别担心---在下面的程序中，您也可以按 `RETURN` 键退出。）

处理信号 `SIGINT`。

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

int main(void)
{
    char s[1024];

    signal(SIGINT, SIG_IGN);    // 忽略由 ^C 引起的 SIGINT 信号

    printf("尝试按下 ^C...（按下 RETURN 键退出）\n");

    // 等待输入一行以防止程序立即退出
    fgets(s, sizeof s, stdin);
}
```

请注意第8行---我们告诉程序忽略 `SIGINT`，即当按下 `CTRL-C` 时引发的中断信号。无论您多少次按下它，信号仍将被忽略。如果您注释掉第8行，您会发现可以毫不留情地按下 `CTRL-C` 并立即退出程序。

## 编写信号处理程序

我提到您还可以编写一个处理程序函数，在引发信号时调用该函数。

这些非常简单，当涉及到规范时也非常受限制。

在开始之前，让我们来看看 `signal()` 调用的函数原型：

``` {.c}
void (*signal(int sig, void (*func)(int)))(int);
```

很容易阅读，对吧？但是，不对！让我们花点时间拆开它进行练习。

基本上，我们将传递我们感兴趣捕获的信号编号，以及传递一个形式为：

``` {.c}
void f(int x);
```

的函数指针，这个函数将执行实际的捕获操作。

那剩下的原型部分怎么处理呢？基本上都是返回类型。你看，`signal()`在成功时将返回你作为 `func` 传递的任何内容... 这意味着它返回一个返回 `void` 并以 `int` 为参数的函数指针。

此外，它还可以在出现错误时返回 [i[`SIG_ERR` 宏]] `SIG_ERR`。

让我们做一个示例，使得你需要按 `CTRL-C` 两次才能退出。

我要强调这个程序在某些方面存在未定义行为。但对你来说这个程序可能能正常运行，而且很难编写出可移植的非平凡演示。

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

int count = 0;

void sigint_handler(int signum)
{
    // 编译器可能会执行：
    //
    //   signal(signum, SIG_DFL)
    //
    // 当处理函数被调用时。因此我们在这里重置处理程序：
    signal(SIGINT, sigint_handler);

    (void)signum;   // 消除未使用变量警告

    count++;                       // 未定义行为
    printf("Count: %d\n", count);  // 未定义行为

    if (count == 2) {
        printf("Exiting!\n");      // 未定义行为
        exit(0);
    }
}
```

```plaintext
int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("尝试按下^C...\n");

    for(;;);  // 永远等待在这里
}
```

在第 14 行，你会注意到我们重设了信号处理程序。这是因为 C 语言可以在运行自定义处理程序之前将信号处理程序重置为其 [`SIG_DFL` 宏] `SIG_DFL` 行为。换句话说，它可能是一次性的。所以我们首先重置它，这样可以处理下一个。

在这种情况下，我们忽略了 `signal()` 的返回值。如果我们之前将其设置为另一个处理程序，它将返回指向该处理程序的指针，我们可以这样得到:

``` {.c}
// old_handler 的类型为 "指向接受单个
// int 参数并返回空的函数指针":

void (*old_handler)(int);

old_handler = signal(SIGINT, sigint_handler);
```

[i[`signal()` 函数]>]

话虽如此，我不确定这种用法的常见情况。但是如果出于某种原因你需要旧处理程序，可以通过这种方式获得。

快速说明第 16 行---这只是告诉编译器不要警告我们没有使用这个变量。就好像在说："我知道我没有使用它；你不必警告我。"

最后你会看到我在几个地方标记了未定义行为。在下一节中会详细讨论这个问题。

## 我们实际可以做什么？

事实证明，在我们的信号处理程序中我们可以做和不能做的事情非常有限。这也是为什么我建议你不要费心这个，而是使用操作系统的信号处理机制（例如类Unix系统的 [`sigaction()` 函数] `sigaction()`）。

维基百科甚至说你唯一真正可以做的可移植的事情是调用 `signal()` 与 `SIG_IGN` 或 `SIG_DFL`，仅此而已。

以下是我们**不**可以跨平台做的事情：
```

* 调用任何标准库函数。
  * 就像 `printf()` 一样。
  * 我认为调用可重新启动/可重入函数可能是安全的，但规范不允许这样做。
* 从本地 `static`、文件范围或线程本地变量中获取或设置值。
  * 除非它是无锁原子对象，或者…
  * 你将值赋给 `volatile sig_atomic_t` 类型的变量。

`sig_atomic_t` 类型

最后一部分--`sig_atomic_t`--是你从信号处理程序中获取数据的通行证。 (除非你想使用无锁原子对象，这超出了本节的范围[令人困惑的是，`sig_atomic_t` 比无锁原子早，不是同一回事。]。) 它是一个整型，可能有符号也可能无符号。并且受到你可以放进去的限制。

你可以通过宏 `SIG_ATOMIC_MIN` 和 `SIG_ATOMIC_MAX` 查看允许的最小和最大值[如果 `sig_atomic_t` 是有符号的，范围至少为`-127` 到 `127`。如果是无符号的，至少为 `0` 到 `255`]。

令人困惑的是，规范还说你不能引用"任何具有静态或线程存储期的对象，除非它是无锁原子对象，否则不能通过赋值给声明为 `volatile sig_atomic_t` 的对象[...]"

我理解的是你不能读取或写入任何不是无锁原子对象的东西。而你可以给一个声明为 `volatile sig_atomic_t` 的对象赋值。

但是你能从中读取吗？我想不出为什么不能，除非规范非常明确地提到赋值。但是如果你不得不读取它并依据它做出任何决定，你可能会为一些竞态条件留下可能性。

考虑到这一点，我们可以重新编写我们的“按两次`CTRL-C`退出”代码，使其更具可移植性，尽管在输出上不那么冗长。

让我们将`SIGINT`处理程序更改为什么也不做，只是增加一个`volatile sig_atomic_t`类型的值。这样它就会统计已经按下的`CTRL-C`的次数。

然后在我们的主循环中，我们将检查该计数器是否超过`2`，如果超过就退出。

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

volatile sig_atomic_t count = 0;

void sigint_handler(int signum)
{
    (void)signum;                       // 未使用的变量警告

    signal(SIGINT, sigint_handler);     // 重置信号处理程序

    count++;                            // 未定义行为
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("按两次^C退出。\n");

    while(count < 2);
}
```

[`sig_atomic_t`类型]

再次是未定义行为？这是因为我们必须读取该值才能递增和存储它。

如果我们只想推迟一次按下`CTRL-C`退出，那么不会有太多麻烦。但是要延迟更多次退出将需要一些荒谬的函数链接。

我们将处理一次，在处理程序中将信号重置为其默认行为（即退出）：

[`SIG_DFL`宏]

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

void sigint_handler(int signum)
{
    (void)signum;                       // 未使用的变量警告
    signal(SIGINT, SIG_DFL);            // 重置信号处理程序
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("按两次^C退出。\n");

    while(1);
}
```

[`SIG_DFL`宏]

以后当我们研究无锁原子变量时，我们会看到一种修复 `count` 版本的方法（假设在您的特定系统上有无锁原子变量可用）。

这就是为什么一开始我建议看看您操作系统内置的信号系统作为一个可能更好的选择。

## 朋友们不要让朋友们使用 `signal()`

再次，请使用您的操作系统内置的信号处理或等效功能。虽然它不在规范中，不那么可移植，但可能功能更加强大。此外，您的操作系统可能有很多在 C 规范中没有定义的信号。而且使用 `signal()` 编写可移植代码是困难的。