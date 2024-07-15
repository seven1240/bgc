```c
#include <stdint.h>

int32_t variable;  // Here we declare a variable of exactly 32 bits

int_least16_t array[10];  // Here we declare an array of at least 16 bits

int_fast8_t value;  // Here we declare a value of at least 8 bits and as fast as possible
```

多快才叫`fast`？肯定可能会快一些。大概。规范没有说快多快，只是说它们将在这个架构上是最快的。大多数C编译器都相当不错，所以你可能只会在需要保证最高速度的地方看到这个被使用（而不是指望编译器生成相当快的代码，尽管它确实是这样的）。

最后，这些无符号数类型在前面带有`u`以加以区分。

例如，这些类型具有以下列出的含义：

[i[`int_leastN_t` types]<]
[i[`uint_leastN_t` types]<]
[i[`int_fastN_t` types]<]
[i[`uint_fastN_t` types]<]
[i[`intN_t` types]<]
[i[`uintN_t` types]<]

``` {.c}
int32_t w;        // w 恰好是32位，有符号
uint16_t x;       // x 恰好是16位，无符号

int_least8_t y;   // y 至少是8位，有符号

uint_fast64_t z;  // z 是最快的表示，至少64位，无符号
```

以下类型是被保证定义的：

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

可能会有其他不同宽度的类型，但这些是可选的。

嘿！固定类型`int16_t`在哪里呢？原来那些是完全可选的...除非满足一定条件^[换句话说，系统具有8、16、32或64位整数，没有填充，并且使用二进制补码表示法，那么对于那个特定位数的`intN_t`变种必须被定义。]. 如果你拥有一台普通的现代计算机系统，这些条件很可能是满足的。如果是这样，你会拥有以下类型：

``` {.c}
int8_t      uint8_t
int16_t     uint16_t
int32_t     uint32_t
int64_t     uint64_t
```

其他带有不同宽度的变体可能被定义，但都是可选的。

[i[`int_leastN_t` 类型]>]
[i[`uint_leastN_t` 类型]>]
[i[`int_fastN_t` 类型]>]
[i[`uint_fastN_t` 类型]>]
[i[`intN_t` 类型]>]
[i[`uintN_t` 类型]>]

## 整型的最大尺寸类型

有一种类型可以容纳系统中可表示的最大整数，无论是有符号的还是无符号的：

[i[`intmax_t` 类型]<]
[i[`uintmax_t` 类型]<]

``` {.c}
intmax_t
uintmax_t
```

[i[`intmax_t` 类型]>]
[i[`uintmax_t` 类型]>]

当你想使用尽可能大的整数时，使用这些类型。

显然，相同符号的任何其他整数类型的值都能适应这种类型。

## 使用固定大小的常量

如果你有一个常量想让它适合特定位数，你可以使用这些宏自动将正确的后缀附加到数字上（例如`22L`或`3490ULL`）。

[i[`INTn_C()` 宏]<]
[i[`UINTn_C()` 宏]<]
[i[`INTMAX_C()` 宏]<]
[i[`UINTMAX_C()` 宏]<]

``` {.c}
INT8_C(x)     UINT8_C(x)
INT16_C(x)    UINT16_C(x)
INT32_C(x)    UINT32_C(x)
INT64_C(x)    UINT64_C(x)
INTMAX_C(x)   UINTMAX_C(x)
```

[i[`INTn_C()` 宏]>]
[i[`UINTMAX_C()` 宏]>]

再次强调，这些仅适用于常量整数值。

例如，我们可以用其中一个来分配常量值，如下所示：

``` {.c}
uint16_t x = UINT16_C(12);
intmax_t y = INTMAX_C(3490);
```

[i[`UINTn_C()` 宏]>]
[i[`INTMAX_C()` 宏]>]

## 固定大小整数的极限值

我们还定义了一些极限值，以便您可以获取这些类型的最大和最小值：

[i[`INTn_MAX` 宏]<]
[i[`INTn_MIN` 宏]<]
[i[`UINTn_MAX` 宏]<]
[i[`INT_LEASTn_MAX` 宏]<]
[i[`INT_LEASTn_MIN` 宏]<]
[i[`UINT_LEASTn_MAX` 宏]<]
[i[`INT_FASTn_MAX` 宏]<]
[i[`INT_FASTn_MIN` 宏]<]
[i[`UINT_FASTn_MAX` 宏]<]
[i[`INTMAX_MAX` 宏]<]
[i[`INTMAX_MIN` 宏]<]
[i[`UINTMAX_MAX` 宏]<]

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

[i[`INTn_MAX` 宏]>]
[i[`UINTn_MAX` 宏]>]
[i[`INT_LEASTn_MAX` 宏]>]
[i[`UINT_LEASTn_MAX` 宏]>]
[i[`INT_FASTn_MAX` 宏]>]
[i[`UINT_FASTn_MAX` 宏]>]
[i[`INTMAX_MAX` 宏]>]
[i[`UINTMAX_MAX` 宏]>]

请注意，所有无符号类型的`MIN`值都是`0`，因此，就此而言，没有相应的宏。

```c
[i[`INTn_MIN` 宏]>]
[i[`INT_LEASTn_MIN` 宏]>]
[i[`INT_FASTn_MIN` 宏]>]
[i[`INTMAX_MIN` 宏]>]

## 格式说明符

为了打印这些类型，你需要向[i[`printf()` 函数]] `printf()` 提供正确的格式说明符。 (同样的问题也适用于使用 [i[`scanf()` 函数]] `scanf()` 获取输入。)

但是，你如何知道底层这些类型的大小呢？幸运的是，C 再次提供了一些宏来帮助处理这个问题。

所有这些都可以在 `<inttypes.h>` 中找到。

现在，我们有很多宏。就像宏的复杂性爆炸一样。所以我将停止列出每一个，并在你应该放置 `8`、`16`、`32` 或 `64` 的地方放置小写字母 `n`。

让我们看看用于打印有符号整数的宏：

[i[`PRIdn` 宏]<]
[i[`PRIin` 宏]<]
[i[`PRIdLEASTn` 宏]<]
[i[`PRIiLEASTn` 宏]<]
[i[`PRIdFASTn` 宏]<]
[i[`PRIiFASTn` 宏]<]
[i[`PRIdMAX` 宏]<]
[i[`PRIiMAX` 宏]<]

``` {.c}
PRIdn    PRIdLEASTn    PRIdFASTn    PRIdMAX
PRIin    PRIiLEASTn    PRIiFASTn    PRIiMAX
```

找到这里的模式。你可以看到对于固定、最少、快速和最大类型都有变体。

而且你还有一个小写的 `d` 和一个小写的 `i`。它们对应于 `printf()` 的格式说明符 `%d` 和 `%i`。

因此，如果我有一个类型为：

``` {.c}
int_least16_t x = 3490;
```

我可以使用 `PRIdLEAST16` 来打印具有等价格式说明符 `%d` 的内容。

但是如何？我们如何使用那个宏呢？

首先，该宏指定一个包含 `printf()` 需要用来打印该类型的字母的字符串。比如，它可以是 `"d"` 或 `"ld"`。

因此，我们需要做的就是将这个嵌入到我们的格式字符串中传递给 `printf()` 调用。
```

为达到这个目的，我们可以利用 C 语言中一个你可能忘记的事实：相邻的字符串字面值会自动连接成一个单独的字符串。例如：

``` {.c}
printf("Hello, " "world!\n");   // 输出 "Hello, world!"
```

而且，由于这些宏是字符串字面值，我们可以这样使用它们：

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

[i[`PRIdn` 宏]>]
[i[`PRIin` 宏]>]
[i[`PRIdLEASTn` 宏]>]
[i[`PRIiLEASTn` 宏]>]
[i[`PRIdFASTn` 宏]>]
[i[`PRIiFASTn` 宏]>]
[i[`PRIdMAX` 宏]>]
[i[`PRIiMAX` 宏]>]

我们还有一堆用于打印无符号类型的宏：

[i[`PRIon` 宏]<]
[i[`PRIun` 宏]<]
[i[`PRIxn` 宏]<]
[i[`PRIXn` 宏]<]
[i[`PRIoLEASTn` 宏]<]
[i[`PRIuLEASTn` 宏]<]
[i[`PRIxLEASTn` 宏]<]
[i[`PRIXLEASTn` 宏]<]
[i[`PRIoFASTn` 宏]<]
[i[`PRIuFASTn` 宏]<]
[i[`PRIxFASTn` 宏]<]
[i[`PRIXFASTn` 宏]<]
[i[`PRIoMAX` 宏]<]
[i[`PRIuMAX` 宏]<]
[i[`PRIxMAX` 宏]<]
[i[`PRIXMAX` 宏]<]

``` {.c}
PRIon    PRIoLEASTn    PRIoFASTn    PRIoMAX
PRIun    PRIuLEASTn    PRIuFASTn    PRIuMAX
PRIxn    PRIxLEASTn    PRIxFASTn    PRIxMAX
PRIXn    PRIXLEASTn    PRIXFASTn    PRIXMAX
```

在这种情况下，`o`、`u`、`x` 和 `X` 对应于 `printf()` 中记录的格式说明符。

而且，与之前一样，小写的 `n` 应替换为 `8`、`16`、`32` 或 `64`。

``` c
// 在你以为你已经够用这些宏的时候，事实证明我们还为 `scanf()` 函数准备了完整的补充设置。

SCNdn    SCNdLEASTn    SCNdFASTn    SCNdMAX
SCNin    SCNiLEASTn    SCNiFASTn    SCNiMAX
SCNon    SCNoLEASTn    SCNoFASTn    SCNoMAX
SCNun    SCNuLEASTn    SCNuFASTn    SCNuMAX
SCNxn    SCNxLEASTn    SCNxFASTn    SCNxMAX
```

// ```[i[`SCNdn` macros]<]```
// ```[i[`SCNin` macros]<]```
// ```[i[`SCNon` macros]<]```
// ```[i[`SCNun` macros]<]```
// ```[i[`SCNxn` macros]<]```
// ```[i[`SCNdLEASTn` macros]<]```
// ```[i[`SCNiLEASTn` macros]<]```
// ```[i[`SCNoLEASTn` macros]<]```
// ```[i[`SCNuLEASTn` macros]<]```
// ```[i[`SCNxLEASTn` macros]<]```
// ```[i[`SCNdFASTn` macros]<]```
// ```[i[`SCNiFASTn` macros]<]```
// ```[i[`SCNoFASTn` macros]<]```
// ```[i[`SCNuFASTn` macros]<]```
// ```[i[`SCNxFASTn` macros]<]```
// ```[i[`SCNdMAX` macros]<]```
// ```[i[`SCNiMAX` macros]<]```
// ```[i[`SCNoMAX` macros]<]```
// ```[i[`SCNuMAX` macros]<]```
// ```[i[`SCNxMAX` macros]<]```

记住：如果你想用`printf()`或者`scanf()`打印一个固定大小的整数类型，记得从`<inttypes.h>`中找到正确对应的格式说明符。