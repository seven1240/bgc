<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 日期和时间功能

> "时间是个幻觉。午餐时间更是如此。"\
> ---福特·完美者，《银河系搭车指南》

[i[日期和时间]<]

这并不算太复杂，但刚开始可能会有点吓人，因为有不同类型可用，以及它们之间的转换方式。

加入格林尼治标准时间（UTC）和本地时间，我们就会陷入与时钟和日期相关的所有「常规乐趣™」。

当然别忘了日期和时间的黄金法则：_永远不要尝试编写自己的日期和时间功能。只使用库提供的功能。_

时间对于普通程序员来说太复杂了。说真的，我们都应该对任何日期和时间库的贡献者致以崇高的敬意，所以把它们列入你的预算吧。

## 快速术语和信息

如果你还没弄清楚，这里简单解释一下几个术语。

* [i[协调世界时]] **UTC**：协调世界时是一个在全球范围内得到普遍认可的绝对时间。地球上的每个人都认为现在是 UTC 时间... 即使他们在不同的本地时间。

* [i[格林尼治标准时间]] **GMT**：格林尼治标准时间，与 UTC 实际上相同。你可能会说 UTC，或者"世界时间"。如果你在特指 GMT 时区，说 GMT。令人困惑的是，C 语言的许多 UTC 函数早于协调世界时而仍然参考格林尼治标准时间。当你看到这一点时，请知道 C 指的是 UTC。

```c
* [i[Local time]] **Local time**: 程序运行的计算机所在地的时间。这个时间是相对于世界标准时间（UTC）的偏移量。尽管世界上有许多时区，但大多数计算机使用本地时间或UTC。

[i[Universal Coordinated Time]<]

一般规则是，如果你在描述一个只发生一次的事件，比如日志记录、火箭发射或者当指针终于指向你期望的位置时，请使用UTC。

[i[Local time]<]

另一方面，如果描述的是每个时区同时发生的事件，比如新年前夕或晚餐时间，请使用本地时间。

由于很多编程语言只擅长在UTC和本地时间之间转换，如果选择以错误的形式存储日期，可能会给自己带来很多麻烦。（问我怎么知道的。）

[i[Local time]>]
[i[Universal Coordinated Time]>]

## 日期类型

在C语言中，有两种主要的日期类型：`time_t`和`struct tm`。

关于这两种类型，规范中并没有详细说明：

* [i[`time_t` 类型]] `time_t`：一个真正能够保存时间的类型。因此，按照规范，这可以是浮点类型或整数类型。在POSIX（类Unix系统中），它是一个整数。它表示的是 _日历时间_，你可以将其理解为UTC时间。

* [i[`struct tm` 类型]] `struct tm`：保存日历时间的各个组成部分。这是一个 _拆分时间_，即时间的各个组成部分，如小时、分钟、秒、日、月、年等。

[i[`time_t` 类型]<]
```

在许多系统中，`time_t` 表示自 [flw[_Epoch_|Unix_time]] 以来的秒数。Epoch 从计算机的角度来看是时间的起点，通常是协调世界时 1970 年 1 月 1 日。`time_t` 可以变为负数以表示 Epoch 之前的时间。据我所知，Windows 的行为和 Unix 相同。

[i[`time_t` 类型]>]
[i[`struct tm` 类型]<]

那`struct tm` 中都有些什么呢？包括以下字段：

``` {.c}
struct tm {
    int tm_sec;    // 分钟之后的秒数 -- [0, 60]
    int tm_min;    // 小时之后的分钟数 -- [0, 59]
    int tm_hour;   // 从午夜开始的小时数 -- [0, 23]
    int tm_mday;   // 月份中的日期 -- [1, 31]
    int tm_mon;    // 从一月起的月份数 -- [0, 11]
    int tm_year;   // 1900 年以来的年数
    int tm_wday;   // 从周日开始的天数 -- [0, 6]
    int tm_yday;   // 从一月一日开始的天数 -- [0, 365]
    int tm_isdst;  // 夏令时标志
};
```

请注意，除了月份中的日期外，其他都是以零为基础。

重要的是要知道，你可以在这些类型中放入任何你想要的值。有助于获取当前时间的函数，但这些类型保存的是_一个_时间，而不是_特定的_时间。

[i[`struct tm` 类型]>]

那么问题来了： “如何初始化这些类型的数据，以及如何在它们之间转换?”

## 初始化和类型之间的转换

[i[`time()` 函数]<]

首先，你可以使用 `time()` 函数获取当前时间并将其存储在 `time_t` 中。

``` {.c}
time_t now;  // 用于保存当前时间的变量

now = time(NULL);  // 可以这样获取它...

time(&now);        // ...或者这样。与上一行相同。
```

太棒了！你有了一个可以获取当前时间的变量。

[i[`time()` 函数]>]
[i[`ctime()` 函数]<]

有趣的是，只有一种便携的方式可以打印出`time_t`中的内容，那就是很少使用的`ctime()`函数，它会以本地时间打印出该值：

``` {.c}
now = time(NULL);
printf("%s", ctime(&now));
```

这将返回一个带有特定格式的字符串，末尾包括一个换行符：

``` {.default}
Sun Feb 28 18:47:25 2021
```

[i[`ctime()`函数]>]

这有点不太灵活。如果你想要更多控制，你应该将那个`time_t`转换为`struct tm`。

### 将`time_t`转换为`struct tm`

[i[`time_t`类型-->转换为`struct tm`]<]

有两种了不起的方法可以进行这种转换：

[i[`localtime()`函数，本地时间]<]

* `localtime()`: 这个函数会将一个`time_t`转换为本地时间中的`struct tm`。

[i[`gmtime()`函数，UTC时间]<]

* `gmtime()`: 这个函数会将一个`time_t`转换为UTC时间中的`struct tm`。（注意`GMT`隐匿在该函数名称中了吗？）

[i[`asctime()`函数]<]

让我们通过使用`asctime()`函数打印出当前时间的`struct tm`：

``` {.c}
printf("Local: %s", asctime(localtime(&now)));
printf("  UTC: %s", asctime(gmtime(&now)));
```

[i[`asctime()`函数]>]
[i[`localtime()`函数]>]
[i[`gmtime()`函数]>]

输出结果（我所处的是太平洋标准时间时区）：

``` {.default}
Local: Sun Feb 28 20:15:27 2021
  UTC: Mon Mar  1 04:15:27 2021
```

一旦你将`time_t`转换为了`struct tm`，就会打开许多可能性。你可以以各种方式打印时间，计算日期是星期几，等等。或者将其转换回`time_t`。

更多内容即将揭晓！

[i[`time_t`类型-->转换为`struct tm`]>]

### 将`struct tm`转换为`time_t`

[i[`struct tm`类型-->转换为`time_t`]<]
[i[`mktime()`函数]<]

如果您想要反向操作的话，可以使用 `mktime()` 来获取相关信息。

`mktime()` 会为 `tm_wday` 和 `tm_yday` 设置值，所以不用费心填写它们，因为它们会被覆盖掉。

此外，您可以将 `tm_isdst` 设置为 `-1` 让系统来决定。或者您也可以手动设置为 true 或者 false。

``` {.c}
// 不要诱使自己给这些数字加前导零（除非您打算让它们成为八进制）！

struct tm some_time = {
    .tm_year=82,   // 自1900年以来的年数
    .tm_mon=3,     // 从一月开始计算的月份 -- [0, 11]
    .tm_mday=12,   // 月份中的日期 -- [1, 31]
    .tm_hour=12,   // 当天的小时数 -- [0, 23]
    .tm_min=0,     // 小时中的分钟数 -- [0, 59]
    .tm_sec=4,     // 分钟中的秒数 -- [0, 60]
    .tm_isdst=-1,  // 夏令时标志
};

time_t some_time_epoch;

some_time_epoch = mktime(&some_time);

printf("%s", ctime(&some_time_epoch));
printf("Is DST: %d\n", some_time.tm_isdst);
```

输出:

``` {.default}
Mon Apr 12 12:00:04 1982
Is DST: 0
```

当您手动加载一个类似这样的 `struct tm` 结构时，它应该是本地时间。`mktime()` 将把该本地时间转换为 `time_t` 日历时间。

[更多关于 [`mktime()` 函数]>]

然而奇怪的是，标准并没有提供一种加载 UTC 时间到 `struct tm` 结构然后将其转换为 `time_t` 的方法。如果您想在类 Unix 系统中这样做，可以尝试使用非标准的 [i[`timegm()` Unix
函数]] `timegm()`。在 Windows 上，[i[`_mkgmtime()` Windows 函数]] `_mkgmtime()`。

[关于 [`struct tm` 类型-->转换为 `time_t`]>]

## 格式化日期输出

我们已经看到了几种将格式化日期输出到屏幕上的方式。使用 `time_t` 我们可以使用 `ctime()`，而使用 `struct tm` 我们可以使用 `asctime()`。

``` {.c}
time_t now = time(NULL);
struct tm *local = localtime(&now);
struct tm *utc = gmtime(&now);

printf("本地时间: %s", ctime(&now));     // 使用time_t打印本地时间
printf("本地时间: %s", asctime(local));  // 使用struct tm打印本地时间
printf("世界时区: %s", asctime(utc));    // 使用struct tm打印世界时区时间
```

但是如果我告诉你，亲爱的读者，有一种方法可以更好地控制日期的打印方式呢？

[i[`strftime()` 函数]<]

当然，我们可以从`struct tm`中提取单独的字段，但是有一个很棒的函数叫做`strftime()`可以为你完成大部分工作。就像`printf()`一样，但用于日期！

让我们看一些示例。在每个示例中，我们传入一个目标缓冲区、要写入的最大字符数，以及一个格式字符串（与`printf()`的风格相似，但并非完全相同），告诉`strftime()`要打印`struct tm`的哪些组件以及如何打印。

您也可以在格式字符串中添加其他常量字符以包括在输出中，就像使用`printf()`一样。

在这种情况下，我们从`localtime()`得到一个`struct tm`，但任何来源都可以正常工作。

``` {.c .numberLines}
#include <stdio.h>
#include <time.h>

int main(void)
{
    char s[128];
    time_t now = time(NULL);

    // %c: 根据当前区域设置打印日期
    strftime(s, sizeof s, "%c", localtime(&now));
    puts(s);   // 星期日 二月 28 22:29:00 2021

    // %A: 星期几的全称
    // %B: 月份的全称
    // %d: 月份中的日期
    strftime(s, sizeof s, "%A, %B %d", localtime(&now));
    puts(s);   // 星期日, 二月 28

    // %I: 小时（12小时制）
    // %M: 分钟
    // %S: 秒
    // %p: AM 或 PM
    strftime(s, sizeof s, "现在是 %I:%M:%S %p", localtime(&now));
    puts(s);   // 现在是 10:29:00 PM
```

```c
    // %F: ISO 8601 yyyy-mm-dd
    // %T: ISO 8601 hh:mm:ss
    // %z: ISO 8601 time zone offset
    strftime(s, sizeof s, "ISO 8601: %FT%T%z", localtime(&now));
    puts(s);   // ISO 8601: 2021-02-28T22:29:00-0800
}
```

在`strftime()`中有非常多的日期打印格式指示符，所以一定要在[fl[`strftime()`参考页面|https://beej.us/guide/bgclr/html/split/time.html#man-strftime]]中进行查看。

[i[`strftime()` 函数]>]

## 使用`timespec_get()`获得更精细的时间分辨率

[i[`timespec_get()` 函数]<]

您可以使用`timespec_get()`获取自纪元以来的秒数和纳秒数。

也许。

实现可能没有纳秒分辨率（即一秒的十亿分之一）所以谁知道您会获得多少有效数字，但是试一试看看。

[i[`struct timespec` 类型]<]

`timespec_get()`需要两个参数。一个是指向`struct timespec`的指针，用于保存时间信息。另一个是`base`，规格允许您将其设置为`TIME_UTC`表示您对自纪元以来的秒数感兴趣。（其他实现可能会为`base`提供更多选项。）

该结构本身有两个字段：

``` {.c}
struct timespec {
    time_t tv_sec;   // 秒
    long   tv_nsec;  // 纳秒（一秒的十亿分之一）
};
```

这是一个示例，我们获取时间并将其分别打印为整数值和浮点值：

``` {.c}
struct timespec ts;

timespec_get(&ts, TIME_UTC);

printf("%ld 秒，%ld 纳秒\n", ts.tv_sec, ts.tv_nsec);

double float_time = ts.tv_sec + ts.tv_nsec/1000000000.0;
printf("%f 秒自纪元以来\n", float_time);
```

示例输出：

``` {.default}
1614581530 秒, 806325800 纳秒
1614581530.806326 秒自纪元以来
```

`struct timespec` 也在一些需要以该分辨率指定时间的线程函数中出现。

[i[`struct timespec` 类型]>]
[i[`timespec_get()` 函数]>]

## 时间差异

[i[日期和时间-->差异]<]

关于获取两个`time_t`之间的差值，有一个要注意的地方：由于规范并未规定该类型如何表示时间，因此可能无法简单地减去两个`time_t`并得到有意义的结果^[在 POSIX 中可以，那里的 `time_t` 明确是整数。不幸的是整个世界并非都是 POSIX，所以我们只能这样了。]。

[i[`difftime()` 函数]<]

幸运的是，您可以使用 `difftime()` 来计算两个日期之间的秒数差。

在以下示例中，我们有两个相隔一段时间的事件，并使用 `difftime()` 来计算差值。

``` {.c .numberLines}
#include <stdio.h>
#include <time.h>

int main(void)
{
    struct tm time_a = {
        .tm_year=82,   // 1900年以来的年数
        .tm_mon=3,     // 自一月起的月数 -- [0, 11]
        .tm_mday=12,   // 一个月中的日数 -- [1, 31]
        .tm_hour=4,    // 当日格林尼治时间的小时数 -- [0, 23]
        .tm_min=00,    // 小时中的分钟数 -- [0, 59]
        .tm_sec=04,    // 分钟中的秒数 -- [0, 60]
        .tm_isdst=-1,  // 夏令时标志
    };

    struct tm time_b = {
        .tm_year=120,  // 1900年以来的年数
        .tm_mon=10,    // 自一月起的月数 -- [0, 11]
        .tm_mday=15,   // 一个月中的日数 -- [1, 31]
        .tm_hour=16,   // 当日格林尼治时间的小时数 -- [0, 23]
        .tm_min=27,    // 小时中的分钟数 -- [0, 59]
        .tm_sec=00,    // 分钟中的秒数 -- [0, 60]
        .tm_isdst=-1,  // 夏令时标志
    };
```

``` {.default}
1217996816.000000 秒（38.596783 年）之间的事件之间的事件
```

到此为止！记得使用`difftime()`来计算时间差。虽然在POSIX系统上可以直接相减，但最好保持可移植性。

[关于`difftime()`函数]
[日期和时间-->差异]
[日期和时间]
```