<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 原子操作 {#chapter-atomics}

> _“他们尝试了都失败了吗？”_ \
> _“哦，不。”她摇了摇头。“他们尝试了就死了。”_
>
> --- 保罗·亚特雷德斯和蓋阿斯·海倫·莫海姆, 《沙丘》

[i[原子变量]<]

这是在C语言中使用多线程时比较具挑战性的一部分。
但我们会尽量轻松地来讲解。

基本上，我会谈谈原子变量更直接的用法，它们是什么，如何工作等等。我还会提及一些对你开放的极其复杂的路径。

但我不会深入探讨那些路径。不仅仅是我几乎没有资格来写这些内容，而且我觉得如果你知道你需要它们，那你的了解比我多。

但即使在基础知识中也有一些奇怪的事情。所以大家系好安全带，因为堪萨斯要消失啦。

## 测试原子操作支持

[i[`__STDC_NO_ATOMICS__` 宏]<]

原子操作是一个可选的特性。有一个宏 `__STDC_NO_ATOMICS__`，如果你 _没有_ 原子操作，它的值为 `1`。

在C11之前可能不存在这个宏，所以我们应该使用 `__STDC_VERSION__` 来测试语言版本^[早期的C89没有 `__STDC_VERSION__` 宏，如果你关心这一点，请用 `#ifdef` 进行检查。]。

``` {.c}
#if __STDC_VERSION__ < 201112L || __STDC_NO_ATOMICS__ == 1
#define HAS_ATOMICS 0
#else
#define HAS_ATOMICS 1
#endif
```

[i[`__STDC_NO_ATOMICS__` 宏]>]

[i[原子变量-->编译相关]<]

如果这些测试通过，那么你可以安全地包含 `<stdatomic.h>`，这是本章其余内容基于的头文件。但如果没有原子操作支持，那么这个头文件甚至可能不存在。

在某些系统上，您可能需要在编译命令行的末尾添加`-latomic`才能使用标头文件中的任何函数。

## 原子变量

以下是原子变量工作原理的部分描述：

如果您有一个共享的原子变量，并且您从一个线程写入它，那么在另一个线程中，写入将是`一刀切`的。

也就是说，另一个线程将看到整个 32 位值的写入。不是其中的一半。一个线程没有办法在另一个线程在原子多字节写入的`中间`打断它。

这几乎就像在获取和设置那个变量时有一把小锁。（或许真的有！请参阅下文的[无锁原子变量](#lock-free-atomic)。）

顺便说一句，如果您使用互斥锁来锁定关键部分，您可以摆脱根本不使用原子操作。只是有一类`无锁数据结构`，它们始终允许其他线程取得进展，而不受互斥锁阻塞… 但要正确地从头开始创建这些是相当困难的，并且是超出本指南范围的事情，可惜。

这只是故事的一部分。但这是我们将从中开始的部分。

在我们进一步之前，您如何声明变量为原子变量？

首先，包含[`stdatomic.h` 标头]`<stdatomic.h>`。

[`atomic_int` 类型]

这样我们就可以使用诸如`atomic_int`这样的类型。

然后我们只需声明变量为该类型即可。

但让我们演示一下，我们有两个线程。第一个线程运行一段时间，然后将一个变量设置为特定值，然后退出。另一个线程运行，直到看到该值被设置，然后退出。

``` {.c .numberLines}
#include <stdio.h>
#include <threads.h>
#include <stdatomic.h>

```c
atomic_int x; // 原子强！哈哈哈！

int thread1(void *arg)
{
    (void)arg;

    printf("线程 1：睡眠 1.5 秒\n");
    thrd_sleep(&(struct timespec){.tv_sec=1, .tv_nsec=500000000}, NULL);

    printf("线程 1：将 x 设置为 3490\n");
    x = 3490;

    printf("线程 1：退出\n");
    return 0;
}

int thread2(void *arg)
{
    (void)arg;

    printf("线程 2：等待 3490\n");
    while (x != 3490) {}  // 在这里忙等

    printf("线程 2：已获得 3490，退出！\n");
    return 0;
}

int main(void)
{
    x = 0;

    thrd_t t1, t2;

    thrd_create(&t1, thread1, NULL);
    thrd_create(&t2, thread2, NULL);

    thrd_join(t1, NULL);
    thrd_join(t2, NULL);

    printf("主程序：线程已完成，x 应当为 3490\n");
    printf("主程序：实际上，x 的值为：%d\n", x);
}
```

你可能在想，如果是普通非原子`int`会发生什么。在我的系统上，它仍然可以工作…除非我进行了优化编译，否则它会在线程2上挂起，等待看到3490被设置。[这是因为在优化时，我的编译器将`x`的值放入一个寄存器中，以加快`while`循环的速度。但是这个寄存器无法知道变量在另一个线程中被更新，因此它永远不会看到`3490`。这与原子性的“全不全”部分实际上没有关系，而更多地与下一部分的同步机制有关]。

但这只是故事的开始。接下来将需要更多的脑力，并与一种称为“同步”的东西有关。

## 同步

[原子变量-->同步机制]

我们故事的下一部分将重点讲述一个线程中的某些内存写入何时对另一个线程可见的问题。

你可能会认为，立即可见，对吧？但事实并非如此。许多事情可能出错。非常奇怪的错。

编译器可能会重新排列内存访问，以至于当你认为相对于另一个设置值时，这可能并不成立。即使编译器没有这样做，你的CPU也可能在运行时这样做。或者可能是有关这种架构的其他问题导致一个CPU上的写入在在另一个CPU上可见之前被延迟。

好消息是，我们可以将所有这些潜在问题归结为一个问题：未同步的内存访问在哪个线程观察到取决于哪个线程，就像代码行本身被重新排列一样。

举个例子，在以下代码中，`x`的写入和`y`的写入哪个先执行？

``` {.c .numberLines}
int x, y;  // 全局

// ...

x = 2;
y = 3;

```c
printf("%d %d\n", x, y);
```

回答：我们不知道。编译器或CPU可能会悄悄地交换第5行和第6行，而我们却对此一无所知。代码将以单线程方式运行，就好像按代码顺序执行一样。

在多线程场景中，我们可能会有类似以下伪代码：

``` {.c .numberLines}
int x = 0, y = 0;

thread1() {
    x = 2;
    y = 3;
}

thread2() {
    while (y != 3) {}  // 自旋
    printf("x is now %d\n", x);  // 2? ...还是0?
}
```

线程2的输出是什么？

嗯，如果 `x` 在 `y` 被赋值为 `3` _之前_ 被赋值为 `2`，那么我会期望输出非常明智：

``` {.default}
x is now 2 
```

但是，如果有什么诡计使得第4行和第5行顺序发生变化，导致我们在打印时看到 `x` 的值为 `0`。

换句话说，除非我们可以在某种程度上说：“在此时此刻，我希望另一个线程中所有先前的写操作在这个线程中可见。”

两个线程在共享内存状态达成一致时会进行_同步。正如我们所见，它们并不总是与代码一致。那么它们如何达成一致呢？

使用原子变量可以强制达成一致^[在我另有说明之前，我一直在谈论_顺序一致_操作。很快会详细说明这是什么意思。]。如果一个线程写入原子变量，就是在表明“将来任何读取此原子变量的线程也将看到我对内存所做的所有更改（无论是否原子），包括对原子变量进行的更改”。

换言之，在更人性化的说法是，让我们围坐在会议桌前，确保我们对共享内存中哪些内存区域保存着什么值达成一致。你同意，你对内存所做的更改将会在我对相同原子变量进行加载后对我可见。

所以我们可以轻松修正我们的示例：

``` {.c .numberLines}
int x = 0;
atomic int y = 0;  // 让 y 成为原子变量

thread1() {
    x = 2;
    y = 3;             // 写入时同步
}

thread2() {
    while (y != 3) {}  // 读取时同步
    printf("x is now %d\n", x);  // 2，没错。
```

因为线程在`y`上同步，所以在线程 2 中，在从`y`读取之后，发生在线程 1 中写入`y`之前的所有写入都是可见的（在`while`循环中）。

这里需要注意几点：

1. 没有线程休眠。同步不是阻塞操作。两个线程一直全速执行直至退出。即使被卡在自旋循环中的线程也不会阻塞其他线程的运行。

2. 同步发生在一个线程读取另一个线程写入的原子变量时。所以当线程 2 读取`y`时，线程 1 中所有之前的内存写入（即设置`x`）将会对线程 2 可见。

3. 注意`x`不是原子的。这没关系，因为我们不是在`x`上同步，而是在线程 1 写入`y`时同步。这意味着线程 1 中之前的所有写入（包括`x`）都将对其它线程可见... 如果那些其他线程读取`y`以进行同步。

强制这种同步效率低下，比起使用普通变量要慢得多。这就是为什么除非特定应用需要，我们才会使用原子操作。

这就是基础知识，让我们深入了解。

[i[原子变量-->同步]<]

## 获取和释放

[i[原子变量-->获取]<]
[i[原子变量-->释放]<]

更多术语！现在学习将会受益匪浅。

当线程读取原子变量时，它被称为一个_获取_操作。

```c
/*
When a thread writes an atomic variable, it is said to be a _release_
operation.
*/

当线程写入原子变量时，这被称为 _release_ 操作。

/*
What are these? Let's line them up with terms you already know when it
comes to atomic variables:
*/

这些是什么？让我们将它们与您对原子变量的已知术语对齐：

**读取 = 加载 = 获取**。就像当您比较原子变量或读取它以将其复制到另一个值时。

**写入 = 存储 = 释放**。就像当您将一个值分配给原子变量时。

当使用具有这些获得/释放语义的原子变量时，C明确定义了可能发生的情况。

获得/释放构成了我们刚刚讨论的同步的基础。

当线程获取一个原子变量时，它可以看到另一个线程设置的值，后者释放了相同的变量。

换句话说：

当线程读取一个原子变量时，它可以看到另一个线程写入到相同变量的值。

同步发生在获取/释放对之间。

更多细节：

对特定原子变量的读取/加载/获取：

- 所有写入（原子或非原子）发生在另一个线程中，在该线程写入/存储/释放该原子变量之前，现在在此线程中可见。

- 其他线程设置的原子变量的新值也在此线程中可见。

- 不会对当前线程中任何变量/内存的读取或写入重新排序，使其在此获取之前发生。

- 当涉及到代码重新排序时，获取行为就像是一个单向障碍；当前线程中的读取和写入可以从获取之前移动到获取之后。但是，更重要的是对于同步，任何内容都不能从获取之后移动到获取之前。

对特定原子变量的写入/存储/释放：
```

* 当前线程中的所有写操作（无论是原子的还是非原子的），在此释放操作之前发生的，将对已经读取/加载/获取同一原子变量的其他线程可见。

* 此线程写入的值对其他线程也是可见的。

* 当前线程中任何变量/内存的读写都不能在此释放操作之后重新排序。

* 在代码重排序方面，释放操作 acts as a one-way barrier: 当前线程中的读写可以从释放操作后移到之前。但更重要的是对于同步，任何操作都不能从释放操作前移到释放操作后。

总之，这是将内存从一个线程同步到另一个线程。第二个线程可以确保变量和内存按照程序员的意图顺序写入。

```
int x, y, z = 0;
atomic_int a = 0;

thread1() {
    x = 10;
    y = 20;
    a = 999;  // 释放操作
    z = 30;
}

thread2()
{
    while (a != 999) { } // 获取操作

    assert(x == 10);  // 从不触发，x始终为10
    assert(y == 20);  // 从不触发，y始终为20

    assert(z == 0);  // 可能触发!!
}
```

在上述示例中，`thread2` 在获取 `a` 后可以确保 `x` 和 `y` 的值，因为它们在 `thread1` 释放原子变量 `a` 之前设置。

但是 `thread2` 无法确定 `z` 的值，因为它发生在释放操作之后。也许将 `z` 的赋值操作移动到了 `a` 的赋值操作之前。

一个重要的注意：释放一个原子变量对不同原子变量的获取操作没有影响。每个变量都与其他变量隔离。

[i[原子变量-->获取操作]>]
[i[原子变量-->释放操作]>]

## 顺序一致性

[i[原子变量-->顺序一致性]<]

```c
// Translate the commented part only
#include <stdatomic.h>

int main() {
    atomic_int data = 0;

    // 你还好吗？我们已经搞定了关于原子操作更简单用法的要点。既然我们不准备在这里讨论更复杂的用法，你可以稍微放松一下。

    // **Sequential consistency** 是所谓的 **memory ordering**。存在许多内存排序方式，但从程序员的角度来看，**sequential consistency** 是 C 语言提供的最合理的内存排序方式。它也是默认的。如果要使用其他内存排序方式，你需要特意去做。

    // 我们迄今为止讨论的所有内容都发生在 **sequential consistency** 的领域内。

    // 我们已经谈论了编译器或 CPU 如何在单个线程中重新排列内存读取和写入，只要遵循 **as-if** 规则。

    // 我们已经看到如何通过对原子变量进行同步来阻止这种行为。

    // 让我们再更正式地形式化一点。

    // 如果操作是 _sequentially consistent_，那意味着到头来，当所有事情都结束时，所有线程都可以悠闲地踱步，拿起他们选择的饮料，达成一致，就内存变化发生的顺序而言。而且这个顺序是由代码指定的。

    // 如果其他人表示“_A_ 绝对在 _B_ 之前发生”，那么没有人会说，“但是 _B_ 不是在 _A_ 之前发生的吗？”他们在这里都是朋友。

    // 特别地，在一个线程内，任何获得和释放都不会被彼此重新排序。这是除了其他内存访问可以在它们周围重新排序的规则之外的另一层保障。

    // 这个规则为原子读取 / 获得和存储 / 释放的进展提供了额外的合理性级别。

    return 0;
}
```

每个C语言中的其他内存顺序都涉及重新排序规则的放松，无论是对获取/释放还是其他内存访问，原子或非原子的。如果你_真地_知道自己在做什么并且需要速度提升，那你可以这样做。_这里有一群龙..._

稍后再详细讨论，但现在让我们专注于安全和实用。

## 原子变量和顺序一致性

[i[原子变量-->顺序一致性]>]

## 原子赋值和操作符

[i[原子变量-->赋值和操作符]<]

原子变量上的某些操作符是原子的。而其他则不是。

让我们从一个反例开始：

``` {.c}
atomic_int x = 0;

thread1() {
    x = x + 3;  // 非原子操作！
}
```

由于赋值语句右侧有一个`x`的读操作，左侧实际上有一个写操作，这是两个操作。另一个线程可能会在中间插入，让你感到不开心。

但你_可以_使用`+=`简写来获得原子操作：

``` {.c}
atomic_int x = 0;

thread1() {
    x += 3;   // 原子操作！
}
```

在这种情况下，`x`将以`3`为单位原子递增---没有其他线程可以在中途插入。

特别地，以下操作符是具有顺序一致性的原子读-修改-写操作，所以可以放心大胆地使用它们。（在示例中，`a`是原子的。）

``` {.c}
a++       a--       --a       ++a
a += b    a -= b    a *= b    a /= b    a %= b
a &= b    a |= b    a ^= b    a >>= b   a <<= b
```

[i[原子变量-->赋值和操作符]>]

## 自动同步的库函数

[i[原子变量-->同步库函数]<]

到目前为止，我们已经讨论了如何使用原子变量进行同步，但事实证明有一些库函数可以在幕后执行一些有限的同步。

``` {.c}
call_once()      thrd_create()       thrd_join()
mtx_lock()       mtx_timedlock()     mtx_trylock()
malloc()         calloc()            realloc()
aligned_alloc()
```

**`call_once()`**---为特定标志同步所有后续对`call_once()`的调用。这样，后续的调用可以放心，如果另一个线程设置了该标志，它们将看到它。

**`thrd_create()`**---与新线程的开始同步。新线程可以确保它将看到父线程在`thrd_create()`调用之前的所有共享内存写入。

**`thrd_join()`**---当一个线程终止时，它与此函数同步。调用`thrd_join()`的线程可以确保它可以看到所有晚于线程的共享写入。

**`mtx_lock()`**---在同一互斥锁上对`mtx_unlock()`的早期调用与此调用同步。这是与我们已经讨论过的获取/释放过程最相似的情况。`mtx_unlock()`对互斥变量执行释放，确保任何随后使用`mtx_lock()`进行获取的线程可以看到临界区中的所有共享内存更改。

**`mtx_timedlock()`**和**`mtx_trylock()`**---类似于`mtx_lock()`的情况，如果此调用成功，较早的对`mtx_unlock()`的调用将与此调用同步。

**动态内存函数**：如果您分配了内存，它将与同一内存的先前释放同步。对该特定内存区域的分配和释放发生在所有线程都同意的单个总顺序中。我认为这里的想法是，如果选择释放该区域，释放可以擦除该区域，我们希望确保后续的分配不会看到未擦除的数据。如果有更多内容请告诉我。
```

```c
// 原子变量-->同步库函数
武装太严肃的话题，我们来看看有哪些可用的类型，以及如何创建新的原子类型。

// `_Atomic` 类型限定符
首先让我们看看内置的原子类型及其`typedef`的内容。（剧透：`_Atomic`是一个类型限定符！）
```

```c
// 以下内容是需要翻译的
|Atomic type|Longhand equivalent|
|-|-|
|[i[`atomic_bool` type]]`atomic_bool`|`_Atomic _Bool`|
|[i[`atomic_char` type]]`atomic_char`|`_Atomic char`|
|[i[`atomic_schar` type]]`atomic_schar`|`_Atomic signed char`|
|[i[`atomic_uchar` type]]`atomic_uchar`|`_Atomic unsigned char`|
|[i[`atomic_short` type]]`atomic_short`|`_Atomic short`|
|[i[`atomic_ushort` type]]`atomic_ushort`|`_Atomic unsigned short`|
|[i[`atomic_int` type]]`atomic_int`|`_Atomic int`|
|[i[`atomic_uint` type]]`atomic_uint`|`_Atomic unsigned int`|
|[i[`atomic_long` type]]`atomic_long`|`_Atomic long`|
|[i[`atomic_ulong` type]]`atomic_ulong`|`_Atomic unsigned long`|
|[i[`atomic_llong` type]]`atomic_llong`|`_Atomic long long`|
|[i[`atomic_ullong` type]]`atomic_ullong`|`_Atomic unsigned long long`|
|[i[`atomic_char16_t` type]]`atomic_char16_t`|`_Atomic char16_t`|
|[i[`atomic_char32_t` type]]`atomic_char32_t`|`_Atomic char32_t`|
|[i[`atomic_wchar_t` type]]`atomic_wchar_t`|`_Atomic wchar_t`|
|[i[`atomic_int_least8_t` type]]`atomic_int_least8_t`|`_Atomic int_least8_t`|
|[i[`atomic_uint_least8_t` type]]`atomic_uint_least8_t`|`_Atomic uint_least8_t`|
|[i[`atomic_int_least16_t` type]]`atomic_int_least16_t`|`_Atomic int_least16_t`|
|[i[`atomic_uint_least16_t` type]]`atomic_uint_least16_t`|`_Atomic uint_least16_t`|
|[i[`atomic_int_least32_t` type]]`atomic_int_least32_t`|`_Atomic int_least32_t`|
|[i[`atomic_uint_least32_t` type]]`atomic_uint_least32_t`|`_Atomic uint_least32_t`|
|[i[`atomic_int_least64_t` type]]`atomic_int_least64_t`|`_Atomic int_least64_t`|
|[i[`atomic_uint_least64_t` type]]`atomic_uint_least64_t`|`_Atomic uint_least64_t`|
|[i[`atomic_int_fast8_t` type]]`atomic_int_fast8_t`|`_Atomic int_fast8_t`|
|[i[`atomic_uint_fast8_t` type]]`atomic_uint_fast8_t`|`_Atomic uint_fast8_t`|
|[i[`atomic_int_fast16_t` type]]`atomic_int_fast16_t`|`_Atomic int_fast16_t`|
|[i[`atomic_uint_fast16_t` type]]`atomic_uint_fast16_t`|`_Atomic uint_fast16_t`|
|[i[`atomic_int_fast32_t` type]]`atomic_int_fast32_t`|`_Atomic int_fast32_t`|
|[i[`atomic_uint_fast32_t` type]]`atomic_uint_fast32_t`|`_Atomic uint_fast32_t`|
|[i[`atomic_int_fast64_t` type]]`atomic_int_fast64_t`|`_Atomic int_fast64_t`|
|[i[`atomic_uint_fast64_t` type]]`atomic_uint_fast64_t`|`_Atomic uint_fast64_t`|
|[i[`atomic_intptr_t` type]]`atomic_intptr_t`|`_Atomic intptr_t`|
|[i[`atomic_uintptr_t` type]]`atomic_uintptr_t`|`_Atomic uintptr_t`|
|[i[`atomic_size_t` type]]`atomic_size_t`|`_Atomic size_t`|
|[i[`atomic_ptrdiff_t` type]]`atomic_ptrdiff_t`|`_Atomic ptrdiff_t`|
|[i[`atomic_intmax_t` type]]`atomic_intmax_t`|`_Atomic intmax_t`|
|[i[`atomic_uintmax_t` type]]`atomic_uintmax_t`|`_Atomic uintmax_t`|
```

``` {.c}
[i[`_Atomic`类型修饰符]>]

随意使用它们！它们与在C++中找到的原子别名一致，如果有帮助的话。

但是如果你想要更多呢？

您可以使用类型修饰符或类型说明符来实现。

[i[`_Atomic`类型说明符]<]

首先，说明符！它是关键字`_Atomic`后跟括号中的类型，适用于`typedef`：

``` {.c}
typedef _Atomic(double) atomic_double;

atomic_double f;
```

关于说明符的限制：您正在使原子化的类型不能是数组类型或函数类型，也不能是原子化或其他修饰符类型。

[i[`_Atomic`类型说明符]>]
[i[`_Atomic`类型修饰符]<]

接下来是修饰符！这是关键字`_Atomic`，但后面没有类型在括号中。

因此，它们执行类似的功能：

``` {.c}
_Atomic(int) i;   // 类型说明符
_Atomic int  j;   // 类型修饰符
```

问题是，您可以在后者中包含其他类型修饰符：

``` {.c}
_Atomic volatile int k;   // 限定的原子变量
```

修饰符的限制：您正在使其原子化的类型不能是数组类型或函数类型。

[i[`_Atomic`类型修饰符]>]

## 无锁原子变量 {#lock-free-atomic}

[i[原子变量-->无锁]<]

硬件架构在原子读写操作中的数据量是有限的。这取决于如何连接在一起。而且各不相同。

如果使用原子类型，可以确保对该类型的访问是原子的……但有个问题：如果硬件无法完成，将使用锁代替。

因此，原子访问变成了锁-访问-解锁，速度会变慢，对信号处理程序也有一些影响。
```

```c
原子标志如下是唯一在所有符合规范的实现中保证无锁的原子类型。在典型的台式机/笔记本电脑世界中，其他较大的类型可能是无锁的。

幸运的是，我们有几种方法来确定特定类型是无锁原子还是不是。

首先，一些宏---您可以在编译时使用这些宏`#if`。它们适用于有符号和无符号类型。

|原子类型|无锁宏|
|-|-|
|`atomic_bool`|[i[`ATOMIC_BOOL_LOCK_FREE`宏]]`ATOMIC_BOOL_LOCK_FREE`|
|`atomic_char`|[i[`ATOMIC_CHAR_LOCK_FREE`宏]]`ATOMIC_CHAR_LOCK_FREE`|
|`atomic_char16_t`|[i[`ATOMIC_CHAR16_T_LOCK_FREE`宏]]`ATOMIC_CHAR16_T_LOCK_FREE`|
|`atomic_char32_t`|[i[`ATOMIC_CHAR32_T_LOCK_FREE`宏]]`ATOMIC_CHAR32_T_LOCK_FREE`|
|`atomic_wchar_t`|[i[`ATOMIC_WCHAR_T_LOCK_FREE`宏]]`ATOMIC_WCHAR_T_LOCK_FREE`|
|`atomic_short`|[i[`ATOMIC_SHORT_LOCK_FREE`宏]]`ATOMIC_SHORT_LOCK_FREE`|
|`atomic_int`|[i[`ATOMIC_INT_LOCK_FREE`宏]]`ATOMIC_INT_LOCK_FREE`|
|`atomic_long`|[i[`ATOMIC_LONG_LOCK_FREE`宏]]`ATOMIC_LONG_LOCK_FREE`|
|`atomic_llong`|[i[`ATOMIC_LLONG_LOCK_FREE`宏]]`ATOMIC_LLONG_LOCK_FREE`|
|`atomic_intptr_t`|[i[`ATOMIC_POINTER_LOCK_FREE`宏]]`ATOMIC_POINTER_LOCK_FREE`|

这些宏可以有_三_种不同的值：

|值|含义|
|-|-|
|`0`|从不无锁。|
|`1`|_有时_无锁。|
|`2`|总是无锁。|
```

等等---某物怎么会“有时”是无锁的呢？这只是意味着在编译时不知道答案，但后来在运行时可能会知道。也许答案取决于你是否在真正的英特尔或 AMD 上运行这段代码，或者类似的情况^[我只是随便举个例子。也许在 Intel/AMD 上无关紧要，但在其他地方可能很重要，该死！]。

不过你总能在运行时用 `atomic_is_lock_free()` 函数进行测试。这个函数会返回当前特定类型是否是原子的真或假。

那我们为什么要在意呢？

无锁是更快的，也许速度是你需要用另一种方式编写代码的一个考虑因素。或者你可能需要在信号处理程序中使用原子变量。

### 信号处理程序和无锁原子变量

如果你在信号处理程序中读取或写入一个共享变量（静态存储期或 `_Thread_Local`），这将是未定义行为[哇！]...除非你做以下几件事之一：

1. 写入一个 `volatile sig_atomic_t` 类型的变量。

2. 读取或写入一个无锁原子变量。

据我所知，无锁原子变量是你能够从信号处理程序中跨平台获取信息的少数几种方法之一。

规范有点模糊，在我看来，关于在信号处理程序中获取或释放原子变量时的内存顺序，并未明确定义。C++ 表示，而且这是有道理的，此类访问与程序的其余部分**无序**[^C++ elaborates that if the signal is the result of a call to `raise()` function, it is sequenced _after_ the `raise()`.]. 毕竟，信号可以在任何时候发出。所以我假设 C 的行为是类似的。

[信号处理程序-->使用无锁原子操作]
[原子变量-->与信号处理程序]

## 原子标志 {#atomic-flags}

[原子变量-->原子标志]
[`atomic_flag` 类型]

标准保证的唯一一种有锁自由原子操作的类型是 `atomic_flag`. 这是一个用于[测试和设置|Test-and-set]操作的不透明类型。

它可以是 _设置_ 或 _清除_。您可以使用以下方式将其初始化为清除状态：

[`ATOMIC_FLAG_INIT` 宏]

``` {.c}
atomic_flag f = ATOMIC_FLAG_INIT;
```

[`ATOMIC_FLAG_INIT` 宏]

[`atomic_flag_test_and_set()` 函数]

您可以使用 `atomic_flag_test_and_set()` 原子地设置标志，并将其先前的状态作为 `_Bool` 返回（对于设置为 true）。

[`atomic_flag_clear()` 函数]

您可以使用 `atomic_flag_clear()` 原子地清除标志。

以下是一个示例，我们将标志初始化为清除状态，设置两次，然后再次清除。

``` {.c}
#include <stdio.h>
#include <stdbool.h>
#include <stdatomic.h>

atomic_flag f = ATOMIC_FLAG_INIT;

int main(void)
{
    bool r = atomic_flag_test_and_set(&f);
    printf("值是: %d\n", r);           // 0

    r = atomic_flag_test_and_set(&f);
    printf("值是: %d\n", r);           // 1
```

```c
#include <stdio.h>
#include <stdatomic.h>

int main(void)
{
    atomic_flag_clear(&f);
    r = atomic_flag_test_and_set(&f);
    printf("Value was: %d\n", r);           // 0
}
```

[i[`atomic_flag_clear()`函数]>]
[i[`atomic_flag_test_and_set()`函数]>]
[i[原子变量-->原子标志]>]
[i[`atomic_flag`类型]>]

## 原子`struct`和`union`

[i[原子变量-->`struct`和`union`]<]

使用`_Atomic`限定符或说明符，可以使`struct`或`union`变得原子！相当惊人。

如果里面的数据量不大（即几个字节），那么得到的原子类型可能是无锁定的。可以用`atomic_is_lock_free()`来测试。

``` {.c .numberLines}
#include <stdio.h>
#include <stdatomic.h>

int main(void)
{
    struct point {
        float x, y;
    };

    _Atomic(struct point) p;

    printf("Is lock free: %d\n", atomic_is_lock_free(&p));
}
```

这里的问题是：无法访问原子`struct`或`union`的字段... 那是什么意义呢？好吧，你可以以原子方式 _复制_ 整个`struct`到非原子变量中，然后再使用它。也可以以原子方式反向复制。

``` {.c .numberLines}
#include <stdio.h>
#include <stdatomic.h>

int main(void)
{
    struct point {
        float x, y;
    };

    _Atomic(struct point) p;
    struct point t;

    p = (struct point){1, 2};  // 原子复制

    //printf("%f\n", p.x);  // 错误

    t = p;   // 原子复制

    printf("%f\n", t.x);  // OK!
}
```

还可以声明一个`struct`，其中各个字段都是原子的。在位域上是否允许原子类型是由实现定义的。

[i[原子变量-->`struct`和`union`]>]

## 原子指针

[i[原子变量-->指针]<]

这里只是一个关于指针使用`_Atomic`定位的说明。

首先，原子指针（即指针值不是原子的，但它指向的东西是）：

``` {.c}
_Atomic int x;
_Atomic int *p;  // p是指向原子int的指针

p = &x;  // 没问题！
```

其次，原子指针指向非原子值（即指针值本身是原子的，但它指向的东西不是）：

``` {.c}
int x;
int * _Atomic p;  // p是指向int的原子指针

p = &x;  // 没问题！
```

最后，原子指针指向原子值（即指针和它指向的东西都是原子的）：

``` {.c}
_Atomic int x;
_Atomic int * _Atomic p;  // p是指向原子int的原子指针

p = &x;  // 没问题！
```

[i[原子变量-->指针]]>

## 内存顺序

[i[原子变量-->内存顺序]<]
[i[内存顺序]<]

我们已经讨论过顺序一致性，这是其中一个明智的选择。但还有其他几种：

|`memory_order`|描述|
|-|-|
|[i[`memory_order_seq_cst` 枚举类型]]`memory_order_seq_cst`|顺序一致性|
|[i[`memory_order_acq_rel` 枚举类型]]`memory_order_acq_rel`|获取/释放|
|[i[`memory_order_release` 枚举类型]]`memory_order_release`|释放|
|[i[`memory_order_acquire` 枚举类型]]`memory_order_acquire`|获取|
|[i[`memory_order_consume` 枚举类型]]`memory_order_consume`|消费|
|[i[`memory_order_relaxed` 枚举类型]]`memory_order_relaxed`|松散|

您可以使用某些库函数指定其他顺序。例如，您可以像这样向原子变量添加一个值：

``` {.c}
atomic_int x = 0;

x += 5;  // 顺序一致性，默认情况
```

或者您可以使用这个库函数做同样的事情：

[i[`atomic_fetch_add()` 函数]<]

``` {.c}
atomic_int x = 0;

使用`atomic_fetch_add(&x, 5);`函数递增`x`变量的值5。// 顺序一致性，默认方式

``` {.c}
[初始化`x`变量为`atomic_int x = 0;`]

使用`atomic_fetch_add_explicit(&x, 5, memory_order_seq_cst);`函数以显式内存顺序执行相同的操作。
```

如果不想要顺序一致性，而是想要获取/释放内存顺序，无论出于什么原因，只需要指定：

``` {.c}
[初始化`x`变量为`atomic_int x = 0;`]

使用`atomic_fetch_add_explicit(&x, 5, memory_order_acq_rel);`函数。
```

我们将在下面详细介绍不同的内存顺序。除非确切了解操作原理，否则请不要改变除顺序一致性以外的任何内容。很容易犯下错误，导致罕见的难以重现的失败。

### 顺序一致性

[原子变量-->顺序一致性]
[内存顺序-->顺序一致性]

- 加载操作会获取（见下文）。
- 存储操作会释放（见下文）。
- 读-修改-写操作会先获取再释放。

另外，为了保持获取和释放的总顺序，不会对获取和释放进行重新排序。 （获取/释放规则不禁止对一个释放后跟随一个获取进行重新排序。但顺序一致性规则禁止这样做。）

[内存顺序-->顺序一致性]
[原子变量-->顺序一致性]

### 获取

[原子变量-->获取]
[内存顺序-->获取]

这就是发生在原子变量加载/读操作时的情况。

- 如果另一个线程释放了该原子变量，那么该线程的所有写操作都将对该线程可见。
- 该载入操作之后的内存访问将不能被重排序到该操作之前。

```c
[i[内存顺序-->获取]>]
[i[原子变量-->获取]>]

### 释放

[i[原子变量-->释放]<]
[i[内存顺序-->获取]<]

这是在原子变量的存储/写入上发生的情况。

* 如果另一个线程后来获取这个原子变量，则在其原子写入之前此线程中的所有内存写入对于那个线程将变得可见。

* 在释放之前发生的该线程中的内存访问不能在其后重新排序。

[i[原子变量-->释放]>]
[i[内存顺序-->释放]>]

### 消费

[i[原子变量-->消费]<]
[i[内存顺序-->消费]<]

这是一个奇怪的情况，类似于获取的一个较不严格的版本。它会影响那些与原子变量“数据相关”的内存访问。

“数据相关”的含义是该原子变量用于计算。

也就是说，如果一个线程消费了一个原子变量，那么该线程中使用该原子变量的所有操作将能够看到释放线程中的内存写入。

与获取不同的是，在释放线程中的内存写入将对当前线程中的**所有**操作可见，而不仅仅是与数据相关的操作。

与获取类似的是，在消费之前有一些操作无法重新排序。对于获取，你不能在其之前重新排序任何操作。对于消费，你不能在加载的原子值之前重新排序任何依赖于它的操作。

[i[原子变量-->消费]>]
[i[内存顺序-->消费]>]

### 获取/释放

[i[原子变量-->获取/释放]<]
[i[内存顺序-->获取/释放]<]

这仅适用于读取-修改-写入操作。它是一个获取和释放的组合。

* 读取时会执行获取。
* 写入时会执行释放。
```

### Atomic Variables - 获取/释放

### 内存顺序 - 获取/释放

### 松散的

不存在规则；就是无政府状态！任何人都可以随意重新排列一切！狗和猫生活在一起---一片混乱！

实际上，还是有一个规则。原子读取和写入仍然是全有或全无的。但操作可以随意重新排序，线程之间没有任何同步。

这种内存顺序有一些使用案例，你可以通过简单的搜索找到，比如简单的计数器。

你还可以使用栅栏在一堆松散的写入之后强制进行同步。

### 原子变量 - 松散的

### 内存顺序 - 松散的

### 内存顺序

### 原子变量 - 内存顺序

## 栅栏

你知道原子变量的释放和获取是如何发生的吗？

嗯，其实也可以在没有原子变量的情况下进行释放或获取。

这就是所谓的“栅栏”。因此，如果你希望一个线程中的所有写入在其他地方可见，你可以在一个线程中设置释放栅栏，另一个线程中设置获取栅栏，就像使用原子变量一样。

由于在栅栏上不太可能进行消费操作^[因为消费是指依赖于已获取的原子变量值的操作，而在栅栏上没有原子变量。]，`memory_order_consume` 被视为获取。

你可以使用任何指定顺序来设置栅栏：

[`atomic_thread_fence()` 函数]

``` {.c}
atomic_thread_fence(memory_order_release);
```

[`atomic_thread_fence()` 函数]

[`atomic_signal_fence()` 函数]

```
嗨，这里还有一个轻量级的围栏供信号处理程序使用，名为 `atomic_signal_fence()`。

它的工作方式与 `atomic_thread_fence()` 完全相同，不同之处在于：

- 它仅处理同一线程内值的可见性；与其他线程没有同步。

- 不会发出任何硬件围栏指令。

如果您想确保非原子操作和松散原子操作的副作用在信号处理程序中是可见的，您可以使用这个围栏。

思想是，信号处理程序正在 _这个_ 线程中执行，而不是另一个线程，因此这是一种更轻量级的方法，可确保信号处理程序之外的更改在其中是可见的（即它们没有被重新排序）。

[i[`atomic_signal_fence()` 函数]>]
[i[原子变量-->围栏]>]

## 参考

如果您想了解更多相关信息，这些内容可能会对您有所帮助：

- Herb Sutter 的 _`atomic<>` Weapons_ 演讲：
  - [fl[第1部分|https://www.youtube.com/watch?v=A8eCGOqgvH4]]
  - [fl[第2部分|https://www.youtube.com/watch?v=KeLBd2EJLOU]]
```

```markdown
* [fl[Jeff Preshing的资料|https://preshing.com/archives/]], 尤其是:
  * [fl[无锁编程简介|https://preshing.com/20120612/an-introduction-to-lock-free-programming/]]
  * [fl[获取和释放语义|https://preshing.com/20120913/acquire-and-release-semantics/]]
  * [fl[_先行发生_关系|https://preshing.com/20130702/the-happens-before-relation/]]
  * [fl[_同步与_关系|https://preshing.com/20130823/the-synchronizes-with-relation/]]
  * [fl[C++11中`memory_order_consume`的目的|https://preshing.com/20140709/the-purpose-of-memory_order_consume-in-cpp11/]]
  * [fl[您可以执行任何类型的原子读-修改-写操作|https://preshing.com/20150402/you-can-do-any-kind-of-atomic-read-modify-write-operation/]]

* CPPReference:
  * [fl[内存顺序|https://en.cppreference.com/w/c/atomic/memory_order]]
  * [fl[原子类型|https://en.cppreference.com/w/c/language/atomic]]

* Bruce Dawson的 [fl[无锁编程注意事项|https://docs.microsoft.com/en-us/windows/win32/dxtecharts/lockless-programming]]

* 在[fl[r/C_Programming|https://www.reddit.com/r/C_Programming/]]上乐于助人且知识渊博的朋友们

[i[原子变量]>]
```