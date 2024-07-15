```markdown
<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 多线程

C11正式引入了多线程到C语言中。如果你之前用过[flw[POSIX threads|POSIX_Threads]]，那它和那个有点奇怪地类似。

如果你没用过，别担心。我们会一起讨论。

需要注意的是，我并不打算让这成为一个完整的经典多线程指南^[我更倾向于无共享的方式，对于经典多线程构造我了解有限。]; 对于这个，你可能需要找一本厚书单独学习。抱歉！

[i[`__STDC_NO_THREADS__` 宏]<]

线程是一个可选功能。如果一个C11编译器定义了`__STDC_NO_THREADS__`，那么库中就**没有**线程。为什么他们选择在宏中采用否定的意义，对我来说有点费解，但就是这样。

你可以这样测试它:

``` {.c}
#ifdef __STDC_NO_THREADS__
#error 编译此程序需要线程支持!
#endif
```

[i[`__STDC_NO_THREADS__` 宏]>]

[i[`gcc` 编译器-->带线程]<]

另外，在构建时可能需要指定某些链接器选项。在类Unix系统中，尝试在命令行末尾附加`-lpthreads`以链接`pthreads`库^[是的，`pthreads`中有一个"`p`"。它是POSIX线程的简称，C11在实现线程时大量借鉴了这个库.]:

``` {.zsh}
gcc -std=c11 -o foo foo.c -lpthreads
```

[i[`gcc` 编译器-->带线程]>]

如果你的系统出现链接器错误，可能是因为适当的库未被包含。


## 背景

线程是让你购买的所有那些闪亮CPU核心在同一个程序中为你工作的一种方式。
```

Normally, a C program just runs on a single CPU core. But if you know how to split up the work, you can give pieces of it to a number of threads and have them do the work simultaneously.

通常情况下，C程序只在单个CPU核心上运行。但是，如果你知道如何分割工作，你就可以将其分配给多个线程，并让它们同时执行工作。

Though the spec doesn't say it, on your system it's very likely that C (or the OS at its behest) will attempt to balance the threads over all your CPU cores.

尽管规范没有明确说明，但在您的系统上，C（或操作系统代表它）很可能会尝试在所有CPU核心上平衡线程的分配。

And if you have more threads than cores, that's OK. You just won't realize all those gains if they're all trying to compete for CPU time.

如果你有的线程比核心多，没关系。只是如果它们都在竞争CPU时间，您就无法实现所有这些收益。

## Things You Can Do

You can create a thread. It will begin running the function you specify. The parent thread that spawned it will also continue to run.

您可以创建一个线程。它将开始运行您指定的函数。生成它的父线程也将继续运行。

And you can wait for the thread to complete. This is called _joining_.

您可以等待线程完成。这称为_joining_。

Or if you don't care when the thread completes and don't want to wait, you can _detach it_.

或者如果您不在乎线程何时完成并且不想等待，可以_detach it_。

A thread can explicitly _exit_, or it can implicitly call it quits by returning from its main function.

线程可以显式_exit_，或者通过从其主函数返回来隐式结束。

A thread can also _sleep_ for a period of time, doing nothing while other threads run.

线程还可以_sleep_一段时间，不做任何事情而让其他线程运行。

The `main()` program is a thread, as well.

`main()`程序也是一个线程。

Additionally, we have thread local storage, mutexes, and conditional variables. But more on those later. Let's just look at the basics for now.

此外，我们还有线程局部存储、互斥量和条件变量。但这些稍后再详细讨论。现在让我们只看基础知识。

## Data Races and the Standard Library

[i[Multithreading-->and the standard library]<]

标准库中的一些函数（例如`asctime()`和`strtok()`）返回或使用不是线程安全的`static`数据元素。但一般情况下，除非另有说明，标准库会尽力使其成为线程安全^[根据 §7.1.4¶5条]。

但要保持警惕。如果标准库函数在不影响您拥有的变量的情况下保持状态，或者如果某个函数返回了您没有传入的指针，则它是不是线程安全的。

[i[Multithreading-->and the standard library]>]

## 创建和等待线程

让我们开始写一些代码吧！

我们将创建一些线程（create）并等待它们完成（join）。

然而，首先我们有一点小东西要了解。

每个线程都由一个不透明的变量标识，类型为`thrd_t`。在你的程序中，它是每个线程的唯一标识符。当你创建一个线程时，它会被赋予一个新的ID。

同时，在创建线程时，你必须给它一个指向要运行的函数的指针，以及一个要传递给它的参数的指针（如果没有要传递的内容，则是`NULL`）。

线程将开始执行你指定的函数。

当你想等待一个线程完成时，你必须指定它的线程ID，这样C知道等待哪一个线程。

所以基本的想法是：

1. 编写一个作为线程“主体”（main）的函数。它不是真正的`main()`函数，但类似于它。线程将从那里开始运行。
2. 从主线程中，使用`thrd_create()`函数启动一个新线程，并传递一个指向要运行的函数的指针。
3. 在那个函数中，让线程执行它需要做的事情。
4. 与此同时，主线程可以继续执行它需要做的任何事情。
5. 当主线程决定时，它可以通过调用`thrd_join()`函数等待子线程完成。通常你**必须**要`thrd_join()`这个线程来清理它后面的内存，否则会内存泄漏^[除非你使用`thrd_detach()`。更多内容稍后再说。]

`thrd_create()`接受一个指向要运行的函数的指针，类型为`thrd_start_t`，它是`int (*)(void *)`类型。这句话是说“一个指向接受`void*`参数并返回`int`的函数的指针”。

<i>【`thrd_create()`函数】</i>

<i>【`thrd_start_t`类型】</i>

`thrd_create()`接受一个指向要运行的函数的指针，它是`thrd_start_t`类型，即`int (*)(void *)`。

让我们创建一个线程吧！我们将从主线程中使用`thrd_create()`启动它来运行一个函数，做一些其他事情，然后等待它完成，调用`thrd_join()`。我已经给线程的主函数命名为`run()`，但只要类型匹配`thrd_start_t`，你可以随意命名它。

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

// 这是线程将要运行的函数。它可以随便命名。
//
// arg 是传递给`thrd_create()`的参数指针。
//
// 父线程稍后将从`thrd_join()`中获取返回值。

int run(void *arg)
{
    int *a = arg;  // 我们将从thrd_create()传入一个 int*

    printf("线程：正在运行带参数 %d 的线程\n", *a);

    return 12;  // 要被thrd_join()捡起的值（随机选择了12）
}

int main(void)
{
    thrd_t t;  // t 将保存线程ID
    int arg = 3490;

    printf("启动一个线程\n");

    // 启动一个线程运行 run() 函数，传递指向 3490 的指针作为参数。同时将线程ID存储在 t 中：

    thrd_create(&t, run, &arg);

    printf("在线程运行的同时做其他事情\n");

    printf("等待线程完成...\n");

    int res;  // 保存线程退出的返回值

    // 在这里等待线程完成；将返回值存储在 res 中：

    thrd_join(t, &res);

    printf("线程以返回值 %d 退出\n", res);
}
```

[i[`thrd_start_t` 类型]>]

看看我们是如何调用`thrd_create()`来执行`run()`函数的呢？然后在`main()`中做其他事情，最后停下来等待线程完成并调用`thrd_join()`。

[i[`thrd_create()` 函数]>]
[i[`thrd_join()` 函数]>]

示例输出（你的结果可能会有所不同）:

将线程启动
在线程运行时执行其他操作
等待线程完成...
线程：使用参数 3490 运行线程
线程以返回值 12 退出

传递给函数的 `arg` 必须具有足够长的生命周期，以便线程在其消失之前可以捡起它。另外，在新线程可以使用它之前，主线程不应该覆盖它。

让我们看一个启动 5 个线程的例子。这里需要注意的一点是我们如何使用 `thrd_t` 数组来跟踪所有线程的 ID。

[thrd_create() 函数]
[thrd_join() 函数]

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    int i = *(int*)arg;

    printf("线程 %d：运行中！\n", i);

    return i;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    int i;

    printf("启动线程...\n");
    for (i = 0; i < THREAD_COUNT; i++)

        // 注意！在下面这行中，我们传递一个指向 i 的指针，
        // 但每个线程看到的是相同的指针。因此，它们
        // 在这里主线程中 i 改变值时会打印出奇怪的东西！
        // （请参考下文。）

        thrd_create(t + i, run, &i);

    printf("在线程运行时进行其他操作...\n");
    printf("等待线程完成...\n");

    for (int i = 0; i < THREAD_COUNT; i++) {
        int res;
        thrd_join(t[i], &res);

        printf("线程 %d 完成！\n", res);
    }

    printf("所有线程完成！\n");
}
```

[thrd_join() 函数]

当我运行线程时，我逐渐将 `i` 从 0 计数到 4，并将其指针传递给 `thrd_create()`。该指针最终会传递到 `run()` 例程中，我们在那里对其进行复制。

[thrd_create() 函数]

够简单吧？这是输出：

``` {.default}
启动线程中...
THREAD 2: 运行中！
THREAD 3: 运行中！
THREAD 4: 运行中！
THREAD 2: 运行中！
在线程运行时进行其他操作...
等待线程完成...
线程 2 已完成！
线程 2 已完成！
THREAD 5: 运行中！
线程 3 已完成！
线程 4 已完成！
线程 5 已完成！
所有线程已完成！
```

哇哇哇---？`THREAD 0` 去哪了？为什么会出现 `THREAD 5`，我们调用 `thrd_create()` 时 `i` 显然永远不会大于 `4` 呢？还有两个 `THREAD 2`？疯狂！

这就是踏入了 [i[多线程-->竞态条件]] _竞态条件_ 的乐园。主线程在线程有机会复制 `i` 之前就修改了它。事实上，`i` 在循环结束前就已经达到了 `5`，而最后一个线程还没有机会复制它。

我们必须有一个针对每个线程的变量，这样我们就可以将其作为 `arg` 传递进去。

我们可以拥有一个巨大的数组。或者我们可以使用 `malloc()` 来分配空间（并在某处释放它---也许是在线程本身释放）。

让我们试一试：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

int run(void *arg)
{
    int i = *(int*)arg;  // 复制参数

    free(arg);  // 参数使用完毕

    printf("THREAD %d: 运行中！\n", i);

    return i;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    int i;

    printf("启动线程中...\n");
    for (i = 0; i < THREAD_COUNT; i++) {

        // 为每个线程参数获取一些空间：

        int *arg = malloc(sizeof *arg);
        *arg = i;

        thrd_create(t + i, run, arg);
    }

    // ...
```

请注意，在 27 至 30 行，我们使用 `malloc()` 为一个 `int` 分配了空间，并将 `i` 的值复制到其中。每个新线程都有自己新分配的变量，并我们将指向该变量的指针传递给 `run()` 函数。

一旦`run()`在第7行创建了自己的`arg`副本，它就会释放通过`malloc()`分配的`int`。现在它有了自己的副本，可以随心所欲地处理它。

运行结果如下：

``` {.default}
启动线程...
线程 0：运行中！
线程 1：运行中！
线程 2：运行中！
线程 3：运行中！
在线程运行时进行其他操作...
等待线程完成...
线程 0 完成！
线程 1 完成！
线程 2 完成！
线程 3 完成！
线程 4：运行中！
线程 4 完成！
所有线程完成！
```

这就对了！0-4号线程全部运行中！

你的运行结果可能会有所不同---线程如何被调度运行是超出了C规范范围的。从上面的示例中我们可以看到，直到0-1号线程完成后，4号线程才开始运行。确实，如果我再次运行，很可能会得到不同的输出。我们无法保证线程执行顺序。

## 分离线程

如果你想启动一个线程并且不用担心之后要`thrd_join()`它，你可以使用`thrd_detach()`。

这会删除父线程获取子线程返回值的能力，但如果你并不关心这一点，只是希望线程在完成后能够自己清理，那么这就是正确的做法。

基本上，我们会这样做：

[`thrd_detach()`函数]

``` {.c}
thrd_create(&t, run, NULL);
thrd_detach(t);
```

其中的`thrd_detach()`调用表示父线程在说，“嘿，我不会等待这个子线程完成了再使用`thrd_join()`。所以当子线程完成时，请自行清理。”

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    (void)arg;

    //printf("线程运行中！%lu\n", thrd_current()); // 非可移植！
    printf("线程运行中！\n");

    return 0;
}

#define THREAD_COUNT 10
```

```c
int main(void)
{
    thrd_t t;

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(&t, run, NULL);
        thrd_detach(t);               // <-- 分离！
    }

    // 睡眠1秒，等待所有线程结束
    thrd_sleep(&(struct timespec){.tv_sec=1}, NULL);
}
```

[i[`thrd_detach()`函数]>]

请注意，在这段代码中，我们使用 `thrd_sleep()` 将主线程休眠1秒---稍后详细说明。

另外，在 `run()` 函数中，我有一行被注释掉的代码，会将线程ID以 `unsigned long` 的形式打印出来。这是非便携的，因为规范没有明确指定 `thrd_t` 在底层是什么类型---我们甚至不知道它可能是一个 `struct`。但这行代码在我的系统上是有效的。

当我运行上面的代码并打印线程ID时，我看到了一点有趣的事情，有些线程具有重复的ID！这似乎不可能，但C允许在相应的线程退出后_重用_线程ID。所以我观察到有些线程在其他线程启动之前就完成了运行。

## 线程本地数据

[i[线程本地数据]<]

线程很有趣，因为它们没有自己的内存，除了局部变量。如果你想要一个 `static` 变量或文件作用域变量，所有线程都会看到相同的变量。

这可能会导致竞态条件，导致发生_奇怪的事情_™。

看看这个例子。我们在 `run()` 的块作用域中有一个`static`变量 `foo`。这个变量将对所有通过 `run()` 函数的线程可见。各个线程可能会相互影响。

每个线程将 `foo` 复制到一个本地变量 `x` 中（这不会在线程之间共享---所有线程都有自己的调用堆栈）。所以它们应该是相同的，对吧？
```

之后我们第一次打印它们，它们是^[虽然我觉得不一定需要。只是线程似乎直到发生像`printf()`这样的系统调用之类的事件才会被重新调度…这就是为什么我在那里用了`printf()`。]。然后紧接着，我们检查确保它们仍然相同。

通常它们是相同的。但并非总是！

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

int run(void *arg)
{
    int n = *(int*)arg;  // Thread number for humans to differentiate

    free(arg);

    static int foo = 10;  // Static value shared between threads

    int x = foo;  // Automatic local variable--each thread has its own

    // 我们刚刚将 x 赋值为 foo，所以在这里它们最好是相等的。
    // （在所有的测试运行中，它们是相等的，但甚至这也没有保证！）

    printf("Thread %d: x = %d, foo = %d\n", n, x, foo);

    // 在这里它们应该是相等的，但并非总是！
    // （有时候它们是相等的，有时候它们不是！）

    // 发生的情况是另一个线程此刻进入并递增 foo，
    // 但是这个线程的 x 保持着之前的值！

    if (x != foo) {
        printf("Thread %d: Craziness! x != foo! %d != %d\n", n, x, foo);
    }

    foo++;  // 递增共享值

    return 0;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    for (int i = 0; i < THREAD_COUNT; i++) {
        int *n = malloc(sizeof *n);  // Holds a thread serial number
        *n = i;
        thrd_create(t + i, run, n);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }
}
```

这里是一个输出示例（尽管每次运行都会有所变化）：

``` {.default}
线程 0: x = 10, foo = 10
线程 1: x = 10, foo = 10
线程 1: 疯狂了吧！x != foo！10 != 11
线程 2: x = 12, foo = 12
线程 4: x = 13, foo = 13
线程 3: x = 14, foo = 14
```

在线程 1 中，在两个 `printf()` 之间，`foo` 的值不知何故从 `10` 变为 `11`，明明两个 `printf()` 之间并没有增量操作！

是另一个线程干预进去了（很可能是线程 0，从情况看来），在线程 1 不知情的情况下增加了 `foo` 的值！

让我们用两种不同的方法来解决这个问题。（如果希望所有线程共享变量，并且不相互干扰，你需要继续阅读[mutex](#mutex)部分。）

### `_Thread_local` 存储类 {#thread-local}

[继续[`_Thread_local`存储类]<]

首先，让我们看看简单的解决方案：`_Thread_local` 存储类。

基本上我们只需要在块作用域的 `static` 变量前面加上这个，问题就解决了！它告诉 C 每个线程应该有自己的这个变量的版本，这样它们就不会相互干扰。

[继续[`thread_local`存储类]<]

[`threads.h` 头文件] `<threads.h>` 定义了 `thread_local` 作为 `_Thread_local` 的别名，这样你的代码看起来就不那么难看了。

让我们把之前的例子中的 `foo` 变成一个 `thread_local` 变量，这样我们就不共享这个数据了。

``` {.c .numberLines startFrom="5"}
int run(void *arg)
{
    int n = *(int*)arg;  // 用于区分线程编号

    free(arg);

    thread_local static int foo = 10;  // <-- 不再共享！！
```

运行结果如下：

``` {.default}
线程 0: x = 10, foo = 10
线程 1: x = 10, foo = 10
线程 2: x = 10, foo = 10
线程 4: x = 10, foo = 10
线程 3: x = 10, foo = 10
```

```c
No more weird problems!

一个问题：如果`thread_local`变量是块作用域，**必须**是`static`。就是这样规矩。 （但这没问题，因为非`static`变量已经是每个线程的独立变量了，因为每个线程都有自己的非`static`变量。）

有点谎言在这里：块作用域的`thread_local`变量也可以是`extern`。

### 另一种选择：线程特定存储

线程特定存储（Thread-specific storage，TSS）是另一种实现每个线程数据的方式。

一个额外的特性是，这些函数允许您指定一个析构函数，在删除TSS变量时会调用该函数。通常这个析构函数是`free()`，用于自动清理`malloc()`分配的每个线程数据。或者如果您不需要销毁任何内容，可以设置为`NULL`。

析构函数的类型是[i[`tss_dtor_t`类型]] `tss_dtor_t`，是一个指向返回`void`并接受`void*`作为参数的函数指针（`void*`指向存储在变量中的数据）。换句话说，它是`void (*)(void*)`，如果这样说清楚了就好。尽管我承认这可能并没有解释清楚。请参阅以下示例。

通常，`thread_local`可能是您的首选，但如果您喜欢析构函数的想法，那就可以利用它。
```

这个用法有点奇怪，因为我们需要一个`[i[tss_t 类型]]`类型的变量`tss_t`来表示每个线程的值。然后我们使用`[i[tss_create() 函数]]`来初始化它。最后我们用`[i[tss_delete() 函数]<]`来摆脱它。请注意，调用`tss_delete()`并不会运行所有的析构函数，而是通过`thrd_exit()`（或者从运行函数返回）来实现。`tss_delete()`只是释放了由`tss_create()`分配的内存。[i[`tss_delete() 函数]>]

在中间，线程可以调用`[i[tss_set() 函数]]`和`[i[tss_get() 函数]]`来设置和获取该值。

在下面的代码中，我们在创建线程之前设置了TSS变量，然后在线程之后进行了清理。

在`run()`函数中，线程为一个字符串`malloc()`了一些空间，并将指针存储在TSS变量中。

当线程退出时，析构函数（在这种情况下是`free()`）会被调用来释放_所有_线程的空间。

[i[`tss_t` 类型]<]
[i[`tss_get() 函数]<]
[i[`tss_set() 函数]<]
[i[`tss_create() 函数]<]
[i[`tss_delete() 函数]<]

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

tss_t str;

void some_function(void)
{
    // 获取这个字符串的每个线程值
    char *tss_string = tss_get(str);

    // 然后打印出来
    printf("TSS 字符串：%s\n", tss_string);
}

int run(void *arg)
{
    int serial = *(int*)arg;  // 获取这个线程的序号
    free(arg);

    // 为了保存这个线程的数据而`malloc()`出空间
    char *s = malloc(64);
    sprintf(s, "线程 %d! :)", serial);  // 快乐的小字符串

    // 将这个TSS变量指向该字符串
    tss_set(str, s);

```c
// 调用一个会获取变量的函数
some_function();

return 0;   // 相当于 thrd_exit(0)
}

#define THREAD_COUNT 15

int main(void)
{
    thrd_t t[THREAD_COUNT];

    // 创建一个新的TSS变量，free()函数是析构函数
    tss_create(&str, free);

    for (int i = 0; i < THREAD_COUNT; i++) {
        int *n = malloc(sizeof *n);  // 存放线程序号
        *n = i;
        thrd_create(t + i, run, n);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }

    // 所有线程都完成了，我们就结束这个
    tss_delete(str);
}
```

[i[`tss_t` 类型]>]
[i[`tss_get()` 函数]>]
[i[`tss_set()` 函数]>]
[i[`tss_create()` 函数]>]
[i[`tss_delete()` 函数]>]

再说一遍，这种方式相比`thread_local`来说有点痛苦，所以除非你真的需要那个析构功能，我会建议用后者。

[i[线程专属存储]>]

## 互斥锁 {#mutex}

[i[互斥锁]<]

如果你只想要一个线程在关键代码部分执行，你可以用互斥锁来保护这部分代码^[互斥锁的缩写，也称为“互斥锁”，即在代码部分加锁，让只有一个线程能执行。]。

举个例子，如果我们有一个`static`变量，希望能够在两个操作中获取和设置它而不被其他线程中断并破坏它，我们可以使用互斥锁来实现这一点。

你可以获取一个互斥锁或释放它。如果你尝试获取互斥锁并成功，你可以继续执行。如果尝试但失败（因为其他线程持有），你将会 _阻塞_^[也就是，你的进程会进入睡眠状态。]，直到互斥锁被释放。
```

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    (void)arg;

    static int serial = 0;   // Shared static variable!

    printf("线程运行中！%d\n", serial);  // 线程正在运行，打印共享变量

    serial++;  // 递增共享变量

    return 0;
}

#define THREAD_COUNT 10

int main(void)
{
    thrd_t t[THREAD_COUNT];

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(t + i, run, NULL);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }
}
```

当运行时，得到类似如下的输出：

``` {.default}
线程运行中！0
线程运行中！0
线程运行中！0
线程运行中！3
线程运行中！4
线程运行中！5
线程运行中！6
线程运行中！7
线程运行中！8
线程运行中！9
```

显然，多个线程会同时进入并运行`printf()`，在任何人有机会更新`serial`变量之前。

我们要做的是将获取变量和设置变量包装在一段受互斥锁保护的代码中。

我们会在文件范围内添加一个新变量来表示`mtx_t`类型的互斥锁，初始化它，然后线程可以在`run()`函数中对其进行锁定和解锁。

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

mtx_t serial_mtx;     // <-- 互斥锁变量

int run(void *arg)
{
    (void)arg;

    static int serial = 0;   // 共享静态变量！

    // 获取互斥锁--所有线程将在此调用上阻塞，直到获得锁：

    mtx_lock(&serial_mtx);           // <-- 获取互斥锁

    printf("Thread running! %d\n", serial);

    serial++;

    // 完成数据的获取和设置后，释放锁。这将解除对mtx_lock()调用上的线程阻塞：

    mtx_unlock(&serial_mtx);         // <-- 释放互斥锁

    return 0;
}

#define THREAD_COUNT 10

int main(void)
{
    thrd_t t[THREAD_COUNT];

    // 初始化互斥锁变量，指示这是一个普通的互斥锁：

    mtx_init(&serial_mtx, mtx_plain);        // <-- 创建互斥锁

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(t + i, run, NULL);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }

    // 完成对互斥锁的操作，销毁：

    mtx_destroy(&serial_mtx);                // <-- 销毁互斥锁
}
```

请注意在`main()`的第38行和第50行如何初始化和销毁互斥锁。

```c
// Initialize mutex
// 初始化互斥锁
mtx_init()

// Destroy mutex
// 销毁互斥锁

但是每个单独的线程在第 15 行获取互斥锁，在第 24 行释放它。

在 `mtx_lock()` 和 `mtx_unlock()` 之间是 _临界区_，即我们不希望多个线程同时操纵的代码区域。

// 获取互斥锁函数
mtx_lock()

// 释放互斥锁函数
mtx_unlock()

现在我们得到正确的输出！

Thread running! 0
Thread running! 1
Thread running! 2
Thread running! 3
Thread running! 4
Thread running! 5
Thread running! 6
Thread running! 7
Thread running! 8
Thread running! 9

如果需要多个互斥锁，没问题：只需有多个互斥锁变量。

并且永远记住多个互斥锁的第一准则：_释放互斥锁的顺序与获取互斥锁的顺序相反！_

### 不同类型的互斥锁

互斥锁类型

正如前面暗示的那样，我们可以使用 `mtx_init()` 创建一些互斥锁类型。 (在表中指出，其中一些类型是通过按位或运算得到的。)

互斥锁超时

|类型|描述|
|-|-|
|`mtx_plain`|常规互斥锁|
|`mtx_timed`|支持超时的互斥锁|
|`mtx_plain` `mtx_recursive`|递归互斥锁|
|`mtx_timed` `mtx_recursive`|支持超时的递归互斥锁|

"递归" 意味着持有锁的人可以在同一把锁上多次调用 `mtx_lock()`。 (在其他人可以获取该互斥锁之前，他们必须相同次数地解锁它。) 这可能会从时间到时间地简化编码，特别是如果要在已经持有互斥锁时调用需要锁定互斥锁的函数。
```

在超时的情况下，线程有机会在一段时间内_尝试_获取锁，但如果在这段时间内无法获取锁，则放弃。

对于超时互斥体，请务必使用`mtx_timed`来创建它：

``` {.c}
mtx_init(&serial_mtx, mtx_timed);
```

然后在等待时，您必须指定UTC时间，它将会在那时解锁^[你可能期望它是“从现在开始”的时间，但你只是愿意这样认为，对吧！]。

从`<time.h>`中的`timespec_get()`函数可以提供帮助。它将为您提供`struct timespec`结构体中的当前UTC时间，这正是我们所需要的。事实上，它似乎只是存在为了这个目的。

它有两个字段：`tv_sec`保存自纪元以来的当前时间（秒），`tv_nsec`保存纳秒（秒的十亿分之一）作为“分数”部分。

因此，您可以加载当前时间，然后加上超时时间，来获得特定的超时时间。

然后调用`mtx_timedlock()`而不是`mtx_lock()`。如果返回值为`thrd_timedout`，则表示超时。

``` {.c}
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);  // 获取当前时间
timeout.tv_sec += 1;               // 1秒后超时

int result = mtx_timedlock(&serial_mtx, &timeout));

if (result == thrd_timedout) {
    printf("互斥锁超时！\n");
}
```

除此之外，超时锁与常规锁相同。

## 条件变量

[i[条件变量]<]

```c
Condition Variables are the last piece of the puzzle we need to make
performant multithreaded applications and to compose more complex
multithreaded structures.

A condition variable provides a way for threads to go to sleep until
some event on another thread occurs.

In other words, we might have a number of threads that are rearing to
go, but they have to wait until some event is true before they continue.
Basically they're being told "wait for it!" until they get notified.

And this works hand-in-hand with mutexes since what we're going to wait
on generally depends on the value of some data, and that data generally
needs to be protected by a mutex.

It's important to note that the condition variable itself isn't the
holder of any particular data from our perspective. It's merely the
variable by which C keeps track of the waiting/not-waiting status of a
particular thread or group of threads.

Let's write a contrived program that reads in groups of 5 numbers from
the main thread one at a time. Then, when 5 numbers have been entered,
the child thread wakes up, sums up those 5 numbers, and prints the
result.

The numbers will be stored in a global, shared array, as will the index
into the array of the about-to-be-entered number.

Since these are shared values, we at least have to hide them behind a
mutex for both the main and child threads. (The main will be writing
data to them and the child will be reading data from them.)

But that's not enough. The child thread needs to block ("sleep") until 5
numbers have been read into the array. And then the parent thread needs
to wake up the child thread so it can do its work.
```

```c
// 当它被唤醒时，它需要持有该互斥锁。而且它会！
// 当线程在条件变量上等待时，它也会获取一个互斥锁
// 当它被唤醒时。

所有这些都围绕着一个额外的`cnd_t`类型变量进行[type[`cnd_t`
type]] `cnd_t`操作，这就是_条件变量_。我们使用[i[`cnd_init()` function]] `cnd_init()`函数创建这个变量，在完成后使用[i[`cnd_destroy()` function]] `cnd_destroy()`函数来销毁它。

但这一切是如何工作的呢？让我们看一下子线程将要做的事情的概述：

[i[`mtx_lock()` function]<]
[i[`mtx_unlock()` function]<]
[i[`cnd_wait()` function]<]
[i[`cnd_signal()` function]<]

1. 使用`mtx_lock()`锁定互斥锁
2. 如果我们尚未输入所有数字，使用`cnd_wait()`等待条件变量
3. 执行需要完成的工作
4. 使用`mtx_unlock()`解锁互斥锁

同时，主线程将执行以下操作：

1. 使用`mtx_lock()`锁定互斥锁
2. 将最近读取的数字存储到数组中
3. 如果数组已满，请使用`cnd_signal()`信号唤醒子线程
4. 使用`mtx_unlock()`解锁互斥锁

[i[`mtx_lock()` function]>]
[i[`mtx_unlock()` function]>]

如果你没有忽视得太厉害（没关系---我不会生气），你可能会注意到一些奇怪的地方：主线程如何能持有互斥锁并且通知子线程，如果子线程必须持有互斥锁才能等待信号呢？他们无法同时持有锁！

实际上，他们确实不会同时持有锁！条件变量背后有一些幕后魔法：当你进行`cnd_wait()`时，它会释放你指定的互斥锁，然后线程进入休眠状态。当有人发出信号唤醒该线程时，它会重新获取锁，就好像什么都没有发生过。
```

```c
// 现在在 `cnd_signal()` 方面有点不同。这不会对互斥锁做任何操作。发出信号的线程仍然必须在等待的线程能够唤醒之前手动释放互斥锁。

// 下一个关于 `cnd_wait()` 的事。如果某些条件尚未满足（例如，在这种情况下，如果还没有输入完所有数字），你可能会调用`cnd_wait()`。就是这样：这个条件应该放在一个`while`循环中，而不是一个`if`语句中。为什么呢？

// 这是因为有一个神秘现象，被称为假唤醒（_spurious wakeup_）。有时，在一些实现中，一个线程可能会在看似“没有原因”的情况下被唤醒出`cnd_wait()`的睡眠状态。有时，在一些实现中，一个线程可能会在看似“没有原因”的情况下被唤醒出 `cnd_wait()` 的睡眠状态。有时，在一些实现中，一个线程可能会在看似“没有原因”的情况下被唤醒出`cnd_wait()`的睡眠状态。[我不是说这是外星人……但这可能是外星人。好吧，实际上更可能是另一个线程可能已被唤醒并先于我们开始工作。]。因此，我们必须检查我们醒来时是否仍然实际满足所需的条件。如果不是，那么我们就再次入睡！

// 让我们开始吧！首先是主线程：

* 主线程会设置互斥锁和条件变量，并启动子线程。

* 然后，它会在一个无限循环中从控制台获取输入的数字。

* 它也会获取互斥锁，将输入的数字存储到一个全局数组中。

* 当数组中有5个数字时，主线程将通知子线程到了唤醒并开始工作的时候。

* 然后主线程将释放互斥锁，并继续从控制台中读取下一个数字。

与此同时，子线程一直在搞自己的事情：

* 子线程获取互斥锁
```

* 只要条件未满足(即共享数组中尚未包含5个数字)，子线程会通过等待条件变量而进入休眠状态。当它等待时，会隐式解锁互斥量。

* 一旦主线程发出信号唤醒子线程，子线程会被唤醒以执行工作，并重新获得互斥锁。

* 子线程对数字求和并重置作为数组索引的变量。

* 然后释放互斥锁并再次在无限循环中运行。

下面是代码！仔细研究一下，你就能看到如何处理上述所有部分：

[i[`mtx_lock()` 函数]<]
[i[`mtx_unlock()` 函数]<]
[i[`mtx_init()` 函数]<]
[i[`mtx_destroy()` 函数]<]
[i[`cnd_init()` 函数]<]
[i[`cnd_destroy()` 函数]<]
[i[`cnd_wait()` 函数]<]
[i[`cnd_signal()` 函数]<]
[i[`cnd_t` 类型]<]

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

#define VALUE_COUNT_MAX 5

int value[VALUE_COUNT_MAX];  // 全局共享数组
int value_count = 0;   // 也是全局共享变量

mtx_t value_mtx;   // 数组的互斥锁
cnd_t value_cnd;   // 数组的条件变量

int run(void *arg)
{
    (void)arg;

    for (;;) {
        mtx_lock(&value_mtx);      // <-- 获取互斥锁

        while (value_count < VALUE_COUNT_MAX) {
            printf("线程: 正在等待\n");
            cnd_wait(&value_cnd, &value_mtx);  // <-- 条件等待
        }

        printf("线程: 已唤醒!\n");

        int t = 0;

        // 求和
        for (int i = 0; i < VALUE_COUNT_MAX; i++)
            t += value[i];

        printf("线程: 总和为 %d\n", t);

        // 重置数组索引
        value_count = 0;

        mtx_unlock(&value_mtx);   // <-- 释放互斥锁
    }

    return 0;
}

int main(void)
{
    thrd_t t;
```

```c
// 创建一个新线程

thrd_create(&t, run, NULL);
thrd_detach(t);

// 设置互斥锁和条件变量

mtx_init(&value_mtx, mtx_plain);
cnd_init(&value_cnd);

for (;;) {
    int n;

    scanf("%d", &n);

    mtx_lock(&value_mtx);    // <-- 锁定互斥锁

    value[value_count++] = n;

    if (value_count == VALUE_COUNT_MAX) {
        printf("主线程：通知线程\n");
        cnd_signal(&value_cnd);  // <-- 发送信号给条件变量
    }

    mtx_unlock(&value_mtx);  // <-- 解锁互斥锁
}

// 清理工作（我知道上面是一个无限循环，但是我
// 至少想假装是规范的）：

mtx_destroy(&value_mtx);
cnd_destroy(&value_cnd);
```

这是在生产者-消费者情况下，常见使用条件变量的方式。如果没有方法让子线程在等待满足某些条件时进入睡眠状态，它将被迫轮询，这会极大地浪费CPU。

### 限时条件等待

[i[条件变量-->超时]<]

`cnd_wait()`有一种变体，允许您指定超时时间，以便在时间到达时停止等待。
```

由于子线程必须重新锁定互斥体，这并不一定意味着一旦超时发生就会立即回到运行状态；你仍然必须等待其他线程释放互斥体。

但这意味着你不必等到 `cnd_signal()` 发生。

为使此工作生效，请调用 `cnd_wait()` 的时候使用 [i[`cnd_timedwait()` 函数]`cnd_timedwait()` 函数。如果它返回值 [i[`thrd_timedout` 宏] `thrd_timedout`，则表示已超时。

时间戳是UTC绝对时间，而非距现在的时间。值得庆幸的是 `<time.h>` 中的 [i[`timespec_get()` 函数] `timespec_get()` 函数似乎就是为这种情况量身定制的。

[i[`timespec_get()` 函数]<]
[i[`cnd_timedwait()` 函数]<]
[i[`thrd_timedout()` 宏]<]

``` {.c}
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);  // 获取当前时间
timeout.tv_sec += 1;               // 超时时间为当前时间后1秒

int result = cnd_timedwait(&condition, &mutex, &timeout));

if (result == thrd_timedout) {
    printf("条件变量超时！\n");
}
```

[i[`timespec_get()` 函数]>]
[i[`cnd_timedwait()` 函数]>]
[i[`thrd_timedout()` 宏]>]
[i[条件变量-->超时]>]

### 广播：唤醒所有等待中的线程

[i[条件变量-->广播]<]

`cnd_signal()` 函数只唤醒一个线程继续工作。根据你的逻辑实现方式，唤醒一个以上的线程继续操作一旦条件满足可能更有意义。

当然，其中只有一个线程能够获取互斥体，但如果情况如下：

* 新唤醒的线程负责唤醒下一个线程，以及---

* 有机会虚假唤醒循环条件会阻止它这样做，那么---

```c
// 你会想要广播唤醒，这样你就能确保至少有一个线程跳出循环来启动下一个。
// 你问怎么做？
// 简单使用 `cnd_broadcast()` 而不是 `cnd_signal()`。使用方式完全一样，唯一不同的是 `cnd_broadcast()` 会唤醒**所有**在条件变量上等待的休眠线程。

// [i[`cnd_broadcast()`函数]<]
// [i[条件变量-->广播]>]
// [i[条件变量]>]

## 运行一个函数一次

// [i[多线程-->一次性函数]<]

// 假设你有一个函数，_可能_被多个线程运行，但你不知道何时运行，也没必要尝试编写所有那些逻辑。

// 有一个解决方法：使用 [`call_once()` 函数] `call_once()`。大量线程可能尝试运行该函数，但只有第一个起作用^[适者生存！对吧？我承认实际上完全不是这样的。]

// 为了使用这个，你需要声明一个特殊的标志变量来跟踪该事物是否已被运行。你还需要一个不带参数且不返回任何值的函数来运行。

// [i[`once_flag`类型]<]
// [i[`ONCE_FLAG_INIT`宏]<]
// [i[`call_once()`函数]<]

``` {.c}
once_flag of = ONCE_FLAG_INIT;  // 初始化方式如下

void run_once_function(void)
{
    printf("我将只运行一次！\n");
}

int run(void *arg)
{
    (void)arg;

    call_once(&of, run_once_function);

    // ...
```

// [i[`once_flag`类型]>]
// [i[`ONCE_FLAG_INIT`宏]>]
// [i[`call_once()`函数]>]

在这个示例中，不管有多少个线程到达 `run()` 函数，`run_once_function()` 只会被调用一次。

// [i[多线程-->一次性函数]>]
// [i[多线程]>]
```