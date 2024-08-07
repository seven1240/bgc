# 日期和时间功能

> “时间是一种幻觉。午餐时间则更是如此。” \
> --- Ford Prefect，《银河系漫游指南》

这并不太复杂，但一开始可能会有点吓人，因为不同类型之间的转换方式是不同的。

混入[UTC (Coordinated Universal Time)]和本地时间后，我们就会面临与时间和日期相关的所有_常规乐趣_™。

当然，永远不要忘记日期和时间的黄金法则：_永远不要试图编写自己的日期和时间功能。只使用库提供的功能。_

对于普通程序员来说，时间太复杂了，无法正确处理。确实，我们都应该感谢每个为任何日期和时间库工作过的人，所以记得在预算中考虑这一点。

## 快速术语和信息

这里有几个快速术语，以防您还不熟悉。

* [i[协调世界时]] **UTC (Coordinated Universal Time)**: 世界协调时间是一个被世界范围内^[至少在地球上。在"外面"可能有各种疯狂的系统...]普遍认可的绝对时间。地球上的每个人都认为现在使用的是UTC时间...尽管他们当地的时间可能不同。

* [i[格林尼治标准时间]] **GMT (Greenwich Mean Time)**: 格林尼治标准时间，实际上和UTC差不多^[好了，别杀了我！GMT在技术上是一个时区，而UTC是一个全球时间系统。另外，一些国家可能会根据夏令时调整GMT，而UTC则永远不会调整夏令时。]。你可能想说UTC，或者"世界时间"。如果您在特指GMT时区，那就说GMT。令人困惑的是，C语言的许多UTC函数早于UTC而且仍然提到格林尼治标准时间。当您看到这些时，请知道C指的是UTC。

* **本地时间**：计算机运行程序所在位置的当前时间。这被描述为与协调世界时 (UTC) 的偏移量。虽然世界上有许多时区，但大多数计算机使用本地时间或者协调世界时。

一般规则是，如果你要描述一个只发生一次的事件，比如日志条目、火箭发射，或者指针最终为你指向的时刻，请使用协调世界时。

另一方面，如果是发生在_每个时区相同时间_的事件，比如新年前夕或晚餐时间，请使用本地时间。

由于许多语言只擅长在协调世界时和本地时间之间进行转换，如果选择将日期存储在错误的格式中，可能会给自己带来很多麻烦。 (问我是如何知道的。)

## 日期类型

在 C 语言中，日期有两种^[诚然，实际上不止两种。]主要类型：`time_t` 和 `struct tm`。

规范实际上对它们没有多少定义：

* `time_t` 类型：一种真正能够保存时间的类型。因此，按照规范，这可以是浮点类型或整数类型。在 POSIX (类Unix系统) 中，它是一个整数类型。这保存了_日历时间_。你可以将其视为协调世界时。

* `struct tm` 类型：保存日历时间的各个组成部分。这是一个_分解时间_，即时间的各个组成部分，如小时、分钟、秒、日、月、年等。

在许多系统上，`time_t` 表示从 [flw[_Epoch_|Unix_time]] 开始的秒数。Epoch 从计算机的角度来看是时间的开始，通常是世界协调时间\（UTC）1970年1月1日。`time_t` 可以为负数，表示Epoch之前的时间。据我所知，Windows的行为与Unix相同。

[i[`time_t` 类型]>]
[i[`struct tm` 类型]<]

`struct tm` 类型包含哪些字段？

``` {.c}
struct tm {
    int tm_sec;    // 分钟后的秒数 -- [0, 60]
    int tm_min;    // 小时后的分钟数 -- [0, 59]
    int tm_hour;   // 自午夜以来的小时数 -- [0, 23]
    int tm_mday;   // 月份中的日子 -- [1, 31]
    int tm_mon;    // 自一月以来的月数 -- [0, 11]
    int tm_year;   // 距 1900 年的年数
    int tm_wday;   // 自周日以来的天数 -- [0, 6]
    int tm_yday;   // 自一月1日以来的天数 -- [0, 365]
    int tm_isdst;  // 夏令时标志
};
```

请注意，除了月份中的日期外，其他字段都是从零开始的。

重要的是要知道，你可以将任何值放入这些类型中。有函数可帮助获取当前时间，但这些类型持有一个时间，而不是确定的时间。

[i[`struct tm` 类型]>]

所以问题是：“如何初始化这些类型的数据，以及如何在它们之间进行转换？”

## 初始化和类型之间的转换

[i[`time()` 函数]<]

首先，你可以使用 `time()` 函数获取当前时间并将其存储在 `time_t` 中。

``` {.c}
time_t now;  // 用于保存当前时间的变量

now = time(NULL);  // 可以这样获取...

time(&now);        // ...或这样。与上一行相同。
```

太好了！现在你有一个变量可以获取当前时间。

[i[`time()` 函数]>]
[i[`ctime()` 函数]<]

有趣的是，打印`time_t`中的内容只有一种便携的方法，那就是很少使用的`ctime()`函数，它打印了本地时间的值：

``` {.c}
now = time(NULL);
printf("%s", ctime(&now));
```

这会返回一个非常特定格式的字符串，末尾包括一个换行符：

``` {.default}
Sun Feb 28 18:47:25 2021
```

[i[`ctime()`函数]>]

所以这有点不灵活。如果你想控制更多，你应该将`time_t`转换为`struct tm`。

### 将`time_t`转换为`struct tm`

[i[`time_t`类型-->转换为`struct tm`]<]

有两种很棒的方法可以进行这种转换：

[i[`localtime()`函数]<]

* `localtime()`：这个函数将`time_t`转换为本地时间下的`struct tm`。

[i[`gmtime()`函数]<]

* `gmtime()`：这个函数将`time_t`转换为UTC时间下的`struct tm`。（看到"ye olde GMT"这个名字中蕴含的格林尼治标准时间了吗？）

[i[`asctime()`函数]<]

让我们通过`asctime()`函数打印出当前的`struct tm`来查看现在是几点：

``` {.c}
printf("Local: %s", asctime(localtime(&now)));
printf("  UTC: %s", asctime(gmtime(&now)));
```

[i[`asctime()`函数]>]
[i[`localtime()`函数]>]
[i[`gmtime()`函数]>]

输出（我在太平洋标准时间区）：

``` {.default}
Local: Sun Feb 28 20:15:27 2021
  UTC: Mon Mar  1 04:15:27 2021
```

一旦你有了`time_t`的`struct tm`，就打开了许多可能性。你可以以各种方式打印时间，找出日期是星期几，等等。或者将其转换回`time_t`。

更多内容即将揭晓！

[i[`time_t`类型-->转换为`struct tm`]>]

### 将`struct tm`转换为`time_t`

[i[`struct tm`类型-->转换为`time_t`]<]
[i[`mktime()`函数]<]

如果你想反向操作，你可以使用 `mktime()` 函数来获取这些信息。

`mktime()` 函数会为你设置 `tm_wday` 和 `tm_yday` 的值，所以不必填写它们，因为它们会被覆盖。

此外，你可以将 `tm_isdst` 设置为 `-1` 让它为你做出判断。或者你也可以手动将其设置为真或假。

``` {.c}
// 不要诱人地在这些数字前加上前导零（除非你是希望它们被解释为八进制）！

struct tm some_time = {
    .tm_year=82,   // 自1900年以来的年数
    .tm_mon=3,     // 自1月的月数 -- [0, 11]
    .tm_mday=12,   // 月中的日期 -- [1, 31]
    .tm_hour=12,   // 自午夜以来的小时数 -- [0, 23]
    .tm_min=0,     // 小时后的分钟数 -- [0, 59]
    .tm_sec=4,     // 分钟后的秒数 -- [0, 60]
    .tm_isdst=-1,  // 夏令时标志
};

time_t some_time_epoch;

some_time_epoch = mktime(&some_time);

printf("%s", ctime(&some_time_epoch));
printf("Is DST: %d\n", some_time.tm_isdst);
```

输出:

``` {.default}
周一 4月 12 12:00:04 1982
夏令时: 0
```

当你手动加载类似的`struct tm`时，它应该是本地时间。`mktime()` 将会将本地时间转换为 `time_t` 日历时间。

奇怪的是，标准却没有提供一种载入UTC时间到 `struct tm` 并将其转换为 `time_t` 的方法。如果你想在类Unix系统中这样做，尝试使用非标准的 [i[`timegm()` Unix 函数]] `timegm()`。在Windows上，使用 [i[`_mkgmtime()` Windows 函数]] `_mkgmtime()`。 

## 格式化日期输出

我们已经看过一些将格式化日期输出打印到屏幕的方法。使用 `time_t` 我们可以使用 `ctime()`，而使用 `struct tm` 可以使用 `asctime()`。

``` {.c}
time_t now = time(NULL);
struct tm *local = localtime(&now);
struct tm *utc = gmtime(&now);

printf("本地时间: %s", ctime(&now));     // 使用 time_t 显示本地时间
printf("本地时间: %s", asctime(local));  // 使用 struct tm 显示本地时间
printf("世界标准时间: %s", asctime(utc));    // 使用 struct tm 显示 UTC

```

```cpp
    // %F: ISO 8601年月日
    // %T: ISO 8601时分秒
    // %z: ISO 8601时区偏移
    strftime(s, sizeof s, "ISO 8601: %FT%T%z", localtime(&now));
    puts(s);   // ISO 8601: 2021-02-28T22:29:00-0800
}
```

`strftime()`有许多用于日期打印的格式说明符，所以一定要在[fl[`strftime()` 参考页面|https://beej.us/guide/bgclr/html/split/time.html#man-strftime]]中查看。

[i[`strftime()`函数]>]

## 使用`timespec_get()`获得更高的分辨率

[i[`timespec_get()`函数]<]

您可以使用`timespec_get()`获得自纪元以来的秒数和纳秒数。

也许。

实现可能没有纳秒分辨率（即一秒十亿分之一），所以无法确定您会获得多少有效数字，但是试试看。

[i[`struct timespec` 类型]<]

`timespec_get()`接受两个参数。一个是指向`struct timespec`来保存时间信息的指针。另一个是`base`，规范允许您将其设置为`TIME_UTC`，表示您对自纪元以来的秒数感兴趣。（其他实现可能为`base`提供更多选项。）

结构本身有两个字段：

``` {.c}
struct timespec {
    time_t tv_sec;   // 秒
    long   tv_nsec;  // 纳秒（一秒十亿分之一）
};
```

这是一个示例，我们在其中获取时间并将其作为整数值和浮点值打印出来：

``` {.c}
struct timespec ts;

timespec_get(&ts, TIME_UTC);

printf("%ld 秒, %ld 纳秒\n", ts.tv_sec, ts.tv_nsec);

double float_time = ts.tv_sec + ts.tv_nsec/1000000000.0;
printf("%f 自纪元以来的秒数\n", float_time);
```

示例输出：

``` {.default}
1614581530 秒, 806325800 纳秒
1614581530.806326 自纪元以来的秒数
```

`struct timespec` 也出现在一些需要能够以该分辨率指定时间的线程函数中。

<!-- `struct timespec` 类型 -->
<!-- `timespec_get()` 函数 -->

## 时间差异

<!-- 日期和时间-->差异

关于获取两个 `time_t` 之间的差异的一个快速说明：由于规范没有规定该类型如何表示时间，因此您可能无法简单地减去两个 `time_t` 并获得任何合理的值。^[在 POSIX 上，当 `time_t` 明确为整数时，您将可以这样做。不幸的是，整个世界并非都在遵循 POSIX 标准。]

<!-- `difftime()` 函数 -->

幸运的是，您可以使用 `difftime()` 函数来计算两个日期之间的秒数差异。

在以下示例中，我们有两个相距一段时间的事件，并使用 `difftime()` 计算差异。

``` {.c .numberLines}
#include <stdio.h>
#include <time.h>

int main(void)
{
    struct tm time_a = {
        .tm_year=82,   // 自1900年以来的年份
        .tm_mon=3,     // 自Januar的月份 -- [0, 11]
        .tm_mday=12,   // 月份中的日期 -- [1, 31]
        .tm_hour=4,    // 自午夜以来的小时数 -- [0, 23]
        .tm_min=00,    // 小时后的分钟数 -- [0, 59]
        .tm_sec=04,    // 分钟后的秒数 -- [0, 60]
        .tm_isdst=-1,  // 夏令时标志
    };

    struct tm time_b = {
        .tm_year=120,  // 自1900年以来的年份
        .tm_mon=10,    // 自Januar的月份 -- [0, 11]
        .tm_mday=15,   // 月份中的日期 -- [1, 31]
        .tm_hour=16,   // 自午夜以来的小时数 -- [0, 23]
        .tm_min=27,    // 小时后的分钟数 -- [0, 59]
        .tm_sec=00,    // 分钟后的秒数 -- [0, 60]
        .tm_isdst=-1,  // 夏令时标志
    };

```
// 在事件之间有1217996816.000000秒（38.596783年）
// 使用 `difftime()` 获取时间差。即使在 POSIX 系统中可以直接相减，最好还是保持可移植性。
```