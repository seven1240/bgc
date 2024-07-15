```c
#include <stdio.h>

int main(void)
{
    char s[10];
    float f = 3.14159;

    // Convert "f" to string, storing in "s", writing at most 10 characters
    // including the NUL terminator

    snprintf(s, 10, "%f", f);

    printf("String value: %s\n", s);  // String value: 3.141590
}
``` 

这样你就可以像处理整数一样使用`%d`或`%u`。

```markdown
用于将字符串转换为数字的基本转换，可以尝试使用`<stdlib.h>`中的`atoi`函数。这些函数的错误处理特性很差（包括如果传入错误的字符串会导致未定义行为），因此需要谨慎使用。

|函数|描述|
|:-|:-|
|`atoi`|字符串转`int`|
|`atof`|字符串转`float`|
|`atol`|字符串转`long int`|
|`atoll`|字符串转`long long int`|

虽然规范中没有提到，但函数开头的`a`代表[flw[ASCII|ASCII]]，所以`atoi()`实际上是"ASCII-to-integer"，但如今这么说有点以ASCII为中心。

这里有一个将字符串转换为`float`的示例：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *pi = "3.14159";
    float f;

    f = atof(pi);

    printf("%f\n", f);
}
```

但是，就像我说的，我们会因为奇怪的事情得到未定义的行为，比如这样：

``` {.c}
int x = atoi("what");  // “what”不是我听说过的任何数字
```

（当我运行这段代码时，得到的是`0`，但你真的不应该依赖这一点。你可能会得到完全不同的结果。）

要获得更好的错误处理特性，让我们看看所有那些`strtol`函数，也在`<stdlib.h>`中。不仅如此，它们还可以转换更多的类型和更多的进制！

|函数|描述|
|:-|:-|
|`strtol`|字符串转`long int`|
|`strtoll`|字符串转`long long int`|
|`strtoul`|字符串转`unsigned long int`|
|`strtoull`|字符串转`unsigned long long int`|
|`strtof`|字符串转`float`|
|`strtod`|字符串转`double`|
|`strtold`|字符串转`long double`|

这些函数都遵循类似的使用模式，并且是很多人第一次接触指向指针的经验！但不要担心---它比看起来要简单。
```

让我们做个示例，将一个字符串转换为`unsigned long`，丢弃错误信息（即输入字符串中有错误字符的信息）：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "3490";

    // 将字符串s（一个十进制数）转换为unsigned long int。
    // NULL表示我们不关心任何错误信息。

    unsigned long int x = strtoul(s, NULL, 10);

    printf("%lu\n", x);  // 3490
}
```

请注意一些细节。尽管我们没有捕获任何关于字符串中出错字符的信息，`strtoul()`不会导致未定义行为；它只会返回`0`。

另外，我们指定这是一个十进制（基数为10）的数字。

这意味着我们能转换不同进制的数字吗？当然可以！让我们来尝试二进制！

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "101010";  // 这个数字代表什么？

    // 将字符串s（一个二进制数）转换为unsigned long int。

    unsigned long int x = strtoul(s, NULL, 2);

    printf("%lu\n", x);  // 42
}
```

好了，这些都很有趣，但那个代码中的`NULL`是什么意思呢？是用来做什么的呢？

这有助于我们确定在处理字符串时是否发生了错误。它是一个指向`char`指针的指针，听起来有点吓人，但一旦理解，就不难了。

让我们做个示例，输入一个故意错误的数字，看看`strtol()`是如何告诉我们第一个无效数字的位置的。

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "34x90";  // "x"不是十进制中的有效数字！
    char *badchar;

    // 将字符串s（一个十进制数）转换为unsigned long int。
```

```c
unsigned long int x = strtoul(s, &badchar, 10);

// It tries to convert as much as possible, so gets this far:

printf("%lu\n", x);  // 34

// But we can see the offending bad character because badchar
// points to it!

printf("Invalid character: %c\n", *badchar);  // "x"
}
```

因此，`strtoul()` 修改了`badchar`指向的内容，以便向我们显示出错的位置^[我们必须将`badchar`的指针传递给`strtoul()`，否则它无法以我们可见的方式修改它，类似于为什么如果要函数能够更改`int`的值，必须向函数传递`int`的指针。]。

但如果一切顺利呢？在这种情况下，`badchar`将指向字符串末尾的`NUL`终止符。因此，我们可以进行检查：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *s = "3490";  // "x"不是十进制中的有效数字！
    char *badchar;

    // 将字符串s（一个十进制数）转换为无符号长整型数。

    unsigned long int x = strtoul(s, &badchar, 10);

    // 检查转换是否成功

    if (*badchar == '\0') {
        printf("成功！%lu\n", x);
    } else  {
        printf("部分转换：%lu\n", x);
        printf("无效字符：%c\n", *badchar);
    }
}
```

因此，这就是答案。在受控情况下，类似`atoi()`的函数功效很好，但是类似`strtol()`的函数在错误处理和输入基数方面提供更多的控制。

[i[类型转换-->字符串]>]

## `char` 转换

[i[类型转换-->`char`]<]

如果您有一个带有数字的单个字符，例如`'5'`...那与值`5`相同吗？

让我们试一试。

``` {.c}
printf("%d %d\n", 5, '5');
```

在我的UTF-8系统上，这个代码输出：

``` {.default}
5 53
```

所以... 不是。那53又代表什么呢？那是UTF-8（和ASCII）编码中字符符号`'5'`的代码点^[每个字符在任何字符编码方案中都有与之关联的值。]

那么我们如何将字符`'5'`（显然值为53）转换为值`5`呢？

用一个巧妙的技巧就可以实现！

C标准保证了这些字符的代码点是按顺序排列的，并且是这样的：

``` {.default}
0  1  2  3  4  5  6  7  8  9
```

思考一下--我们如何利用这点？接下来有剧透...

让我们看看UTF-8中的字符及其代码点：

``` {.default}
0  1  2  3  4  5  6  7  8  9
48 49 50 51 52 53 54 55 56 57
```

你能看到`'5'`对应`53`，正如我们得到的那样。而`'0'`对应`48`。

因此，我们可以从任何数字字符中减去`'0'`来得到其数值：

``` {.c}
char c = '6';

int x = c;  // x的值为54，即'6'的代码点

int y = c - '0'; // y的值为6，正如我们想要的
```

我们也可以反过来转换，只需要加上相应的值即可。

``` {.c}
int x = 6;

char c = x + '0';  // c的值为54

printf("%d\n", c);  // 输出54
printf("%c\n", c);  // 输出6（使用%c）
```

你可能会认为这是一个奇怪的转换方式，按照如今的标准来看，的确如此。但是在古老时代的计算机实际上是由木头制成时，这就是进行此类转换的方法。而且这种方法是行之有效的，所以C从未修改过它。

[i[类型转换-->`char`]>]

## 数值转换

[i[类型转换-->数值]<]

### 布尔值

[i[类型转换-->布尔值]<]

如果将零转换为`bool`，结果为`0`。否则为`1`。

### 整数到整数的转换

[i[类型转换-->整数]<]

```c
如果整数类型转换为无符号类型而不适合，无符号结果会像里程表一样循环直到适应为止。在您的实现中，实际上可能发生的情况是高阶位被丢弃，因此将16位数字 `0x1234` 转换为8位数字最终变为 `0x0034`，或者只是 `0x34`。

如果整数类型转换为有符号数字而不适合，则结果是实现定义的！会发生一些文档化的事情，但您需要查阅一下。同样，在您的系统上发生的实际情况是，原始数字的位模式将被截断，然后用来表示有符号数字，使用补码。例如，我的系统将 `unsigned char` 的 `192` 转换为 `signed char` 的 `-64`。在补码中，这两个数字的位模式均为二进制 `11000000`。

### 整数和浮点数转换

如果浮点数类型转换为整数类型，则小数部分被丢弃^[实际上---通常只是被正常丢弃]。

但---这里有个关键---如果数字太大而无法适应整数，您会得到未定义行为。所以不要这样做。

由整数或浮点数转换为浮点数，C会尽最大努力找到最接近整数的浮点数。

再次强调，如果原始值无法表示，则是未定义行为。

## 隐式转换

这些是编译器在您混合使用不同类型时自动为您执行的转换。

### 整数提升 {#integer-promotions}
```

在许多地方，如果`int`可以用于表示`char`或`short`（有符号或无符号）的值，那么该值会被_提升_为`int`。如果它在`int`中无法容纳，那么它会被提升为`unsigned int`。

这就是我们可以这样做的方式：

``` {.c}
char x = 10, y = 20;
int i = x + y;
```

在这种情况下，C会在进行数学运算之前将`x`和`y`提升为`int`。

整数提升发生在普通算术转换期间，包括可变参数函数^[具有可变数量参数的函数。]，一元`+`和`-`运算符，或将值传递给没有原型的函数时^[这很少会这样做，因为编译器会报错，而拥有原型是正确的做法。我认为这仍然有效是出于历史原因，在原型出现之前。]。

### 普通算术转换 {#usual-arithmetic-conversions}

这些是 C 在您请求的数值操作周围执行的自动转换。（顺便说一句，这确实是它们被称为的名称，根据 C11 §6.3.1.8。）请注意，对于本节，我们只讨论数值类型---字符串会在稍后介绍。

这些转换回答了当您混合类型时会发生什么情况的问题，就像这样：

``` {.c}
int x = 3 + 1.2;   // 混合int和double
                   // 4.2 被转换为int
                   // 4 存储在x中

float y = 12 * 2;  // 混合float和int
                   // 24 被转换为float
                   // 24.0 存储在y中
```

它们变成`int`吗？它们变成`float`吗？它是如何工作的？

这里是为了方便理解而改述的步骤。

1. 如果表达式中有一个是浮点类型，将其他内容转换为该浮点类型。

```c
// 否则，如果两个类型都是整数类型，则对每个执行整数提升，然后将操作数的类型变得足够大，以容纳最大的公共值。有时这涉及将有符号类型更改为无符号类型。

如果想了解具体细节，请查看C11 §6.3.1.8。但你可能并不关心。

只要记住整数类型在其中有浮点类型时会变成浮点类型，并且编译器会尽力确保混合整数类型不会溢出。

最后，如果从一种浮点类型转换为另一种，编译器将尝试进行精确转换。如果无法实现精确转换，它将尽力以最佳逼近方式处理。如果数字过大无法容纳转换为的类型，则会发生未定义行为！

### `void*`

`void*` 类型很有趣，因为它可以从任何指针类型转换为任何指针类型。

``` {.c}
int x = 10;

void *p = &x;  // &x 的类型是 int*，但我们将其存储在 void* 中

int *q = p;    // p 是 void*，但我们将其存储为 int*
```

[i[类型转换-->隐式]>]

## 显式转换

[i[类型转换-->显式]<]

这些是需要请求的从一种类型转换为另一种类型的转换；编译器不会自动进行。

您可以通过使用 `=` 将一个类型赋给另一个类型来进行类型转换。

您还可以使用 _强制转换_ 明确转换。

[i[类型转换-->数值]>]

### 强制转换

[i[类型转换-->强制转换]<]

您可以通过在表达式前面的括号中放置新类型来显式更改表达式的类型。一些C开发人员会对这种做法表示反感，除非绝对必要，但很可能你会在一些C代码中看到这种用法。

让我们来做一个例子，我们要将一个 `int` 转换为 `long`，以便我们可以将其存储在一个 `long` 中。
```

注意：这个示例是刻意编造的，在这种情况下进行强制转换是完全不必要的，因为`x + 12`表达式会自动转换为`long int`以匹配`y`的更宽类型。

``` {.c}
int x = 10;
long int y = (long int)x + 12;
```

在这个示例中，即使`x`之前是`int`类型，表达式`(long int)x`的类型是`long int`。我们可以说：“我们将`x`转换为`long int`。”

更常见的情况是，您可能会看到强制转换用于将`void*`转换为特定指针类型，以便对其进行解引用。

内置的`qsort()`函数中的回调可能展示这种行为，因为其中传递的是`void*`：

``` {.c}
int compar(const void *elem1, const void *elem2)
{
    if (*((const int*)elem2) > *((const int*)elem1)) return 1;
    if (*((const int*)elem2) < *((const int*)elem1)) return -1;
    return 0;
}
```

但您也可以通过赋值来清晰地编写：

``` {.c}
int compar(const void *elem1, const void *elem2)
{
    const int *e1 = elem1;
    const int *e2 = elem2;

    return *e2 - *e1;
}
```

另一个您经常会看到强制转换的地方是为了避免使用`%p`打印指针值时出现警告，因为`%p`除了`void*`之外对任何其他类型都很挑剔：

``` {.c}
int x = 3490;
int *p = &x;

printf("%p\n", p);
```

会生成这个警告：

``` {.default}
warning: format ‘%p’ expects argument of type ‘void *’, but argument
         2 has type ‘int *’
```

您可以通过强制转换来修复它：

``` {.c}
printf("%p\n", (void *)p);
```

另一个情况是在显式指针更改时，如果您不想使用中间的`void*`，但这种情况也非常罕见：

``` {.c}
long x = 3490;
long *p = &x;
unsigned char *c = (unsigned char *)p;
```

```c
// 用于字符转换函数的第三个常见场景是需要在 [fl[`<ctype.h>`|https://beej.us/guide/bgclr/html/split/ctype.html]] 中的字符转换函数中，将可能有疑问的有符号值转换为 `unsigned char` 以避免未定义行为。

// 再次强调，在实践中很少真的 _需要_ 进行强制转换。如果发现自己需要进行强制转换，可能有另一种方法可以实现相同的目的，或者可能是不必要的强制转换。

// 但也有可能是确实需要的。个人而言，我尽量避免使用强制转换，但必要时也会毫不犹豫地使用。

[i[Type conversions-->explicit]>]
[i[Type conversions-->casting]>]
[i[Type conversions]>]
```