<!-- Beej的C指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 外部环境

当你运行一个程序时，实际上是你在与 shell 交流，说：“嘿，请运行这个东西。” 然后 shell 说：“好的”，然后告诉操作系统：“嘿，请创建一个新进程并运行这个东西。” 如果一切顺利，操作系统会执行并运行你的程序。

但在 shell 外部还有一个完整的世界，可以通过C语言从内部与之交互。 在这一章节中，我们将看一些相关内容。

## 命令行参数

命令行实用程序接受_命令行参数_。例如，如果我们想要查看所有以`.txt`结尾的文件，我们可以在类 Unix 系统上输入类似的命令：

``` {.zsh}
ls *.txt
```

(在 Windows 系统上用 `dir` 代替 `ls`)。

在这种情况下，命令是 `ls`，但它的参数是所有以`.txt`结尾的文件^[从历史上看，MS-DOS和Windows程序会不同于Unix。在Unix中，shell会在程序看到之前将通配符扩展为所有匹配的文件，而微软的变种会将通配符表达式传递给程序来处理。无论如何，都会有被传递到程序的参数]。

那么我们如何查看从命令行传递给程序的内容呢？

假设我们有一个名为 `add` 的程序，它会对命令行上传递的所有数字进行相加并打印结果：

``` {.zsh}
./add 10 30 5
45
```

那肯定可以付账单！

但说真的，这是一个很好的工具，可以帮助我们看到如何从命令行获取这些参数并对其进行解析。

首先，让我们看看如何获取这些参数。为此，我们需要一个新的 `main()`！

这是一个打印所有命令行参数的程序。例如，如果我们将可执行文件命名为 `foo`，我们可以这样运行它：

``` {.zsh}
./foo i like turtles
```

然后我们会看到这个输出：

``` {.default}
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

有点奇怪，因为第零个参数是可执行文件的名字。但这只是需要习惯的事情。参数本身会直接跟在后面。

源代码：

[i[`argc` 参数]<]
[i[`argv` 参数]<]
[i[`main()` 函数-->命令行选项]<]

``` {.c .numberLines}
#include <stdio.h>

int main(int argc, char *argv[])
{
    for (int i = 0; i < argc; i++) {
        printf("arg %d: %s\n", i, argv[i]);
    }
}
```

哇！`main()` 函数的签名是什么意思？`argc` 和 `argv`^[由于它们只是常规的参数名，你实际上不必称它们为 `argc` 和 `argv`。但使用这些名称非常习惯，如果你变得有创意，其他 C 程序员会怀疑地看着你！]（读作 _arg-cee_ 和 _arg-vee_ ）？

我们先从简单的开始：`argc`。这是参数的数量，包括程序名本身。如果你将所有参数视为一个字符串数组，这也确实就是它们的类型，那么你可以认为 `argc` 是该数组的长度，因为它确实如此。

因此，在那个循环中，我们遍历了所有的 `argv` 并逐个打印它们出来，所以对于给定的输入：

``` {.zsh}
./foo i like turtles
```

我们会得到对应的输出：

``` {.default}
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

有了这个理解，我们应该可以利用这个加法程序了。

我们的计划：

- 查看所有命令行参数（超过 `argv[0]`，程序名）
- 转换为整数
- 加到一个运行总数
- 打印结果

让我们开始吧！

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;

    for (int i = 1; i < argc; i++) {  // 从 1 开始，第一个参数
        int value = atoi(argv[i]);    // 使用 strtol() 进行更好的错误处理

        total += value;
    }

    printf("%d\n", total);
}
```

[注释: `main()` 函数-->命令行选项]
[注释: `argc` 参数]

示例运行：

``` {.zsh}
$ ./add
0
$ ./add 1
1
$ ./add 1 2
3
$ ./add 1 2 3
6
$ ./add 1 2 3 4
10
```

当然，如果传入非整数可能会出错，但如何防范留给读者练习。

### `argv` 的最后一个是 `NULL`

关于 `argv` 的一个有趣小知识是，在最后一个字符串后面是指向 `NULL` 的指针。

也就是说：

``` {.c}
argv[argc] == NULL
```

这总是成立的！

这可能看起来毫无意义，但事实证明在一些地方会有用；我们现在介绍一个使用方法。

### 另一种写法：`char **argv`

记住，当调用函数时，C 不区分函数签名中的数组表示法和指针表示法。

也就是说，以下两者相同：

``` {.c}
void foo(char a[])
void foo(char *a)
```

现在，将 `argv` 视为字符串数组，即 `char*` 数组，是很方便的，所以这样写也讲得通：

``` {.c}
int main(int argc, char *argv[])
```

但由于等价性，你也可以写成：

``` {.c}
int main(int argc, char **argv)
```

是的，那是指针的指针！如果这样更容易理解，可以将其看作是指向字符串的指针。但实际上，它是指向指向 `char` 的值的指针。

另外要记住这些是等效的：

``` {.c}
argv[i]
*(argv + i)
```

这意味着你可以在 `argv` 上进行指针算术运算。

因此，消耗命令行参数的另一种方法可能是只需沿着 `argv` 数组移动指针，直到碰到末尾的 `NULL`。

让我们修改我们的加法器来实现这一点：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;
    
    // 一个巧妙的技巧，让编译器停止警告未使用的变量 argc：
    (void)argc;

    for (char **p = argv + 1; *p != NULL; p++) {
        int value = atoi(*p);  // 为了更好的错误处理，使用 strtol()

        total += value;
    }

    printf("%d\n", total);
}
```

个人而言，我使用数组表示法来访问 `argv`，但也见过这种风格在使用。

### 有趣的事实

[i[`argc` 参数]<]

关于 `argc` 和 `argv` 的还有一些事情。

* 有些环境可能不会将 `argv[0]` 设为程序名称。如果不可用，`argv[0]` 将是一个空字符串。我从未见过这种情况发生。

* 规范实际上对实现如何处理 `argv` 及这些值的来源有很大的自由度。但我在的每个系统都是以我们在本节中讨论的方式工作。

* 你可以修改 `argc`、`argv` 或 `argv` 指向的任何字符串。 (只是不要使这些字符串比它们原本更长！)

* 在某些类 Unix 系统上，修改字符串 `argv[0]` 会导致 `ps` 命令的输出发生变化^[`ps`， Process Status，是一个 Unix 命令，用于查看当前正在运行的进程。]。

通常，如果你有一个名为 `foo` 的程序，你使用 `./foo` 运行它，你可能在 `ps` 命令的输出中看到这样的内容：

``` {.default}
   4078 tty1     S      0:00 ./foo
  ```

  但是如果你像这样修改`argv[0]`，要小心确保新的字符串 `"Hi!  "` 和旧的字符串 `"./foo"` 长度相同：

  ``` {.c}
  strcpy(argv[0], "Hi!  ");
  ```

  然后在程序`./foo`仍在执行时运行`ps`，我们将会看到这样：

  ``` {.default}
   4079 tty1     S      0:00 Hi!  
  ```

  这种行为不在规范范围内，且高度依赖系统。

[i[`argc`参数]>]
[i[`argv`参数]>]
[i[命令行参数]>]

## 退出状态 {#exit-status}

[i[退出状态]<]

你有没有注意到`main()`的函数签名中指定返回类型为`int`？这是怎么回事呢？这与一个叫做 _退出状态_ 的东西有关，退出状态是一个可以返回给启动你的程序的整数，让它知道事情进行得如何。

在C中，程序有很多种方式退出，包括从`main()`中`return`，或调用`exit()`的变体之一。

所有这些方法都接受一个`int`作为参数。

[i[`main()`函数-->从中返回]<]
顺便说一句：你有没有注意到在我几乎所有的示例中，即使`main()`应该返回一个 `int`，实际上我并没有`return`任何东西？在任何其他函数中这将是非法的，但在C中有一个特殊情况：如果执行到`main()`的末尾而没有找到`return`，它会自动执行`return 0`。
[i[`main()`函数-->从中返回]>]

但是`0`代表什么？我们可以在那里放入什么其他数字？它们是如何使用的？

规范对此问题既明确又模糊，这是常见的。明确是因为它详细说明了你可以做什么，但模糊在于它并没有特别限制它。

唯一的办法就是_继续努力_并弄清楚吧！

让我们来深入了解一下: 原来当你运行你的程序时，_你是从另一个程序中运行的_。

通常这个另一个程序是某种类型的[flw[shell|Shell_(computing)]]，它本身并不做什么，只是启动其他程序。

但这是一个多阶段的过程，尤其在命令行 shell 中特别明显：

1. Shell 启动你的程序
2. Shell 通常会进入睡眠状态 (对于命令行 shell)
3. 你的程序运行
4. 你的程序终止
5. Shell 唤醒并等待另一个命令

现在，在步骤 4 和 5 之间会有一些通信发生：程序可以返回一个 _状态值_，供 shell 查询。通常，这个值用来指示你的程序成功或失败，并且如果失败，是什么类型的失败。

这个值就是我们从 `main()` 中 `return` 出去的。就是状态。

现在，C 标准允许有两种不同的状态值，在 `<stdlib.h>` 中有宏定义：

[i[`EXIT_SUCCESS` 宏]<]
[i[`EXIT_FAILURE` 宏]<]

|状态|描述|
|-|-|
|`EXIT_SUCCESS` 或 `0`|程序正常终止。|
|`EXIT_FAILURE`|程序因错误而终止。|

让我们写一个简短的程序，用于从命令行中将两个数字相乘。我们要求你一定要指定两个数值。如果你没有指定，我们将打印一个错误消息，并以错误状态退出。

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    if (argc != 3) {
        printf("使用方法: mult x y\n");
        return EXIT_FAILURE;   // 指示给 shell 是没成功的
    }

    printf("%d\n", atoi(argv[1]) * atoi(argv[2]));

    return 0;  // 和 EXIT_SUCCESS 一样，一切顺利。
}
```

`EXIT_SUCCESS` 宏  
`EXIT_FAILURE` 宏  

现在如果我们尝试运行这段代码，直到我们指定了正确数量的命令行参数之前，我们会得到预期的效果：

``` {.zsh}
$ ./mult
usage: mult x y

$ ./mult 3 4 5
usage: mult x y

$ ./mult 3 4
12
```

从 Shell 获取退出状态

但这并没有真正显示我们返回的退出状态，对吗？不过我们可以让 Shell 打印出来。假设你在运行 Bash 或其他 POSIX shell，你可以使用 `echo $?` 命令来查看它^[在 Windows 的 `cmd.exe` 中，键入 `echo %errorlevel%`。在 PowerShell 中，键入 `$LastExitCode`.]。

让我们试试：

``` {.zsh}
$ ./mult
usage: mult x y
$ echo $?
1

$ ./mult 3 4 5
usage: mult x y
$ echo $?
1

$ ./mult 3 4
12
$ echo $?
0
```

从 Shell 获取退出状态

有趣！我们发现在我的系统上，`EXIT_FAILURE` 宏 是 `1`。规范没有明确说明这一点，所以它可能是任何数字。但是尝试一下；在你的系统上也可能是 `1`。

### 其他退出状态值

状态值 `0` 明确表示成功，但其他整数呢，甚至是负数呢？

在这里我们离开了 C 规范，进入了 Unix 领域。一般来说，`0` 表示成功，一个正的非零数字表示失败。所以你只能有一种成功状态，但可以有多种失败状态。Bash 表示退出码应该在 0 到 255 之间，虽然一些代码是保留的。

简而言之，如果你想在 Unix 环境中表示不同的错误退出状态，你可以从 `1` 开始，逐渐递增。

在 Linux 上，如果尝试使用范围在 0-255 之外的任何代码，它将对代码执行 `0xff` 的按位与操作，有效地将其夹在该范围内。

你可以编写脚本让 Shell 以后使用这些状态代码来决定下一步该做什么。

## 环境变量

在我开始之前，我需要提醒你，C语言没有明确规定环境变量是什么。因此，我将描述我所知道的每个主要平台上通用的环境变量系统。

基本上，环境是要运行你的程序的程序，比如bash shell。它可能已经定义了一些bash变量。以防你不知道，shell可以创建它自己的变量。每个shell都不同，但在bash中你可以键入`set`来显示所有的变量。

这是我bash shell中定义的61个变量的摘录：

``` 
HISTFILE=/home/beej/.bash_history
HISTFILESIZE=500
HISTSIZE=500
HOME=/home/beej
HOSTNAME=FBILAPTOP
HOSTTYPE=x86_64
IFS=$' \t\n'
```

注意它们是以键/值对的形式存在的。例如，一个键是`HOSTTYPE`，其值是`x86_64`。 从C语言的角度看，所有的值都是字符串，即使它们是数字^[如果你需要一个数值，请使用`atoi()`或者`strtol()`之类的函数将字符串转换为数值]。

因此，总而言之，你可以从你的C程序内部获取这些值。

`getenv()` 函数

让我们编写一个程序，使用标准的`getenv()`函数查找你在shell中设置的值。

`getenv()`将返回一个指向值字符串的指针，或者如果环境变量不存在则返回`NULL`。

```c
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char *val = getenv("FROTZ");  // 尝试获取值

    // 检查变量是否存在
    if (val == NULL) {
        printf("无法找到FROTZ环境变量\n");
        return EXIT_FAILURE;
    }

    printf("值：%s\n", val);
}
```

如果我直接运行这个代码，会得到如下输出：

``` {.zsh}
$ ./foo
无法找到 FROTZ 环境变量
```

这是合理的，因为我还没有设置它。

在bash中，我可以将它设置为某个值^【在Windows的CMD.EXE中，使用`set FROTZ=value`。在PowerShell中，使用`$Env:FROTZ=value`。】:

``` {.zsh}
$ export FROTZ="C is awesome!"
```

然后如果我再次运行它，会得到：

``` {.zsh}
$ ./foo
值: C is awesome!
```

通过这种方式，你可以在环境变量中设置数据，并在你的C代码中获取并相应地修改行为。

### 设置环境变量

这不是标准的方法，但很多系统提供了设置环境变量的方法。

如果是类Unix系统，查看[i[`putenv()` 函数]]`putenv()`、[i[`setenv()`]`setenv()`和[i[`unsetenv()` 函数]]`unsetenv()`的文档。在Windows系统中，参考[i[`_putenv()` 函数]]`_putenv()`。

### 类Unix系统的替代环境变量

如果你使用的是类Unix系统，很可能有其他方法可以获取环境变量。请注意，尽管规范指出这是一种常见扩展，但它并不真正属于C标准。然而，它是POSIX标准的一部分。

[i[`environ` 变量]<]

其中一种方法是使用一个名为`environ`的变量，必须这样声明：

``` {.c}
extern char **environ;
```

它是一个以`NULL`指针结尾的字符串数组。

在使用之前应该自己声明它，否则你可能会在非标准的`<unistd.h>`头文件中找到它。

每个字符串的格式是`"key=value"`，所以如果想获取键和值，你需要自己分割和解析。

以下是循环遍历和打印环境变量的两种不同方法的示例：

``` {.c .numberLines}
#include <stdio.h>
```

```c
extern char **environ;  // 必须是 extern，并命名为 "environ"

int main(void)
{
    for (char **p = environ; *p != NULL; p++) {
        printf("%s\n", *p);
    }

    // 或者你也可以这样做：
    for (int i = 0; environ[i] != NULL; i++) {
        printf("%s\n", environ[i]);
    }
}
```

对应的输出可能会是这样：

```plaintext
SHELL=/bin/bash
COLORTERM=truecolor
TERM_PROGRAM_VERSION=1.53.2
LOGNAME=beej
HOME=/home/beej
... 等等 ...
```

尽可能使用 `getenv()`，因为它更具可移植性。但如果必须遍历环境变量，使用 `environ` 可能是一种方法。

另一种非标准的获取环境变量的方式是将其作为 `main()` 的一个参数。它的工作方式类似，但你不需要添加 `extern` 的 `environ` 变量。截至我所知，即使 POSIX 规范也不支持这种用法，但在 Unix 领域很常见。

```c
#include <stdio.h>

int main(int argc, char **argv, char **env)  // <-- env!
{
    (void)argc; (void)argv;  // 抑制未使用的警告

    for (char **p = env; *p != NULL; p++) {
        printf("%s\n", *p);
    }

    // 或者你也可以这样做：
    for (int i = 0; env[i] != NULL; i++) {
        printf("%s\n", env[i]);
    }
}
```

就像使用 `environ` 一样不够通用，甚至更不具备可移植性。但设定目标总是好的。
```