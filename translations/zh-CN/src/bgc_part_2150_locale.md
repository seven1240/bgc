``` {.c}
setlocale(LC_ALL, "");  // 使用当前环境的区域设置来设置所有内容
```

您需要调用这个函数，以便程序使用当前的区域设置进行初始化。

更详细地说，还有一件事可以做，而且保持可移植性：

``` {.c}
setlocale(LC_ALL, "C");  // 使用默认的 C 区域设置
```

但这在每次程序启动时默认调用，所以没有必要自己执行。

``` {.c}
setlocale(LC_ALL, "en_US.UTF-8");  // 非可移植方式！

那样也能运行。但是这只适用于具有完全相同区域设置名称的系统，而且无法保证这一点。

通过在第二个参数传入空字符串（""），你在告诉C，“嘿，弄清楚这个系统上当前的区域设置，这样我就不用告诉你了。”

[i[`setlocale()`函数]→]

## 获取货币区域设置

[i[Locale-->money]→]

因为移动绿色纸币似乎是通往幸福的关键^["这颗行星曾经---或者应该说曾经---有一个问题，即大多数居住在上面的人几乎一直都不开心。对于这个问题，提出了许多解决方案，但其中大多数都与小额绿色纸币的流通有关，这很奇怪，因为总的来说并不是小额绿色纸币让人不开心。" ---《银河系漫游指南》，道格拉斯·亚当斯]，让我们谈谈货币区域设置。在编写可移植代码时，你需要知道怎么输入金额，对吧？无论是"$"、"€"、"¥"还是"£"。

[i[`localeconv()`函数]→]

如何编写这样的代码而不会发疯？幸运的是，一旦调用了`setlocale(LC_ALL, "")`，你可以通过调用`localeconv()`来查找这些信息：

``` {.c}
struct lconv *x = localeconv();
```

这个函数返回指向静态分配的`struct lconv`的指针，里面包含你要找的全部有价值的信息。

下面是`struct lconv`的字段及其含义。
```

首先，一些约定。`_p_` 表示"正面"，`_n_` 表示"负面"，而 `int_` 表示"国际化"。尽管很多这些变量是 `char` 类型或 `char*` 类型，但大多数（或者它们所指向的字符串）实际上被视为整数^[记住 `char` 只是一个字节大小的整数。]。

在我们继续之前，请了解 `CHAR_MAX` （来自 `<limits.h>`）是 `char` 类型能够保存的最大值。同样，很多后续的 `char` 值使用这一点来表示在给定区域设置中该值不可用。

|Field|Description|
|-----|-----------|
|`char *mon_decimal_point`|货币的小数点分隔符，例如 `"."`。|
|`char *mon_thousands_sep`|货币的千位分隔符，例如 `","`。|
|`char *mon_grouping`|货币的分组描述（见下文）。|
|`char *positive_sign`|正数货币的符号，例如 `"+"` 或 `""`。|
|`char *negative_sign`|负数货币的符号，例如 `"-"`。|
|`char *currency_symbol`|货币符号，例如 `"$"`。|
|`char frac_digits`|打印货币金额时，小数点后要打印多少位数，例如 `2`。|
|`char p_cs_precedes`|如果 `currency_symbol` 在非负货币金额之前，则为 `1`，在之后则为 `0`。|
|`char n_cs_precedes`|如果 `currency_symbol` 在负货币金额之前，则为 `1`，在之后则为 `0`。|
|`char p_sep_by_space`|确定非负金额中 `currency symbol` 与值之间的分隔情况（见下文）。|
|`char n_sep_by_space`|确定负金额中 `currency symbol` 与值之间的分隔情况（见下文）。|
|`char p_sign_posn`|确定非负值的 `positive_sign` 位置。|
|`char p_sign_posn`|确定负值的 `positive_sign` 位置。|
|`char *int_curr_symbol`|国际货币符号，例如 `"USD "`。|
|`char int_frac_digits`|`frac_digits` 的国际值。|
|`char int_p_cs_precedes`|`p_cs_precedes` 的国际值。|
|`char int_n_cs_precedes`|`n_cs_precedes` 的国际值。|
|`char int_p_sep_by_space`|`p_sep_by_space` 的国际值。|
|`char int_n_sep_by_space`|`n_sep_by_space` 的国际值。|
|`char int_p_sign_posn`|`p_sign_posn` 的国际值。|
|`char int_n_sign_posn`|`n_sign_posn` 的国际值。|

### 货币数字分组 {#monetary-digit-grouping}

这个概念有点复杂。`mon_grouping`是一个`char*`类型，所以你可能会觉得它是一个字符串。但在这种情况下，不是的，实际上它是一个`char`数组。它应该以`0`或`CHAR_MAX`结尾。

这些值描述了如何对货币小数点左侧（整数部分）的数字进行分组。

例如，我们可能有：

``` {.default}
  2   1   0
 --- --- ---
$100,000,000.00
```

这是三位一组。第0组（小数点左侧）有3位数。第1组（再往左的一组）有3位数，最后一个也有3位数。

因此，我们可以用一堆整数值表示组大小，从右边（小数点）到左边描述这些组：

``` {.default}
3 3 3
```

这对于高达$100,000,000的值有效。

但如果我们有更大的值怎么办？我们可以继续添加`3`...

``` {.default}
3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3 3
```

但那太疯狂了。幸运的是，我们可以指定`0`来表示重复前一个组大小：

``` {.default}
3 0
```

这意味着每3位数就重复一次。这将处理$100，$1,000，$10,000，$10,000,000，$100,000,000,000等情况。

你可以通过这些来指示一些奇怪的分组。

例如：

``` {.default}
4 3 2 1 0
```

将表示：

``` {.default}
$1,0,0,0,0,00,000,0000.00
```

还可能出现的一种值是`CHAR_MAX`。这表示不再进行分组，可以出现在数组中的任何位置，包括第一个值。

``` {.default}
3 2 CHAR_MAX
```

将表示：

``` {.default}
100000000,00,000.00
```

例如。

而在第一个数组位置上简单地放置`CHAR_MAX`将告诉你根本不需要分组。

### 分隔符和符号位置

处理与货币符号周围的空格的所有`sep_by_space`变体。有效值包括：

|数值|描述|
|:--:|------------|
|`0`|货币符号和数值之间没有空格。|
|`1`|用空格将货币符号（及符号，如果有的话）与数值分开。|
|`2`|将符号符号与货币符号（如果相邻）之间用空格分开，否则将符号符号与数值分开。|

`sign_posn`变体由以下值确定：

|数值|描述|
|:--:|------------|
|`0`|在值和货币符号周围放括号。|
|`1`|将符号字符串放在货币符号和值之前。|
|`2`|将符号字符串放在货币符号和值之后。|
|`3`|将符号字符串直接放在货币符号前面。|
|`4`|将符号字符串直接放在货币符号后面。|

### 示例数值

当我在我的系统上获取数值时，我看到的情况如下（分组字符串以单个字节值显示）：

``` {.c}
mon_decimal_point  = "."
mon_thousands_sep  = ","
mon_grouping       = 3 3 0
positive_sign      = ""
negative_sign      = "-"
currency_symbol    = "$"
frac_digits        = 2
p_cs_precedes      = 1
n_cs_precedes      = 1
p_sep_by_space     = 0
n_sep_by_space     = 0
p_sign_posn        = 1
n_sign_posn        = 1
int_curr_symbol    = "USD "
int_frac_digits    = 2
int_p_cs_precedes  = 1
int_n_cs_precedes  = 1
int_p_sep_by_space = 1
int_n_sep_by_space = 1
int_p_sign_posn    = 1
int_n_sign_posn    = 1
```

## 本地化具体细节

```c
注意我们之前是如何将宏`LC_ALL`传递给`setlocale()`的...这表明可能存在一些变体，允许你更精确地设置locale的_部分_。

让我们来看看你可以为这些值看到的：

|宏|描述|
|----|--------------|
|[i[`setlocale()`函数-->`LC_ALL`宏]]`LC_ALL`|将所有以下内容设置为给定的locale。|
|[i[`setlocale()`函数-->`LC_COLLATE`宏]]`LC_COLLATE`|控制`strcoll()`和`strxfrm()`函数的行为。|
|[i[`setlocale()`函数-->`LC_CTYPE`宏]]`LC_CTYPE`|控制字符处理函数的行为^[除了`isdigit()`和`isxdigit()`。]。|
|[i[`setlocale()`函数-->`LC_MONETARY`宏]]`LC_MONETARY`|控制`localeconv()`返回的值。|
|[i[`setlocale()`函数-->`LC_NUMERIC`宏]]`LC_NUMERIC`|控制`printf()`系列函数的小数点。|
|[i[`setlocale()`函数-->`LC_TIME`宏]]`LC_TIME`|控制`strftime()`和`wcsftime()`时间和日期打印函数的时间格式化。|

[p`setlocale()`函数-->`LC_ALL`宏]看到设置`LC_ALL`是相当常见的，但是，嘿，至少你有选择。

另外我要指出[p`setlocale()`函数-->`LC_CTYPE`宏]`LC_CTYPE`是其中一个重要的，因为它涉及到宽字符，这是一个我们稍后会谈到的重要课题。

[p[Locale]>]
```