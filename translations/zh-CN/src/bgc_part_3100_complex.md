<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 复数

[flw[复数|Complex number]]的简要介绍直接摘自维基百科：

> **复数** 是一个可以用形式 $a+bi$ 表示的数，其中 $a$ 和 $b$ 是实数（即C中的浮点类型），$i$ 代表虚数单位，满足方程 $i^2=−1$。由于没有实数能满足这个方程，$i$ 被称为虚数。对于复数 $a+bi$，$a$ 被称为**实部**，$b$ 被称为**虚部**。

但我只会说这么多。我们假设如果你正在阅读这一章，你知道什么是复数以及你想要用它们做什么。

我们需要涵盖的只是C语言计算复数的功能。

事实证明，在编译器中对复数的支持是一个可选项。并非所有符合标准的编译器都能够支持。而且，支持的编译器可能在完整性方面有所不同。

您可以使用以下代码测试系统是否支持复数：

[`__STDC_NO_COMPLEX__` 宏]<

``` {.c}
#ifdef __STDC_NO_COMPLEX__
#error 复数不受支持！
#endif
```

[`__STDC_NO_COMPLEX__` 宏]>]

[`__STDC_IEC_559_COMPLEX__` 宏]<

此外，还有一个宏指示符合IEEE 754标准的浮点数学运算和`_Imaginary`类型的存在。

``` {.c}
#if __STDC_IEC_559_COMPLEX__ != 1
#error 需要支持IEC 60559复数！
#endif
```

[`__STDC_IEC_559_COMPLEX__` 宏]>]

有关详细信息，请参阅C11规范的附录G。

## 复数类型

要使用复数，需包含头文件 [ `complex.h` ](`#include <complex.h>`).

通过这样，您至少会得到两种类型：

复数类型

``` {.c}
_Complex
complex
```

这两者意思相同，所以你可以直接使用更美观的 `complex`。

虚数类型

``` {.c}
_Imaginary
imaginary
```

这两者也意思相同，所以你可以直接使用更美观的 `imaginary`。

你还可以获得表示虚数 $i$ 的值：

``` {.c}
I
_Complex_I
_Imaginary_I
```

宏 `I` 被设置为 `_Imaginary_I`（如果可用），否则为 `_Complex_I`。所以可以直接使用 `I` 来表示虚数。

注意：我曾说过，如果编译器将 `__STDC_IEC_559_COMPLEX__` 设置为 `1`，则必须支持 `_Imaginary` 类型以符合规范。这是我对规范的理解。但是，我不知道有哪个编译器实际上支持 `_Imaginary`，即使他们设置了 `__STDC_IEC_559_COMPLEX__`。所以我将在这里编写一些包含该类型的代码，但无法进行测试。抱歉！

好了，现在我们知道有一个 `complex` 类型，我们如何使用它呢？

## 复数赋值

因为复数有实部和虚部，而两者都依赖于浮点数来存储值，我们需要告诉 C 使用什么精度来存储复数的这两部分。

复数浮点数类型
复数双精度浮点数类型
复数长双精度浮点数类型

我们可以通过将 `float`、`double` 或 `long double` 悬挂到 `complex` 上来实现这一点，可以在其前或其后。

让我们定义一个使用 `float` 作为其组件的复数：

``` {.c}
float complex c;   // 规范更喜欢这种方式
complex float c;   // 同样的事情--顺序无关紧要
```

所以这对于声明很方便，但如何初始化它们或对它们赋值呢？

结果是我们可以使用一些非常自然的符号。举个例子！

``` {.c}
double complex x = 5 + 2*I;
double complex y = 10 + 3*I;
```

分别对应 $5+2i$ 和 $10+3i$。

对于 `complex float` 类型
对于 `complex double` 类型
复数-->声明

## 构造、解构和打印

我们接近了……

我们已经看到了写复数的一种方式：

``` {.c}
double complex x = 5 + 2*I;
```

也没有问题使用其他浮点数来构建它：

``` {.c}
double a = 5;
double b = 2;
double complex x = a + b*I;
```

`CMPLX()` 宏

还有一组宏来帮助构建这些复数。上述代码可以使用 `CMPLX()` 宏来编写，就像这样：

``` {.c}
double complex x = CMPLX(5, 2);
```

根据我的研究，这些几乎是等效的：

``` {.c}
double complex x = 5 + 2*I;
double complex x = CMPLX(5, 2);
```

但`CMPLX()`宏会正确处理每次都会出现的虚部负零，而另一种方法可能会将它们转换为正零。我认为这是一个比较难查证的问题，我会接受任何更多信息^^。`I`可以被定义为`_Complex_I`或`_Imaginary_I`，如果后者存在的话。`_Imaginary_I`将处理有符号零，但`_Complex_I`可能不会。这对分支切割和其他复数数学相关的事情有影响。也许。你能察觉到我确实有些不擅长这个领域吗？无论如何，`CMPLX()`宏的行为就好像`I`被定义为`_Imaginary_I`，具有有符号的零，即使在系统中`_Imaginary_I`不存在的情况下。这似乎意味着如果虚部有可能是零，你应该使用这个宏... 但如果我错了，有人应该纠正我！

`CMPLX()`宏适用于 `double` 类型。还有另外两个宏适用于 `float` 和 `long double`：`CMPLXF()` 和 `CMPLXL()`。 （这些 "f" 和 "l" 后缀几乎出现在所有与复数相关的函数中。）

现在让我们尝试相反的操作：如果我们有一个复数，我们如何将其分解为实部和虚部？

在这里我们有两个函数可以从数中提取实部和虚部：`creal()` 和 `cimag()`：

``` {.c}
double complex x = 5 + 2*I;
double complex y = 10 + 3*I;

printf("x = %f + %fi\n", creal(x), cimag(x));
printf("y = %f + %fi\n", creal(y), cimag(y));
```

输出为：

``` {.default}
x = 5.000000 + 2.000000i
y = 10.000000 + 3.000000i
```

请注意，我在`printf()`格式字符串中使用的`i`是一个字面值 `i`，会被打印出来---它不是格式说明符的一部分。 `creal()` 和 `cimag()` 的返回值都是 `double`。

一如既往，这些函数还有`float` 和 `long double`变体：[`crealf()` 函数] `crealf()`，[`cimagf()` 函数] `cimagf()`，[`creall()` 函数] `creall()`，和 [`cimagl()`函数] `cimagl()`。

[`creal()`函数]
[`cimag()`函数]

## 复数运算和比较

[复数-->算术]

复数之间可以进行算术运算，不过这是超出本指南范围的数学知识。

``` {.c .numberLines}
#include <stdio.h>
#include <complex.h>

int main(void)
{
    double complex x = 1 + 2*I;
    double complex y = 3 + 4*I;
    double complex z;

    z = x + y;
    printf("x + y = %f + %fi\n", creal(z), cimag(z));

    z = x - y;
    printf("x - y = %f + %fi\n", creal(z), cimag(z));

    z = x * y;
    printf("x * y = %f + %fi\n", creal(z), cimag(z));

    z = x / y;
    printf("x / y = %f + %fi\n", creal(z), cimag(z));
}
```

得到的结果为：

``` {.default}
x + y = 4.000000 + 6.000000i
x - y = -2.000000 + -2.000000i
x * y = -5.000000 + 10.000000i
x / y = 0.440000 + 0.080000i
```

你也可以比较两个复数是否相等（或不等）：

``` {.c .numberLines}
#include <stdio.h>
#include <complex.h>

int main(void)
{
    double complex x = 1 + 2*I;
    double complex y = 3 + 4*I;

    printf("x == y = %d\n", x == y);  // 0
    printf("x != y = %d\n", x != y);  // 1
}
```

输出为：

``` {.default}
x == y = 0
x != y = 1
```

它们相等，如果两个组件相等。请注意，与所有浮点数一样，由于舍入误差，它们可能在足够接近时是相等的。[[这个简单的陈述并不能真正展现出需要理解浮点数实际运作的大量工作量。https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/]]

## 复数数学

但等等！除了简单的复数算术外，还有更多！

这里是一个关于复数可用的所有数学函数的摘要表格。

我只列出了每个函数的 `double` 版本，但对于所有这些函数，你可以通过在函数名称后附加 `f` 来获得 `float` 版本，也可以通过附加 `l` 来获得 `long double` 版本。

例如，用于计算复数绝对值的 `cabs()` 函数还有 `cabsf()` 和 `cabsl()` 变体。我会为简洁起见省略它们。

### 三角函数

|函数|描述|
|-|-|
|[i[`ccos()` 函数]]`ccos()`|余弦|
|[i[`csin()` 函数]]`csin()`|正弦|
|[i[`ctan()` 函数]]`ctan()`|正切|
|[i[`cacos()` 函数]]`cacos()`|反余弦|
|[i[`casin()` 函数]]`casin()`|反正弦|
|[i[`catan()` 函数]]`catan()`|玩《卡坦岛》|
|[i[`ccosh()` 函数]]`ccosh()`|双曲余弦|
|[i[`csinh()` 函数]]`csinh()`|双曲正弦|
|[i[`ctanh()` 函数]]`ctanh()`|双曲正切|
|[i[`cacosh()` 函数]]`cacosh()`|反双曲余弦|
|[i[`casinh()` 函数]]`casinh()`|反双曲正弦|
|[i[`catanh()` 函数]]`catanh()`|反双曲正切|

### 指数和对数函数

### 幂函数和绝对值函数

|函数|描述|
|-|-|
|[i[`cabs()` 函数]]`cabs()`|绝对值|
|[i[`cpow()` 函数]]`cpow()`|幂|
|[i[`csqrt()` 函数]]`csqrt()`|平方根|

### 操作函数

|函数|描述|
|-|-|
|[i[`creal()` 函数]]`creal()`|返回实部|
|[i[`cimag()` 函数]]`cimag()`|返回虚部|
|[i[`CMPLX()` 宏]]`CMPLX()`|构造复数|
|[i[`carg()` 函数]]`carg()`|幅角/相位角|
|[i[`conj()` 函数]]`conj()`|共轭[^4a34]|
|[i[`cproj()` 函数]]`cproj()`|投影到黎曼球面|

[^4a34]: 这是唯一一个没有额外前置 `c` 的函数，很奇怪。