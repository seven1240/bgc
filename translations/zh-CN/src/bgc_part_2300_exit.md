```c
  这里是`main()`的`return`规则：
```

* 你可以用`return`语句从`main()`中返回退出状态。`main()`是唯一拥有这种特殊行为的函数。在其他函数中使用`return`只是从该函数返回给调用者。
* 如果你没有明确使用`return`而是直接从`main()`结尾跳出，那就跟返回`0`或`EXIT_SUCCESS`一样。
* `exit()`

这个也出现过几次。如果你在程序中的任何地方调用`exit()`，程序将在那一点退出。

传递给`exit()`的参数是退出状态。

### 使用`atexit()`设置退出处理程序

你可以注册在程序退出时调用的函数，无论是通过从`main()`返回还是调用`exit()`函数。

使用带有处理程序函数名称的`atexit()`调用可以完成。你可以注册多个退出处理程序，它们将按照注册的相反顺序调用。

以下是一个示例：

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

void on_exit_1(void)
{
    printf("调用退出处理程序1！\n");
}

void on_exit_2(void)
{
    printf("调用退出处理程序2！\n");
}

int main(void)
{
    atexit(on_exit_1);
    atexit(on_exit_2);
    
    printf("即将退出...\n");
}
```

输出结果是：

``` {.default}
即将退出...
调用退出处理程序2！
调用退出处理程序1！
```

## 使用`quick_exit()`更快退出

这类似于正常退出，不同之处在于：

* 打开的文件可能不会被刷新。
* 临时文件可能不会被删除。
* `atexit()`处理程序不会被调用。

但是有一种方法可以注册退出处理程序：类似于调用`atexit()`的方式，调用`at_quick_exit()`。

``` {.c .numberLines}
#include <stdio.h>
#include <stdlib.h>

void on_quick_exit_1(void)
{
    printf("Quick exit handler 1 called!\n");
}

void on_quick_exit_2(void)
{
    printf("Quick exit handler 2 called!\n");
}

void on_exit(void)
{
    printf("Normal exit--I won't be called!\n");
}

int main(void)
{
    at_quick_exit(on_quick_exit_1);
    at_quick_exit(on_quick_exit_2);

    atexit(on_exit);  // This won't be called

    printf("About to quick exit...\n");

    quick_exit(0);
}
```

这将输出：

``` {.default}
About to quick exit...
Quick exit handler 2 called!
Quick exit handler 1 called!
```

它的工作方式类似于 `exit()`/`atexit()`，不同之处在于文件刷新和清理可能不会执行。

[了解更多[`quick_exit()` 函数]>]

## 以原子方式摧毁它：`_Exit()`

[了解更多[`_Exit()` 函数]<]

调用 `_Exit()` 会立即退出，毫无疑问。不会执行退出时回调函数。文件不会被刷新。临时文件不会被删除。

如果你必须立即退出，就使用这个。

## 有时需要退出：`assert()`

`assert()` 语句用于坚持某些条件为真，否则程序将退出。

开发者经常使用 assert 来捕获绝不应发生的错误类型。

``` {.c}
#define PI 3.14159

assert(PI > 3);   // 当然，是正确的，继续执行
```

与：

``` {.c}
goats -= 100;

assert(goats >= 0);  // 不能有负的羊
```

在这种情况下，如果尝试运行并且 `goats` 减少到小于 `0`，则会发生如下情况：

``` {.default}
goat_counter: goat_counter.c:8: main: Assertion `goats >= 0' failed.
Aborted
```

然后我被返回到命令行。

这并不是很用户友好，因此仅用于用户永远不会看到的事情。通常人们会[编写自己的assert宏，以便更容易地关闭assert](#my-assert)。

```c
// Abnormal Exit: `_Exit()`

## 异常退出：`abort()`

// You can use this if something has gone horribly wrong and you want to
// indicate as much to the outside environment. This also won't necessarily
// clean up any open files, etc.

// I've rarely seen this used.

// Some foreshadowing about _signals_: this actually works by raising a
// `SIGABRT` signal which will end the process.

// What happens after that is up to the system, but on Unix-likes, it was
// common to dump core as the program terminated.

// [`abort()`函数]
// 退出
```