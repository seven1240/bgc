<!-- Beej的C指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# C 预处理器

在你的程序被编译之前，它实际上会经历一个叫做“_预处理_”的阶段。几乎就像在C语言的“_顶层_”运行着另一种语言一样。它会输出C代码，然后再进行编译。

我们已经在一定程度上见过这个过程，比如 `#include`！那就是C 预处理器！当它看到这个指令时，它会直接在那里包含指定的文件，就好像你自己在这里输入了一样。然后编译器会构建整个过程。

但实际上它比仅能包含文件要强大得多。你可以定义会被替换的 _宏_ ... 甚至可以定义带参数的宏!

## `#include`

[i[`#include` 指令]<]

让我们从我们已经见过很多次的这个开始。这当然是包含其他源文件到你的源文件中的一种方式。在使用头文件时非常常见。

尽管规范允许使用`#include`进行各种行为，但我们将采取更加实用的方法，讨论我见过的每个系统上的工作方式。

我们可以将头文件分为两类：系统头文件和本地头文件。像 `stdio.h`、 `stdlib.h` 、`math.h` 之类的内置头文件，你可以使用尖括号来包含:

``` {.c}
#include <stdio.h>
#include <stdlib.h>
```

尖括号告诉C，“嘿，不要在当前目录中查找这个头文件---请在整个系统范围内的包含目录中查找。”

[i[`#include` 指令-->本地文件]<]

这当然意味着必须有一种方式来包含当前目录中的本地文件。而事实上有：使用双引号:

``` {.c}
#include "myheader.h"
```

或者你也很可能使用斜杠和点来查找相对目录中的文件，像这样：

``` {.c}
#include "mydir/myheader.h"
#include "../someheader.py"
```

在 `#include` 语句中，不要使用反斜杠（`\`）作为路径分隔符！这是未定义的行为！只能使用正斜杠（`/`），即使在 Windows 上也是如此。

简而言之，对于系统包含使用尖括号（`<` 和 `>`），对于个人包含使用双引号（`"`）。

## 简单宏

宏是一个标识符，在编译器看到之前就会被扩展为另一段代码。可以将其看作一个占位符---当预处理器看到这些标识符时，它会将其替换为你定义的另一个值。

我们通过`#define`（通常读作 “井号定义”）来实现。这里是一个示例：

``` {.c .numberLines}
#include <stdio.h>

#define HELLO "Hello, world"
#define PI 3.14159

int main(void)
{
    printf("%s, %f\n", HELLO, PI);
}
```

在第 3 和第 4 行，我们定义了几个宏。在代码的其他地方出现这些宏（第 8 行），它们将被替换为定义的值。

从 C 编译器的角度来看，它就好像我们实际上写的是这样：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    printf("%s, %f\n", "Hello, world", 3.14159);
}
```

看到了吗？ `HELLO` 被替换为 `"Hello, world"`，`PI` 被替换为 `3.14159`。从编译器的角度看，就好像这些值直接出现在代码中一样。

请注意，宏本身实际上没有特定类型。它们实际上只是被替换为它们`#define` 的内容。如果最终的 C 代码是无效的，编译器将会报错。

你也可以定义一个没有值的宏：

``` {.c}
#define EXTRA_HAPPY
```

在这种情况下，宏是存在且被定义的，但被定义为什么也没有。因此，它在文本中的任何位置都将被替换为什么也没有。我们稍后会看到此用法。

尽管技术上不要求，但惯例上将宏名称写成 `ALL_CAPS`。

总的来说，这为您提供了一种定义有效全局且可在任何地方使用的常量值的方式。即使在 `const` 变量无法使用的地方，例如在 `switch` `case` 和固定数组长度中也可以使用。

尽管如此，网上存在一场争论，即在一般情况下，是否使用具有类型的 `const` 变量比 `#define` 宏更好。

它还可以用于替换或修改关键字，这是 `const` 完全陌生的概念，尽管这种做法应该节制使用。

## 条件编译

将预处理器设定为决定是否将某些代码块提交给编译器，或在编译之前将其完全移除是可能的。

我们通过基本上将代码包装在条件块中来做到这一点，类似于 `if`-`else` 语句。

### 如果被定义，则 `#ifdef` 和 `#endif`

首先，让我们尝试编译特定的代码，取决于宏是否被定义。

``` {.c .numberLines}
#include <stdio.h>

#define EXTRA_HAPPY

int main(void)
{

#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#endif

    printf("OK!\n");
}
```

在这个示例中，我们定义了 `EXTRA_HAPPY`（虽然没有赋值，但是被定义了），然后在第8行我们使用 `#ifdef` 指令来检查它是否被定义。如果被定义了，随后的代码直到 `#endif` 都会被包含进来。

因为它被定义了，这段代码将会被包含在编译中，输出将会是：

``` {.default}
I'm extra happy!
OK!
```

如果我们将 `#define` 注释掉，如下所示：

``` {.c}
//#define EXTRA_HAPPY
```

那么它就不会被定义，代码也不会被包含在编译中。输出将只是：

``` {.default}
OK!
```

重要的是要记住，这些决定发生在编译时！代码实际上会根据条件被编译或移除。这与在程序运行时评估的标准 `if` 语句形成对比。

### 未定义时，`#ifndef`

还有一种负面意义的"如果定义了"： "如果未定义"，或者 `#ifndef`。我们可以改变之前的示例来根据某个东西是否被定义来输出不同的内容：

``` {.c .numberLines startFrom="8"}
#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#endif

#ifndef EXTRA_HAPPY
    printf("I'm just regular\n");
#endif
```

我们将在下一节看到一个更清晰的方式来做到这一点。

将所有这些联系回头文件，我们已经看到了如何通过将它们包裹在预处理器指令中，让头文件只被包含一次：

``` {.c}
#ifndef MYHEADER_H  // myheader.h 的第一行
#define MYHEADER_H

int x = 12;

#endif  // myheader.h 的最后一行
```

这展示了一个宏如何在不同文件和多次`#include`中持久存在。如果它还没有被定义，让我们定义它并编译整个头文件。

但是下一次它被包含时，我们会发现`MYHEADER_H`已经被定义，所以我们不会将头文件发送给编译器---它会被有效地移除。

### `#else`

但这不是我们唯一能做的事情！还有一个`#else`可以加入。

让我们修改上一个例子:

``` {.c .numberLines startFrom="8"}
#ifdef EXTRA_HAPPY
    printf("I'm extra happy!\n");
#else
    printf("I'm just regular\n");
#endif
```

现在如果`EXTRA_HAPPY`没有被定义，它会进入`#else`子句并打印:

``` {.default}
I'm just regular
```

### Else-If: `#elifdef`, `#elifndef`

这个功能是 C23 中的新功能！

但是如果你需要更复杂的内容呢？也许你需要一个 if-else 级联结构才能正确搭建你的代码?

幸运的是我们有这些指令可用。我们可以使用`#elifdef`代表"else if defined":

``` {.c}
#ifdef MODE_1
    printf("This is mode 1\n");
#elifdef MODE_2
    printf("This is mode 2\n");
#elifdef MODE_3
    printf("This is mode 3\n");
#else
    printf("This is some other mode\n");
#endif
```

另一方面，你可以使用`#elifndef`代表"else if not defined"。

### General Conditional: `#if`, `#elif`

这和`#ifdef`和`#ifndef`指令非常类似，你也可以有一个`#else`，并以`#endif`结尾。

唯一的区别在于 `#if` 后面的常量表达式必须为真（非零）以便编译 `#if` 中的代码。因此，我们不再是查看某个是否被定义，而是要求一个表达式求值为真。

``` {.c .numberLines}
#include <stdio.h>

#define HAPPY_FACTOR 1

int main(void)
{

#if HAPPY_FACTOR == 0
    printf("I'm not happy!\n");
#elif HAPPY_FACTOR == 1
    printf("I'm just regular\n");
#else
    printf("I'm extra happy!\n");
#endif

    printf("OK!\n");
}
```

再次强调，对于未匹配的 `#if` 子句，编译器甚至都看不到那些行。对于上面的代码，预处理完成后，编译器看到的只有：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{

    printf("I'm just regular\n");

    printf("OK!\n");
}
```

有时候这样用来快速注释掉大量代码^[你不能总是简单地用 `/*` `*/` 注释包裹代码，因为这样是无法嵌套的。]。

如果你在一个 C23 之前的编译器上，并且没有 `#elifdef` 或 `#elifndef` 指令支持，该如何实现相同的效果呢？即，如果我想要这样：

``` {.c}
#ifdef FOO
    x = 2;
#elifdef BAR  // 潜在错误：在 C23 之前不被支持
    x = 3;
#endif
```

怎么做呢？

事实证明预处理器有一个名为 `defined` 的运算符可以和 `#if` 语句一起使用。

以下两种写法是等效的：

[i[`#if defined`指令]<]
``` {.c}
#ifdef FOO
#if defined FOO
#if defined(FOO)   // 括号可选
```

这些也是：

``` {.c}
#ifndef FOO
#if !defined FOO
#if !defined(FOO)   // 括号可选
```

请注意，我们可以使用标准逻辑非运算符 (`!`) 表示 "未定义"。

现在我们回到了 `#if` 领域，可以毫不顾忌地使用 `#elif` 了！

这段错误的代码：

``` {.c}
#ifdef FOO
    x = 2;
#elifdef BAR  // 潜在错误：C23 之前不支持
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

[i[`#if defined`指令]>]
[i[`#if`指令]>]
[i[条件编译]>]

### 丢弃宏：`#undef`

[i[`#undef`指令]<]

如果你定义了某个东西但不再需要它，可以使用 `#undef` 取消定义它。

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
#define GOATS

#ifdef GOATS
    printf("检测到山羊！\n");  // 输出
#endif

#undef GOATS  // 取消定义 GOATS

#ifdef GOATS
    printf("再次检测到山羊！\n"); // 不输出
#endif
}
```

[i[`#undef`指令]>]

## 内置宏

[i[预处理器-->预定义宏]<]

标准定义了许多内置宏，可以用于条件编译中进行测试和使用。让我们在这里看看这些。

### 必需的宏

这些都已定义：

[i[`__DATE__`宏]<]
[i[`__TIME__`宏]<]
[i[`__FILE__`宏]<]
[i[`__LINE__`宏]<]
[i[`__func__` 标识符]<]
[i[`__STDC_VERSION__`宏]<]

|宏|描述|
|-|-|
|`__DATE__`|编译日期---就像在编译此文件时一样---以 `Mmm dd yyyy` 格式|
|`__TIME__`|编译时间以 `hh:mm:ss` 格式|
|`__FILE__`|包含此文件名称的字符串|
|`__LINE__`|此宏出现在的文件的行号|
|`__func__`|此函数名称，作为字符串^[这实际上不是一个宏---严格来说是一个标识符。但它是唯一的预定义标识符，感觉非常类似宏，所以包括在这里。有点像叛逆。]|
|`__STDC__`|如果是标准 C 编译器则定义为 `1`|
|`__STDC_HOSTED__`|如果编译器为 _托管实现_ 则为 `1`^[托管实现基本上意味着您正在运行完整的 C 标准，很可能在某种操作系统上。如果在某种嵌入式系统的裸机上运行，则可能是 _独立实现_。]，否则为 `0`|
|`__STDC_VERSION__`|C 的版本，以 `yyyymmL` 形式的常量 `long int`，例如 `201710L`|

让我们把这些放在一起。

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

`__FILE__`、`__func__` 和 `__LINE__` 在向开发人员报告错误状况时特别有用。`<assert.h>` 中的 `assert()` 宏使用这些信息来指出代码中断言失败的位置。

`__STDC_VERSION__`

如果你感到好奇，以下是 C 语言规范不同主要版本的版本号：

|发布版|ISO/IEC 版本|`__STDC_VERSION__`|
|-|-|-|
|C89|ISO/IEC 9899:1990|未定义|
|**C89**|ISO/IEC 9899:1990/Amd.1:1995|`199409L`|
|**C99**|ISO/IEC 9899:1999|`199901L`|
|**C11**|ISO/IEC 9899:2011/Amd.1:2012|`201112L`|

请注意，该宏在 C89 中原本并不存在。

另外要注意的是，计划是版本号会严格递增，所以你可以始终检查是否至少是 C99：

``` {.c}
#if __STDC_VERSION__ >= 199901L
```

### 可选宏

你的实现可能也定义了这些宏，也可能没有。

|宏|描述|
|-|-|
|`__STDC_ISO_10646__`|如果定义了，`wchar_t`保存Unicode值，否则保存其他值|
|`__STDC_MB_MIGHT_NEQ_WC__`|`1`表示多字节字符的值可能不完全映射到宽字符的值|
|`__STDC_UTF_16__`|`1`表示系统在`char16_t`类型中使用UTF-16编码|
|`__STDC_UTF_32__`|`1`表示系统在`char32_t`类型中使用UTF-32编码|
|`__STDC_ANALYZABLE__`|`1`表示代码可被分析|
|`__STDC_IEC_559__`|如果支持IEEE-754（又称IEC 60559）浮点数，则为`1`|
|`__STDC_IEC_559_COMPLEX__`|如果支持IEC 60559复杂浮点数，则为`1`|
|`__STDC_LIB_EXT1__`|如果此实现支持各种“安全”的替代标准库函数（函数名有“_s”后缀）则为`1`|
|`__STDC_NO_ATOMICS__`|如果此实现**不**支持`_Atomic`或`<stdatomic.h>`，则为`1`|
|`__STDC_NO_COMPLEX__`|如果此实现**不**支持复数类型或`<complex.h>`，则为`1`|
|`__STDC_NO_THREADS__`|如果此实现**不**支持`<threads.h>`，则为`1`|
|`__STDC_NO_VLA__`|如果此实现**不**支持可变长度数组，则为`1`|

## 带参数的宏

带参数的宏比简单的替换更强大。您可以设置它们接受参数，然后进行替换。

通常会有一个问题，即何时使用带参数的宏与函数。简短的答案是：使用函数。但是您会在很多地方和标准库中看到许多宏。人们倾向于将它们用于简短的数学计算，以及可能因平台而异的功能。您可以为一个平台或另一个平台定义不同的关键字。

### 带一个参数的宏

让我们从一个简单的平方数的宏开始：

```c
#include <stdio.h>

#define SQR(x) x * x  // 不完全正确，但请忍耐

int main(void)
{
    printf("%d\n", SQR(12));  // 144
}
```

这意味着“在任何地方看到带有某个值的 `SQR`，都将其替换为该值乘以自身”。

因此，第7行将被更改为：

```c
    printf("%d\n", 12 * 12);  // 144
```

C语言轻松转换为144。

但是我们在这个宏中犯了一个初级错误，我们需要避免这种错误。

让我们来看看。如果我们想计算 `SQR(3 + 4)` 呢？嗯，$3+4=7$，所以我们必须要计算 $7^2=49$。就是这个，`49`---最后的答案。

让我们将它放入我们的代码中，看看我们得到了... 19？

```c
    printf("%d\n", SQR(3 + 4));  // 19!!??
```

发生了什么？

如果我们按照宏展开，我们得到

```c
    printf("%d\n", 3 + 4 * 3 + 4);  // 19!
```

糟糕！由于乘法优先级较高，我们首先进行 $4\times3=12$ 的计算，然后得到 $3+12+4=19$。这不是我们想要的结果。

因此，我们必须修复这个问题，使其正确。

**这是如此常见，以至于您应该在每次制作有参数的数学宏时自动执行它！**

修复方法很简单：只需添加一些括号！

``` {.c .numberLines startFrom="3"}
#define SQR(x) (x) * (x)   // 更好…但还不够好！
```

现在我们的宏扩展为：

``` {.c .numberLines startFrom="7"}
    printf("%d\n", (3 + 4) * (3 + 4));  // 49！哇哈哈！
```

但实际上我们仍然有同样的问题，如果我们附近有一个比乘法(`*`)优先级更高的运算符，这个问题可能会显现出来。

因此，将宏放在额外的括号中是安全且正确的方法，如下所示：

``` {.c .numberLines startFrom="3"}
#define SQR(x) ((x) * (x))   // 好！
```

只需养成做数学宏时这样做的习惯，就不会出错。

### 带有多个参数的宏

您可以叠加这些东西多少次都可以：

``` {.c}
#define TRIANGLE_AREA(w, h) (0.5 * (w) * (h))
```

让我们做一些使用二次方程式求解 $x$ 的宏。以防您没有记住，对于以下形式的方程：

$ax^2+bx+c=0$

您可以使用二次方程式求解 $x$：

$x=\displaystyle\frac{-b\pm\sqrt{b^2-4ac}}{2a}$

这很疯狂。还请注意方程式中的正负号($\pm$)，表明实际上有两个解。

因此，让我们分别制定两个解的宏：

``` {.c}
#define QUADP(a, b, c) ((-(b) + sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
#define QUADM(a, b, c) ((-(b) - sqrt((b) * (b) - 4 * (a) * (c))) / (2 * (a)))
```

这样我们就得到了一些数学。但让我们再定义一个，我们可以将其用作 `printf()` 的参数，以打印两个答案。

``` {.c}
//          宏                 替换部分
//      |-----------| |----------------------------|
#define QUAD(a, b, c) QUADP(a, b, c), QUADM(a, b, c)
```

这只是一对由逗号分隔的几个值---我们可以将其作为类似于`printf()`的“组合”参数使用，像这样：

``` {.c}
printf("x = %f or x = %f\n", QUAD(2, 10, 5));
```

让我们把它整合到一些代码中：

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

这给出了下面的输出：

``` {.default}
2*x^2 + 10*x + 5 = 0
x = -0.563508 or x = -4.436492
```

插入其中任何一个值都将使我们得到大致为零的结果（因为数字不精确）：

$2 \times -0.563508^2 + 10 \times -0.563508 + 5 \approx 0.000003$

### 具有可变参数的宏

[i[Preprocessor-->macros with variable arguments]<]

还有一种方法可以使宏传递可变数量的参数，即在已知的命名参数后加上省略号(`...`)。当宏被展开时，所有额外参数将作为逗号分隔列表出现在`__VA_ARGS__`宏中，并可以从那里替换：

``` {.c .numberLines}
#include <stdio.h>

// 将前两个参数组合成一个数字，然后对其余参数进行逗号列表处理：

#define X(a, b, ...) (10 * (a) + 20 * (b)), __VA_ARGS__

int main(void)
{
    printf("%d %f %s %d\n", X(5, 4, 3.14, "Hi!", 12));
}
```

第10行发生的替换将是：
```

``` {.c .numberLines startFrom="10"}
    printf("%d %f %s %d\n", (10*(5) + 20*(4)), 3.14, "Hi!", 12);
```

输出为：

``` {.default}
130 3.140000 Hi! 12
```

您也可以通过在`__VA_ARGS__`前面加上`#`来将其“字符串化”：

``` {.c}
#define X(...) #__VA_ARGS__

printf("%s\n", X(1,2,3));  // 打印出 "1, 2, 3"
```

### 字符串化

已经在上面提到过，您可以通过在替换文本中的参数前加上`#`来将任何参数转换为字符串。

例如，我们可以使用这个宏和`printf()`将任何内容打印为字符串：

``` {.c}
#define STR(x) #x

printf("%s\n", STR(3.14159));
```

在这种情况下，替换会变为：

``` {.c}
printf("%s\n", "3.14159");
```

让我们看看是否可以更有效地利用这一点，以便可以将任何`int`变量名传递给一个宏，并使其打印出其名称和值。

``` {.c .numberLines}
#include <stdio.h>

#define PRINT_INT_VAL(x) printf("%s = %d\n", #x, x)

int main(void)
{
    int a = 5;

    PRINT_INT_VAL(a);  // 打印出 "a = 5"
}
```

在第9行，我们得到以下宏替换：

``` {.c .numberLines startFrom="9"}
    printf("%s = %d\n", "a", 5);
```

### 连接

我们也可以使用`##`将两个参数连接在一起。有趣的时刻！

``` {.c}
#define CAT(a, b) a ## b

printf("%f\n", CAT(3.14, 1592));   // 3.141592
```

## 多行宏

如果您使用反斜杠(`\`)转义换行符，那么可以将宏延续到多行。

让我们编写一个多行宏，该宏打印从`0`到传入的两个参数的乘积的数字。

一些要注意的地方：

- 每行末尾都有转义符号，除了最后一行，表示宏还在继续。
- 整个部分都包裹在一个带花括号的 `do`-`while(0)` 循环中。

后一点可能有点奇怪，但这是为了吸收程序员在宏后面加上分号这一行为。

起初，我认为只使用花括号就足够了，但有一个情况会出错，如果程序员在宏后面加上分号。下面是该案例：

看起来很简单，但如果没有语法错误，它将无法构建：

你看到了吗？

让我们看看扩展：

`;` 结束了 `if` 语句，因此 `else` 就变成了非法的。

因此，在多行宏周围加上一个 `do`-`while(0)`。

将断言添加到代码中是捕捉你认为不应发生的条件的好方法。 C提供了`assert()`功能。它检查一个条件，如果条件为假，程序会报错告诉你断言失败的文件和行号。

但这还不够。

1. 首先，你不能为assert指定附加消息。
2. 其次，没有一个简单的开关可以控制所有的断言。

我们可以用宏来解决第一个问题。

基本上，当我有这段代码时：

``` {.c}
ASSERT(x < 20, "x must be under 20");
```

我希望类似这样的事情发生（假设`ASSERT()`在`foo.c`的第220行）：

``` {.c}
if (!(x < 20)) {
    fprintf(stderr, "foo.c:220: assertion x < 20 failed: ");
    fprintf(stderr, "x must be under 20\n");
    exit(1);
}
```

我们可以使用`__FILE__`宏得到文件名，使用`__LINE__`得到行号。消息已经是一个字符串，但`x < 20`不是，因此我们需要用`#`来将其转换为字符串。我们可以通过在行尾使用反斜杠转义来创建多行宏。

``` {.c}
#define ASSERT(c, m) \
do { \
    if (!(c)) { \
        fprintf(stderr, __FILE__ ":%d: assertion %s failed: %s\n", \
                        __LINE__, #c, m); \
        exit(1); \
    } \
} while(0)
```

（`__FILE__`以那种方式放在前面看起来有点奇怪，但请记住它是一个字符串字面值，并且相邻的字符串字面值会自动连接起来。另一方面，`__LINE__`只是一个`int`。）

这样就可以了！如果我运行这个：

``` {.c}
int x = 30;

ASSERT(x < 20, "x must be under 20");
```

我会得到这个输出：

```
foo.c:23: assertion x < 20 failed: x must be under 20
```

非常不错！

唯一剩下的就是找到一种方法来开关它，我们可以通过条件编译来实现。

下面是完整的示例:

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
#define ASSERT(c, m)  // 如果未启用则为空宏
#endif

int main(void)
{
    int x = 30;

    ASSERT(x < 20, "x 必须小于 20");
}
```

执行后的输出为:

``` {.default}
foo.c:23: assertion x < 20 failed: x must be under 20
```

## `#error` 指令

[i[`#error` directive]<]

这个指令会导致编译器一看到它就报错。

通常，它用在条件语句中，以阻止编译直到满足一些先决条件:

``` {.c}
#ifndef __STDC_IEC_559__
    #error I really need IEEE-754 floating point to compile. Sorry!
#endif
```

[i[`#error` directive]>]
[i[`#warning` directive]<]

有些编译器有一个非标准的补充指令 `#warning`，会输出一个警告但不会停止编译，但这不在 C11 规范中。

[i[`#warning` directive]>]

## `#embed` 指令

[i[`#embed` directive]<]

<!-- Godbolt 演示: https://godbolt.org/z/Kb3ejE7q5 -->

C23 中新加入的功能!

目前我的编译器还没有支持，所以请对这部分持保留态度！

大意是你可以将文件的字节包含为整数常量，就好像你自己打出来一样。

举个例子，如果你有一个名为 `foo.bin` 的二进制文件，里面包含四个值为 11, 22, 33, 和 44 的字节，然后这样做：

``` {.c}
int a[] = {
#embed "foo.bin"
};
```

就好像你实际打出了这个:

``` {.c}
int a[] = {11,22,33,44};
```

这是一种非常强大的方法，可以使用二进制数据初始化一个数组，而不需要先将所有数据转换成代码---预处理器会为你完成这个工作！

一个更典型的使用情况可能是包含一个小图像的文件，你不希望在运行时加载。

这里是另一个示例：

``` {.c}
int a[] = {
#embed <foo.bin>
};
```

如果你使用尖括号，预处理器会在一系列实现定义的位置中查找文件，就像 `#include` 会做的那样。如果你使用双引号而资源找不到，编译器会像你使用尖括号一样尝试最后的绝望性查找文件。

`#embed` 的工作方式类似于 `#include`，它会在编译器看到数据之前将值有效地粘贴进去。这意味着你可以在各种地方使用它：

```
return
#embed "somevalue.dat"
;
```

或者

```
int x =
#embed "xvalue.dat"
;
```

现在---这些值总是字节吗？意思是它们的值会在 0 到 255 之间，包括这两个值？答案通常是："是"，除非是 "否"。

从技术上讲，这些元素的宽度将是 `CHAR_BIT` 位。在你的系统上，这极有可能是 8，所以你会得到在值域 0-255 之间的数值。（它们永远是非负的。）

此外，可能会有一种实现让你以某种方式覆盖这一点，比如通过命令行或参数。

文件的位大小必须是元素大小的倍数。也就是说，如果每个元素是 8 位，那么文件大小（以位为单位）必须是 8 的倍数。在日常使用中，这是一种令人困惑的说法，其实就是说每个文件必须是一个整数字节的倍数... 当然，它就是这样的。说实话，我甚至都不确定为什么要写这一段。如果你真的那么好奇，就去阅读标准吧。

你可以指定各种参数给 `#embed` 指令。这里有一个使用尚未介绍的 `limit()` 参数的例子：

``` {.c}
int a[] = {
#embed "/dev/random" limit(5)
};
```

但如果你已经在其他地方定义了 `limit` 会怎样呢？幸运的是，你可以在关键词周围加上 `__`，它将以同样的方式工作：

``` {.c}
int a[] = {
#embed "/dev/random" __limit__(5)
};
```

现在...这个 `limit` 是什么呢？

### `limit()` 参数

你可以使用这个参数指定要嵌入的元素数量的限制。

这是一个最大值，而不是绝对值。如果被嵌入的文件短于指定的限制，只会导入那么多字节。

上面的 `/dev/random` 示例展示了这个的动机---在 Unix 下，那是一个会返回无限随机数流的 _字符设备文件_。

嵌入无限字节数会对你的 RAM 造成压力，因此 `limit` 参数让你有一种在一定数量后停止的方法。

最后，你可以在你的 `limit` 中使用 `#define` 宏，如果你感兴趣的话。

### `if_empty` 参数

[i[`if_empty()` embed parameter]<]

这个参数定义如果文件存在但不包含数据，嵌入结果应该是什么。假设文件 `foo.dat` 包含一个值为 123 的单个字节。如果我们这样做：

``` {.c}
int x = 
#embed "foo.dat" if_empty(999)
;
```

我们会得到：

``` {.c}
int x = 123;   // 当 foo.dat 包含一个值为 123 的字节时
```

但如果文件 `foo.dat` 的长度为零字节（即没有任何数据并且为空）呢？如果是这种情况，它将扩展为：

``` {.c}
int x = 999;   // 当 foo.dat 是空的时候
```

值得注意的是，如果`limit`设为`0`，那么`if_empty`将总是被替换。换句话说，零限制实际上意味着文件是空的。

这将总是发出`x = 999`，不管`foo.dat`中是什么内容：

``` {.c}
int x = 
#embed "foo.dat" limit(0) if_empty(999)
;
```

[i[`if_empty()` embed parameter]>]

### `prefix()` 和 `suffix()` 参数

[i[`prefix()` embed parameter]<]
[i[`suffix()` embed parameter]<]

这是在嵌入数据之前添加一些数据的方法。

请注意，这些参数仅影响非空数据！如果文件为空，`prefix`和`suffix`都不会产生任何效果。

以下是一个示例，我们在嵌入三个随机数时，前缀为`11,`，后缀为`,99`：

``` {.c}
int x[] = {
#embed "/dev/urandom" limit(3) prefix(11,) suffix(,99)
};
```

示例结果：

``` {.c}
int x[] = {11,135,116,220,99};
```

您不必同时使用`prefix`和`suffix`。您可以同时使用、单独使用其中一个，或者都不使用。

我们可以利用这样的特性，只应用于非空文件，产生出效果整洁的效果，正如下面从规范中抄袭的示例所示。

假设我们有一个名为`foo.dat`的文件，里面包含一些数据。我们想要使用这些数据来初始化一个数组，然后我们想要在数组后面加一个零元素。

没问题，对吧？

``` {.c}
int x[] = {
#embed "foo.dat" suffix(,0)
};
```

如果`foo.dat`里面是11, 22 和 33，我们会得到：

``` {.c}
int x[] = {11,22,33,0};
```

但等等！如果`foo.dat`是空的呢？那么我们会得到：

``` {.c}
int x[] = {};
```

那就不太好了。

但我们可以这样修复：

``` {.c}
int x[] = {
#embed "foo.dat" suffix(,)
    0
};
```

因为如果文件是空的话，`suffix`参数将被省略，这会变成：

``` {.c}
int x[] = {0};
```

这样就没问题了。

### `__has_embed()`标志符

这是一个很好的方法，用于测试特定文件是否可嵌入，以及文件是否为空。

你可以使用它与 `#if` 指令一起。

以下是一段代码块，从随机数发生器字符设备中获取5个随机数。 如果没有该设备，它会尝试从文件 `myrandoms.dat` 中获取。 如果文件也不存在，它会使用一些硬编码的值：

``` {.c}
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

从技术上讲，`__has_embed()`标志符可解析为以下三个值之一：

|`__has_embed()` 结果|描述|
|-|-|
|`__STDC_EMBED_NOT_FOUND__`|如果文件未找到。|
|`__STDC_EMBED_FOUND__`|如果文件被找到且不为空。|
|`__STDC_EMBED_EMPTY`|如果文件被找到且为空。|

我有充分理由认为 `__STDC_EMBED_NOT_FOUND__` 是 `0`，而其他的不为零（因为这在提案中是隐含的，并且逻辑上是有意义的），但我在这个版本的草案规范中找不到它。

[i[`__has_embed()`标志符]>]

待办事项

### 其他参数

编译器实现可以定义所需的其它嵌入参数---在你的编译器文档中查找这些非标准参数。

例如：

``` {.c}
#embed "foo.bin" limit(12) frotz(lamp)
```

这些可能通常会有一个前缀，以帮助进行命名空间：

``` {.c}
#embed "foo.bin" limit(12) fmc::frotz(lamp)
```

在使用它们之前检测它们是否可用可能是明智的，幸运的是我们可以使用`__has_embed`来帮助我们这里。

通常，`__has_embed()`只会告诉我们文件是否存在。但是——这里有趣的地方——如果任何额外的参数也不被支持，它也会返回false！

因此，如果我们给它一个我们“知道”存在的文件以及我们想要测试存在性的参数，它就会有效地告诉我们该参数是否受支持。

然而，哪个文件“总是”存在呢？原来我们可以使用`__FILE__`宏，它会展开为引用它的源文件的名称！那个文件“必须”存在，否则在鸡和蛋之间会出现严重问题。

让我们测试一下`frotz`参数，看看我们能否使用它：

``` {.c}
#if __has_embed(__FILE__ fmc::frotz(lamp))
    puts("fmc::frotz(lamp) is supported!");
#else
    puts("fmc::frotz(lamp) is NOT supported!");
#endif
```

### 嵌入多字节值

如果不是单个字节，而是一些`int`，会怎么样呢？嵌入文件中的多字节值怎么办？

这不是C23标准支持的事情，但未来可能会为其定义实现扩展。

[i[`#embed` directive]>]

## `#pragma`指令 {#pragma}

[i[`#pragma` directive]<]

这是一个时髦的指令，是"pragmatic"的缩写。您可以使用它来... 做任何您的编译器支持您使用它做的事情。

基本上，您只有在某些文档告诉您这样做时才会将其添加到您的代码中。

### 非标准Pragma指令

[i[`#pragma` directive-->非标准Pragma指令]<]

这是一个使用 `#pragma` 的非标准示例，通过其使编译器使用多个线程并行执行 `for` 循环（如果编译器支持 [fl[OpenMP|https://www.openmp.org/]] 扩展）：

``` {.c}
#pragma omp parallel for
for (int i = 0; i < 10; i++) { ... }
```

全球范围内都有各种各样的 `#pragma` 指令文档记录。

所有未识别的 `#pragma` 都将被编译器忽略。

### 标准 Pragmas

还有一些标准的指令，这些指令以 `STDC` 开头，并遵循相同的形式：

``` {.c}
#pragma STDC pragma_name on-off
```

`on-off` 部分可以是 `ON`、`OFF` 或 `DEFAULT`。

`pragma_name` 可以是以下之一：

[i[`FP_CONTRACT` pragma]<]
[i[`CX_LIMITED_RANGE` pragma]<]

|Pragma 名称|描述|
|-|-|
|`FP_CONTRACT`|允许将浮点表达式合并为单个操作，以避免由多次操作可能导致的舍入误差。|
|[i[`FENV_ACCESS` pragma]]`FENV_ACCESS`|如果计划访问浮点状态标志，则设置为 `ON`。如果为 `OFF`，编译器可能会执行导致标志中的值不一致或无效的优化。|
|`CX_LIMITED_RANGE`|设置为 `ON` 以允许编译器在执行复杂算术时跳过溢出检查。默认为 `OFF`。|

例如：

``` {.c}
#pragma STDC FP_CONTRACT OFF
#pragma STDC CX_LIMITED_RANGE ON
```

[i[`FP_CONTRACT` pragma]>]

关于 `CX_LIMITED_RANGE`，规范指出：

> 该指令的目的是允许实现使用以下公式：
>
> $(x+iy)\times(u+iv) = (xu-yv)+i(yu+xv)$
>
> $(x+iy)/(u+iv) = [(xu+yv)+i(yu-xv)]/(u^2+v^2)$
>
> $|x+iy|=\sqrt{x^2+y^2}$
>
> 在程序员可以确定它们是安全的情况下。

### `_Pragma` 运算符

这是另一种声明`pragma`的方式，你可以在宏中使用。

下面两种方式是等效的：

``` {.c}
#pragma "不必要的" 引号
_Pragma("\"不必要的\" 引号")
```

这可以在宏中使用，如果需要的话：

``` {.c}
#define PRAGMA(x) _Pragma(#x)
```

这可以被用在一个宏中：

## `#line` 指令

这允许你覆盖 `__LINE__` 和 `__FILE__` 的值。 如果你需要的话。

我从来没有想过要这么做，但在K&R2中，他们写道：

> 为了其他生成C程序的预处理器的利益 [...]

所以也许有这方面的用途。

覆盖行号为，比如300：

``` {.c}
#line 300
```

`__LINE__` 将从那里继续计数。

覆盖行号和文件名：

``` {.c}
#line 300 "新文件名"
```

## 空指令

独立一行的 `#` 会被预处理器忽略。老实说，我不知道这种情况下的用例是什么。

我看到过这样的例子：

``` {.c}
#ifdef FOO
    #
#else
    printf("Something");
#endif
```

这只是装饰性的；只有含有独立`#`的行可以被删除而不会产生任何不良影响。

或者是为了外观上的一致性，比如这样：

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

但是，从美学角度来看，这实在太丑陋了。

另一篇帖子提到了消除注释---在GCC中，`#`后面的注释不会被编译器看到。我对此并不怀疑，但规范似乎并没有说这是标准行为。

我的理由搜索并没有带来太多成果。所以我打算说这是一些经典的 C  esoterica。