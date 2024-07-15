<!-- Beej's指南 上的C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# C预处理器

在编译程序之前，实际上会经过一个名为_预处理_的阶段。这几乎就像在C语言之上有一种语言在先运行。它输出C代码，然后再进行编译。

我们已经在以前见过这个过程，例如`#include`！那就是C预处理器！遇到这个指令，它就会在那儿马上包含指定的文件，就好像你自己输入一样。然后编译器就会构建整个程序。

但这个预处理器远比仅仅包含文件强大。你可以定义_宏_来进行替换...甚至可以定义带参数的宏！

## `#include`

[<i[`#include`指令]<]

让我们从我们之前见过很多次的开始吧。这当然是一种在源文件中包含其他源代码的方法。在头文件中非常常用。

虽然规范允许`#include`做各种各样的事情，但我们将采用更实用的方式并讨论我见过的每个系统上的运作方式。

我们可以将头文件分为两类：系统和本地。像`stdio.h`、`stdlib.h`、`math.h`等内置的文件，可以用尖括号引入：

``` {.c}
#include <stdio.h>
#include <stdlib.h>
```

尖括号告诉C：“嘿，不要在当前目录查找这个头文件——而是去系统范围的包含目录里找。”

[<i[`#include`指令-->本地文件]<]

当然，这意味着肯定存在一种方式可以从当前目录包含本地文件。确实有这种方法，用双引号：

``` {.c}
#include "myheader.h"
```

或者你很可能可以使用斜杠和点的相对目录来查找：

```

``` {.c}
#include "mydir/myheader.h"
#include "../someheader.py"
```

在`#include`中不要使用反斜杠(`\`)来表示路径分隔符！这种行为是未定义的！即使在Windows上，也只能使用正斜杠(`/`)。

简言之，对于系统包含项，请使用尖括号（`<` 和 `>`），对于个人包含项，请使用双引号（`"`）。

## 简单宏

`#include``指令-->本地文件]]`

`#include`指令`

一个 _宏_ 是一个标识符，在编译器看到之前就会被 _展开_ 成另一段代码。可以把它想象成一个占位符---当预处理器看到这些标识符之一时，它会用你定义的另一个值来替换它。

`#define``指令`

我们用`#define`（常常读作"pound define"）来做到这一点。以下是一个示例：

``` {.c .numberLines}
#include <stdio.h>

#define HELLO "Hello, world"
#define PI 3.14159

int main(void)
{
    printf("%s, %f\n", HELLO, PI);
}
```

在第3和第4行，我们定义了一些宏。这些宏在代码的其他地方（第8行）出现时，它们将被替换为定义的值。

从C编译器的角度来看，这完全相当于我们写了这个代码：

``` {.c .nubmerLines}
#include <stdio.h>

int main(void)
{
    printf("%s, %f\n", "Hello, world", 3.14159);
}
```

注意，宏本身没有具体的类型。实际上，它们只是被替换为`#define`中定义的任何内容。如果生成的C代码无效，编译器会报错。

还可以定义一个没有值的宏：

``` {.c}
#define EXTRA_HAPPY
```

在这种情况下，宏是存在并且被定义了，但是被定义为什么都不是。所以无论在文本中出现的地方都将被替换成空白。我们稍后会看到这种用法。

通常习惯将宏名称写成`全大写`，即使从技术上讲并不是必需的。

[i[`#define`指令-->与`const`]<]

总的来说，这给了你一种定义常量值的方式，这些常量值在实质上是全局的，并且可以在_任何_地方使用。即使在`const`变量不适用的地方，比如`switch` `case`和固定数组长度。

尽管如此，线上争论持续不断，关于在一般情况下，强类型的`const`变量是否比`#define`宏更好。

它也可以用来替换或修改关键字，这是完全陌生于`const`的概念，尽管这种实践应该谨慎使用。

[i[`#define`指令-->与`const`]>]
[i[`#define`指令]>]
[i[预处理器-->宏]>]

## 条件编译

[i[条件编译]<]

可以让预处理器决定是否向编译器呈现特定的代码块，或者在编译之前将其完全删除。

我们通过基本上将代码包装在条件块中来实现，类似于`if`-`else`语句。

### 如果定义了，`#ifdef` 和 `#endif`

首先，让我们尝试根据宏是否被定义来编译特定的代码。

[i[`#ifdef`指令]<]
[i[`#endif`指令]<]

``` {.c .numberLines}
#include <stdio.h>

#define EXTRA_HAPPY

int main(void)
{

#ifdef EXTRA_HAPPY
    printf("我很开心！\n");
#endif

    printf("好的！\n");
}
```

在那个例子中，我们定义了 `EXTRA_HAPPY`（虽然它本身实际上是没有的，但这个标识被定义了），然后在第8行我们使用 `#ifdef` 指令来检查它是否被定义了。如果被定义了，后续的代码直到 `#endif` 都将被包含进来。

所以因为它被定义了，这段代码将会被包含在编译中，输出将会是：

``` {.default}
I'm extra happy!
OK!
```

如果我们注释掉 `#define`，就像这样：

``` {.c}
//#define EXTRA_HAPPY
```

此时它就没有被定义，这段代码也不会被包含在编译过程中。输出将仅仅是：

``` {.default}
OK!
```

要记住这些判断都是在编译时发生的！代码实际上会根据条件被编译进去或者被移除掉。这与标准的 `if` 语句不同，标准 `if` 语句是在程序运行时被评估的。

### 如果未定义，使用 `#ifndef`

除了"如果定义了"，还有相反的情况："如果未定义"，即 `#ifndef`。我们可以根据某些东西是否被定义来改变先前示例的输出：

[i[`#ifndef` directive]<]
[i[`#endif` directive]<]

``` {.c .numberLines startFrom="8"}
#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#endif

#ifndef EXTRA_HAPPY
    printf("I'm just regular\n");
#endif
```

我们将在下一节看到一个更加简洁的做法。

将所有内容与头文件联系起来，我们已经看到如何通过使用预处理指令将头文件仅包含一次：

``` {.c}
#ifndef MYHEADER_H  // myheader.h 的第一行
#define MYHEADER_H

int x = 12;

#endif  // myheader.h 的最后一行
```

[i[`#ifndef` directive]>]
[i[`#endif` directive]>]

这个示例展示了宏是如何在文件间以及多个`#include`中持续存在的。如果还未定义，我们就去定义并编译整个头文件。

但下次它被包含时，我们发现`MYHEADER_H`已经被定义了，所以我们不再把头文件发送给编译器---它会被有效地删除。

### `#else`

[i[`#else`指令]<]

但这并不是我们所有的伎俩！还有一个`#else`可以加入混合。

[i[`#ifdef`指令]<]
[i[`#endif`指令]<]

让我们修改之前的例子：

``` {.c .numberLines startFrom="8"}
#ifdef EXTRA_HAPPY
    printf("我特别开心！\n");
#else
    printf("我就是普通开心\n");
#endif
```

[i[`#ifdef`指令]>]
[i[`#else`指令]>]
[i[`#endif`指令]>]

现在如果`EXTRA_HAPPY`未定义，它会进入`#else`从句并打印：

``` {.default}
我就是普通开心
```

### 如果-否则: `#elifdef`，`#elifndef`

[i[`#elifdef`指令]<]
[i[`#elifndef`指令]<]

这个功能是C23新增的！

如果你需要更复杂的东西怎么办呢？也许你需要一个if-else级联结构来正确构建你的代码？

幸运的是这些指令现成可用。我们可以使用`#elifdef`表示"else if defined"：

``` {.c}
#ifdef MODE_1
    printf("这是mode 1\n");
#elifdef MODE_2
    printf("这是mode 2\n");
#elifdef MODE_3
    printf("这是mode 3\n");
#else
    printf("这是其他模式\n");
#endif
```

[i[`#elifdef`指令]>]

另一方面，你可以使用`#elifndef`表示"else if not defined"。

[i[`#elifndef`指令]>]

### 通用条件: `#if`，`#elif`

[i[`#if`指令]<]
[i[`#elif`指令]<]

这与`#ifdef`和`#ifndef`指令非常类似，你也可以有一个`#else`，整个结构以`#endif`结束。

唯一的区别在于`#if`后面的常量表达式必须计算为真（非零），才会编译`#if`中的代码。因此，我们不再关心某事物是否被定义，而是想要一个计算结果为真的表达式。

``` {.c .numberLines}
#include <stdio.h>

#define HAPPY_FACTOR 1

int main(void)
{

#if HAPPY_FACTOR == 0
    printf("我不快乐！\n");
#elif HAPPY_FACTOR == 1
    printf("我很普通\n");
#else
    printf("我非常快乐！\n");
#endif

    printf("OK!\n");
}
```

[ ]表示未匹配的`#if`子句，编译器甚至不会看到那些行。对于以上代码，预处理器完成后，编译器看到的只有：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{

    printf("我很普通\n");

    printf("OK!\n");
}
```

这种方法还可用于快速注释掉大量代码^[你不能总是简单地将代码用`/*` `*/`进行注释，因为这种方式无法嵌套]。

``` {.c}
#if 0
    printf("所有这些代码"); /* 实际上被 */
    printf("注释掉了"); // 通过#if 0
#endif
```

如果你使用的是早于C23的编译器，不支持`#elifdef`或`#elifndef`指令，该怎么办？如何通过`#if`实现相同效果？也就是说，如果我想要这样：

``` {.c}
#ifdef FOO
    x = 2;
#elifdef BAR  // 潜在错误：在C23之前不支持
    x = 3;
#endif
```

该如何实现呢？

事实证明，预处理器有一个名为`defined`的运算符，我们可以在`#if`语句中使用。

这两者是等效的：

``` {.c}
#ifdef FOO
#if defined FOO
#if defined(FOO)   // 括号是可选的
```

和这些一样：

``` {.c}
#ifndef FOO
#if !defined FOO
#if !defined(FOO)   // 括号是可选的
```

请注意我们可以使用标准的逻辑非运算符 (`!`) 来表示"未定义"。

现在我们回到了 `#if` 领域，可以毫无顾忌地使用 `#elif`！

这个错误的代码：

``` {.c}
#ifdef FOO
    x = 2;
#elifdef BAR  // 潜在错误：C23 版本之前不支持
    x = 3;
#endif
```

可以替换为：

``` {.c}
#if defined FOO
    x = 2;
#elif defined BAR
    x = 3;
#endif
```

### 失去一个宏：`#undef`

如果你定义了某个东西但不再需要，可以用 `#undef` 取消定义。

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
#define GOATS

#ifdef GOATS
    printf("检测到了山羊！\n");  // 打印
#endif

#undef GOATS  // 移除 GOATS 的定义

#ifdef GOATS
    printf("又一次检测到了山羊！\n"); // 不打印
#endif
}
```

## 内置宏

标准定义了许多可以用于条件编译的内置宏。让我们来看看这些。

### 必需的宏

以下这些都已定义：

`__DATE__` 宏

`__TIME__` 宏

`__FILE__` 宏

`__LINE__` 宏

`__func__` 标识符

`__STDC_VERSION__` 宏
```

|宏|描述|
|-|-|
|`__DATE__`|编译日期---就是你编译这个文件的时候---格式为`月 日 年`|
|`__TIME__`|编译时间，格式为`时:分:秒`|
|`__FILE__`|包含当前文件名的字符串|
|`__LINE__`|出现该宏的文件中的行号|
|`__func__`|所在函数的名称，以字符串形式表示^[严格来说这并不是一个宏---它在技术上是一个标识符。但它是唯一预定义的标识符，感觉很像宏，所以我在这里包括了它。就像一个叛逆分子。]|
|[i[`__STDC__` 宏]]`__STDC__`|如果这是一个标准C编译器，则定义为`1`|
|[i[`__STDC_HOSTED__` 宏]]`__STDC_HOSTED__`|如果编译器是_面向主机的实现_，则为`1`^[一个面向主机的实现基本上意味着你正在运行完整的C标准，很可能在某种操作系统上。如果你在某种嵌入式系统中运行底层系统，那么你很可能是在一个_独立实现_上。]，否则为`0`|
|`__STDC_VERSION__`|C的版本，以`yyyymmL`形式表示的常量`long int`，例如`201710L`|

将这些放在一起。

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    printf("This function: %s\n", __func__);
    printf("This file: %s\n", __FILE__);
    printf("This line: %d\n", __LINE__);
    printf("Compiled on: %s %s\n", __DATE__, __TIME__);
    printf("C Version: %ld\n", __STDC_VERSION__);
}
```

[i[`__DATE__`宏]>]
[i[`__TIME__`宏]>]

在我的系统上输出为:

``` {.default}
This function: main
This file: foo.c
This line: 7
Compiled on: Nov 23 2020 17:16:27
C Version: 201710
```

`__FILE__`、`__func__` 和 `__LINE__` 对于向开发人员报告错误条件非常有用。`<assert.h>` 中的 `assert()` 宏使用这些来指出断言失败的代码位置。

#### `__STDC_VERSION__`s

如果你想知道，这里列出了不同 C 语言规范主要发布版本的版本号：

|发布|ISO/IEC 版本|`__STDC_VERSION__`|
|-|-|-|
|C89|ISO/IEC 9899:1990|未定义|
|**C89**|ISO/IEC 9899:1990/Amd.1:1995|`199409L`|
|**C99**|ISO/IEC 9899:1999|`199901L`|
|**C11**|ISO/IEC 9899:2011/Amd.1:2012|`201112L`|

请注意，该宏最初在 C89 中并不存在。

也请注意，计划是版本号将严格增加，因此你可以始终检查是否至少为 C99：

``` {.c}
#if __STDC_VERSION__ >= 1999901L
```

### 可选宏

您的实现可能定义了这些宏，或者可能没有定义。

| 宏              | 描述                                                                                                          |
|-----------------|---------------------------------------------------------------------------------------------------------------|
| [i[`__STDC_ISO_10646__` 宏]`__STDC_ISO_10646__` |若定义了此宏，则`wchar_t`保存Unicode值，否则保存其他数值|
| [i[`__STDC_MB_MIGHT_NEQ_WC__` 宏]`__STDC_MB_MIGHT_NEQ_WC__` |`1` 表示多字节字符中的值可能与宽字符中的值不完全对应|
| [i[`__STDC_UTF_16__` 宏]`__STDC_UTF_16__` |`1` 表示系统在 `char16_t` 类型中使用UTF-16编码|
| [i[`__STDC_UTF_32__` 宏]`__STDC_UTF_32__` |`1` 表示系统在 `char32_t` 类型中使用UTF- 32编码|
| [i[`__STDC_ANALYZABLE__` 宏]`__STDC_ANALYZABLE__` |`1` 表示代码可分析^[好吧，我知道这个回答有点含糊。基本上有一个可选的扩展，编译器可以实现，同意限制某些类型的未定义行为，使得C代码更适合静态代码分析。不太可能需要使用这个。]|
| [i[`__STDC_IEC_559__` 宏]`__STDC_IEC_559__` |若支持IEEE-754（也称为IEC 60559）浮点数，则返回 `1` |
| [i[`__STDC_IEC_559_COMPLEX__` 宏]`__STDC_IEC_559_COMPLEX__` |若支持IEC 60559复数浮点数，则返回 `1` |
| [i[`__STDC_LIB_EXT1__` 宏]`__STDC_LIB_EXT1__` |若此实现支持多种“安全”的备用标准库函数（以 `_s` 后缀结尾命名），则返回 `1`|
| [i[`__STDC_NO_ATOMICS__` 宏]`__STDC_NO_ATOMICS__` |若此实现**不**支持 `_Atomic` 或 `<stdatomic.h>`，则返回 `1`|
| [i[`__STDC_NO_COMPLEX__` 宏]`__STDC_NO_COMPLEX__` |若此实现**不**支持复数类型或 `<complex.h>`，则返回 `1`|
| [i[`__STDC_NO_THREADS__` 宏]`__STDC_NO_THREADS__` |若此实现**不**支持 `<threads.h>`，则返回 `1`|
| [i[`__STDC_NO_VLA__` 宏]`__STDC_NO_VLA__` |若此实现**不**支持可变长度数组，则返回 `1`|

[Preprocessor-->预定义宏]

## 带参数的宏

[Preprocessor-->带参数的宏]

尽管宏比简单的替换功能更强大。你可以设置宏以接受替换的参数。

关于何时使用带参数的宏或函数经常会有疑问。简短回答: 使用函数。但在实际应用中和标准库中你会看到很多宏。人们倾向于将它们用于短小的数学操作，以及那些可能会因平台而异的特性。你可以为不同平台定义不同的关键字。

### 带单个参数的宏

我们从一个简单的平方数例子开始:

[`#define` directive]

``` {.c .numberLines}
#include <stdio.h>

#define SQR(x) x * x  // 不完全正确，但请谅解

int main(void)
{
    printf("%d\n", SQR(12));  // 144
}
```

这段代码的含义是"任何地方看到 `SQR` 与某个值，替换为该值与自身的乘积"。

所以第 7 行会被改写为:

``` {.c .numberLines startFrom="7"}
    printf("%d\n", 12 * 12);  // 144
```

C 会很自然地将它转换为 144。

但是在这个宏中我们犯了一个初级错误，这是需要避免的。

我们来看看。如果我们想计算 `SQR(3 + 4)`？嗯，$3+4=7$，所以我们希望计算 $7^2=49$。这就是答案：`49`。

让我们将其放入代码中并查看我们得到... 19？

``` {.c .numberLines startFrom="7"}
    printf("%d\n", SQR(3 + 4));  // 19!!??
```

发生了什么？

如果我们展开这个宏，我们会得到

``` {.c .numberLines startFrom="7"}
    printf("%d\n", 3 + 4 * 3 + 4);  // 19!
```

哎呀！因为乘法优先级更高，我们先做了 $4\times3=12$，然后得到 $3+12+4=19$。这不是我们想要的结果。

因此我们需要修复这个问题才能正确。

**这很常见，每次你创建一个带参数的数学宏时，你应该自动做这个！**

修复方法很简单：只需要加上一些括号！

``` {.c .numberLines startFrom="3"}
#define SQR(x) (x) * (x)   // 更好了...但还不够好！
```

现在我们的宏展开为：

``` {.c .numberLines startFrom="7"}
    printf("%d\n", (3 + 4) * (3 + 4));  // 49！耶！
```

实际上我们还是有同样的问题，如果我们附近有比乘法(`*`)优先级更高的运算符的话，问题可能会显现出来。

所以，安全、正确的方式是将整个宏放在额外的括号中，如下所示：

``` {.c .numberLines startFrom="3"}
#define SQR(x) ((x) * (x))   // 好！
```

当你创建一个数学宏时养成这个习惯，就不会出错。

### 带有多个参数的宏

你可以堆叠这些东西，如你所愿：

``` {.c}
#define TRIANGLE_AREA(w, h) (0.5 * (w) * (h))
```

让我们做一些使用二次公式解出$x$的宏。以防你头脑中没有它，对于这种形式的方程式：

$ax^2+bx+c=0$

你可以用二次公式解出$x$：

$x=\displaystyle\frac{-b\pm\sqrt{b^2-4ac}}{2a}$

这很疯狂。同时注意其中的正负号($\pm$)，表明实际上有两个解。

所以让我们为两个都制作宏：

``` {.c}
#define QUADP(a, b, c) ((-(b) + sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUADM(a, b, c) ((-(b) - sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
```

这让我们能进行一些数学计算。但让我们再定义一个，我们可以将其用作`printf()`的参数来打印出两个答案。

``` {.c}
//          宏                  替换
//      |-----------| |----------------------------|
#define QUAD(a, b, c) QUADP(a, b, c), QUADM(a, b, c)
```

这只是用逗号分隔的几个值---我们可以将其作为`printf()`的“组合”参数，像这样使用：

``` {.c}
printf("x = %f or x = %f\n", QUAD(2, 10, 5));
```

让我们将其整合到一些代码中：

``` {.c .numberLines}
#include <stdio.h>
#include <math.h>  // 用于 sqrt()

#define QUADP(a, b, c) ((-(b) + sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUADM(a, b, c) ((-(b) - sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUAD(a, b, c) QUADP(a, b, c), QUADM(a, b, c)

int main(void)
{
    printf("2*x^2 + 10*x + 5 = 0\n");
    printf("x = %f or x = %f\n", QUAD(2, 10, 5));
}
```

这会给我们以下输出：

``` {.default}
2*x^2 + 10*x + 5 = 0
x = -0.563508 or x = -4.436492
```

将这些值之一插入得到接近零的数值（有点偏差，因为数值并非精确）：

$2\times-0.563508^2+10\times-0.563508+5\approx0.000003$

### 带有可变参数的宏

[i[预处理器-->带有可变参数的宏]<]

还有一种方法可以将可变数量的参数传递给宏，只需在已知的命名参数后使用省略号(`...`)。当宏被展开时，所有额外的参数将在`__VA_ARGS__`宏中以逗号分隔的列表形式出现，并且可以从那里替换：

``` {.c .numberLines}
#include <stdio.h>

// 将前两个参数合并为一个数字，然后将其余参数作为逗号分隔列表：

#define X(a, b, ...) (10*(a) + 20*(b)), __VA_ARGS__

int main(void)
{
    printf("%d %f %s %d\n", X(5, 4, 3.14, "Hi!", 12));
}
```

第10行进行的替换将是：
```

``` {.c .numberLines startFrom="10"}
    printf("%d %f %s %d\n", (10*(5) + 20*(4)), 3.14, "Hi!", 12);
```

的输出是:

``` {.default}
130 3.140000 Hi! 12
```

你也可以通过在 `__VA_ARGS__` 前面加上 `#` 来将其 “字符串化”：

``` {.c}
#define X(...) #__VA_ARGS__

printf("%s\n", X(1,2,3));  // 输出 "1, 2, 3"
```

### 字符串化

你可以通过在替换文本中的参数前面加上 `#` 将任何参数转换为字符串。

例如，我们可以使用这个宏和`printf()`将任何内容打印为字符串：

``` {.c}
#define STR(x) #x

printf("%s\n", STR(3.14159));
```

在这种情况下，替换为：

``` {.c}
printf("%s\n", "3.14159");
```

让我们看看是否可以更大程度地使用这个方法，使得我们可以将任何 `int` 变量名传递给宏，并打印出其名称和值。

``` {.c .numberLines}
#include <stdio.h>

#define PRINT_INT_VAL(x) printf("%s = %d\n", #x, x)

int main(void)
{
    int a = 5;

    PRINT_INT_VAL(a);  // 打印 "a = 5"
}
```

在第9行，我们得到以下宏替换：

``` {.c .numberLines startFrom="9"}
    printf("%s = %d\n", "a", 5);
```

### 连接

我们还可以使用 `##` 将两个参数连接在一起。很有意思！

``` {.c}
#define CAT(a, b) a ## b

printf("%f\n", CAT(3.14, 1592));   // 3.141592
```

## 多行宏

如果使用反斜杠 (`\`) 转义换行符，可以将宏连接到多行。

让我们编写一个多行宏，该宏从 `0` 打印数字到传入参数的乘积。

``` {.c .numberLines}
#include <stdio.h>

#define PRINT_NUMS_TO_PRODUCT(a, b) do { \
    int product = (a) * (b); \
    for (int i = 0; i < product; i++) { \
        printf("%d\n", i); \
    } \
} while(0)

int main(void)
{
    PRINT_NUMS_TO_PRODUCT(2, 4);  // 输出从0到7的数字
}
```

向您的代码添加断言是捕捉您认为不应该发生的情况的好方法。C 提供了 `assert()` 功能。它检查一个条件，如果为假，程序会停止运行并告诉您断言失败的文件和行号。

但这还不够。

1. 首先，断言中无法指定额外的消息。
2. 其次，没有简单的开关来控制所有断言。

我们可以通过宏来解决第一个问题。

基本上，当我有这样的代码:

``` {.c}
ASSERT(x < 20, "x 必须小于 20");
```

我希望发生类似下面的事情（假设 `ASSERT()` 在 `foo.c` 的第220行）:

``` {.c}
if (!(x < 20)) {
    fprintf(stderr, "foo.c:220: assertion x < 20 失败: ");
    fprintf(stderr, "x 必须小于 20\n");
    exit(1);
}
```

我们可以通过 `__FILE__` 宏获取文件名，通过 `__LINE__` 获取行号。消息已经是一个字符串，但 `x < 20` 不是，所以我们需要用 `#` 将其转换为字符串。我们可以通过在行尾使用反斜杠转义创建多行宏。

``` {.c}
#define ASSERT(c, m) \
do { \
    if (!(c)) { \
        fprintf(stderr, __FILE__ ":%d: assertion %s 失败: %s\n", \
                        __LINE__, #c, m); \
        exit(1); \
    } \
} while(0)
```

（`__FILE__` 这样放在前面看起来有点奇怪，但请记住这是一个字符串文字，相邻的字符串文字会自动连接起来。另一方面，`__LINE__` 是一个 `int`。）

这很好用！如果我运行这段代码:

``` {.c}
int x = 30;

ASSERT(x < 20, "x 必须小于 20");
```

我会得到以下输出:

```
foo.c:23: assertion x < 20 失败: x 必须小于 20
```

非常好！

唯一剩下的问题是如何开关它，我们可以通过条件编译来实现。

以下是完整的示例：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

#define ASSERT_ENABLED 1

#if ASSERT_ENABLED
#define ASSERT(c, m) \
do { \
    if (!(c)) { \
        fprintf(stderr, __FILE__ ":%d: assertion %s failed: %s\n", \
                        __LINE__, #c, m); \
        exit(1); \
    } \
} while(0)
#else
#define ASSERT(c, m)  // 如果未启用，则为空宏
#endif

int main(void)
{
    int x = 30;

    ASSERT(x < 20, "x必须小于20");
}
```

这将输出：

``` {.default}
foo.c:23: assertion x < 20 failed: x must be under 20
```

## `#error` 指令

[i[`#error` 指令]<]

此指令会在编译器首次看到时导致错误。

通常，它用在条件语句内，以确保满足某些先决条件后才编译：

``` {.c}
#ifndef __STDC_IEC_559__
    #error 我需要 IEEE-754 浮点数才能编译。抱歉！
#endif
```

[i[`#error` 指令]>]
[i[`#warning` 指令]<]

有些编译器有一个非标准的辅助 `#warning` 指令，会输出一个警告但不会停止编译，但这并不在 C11 规范中。

[i[`#warning` 指令]>]

## `#embed` 指令

[i[`#embed` 指令]<]

<!-- Godbolt 演示：https://godbolt.org/z/Kb3ejE7q5 -->

C23 的新特性！

目前我的编译器还无法使用，所以对这一部分暂时保留疑虑！

概念很简单，你可以将文件中的字节作为整数常量包含进来，就好像直接键入一样。

例如，如果你有一个名为 `foo.bin` 的二进制文件，其中包含四个具有十进制值 11、22、33 和 44 的字节，你可以这样做：

``` {.c}
int a[] = {
#embed "foo.bin"
};
```

这等同于：

``` {.c}
int a[] = {11,22,33,44};
```

这是一种非常强大的方法，可以使用二进制数据初始化数组，而无需先将所有数据转换为代码---预处理器会为您完成！更典型的用例可能是一个包含要显示的小图像的文件，您不想在运行时加载。下面是另一个示例：

``` {.c}
int a[] = {
#embed <foo.bin>
};
```

如果使用尖括号，预处理器将在一系列特定于实现的位置查找文件，就像`#include`会做的那样。如果使用双引号且未找到资源，编译器将尝试将其视为您最后绝望的尝试使用尖括号查找文件。

`#embed`的工作方式类似于`#include`，它有效地在编译器看到它们之前将值粘贴在其中。这意味着您可以在各种地方使用它：

``` {.c}
return
#embed "somevalue.dat"
;
```

或者

``` {.c}
int x =
#embed "xvalue.dat"
;
```

现在---这些始终是字节吗？意味着它们的值范围从0到255，包括0和255？答案绝对默认是“是”，除非是“否”。

在技术上，元素的位宽将是`CHAR_BIT`位。 在您的系统上很可能是8，因此您将在值中得到0-255的范围（始终是非负的）。

而且，可能会有一种实现方式允许覆盖这一点，例如在命令行上或者用参数。

文件的位大小必须是元素大小的倍数。 也就是说，如果每个元素是8位，那么文件大小（以位为单位）必须是8的倍数。 在日常使用中，这是说每个文件都需要是整数字节数的一个令人困惑的方式... 当然了，事实上确实如此。 老实说，我甚至不确定我为什么要费力写这段话。 如果您真的是那么好奇，请阅读规范。

### `#embed` 参数

在 `#embed` 指令中，你可以指定各种不同的参数。以下是一个示例，使用了尚未介绍的 `limit()` 参数：

``` {.c}
int a[] = {
#embed "/dev/random" limit(5)
};
```

但如果你的某处已经定义了 `limit`，别担心，你可以在关键词周围加上 `__`，它会起到同样的作用：

``` {.c}
int a[] = {
#embed "/dev/random" __limit__(5)
};
```

那么...这个 `limit` 是什么东西呢？

### `limit()` 参数

通过这个参数，你可以指定嵌入的元素数量上限。

这是一个最大值，而非绝对值。如果要嵌入的文件比指定的上限要短，则只会导入这么多字节。

上面提到的 `/dev/random` 示例是这个参数的一个典型应用场景---在 Unix 中，这是一个_字符设备文件_，会返回一个无限流的相当随机的数字序列。

嵌入无限字节数会对你的内存产生负担，所以 `limit` 参数给你一种在一定数量后停止的方式。

最后，你也可以在你的 `limit` 中使用 `#define` 宏，如果你感兴趣的话。

### `if_empty` 参数

[i[`if_empty()` 嵌入参数]<]

这个参数定义了如果文件存在但不包含任何数据时，嵌入结果应该是什么。假设文件 `foo.dat` 包含一个数值为 123 的单个字节。如果我们这样做：

``` {.c}
int x = 
#embed "foo.dat" if_empty(999)
;
```

我们会得到：

``` {.c}
int x = 123;   // 当 foo.dat 包含一个字节为 123 时
```

但如果文件 `foo.dat` 是零字节长（即不包含任何数据且为空）呢？如果是这种情况，结果会变为：

``` {.c}
int x = 999;   // 当 foo.dat 是空的时候
```

值得注意的是，如果将 `limit` 设置为 `0`，那么 `if_empty` 将始终被替代。换句话说，零限制实际上意味着文件为空。

无论 `foo.dat` 中的内容是什么，都将始终发出 `x = 999`：

``` {.c}
int x = 
#embed "foo.dat" limit(0) if_empty(999)
;
```

[i[`if_empty()` 嵌入参数]>]

### `prefix()` 和 `suffix()` 参数

[i[`prefix()` 嵌入参数]<]
[i[`suffix()` 嵌入参数]<]

这是在嵌入数据前添加一些数据的方法。

请注意，这些仅影响非空数据！如果文件为空，`prefix` 和 `suffix` 都不会产生任何效果。

以下是一个示例，我们将嵌入三个随机数字，但用 `11,` 在数字前缀，并用 `,99` 在数字后缀：

``` {.c}
int x[] = {
#embed "/dev/urandom" limit(3) prefix(11,) suffix(,99)
};
```

示例结果：

``` {.c}
int x[] = {11,135,116,220,99};
```

并不要求同时使用 `prefix` 和 `suffix`。您可以同时使用、一个使用、另一个使用，或者两者都不使用。

我们可以利用这些参数仅应用于非空文件的特性，以优雅的方式展示，正如从规范中无耻地拿走的以下示例所示。

假设我们有一个名为 `foo.dat` 的文件中有一些数据。我们希望使用这些数据初始化一个数组，然后我们希望在数组上有一个以零元素结尾的后缀。

没有问题，对吧？

``` {.c}
int x[] = {
#embed "foo.dat" suffix(,0)
};
```

如果 `foo.dat` 中有 11、22 和 33，我们将得到：

``` {.c}
int x[] = {11,22,33,0};
```

但等等！如果 `foo.dat` 是空的呢？那么我们会得到：

``` {.c}
int x[] = {};
```

这不太好。

不过，我们可以这样修复：

``` {.c}
int x[] = {
#embed "foo.dat" suffix(,)
    0
};
```

因为如果文件为空，`suffix` 参数将被省略，所以结果会变成：

``` {.c}
int x[] = {0};
```

这是可以接受的。

```c
    int random_nums[] = {
#if __has_embed("/dev/urandom")
    #embed "/dev/urandom" limit(5)
#elif __has_embed("myrandoms.dat")
    #embed "myrandoms.dat" limit(5)
#else
    140,178,92,167,120
#endif
    };
```

在这里，有一段代码将从随机数生成器字符设备中获取5个随机数。如果该设备不存在，它将尝试从文件`myrandoms.dat`中获取。如果该文件也不存在，它将使用一些硬编码的值。

```c
|`__has_embed()` 结果|描述|
|-|-|
|`__STDC_EMBED_NOT_FOUND__`|如果文件未找到。|
|`__STDC_EMBED_FOUND__`|如果文件被找到且非空。|
|`__STDC_EMBED_EMPTY`|如果文件被找到且为空。|
```

我有充分的理由相信`__STDC_EMBED_NOT_FOUND__`是`0`，而其他则不是零（因为在提案中是暗示的，且在逻辑上也是有意义的），但在这个草案规范中，我很难找到相关的条文。

```c
#embed "foo.bin" limit(12) frotz(lamp)
```

这些参数通常可能会有一个前缀，以帮助进行命名空间处理：

```c
#embed "foo.bin" limit(12) fmc::frotz(lamp)
```

检测这些内容是否可用再使用可能是明智的做法，幸运的是我们可以使用`__has_embed`来帮助我们。

通常，`__has_embed()`会告诉我们文件是否存在。但是——这里有趣的一点是——如果其他参数也不受支持，它也会返回false！

所以，如果我们给它一个我们**知道**存在的文件以及我们想要测试存在性的参数，它会有效地告诉我们该参数是否被支持。

不过哪个文件**一直**存在呢？原来我们可以使用`__FILE__`宏，它会扩展为引用它的源文件的名称！那个文件**必须**存在，否则在龙生九子的问题上就有严重错误。

让我们测试一下 `frotz` 参数，看看我们能否使用它：

``` {.c}
#if __has_embed(__FILE__ fmc::frotz(lamp))
    puts("fmc::frotz(lamp) 受支持！");
#else
    puts("fmc::frotz(lamp) 不受支持！");
#endif
```

### 嵌入多字节值

要插入一些`int`值，而不是单个字节呢？还有嵌入文件中的多字节值是怎么样的呢？

这不是C23标准支持的内容，但将来可能会为其定义实现扩展。

[i[`#embed` directive]>]

## `#pragma` 指令 {#pragma}

[i[`#pragma` directive]<]

这是一个很时髦的指令，短为“实用”。您可以用它来做... 嗯，任何您的编译器支持您使用的事情。

基本上，您唯一会将其添加到代码中的场合就是如果文档告诉您这样做时。

### 非标准 Pragma

[i[`#pragma` directive-->nonstandard pragmas]<]

这里有一个非标准的 `#pragma` 使用示例，可以让编译器使用多线程并行执行 `for` 循环（如果编译器支持 [fl[OpenMP|https://www.openmp.org/]] 扩展）:

``` {.c}
#pragma omp parallel for
for (int i = 0; i < 10; i++) { ... }
```

全球各地都有各种各样的 `#pragma` 指令文档记录着。

所有未被识别的 `#pragma` 指令都会被编译器忽略。

[i[`#pragma` directive-->nonstandard pragmas]>]

### 标准 Pragmas

也有一些标准的指令，它们以 `STDC` 开头，并且遵循相同的形式：

``` {.c}
#pragma STDC pragma_name on-off
```

`on-off` 部分可以是 `ON`、`OFF` 或 `DEFAULT`。

而 `pragma_name` 可以是以下之一：

[i[`FP_CONTRACT` pragma]<]
[[`CX_LIMITED_RANGE` pragma]<]

|Pragma 名称|描述|
|-|-|
|`FP_CONTRACT`|允许将浮点表达式合并为单个操作，以避免多个操作可能导致的舍入错误。|
|`FENV_ACCESS`|如果打算访问浮点状态标志，则设置为 `ON`。如果是 `OFF`，编译器可能进行导致标志中的值不一致或无效的优化。|
|`CX_LIMITED_RANGE`|设置为 `ON` 以允许编译器在执行复杂算术时跳过溢出检查。默认为 `OFF`。|

例如：

``` {.c}
#pragma STDC FP_CONTRACT OFF
#pragma STDC CX_LIMITED_RANGE ON
```

[i[`FP_CONTRACT` pragma]>]

至于 `CX_LIMITED_RANGE`，规范指出：

> 该指令的目的是允许实现使用以下公式：
>
> $(x+iy)\times(u+iv) = (xu-yv)+i(yu+xv)$
>
> $(x+iy)/(u+iv) = [(xu+yv)+i(yu-xv)]/(u^2+v^2)$
>
> $|x+iy|=\sqrt{x^2+y^2}$
>
> 在这些公式中，程序员可以确定其安全性。

### `_Pragma` 操作符

这是在宏中声明编译指示的另一种方式。

它们是等价的：

``` {.c}
#pragma "不必要的" 引号
_Pragma("\"不必要的\" 引号")
```

在需要的情况下，可以在宏中使用这个：

``` {.c}
#define PRAGMA(x) _Pragma(#x)
```

可以在宏中使用 `_Pragma` 操作符，如果需要的话。

## `#line` 指令

这允许您覆盖 `__LINE__` 和 `__FILE__` 的值。如果你需要的话。

我从未想过要这样做，但在 K&R2 中，他们写道：

> 为了其他生成 C 程序的预处理器的利益 [...]

所以也许有那个。

要将行号覆盖为，比如 300：

``` {.c}
#line 300
```

`__LINE__` 将从那里继续计数。

要覆盖行号和文件名：

``` {.c}
#line 300 "newfilename"
```

## 空指令

独立一行的 `#` 被预处理器忽略。现在，坦率地说，我不知道这的用例是什么。

我看到类似这样的例子：

``` {.c}
#ifdef FOO
    #
#else
    printf("Something");
#endif
```

这只是表面的；带有独立 `#` 的行可以被删除，不会产生任何不良影响。

或者可能是为了美学的一致性，像这样：

``` {.c}
#
#ifdef FOO
    x = 2;
#endif
#
#if BAR == 17
    x = 12;
#endif
#
```

但是，就美学而言，这很丑陋。

另一篇帖子提到了去除评论---在 GCC 中，`#` 后的评论不会被编译器看到。我不怀疑，但规范似乎没有说这是标准行为。

我的理由搜索没有取得太多成果。所以我要说这是一些经典的 C 专业知识。