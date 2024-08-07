当前阶段 1: BGC 和 man 手册
阶段 2: bgclr_part_0150

# C23 内容

## 标准库（新函数）

* 在 `<string.h>` 中添加 `memset_explicit()` 函数以擦除敏感数据，无论如何都必须执行内存存储，不受优化影响。

* 在 `<string.h>` 中添加 `memccpy()` 函数以高效地连接字符串 - 类似于 POSIX 和 SVID C 扩展。

* 在 `<string.h>` 中添加 strdup() 和 strndup() 函数以分配字符串的副本 - 类似于 POSIX 和 SVID C 扩展。

* 在 `<stdlib.h>` 中添加 memalignment() 函数以确定指针的字节对齐。

在新的头文件<stdbit.h>中添加位操作函数/宏/类型，用于检查许多整数类型。所有名称均以stdc_开头，以最小化与旧代码和第三方库的冲突。

- 在接下来的内容中，用uc、us、ui、ul、ull替换*以表示五个函数名称，或留空以表示通用类型宏。
- 添加stdc_count_ones*()和stdc_count_zeros*()函数用于计算值中1或0比特的数量。
- 添加stdc_leading_ones*()和stdc_leading_zeros*()函数用于计算值中领先的1或0比特的数量。
- 添加stdc_trailing_ones*()和stdc_trailing_zeros*()函数用于计算值中尾随的1或0比特的数量。
- 添加stdc_first_leading_one*()和stdc_first_leading_zero*()函数用于找到值中首个带有1或0的领先比特。
- 添加stdc_first_trailing_one*()和stdc_first_trailing_zero*()函数用于找到值中首个带有1或0的尾随比特。
- 添加stdc_has_single_bit*()函数用于确定值是否为2的幂（只有一个1比特时返回true）。
- 添加stdc_bit_floor*()函数用于确定不大于值的最大整数2的幂。
- 添加stdc_bit_ceil*()函数用于确定不小于值的最小整数2的幂。
- 添加stdc_bit_width*()函数用于确定表示值所需的比特数。

在<time.h>中添加timegm()函数，将时间结构转换为日历时间值，类似于glibc和musl库中的函数。

## 标准库（现有函数）

- 为printf()函数族添加%b二进制转换格式说明符，将非零值前缀设为0b，类似于%x的工作方式。鼓励之前未使用%B作为自己扩展的实现添加并将非零值前缀设为0B，类似于%X的工作方式。

* 为 scanf() 函数族添加了 %b 二进制转换格式说明符。

* 为 strtol() 和 wcstol() 函数族添加了 0b 和 0B 二进制转换支持。

* 使函数 bsearch()、bsearch_s()、memchr()、strchr()、strpbrk()、strrchr()、strstr() 及其宽字符对应函数 wmemchr()、wcschr()、wcspbrk()、wcsrchr()、wcsstr()，如果给定一个 const 限定对象，则返回一个 const 限定对象。

## 预处理器

* 添加了 #elifdef 和 #elifndef 指令，本质上相当于 #elif defined 和 #elif !defined。这两个指令被加入到 C++23 标准和 GCC 12 编译器中。

* 添加了 `#embed` 指令用于包含二进制资源。

书签

* 添加了 `#warning` 指令用于诊断。

* 添加了 `__has_include` 指令，允许通过预处理指令检查头文件的可用性。

* 添加了 `__has_c_attribute` 指令，允许通过预处理指令检查属性的可用性（查看 "C++ 兼容性" 组以了解新的属性功能）。

* 为可变参数宏添加了 `__VA_OPT__` 功能宏，仅在传递给包含它的宏的可变参数时才扩展到其参数。

## 类型

* 添加了 nullptr_t 类型。

* 为比特精确整数添加了 _BitInt(N) 和 unsigned _BitInt(N) 类型。添加了 BITINT_MAXWIDTH 宏供最大比特宽度使用。添加了用于检查整数运算的 ckd_add()、ckd_sub()、ckd_mul() 宏。

* 可变修改类型（但不包括在栈上分配的自动变量的可变长度数组）成为一个强制性特性。

* 对 typeof(...) 操作符的标准化。

* auto 关键字的含义已更改，导致类型推断，同时如果与类型一起使用，则保留其用作存储类指定符的旧含义。

* 对于 nullptr_t 类型添加 nullptr 常量。
* 添加 wb 和 uwb 整数字面量后缀，用于 _BitInt(N) 和 unsigned _BitInt(N) 类型，例如 6uwb 表示一个 unsigned _BitInt(3)，-6wb 表示一个 signed _BitInt(4)，它有三个值位和一个符号位。
* 添加 0b 和 0B 二进制字面常数前缀，例如 0b10101010（等同于 0xAA）。
* 为文字常量添加 ' 数字分隔符，例如 0xFE'DC'BA'98（等同于 0xFEDCBA98），299'792'458（等同于 299792458），1.414'213'562（等同于 1.414213562）。
* 添加指定枚举的基础类型的能力。
* 允许枚举没有固定的基础类型，可以存储 int 无法表示的值。

## 关键字

* 添加 true 和 false 关键字。
* 添加 alignas、alignof、bool、static_assert、thread_local 关键字。先前定义的关键字成为备用拼写：_Alignas、_Alignof、_Bool、_Static_assert、_Thread_local。
* 添加 _BitInt 关键字（参见“类型”组）
* 添加 typeof 和 typeof_unqual 关键字（参见“类型”组）
* 添加 nullptr 关键字（参见“常量”组）
* 添加 constexpr 关键字（参见“其他”组）
* 添加 _Decimal32、_Decimal64、_Decimal128 关键字用于（可选）十进制浮点算术（参见“其他”组）

## C++ 兼容性

* 使用双方括号 [[]] 添加 C++11 风格的属性语法。添加属性 [[deprecated]],[[fallthrough]],[[maybe_unused]],[[nodiscard]],[[noreturn]] 以与 C++11 兼容，并弃用 C11 中引入的 _Noreturn、noreturn、头文件 <stdnoreturn.h> 功能。为了与 C++23 兼容，允许重复属性。所有标准属性也可以用双下划线括起来（例如[[__deprecated__]] 等同于 [[deprecated]]）。
* 为了与 C++17 兼容，为字符字面值添加 u8 前缀以表示 UTF-8 编码。
* 为了与 C++23 兼容，添加 #elifdef 和 #elifndef 预处理指令。（参见“预处理器”组）
* 为了与 C++17 兼容，添加了单参数 _Static_assert。

- 支持 ISO/IEC 60559:2020 标准，这是 IEEE 754 浮点运算的当前版本，包括扩展的二进制浮点运算和（可选的）十进制浮点运算。
- 标签可以出现在声明之前和复合语句的末尾。
- 函数定义中可以使用未命名参数。
- 更好地支持在数组中使用 const。
- 使用 {} 进行零初始化（包括可变长度数组的初始化）。
- 对象使用 constexpr 限定符，不同于 C++ 的等效物，函数不能使用该限定符。
- 可变参函数不再需要省略号前的命名参数，而且 va_start 宏不再需要第二个参数，如果存在的话，在第一个参数之后不会评估任何参数。
- 添加 char8_t 类型用于存储 UTF-8 编码的数据，并将 u8 字符常量和字符串文字的类型更改为 char8_t。此外，函数 mbrtoc8() 和 c8rtomb() 用于将窄多字节字符转换为 UTF-8 编码和一个UTF-8代码点到窄多字节字符表示的单一代码点。
- 允许存储类修饰符出现在复合文字定义中。 

## 杂项

- 更新 C 版本引用
- 在 stdlib.h 中添加 strfrom 函数
- 验证其他库函数