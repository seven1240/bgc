```c

#include <stdio.h>

int main(void)
{
    int a = 12;         // Local to outer block, but visible in inner block

    if  (a == 12) {
        int b = 99;     // Local to inner block, not visible in outer block

        printf("%d %d\n", a, b);  // OK: "12 99"
    }

    printf("%d\n", a);  // OK, we're still in a's scope

    printf("%d\n", b);  // ILLEGAL, out of b's scope
}
```

在历史上，C需要在代码块内的任何代码之前定义所有变量，但在C99标准中不再需要这样做。

### 变量隐藏

如果你在内部作用域和外部作用域有同名变量，那么在内部作用域运行时，内部作用域的变量优先级更高。换句话说，在其生存期内，它会**隐藏**外部作用域的变量。

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 10;

    {
        int i = 20;

        printf("%d\n", i);  // 内部作用域的 i，20（外部的 i 被隐藏）
    }

    printf("%d\n", i);  // 外部作用域的 i，10
}
```

你可能已经注意到，在这个示例中，我只是在第7行扔进了一个块，没有`for`或`if`语句来启动它！这是完全合法的。有时开发人员会想将一堆本地变量组合在一起进行快速计算，就会这样做，但很少见。

## 文件作用域

如果你在一个代码块之外定义一个变量，那个变量就具有_文件作用域_。它在后面所有函数中都可见，并在它们之间共享。（一个例外是如果一个块定义了同名变量，它会隐藏文件作用域的变量。）

这与其他语言中你会认为的“全局”作用域最接近。

例如：

``` {.c .numberLines}
#include <stdio.h>

int shared = 10;    // 文件作用域！在此之后整个文件可见！

void func1(void)
{
    shared += 100;  // 现在 shared 是 110
}

void func2(void)
{
    printf("%d\n", shared);  // 输出"110"
}

int main(void)
{
    func1();
    func2();
}
```

请注意，如果`shared`在文件底部声明，代码无法通过编译。必须在任何函数使用之前声明它。

有一些方法可以进一步修改文件范围内的项目，具体来说是使用`static`和`extern`，但我们稍后再详细讨论。

## `for`循环作用域

我真的不知道该怎么称呼这种情况，因为C11 §6.8.5.3¶1没有为其命名。在本指南中，我们已经多次实现过这种情况。这种情况是当你在`for`循环的第一个子句中声明一个变量时：

``` {.c}
for (int i = 0; i < 10; i++)
    printf("%d\n", i);

printf("%d\n", i);  // 非法--i仅在for循环内可见
```

在这个例子中，`i`的生命周期从它被定义的那一刻开始，持续到循环结束。

如果循环体被包裹在一个块中，那么在`for`循环中定义的变量可以从内部作用域中访问。

除非当然，内部作用域将其隐藏。这个荒谬的例子会打印出`999`五次：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    for (int i = 0; i < 5; i++) {
        int i = 999;  // 隐藏了`for`循环作用域中的i
        printf("%d\n", i);
    }
}
```

## 函数作用域注记

C规范确实提到了_函数作用域_，但它仅仅与_标签_一起使用，这是我们尚未讨论的内容。以后再详细了解。