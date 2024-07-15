```c
    // 目前进行第一阶段：BGC 和 man 手册
    // 第二阶段：bgclr_part_0150

    // C23 内容

    // 标准库（新增函数）

    // 在 `<string.h>` 中添加 `memset_explicit()` 函数来擦除敏感数据，无论是否进行优化，内存存储必须始终执行。

    // 在 `<string.h>` 中添加 `memccpy()` 函数来高效地连接字符串 - 类似于 POSIX 和 SVID C 扩展。

    // 在 `<string.h>` 中添加 strdup() 和 strndup() 函数来分配字符串的副本 - 类似于 POSIX 和 SVID C 扩展。

    // 在 `<stdlib.h>` 中添加 memalignment() 函数来确定指针的字节对齐方式。
```

```c
// 在新的标题 <stdbit.h> 中添加位操作的实用函数/宏/类型，以检查许多整数类型。所有函数都以 stdc_ 开头，以最小化与旧代码和第三方库的冲突。

// 在下面的代码中，用 uc、us、ui、ul、ull 替换 * 以得到五个函数名，或留空得到一个类型通用的宏。

// 添加 stdc_count_ones*() 和 stdc_count_zeros*() 来统计值中 1 或 0 位的数量。

// 添加 stdc_leading_ones*() 和 stdc_leading_zeros*() 来计算值中领先的 1 或 0 位的数量。

// 添加 stdc_trailing_ones*() 和 stdc_trailing_zeros*() 来统计值中尾随的 1 或 0 位的数量。

// 添加 stdc_first_leading_one*() 和 stdc_first_leading_zero*() 来找到值中第一个领先的位为 1 或 0。

// 添加 stdc_first_trailing_one*() 和 stdc_first_trailing_zero*() 来找到值中第一个尾随的位为 1 或 0。

// 添加 stdc_has_single_bit*() 来判断值是否是 2 的幂（当且仅当存在唯一的 1 位时返回 true）。

// 添加 stdc_bit_floor*() 来确定不大于给定值的最大整数幂次 2。

// 添加 stdc_bit_ceil*() 来确定不小于给定值的最小整数幂次 2。

// 添加 stdc_bit_width*() 来确定表示一个值所需的位数。

// 在 <time.h> 中添加 timegm() 函数，将时间结构转换成日历时间值，与 glibc 和 musl 库中的函数类似。

// 标准库（已有函数）

// 为 printf() 函数族添加 %b 二进制转换格式说明符，用 0b 在非零值前面表示，类似于 %x 的工作方式。以前没有使用 %B 作为自己扩展的实现被鼓励实现，并用 0B 在非零值前面表示，类似于 %X 的工作方式。
```

```c
// 为scanf()函数系列增加%b二进制转换说明符。
// 为strtol()和wcstol()函数系列增加0b和0B二进制转换支持。
// 如果这些函数的参数为const限定对象，则让函数bsearch()、bsearch_s()、memchr()、strchr()、strpbrk()、strrchr()、strstr()及其宽字符对应函数wmemchr()、wcschr()、wcspbrk()、wcsrchr()、wcsstr()返回一个带有const限定的对象。

## 预处理器

// 增加#ifdef和#ifndef指令实际上等同于#if defined和#if !defined。这两个指令被添加到C++23标准和GCC 12编译器中。
// 增加`#embed`指令用于包含二进制资源。

书签

// 增加`#warning`指令用于诊断。
// 添加`__has_include`允许通过预处理器指令检查头文件的可用性。
// 添加`__has_c_attribute`允许通过预处理器指令检查属性的可用性（请参阅“C++兼容性”组中的新属性特性）。
// 添加`__VA_OPT__`函数宏，用于可变参数宏，仅在包含宏的容器中传递了可变参数时扩展到其参数。

## 类型

// 添加nullptr_t类型。
// 为比特精准整数添加_BitInt(N)和unsigned _BitInt(N)类型。添加BITINT_MAXWIDTH宏表示最大比特宽度。添加ckd_add()、ckd_sub()、ckd_mul()宏，用于检查整数运算。
// 可变修改类型（但不包括在堆栈上分配的自动变量的VLA）成为一项强制性功能。
// 标准化typeof(...)运算符。
// auto关键词的含义更改为在进行类型推断的同时保留其旧含义，即如果与类型一起使用，则作为存储类说明符。

## 常量
```

```c
// 添加`nullptr_t`类型的`nullptr`常量。[21]
// 为`_BitInt(N)`和`unsigned _BitInt(N)`类型添加`wb`和`uwb`整数字面值后缀，[28]比如`6uwb`会得到一个无符号`_BitInt(3)`，而`-6wb`会得到一个有符号`_BitInt(4)`，其中有三个数值位和一个符号位。
// 添加`0b`和`0B`二进制面值常量前缀，[29]比如`0b10101010`（等同于`0xAA`）。
// 为字面常量添加`'`数字分隔符，[30]比如`0xFE'DC'BA'98`（等同于`0xFEDCBA98`）、`299'792'458`（等同于`299792458`）、`1.414'213'562`（等同于`1.414213562`）。
// 添加指定枚举底层类型的能力。[31]
// 允许枚举没有固定底层类型的情况下存储不能用`int`表示的值。[32]

## 关键词

// 添加`true`和`false`关键词。[33]
// 添加`alignas`、`alignof`、`bool`、`static_assert`、`thread_local`关键词。原先定义的关键词变成另一种拼写形式：`_Alignas`、`_Alignof`、`_Bool`、`_Static_assert`、`_Thread_local`。[34]
// 添加`_BitInt`关键词（见"types"组）
// 添加`typeof`和`typeof_unqual`关键词（见“types”组）
// 添加`nullptr`关键词（见“constants”组）
// 添加`constexpr`关键词（见“other”组）
// 为（可选）十进制浮点运算添加`_Decimal32`、`_Decimal64`、`_Decimal128`关键词（见“other”组）

## C++兼容性
```

* 使用双方括号`[[]]`添加C++11风格的属性语法。添加属性`[[deprecated]]`, `[[fallthrough]]`, `[[maybe_unused]]`, `[[nodiscard]]`, `[[noreturn]]`以保持与C++11的兼容性，然后弃用在C11中引入的`_Noreturn`, `noreturn`, 头文件 `<stdnoreturn.h>` 特性。允许重复属性以保持与C++23的兼容性。所有标准属性也可以用双下划线包围（例如 `[[__deprecated__]]` 相当于 `[[deprecated]]`）。
* 为了与C++17兼容，为字符字面量添加`u8`前缀，表示UTF-8编码。
* 为了与C++23兼容，添加`#elifdef` 和 `#elifndef` 预处理指令（见“预处理器”组）。
* 为了与C++17兼容，添加单参数的 `_Static_assert`。

```c
* 支持 ISO/IEC 60559:2020 标准，即 IEEE 754 浮点运算标准的当前版本，包括扩展二进制浮点运算和（可选的）十进制浮点运算。[46][47]
* 标签可以出现在声明之前或复合语句的末尾。[48]
* 函数定义中可以使用无名称参数。[49]
* 更好地支持使用 const 修饰数组。[50]
* 使用 {} 进行零初始化（包括可变长度数组的初始化）。[51]
* 为对象引入 constexpr 修饰符，但不支持函数，这与 C++ 的等效处理方式不同。[52]
* 可变参数函数不再需要省略号前的命名参数，va_start 宏也不再需要第二个参数，如果有的话也不评估第一个参数之后的任何参数。[53]
* 添加 char8_t 类型用于存储 UTF-8 编码数据，并将 u8 字符常量和字符串字面值的类型更改为 char8_t。此外，引入函数 mbrtoc8() 和 c8rtomb() 用于将窄多字节字符转换为 UTF-8 编码和从 UTF-8 转换为窄多字节字符表示的单一代码点。[54]
* 允许存储类别修饰符出现在复合字面量定义中。[55]

## 其他

* 更新 C 版本参考
* stdlib.h 中的 strfrom 函数
* 验证其他库函数
```