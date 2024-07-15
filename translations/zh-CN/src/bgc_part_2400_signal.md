<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 信号处理

在开始之前，我建议您通常忽略这整个章节，而使用您的操作系统（很可能）更优越的信号处理函数。类Unix操作系统有`sigaction()`函数家族的函数，而Windows有……无论它是什么[^似乎在深层并不采用Unix风格的信号，而是为控制台应用程序模拟了这种机制]。

说完这个，信号是什么？

## 信号是什么？

_信号_ 是在各种外部事件发生时产生的。您的程序可以配置为在发生事件时被中断以处理该信号，然后选择性地在处理完信号后继续之前的操作。

可以将其想象成某种函数，当其中一个外部事件发生时会自动调用。

这些事件是什么？在您的系统中，可能有很多种，但在C规范中只有几种：

|信号|描述|
|-|-|
|`SIGABRT`|异常终止---当调用`abort()`时发生的情况。|
|`SIGFPE`|浮点异常。|
|`SIGILL`|非法指令。|
|`SIGINT`|中断---通常是按下`CTRL-C`导致的结果。|
|`SIGSEGV`|"分段错误"：无效的内存访问。|
|`SIGTERM`|请求终止。|

您可以通过使用`signal()`函数为您的程序设置忽略、处理或允许默认操作来处理这些信号。

## 使用 `signal()` 处理信号

`signal()` 函数接受两个参数：待处理的信号和处理信号时要采取的操作。

操作可以是以下三种之一：

* 指向处理器函数的指针。
* `SIG_IGN` 宏用于忽略信号。
* `SIG_DFL` 宏用于恢复信号的默认处理器。

让我们写一个无法通过`CTRL-C`退出的程序。(别担心---在下面的程序中，您也可以按`RETURN`键退出。)

`SIGINT` 信号

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

int main(void)
{
    char s[1024];

    signal(SIGINT, SIG_IGN);    // 忽略 SIGINT，由^C触发

    printf("尝试按^C... (按回车键退出)\n");

    // 等待输入一行文本，防止程序立即退出
    fgets(s, sizeof s, stdin);
}
```

看看第8行---我们告诉程序忽略`SIGINT`，即按下`CTRL-C`时产生的中断信号。无论你如何按下，信号都会被忽略。如果注释掉第8行，你会发现可以随心所欲地按`CTRL-C`并立即退出程序。

`SIGINT` 信号
`SIG_IGN` 宏

## 编写信号处理器

我提到您也可以编写一个在触发信号时调用的处理器函数。

这些是相当简单直接的，当涉及到规范时也非常有限。

`signal()` 函数

在我们开始之前，让我们看一下`signal()`调用的函数原型：

``` {.c}
void (*signal(int sig, void (*func)(int)))(int);
```

读起来很简单，对吧？

_错！_ `:)`

让我们花点时间拆解它来进行练习。

`signal()` 接受两个参数：一个表示信号的整数`sig`，和一个指向处理器的指针`func`（处理器返回`void`并以`int`作为参数），如下所示：

``` {.c}
返回值代表我们正在返回一个以int为参数的函数指针，该函数返回void
```

```c
int main(void)
{
    // 我们重置信号处理程序。
    signal(SIGINT, sigint_handler);

    printf("试着按下^C...\n");

    for(;;);  // 永远在这里等待
}
```

[i[`SIG_INT` 信号]>]

其中一个你会注意到的是在第14行，我们重置了信号处理程序。这是因为C语言可以在运行自定义处理程序之前将信号处理程序重置为其[i[`SIG_DFL`宏]] `SIG_DFL`行为。换句话说，它可能是一次性的。所以我们首要重置它，以便为下一个处理它。

在这种情况下，我们忽略了`signal()`的返回值。如果我们之前将其设置为不同的处理程序，它将返回指向该处理程序的指针，我们可以这样获取：

``` {.c}
// old_handler的类型是"接收一个int参数并返回void的函数指针":

void (*old_handler)(int);

old_handler = signal(SIGINT, sigint_handler);
```

[i[`signal()`函数]>]

也就是说，我不确定常见的用例是什么。但如果出于某种原因需要旧处理程序，可以通过这种方式获得。

快速提醒，第16行只是告诉编译器不要警告我们未使用此变量。就像在说，“我知道我没用它；你不必警告我。”

最后，你会看到我在几个地方标记了未定义行为。更多细节请看下一节。

## 我们实际上能做什么？

事实证明，在我们的信号处理程序中我们实际上能做什么和不能做什么是相当有限的。这也是为什么我建议你甚至不要尝试使用这个，而是使用操作系统的信号处理机制（例如，在类Unix系统中使用[i[`sigaction()`函数]] `sigaction()`）。

维基百科甚至认为，你真正可以做的唯一具有可移植性的事情就是将`signal()`与`SIG_IGN`或`SIG_DFL`一起使用。

这是我们**无法**在所有地方都能做的事情：

```c
// 信号处理--->限制

* 调用任何标准库函数。
  * 比如 `printf()`，举个例子。
  * 我觉得调用可重入函数应该是安全的，
     但规范并不允许这样做自由。
* 获取或设置本地 `static`、文件作用域或线程本地变量的值。
  * 除非它是一个无锁原子对象或...
  * 你正在赋值给 `volatile sig_atomic_t` 类型的变量。

// `sig_atomic_t` 类型

那最后一点--`sig_atomic_t`--就是你从信号处理程序中获取数据的通行证。(除非你想使用无锁原子对象，这已超出本节的范围^[令人困惑的是，`sig_atomic_t` 早于无锁原子并不是同一物件。]。) 它是一个整数类型，可能有符号，也可能没有。并且受限于你可以放入其中的内容。

你可以在宏 `SIG_ATOMIC_MIN` 和 `SIG_ATOMIC_MAX` 中查看最小和最大允许的值^[如果 `sig_atomic_t` 是有符号的，范围至少为 `-127` 到 `127`。如果是无符号的，至少为 `0` 到 `255`。]。

令人困惑的是，规范还指出你不能引用“任何静态或线程存储期限的对象，除非它是无锁原子对象，否则除了通过将值分配给声明为 `volatile sig_atomic_t` 的对象外...”

我理解这意味着你不能读取或写入任何不是无锁原子对象的东西。同时你可以向类型为 `volatile sig_atomic_t` 的对象赋值。

但是你能从中读取吗？我并不明白为什么不能，除了规范非常强调提到赋值之外。但是如果你必须读取它并且基于此做出任何决定，可能会为某种竞争条件敞开空间。

// 信号处理--->限制
```

带着这个想法，我们可以重新编写我们的“连续按两次`CTRL-C`退出”的代码，让它更具可移植性，同时在输出上更简洁。

让我们将`SIGINT`处理程序修改为不做任何事情，只是递增一个`volatile sig_atomic_t`类型的值。这样它将统计被按下的`CTRL-C`次数。

然后在我们的主循环中，我们将检查该计数器是否超过`2`，如果超过则退出。

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

volatile sig_atomic_t count = 0;

void sigint_handler(int signum)
{
    (void)signum;                    // 未使用变量警告

    signal(SIGINT, sigint_handler);  // 重置信号处理程序

    count++;                         // 未定义行为
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("按两次^C退出。\n");

    while(count < 2);
}
```

[点击[`sig_atomic_t`类型]>]

再次未定义行为？据我的理解是这样的，因为我们必须读取值才能递增和存储它。

如果我们只想推迟一次按下`CTRL-C`后的退出，那么不会有太多麻烦。但如果想再次延迟退出，就需要一些荒谬的函数链接。

我们要做的是处理一次，处理程序将将信号重置为其默认行为（即退出）：

[点击[`SIG_DFL`宏]<]

``` {.c .numberLines}
#include <stdio.h>
#include <signal.h>

void sigint_handler(int signum)
{
    (void)signum;                      // 未使用变量警告
    signal(SIGINT, SIG_DFL);           // 重置信号处理程序
}

int main(void)
{
    signal(SIGINT, sigint_handler);

    printf("按两次^C退出。\n");

    while(1);
}
```

[点击[`SIG_DFL`宏]>]

```c
// Translate the following content
Later when we look at lock-free atomic variables, we'll see a way to fix the `count` version (assuming lock-free atomic variables are available on your particular system).

This is why at the beginning, I was suggesting checking out your OS's built-in signal system as a probably-superior alternative.

## Friends Don't Let Friends `signal()`

Again, use your OS's built-in signal handling or the equivalent. It's not in the spec, not as portable, but probably is far more capable. Plus your OS probably has a number of signals defined that aren't in the C spec. And it's difficult to write portable code using `signal()` anyway.

[i[`signal()` function]>]
[i[Signal handling]>]
```