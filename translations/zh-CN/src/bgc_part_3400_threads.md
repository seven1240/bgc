<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 多线程

C11 正式引入了多线程到 C 语言中。如果你之前使用过 [flw[POSIX 线程|POSIX_Threads]]，你会发现它非常酷似。

如果你没有使用过，别担心。我们会逐步讲解。

但需要注意的是，我并不打算把这当作一个完整的经典多线程教程^[我更喜欢彻头彻尾的“共享无处不在”，而且我对于经典多线程构造的技能已经相当生疏，可以说是相当生疏了。]; 你可能需要找一本厚厚的专门书籍来学习，抱歉！

[i[`__STDC_NO_THREADS__` 宏]<]

线程是一个可选特性。如果一个 C11 及以上的编译器定义了 `__STDC_NO_THREADS__`，那么线程将 **不** 包含在库中。为什么他们决定在这个宏中使用否定意义，这超出了我的理解范畴，但事实就是如此。

你可以这样测试它：

``` {.c}
#ifdef __STDC_NO_THREADS__
#error I need threads to build this program!
#endif
```

[i[`__STDC_NO_THREADS__` 宏]>]

[i[`gcc` 编译器-->带有线程]<]

此外，在构建时可能需要指定某些链接选项。在类 Unix 系统中，尝试在命令行的结尾添加 `-lpthreads` 来将 `pthreads` 库链接到程序中^[是的，`pthreads` 中的“p”。它是 POSIX 线程的缩写，C11 在实现线程时从这个库中大量借鉴。]：

``` {.zsh}
gcc -std=c11 -o foo foo.c -lpthreads
```

[i[`gcc` 编译器-->带有线程]>]

如果在你的系统上出现链接错误，可能是因为适当的库未被包含。

## 背景

线程是让你购买的那些闪亮 CPU 核心在同一个程序中为你工作的方式。

通常，C程序只在单个CPU核心上运行。但是如果你知道如何拆分工作，你可以将其部分分配给多个线程，并让它们同时进行工作。

尽管规范没有说明，但在你的系统上，很可能C（或在其要求下的操作系统）会尝试平衡线程在所有CPU核心上的分布。

如果线程数多于核心数，也没关系。如果它们都试图竞争CPU时间，你就无法实现所有那些好处。

## 你可以做的事情

你可以创建一个线程。它将开始运行你指定的函数。创建它的父线程也将继续运行。

你可以等待线程完成。这被称为“加入”。

或者如果你不在乎线程何时完成且不想等待，你可以“分离”它。

一个线程可以显式“退出”，或者通过从其主要函数返回隐式退出。

一个线程也可以“睡眠”一段时间，不做任何事情，让其他线程运行。

`main()`程序也是一个线程。

另外，我们有线程本地存储、互斥锁和条件变量。但等会再说。现在让我们只看看基础知识。

## 数据竞争和标准库

标准库中的一些函数（例如`asctime()`和`strtok()`）会返回或使用不是线程安全的`static`数据元素。但一般来说，除非另有说明，标准库会努力确保是线程安全的^[根据 §7.1.4¶5.]。

但要小心。如果标准库函数在你不拥有的变量中保持状态，或者如果一个函数返回了你没有传入的指针，那就不是线程安全的。

## 创建并等待线程

让我们来写一些代码！

我们将创建一些线程，等待它们完成（join）。

不过首先有一点需要理解。

每个线程都由一个类型为`thrd_t`的不透明变量标识。它是程序中每个线程的唯一标识符。当你创建一个线程时，它会被赋予一个新的ID。

另外，当你创建线程时，你必须给它一个指向要运行的函数的指针，以及一个要传递给它的参数的指针（如果没有要传递的内容则为`NULL`）。

线程将开始执行你指定的函数。

当你想要等待一个线程完成时，你必须指定它的线程ID，这样C才知道等待哪一个。

所以基本的思路是：

1. 编写一个充当线程“`main`”的函数。它不是`main()`本身，但类似于它。线程将从那里开始运行。
2. 从主线程中，使用`thrd_create()`函数启动一个新线程，并传递一个指向要运行的函数的指针。
3. 在该函数中，让线程执行必须做的事情。
4. 与此同时，主线程可以继续执行它应该执行的任务。
5. 当主线程决定时，它可以通过调用`thrd_join()`函数等待子线程完成。通常你**必须**`thrd_join()`线程以清理其内存，否则会发生内存泄漏^[除非你使用`thrd_detach()`。稍后我们会详细介绍此功能。]

`thrd_create()`接受一个指向要运行的函数的指针，类型为`thrd_start_t`，即`int (*)(void *)`。这是指向一个函数，该函数以`void*`作为参数，返回一个`int`的指针。

[i[`thrd_create()`函数]<]
[i[`thrd_start_t`类型]<]

`thrd_join()`函数接受一个指向要运行的函数的指针，类型为`thrd_start_t`，即`int (*)(void *)`。这是指向一个以`void*`为参数的函数，并返回一个`int`的指针。

让我们创建一个线程！我们将从主线程中使用`thrd_create()`启动它，让它运行一个函数，然后等待它完成，使用`thrd_join()`。我已经给线程的主函数命名为`run()`，但你可以用任何名称，只要类型与`thrd_start_t`匹配即可。

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

// 这是线程将运行的函数。它可以被命名为任何名称。
//
// arg是传递给`thrd_create()`的参数指针。
//
// 父线程稍后将从`thrd_join()`中获取返回值。

int run(void *arg)
{
    int *a = arg;  // 我们将从thrd_create()传入一个int*

    printf("THREAD: 运行带参数 %d 的线程\n", *a);

    return 12;  // 要被thrd_join()接收的值（随机选择了12）
}

int main(void)
{
    thrd_t t;  // t将保存线程ID
    int arg = 3490;

    printf("启动一个线程\n");

    // 启动一个线程运行run()函数，传入指向3490的指针作为参数。同时在t中存储线程ID：

    thrd_create(&t, run, &arg);

    printf("线程运行时做其他事情\n");

    printf("等待线程完成...\n");

    int res;  // 保存线程退出时的返回值

    // 在这里等待线程完成；将返回值存储在res中：

    thrd_join(t, &res);

    printf("线程退出，返回值为 %d\n", res);
}
```

看到我们是如何用`thrd_create()`调用`run()`函数的吗？然后在`main()`中做其他事情，最后停下来等待线程完成，使用`thrd_join()`。

[`thrd_start_t`类型]

看看我们是如何在`thrd_create()`中调用`run()`函数的？然后在`main()`中做其他事情，最后停下来等待线程完成，使用`thrd_join()`。

[`thrd_create()`函数]
[`thrd_join()`函数]

示例输出（你的可能会有所不同）:

``` {.default}
启动线程
在线程运行时执行其他操作
等待线程完成...
线程：使用参数 3490 运行线程
线程返回值为 12 退出
```

传递给函数的 `arg` 必须具有足够长的生命周期，以便线程在其消失之前捡起它。另外，在新线程可以使用它之前，主线程不应该覆盖它。

让我们来看一个启动 5 个线程的例子。在这里需要注意的一点是我们如何使用一个 `thrd_t` 数组来跟踪所有线程的ID。

在运行多个线程的例子中，我们可能会面对指向同一变量的指针问题。这会导致不同线程在主线程中修改值时打印出奇怪的内容。

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

        // 注意！在下面的这行中，我们传递了一个指向 i 的指针，
        // 但是每个线程看到的是同一个指针。所以当主线程中的 i 
        // 的值变化时，它们会打印出奇怪的内容！（详见下文。）

        thrd_create(t + i, run, &i);

    printf("在线程运行时执行其他操作...\n");
    printf("等待线程完成...\n");

    for (int i = 0; i < THREAD_COUNT; i++) {
        int res;
        thrd_join(t[i], &res);

        printf("线程 %d 完成！\n", res);
    }

    printf("所有线程完成！\n");
}
```

``` {.default}
启动线程...
线程 2: 运行中！
线程 3: 运行中！
线程 4: 运行中！
线程 2: 运行中！
在线程运行时做其他事情...
等待线程完成...
线程 2 完成！
线程 2 完成！
线程 5: 运行中！
线程 3 完成！
线程 4 完成！
线程 5 完成！
所有线程完成！
```

哇啊——？`线程 0` 在哪里？为什么我们有一个 `线程 5` 当我们调用 `thrd_create()` 时明显 `i` 永远不会超过 `4`？而且有两个 `线程 2`？疯狂！

这已经进入了有趣的 [i[多线程-->竞争条件]] _竞争条件_ 领域。主线程在线程有机会复制之前就修改了 `i`。实际上，在最后一个线程有机会复制之前，`i` 已经增加到了 `5` 并结束了循环。

我们必须有一个每个线程都可以引用的线程变量，以便将其作为 `arg` 传递。

我们可以有一个大数组。或者我们可以 `malloc()` 空间（并在某处释放它---也许在线程本身）。

让我们尝试一下：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

int run(void *arg)
{
    int i = *(int*)arg;  // 复制 arg

    free(arg);  // 用完这个了

    printf("线程 %d: 运行中！\n", i);

    return i;
}

#define 线程数 5

int main(void)
{
    thrd_t t[线程数];

    int i;

    printf("启动线程...\n");
    for (i = 0; i < 线程数; i++) {

        // 为每个线程参数获取一些空间：

        int *arg = malloc(sizeof *arg);
        *arg = i;

        thrd_create(t + i, run, arg);
    }

    // ...
```

注意第 27-30 行中我们为一个 `int` 类型的变量 `malloc()` 了空间，并将 `i` 的值复制进去。每个新线程都有自己的刚 `malloc()` 的变量，并且我们将指向它的指针传递给 `run()` 函数。

一旦在第7行上`run()`创建了自己的`arg`副本，它就会`free()`被`malloc()`分配的`int`。现在它有了自己的副本，可以随心所欲地处理它。

运行后显示结果如下:

``` {.default}
启动线程...
THREAD 0: 运行中！
THREAD 1: 运行中！
THREAD 2: 运行中！
THREAD 3: 运行中！
在线程运行时做其他事情...
等待线程完成...
线程 0 完成！
线程 1 完成！
线程 2 完成！
线程 3 完成！
THREAD 4: 运行中！
线程 4 完成！
所有线程完成！
```

这就是结果！线程 0-4 全部生效！

您的运行结果可能会有所不同---线程如何被调度运行超出了 C 规范范围。在以上示例中，我们看到线程 4 甚至直到线程 0-1 完成后才开始运行。实际上，如果我再次运行这个程序，我很可能会得到不同的输出。我们不能保证线程的执行顺序。

## 分离线程

如果您想"发射并忘记"一个线程（即无需稍后`thrd_join()`它），您可以使用`thrd_detach()`来实现。

这会移除父线程从子线程获取返回值的能力，但如果您不在意这一点，只是想让线程在完成后自行清理，这是一个不错的方式。

基本上，我们会这样做:

[i[`thrd_detach()` 函数]<]

``` {.c}
thrd_create(&t, run, NULL);
thrd_detach(t);
```

在这里，`thrd_detach()`调用表示父线程在说："嘿，我不会等待这个子线程与`thrd_join()`完成。所以当它完成后，自行清理吧。"

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

int run(void *arg)
{
    (void)arg;

    //printf("线程运行中！ %lu\n", thrd_current()); // 不可移植的！
    printf("线程运行中！\n");

    return 0;
}

#define THREAD_COUNT 10
```

```
int main(void)
{
    thrd_t t;

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_create(&t, run, NULL);
        thrd_detach(t);               // <-- 分离!
    }
    
    // 睡眠一秒钟，让所有线程完成
    thrd_sleep(&(struct timespec){.tv_sec=1}, NULL);
}
```

请注意，在此代码中，我们使用`thrd_sleep()`将主线程休眠1秒钟，稍后详细讨论。

还有一个`run()`函数中有一行被注释掉的代码，用于打印线程ID为`unsigned long`。这是不可移植的，因为规范没有说明`thrd_t`在底层是什么类型---它可能是一个`struct`。但是在我的系统上，这行代码有效。

当我运行上面的代码并打印出线程ID时，看到一些线程具有重复的ID！这似乎是不可能的，但C语言允许在线程退出后_重新使用_线程ID。所以我看到一些线程在其他线程启动之前完成了运行。

## 线程本地数据

线程很有趣，因为它们除了局部变量外，没有自己的内存。如果您想要一个`static`变量或文件范围变量，所有线程都能看到同一个变量。

这可能导致竞争条件，导致 _奇怪的问题_™ 发生。

看看这个例子。我们在`run()`函数的块范围内有一个`static`变量`foo`。这个变量将对所有通过`run()`函数的线程可见。各个线程可以相互干扰。

每个线程将`foo`复制到一个名为`x`的局部变量中（这个变量不在线程之间共享---所有线程都有自己的调用栈）。所以它们应该是相同的，对吧？

第一次打印它们时，它们通常是^[虽然我认为它们不一定要相等。只是线程似乎不会重新调度直到发生像 `printf()` 这样的系统调用... 这就是为什么我在这里用了 `printf()`。]。但紧接着，我们检查以确保它们仍然相等。

它们通常是相同的。但并不总是！

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

int run(void *arg)
{
    int n = *(int*)arg;  // 用于人类区分的线程编号

    free(arg);

    static int foo = 10;  // 线程间共享的静态值

    int x = foo;  // 自动的局部变量--每个线程有自己的

    // 我们刚刚将 x 赋值为 foo，所以它们在这里应该是相等的。
    // (在我的所有测试运行中，它们是相等的，但这也不是绝对的！)

    printf("线程 %d：x = %d, foo = %d\n", n, x, foo);

    // 在这里它们也应该是相等的，但并非总是！
    // (有时它们相等，有时不相等！)

    // 发生的情况是另一个线程进来并此刻递增 foo，
    // 但此线程的 x 保持之前的值！

    if (x != foo) {
        printf("线程 %d：发狂！x != foo！%d != %d\n", n, x, foo);
    }

    foo++;  // 递增共享值

    return 0;
}

#define THREAD_COUNT 5

int main(void)
{
    thrd_t t[THREAD_COUNT];

    for (int i = 0; i < THREAD_COUNT; i++) {
        int *n = malloc(sizeof *n);  // 保存线程序号
        *n = i;
        thrd_create(t + i, run, n);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }
}
```

这是一个示例输出（尽管这会因每次运行而异）：

``` {.default}
线程 0: x = 10, foo = 10
线程 1: x = 10, foo = 10
线程 1: 疯狂了! x != foo! 10 != 11
线程 2: x = 12, foo = 12
线程 4: x = 13, foo = 13
线程 3: x = 14, foo = 14
```

在线程 1 中，两个 `printf()` 之间，`foo` 的值在没有明显增量的情况下从 `10` 改变为 `11`！

很明显，是另一个线程（可能是线程 0）在线程 1 不知情的情况下递增了 `foo` 的值！

让我们以两种不同的方式解决这个问题。（如果要使所有线程共享变量且不相互干扰，你需要继续阅读 [mutex](#mutex) 部分。）

### `_Thread_local` 存储类 {#thread-local}

首要问题时，我们来看看简单的方法：`_Thread_local` 存储类。

基本上，我们只需要在块作用域的 `static` 变量前面加上这个关键字，问题就解决了！ 它告诉 C 每个线程应该有自己的版本，这样它们就不会互相干扰。

[i[`thread_local` storage class]<]

[i[`threads.h` header file]] `<threads.h>` 定义了 `thread_local` 作为 `_Thread_local` 的别名，这样你的代码看起来就不会那么难看。

让我们将前面的示例中的 `foo` 变量改成 `thread_local` 变量，这样我们就不会共享那些数据了。

``` {.c .numberLines startFrom="5"}
int run(void *arg)
{
    int n = *(int*)arg;  // 用于区分线程编号的变量

    free(arg);

    thread_local static int foo = 10;  // <-- 不再共享！
```

执行后，我们得到：

``` {.default}
线程 0: x = 10, foo = 10
线程 1: x = 10, foo = 10
线程 2: x = 10, foo = 10
线程 4: x = 10, foo = 10
线程 3: x = 10, foo = 10
```

不再有奇怪的问题！

有一点：如果一个 `thread_local` 变量是块作用域，它**必须**是`static`。这就是规定。 (但这没关系，因为非`static` 变量已经是线程特定的，因为每个线程都有自己的非`static` 变量。)

有点小小的谎言：块作用域的 `thread_local` 变量也可以是`extern`。

### 另一种选择：线程特定存储

线程特定存储 (TSS) 是另一种获取每个线程数据的方式。

一个额外的特性是，这些函数允许您指定一个析构函数，当 TSS 变量被删除时将调用该函数。通常这个析构函数是`free()`来自动清理`malloc()`分配的线程特定数据。或者如果你不需要销毁任何东西，那么是`NULL`。

析构函数的类型是 [i[`tss_dtor_t` 类型]] `tss_dtor_t`，它是一个指向返回`void`并接受`void*`参数的函数指针（`void*`指向存储在变量中的数据）。换句话说，它是一个`void (*)(void*)`，如果这样说清楚了的话。我承认这可能并没有。看看下面的例子。

一般来说，`thread_local` 可能是您的首选，但如果你喜欢析构函数的想法，那么您可以利用它。

使用有点奇怪，我们需要一个 `tss_t` 类型的变量 `tss_t`，以便在每个线程基础上表示值。然后我们使用 `tss_create()` 函数对其进行初始化。最后我们用 `tss_delete()` 函数来删除它。请注意，调用` tss_delete()` 不会运行所有析构函数，而是由 `thrd_exit()` 函数（或从运行函数返回）来运行析构函数。`tss_delete()` 只是释放 `tss_create()` 分配的任何内存。

在中间，线程可以调用 `tss_set()` 和 `tss_get()` 函数来设置和获取值。

在以下代码中，在创建线程之前设置了TSS变量，然后在线程结束后清理。

在 `run()` 函数中，线程为一个字符串分配了一些空间，并将指针存储在 TSS 变量中。

当线程退出时，析构函数（在这种情况下为 `free()`）被调用以释放所有线程的资源。

` tss_t` 类型
 `tss_get()` 函数
 `tss_set()` 函数
 `tss_create()` 函数

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <threads.h>

tss_t str;

void some_function(void)
{
    // 获取此字符串的每个线程值
    char *tss_string = tss_get(str);

    // 然后打印它
    printf("TSS字符串：%s\n", tss_string);
}

int run(void *arg)
{
    int serial = *(int*)arg;  // 获取本线程的序列号
    free(arg);

    // malloc() 空间来保存此线程的数据
    char *s = malloc(64);
    sprintf(s, "线程 %d! :)", serial);  // 快乐的小字符串

    // 将此 TSS 变量设置为指向字符串
    tss_set(str, s);
```

// 调用一个会获取变量的函数
some_function();

return 0;   // 等同于 thrd_exit(0)
}

#define THREAD_COUNT 15

int main(void)
{
    thrd_t t[THREAD_COUNT];

    // 创建一个新的TSS变量，free()函数是析构函数
    tss_create(&str, free);

    for (int i = 0; i < THREAD_COUNT; i++) {
        int *n = malloc(sizeof *n);  // 保存线程序号
        *n = i;
        thrd_create(t + i, run, n);
    }

    for (int i = 0; i < THREAD_COUNT; i++) {
        thrd_join(t[i], NULL);
    }

    // 所有线程都执行完毕，可以删除这个TSS变量
    tss_delete(str);
}
```

如果有多个线程在等待互斥锁的释放，会选择其中一个来运行（在我们看来是随机的），而其他线程会继续睡眠。

我们的计划是首先初始化一个互斥锁变量，使其可以通过`mtx_init()`函数进行使用。

然后后续线程可以调用`mtx_lock()`和`mtx_unlock()`函数来获取和释放互斥锁。

当我们完全不再需要这个互斥锁时，可以使用`mtx_destroy()`函数将其销毁，这是`mtx_init()`函数的逻辑相反操作。

首先，让我们看一下一些不使用互斥锁的代码，试图打印一个共享（静态）序号然后对其进行递增。由于我们没有在获取值（打印它）和设置值（递增）之间使用互斥锁，线程可能会在关键部分相互干扰。

当我运行这段代码时，输出看起来像这样:

``` 
Thread running! 0
Thread running! 0
Thread running! 0
Thread running! 3
Thread running! 4
Thread running! 5
Thread running! 6
Thread running! 7
Thread running! 8
Thread running! 9
```

显然有多个线程进入并运行 `printf()`，在任何人有机会更新 `serial` 变量之前。

我们要做的是将获取变量和设置变量包装在一个受互斥锁保护的代码段中。

我们将在文件范围内添加一个新变量来表示 `mtx_t` 类型的互斥锁 `mtx_t`，初始化它，然后线程可以在 `run()` 函数中锁定和解锁它。

看看在 `main()` 的第38行和第50行如何初始化和销毁互斥锁。

初始化 `mtx_init()` 函数
销毁 `mtx_destroy()` 函数

但每个单独的线程在第15行获取互斥锁，在第24行释放它。

在 `mtx_lock()` 和 `mtx_unlock()` 之间是 _关键区域_，即我们不希望多个线程同时干扰的代码区域。

`mtx_lock()` 函数
`mtx_unlock()` 函数

现在我们得到了正确的输出！

如果需要多个互斥锁，没问题：只需要多个互斥锁变量即可。

始终记住多个互斥锁的第一准则：_解锁互斥锁的顺序与锁定它们的顺序相反！_

### 不同的互斥锁类型

正如前面提示的那样，我们有一些可以通过 `mtx_init()` 创建的互斥锁类型。 (其中一些类型是通过按位或操作得到的，如表中所示。)

|类型|描述|
|-|-|
|`mtx_plain` 宏|常规的互斥锁|
|`mtx_timed` 宏|支持超时的互斥锁|
|`mtx_plain mtx_recursive` 宏|递归互斥锁|
|`mtx_timed mtx_recursive` 宏|支持超时的递归互斥锁|

"递归"意味着锁的持有者可以在同一把锁上多次调用 `mtx_lock()`。 (在其他人获取互斥锁之前，他们必须相同次数地解锁它。) 这可能有时会方便编码，特别是当您在已持有互斥锁时调用需要锁定互斥锁的函数时。

超时为线程提供了一次尝试在一段时间内获取锁的机会，但如果在该时间范围内无法获取锁，则放弃。

对于一个带有超时的互斥体，确保使用`mtx_timed`来创建它：

```{.c}
mtx_init(&serial_mtx, mtx_timed);
```

然后当等待它时，您必须指定一个UTC时间，表示在这个时间将解锁^[你可能期望它为“从现在开始”的时间，但你真的只是想这样认为，对吧！]。

在这里，`<time.h>`中的`timespec_get()`函数可以提供帮助。它将为您获取当前的UTC时间，以`struct timespec`结构返回，正是我们所需要的。事实上，它似乎仅仅存在于这个目的。

它有两个字段：`tv_sec`中保存自纪元时以来的当前时间（秒），而`tv_nsec`中的纳秒（十亿分之一秒）作为“小数”部分。

因此，您可以使用当前时间初始化它，然后再添加以获得特定的超时时间。

然后调用`mtx_timedlock()`，而不是`mtx_lock()`。如果它返回值为`thrd_timedout`，则表示超时。

```{.c}
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);  // 获取当前时间
timeout.tv_sec += 1;               // 1秒后超时

int result = mtx_timedlock(&serial_mtx, &timeout);

if (result == thrd_timedout) {
    printf("互斥体锁定超时！\n");
}
```

除此之外，带有超时的锁与常规锁相同。

[互斥体-->超时]
[互斥体-->类型]
[互斥体]

## 条件变量

[条件变量]

条件变量是我们需要的最后一个要素，用于实现高效的多线程应用程序和构建更复杂的多线程结构。

条件变量提供了一种方法，使线程能够在另一个线程上发生某些事件之前进入休眠状态。

换句话说，我们可能有许多准备好继续的线程，但它们必须等到某个事件为真才能继续。基本上它们被告知“等等看！”直到它们收到通知为止。

这与互斥锁紧密合作，因为我们将等待的内容通常取决于某些数据的值，而这些数据通常需要由互斥锁保护。

需要注意的是，从我们的角度来看，条件变量本身并不持有任何特定的数据。它只是C用来跟踪特定线程或一组线程的等待/非等待状态的变量。

让我们编写一个不太真实的程序，从主线程逐个读入5个数字的组。然后，当输入了5个数字后，子线程将唤醒，对这5个数字求和，并打印结果。

这些数字将存储在一个全局共享的数组中，与即将输入数字的数组索引一样。

由于这些是共享值，我们至少必须在主线程和子线程之间的互斥锁后隐藏它们。 （主线程将向其中写入数据，子线程将从中读取数据。）

但这还不够。子线程需要阻塞（“休眠”）直到5个数字被读入数组。然后父线程需要唤醒子线程，以便它可以开始工作。

当线程唤醒时，它需要持有那个互斥锁。它会持有！当一个线程在条件变量上等待时，它在唤醒时也会获取一个互斥锁。

所有这些都围绕着一个额外的[type[`cnd_t`类型]] `cnd_t`变量展开，它是 _条件变量_。我们使用[i[`cnd_init()`函数]] `cnd_init()`函数创建这个变量，在使用完毕后使用[i[`cnd_destroy()`函数]] `cnd_destroy()`函数销毁它。

但是这一切是如何运作的呢？让我们看一下子线程要做的工作大纲：

[i[`mtx_lock()`函数]<]
[i[`mtx_unlock()`函数]<]
[i[`cnd_wait()`函数]<]
[i[`cnd_signal()`函数]<]

1. 用`mtx_lock()`锁定互斥锁
2. 如果我们还没有输入完所有数字，在条件变量上等待，使用`cnd_wait()`
3. 完成需要做的工作
4. 用`mtx_unlock()`解锁互斥锁

同时主线程将作如下操作：

1. 用`mtx_lock()`锁定互斥锁
2. 将最近读取的数字存储到数组中
3. 如果数组已满，用`cnd_signal()`唤醒子线程
4. 用`mtx_unlock()`解锁互斥锁

[i[`mtx_lock()`函数]>]
[i[`mtx_unlock()`函数]>]

如果你没有太草率地浏览（没关系——我不介意），你可能会注意到一些奇怪之处：主线程如何持有互斥锁并唤醒子线程，如果子线程必须持有互斥锁等待信号的话？他们不能同时持有这个锁！

实际上他们并不会！条件变量有一些幕后魔法：当你调用`cnd_wait()`时，它释放你指定的互斥锁，线程进入睡眠状态。当有人唤醒该线程时，它会重新获取锁，就好像什么也没有发生过。

`cnd_signal()` 函数的工作方式有些不同。这个函数不对互斥锁做任何操作。发出信号的线程仍然必须在等待的线程可以唤醒之前手动释放互斥锁。

在 `cnd_wait()` 上还有一件事情。如果还有一些条件未满足（例如在这种情况下，如果尚未输入所有的数字），你可能会调用 `cnd_wait()`。因此它们被称为“条件变量”！这里解释一下：这个条件应该在一个 `while` 循环中，而不是在一个 `if` 语句中。为什么呢？

这是因为有一个神秘的现象被称为 "伪唤醒"。有时候，在某些实现中，一个线程可以在 `cnd_wait()` 睡眠中貌似“毫无理由”地被唤醒。有可能是另一个线程被唤醒并且先开始工作了。因此，我们必须检查在唤醒时我们所需的条件是否仍然满足。如果不满足，我们就继续睡觉！

现在让我们开始这个任务！从主线程开始：

- 主线程会设置互斥锁和条件变量，并启动子线程。

- 然后，它将无限循环从控制台获取输入的数字。

- 它还会获取互斥锁，将输入的数字存储到全局数组中。

- 当数组中有5个数字时，主线程将通知子线程是时候唤醒并开始工作了。

- 然后主线程将解锁互斥锁并继续读取下一个数字从控制台。

与此同时，子线程也在做自己的事情： 

- 子线程获取互斥锁

* 当条件尚未满足（即，共享数组尚未有5个数字时），子线程通过等待条件变量来休眠。当它等待时，它会隐式解锁互斥锁。

* 一旦主线程发信号唤醒子线程，子线程就醒来执行工作并重新获得互斥锁。

* 子线程对数字求和并重置作为数组索引的变量。

* 然后释放互斥锁并在无限循环中再次运行。

以下是代码！好好学习一下，看看所有上述部分是如何处理的：

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>

#define VALUE_COUNT_MAX 5

int value[VALUE_COUNT_MAX];  // 共享全局
int value_count = 0;   // 也是共享全局

mtx_t value_mtx;   // 围绕 value 的互斥锁
cnd_t value_cnd;   // 在 value 上的条件变量

int run(void *arg)
{
    (void)arg;

    for (;;) {
        mtx_lock(&value_mtx);      // <-- 抢占互斥锁

        while (value_count < VALUE_COUNT_MAX) {
            printf("Thread: is waiting\n");
            cnd_wait(&value_cnd, &value_mtx);  // <-- 等待条件
        }

        printf("Thread: is awake!\n");

        int t = 0;

        // 把所有数字相加起来
        for (int i = 0; i < VALUE_COUNT_MAX; i++)
            t += value[i];

        printf("Thread: total is %d\n", t);

        // 重置主线程的输入索引
        value_count = 0;

        mtx_unlock(&value_mtx);   // <-- 解锁互斥锁
    }

    return 0;
}

int main(void)
{
    thrd_t t;
```

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
        printf("主线程：正在通知线程\n");
        cnd_signal(&value_cnd);  // <-- 发送条件信号
    }

    mtx_unlock(&value_mtx);  // <-- 解锁互斥锁
}

// 清理（我知道上面有一个无限循环，但我至少想要假装是正确的）：

mtx_destroy(&value_mtx);
cnd_destroy(&value_cnd);
```

由于子线程必须重新锁定互斥体，这并不一定意味着一旦超时发生你就立刻恢复；你仍然需要等待其他线程释放互斥体。

但这意味着你不会一直等待`cnd_signal()`的发生。

为了使此操作生效，调用`cnd_timedwait()`函数，而不是`cnd_wait()`。如果它返回值为`thrd_timedout`，则表示已超时。

时间戳是绝对的UTC时间，而不是与当前时间的时间差。值得庆幸的是，`<time.h>`中的`timespec_get()`函数似乎恰好是为这种情况量身定制的。

``` {.c}
struct timespec timeout;

timespec_get(&timeout, TIME_UTC);  // 获取当前时间
timeout.tv_sec += 1;               // 1秒后超时

int result = cnd_timedwait(&condition, &mutex, &timeout));

if (result == thrd_timedout) {
    printf("条件变量超时！\n");
}
```

### 广播：唤醒所有等待的线程

`cnd_signal()`函数只唤醒一个线程继续工作。根据你的逻辑实现方式，唤醒多个线程继续一旦条件满足可能是有意义的。

当然只有其中一个可以抢到互斥体，但如果情况是：

* 新唤醒的线程负责唤醒下一个线程，以及---

* 有可能虚假唤醒循环条件会阻止它这样做，那么---

你会想要广播唤醒，这样你就能确保至少有一个线程从循环中退出来启动下一个。

你会问，如何实现？

简单地使用 `cnd_broadcast()` 替代 `cnd_signal()`。用法完全相同，不同之处在于 `cnd_broadcast()` 会唤醒**所有**在该条件变量上等待的睡眠线程。

## 运行一个函数一次

假设你有一个函数可能被多个线程运行，但你不知道什么时候运行，也不值得尝试编写所有的逻辑。

有一种解决方法：使用 `call_once()`。大量的线程可以尝试运行该函数，但只有第一个才起作用。^[适者生存！对吧？我承认实际上并不完全是这样。]

为了使用它，你需要声明一个特殊的标志变量来跟踪事务是否已经运行。你还需要一个不带参数且不返回值的函数来运行。

``` {.c}
once_flag of = ONCE_FLAG_INIT;  // 初始化它

void run_once_function(void)
{
    printf("我只会运行一次！\n");
}

int run(void *arg)
{
    (void)arg;

    call_once(&of, run_once_function);

    // ...
```

在这个例子中，无论有多少线程到达 `run()` 函数，`run_once_function()` 仅会被调用一次。