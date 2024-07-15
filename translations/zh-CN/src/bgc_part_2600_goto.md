<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# `goto`

[i[`goto` 语句]<]

`goto` 语句被普遍推崇，并且可以在此处毫无争议地展示出来。

开玩笑！多年来，关于`goto`语句是否（通常不）[flw[被视为有害|Goto#Criticism]]，一直存在着很多争论。

在本程序员看来，你应该使用能产生 _最好_ 代码的构造方式，考虑可维护性和速度。有时候，这可能会是`goto`！

在这一章节中，我们将了解`goto`在C语言中的运作方式，然后看看一些常见的情况，其中会使用`goto`^[我想指出，在所有这些情况下使用`goto`是可以避免的。您可以使用变量和循环来代替。只是有些人认为在这些情况下，`goto`生成了最佳的代码。]。

## 一个简单的例子

[i[标签]<]

在这个例子中，我们将使用`goto`语句跳过一行代码，直接跳转到一个 _标签_。标签是一个标识符，它可以成为`goto`的目标---它以冒号（`:`）结尾。

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    printf("One\n");
    printf("Two\n");

    goto skip_3;

    printf("Three\n");

skip_3:

    printf("Five!\n");
}
```

输出结果为：

``` {.default}
One
Two
Five!
```

`goto`将执行跳转到指定的标签，跳过中间的所有内容。

您可以使用`goto`向前或向后跳转。

``` {.c}
infinite_loop:
    print("Hello, world!\n");
    goto infinite_loop;
```

标签在执行过程中被跳过。以下代码将按顺序打印出所有三个数字，就好像标签不存在一样：

``` {.c}
    printf("Zero\n");
label_1:
label_2:
    printf("One\n");
label_3:
    printf("Two\n");
label_4:
    printf("Three\n");
```

正如你所注意到的那样，将标签左对齐是一种常见的约定。这样做可以提高可读性，因为读者可以快速扫描以找到目标。

标签具有**函数范围**。也就是说，无论它们出现在多少层嵌套的代码块中，你仍然可以从函数的任何地方`goto`到它们。

这也意味着你只能`goto`位于相同函数中的标签。其他函数中的标签对于`goto`而言超出范围。同时，这意味着你可以在两个函数中使用相同的标签名称，但不能在同一个函数中使用相同的标签名称。

[i[标签]>]

## 标记过的`continue`

[i[`goto`语句-->作为标记过的`continue`]<]

在某些语言中，你实际上可以为`continue`语句指定一个标签。C语言不允许这样做，但你可以很容易地使用`goto`来代替。

让我们看一个嵌套循环中的`continue`语句的示例：

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d, %d\n", i, j);
        continue;   // 总是继续下一个j
    }
}
```

正如我们所看到的那样，`continue`，就像所有的`continue`一样，会进入最近包围它的循环的下一个迭代。但如果我们想要在外部的下一个循环中继续，也就是包含`i`的循环呢？

嗯，我们可以使用`break`返回到外部循环，对吧？

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d, %d\n", i, j);
        break;     // 使我们进入i的下一个迭代
    }
}
```

这样我们可以脱离两层嵌套的循环。但如果我们再嵌套另一个循环，那我们就没有其他选择了。如果没有任何语句可以使我们进入`i`的下一个迭代，那该怎么办呢？

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        for (int k = 0; k < 3; k++) {
            printf("%d, %d, %d\n", i, j, k);

            continue;  // 继续下一个 k 的迭代
            break;     // 跳出当前循环，继续下一个 j 的迭代
            ????;      // 快速跳出当前循环，继续下一个 i 的迭代???
        }
    }
}
```

使用`goto`语句为我们提供了一种方法！

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        for (int k = 0; k < 3; k++) {
            printf("%d, %d, %d\n", i, j, k);

            goto continue_i;   // 现在继续 i 循环！！
        }
    }
continue_i: ;
}
```

在最后有个`;`---这是因为你不能让一个标签指向复合语句的末尾(或者变量声明之前)。

[i[`goto`语句-->作为标记的`continue`]>]

## 跳出

[i[`goto`语句-->用于跳出]<]

当你深度嵌套在某些代码中时，你可以使用`goto`以一种比嵌套更多`if`并使用标志变量更清晰的方式跳出。

``` {.c}
// 伪代码

for(...) {
    for (...) {
        while (...) {
            do {
                if (some_error_condition)
                    goto bail;

            } while(...);
        }
    }
}

bail:
// 这里进行清理
```

没有`goto`，你需要在所有循环中检查错误条件标志才能完全跳出。

[i[`goto`语句-->用于跳出]>]

## 标记的`break`

[i[`goto`语句-->作为标记的`break`]<]

这与`continue`仅继续最内层循环的情形非常相似。`break`也只会跳出最内层循环。

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d, %d\n", i, j);
            break;   // 仅跳出j循环
        }
    }

    printf("完成！\n");
```

但是我们可以使用`goto`进行更远的跳出：

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d, %d\n", i, j);
            goto break_i;   // 现在跳出i循环！
        }
    }

break_i:

    printf("完成！\n");
```

[i[`goto`语句-->作为标记的`break`]>]

## 多层清理

[i[`goto`语句-->多级清理]<]

如果您在调用多个函数来初始化多个系统，并且其中一个失败了，那么您应该只对到目前为止已经初始化的系统进行去初始化。

让我们做一个虚假例子，我们开始初始化系统并检查是否有任何返回错误（我们将使用`-1`表示错误）。如果其中一个出现错误，我们必须只关闭到目前为止已经初始化的系统。

``` {.c}
    if (init_system_1() == -1)
        goto shutdown;

    if (init_system_2() == -1)
        goto shutdown_1;

    if (init_system_3() == -1)
        goto shutdown_2;

    if (init_system_4() == -1)
        goto shutdown_3;

    do_main_thing();   // 运行我们的程序

    shutdown_system4();

shutdown_3:
    shutdown_system3();

shutdown_2:
    shutdown_system2();

shutdown_1:
    shutdown_system1();

shutdown:
    print("所有子系统已关闭。\n");
```

请注意我们按照初始化子系统的相反顺序关闭。因此，如果子系统4启动失败，它将按照顺序关闭3、2，然后是1。

[i[`goto`语句-->多层清理]>]

## 尾递归优化

```
[i[`goto`语句-->尾递归优化]<]
[i[尾递归优化-->使用`goto`]<]

有点。仅限递归函数。

如果你不熟悉，[flw[尾递归优化（TCO）|尾递归]] 是一种在非常特定情况下调用其他函数时不浪费堆栈空间的技术。不幸的是，详细信息超出了本指南的范围。

但如果你有一个递归函数，知道可以通过这种方式优化，你可以利用这个技术。（注意，由于标签的函数范围，你无法尾调用其他函数。）

让我们来看一个简单的例子，阶乘。

这是一个递归版本，不是TCO的，但可以改造成TCO！

``` {.c .numberLines}
#include <stdio.h>
#include <complex.h>

int factorial(int n, int a)
{
    if (n == 0)
        return a;

    return factorial(n - 1, a * n);
}

int main(void)
{
    for (int i = 0; i < 8; i++)
        printf("%d! == %ld\n", i, factorial(i, 1));
}
```

要实现，你可以用两步替换函数调用：

1. 设置参数的值为下一个调用时的值。
2. `goto` 函数的第一行处的标签。

让我们来试试：

``` {.c .numberLines}
#include <stdio.h>

int factorial(int n, int a)
{
tco:  // 添加这行

    if (n == 0)
        return a;

    // 通过设置新参数值和跳转到函数开头的方式替换 return

    //return factorial(n - 1, a * n);

    int next_n = n - 1;  // 看看这里如何对应
    int next_a = a * n;  // 递归参数？

    n = next_n;   // 设置参数为新值
    a = next_a;

    goto tco;   // 再来一次！
}

int main(void)
{
    for (int i = 0; i < 8; i++)
        printf("%d! == %d\n", i, factorial(i, 1));
}
```
```

我在上面使用了临时变量来设置函数起始处跳转前的参数值。看看它们是如何对应递归调用中的递归参数的？

那现在，为什么要使用临时变量呢？其实我也可以这样做：

``` {.c}
    a *= n;
    n -= 1;

    goto tco;
```

其实这样也完全没问题。但是如果我不小心颠倒了这两行代码的顺序：

``` {.c}
    n -= 1;  // 有问题
    a *= n;
```

---那我们就麻烦了。我们在使用`n`来修改`a`之前就修改了 `n`。这不好，因为递归调用的时候不是这么工作的。使用临时变量可以避免这个问题，即使你没有留意。而且编译器很可能会将它们优化掉。

[i[`goto` 语句-->尾调用优化]>]
[i[尾调用优化-->使用 `goto`]>]

## 重新启动被中断的系统调用

[i[`goto` 语句-->重新启动系统调用]<]

这超出了规范范围，但在类Unix系统中很常见。

某些长时间运行的系统调用在被信号中断时可能会返回错误，`errno`会被设置为 `EINTR` 来表示系统调用本身没问题，只是被中断了。

在这种情况下，程序员很常见地会想要重新启动调用并再次尝试。

``` {.c}
retry:
    byte_count = read(0, buf, sizeof(buf) - 1);  // Unix read() 系统调用

    if (byte_count == -1) {            // 出错了...
        if (errno == EINTR) {          // 但只是被中断
            printf("重新启动...\n");
            goto retry;
        }
```

许多类Unix系统可以通过将`SA_RESTART`标志传递给`sigaction()`来请求操作系统自动重新启动任何慢系统调用，而不是因为`EINTR`而失败。

再说一遍，这是特定于Unix的，超出了C标准范围。

尽管如此，任何时候都可以使用类似的技术来重新启动任何函数。

[i[`goto`语句-->重新启动系统调用]>]

## `goto`和线程抢占

[i[`goto`语句-->线程抢占]<]

这个例子直接摘自 [_Operating Systems: Three Easy
Pieces_](http://www.ostep.org/)，这是另一本出色的书，作者们和我一样认为优质的书应该可以免费下载。不是我个人主观看法，哈哈。

``` {.c}
retry:

    pthread_mutex_lock(L1);

    if (pthread_mutex_trylock(L2) != 0) {
        pthread_mutex_unlock(L1);
        goto retry;
    }

    save_the_day();

    pthread_mutex_unlock(L2);
    pthread_mutex_unlock(L1);
```

这里的线程高兴地获得了互斥锁 `L1`，但可能无法获得由互斥锁 `L2` 保护的第二个资源（比如说有其他不合作的线程正在持有）。如果我们的线程无法获取 `L2` 锁，则解锁 `L1`，然后使用 `goto` 来干净地重试。

希望我们的英雄线程最终能够成功获取这两个互斥锁并解决问题，同时避免邪恶的死锁。

[i[`goto`语句-->线程抢占]>]

## `goto`和变量作用域

[i[`goto`语句-->变量作用域]<]

我们已经看到标签具有函数作用域，但如果我们跳过某些变量的初始化会出现奇怪的情况。

看这个例子，在我们跳转到变量 `x` 超出作用域的地方到其作用域中（在块内）。

``` {.c}
    goto label;

    {
        int x = 12345;

label:
        printf("%d\n", x);
    }
```

这会编译并运行，但给出警告：

``` {.default}
warning: ‘x’ is used uninitialized in this function
```

然后当我运行它时打印出`0`（可能会有所不同）。

基本上发生的事情是，我们跳转到了`x`的作用域（所以在`printf()`中引用它没问题），但是我们跳过了实际将其初始化为`12345`的那一行。因此该值是不确定的。

修复的方法当然是在标签之后以某种方式进行初始化。

``` {.c}
    goto label;

    {
        int x;

label:
        x = 12345;
        printf("%d\n", x);
    }
```

让我们看另一个例子。

``` {.c}
    {
        int x = 10;

label:

        printf("%d\n", x);
    }

    goto label;
```

这里会发生什么？

第一次通过该块时，一切正常。`x`是`10`，就打印出了这个值。

但是在`goto`之后，我们跳转到了`x`的作用域，但是超出了它的初始化范围。这意味着我们仍然可以打印它，但值是不确定的（因为它尚未重新初始化）。

在我的机器上，它再次打印出`10`（到无穷大），但那只是运气。在`goto`之后，它可以打印出任何值，因为`x`未初始化。

[i[`goto`语句-->变量作用域]>]

## `goto`和可变长度数组

[i[`goto`语句-->与可变长度数组一起]<]

在涉及VLA和`goto`时，有一个规则：你不能从VLA的作用域之外跳转到该VLA的作用域。

如果我尝试这样做：

``` {.c}
    int x = 10;

    goto label;

    {
        int v[x];

label:

        printf("Hi!\n");
    }
```

我会得到一个错误：

``` {.default}
error: jump into scope of identifier with variably modified type
```

你可以提前跳转到VLA声明的位置，就像这样： 

``` {.c}
    int x = 10;

    goto label;

    {
label:  ;
        int v[x];

        printf("Hi!\n");
    }
```

因为这样可以确保在其超出范围并且不可避免地被释放之前，VLA 能够被适当地分配。