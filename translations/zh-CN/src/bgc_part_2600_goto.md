# `goto`

`goto`语句被普遍崇拜，可以毫不争议地在这里介绍。

开个玩笑！多年来，人们一直争论不休，关于是否（通常不是）认为`goto`是[flw[有害的|Goto#Criticism]]。

在这个程序员的看法中，您应该使用导致最佳代码的构造方式，同时考虑可维护性和速度。有时候这可能会是`goto`！

在本章中，我们将看到`goto`在C中的工作原理，然后看看一些常见的使用情况^[我想指出，在所有这些情况下使用`goto`是可以避免的。您可以使用变量和循环代替。只是有些人认为`goto`在这些情况下产生了最佳的代码。]。

## 一个简单示例

[标签]

在这个示例中，我们将使用`goto`跳过一行代码，跳到一个_标签_。 标签是一个可以作为`goto`目标的标识符---它以冒号（`:`）结束。

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

输出是：

``` {.default}
One
Two
Five!
```

`goto`会将执行跳转到指定的标签，跳过中间的所有内容。

您可以通过`goto`向前或向后跳转。

``` {.c}
infinite_loop:
    print("Hello, world!\n");
    goto infinite_loop;
```

标签在执行过程中会被跳过。以下代码将按顺序打印出所有三个数字，就好像标签不存在一样：

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

正如您已经注意到的，通常约定俗成的是将标签左对齐。这样做可以增加可读性，因为读者可以快速浏览以找到目标。

标签具有函数作用域。也就是说，无论在嵌套多少层的块中它们出现，您仍然可以从函数的任何地方`goto`到它们。

这也意味着您只能`goto`与`goto`本身相同函数中的标签。在`goto`的视角下，其他函数中的标签超出了作用域。这也意味着您可以在两个函数中使用相同的标签名称---只是不能在同一个函数中使用相同的标签名称。

## 带标签的 'continue'

在某些语言中，您实际上可以为`continue`语句指定一个标签。C语言不允许，但您可以轻松地使用`goto`来代替。

要了解这个问题，请查看这个嵌套循环中的`continue`：

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d, %d\n", i, j);
        continue;   // 总是跳到下一个j
    }
}
```

正如我们所看到的，那个`continue`，就像所有的`continue`一样，进入最近外围循环的下一次迭代。如果我们想在下一个更外层的循环中`continue`，也就是带有`i`的循环，怎么办呢？

那么，我们可以通过`break`来返回到外部循环，对吗？

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("%d, %d\n", i, j);
        break;     // 让我们进入i的下一次迭代
    }
}
```

这将让我们脱离两层嵌套循环。但是如果我们再嵌套另一个循环，我们就没什么选择了。如果没有任何语句可以让我们退出到`i`的下一个迭代，该怎么办呢？

``` {.c}
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        for (int k = 0; k < 3; k++) {
            printf("%d, %d, %d\n", i, j, k);

            continue;  // 进入下一个 k 的迭代
            break;     // 进入下一个 j 的迭代
            ????;      // 进入下一个 i 的迭代???

        }
    }
}
```

使用 `goto` 语句提供了一种方法！

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            for (int k = 0; k < 3; k++) {
                printf("%d, %d, %d\n", i, j, k);

                goto continue_i;   // 现在继续执行 i 循环！！
            }
        }
continue_i: ;
    }
```

我们在最后加了一个 `;`---因为你不能有一个标号指向块语句的结尾（或在变量声明之前）。

[i[`goto` statement-->as labeled `continue`]>]

## 跳出

[i[`goto` statement-->for bailing out]<]

当你在代码中很深的嵌套中时，你可以使用 `goto` 来以比再嵌套更多 `if` 语句和使用标志变量更清晰的方式跳出。

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
    // 在这里做清理工作
```

没有 `goto`，你必须在所有循环中检查错误条件标志才能完全跳出。

[i[`goto` statement-->for bailing out]>]

## 标号 `break`

[i[`goto` statement-->as labeled `break`]<]

这与 `continue` 只继续最内部循环的情况非常相似。`break` 也只能跳出最内部循环。

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d, %d\n", i, j);
            break;   // 仅跳出 j 循环
        }
    }

    printf("完成！\n");
```

但我们可以使用 `goto` 来跳出更远的循环：

``` {.c}
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d, %d\n", i, j);
            goto break_i;   // 现在跳出 i 循环！
        }
    }

break_i:

    printf("完成！\n");
```

## 多层清理

如果您正在调用多个函数来初始化多个系统，并且其中一个失败了，您应该只去初始化到目前为止的系统。

让我们举一个假的例子，我们开始初始化系统并检查是否有任何返回错误（我们将使用 `-1` 表示错误）。如果其中一个失败了，我们必须只关闭到目前为止初始化的系统。

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

shutdown_3:
    shutdown_system3();

shutdown_2:
    shutdown_system2();

shutdown_1:
    shutdown_system1();

shutdown:
    printf("所有子系统已关闭。\n");
```

请注意我们以初始化子系统的相反顺序关闭子系统。因此，如果子系统 4 无法启动，它将以 3、2、1 的顺序关闭。

[i[`goto` 语句-->多层清理]>]

## 尾递归优化

Kinda. 仅适用于递归函数。

如果你不熟悉，[flw[Tail Call Optimization (TCO)|尾调用优化]] 是一种在特定情况下调用其他函数时不浪费栈空间的方法。不幸的是，细节超出了这个指南的范围。

但如果你有一个递归函数，你知道可以通过这种方式优化，你可以利用这个技术。（注意，由于标签的函数作用域，你不能对其他函数进行尾调用。）

让我们用一个简单的例子，阶乘，来说明。

这里是一个不是TCO的递归版本，但它可以变成TCO！

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

要实现这一点，你可以将调用替换为两个步骤：

1. 将参数的值设置为下一次调用时的值。
2. `goto` 到函数的第一行的标签。

让我们试一试：

``` {.c .numberLines}
#include <stdio.h>

int factorial(int n, int a)
{
tco:  // 加入这个

    if (n == 0)
        return a;

    // 通过设置新的参数值并跳到函数的开始来替代返回

    //return factorial(n - 1, a * n);

    int next_n = n - 1;  // 看看这些是如何与前面的递归参数匹配的？
    int next_a = a * n;

    n = next_n;   // 将参数设置为新值
    a = next_a;

    goto tco;   // 重复！
}

int main(void)
{
    for (int i = 0; i < 8; i++)
        printf("%d! == %d\n", i, factorial(i, 1));
}
```

我之前使用临时变量来设置函数跳转到起始点前的参数下一个值。看一下它们是如何对应于递归调用中的递归参数的呢？

现在，为什么要使用临时变量呢？我可以用下面这种方式代替：

``` {.c}
    a *= n;
    n -= 1;

    goto tco;
```

这实际上也可以正常运行。但是如果不小心颠倒了这两行代码：

``` {.c}
    n -= 1;  // 不好的消息
    a *= n;
```

---那我们就有麻烦了。在修改`a`之前修改了`n`。这是不好的，因为当进行递归调用时不是这样工作的。即使你不留心，使用临时变量可以避免这个问题。而且编译器很可能会将其优化掉。

[goto语句-->尾递归优化]
[尾递归优化-->使用`goto`]

## 重新启动被中断的系统调用

[goto语句-->重新开启系统调用]

这超出了规范，但在类Unix系统中经常见到。

某些长时间运行的系统调用如果被信号中断可能会返回错误，并且`errno`会设置为`EINTR`以指示系统调用本来没有问题，只是被中断了。

在这些情况下，程序员很常见地希望重新启动调用并再次尝试。

``` {.c}
retry:
    byte_count = read(0, buf, sizeof(buf) - 1);  // Unix read()系统调用

    if (byte_count == -1) {            // 发生了错误...
        if (errno == EINTR) {          // 但只是被中断了
            printf("重新启动...\n");
            goto retry;
        }
```

许多类Unix系统使用`SA_RESTART`标志，您可以传递给`sigaction()`请求操作系统自动重新启动任何较慢的系统调用，而不是使用`EINTR`失败。

再次说明，这是特定于Unix的，不符合C标准。

即便如此，任何时候都可以使用类似的技巧来重新启动任何函数。

## `goto`语句和线程抢占

这个示例直接摘自《操作系统：三部曲》（_Operating Systems: Three Easy
Pieces_），这是另一本出色的书籍，来自志同道合的作者，他们也认为优质的书籍应该免费下载。并不是说我有偏见，或者什么的。

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

线程快乐地获取了互斥量 `L1`，但随后可能无法获得由互斥量 `L2` 保护的第二资源（假设有其他不合作的线程持有）。如果我们的线程无法获取 `L2` 锁，则会解锁 `L1`，然后使用 `goto` 干净地重试。

我们希望我们的英雄线程最终能够获取两个互斥量并挽救一天，同时避免邪恶的死锁。

## `goto`语句和变量作用域

我们已经看到标签具有函数作用域，但如果我们跳过某个变量的初始化就跳转会发生一些奇怪的事情。

看看这个例子，我们从变量 `x` 超出作用域的地方跳转到其作用域中间（在块中）。

``` {.c}
    goto label;

    {
        int x = 12345;

label:
        printf("%d\n", x);
    }
```

这会编译并运行，但会给出警告：

``` {.default}
warning: ‘x’ is used uninitialized in this function
```

然后执行时会打印出 `0`（实际结果可能会有所不同）。

基本上发生的情况是，我们跳转到了 `x` 的作用域（所以在 `printf()` 中引用它是可以的），但是我们跳过了实际将其初始化为 `12345` 的那一行。所以该值是不确定的。

修复方法当然是在标签之后以某种方式进行初始化。

``` {.c}
    goto label;

    {
        int x;

label:
        x = 12345;
        printf("%d\n", x);
    }
```

让我们再看一个例子。

``` {.c}
    {
        int x = 10;

label:

        printf("%d\n", x);
    }

    goto label;
```

这里会发生什么？

第一次通过代码块时，一切正常。`x` 是 `10`，打印出来就是 `10`。

但是在 `goto` 之后，我们跳转到了 `x` 的作用域，但是已经跳过了初始化。这意味着我们仍然可以打印它，但是值是不确定的（因为它没有被重新初始化）。

在我的机器上，它再次打印出 `10`（无限循环），但这只是运气。在 `goto` 之后，它可以打印出任何值，因为 `x` 是未初始化的。

[i[使用`goto`语句-->变量作用域]>]

## `goto` 和可变长度数组

[i[使用`goto`语句-->用于可变长度数组]>]

在使用可变长度数组和 `goto` 时，有一个规则：不能从可变长度数组的作用域外跳转到该可变长度数组的作用域中。

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

你可以在可变长度数组声明之前跳转进去，像这样：{}

``` {.c}
    int x = 10;

    goto label;

    {
label:  ;
        int v[x];

        printf("Hi!\n");
    }
```

因为这样可以在可变长度数组不可避免地超出范围时正确分配它。