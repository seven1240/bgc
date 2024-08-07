<!-- C语言指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 定宽整型

C语言有各种大小的整数类型，比如`int`和`long`等。你可以查看[有关限制的部分](#limits-macros)来了解`INT_MAX`表示的最大整数是多少。

这些类型大小有多大呢？也就是占用了多少字节？我们可以使用`sizeof`来得到这个答案。

但是如果我想反过来呢？如果我需要一个刚好是32位（4字节）、至少16位等等的类型该怎么办呢？

我们如何声明一个特定大小的类型呢？

头文件[`stdint.h`文件]`<stdint.h>`给我们提供了一种方式。

## 比特大小的类型

对于有符号和无符号整数，我们可以指定一个包含一定数量比特的类型，当然有一些注意事项。

这些类型主要分为三类（在这些示例中，`N`将被一定数量的比特替代）：

* 精确为一定大小的整数类型（`intN_t`）
* 至少为一定大小的整数类型（`int_leastN_t`）
* 至少为一定大小且尽可能快速的整数类型（`int_fastN_t`）

*注：一些体系结构具有CPU和RAM可以以比其他体系结构更快的速度处理的不同大小数据。在这些情况下，如果你需要最快的8位整数，它可能会给你一个16位或32位整数类型，因为那样更快。所以使用这种类型，你可能不知道类型的大小，但它将至少和你指定的一样大。*

`快速`有多快？肯定可能会快一点点。可能。规范并未说明有多快，只是说它们将在这个架构上是最快的。大多数C编译器都相当不错，因此你可能只会在需要保证最大可能速度的地方看到这样的用法（而不只是希望编译器生成非常快速的代码，事实上它已经是这样了）。

最后，这些无符号数类型都有一个前导的 `u` 以区别它们。

例如，这些类型具有相应的列出的含义：

``` {.c}
int32_t w;        // x 是精确地 32 位，有符号的
uint16_t x;       // y 是精确地 16 位，无符号的

int_least8_t y;   // y 至少是 8 位，有符号的

uint_fast64_t z;  // z 是至少 64 位最快表示，无符号的
```

以下类型是被保证定义了的：

``` {.c}
int_least8_t      uint_least8_t
int_least16_t     uint_least16_t
int_least32_t     uint_least32_t
int_least64_t     uint_least64_t

int_fast8_t       uint_fast8_t
int_fast16_t      uint_fast16_t
int_fast32_t      uint_fast32_t
int_fast64_t      uint_fast64_t
```

可能还有其他不同宽度的类型，但那些是可选的。

嘿！固定类型`int16_t`在哪里？原来这些完全是可选的……除非满足某些条件^[即，系统具有8、16、32或64位整数，没有填充，使用二进制补码表示，此时针对该位数的`intN_t`变体必须被定义。]。如果你有一台普通的现代计算机系统，那么这些条件可能会被满足。如果满足了，你会有以下这些类型：

``` {.c}
int8_t      uint8_t
int16_t     uint16_t
int32_t     uint32_t
int64_t     uint64_t
```

其他不同宽度的变体可能会被定义，但是它们是可选的。

## 最大整数大小类型

有一种类型可以存储系统中可表示的最大整数，包括有符号和无符号的：

[i[`intmax_t`类型]<]
[i[`uintmax_t`类型]<]

``` {.c}
intmax_t
uintmax_t
```

当你希望尽可能地使用大数时可以使用这些类型。

显然，任何其他相同符号的整数类型的值都能被放入这种类型中。

## 使用固定尺寸常量

如果你有一个常量，希望它适应特定数量的位数，你可以使用这些宏来自动附加正确的后缀到数字上（例如，`22L`或`3490ULL`）。

[i[`INTn_C()`宏]<]
[i[`UINTn_C()`宏]<]
[i[`INTMAX_C()`宏]<]
[i[`UINTMAX_C()`宏]<]

``` {.c}
INT8_C(x)     UINT8_C(x)
INT16_C(x)    UINT16_C(x)
INT32_C(x)    UINT32_C(x)
INT64_C(x)    UINT64_C(x)
INTMAX_C(x)   UINTMAX_C(x)
```

再次强调，这些仅适用于常量整数值。

例如，我们可以使用其中一个来分配常量值，就像这样：

``` {.c}
uint16_t x = UINT16_C(12);
intmax_t y = INTMAX_C(3490);
```

## 固定大小整数的极限值

我们还定义了一些极限值，这样您可以获得这些类型的最大值和最小值：

``` {.c}
INT8_MAX           INT8_MIN           UINT8_MAX
INT16_MAX          INT16_MIN          UINT16_MAX
INT32_MAX          INT32_MIN          UINT32_MAX
INT64_MAX          INT64_MIN          UINT64_MAX

INT_LEAST8_MAX     INT_LEAST8_MIN     UINT_LEAST8_MAX
INT_LEAST16_MAX    INT_LEAST16_MIN    UINT_LEAST16_MAX
INT_LEAST32_MAX    INT_LEAST32_MIN    UINT_LEAST32_MAX
INT_LEAST64_MAX    INT_LEAST64_MIN    UINT_LEAST64_MAX

INT_FAST8_MAX      INT_FAST8_MIN      UINT_FAST8_MAX
INT_FAST16_MAX     INT_FAST16_MIN     UINT_FAST16_MAX
INT_FAST32_MAX     INT_FAST32_MIN     UINT_FAST32_MAX
INT_FAST64_MAX     INT_FAST64_MIN     UINT_FAST64_MAX

INTMAX_MAX         INTMAX_MIN         UINTMAX_MAX
```

请注意，所有无符号类型的 `MIN` 值均为 `0`，因此，因为没有相应的宏，故不再列出。

## 格式说明符

要打印这些类型，你需要向 `printf()` 函数发送正确的格式说明符。 （类似地，使用 `scanf()` 函数获取输入也存在相同的问题。）

但是，你如何才能知道这些类型在内部的大小是多少呢？幸运的是，再次，C语言提供了一些宏来帮助解决这个问题。

所有这些都可以在 `<inttypes.h>` 中找到。

现在，我们有一堆宏。就像宏的复杂性爆炸一样。 所以我将停止列出每一个，只在应该放置 `8`、`16`、`32` 或 `64` 的地方放置小写字母 `n`。

让我们看看用于打印有符号整数的宏：

寻找其中的模式。你可以看到有固定、最小、快速和最大类型的变体。

而且你还有一个小写的 `d` 和一个小写的 `i`。 这些对应于 `printf()` 中的格式说明符 `%d` 和 `%i`。

因此，如果我有类型为：

``` {.c}
int_least16_t x = 3490;
```

我可以使用 `PRIdLEAST16` 来打印相应的格式说明符 `%d`。

但是怎么做呢？我们如何使用这个宏呢？

首先，该宏指定包含 `printf()` 需要用于打印该类型的字母或字母的字符串。比如，它可以是 `"d"` 或 `"ld"`。

因此，我们只需要将其嵌入到传递给 `printf()` 调用的格式字符串中即可。

为了做到这一点，我们可以利用关于C语言的一个你可能已经忘记的事实：相邻的字符串文字会自动连接成一个单一的字符串。例如：

``` {.c}
printf("Hello, " "world!\n");   // 打印出 "Hello, world!"
```

而且由于这些宏是字符串文字，我们可以这样使用它们：

``` {.c .numberLines}
#include <stdio.h>
#include <stdint.h>
#include <inttypes.h>

int main(void)
{
    int_least16_t x = 3490;

    printf("The value is %" PRIdLEAST16 "!\n", x);
}
```

我们还有一堆用于打印无符号类型的宏：

``` {.c}
PRIon    PRIoLEASTn    PRIoFASTn    PRIoMAX
PRIun    PRIuLEASTn    PRIuFASTn    PRIuMAX
PRIxn    PRIxLEASTn    PRIxFASTn    PRIxMAX
PRIXn    PRIXLEASTn    PRIXFASTn    PRIXMAX
```

在这种情况下，`o`、`u`、`x`和`X` 对应于`printf()`中的文档化格式说明符。

和之前一样，小写的`n` 应该替换成`8`、`16`、`32`或`64`。

[`PRIon` 宏]
[`PRIun` 宏]
[`PRIxn` 宏]
[`PRIXn` 宏]
[`PRIoLEASTn` 宏]
[`PRIuLEASTn` 宏]
[`PRIxLEASTn` 宏]
[`PRIXLEASTn` 宏]
[`PRIoFASTn` 宏]
[`PRIuFASTn` 宏]
[`PRIxFASTn` 宏]
[`PRIXFASTn` 宏]
[`PRIoMAX` 宏]
[`PRIuMAX` 宏]
[`PRIxMAX` 宏]
[`PRIXMAX` 宏]

当你认为你对宏已经足够了的时候，原来我们还有一个完整的互补组的 [  `scanf()` 函数] 的相应设置 `scanf()`!

[`SCNdn` 宏]
[`SCNin` 宏]
[`SCNon` 宏]
[`SCNun` 宏]
[`SCNxn` 宏]
[`SCNdLEASTn` 宏]
[`SCNiLEASTn` 宏]
[`SCNoLEASTn` 宏]
[`SCNuLEASTn` 宏]
[`SCNxLEASTn` 宏]
[`SCNdFASTn` 宏]
[`SCNiFASTn` 宏]
[`SCNoFASTn` 宏]
[`SCNuFASTn` 宏]
[`SCNxFASTn` 宏]
[`SCNdMAX` 宏]
[`SCNiMAX` 宏]
[`SCNoMAX` 宏]
[`SCNuMAX` 宏]
[`SCNxMAX` 宏]

``` {.c}
SCNdn    SCNdLEASTn    SCNdFASTn    SCNdMAX
SCNin    SCNiLEASTn    SCNiFASTn    SCNiMAX
SCNon    SCNoLEASTn    SCNoFASTn    SCNoMAX
SCNun    SCNuLEASTn    SCNuFASTn    SCNuMAX
SCNxn    SCNxLEASTn    SCNxFASTn    SCNxMAX
```

[`SCNdn` 宏]
[`SCNin` 宏]
[`SCNon` 宏]
[`SCNun` 宏]
[`SCNxn` 宏]
[`SCNdLEASTn` 宏]
[`SCNiLEASTn` 宏]
[`SCNoLEASTn` 宏]
[`SCNuLEASTn` 宏]
[`SCNxLEASTn` 宏]
[`SCNdFASTn` 宏]
[`SCNiFASTn` 宏]
[`SCNoFASTn` 宏]
[`SCNuFASTn` 宏]
[`SCNxFASTn` 宏]
[`SCNdMAX` 宏]
[`SCNiMAX` 宏]
[`SCNoMAX` 宏]
[`SCNuMAX` 宏]
[`SCNxMAX` 宏]

记住：当你想要用 `printf()` 或 `scanf()` 打印固定大小的整数类型时，要从 `<inttypes.h>` 中获取正确对应的格式说明符。