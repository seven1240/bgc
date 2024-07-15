```c
<!-- Beej's C指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 你好，世界！

## 从C语言可以期望什么

> _“这阶梯通往哪里？”_ \
> _“通往上面。”_
>
> ---雷·斯坦兹和彼得·威尔科门，《捉鬼敢死队》

C是一种低级语言。

它过去不是这样。在那些人们用花岗岩雕刻穿孔卡的年代，C语言是一种摆脱像[flw[assembly|汇编语言]]那样低级语言枯燥的方式。

但现在在这个现代时代，目前的语言提供了许多1972年C语言问世时不存在的功能。这意味着C语言是一种基本的语言，功能不多。它可以做_任何事情_，但你得为此努力。

所以为什么今天还要使用它呢？

* 作为学习工具：C语言不仅是一段值得尊重的计算机历史，而且与现代语言不同，它与计算机内存的接口有着连接。当你学习C语言时，你将了解软件如何在低级别与计算机内存进行交互。没有安全带。你会写会崩溃的软件，我向你保证。那就是乐趣的一部分！

* 作为实用工具：C语言仍然被用于某些应用，比如构建[flw[操作系统|操作系统]]或在[flw[嵌入式系统|嵌入式系统]]中。（尽管[flw[Rust|Rust（编程语言）]]编程语言也在瞄准这两个领域！）

如果你熟悉另一种语言，C语言的很多东西都很容易。C语言启发了许多其他语言，在Go、Rust、Swift、Python、JavaScript、Java以及各种其他语言中都能看到它的影子。这些部分会让你感到亲切。
```

C 里让人卡壳的就是_指针_。几乎所有其他东西都很熟悉，但指针就是个怪兽。指针背后的概念可能是你已经知道的，但 C 要求你明确地表达它，使用你可能从未见过的操作符。

尤其阴险的是，一旦你[理解（Grok）||flw[_grok_|Grok]]指针，它们突然就变得简单了。但在那一刻之前，它们就像是滑溜的鳗鱼。

C 中的其他一切都只是记住另一种（有时甚至是相同的）做事情方式。指针就是个怪物。而且，可以说，甚至指针也只是一个你可能已经熟悉的主题的变化。

因此，准备好迎接一个距离机器核心最近的激动人心的冒险，这是不能不提到的影响最大的计算机语言之一^[我知道会有人和我争论，但它至少得是前三名吧？]。紧紧抓住！


## 你好，世界！

[i[你好，世界]()]这是一个 C 程序的典范示例。人人都用它。（请注意，左边的数字仅供读者参考，不是源代码的一部分。）

``` {.c .numberLines}
/* Hello world 程序 */

#include <stdio.h>

int main(void)
{
    printf("你好，世界！\n");  // 实际上在这里做点事情
}
```

我们将穿上长袖重型橡胶手套，拿起手术刀，深入剖析这个东西，看看是什么让它运转。所以，做好准备吧，因为我们要开始了。小心翼翼地剖开...

```c
/* 让我们先简单地解释一下：在`/*`和`*/`之间的任何内容都是注释，编译器会完全忽略它。`//`后面的任何内容也是一样。这让你可以给自己和其他人留下消息，这样当你在遥远的未来回头阅读你的代码时，你还能知道当初你到底想做什么。相信我，你会忘记的；这种事情经常发生。 */

/* 现在，`#include`是什么？ 看起来挺复杂的！它告诉C预处理器从另一个文件中提取内容并将其插入到代码的那个位置。 

等等---C预处理器是什么呢？好问题。 编译有两个阶段：预处理和编译。 井号开头的任何内容，或者说"井号"（`#`），是预处理器在编译器开始工作之前操作的内容。常见的预处理指令有`#include`和`#define`。更多内容稍后解释。*/

/* 在我们继续之前，为什么我会特意指出井号叫做八齿十字呢？答案很简单：我觉得八齿十字这个词非常有趣，所以我一有机会就会不经意地将它故意传播开来。八齿十字。八齿十字，八齿十字，八齿十字。*/
```

```c
// Translate the following content:
// So _anyway_. After the C preprocessor has finished preprocessing everything, the results are ready for the compiler to take them and produce [flw[assembly code|Assembly_language]], [flw[machine code|Machine_code]], or whatever it's about to do. Machine code is the "language" the CPU understands, and it can understand it _very rapidly_. This is one of the reasons C programs tend to be quick.

所以——无论如何。在C预处理器完成预处理工作后，结果就已经准备就绪，等待编译器接手并生成[flw[汇编代码|Assembly_language]]、[flw[机器码|Machine_code]]或者其他接下来会进行的操作。机器码是CPU能够理解的“语言”，而且CPU能够_非常快速_地理解它。这也是C程序往往表现迅速的原因之一。

// Translate the following content:
// Don't worry about the technical details of compilation for now; just know that your source runs through the preprocessor, then the output of that runs through the compiler, then that produces an executable for you to run.

现在不要担心编译的技术细节；只需要知道你的源代码会经过预处理器处理，然后处理后的结果会通过编译器，最终生成一个可执行文件供你运行。

// Translate the following content:
// What about the rest of the line? [i[`stdio.h` header file]<]What's `<stdio.h>`? That is what is known as a header file. It's the dot-h at the end that gives it away.  In fact it's the "Standard I/O" (`stdio`) header file that you will grow to know and love. It gives us access to a bunch of I/O functionality^[Technically, it contains preprocessor directives and function prototypes (more on that later) for common input and output needs.]. For our demo program, we're outputting the string "Hello, World!", so we in particular need access to the [i[`printf()` function]<] `printf()` function to do this. The `<stdio.h>` file gives us this access. Basically, if we tried to use `printf()` without `#include <stdio.h>`, the compiler would have complained to us about it.

那么剩下的一行呢？[`stdio.h`头文件]是什么意思？这就是所谓的头文件。它末尾的`.h`就是它的特征。实际上，这是“标准I/O”（stdio）头文件，你会逐渐熟悉并喜爱上它。它为我们提供了许多I/O功能的访问^[从技术角度讲，它包含了用于常见输入输出需要的预处理器指令和函数原型（稍后详细介绍）]。对于我们的演示程序，我们要输出字符串“Hello, World!”，因此我们特别需要使用[`printf()`函数]`printf()`函数来实现。`<stdio.h>`文件给了我们这个访问权限。基本上，如果我们在没有`#include <stdio.h>`的情况下尝试使用`printf()`，编译器就会对缺失的部分向我们抱怨。

// Translate the following content:
// How did I know I needed to `#include <stdio.h>` for `printf()`?[i[`printf()` function]>] Answer: it's in the documentation. If you're on a Unix system, `man 3 printf` and it'll tell you right at the top of the man page what header files are required. Or see the reference section in this book. `:-)` [`stdio.h` header file]>]

我是如何知道我需要为`printf()`加入`#include <stdio.h>`？[`printf()`函数]回答：这在文档中有说明。如果你使用Unix系统，输入`man 3 printf`，它会告诉你man手册开头列出了需要的头文件。或者查看本书的参考部分。`:-)`
```

```c
// 第一个代码行结束后终于要松口气了！但说实话，已经彻底剖析完毕。没有什么神秘可以留下！

// 所以稍微休息一下吧，回顾一下示例代码。只剩下几行简单的内容了。

// 欢迎回来！我知道你其实没有休息；我只是逗你玩。

// 下一行是 `main()` 函数。这是函数 `main()` 的定义；大括号 `{` 和 `}` 之间的所有内容都是函数定义的一部分。

// （无论如何，你是如何“调用”不同函数的呢？答案就在 `printf()` 行，不过我们等一会儿再说。）

// 现在，主函数在很多方面都很特殊，但有一个方面尤为突出：当程序开始执行时，它将被自动调用。在调用 `main()` 之前，不会调用任何你自己写的东西。在我们的示例中，这很好，因为我们只想打印一行内容然后退出。

// 噢，还有一件事：一旦程序执行超过了 `main()` 的结尾处的大括号，程序就会退出，然后你就会回到命令提示符。

// 现在，我们知道该程序引入了一个头文件 `stdio.h`，并声明了一个会在程序启动时执行的 `main()` 函数。那么 `main()` 函数中有什么好的东西呢？

// 很高兴你问到。真的！我们只有一个好东西：调用函数 `printf()`。你可以通过多种方式知道这是一个函数调用而不是函数定义，但其中一个指标是在它后面没有大括号。你用分号结束函数调用，这样编译器就知道这是表达式的结尾。你会看到几乎每个东西后面都要加上分号。
```

你在调用函数`printf()`时传递了一个参数：一个当你调用它时要打印的字符串。哦，耶 --- 我们在调用函数！我们很厉害！等等，等等 --- 别自以为是。那个字符串末尾的疯狂`\n`是什么？好吧，字符串中大多数字符将会像它们存储的那样打印出来。但是有一些特殊字符，你无法在屏幕上直接打印出来，它们被嵌入为两个字符的反斜杠代码。其中最流行的之一就是`\n`（读作"反斜杠-N"或简单地"换行符"），对应着换行字符。这个字符会导致进一步的打印在下一行的开头而不是当前行的末尾。就像在一行的末尾按下回车键一样。

所以将那段代码复制到一个名为`hello.c`的文件中，并进行构建。在类Unix平台（如Linux、BSD、Mac或WSL）上，你可以通过以下命令构建：

```
gcc -o hello hello.c
```

（这意味着"编译`hello.c`，然后输出一个名为`hello`的可执行文件"。）

一旦完成这个步骤，你将会得到一个名为`hello`的文件，可以通过以下命令运行：

```
./hello
```

（前面的`./`告诉shell从当前目录运行。）

看看会发生什么：

```
Hello, World! 
```

已经完成并通过了测试！发布吧！

## 编译细节

让我们再详细谈谈如何构建C程序以及在幕后发生了什么。

和其他编程语言一样，C有_源代码_。但是，取决于你来自的编程语言，你可能从未需要将你的源代码_编译_成一个_可执行文件_。

编译是将你的 C 源代码转换为操作系统可以执行的程序的过程。

JavaScript 和 Python 开发人员根本不习惯有单独的编译步骤——尽管在幕后是正在进行编译！Python 将你的源代码编译成称为 _字节码_ 的东西，Python 虚拟机可以执行。Java 开发人员习惯于编译，但这会生成 Java 虚拟机的字节码。

在编译 C 时，会生成 _机器码_。这是 CPU 可以直接快速执行的 1 和 0。

> 通常不经编译的语言被称为 _解释_ 语言。但正如我们提到的 Java 和 Python，它们也有一个编译步骤。并没有规定 C 不能被解释。(有 C 解释器存在！) 简而言之，这是一片灰色地带。总的来说，编译只是将源代码转换为另一种更易执行的形式。

C 编译器是进行编译的程序。

正如我们已经提到的，`gcc` 是安装在许多 [flw[类 Unix 操作系统|Unix]] 上的编译器。通常在终端命令行中运行，但不总是。你也可以在 IDE 中运行它。

那么我们如何进行命令行构建？

## 使用 `gcc` 进行构建

[i[`gcc` 编译器]]如果在当前目录中有一个名为 `hello.c` 的源文件，你可以通过在终端中键入以下命令将其构建为名为 `hello` 的程序：

``` {.zsh}
gcc -o hello hello.c
```

`-o` 表示“输出到该文件”^[如果您没有指定输出文件名，它将默认导出到名为 `a.out` 的文件---这个文件名深深植根于 Unix 的历史。]。并且在末尾还有 `hello.c`，这是我们想要编译的文件名。

如果您的源代码被拆分成多个文件，您可以将它们一起编译（几乎就像它们是一个文件一样，但实际规则比那更复杂），只需要将所有的 `.c` 文件放在命令行上：

``` {.zsh}
gcc -o awesomegame ui.c characters.c npc.c items.c
```
[i[`gcc` 编译器]>]

它们将会一起构建为一个大可执行文件。

这就足够让您开始了---稍后我们会详细讨论多个源文件、目标文件以及各种乐趣事项。[i[Compilation]>]

## 使用 `clang` 进行构建

在 Mac 上，系统自带的编译器不是 `gcc`---而是 `clang`[i[`clang` 编译器]]。但还安装了一个包装器，因此您可以运行 `gcc` 并让其正常工作。

您也可以通过[fl[Homebrew|https://formulae.brew.sh/formula/gcc]]或其他途径来正确安装 `gcc`[i[`gcc` 编译器]]编译器。

## 从集成开发环境构建

[i[集成开发环境]<]如果您使用的是 _集成开发环境_（IDE），则可能无需从命令行构建。

在 Visual Studio 中，`CTRL-F7` 会进行构建，`CTRL-F5` 会运行。

在 VS Code 中，您可以按 `F5` 通过调试器运行。 （您需要安装 C/C++ 扩展。）

在 XCode 中，您可以使用 `COMMAND-B` 进行构建，使用 `COMMAND-R` 运行。 要获取命令行工具，请搜索“XCode 命令行工具”，您将找到安装说明。

为了开始入门，我鼓励您也尝试从命令行构建---这是历史的延续![i[集成开发环境]>]

## C 版本

```c
|版本|描述|
|-----|--------------|
|K&R C|1978年发布的初始版本，以Brian Kernighan和Dennis Ritchie命名。Ritchie设计并编写了这种语言，Kernighan成为了这方面书籍的合著者。如今很少看到原始的K&R代码。如果你看到了，它会显得陌生，就像中古英语对现代英语读者的感觉一样。|
|**C89**，ANSI C，C90|1989年，美国国家标准学会（ANSI）发布了一份C语言规范，为当今的C设定了基调。一年后，国际标准化组织（ISO）接手发布了内容相同的C90版本。|
|C95|很少提及的C89补充版本，包括宽字符支持。|
|**C99**|首次大规模更新，增加了许多语言功能。大多数人记得的是添加了`//`风格的注释。截至撰写本文时，这是目前应用最广泛的C版本。|
|**C11**|这一重大版本更新包括Unicode支持和多线程技术。请注意，如果开始使用这些语言功能，可能会在那些仍停留在C99阶段的地方牺牲可移植性。但说实话，1999年已经过去好一阵子了。|
|C17，C18|C11的错误修正更新。C17似乎是官方名称，但出版被推迟到2018年。据我了解，这两者是可以互换使用的，但C17更受青睐。|
|C2x|接下来会发生什么呢！预计最终会成为C23。|
```

您可以通过`-std=`命令行参数强制GCC使用这些标准之一。如果您想让它对标准要求苛刻一点，可以添加`-pedantic`。

例如：

``` {.zsh}
gcc -std=c11 -pedantic foo.c
```

对于这本书，我使用带有所有警告设置的C2x编译程序：

``` {.zsh}
gcc -Wall -Wextra -std=c2x -pedantic foo.c
```