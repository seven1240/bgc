```c
<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# Unicode, Wide Characters, and All That

[i[Unicode]<]

在我们开始之前，请注意，这是 C 语言中一处活跃的语言发展领域，因为它正努力克服一些，嗯，_成长的烦恼_。当 C2x 推出时，这里的更新可能会发生变化。

大多数人基本上对一个看似简单的问题感兴趣，即，“我如何在 C 中使用某种字符集？”我们会解决这个问题。但正如我们即将看到的，它可能已经在您的系统上工作。或者您可能需要求助于第三方库。

本章中我们将讨论许多事项---有些是平台无关的，有些是特定于 C 语言的。

我们首先来概述一下我们将要讨论的内容：

* Unicode 背景
* 字符编码背景
* 源和执行字符集
* 使用 Unicode 和 UTF-8
* 使用其他字符类型，如 `wchar_t`、`char16_t` 和 `char32_t`

让我们开始吧！

## 什么是 Unicode？

在过去，美国和世界许多地方普遍使用 7 位或 8 位编码来表示内存中的字符。这意味着我们总共可以有 128 或 256 个字符（包括不可打印字符）。这对以美国为中心的世界来说是可以接受的，但事实证明其他地方实际上也有其他语系---谁知道呢？中文有超过 50,000 个字符，这在一个字节中是无法表示的。

因此人们想出了各种替代方法来表示他们自定义的字符集。这很好，但却变成了一个兼容性噩梦。

```

为了解决这个问题，Unicode 应运而生。一个字符集统治它们所有。
它无限延伸（实际上），所以我们永远不会用完空间来存储新字符。
它包含了中文、拉丁文、希腊文、楔形文字、国际象棋符号、表情符号... 几乎涵盖了一切！而且还在不断添加！

## 代码点

[i[Unicode-->代码点]<]

我想在这里谈论两个概念。有点令人困惑，因为它们都是数字... 但是请耐心听我解释。

让我们粗略地定义 _代码点_ 为表示一个字符的数值。 （代码点也可以代表不可打印的控制字符，但是请暂时理解为像字母“B”或字符“π”之类的东西。）

每个代码点代表一个独特的字符。每个字符都有一个与之关联的独特数值代码点。

举例来说，在Unicode中，数值66代表“B”，960代表“π”。其他不是Unicode的字符映射可能使用不同的值，但让我们忘记它们，集中精力学习Unicode，未来的趋势！

所以有一点很重要：有一个数字代表每个字符。在Unicode中，这些数字范围从0到100余万。

[i[Unicode-->代码点]>]

明白了吗？

因为我们即将改变视角。

## 编码

[i[Unicode-->编码]<]

如果你还记得，一个8位字节可以保存0-255之间的值。对于“B”来说很好，因为它是66，可以放进一个字节。但是“π”是960，一个字节放不下！我们需要另一个字节。如何在内存中存储这些信息呢？或者像195,024这样更大的数字呢？那会需要多个字节来存储。

关键问题是：这些数字如何在内存中表示？这就是我们所说的字符的_编码_。

```c
// So we have two things: one is the code point which tells us effectively
// the serial number of a particular character. And we have the encoding
// which tells us how we're going to represent that number in memory.

// 现在我们有两个要素：一个是代码点，有效地告诉我们特定字符的序号。
// 另一个是编码方式，告诉我们要如何在内存中表示这个数字。

// There are plenty of encodings. You can make up your own right now, if
// you want^[For example, we could store the code point in a big-endian
// 32-bit integer. Straightforward! We just invented an encoding! Actually
// not; that's what UTF-32BE encoding is. Oh well---back to the grind!].
// But we're going to look at some really common encodings that are in use
// with Unicode.

// 有很多种编码方式。如果愿意，你可以随时创造自己的编码方式。
// 比如，我们可以将代码点存储在大端序的32位整数中。简单明了！我们刚刚发明了一种编码方式！实际上并没有；这就是UTF-32BE编码。哎呀，回到现实吧！

[i[Unicode-->UTF-8]<]
[i[Unicode-->UTF-16]<]
[i[Unicode-->UTF-32]<]

|编码方式|描述|
|:-----------------:|:--------------------------------------------------------------|
|UTF-8|一个以字节为基础的编码，每个字符使用可变数量的字节。这是首选的编码方式。|
|UTF-16|每个字符16位[^091d]编码方式。|
|UTF-32|每个字符32位编码方式。|

[^091d]: 哎哟。技术上来说，它是可变宽度的---有一种方法可以通过将两个UTF-16字符放在一起来表示大于$2^{16}$的代码点。

With UTF-16 and UTF-32, the byte order matters, so you might see
UTF-16BE for big-endian and UTF-16LE for little-endian. Same for UTF-32.
Technically, if unspecified, you should assume big-endian. But since
Windows uses UTF-16 extensively and is little-endian, sometimes that is
assumed^[There's a special character called the _Byte Order Mark_ (BOM),
code point 0xFEFF, that can optionally precede the data stream and
indicate the endianess. It is not required, however.].

Let's look at some examples. I'm going to write the values in hex
because that's exactly two digits per 8-bit byte, and it makes it easier
to see how things are arranged in memory.
```

```c
|字符|代码点|UTF-16BE|UTF-32BE|UTF-16LE|UTF-32LE|UTF-8|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|`A`|41|0041|00000041|4100|41000000|41|
|`B`|42|0042|00000042|4200|42000000|42|
|`~`|7E|007E|0000007E|7E00|7E000000|7E|
|`π`|3C0|03C0|000003C0|C003|C0030000|CF80|
|`€`|20AC|20AC|000020AC|AC20|AC200000|E282AC|

在这里找找规律。请注意，UTF-16BE 和  UTF-32BE 就是**直接**使用 16 和 32 位值来表示代码点^[_在只有两个字节的字符中是这样的_]。

**小端** 和 **大端** 相同，只是字节排列顺序不同。

然后是 UTF-8。首先你可能会注意到，**单字节**代码点以单独一个字节表示。这挺好。同时你可能也会发现不同的代码点占用不同数量的字节。这就是**变长编码**。

所以一旦超过某个值，UTF-8 就会使用额外的字节来存储这些值。而且这些字节看起来与代码点的值也没有关联。

[flw[UTF-8 编码的细节|UTF-8]] 超出了本指南的范围，但只需要知道它每个代码点需要不同数量的字节来编码，并且这些字节的值**除了前 128 个代码点之外**不与代码点匹配。如果你真的想了解更多，[fl[Computerphile 有一个由 Tom
Scott 讲解的很棒的 UTF-8 视频|https://www.youtube.com/watch?v=MijmeoH9LT4]]。

从北美的角度看，Unicode 和 UTF-8 的一个很棒之处就是：它与 **7 位 ASCII 编码** 是**向后兼容**的！所以如果你习惯 ASCII，那么 UTF- 8 就一样！每个以 ASCII 编码的文件也是用 UTF-8 编码的！（但反过来就不行了，显然。）
```

可能是这最后一点，比其他任何因素都更推动UTF-8成为全球通用的编码方式。

## 源码和执行字符集 {#src-exec-charset}

在用C语言编程时，至少有三种字符集起作用：

* 你的代码存在在磁盘上的字符集。
* 编译器在开始编译时将其转换的字符集(称为_源字符集_)。这可能与磁盘上的字符集相同，也可能不同。
* 编译器将源字符集转换为执行时的字符集(称为_执行字符集_)。这可能与源字符集相同，也可能不同。

你的编译器可能有选项在构建时选择这些字符集。

源码和执行时的基本字符集将包含以下字符：

``` {.default}
A B C D E F G H I J K L M
N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m
n o p q r s t u v w x y z
0 1 2 3 4 5 6 7 8 9
! " # % & ' ( ) * + , - . / :
; < = > ? [ \ ] ^ _ { | } ~
空格 制表符 垂直制表符 换页符 换行符
```

这些是你可以在源码中使用的字符，保证100%可移植。

在执行时的字符集还会额外包含响铃、退格、回车和换行等字符。

但大多数人不会走到那种极端，会在源码和可执行文件中自由使用扩展字符集，尤其是现在Unicode和UTF-8变得更加普遍。我是说，基本字符集甚至不允许使用`@`、`$`或`` ` ``！

值得注意的是，要仅使用基本字符集输入 Unicode 字符是一件头疼的事情（虽然可以通过转义序列来实现）。

## 在 C 语言中的 Unicode {#unicode-in-c}

在讨论 C 中的编码之前，让我们从代码点角度来看 Unicode。在 C 中指定 Unicode 字符的方法，这些字符将被编译器转换为执行字符集^[据推测，编译器会尽最大努力将代码点转换为输出编码，但在规范中我找不到任何保证]。

那么我们该如何做呢？

比如欧元符号，代码点为 0x20AC。（我用十六进制写了它，因为在 C 中表示它有两种方式。）我们怎样把它放进我们的 C 代码中呢？

使用 `\u` 转义将其放入字符串中，例如 `"\u20AC"`（大小写对于十六进制无关紧要）。在 `\u` 之后必须放上**确切的四个**十六进制数字，必要时在前面用零补齐。

下面是一个示例：

``` {.c}
char *s = "\u20AC1.23";

printf("%s\n", s);  // €1.23
```

因此，`\u` 适用于 16 位的 Unicode 代码点，但对于大于 16 位的代码点呢？这时就需要大写形式：`\U`。

例如：

``` {.c}
char *s = "\U0001D4D1";

printf("%s\n", s);  // 打印数学字母 "B"
```

它与 `\u` 的使用方式相同，只不过有 32 位而非 16 位。以下两者是等效的：

``` {.c}
\u03C0
\U000003C0
```

再者，在编译过程中，这些被翻译为[i[字符集-->执行]]执行字符集。它们代表的是Unicode代码点，而不是任何特定的编码。此外，如果一个Unicode代码点在执行字符集中无法表示，编译器可以对其进行任何处理。

现在，你可能想知道为什么不能直接这样做：

``` {.c}
char *s = "€1.23";

printf("%s\n", s);  // €1.23
```

可能你确实可以，尤其是在使用现代编译器时。[i[字符集-->源码]]源字符集将会被编译器自动转换为[i[字符集-->执行]]执行字符集。但是，如果编译器发现任何不在其扩展字符集中的字符，编译器可能会报错，而€符号显然不在[i[字符集-->基础]]基础字符集中。

[i[`\u` Unicode转义]<]
[i[`\U` Unicode转义]<]

规范的警告：不能使用`\u`或`\U`来编码小于0xA0的任何代码点，除了0x24（`$`）、0x40（`@`）和0x60（`` ` ``）外——是的，这恰好是基础字符集中缺失的常见标点符号的三元组。显然，在即将发布的规范版本中，此限制会放宽。

[i[`\u` Unicode转义]>]
[i[`\U` Unicode转义]>]

最后，你也可以在你的代码标识符中使用这些，但有些限制。但我不想深入讨论这些。我们这一章主要关注字符串处理。

至此，关于C语言中的Unicode就介绍到这里（编码除外）。

## 迅速了解UTF-8之前的简短说明 {#utf8-quick}

[i[Unicode-->UTF-8]<]

```c
    /* 
    可能是你磁盘上的源文件、扩展的源字符和扩展的执行字符都是以UTF-8格式的。而你使用的库期望UTF-8。这就是UTF-8无处不在的光辉未来。

    如果是这样的情况，你也不介意在不同于此的系统上会导致不可移植性，那就直接运行吧。随意将Unicode字符放入你的源代码和数据中。使用普通的C字符串就好啦，幸福就在其中。

    很多事情会顺利进行（尽管不具备可移植性），因为UTF-8字符串可以像其他C字符串一样安全地以NUL结尾。但也许为了更轻松地处理字符而失去移植性是值得的权衡。

    但也有一些需要注意的地方：

    * 诸如`strlen()`只报告字符串中的字节数，而不一定是字符数。([i[`mbstowcs()`函数-->使用UTF-8]]当您将其转换为宽字符时，`mbstowcs()`会返回字符串中的字符数。POSIX进行了扩展，因此如果只想要字符计数，可以将第一个参数传递为`NULL`)。

    * 以下情况对于多字节字符不会正常工作：[i[`strtok()`函数-->使用UTF-8]]`strtok()`、[i[`strchr()`函数-->使用UTF-8]]`strchr()`（使用[i[`strstr()`函数-->使用UTF-8]]`strstr()`来替代）、`strspn()`类型的函数、[i[`toupper()`函数-->使用UTF-8]]`toupper()`、[i[`tolower()`函数-->使用UTF-8]]`tolower()`、[i[`isalpha()`函数-->使用UTF-8]]`isalpha()`类型的函数，也可能有更多。要小心任何操作字节的东西。
    */
```

```markdown
* [i[`printf()` 函数-->UTF-8]]`printf()` 的变种允许仅打印字符串的部分字节^[使用类似 `"%.12s"` 的格式说明符，例如。]。您希望确保打印正确数量的字节以便在字符边界结束。

* [i[`malloc()` 函数-->UTF-8]]如果您想为字符串分配空间，或为其声明`char`数组，请注意最大大小可能超出您的预期。每个字符最多可能占用[i[`MB_LEN_MAX` 宏]]`MB_LEN_MAX` 字节（来自 `<limits.h>`）--- 除了基本字符集中保证为一个字节的字符。

可能还有其他一些情况我还没有发现的。让我知道哪些陷阱存在...

[i[Unicode-->UTF-8]>]

## 不同种类的字符

我想介绍更多种类的字符。我们习惯于 `char`，对吧？

但这太简单了。让我们把事情搞得更复杂一些吧！耶！

### 多字节字符

[i[多字节字符]<]

首先，我想让你潜在地改变一下对字符串（`char`数组）的看法。这些是由多字节字符组成的 _多字节字符串_。

没错---您平常所说的字符串就是多字节的。当有人说“C 字符串”时，他们指的是“C 多字节字符串”。

即使字符串中的特定字符只有一个字节，或者字符串由单个字符组成，它也被称为多字节字符串。

例如：

``` {.c}
char c[128] = "Hello, world!";  // 多字节字符串
```

```c
// 我们这里说的是，基本字符集中没有的特定字符可能由多个字节组成。最多可以有[`MB_LEN_MAX`宏]（来自 `<limits.h>`）。当然，在屏幕上看起来只是一个字符，但它可能有多个字节。

你可以在里面抛入 Unicode 值，就像我们之前看到的：

char *s = "\u20AC1.23";

printf("%s\n", s);  // €1.23

但是这里有些奇怪的地方，因为看看这个：

// 使用 UTF-8 的[`strlen()`函数]<

char *s = "\u20AC1.23";  // €1.23

printf("%zu\n", strlen(s));  // 7!

字符串"€1.23"的长度是 `7`？！是的！嗯，在我的系统上是的！记住，`strlen()`返回的是字符串中的字节数，而不是字符数。（当我们接触“宽字符”时，会看到如何获取字符串中的字符数。）

// 使用 UTF-8 的[`strlen()`函数]>

请注意，虽然 C 允许单个多字节的`char`常量（而不是 `char*`），但这些行为因实现而异，你的编译器可能会警告。

例如，GCC 会对以下两行警告多字符字符常量，并且在我的系统上打印出 UTF-8 编码：

printf("%x\n", '€');
printf("%x\n", '\u20ac');

// 多字节字符>

### 宽字符 {#wide-characters}

// 宽字符<

如果你不是多字节字符，那么就是_宽字符_。

宽字符是一个单一值，可以唯一代表当前区域设置中的任何字符。它类似于 Unicode 代码点。但也可能不是。或者可能是。
```

基本上，多字节字符字符串是字节数组，而宽字符字符串是字符的数组。这样你就可以开始基于每个字符考虑，而不是基于每个字节考虑（后者当字符开始占据可变数量的字节时就变得很混乱）。

宽字符可以用许多类型来表示，但最为突出的是`wchar_t`。这是主要的类型。就像`char`一样，只是更宽。

你可能想知道如果无法确定是Unicode还是其他编码，那么如何让你在编写代码时更有灵活性？`wchar_t`打开了一些大门，因为有一套丰富的函数可以处理`wchar_t`字符串（比如获取长度等）而不必关心编码。

## 使用宽字符和`wchar_t`

是时候介绍一个新类型：`wchar_t`。这是主要的宽字符类型。还记得`char`只有一个字节吗？而一个字节可能不足以表示所有字符，对吧？这个类型就够了。

要使用`wchar_t`，需要包含`<wchar.h>`。

它有多少字节？嗯，这不是很清楚。可能是16位。也可能是32位。

但是，你可能会说---如果只有16位，那不能容纳所有Unicode码点，是吗？没错---确实不能。规范不要求如此。它只需要能够表示当前区域设置中的所有字符。

这可能在具有16位`wchar_t`的平台上（咳咳---Windows）导致Unicode问题。但这超出了本指南的范围。

你可以通过使用`L`前缀声明此类型的字符串或字符，并使用`%ls`（"ell ess"）格式说明符来打印它们。或者用`%lc`打印单个`wchar_t`。

``` {.c}
wchar_t *s = L"Hello, world!";
wchar_t c = L'B';

printf("%ls %lc\n", s, c);
```

现在——这些字符是以Unicode代码点的形式存储的，还是不是呢？
这取决于实现。但你可以通过宏[__STDC_ISO_10646__宏]`__STDC_ISO_10646__`来测试它们是否是。如果已定义这个宏，那么答案是，“它是Unicode！”

更详细地说，该宏中的值是一个形式为`yyyymm`的整数，让你知道你可以依赖于哪个Unicode标准——无论那个日期生效的标准是什么。

但你如何使用它们呢？

### 多字节到`wchar_t`转换

那么我们如何从面向字节的标准字符串转换为面向字符的宽字符串，以及反向转换呢？

我们可以使用一些字符串转换函数来实现这一点。

首先，这些函数中你会看到的一些命名约定：

* `mb`：多字节
* `wc`：宽字符
* `mbs`：多字节字符串
* `wcs`：宽字符字符串

因此，如果我们想要将一个多字节字符串转换为宽字符字符串，我们可以调用`mbstowcs()`。反之亦然：`wcstombs()`。




|转换函数|描述|
|-|-|
|[`mbtowc()`函数]`mbtowc()`|将多字节字符转换为宽字符。|
|[`wctomb()`函数]`wctomb()`|将宽字符转换为多字节字符。|
|[`mbstowcs()`函数]`mbstowcs()`|将多字节字符串转换为宽字符串。|
|[`wcstombs()`函数]`wcstombs()`|将宽字符串转换为多字节字符串。|

让我们做一个快速演示，将一个多字节字符串转换为宽字符字符串，并使用它们各自的函数比较字符串长度。

[`mbstowcs()`函数] 

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <wchar.h>
#include <string.h>
#include <locale.h>
```

```c
int main(void)
{
    setlocale(LC_ALL, "");

    char *mb_string = "The cost is \u20ac1.23";  // €1.23
    size_t mb_len = strlen(mb_string);

    wchar_t wc_string[128];  // Holds up to 128 wide characters

    size_t wc_len = mbstowcs(wc_string, mb_string, 128);

    printf("multibyte: \"%s\" (%zu bytes)\n", mb_string, mb_len);
    printf("wide char: \"%ls\" (%zu characters)\n", wc_string, wc_len);
}
```

在我的系统上，这会输出：

``` {.default}
multibyte: "The cost is €1.23" (19 bytes)
wide char: "The cost is €1.23" (17 characters)
```

（您的系统可能根据区域设置的不同在字节数上有所变化。）

值得注意的一点是，`mbstowcs()` 除了将多字节字符串转换为宽字符字符串之外，还返回宽字符字符串的长度（以字符计）。在符合 POSIX 标准的系统上，您可以利用特殊模式，仅返回给定多字节字符串的字符长度：只需将目的地设为 `NULL`，最大要转换的字符数设为 `0`（此值会被忽略）。

（在下面的代码中，我在使用我的扩展源字符集---您可能需要用 `\u` 转义序列替换它们。）

``` {.c}
setlocale(LC_ALL, "");

// 以下字符串有 7 个字符
size_t len_in_chars = mbstowcs(NULL, "§¶°±π€•", 0);

printf("%zu", len_in_chars);  // 7
```

这也是一种非便携式的 POSIX 扩展。
```

而且，当然，如果你想反过来转换，就用`wcstombs()`函数。

## 宽字符功能

一旦我们进入宽字符领域，我们就可以运用各种功能。我会在这里总结一堆函数，但基本上我们拥有的是多字节字符串函数的宽字符版本（比如，我们知道用于多字节字符串的`strlen()`；宽字符字符串则有[i['wcslen()'函数]]`wcslen()`。

### `wint_t`

[i[`wint_t`类型]<]

许多这些函数使用`wint_t`来存储单个字符，无论是传入还是返回的。

它在性质上与`wchar_t`相关。`wint_t`是一个整数，可以表示扩展字符集中的所有值，还有一个特殊的文件结尾字符，`WEOF`。

[i[`wint_t`类型]>]

这被许多面向单个字符的宽字符函数使用。

### I/O 流定向 {#io-stream-orientation}

[i[I/O流定向]<]

简而言之，不要混合使用面向字节的函数（如`fprintf()`）和面向宽字符的函数（如`fwprintf()`）。决定流是面向字节还是面向宽字符，并坚持使用那类I/O函数。

更详细的情况是：流可以是面向字节或面向宽字符的。当流首次创建时，它没有定向，但第一个读取或写入将设置定向。

如果首先使用宽操作（如`fwprintf()`），它会将流定向为宽字符。

如果首先使用字节操作（如`fprintf()`），它会将流定向为字节。

您可以通过调用`fwide（）`函数手动设置流的无方向性。您可以使用相同的函数获取流的方向。

如果需要在中途更改方向，则可以使用`freopen（）`。

### I/O 函数

通常包括`<stdio.h>`和`<wchar.h>`。

|I/O 函数|描述|
|-|-|
|`wprintf（）`|格式化的控制台输出。|
|`wscanf（）`|格式化的控制台输入。|
|`getwchar（）`|基于字符的控制台输入。|
|`putwchar（）`|基于字符的控制台输出。|
|`fwprintf（）`|格式化的文件输出。|
|`fwscanf（）`|格式化的文件输入。|
|`fgetwc（）`|基于字符的文件输入。|
|`fputwc（）`|基于字符的文件输出。|
|`fgetws（）`|基于字符串的文件输入。|
|`fputws（）`|基于字符串的文件输出。|
|`swprintf（）`|格式化的字符串输出。|
|`swscanf（）`|格式化的字符串输入。|
|`vfwprintf（）`|变参格式化的文件输出。|
|`vfwscanf（）`|变参格式化的文件输入。|
|`vswprintf（）`|变参格式化的字符串输出。|
|`vswscanf（）`|变参格式化的字符串输入。|
|`vwprintf（）`|变参格式化的控制台输出。|
|`vwscanf（）`|变参格式化的控制台输入。|
|`ungetwc（）`|将宽字符推回到输出流中。|
|`fwide（）`|获取或设置流的多字节/宽字符方向。|

### 类型转换函数

通常需要包含`<wchar.h>`标头文件。

| 转换函数 | 描述 |
|-|-|
| [i[`wcstod()` 函数]] `wcstod()` | 将字符串转换为 `double`。|
| [i[`wcstof()` 函数]] `wcstof()` | 将字符串转换为 `float`。|
| [i[`wcstold()` 函数]] `wcstold()` | 将字符串转换为 `long double`。|
| [i[`wcstol()` 函数]] `wcstol()` | 将字符串转换为 `long`。|
| [i[`wcstoll()` 函数]] `wcstoll()` | 将字符串转换为 `long long`。|
| [i[`wcstoul()` 函数]] `wcstoul()` | 将字符串转换为 `unsigned long`。|
| [i[`wcstoull()` 函数]] `wcstoull()` | 将字符串转换为 `unsigned long long`。|

### 字符串和内存复制函数

通常需要包含`<wchar.h>`标头文件。

| 复制函数 | 描述 |
|----|----------------------------------------------|
| [i[`wcscpy()` 函数]] `wcscpy()` | 复制字符串。|
| [i[`wcsncpy()` 函数]] `wcsncpy()` | 复制字符串，限制长度。|
| [i[`wmemcpy()` 函数]] `wmemcpy()` | 复制内存。|
| [i[`wmemmove()` 函数]] `wmemmove()` | 复制可能重叠的内存。|
| [i[`wcscat()` 函数]] `wcscat()` | 连接字符串。|
| [i[`wcsncat()` 函数]] `wcsncat()` | 连接字符串，限制长度。|

### 字符串和内存比较函数

通常需要包含`<wchar.h>`标头文件。

```c
|Comparing Function|Description|
|-------------------|---------------------------------------------------------------|
|[i[`wcscmp()` function]]`wcscmp()`|比较字符串的字典序。|
|[i[`wcsncmp()` function]]`wcsncmp()`|比较字符串的字典序，有长度限制。|
|[i[`wcscoll()` function]]`wcscoll()`|按照地域设置比较字符串的字典序。|
|[i[`wmemcmp()` function]]`wmemcmp()`|比较内存中的字典序。|
|[i[`wcsxfrm()` function]]`wcsxfrm()`|将字符串转换为使得`wcscmp()`表现像`wcscoll()`的形式。[^97d0]|

[^97d0]: `wcscoll()` 相当于`wcsxfrm()`之后再跟`wcscmp()`。

### String Searching Functions

通常包含 `<wchar.h>` 来调用这些函数。

|Searching Function|Description|
|-|-|
|[i[`wcschr()` function]]`wcschr()`|在字符串中查找字符。|
|[i[`wcsrchr()` function]]`wcsrchr()`|从后往前在字符串中查找字符。|
|[i[`wmemchr()` function]]`wmemchr()`|在内存中查找字符。|
|[i[`wcsstr()` function]]`wcsstr()`|在字符串中查找子字符串。|
|[i[`wcspbrk()` function]]`wcspbrk()`|在字符串中查找某个字符集合中的任意字符。|
|[i[`wcsspn()` function]]`wcsspn()`|找到包含给定字符集合中任意字符的子字符串的长度。|
|[i[`wcscspn()` function]]`wcscspn()`|找到在给定字符集合中出现之前的子字符串的长度。|
|[i[`wcstok()` function]]`wcstok()`|在字符串中查找标记。|

### Length/Miscellaneous Functions

通常包含 `<wchar.h>` 来调用这些函数。

|Length/Misc Function|Description|
|-|-|
|[i[`wcslen()` function]]`wcslen()`|返回字符串的长度。|
|[i[`wmemset()` function]]`wmemset()`|在内存中设置字符。|
|[i[`wcsftime()` function]]`wcsftime()`|格式化日期和时间输出。|

### Character Classification Functions

这些函数需包含 `<wctype.h>`。
```

|长度/其他功能|描述|
|-|-|
|[i[`iswalnum()` 函数]]`iswalnum()`|如果字符是字母数字，则为真。|
|[i[`iswalpha()` 函数]]`iswalpha()`|如果字符是字母，则为真。|
|[i[`iswblank()` 函数]]`iswblank()`|如果字符是空格（类似空格，但不是换行符），则为真。|
|[i[`iswcntrl()` 函数]]`iswcntrl()`|如果字符是控制字符，则为真。|
|[i[`iswdigit()` 函数]]`iswdigit()`|如果字符是数字，则为真。|
|[i[`iswgraph()` 函数]]`iswgraph()`|如果字符可打印（空格除外），则为真。|
|[i[`iswlower()` 函数]]`iswlower()`|如果字符是小写，则为真。|
|[i[`iswprint()` 函数]]`iswprint()`|如果字符可打印（包括空格），则为真。|
|[i[`iswpunct()` 函数]]`iswpunct()`|如果字符是标点符号，则为真。|
|[i[`iswspace()` 函数]]`iswspace()`|如果字符是空白字符，则为真。|
|[i[`iswupper()` 函数]]`iswupper()`|如果字符是大写，则为真。|
|[i[`iswxdigit()` 函数]]`iswxdigit()`|如果字符是十六进制数字，则为真。|
|[i[`towlower()` 函数]]`towlower()`|将字符转换为小写。|
|[i[`towupper()` 函数]]`towupper()`|将字符转换为大写。|

## 解析状态，可重启功能

[i[多字节字符-->解析状态]<]

我们将深入了解多字节转换的内部工作原理，虽然概念性地理解这一点很重要。

想象一下，你的程序如何获取一系列多字节字符并将它们转换为宽字符，或者反之亦然。在某个时刻，它可能正在解析一个字符的中间部分，或者在确定最终值之前可能需要等待更多字节。

此解析状态存储在类型为 `mbstate_t` 的不透明变量中，并且在每次执行转换时使用。这是转换函数跟踪处理进程的方式。

如果在处理过程中切换到不同的字符序列，或尝试定位到输入序列中的其他位置，可能会因此感到困惑。

现在你可能会质疑我：我们刚才进行了一些转换操作，但我并没有提到任何 `mbstate_t`。

这是因为像 `mbstowcs()`、`wctomb()` 等转换函数各自具有自己的 `mbstate_t` 变量。不过，每个函数只有一个此类变量，因此在编写多线程代码时，不能安全地使用它们。

幸运的是，C定义了可重新启动版本的这些函数，如果需要，您可以按线程传递自己的 `mbstate_t`。如果正在进行多线程开发，请使用这些版本！

关于初始化 `mbstate_t` 变量的快速说明：只需使用 `memset()` 将其初始化为零即可。没有内置函数可以强制对其进行初始化。

``` {.c}
mbstate_t mbs;

// 将状态设置为初始状态
memset(&mbs, 0, sizeof mbs);
```

以下是可重新启动的转换函数列表---请注意在“源”类型后面加上“r”的命名约定：

* `mbrtowc()`---多字节转宽字符
* `wcrtomb()`---宽字符转多字节
* `mbsrtowcs()`---多字节字符串转宽字符字符串
* `wcsrtombs()`---宽字符字符串转多字节字符串

这些函数与其不可重新启动的对应函数非常相似，不同之处在于需要传入指向您自己 `mbstate_t` 变量的指针。此外，它们还会修改源字符串指针（以帮助您处理无效字节），因此保存原始副本可能会有用。

以下是前一章节中的示例，经过重新处理以接受我们自己的`mbstate_t`。

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>
#include <wchar.h>
#include <string.h>
#include <locale.h>

int main(void)
{
    // 退出 C 语言环境，切换到一个可能包含欧元符号的语言环境
    setlocale(LC_ALL, "");

    // 包含欧元符号（Unicode 码点 20ac）的原始多字节字符串
    char *mb_string = "The cost is \u20ac1.23";  // €1.23
    size_t mb_len = strlen(mb_string);

    // 用于保存转换后字符串的宽字符数组
    wchar_t wc_string[128];  // 可容纳最多 128 个宽字符

    // 设置转换状态
    mbstate_t mbs;
    memset(&mbs, 0, sizeof mbs);  // 初始状态

    // mbsrtowcs() 会修改输入指针，使其指向第一个无效字符，如果成功则指向 NULL。我们复制一个指针作为 mbsrtowcs() 的输入，以便保持原始指针不变。
    //
    // 这个示例可能会成功，但我们会在后面进行检查。
    const char *invalid = mb_string;

    // 将多字节字符串转换为宽字符; 这将返回宽字符的数量
    size_t wc_len = mbsrtowcs(wc_string, &invalid, 128, &mbs);

    if (invalid == NULL) {
        printf("未发现无效字符\n");

        // 打印结果--注意宽字符字符串的 %ls
        printf("多字节字符串: \"%s\" （%zu 字节）\n", mb_string, mb_len);
        printf("宽字符: \"%ls\" （%zu 个字符）\n", wc_string, wc_len);
    } else {
        ptrdiff_t offset = invalid - mb_string;
        printf("位于偏移量 %td 处的无效字符\n", offset);
    }
}
```

``` {.c}
mbstowcs(NULL, NULL, 0);   // 重置 mbstowcs() 的解析状态
mbstowcs(dest, src, 100);  // 解析一些内容
```

对于 I/O，每个宽字符流都管理自己的 `mbstate_t` 并在进行输入和输出转换时使用它。

而一些字节导向的 I/O 函数，比如 `printf()` 和 `scanf()` 在执行任务时会维护自己的内部状态。

最后，这些可重新启动的转换函数实际上会有自己的内部状态，如果你给 `mbstate_t` 参数传入 `NULL` 的话。这会让它们更像不可重新启动的对应函数。

[i[多字节字符-->解析状态]>]
[i[宽字符]>]

## Unicode 编码和 C

在这一部分，我们将看到当涉及到三种具体的 Unicode 编码时，C 能做什么（和不能做什么）。

### UTF-8

[i[Unicode-->UTF-8]<]

在进入本节之前，请阅读上面的[UTF-8 简要说明](#utf8-quick)以便温习一下。

除此以外，C 对 UTF-8 有哪些能力呢？

很不幸，并不多。

[i[`u8` UTF-8前缀]<]

你可以告诉 C 你明确希望一个字符串文字以 UTF-8 编码，它会为你做这件事。你可以使用 `u8` 来给字符串添加前缀：

``` {.c}
char *s = u8"Hello, world!";

printf("%s\n", s);   // Hello, world!--如果你可以输出 UTF-8
```

那么，你能在其中放入 Unicode 字符吗？

``` {.c}
char *s = u8"€123";
```

[i[`u8` UTF-8前缀]>]

当然可以！如果扩展源字符集支持。 （gcc 支持。）

如果它不支持呢？你可以用友好邻邦的 `\u` 和 `\U` 指定 Unicode 代码点，[就像上面提到的那样](#unicode-in-c)。

但就是这样。在标准库中没有便捷的方法可以将任意输入转换为UTF-8，除非您的区域设置是UTF-8。或者解析UTF-8，除非您的区域设置是UTF-8。

所以，如果您想要这样做，要么是在UTF-8的区域设置中：

``` {.c}
setlocale(LC_ALL, "");
```

要么在本地机器上找出UTF-8区域设置名称并显式设置，就像这样：

``` {.c}
setlocale(LC_ALL, "en_US.UTF-8");  // 非便携式名称
```

或者使用[第三方库](#utf-3rd-party)。

[i[Unicode-->UTF-8]>]

### UTF-16、UTF-32、`char16_t`和`char32_t`

[i[Unicode-->UTF-16]<]
[i[Unicode-->UTF-32]<]
[i[`char16_t` 类型]<]
[i[`char32_t` 类型]<]

`char16_t`和`char32_t`是另外两种潜在的宽字符类型，分别具有16位和32位的大小。并不一定是宽字符，因为如果它们无法表示当前区域设置中的每个字符，则会失去宽字符的特性。但规范中将它们称为"宽字符"类型，所以我们就这样吧。

这些类型在一定程度上使得处理Unicode更加友好。

要使用，包含`<uchar.h>`。(是"u"，不是"w"。)

在 macOS 中没有这个头文件---很遗憾。如果您只需要这些类型，可以：

``` {.c}
#include <stdint.h>

typedef int_least16_t char16_t;
typedef int_least32_t char32_t;
```

但如果您也需要函数，那就得自己搞定了。

[i[带有`u`的Unicode前缀]<]
[i[带有`U`的Unicode前缀]<]

假设您一切就绪，您可以用带有`u`和`U`前缀声明这些类型的字符串或字符：

``` {.c}
char16_t *s = u"Hello, world!";
char16_t c = u'B';

char32_t *t = U"Hello, world!";
char32_t d = U'B';
```

[i[`char32_t` 类型]>]
[i[带有`u`的Unicode前缀]>]
[i[带有`U`的Unicode前缀]>]

```c
现在---这些值存储在UTF-16还是UTF-32中？这取决于实现。

但是您可以测试一下它们是哪种。如果宏 [i[`__STDC_UTF_16__`宏]] `__STDC_UTF_16__` 或者 [i[`__STDC_UTF_32__ 宏`]] `__STDC_UTF_32__` 被定义为 `1`，那意味着这些类型分别保存UTF-16或UTF-32。

如果你好奇，我知道你很好奇，这些值，如果是UTF-16或UTF-32，将以本机字节顺序存储。也就是说，您应该能够直接将它们与Unicode代码点值进行比较：

``` {.c}
char16_t pi = u"\u03C0";  // 圆周率符号

#if __STDC_UTF_16__
pi == 0x3C0;  // 始终为真
#else
pi == 0x3C0;  // 可能不为真
#endif
```

[i[`char16_t`类型]>]
[i[Unicode-->UTF-16]>]
[i[Unicode-->UTF-32]>]

### 多字节转换

您可以使用许多辅助函数将您的多字节编码转换为`char16_t`或`char32_t`。

(就像我说的，不过，结果可能不是UTF-16或UTF-32，除非相应的宏被设置为`1`。)

所有这些函数都是可重新启动的（即，您传入自己的`mbstate_t`），并且它们逐字符操作^[有点复杂---对于多个`char16_t` UTF-16编码，情况可能会有所不同。]。

|转换函数|描述|
|-|-|
|[i[`mbrtoc16()`函数]]`mbrtoc16()`|将多字节字符转换为`char16_t`字符。|
|[i[`mbrtoc32()`函数]]`mbrtoc32()`|将多字节字符转换为`char32_t`字符。|
|[i[`c16rtomb()`函数]]`c16rtomb()`|将`char16_t`字符转换为多字节字符。|
|[i[`c32rtomb()`函数]]`c32rtomb()`|将`char32_t`字符转换为多字节字符。|

### 第三方库 {#utf-3rd-party}
```

```c
/*
For heavy-duty conversion between different specific encodings, there
are a couple mature libraries worth checking out. Note that I haven't
used either of these.

* [flw[iconv|Iconv]]---Internationalization Conversion, a common
  POSIX-standard API available on the major platforms.
* [fl[ICU|http://site.icu-project.org/]]---International Components for
  Unicode. At least one blogger found this easy to use.

If you have more noteworthy libraries, let me know.

[i[Unicode]>]
*/
```

在处理不同特定编码之间的繁重转换时，有几个成熟的库值得一试。请注意，我尚未使用这两个库。

* [flw[iconv|Iconv]]---国际化转换，这是一个常见的 POSIX 标准 API，在主要平台上都可以使用。
* [fl[ICU|http://site.icu-project.org/]]---联合国际化组件。至少有一位博主发现这个库易于使用。

如果您有更值得注意的库，请告诉我。

[i[Unicode]>]