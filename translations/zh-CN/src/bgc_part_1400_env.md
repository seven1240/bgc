```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    /* Our code will go here */
    return 0;
}
```

这里有一个打印出所有命令行参数的程序。例如，如果我们将可执行文件命名为 `foo`，可以这样运行它：

``` {.zsh}
./foo i like turtles
```

然后我们将看到这个输出：

``` {.default}
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

有点奇怪，因为第零个参数是可执行文件的名称本身。但这只是需要习惯的事情。参数本身紧随其后。

来源：

[argc 参数]
[argv 参数]
[main() 函数-->命令行选项]

``` {.c .numberLines}
#include <stdio.h>

int main(int argc, char *argv[])
{
    for (int i = 0; i < argc; i++) {
        printf("arg %d: %s\n", i, argv[i]);
    }
}
```

哇！`main()` 函数的签名是什么情况呢？`argc` 和 `argv` 是什么[^由于它们只是普通的参数名，实际上你不必叫它们 `argc` 和 `argv`。但使用这些名称非常习惯，如果你有创意改变它们，其他 C 程序员确实会怀疑地审视你]（发音为 _arg-cee_ 和 _arg-vee_）？

先从简单的开始解释：`argc`。这是参数的数量，包括程序名称本身。如果你将所有参数想象成一个字符串数组，这也确实就是它们的类型，那么你可以将 `argc` 看作是这个数组的长度，这也正是它的含义。

因此，在循环中我们是在遍历所有的 `argv`，并逐一打印出它们，因此对于给定的输入：

``` {.zsh}
./foo i like turtles
```

我们将得到相应的输出：

``` {.default}
arg 0: ./foo
arg 1: i
arg 2: like
arg 3: turtles
```

理解了这一点，我们应该准备好开始我们的加法程序了。

我们的计划：

* 查看所有命令行参数（超过`argv[0]`，即程序名称）
* 将它们转换为整数
* 加入一个正在计算的总数
* 打印结果

让我们开始吧！

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;

    for (int i = 1; i < argc; i++) {  // 从1开始，第一个参数
        int value = atoi(argv[i]);    // 对于更好的错误处理，请使用strtol()

        total += value;
    }

    printf("%d\n", total);
}
```

[i[`main()` 函数-->命令行选项]>]
[i[`argc` 参数]>]

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

当然，如果你传入一个非整数，它可能会报错，但如何防范这种情况就留给读者作为一个练习。

### 最后一个 `argv` 是 `NULL`

有趣的小知识关于`argv`是，在最后一个字符串之后是一个指向`NULL`的指针。

也就是说：

``` {.c}
argv[argc] == NULL
```

总是成立的！

这看起来可能毫无意义，但事实证明在一些地方还是有用的；我们现在就来看其中之一。

### 另一个形式：`char **argv`

记住，当你调用一个函数时，C不会区分函数签名中的数组表示法和指针表示法。

也就是说，以下两种方式是等价的：

``` {.c}
void foo(char a[])
void foo(char *a)
```

现在，把`argv`当作一个字符串数组，也就是`char*`数组的数组，这种方式很方便，因此这个写法也是合理的：

``` {.c}
int main(int argc, char *argv[])
```

但由于等价性，你也可以这样写：

``` {.c}
int main(int argc, char **argv)
```

没错，这是一个指针的指针！如果这种理解更容易一些，可以把它看作是指向一个字符串的指针。但实际上，它是指向指向`char`的值的指针。

同样要记住这些是等价的：

``` {.c}
argv[i]
*(argv + i)
```

这意味着你可以对`argv`进行指针算术运算。

因此，另一种消耗命令行参数的方法可能是沿着`argv`数组逐步移动指针，直到我们遇到末尾的`NULL`为止。

让我们修改我们的加法器来实现这一点：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    int total = 0;
    
    // 一个巧妙的技巧，让编译器停止警告未使用的变量argc：
    (void)argc;

    for (char **p = argv + 1; *p != NULL; p++) {
        int value = atoi(*p);  // 为了更好的错误处理，可以使用strtol()

        total += value;
    }

    printf("%d\n", total);
}
```

我个人使用数组表示法来访问`argv`，但也看过这种风格在别处流传。

### 有趣事实

[i[`argc`参数]<]

关于`argc`和`argv`还有一些事情。

* 有些环境可能不会将`argv[0]`设置为程序名称。如果不可用，`argv[0]`将是一个空字符串。我从未见过这种情况发生。

* 规范实际上对于实现可以如何处理`argv`和这些值的来源相当宽松。但在我所在的所有系统中，处理方式都是一致的，就像我们在本节中讨论的那样。

* 您可以修改`argc`、`argv`或`argv`指向的任何字符串。 （只是不要使这些字符串比它们本来的长度更长！）

* 在某些类Unix系统上，修改字符串`argv[0]`会导致`ps`命令的输出发生变化^[\[ps\]，进程状态，是一个Unix命令，用来查看当前运行的进程]。

通常，如果你有一个名为`foo`的程序，用`./foo`运行它，你可能会在`ps`命令的输出中看到这样的内容：

``` {.default}
   4078 tty1     S      0:00 ./foo
```

但是如果你像这样修改`argv[0]`，要小心确保新的字符串`"Hi!  "`和旧的字符串`"./foo"`长度相同：

``` {.c}
strcpy(argv[0], "Hi!  ");
```

然后在程序`./foo`仍在执行时运行`ps`命令，我们将看到下面这样的输出：

``` {.default}
   4079 tty1     S      0:00 Hi!  
```

这种行为不在规范内，而且高度依赖于系统。

## 退出状态 {#exit-status}

你有没有注意到`main()`函数的函数签名是`int`类型的返回值？这是怎么回事？这与一个叫做“退出状态”的东西有关，它是一个整数，可以返回给启动你的程序的程序，让它知道事情进展如何。

现在，在C中，程序有多种退出的方式，包括从`main()`中`return`，或调用`exit()`的变体之一。

所有这些方法都接受一个`int`作为参数。

顺便说一下：你是否注意到在我的所有示例中，即使`main()`应该返回一个`int`，但我实际上并没有`return`任何东西？在任何其他函数中，这将是非法的，但在C中有一个特殊情况：如果执行到了`main()`的结尾没有找到`return`，它会自动执行`return 0`。

但是`0`代表什么？我们可以放什么其他数字？它们是如何使用的？

规范在这个问题上既清晰又模糊，这是很常见的。清晰是因为它详细说明了可以做什么，但模糊是因为它并没有特别限制它。

唯有继续前行，摸索弄明白！

让我们以《盗梦空间》为例：当你运行程序时，**实际上是从另一个程序中运行它的**。

通常，这个另一个程序是某种类型的[shell|Shell_(computing)]，本身并没有太多功能，除了启动其他程序。

但这是一个多阶段的过程，特别在命令行shell中更是明显：

1. Shell启动你的程序
2. Shell通常会进入休眠状态（对于命令行shell）
3. 你的程序运行
4. 你的程序终止
5. Shell唤醒并等待下一个命令

现在，在步骤4和5之间会有一小段通信：程序可以返回一个状态值，供shell进行查询。通常，这个值用于指示你的程序是成功还是失败，并且如果失败，是什么类型的失败。

这个值就是我们从`main()`函数中返回的。那就是状态。

C语言规范允许两种不同的状态值，它们在`<stdlib.h>`中定义了宏名称：

[`EXIT_SUCCESS`宏]
[`EXIT_FAILURE`宏]

|状态|描述|
|-|-|
|`EXIT_SUCCESS`或`0`|程序成功终止。|
|`EXIT_FAILURE`|程序以错误终止。|

让我们编写一个简短的程序，从命令行中传入两个数字并将它们相乘。我们要求你准确指定两个值。如果你没有这样做，我们将打印一个错误消息，并以错误状态退出。

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
    if (argc != 3) {
        printf("用法: mult x y\n");
        return EXIT_FAILURE;   // 向shell指示它没有成功工作
    }

    printf("%d\n", atoi(argv[1]) * atoi(argv[2]));

    return 0;  // 和EXIT_SUCCESS相同，一切正常。
}
```

```c
[i[`EXIT_SUCCESS` 宏]>]
[i[`EXIT_FAILURE` 宏]>]

现在如果我们尝试运行这个程序，在输入确切的命令行参数数量之前会得到预期的效果：

``` {.zsh}
$ ./mult
usage: mult x y

$ ./mult 3 4 5
usage: mult x y

$ ./mult 3 4
12
```

[i[退出状态-->从shell中获取]<]

但这并没有真正显示我们返回的退出状态，是吧？不过我们可以让shell将其打印出来。假设您正在运行Bash或其他POSIX shell，您可以使用 `echo $?` 命令来查看它^[在Windows的 `cmd.exe` 中，输入 `echo %errorlevel%`。在PowerShell中，输入 `$LastExitCode`。]。

让我们试一下：

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

[i[退出状态-->从shell中获取]>]

有趣！我们看到在我的系统上，[i[`EXIT_FAILURE` 宏]]`EXIT_FAILURE` 是 `1`。规范没有明确说明，所以它可能是任何数字。不过试一下；在您的系统上，它可能也是 `1`。

### 其他退出状态值

状态 `0` 绝对表示成功，但其他整数呢，甚至是负数呢？

在这里，我们超出了C规范的范畴，进入了Unix领域。一般来说，`0` 表示成功，正的非零数字表示失败。因此您只能有一种成功状态，但可以有多种失败状态。Bash规定退出代码应在0到255之间，尽管有一些代码是预留的。

简而言之，如果您想在Unix环境中指示不同的错误退出状态，您可以从 `1` 开始，然后逐渐增加。

在Linux上，如果您尝试使用0-255范围之外的代码，系统会将该代码与 `0xff` 位运算与，有效地将其限制在该范围内。

您可以编写脚本，然后使用这些状态代码来决定接下来该执行什么操作。
```

```c
int main(void)
{
    char *val = getenv("FROTZ");  // 尝试获取值

    // 检查确保它存在
    if (val == NULL) {
        printf("无法找到 FROTZ 环境变量\n");
        return EXIT_FAILURE;
    }

    printf("数值: %s\n", val);
}
```

如果我直接运行这个程序，会得到这个输出：

``` {.zsh}
$ ./foo
找不到 FROTZ 环境变量
```

这很合理，因为我还没有设置它。

在 bash 中，可以用以下方法设置它为某个值。^[在 Windows CMD.EXE 中，使用 `set
FROTZ=value`。在 PowerShell 中，使用 `$Env:FROTZ=value`]：

``` {.zsh}
$ export FROTZ="C is awesome!"
```

然后如果我运行它，会得到：

``` {.zsh}
$ ./foo
Value: C is awesome!
```

通过这种方式，你可以在环境变量中设置数据，并在 C 代码中获取它，根据需要调整行为。

### 设置环境变量

虽然这不是标准做法，但很多系统提供了设置环境变量的方法。

如果在类 Unix 系统上，查看[i[`putenv()` 函数]]`putenv()`、[i[`setenv()`]`setenv()` 和[i[`unsetenv()` 函数]]`unsetenv()` 的文档。在 Windows 上，参考[i[`_putenv()` 函数]]`_putenv()`。

### 类 Unix 系统的替代环境变量

如果你在类 Unix 系统上，很可能有另外几种获取环境变量的方法。请注意，虽然规范指出这是一个常见扩展，但它并不真正属于 C 标准。不过，这是 POSIX 标准的一部分。

[i[`environ` 变量]<]

其中一种就是一个名为 `environ` 的变量，必须这样声明：

``` {.c}
extern char **environ;
```

这是一个以 `NULL` 指针结尾的字符串数组。

在使用之前，你应该自行声明它，或者可能在非标准的 `<unistd.h>` 头文件中找到它。

每个字符串的形式都是 `"key=value"`，所以如果想要获取键和值，就必须自行拆分和解析。

以下是一个循环遍历并打印环境变量的示例：

``` {.c .numberLines}
#include <stdio.h>
```

```c
extern char **environ;  // 必须是 extern 并命名为 "environ"

int main(void)
{
    for (char **p = environ; *p != NULL; p++) {
        printf("%s\n", *p);
    }

    // Or you could do this:
    for (int i = 0; environ[i] != NULL; i++) {
        printf("%s\n", environ[i]);
    }
}
```

对于输出的一堆看起来像这样的内容：

``` {.default}
SHELL=/bin/bash
COLORTERM=truecolor
TERM_PROGRAM_VERSION=1.53.2
LOGNAME=beej
HOME=/home/beej
... etc ...
```

如果可能的话，请使用 `getenv()`，因为它更可移植。但如果必须遍历环境变量，请使用 `environ` 可能是比较好的方式。

通过将环境变量作为 `main()` 的一个参数的非标准方法也可以获得环境变量。它的工作方式基本相同，但你避免需要添加你的 `extern` `environ` 变量。根据我所知，甚至 POSIX 规范都不支持这种方式，但在 Unix 领域很常见。

``` {.c .numberLines}
#include <stdio.h>

int main(int argc, char **argv, char **env)  // <-- env!
{
    (void)argc; (void)argv;  // 抑制未使用警告

    for (char **p = env; *p != NULL; p++) {
        printf("%s\n", *p);
    }

    // Or you could do this:
    for (int i = 0; env[i] != NULL; i++) {
        printf("%s\n", env[i]);
    }
}
```

就像使用 `environ` 一样，但更不具有可移植性。制定目标是很好的。