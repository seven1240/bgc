```c
#ifdef __STDC_NO_COMPLEX__
#error 复数不受支持！
#endif
```

```c
#if __STDC_IEC_559_COMPLEX__ != 1
#error 需要 IEC 60559 复数支持！
#endif
```

将代码中的注释内容翻译成简体中文:

[i[`_Complex` type]] -> `_Complex`类型

[i[`complex` type]<] -> `complex`类型

``` {.c}
_Complex
complex
```

那两者的意思是一样的，所以最好用更漂亮的`complex`。

[i[`complex` type]>] -> `complex`类型

你也可以得到一些供虚数使用的类型，如果你的实现符合IEC 60559标准：

[i[`_Imaginary` type]] -> `_Imaginary`类型

[i[`imaginary` type]<] -> `imaginary`类型

``` {.c}
_Imaginary
imaginary
```

这两者意思也是一样的，所以最好用更漂亮的`imaginary`。

[i[`imaginary` type]>] -> `imaginary`类型

你还可以获得用于表示虚数$i$的值：

[i[`I` macro]<] -> `I`宏

[i[`_Complex_I` macro]<] -> `_Complex_I`宏

[i[`_Imaginary_I` macro]<] -> `_Imaginary_I`宏

``` {.c}
I
_Complex_I
_Imaginary_I
```

宏`I`被设置为`_Imaginary_I`（如果可用），或者`_Complex_I`。所以只需使用`I`来表示虚数。

[i[`I` macro]>] -> `I`宏

[i[`_Complex_I` macro]>] -> `_Complex_I`宏

[i[`__STDC_IEC_559_COMPLEX__` macro]<] -> `__STDC_IEC_559_COMPLEX__`宏

一句话：我已经说过了，如果编译器将`__STDC_IEC_559_COMPLEX__`设置为`1`，必须支持`_Imaginary`类型才能符合标准。这是我理解规范的方式。然而，我不知道有哪个编译器实际上支持`_Imaginary`，尽管它们都设置了`__STDC_IEC_559_COMPLEX__`。所以我将在这里写一些有关这个类型的代码，但没有办法进行测试。抱歉！

[i[`__STDC_IEC_559_COMPLEX__` macro]>] -> `__STDC_IEC_559_COMPLEX__`宏

[i[`_Imaginary_I` macro]>] -> `_Imaginary_I`宏

现在我们知道有`complex`类型了，那么我们如何使用它？

## 赋值复数

[i[Complex numbers-->declaring]<] -> 复数-->声明

由于复数有实部和虚部，但它们都依赖于浮点数来存储值，我们需要告诉C使用什么精度来处理复数的这两个部分。

[i[`complex float` type]<] -> `complex float`类型

[i[`complex double` type]<] -> `complex double`类型

[i[`complex long double` type]<] -> `complex long double`类型

我们只需将`float`、`double`或`long double`与`complex`结合起来，无论是在其之前还是之后。让我们来定义一个使用`float`作为其组成部分的复数：

``` {.c}
float complex c;   // 规范更偏向于这种方式
complex float c;   // 同样的事情--顺序不重要
```

这样在声明时很不错，但是如何初始化或赋值呢？

原来我们可以使用一些非常自然的表示法。举个例子！

``` {.c}
double complex x = 5 + 2*I;
double complex y = 10 + 3*I;
```

对应着$5+2i$和$10+3i$。

对于`complex float`类型、`complex double`类型以及复数-->声明，我们正在逐步接近...

我们已经看到了一个写复数的方法：

``` {.c}
double complex x = 5 + 2*I;
```

还可以毫无问题地使用其他浮点数来构建它：

``` {.c}
double a = 5;
double b = 2;
double complex x = a + b*I;
```

此外，还有一组宏来帮助构建这些内容。上面的代码可以使用`CMPLX()`宏来编写，如下所示：

``` {.c}
double complex x = CMPLX(5, 2);
```

据我在调查中所了解，这两种方法几乎是等效的：

``` {.c}
double complex x = 5 + 2*I;
double complex x = CMPLX(5, 2);
```

但 `CMPLX()` 宏将会在每次正确处理负零作为虚部的情况，而另一种方式可能会将其转换为正零。 我~^[这个更难查证，我会感激任何有关的信息。`I` 可能被定义为 `_Complex_I` 或 `_Imaginary_I`，如果后者存在的话。`_Imaginary_I` 将处理有符号零，但 `_Complex_I` 可能不会。这会影响到分支切割以及其他复数计算中的数学问题。也许。你能感觉到我真的有点超出自己的领域吗？无论如何，`CMPLX()` 宏表现得就好像 `I` 被定义为 `_Imaginary_I`，带有有符号零，即使系统中没有 `_Imaginary_I` 存在。] 这似乎暗示如果虚部有可能是零，你应该使用这个宏... 但如果我错了，有人应该更正我！

`CMPLX()` 宏适用于 `double` 类型。还有两个宏适用于 `float` 和 `long double`：[i[`CMPLXF()` 宏]] `CMPLXF()` 和 [i[`CMPLXL()` 宏]] `CMPLXL()`。 （这些"f"和"l"后缀几乎出现在所有与复数有关的函数中。）

[i[`CMPLX()` 宏]>]

现在让我们尝试反过来：如果我们有一个复数，如何将其分解为实部和虚部？

[i[`creal()` 函数]<]
[i[`cimag()` 函数]<]

这里有两个函数，可以从数字中提取实部和虚部： `creal()` 和 `cimag()`：

``` {.c}
double complex x = 5 + 2*I;
double complex y = 10 + 3*I;

printf("x = %f + %fi\n", creal(x), cimag(x));
printf("y = %f + %fi\n", creal(y), cimag(y));
```

得到输出：

``` {.default}
x = 5.000000 + 2.000000i
y = 10.000000 + 3.000000i
```

请注意，`printf()` 格式字符串中的 `i` 是一个字面值 `i`，会被打印出来---它不是格式指示符的一部分。`creal()` 和 `cimag()` 的返回值都是 `double` 类型。

就像往常一样，这些函数还有 `float` 和 `long double` 的变体：[`crealf()` 函数] `crealf()`，[`cimagf()` 函数] `cimagf()`，[`creall()` 函数] `creall()`，和 [`cimagl()` 函数] `cimagl()`。

[【`creal()` 函数】>]
[【`cimag()` 函数】>]

## 复数算术和比较

[【复数-->算术】<]

虽然这些函数的数学运算超出了本指南的范围，但是在复数上可以进行算术运算。

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

结果如下:

``` {.default}
x + y = 4.000000 + 6.000000i
x - y = -2.000000 + -2.000000i
x * y = -5.000000 + 10.000000i
x / y = 0.440000 + 0.080000i
```

您也可以比较两个复数是否相等（或不相等）：

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

输出为:

``` {.default}
x == y = 0
x != y = 1
```

它们相等，如果两个分量相等。请注意，就像所有浮点数一样，由于舍入误差，它们如果足够接近可能会相等^[这种陈述的简单性并不能充分展现理解浮点数实际功能所需的大量工作。https://randomascii.wordpress.com/2012/02/25/comparing-floating-point-numbers-2012-edition/]。

## 复数算术

但等等！复数算术不仅仅是简单的复数运算！

这里是一个总结表格，列出了可用于复数的所有数学函数。

我只列出每个函数的`double`版本，但对于所有这些函数，你都可以通过在函数名称后添加`f`获得`float`版本，通过添加`l`获得`long double`版本。

例如，用于计算复数绝对值的`cabs()`函数也有`cabsf()`和`cabsl()`变体。为简洁起见，我省略了它们。

### 三角函数

|函数名|描述|
|-|-|
|[i[`ccos()` 函数]]`ccos()`|余弦|
|[i[`csin()` 函数]]`csin()`|正弦|
|[i[`ctan()` 函数]]`ctan()`|正切|
|[i[`cacos()` 函数]]`cacos()`|反余弦|
|[i[`casin()` 函数]]`casin()`|反正弦|
|[i[`catan()` 函数]]`catan()`|玩卡坦岛|
|[i[`ccosh()` 函数]]`ccosh()`|双曲余弦|
|[i[`csinh()` 函数]]`csinh()`|双曲正弦|
|[i[`ctanh()` 函数]]`ctanh()`|双曲正切|
|[i[`cacosh()` 函数]]`cacosh()`|反双曲余弦|
|[i[`casinh()` 函数]]`casinh()`|反双曲正弦|
|[i[`catanh()` 函数]]`catanh()`|反双曲正切|

### 指数和对数函数

### Power and Absolute Value Functions

|功能|描述|
|-|-|
|[`cabs()`函数]|绝对值|
|[`cpow()`函数]|乘幂|
|[`csqrt()`函数]|平方根|

### 操作函数

|功能|描述|
|-|-|
|[`creal()`函数]|返回实部|
|[`cimag()`函数]|返回虚部|
|[`CMPLX()`宏]|构造复数|
|[`carg()`函数]|参数/相位角|
|[`conj()`函数]|共轭[^4a34]|
|[`cproj()`函数]|在黎曼球上的投影|

[^4a34]: 这是唯一一个没有以额外前导的"c"开头的。