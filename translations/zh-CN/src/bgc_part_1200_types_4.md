<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 类型 IV：限定符和说明符

既然我们已经掌握了一些类型，那么接下来就是给这些类型加上一些额外的属性来控制它们的行为。这就是 _类型限定符_ 和 _存储类说明符_。

## 类型限定符

[i[类型限定符]<]

这些将允许你声明常量值，并且给编译器提供它可以使用的优化提示。

### `const`

[i[`const` 类型限定符]<]

这是你会经常见到的最常用类型限定符。它表示变量是常量，任何试图修改它的尝试都会让编译器非常愤怒。

``` {.c}
const int x = 2;

x = 4;  // 编译器发出呕吐声，无法对常量赋值
```

你无法改变一个 `const` 值。

通常你会在函数的参数列表中看到 `const`：

``` {.c}
void foo(const int x)
{
    printf("%d\n", x + 30);  // 没问题，不会修改 "x"
}
```

#### `const` 和指针

[i[`const` 类型限定符-->和指针]<]

这个有点复杂，因为在指针方面有两种用法，意义也各不相同。

一方面，我们可以使得无法改变指针所指向的内容。你可以通过在类型声明的时候，将 `const` 放在类型名称（在星号之前）前面来实现。

``` {.c}
int x[] = {10, 20};
const int *p = x; 

p++;  // 我们可以修改 p，没有问题

*p = 30; // 编译器错误！无法改变其指向的内容
```

有些让人困惑的是，这两种用法是等效的：

``` {.c}
const int *p;  // 无法修改 p 所指向的内容
int const *p;  // 无法修改 p 所指向的内容，就像前一行一样
```

很好，所以我们无法改变指针所指向的东西，但是我们可以改变指针本身。如果我们希望反过来呢？我们希望能够改变指针所指向的内容，但不能改变指针本身吗？

只需将 `const` 放在声明中的星号后面即可：

``` {.c}
int *const p;   // 无法通过指针算术修改 "p"

p++;  // 编译器会报错！
```

但是我们可以修改它们所指向的内容：

``` {.c}
int x = 10;
int *const p = &x;

*p = 20;   // 将 "x" 设置为 20，没问题
```

你也可以同时使两者都为 `const`：

``` {.c}
const int *const p;  // 无法修改 p 或 *p！
```

最后，如果您有多级间接指针，您应该为适当的级别添加 `const`。仅仅因为一个指针是 `const` 的，并不意味着它指向的指针也必须是。您可以像下面的例子一样明确设置它们：

``` {.c}
char **p;
p++;     // OK！
(*p)++;  // OK！

char **const p;
p++;     // 错误！
(*p)++;  // OK！

char *const *p;
p++;     // OK！
(*p)++;  // 错误！

char *const *const p;
p++;     // 错误！
(*p)++;  // 错误！
```

[i[`const` 类型修饰符-->和指针]>]

#### `const` 的正确性

[i[`const` 类型修饰符-->正确性]<]

我还需要提到的一点是，编译器将对类似以下代码发出警告：

``` {.c}
const int x = 20;
int *p = &x;
```

警告内容可能类似于：

``` {.default}
initialization discards 'const' qualifier from pointer type target
```

发生了什么呢？

我们需要查看赋值语句两侧的类型：

``` {.c}
    const int x = 20;
    int *p = &x;
//    ^       ^
//    |       |
//  int*    const int*
```

编译器警告我们，赋值语句右边的值是`const`，但左边的值不是。编译器告诉我们，它正在丢弃右边表达式的"const性"。

换句话说，我们_可以_尝试以下操作，但这是错误的。编译器会发出警告，这是未定义行为：

``` {.c}
const int x = 20;
int *p = &x;

*p = 40;  // 未定义行为--也许修改了"x"，也许没有！

printf("%d\n", x);  // 40，如果你运气好的话
```

[i[`const` 类型限定符-->正确性]>]
[i[`const` 类型限定符]>]

### `restrict`

[i[`restrict` 类型限定符]<]

TLDR: 你永远不需要使用它，每次看到它都可以忽略。如果正确使用，可能会获得一些性能提升。如果使用不当，将导致未定义行为。

`restrict` 是一种提示给编译器的方法，表明特定内存块只会被一个指针访问，而不会被其他指针访问。（也就是说，`restrict`指向的对象不会发生别名现象。）如果开发人员声明一个指针为`restrict`，然后以另一种方式访问该指向的对象（例如，通过另一个指针），行为将是未定义的。

基本上你在告诉 C 语言，“嘿---我保证这一个单一指针是我访问这块内存的唯一方式，如果我说谎，你可以对我进行未定义的行为。”

C 语言利用这些信息进行某些优化。例如，如果你在循环中反复解引用`restrict`指针，C 可能会决定将结果缓存到寄存器中，并在循环完成时仅存储最终结果。如果其他任何指向同一内存并在循环中访问该内存的指针，结果将不准确。

```c
// 注意，如果指向的对象从未被写入，`restrict` 将不起作用。这关乎围绕内存写操作的优化。

// 让我们编写一个交换两个变量的函数，我们将使用 `restrict` 关键字来确保 C 知道我们永远不会传入指向同一内容的指针。然后让我们试一试错误地传入指向同一内容的指针。

void swap(int *restrict a, int *restrict b)
{
    int t;

    t = *a;
    *a = *b;
    *b = t;
}

int main(void)
{
    int x = 10, y = 20;

    swap(&x, &y);  // 正确！上面的"a"和"b"指向不同的内容

    swap(&x, &x);  // 未定义行为！"a"和"b"指向相同的内容
}

// 如果去掉上面的 `restrict` 关键字，那将允许两个调用安全运行。但是编译器可能无法进行优化。

// `restrict` 具有块作用域，也就是说，约束只在使用它的作用域中有效。如果它在函数的参数列表中，那它就在函数的块作用域内。

// 如果受限指针指向数组，那只适用于数组中的各个对象。其他指针可以读取和写入这个数组，只要它们不读取或写入受限对象的任何相同元素。

// 如果不在任何函数外部，即文件范围内，该约束则涵盖整个程序。

// 你可能在像 `printf()` 这样的库函数中看到这一点：

int printf(const char * restrict format, ...);

// 再次强调，这只是告诉编译器，在 `printf()` 函数内部，只会有一个指针引用该 `format` 字符串的任何部分。

// 最后一点：如果出于某种原因在函数参数中使用数组表示法而不是指针表示法，你可以像这样使用 `restrict`：
```

``` {.c}
void foo(int p[restrict])     //没有大小

void foo(int p[restrict 10])  //或带有大小
```

但指针表示法更常见。

[i[`restrict` 类型限定符]>]

### `volatile`

[i[`volatile` 类型限定符]<]

除非直接处理硬件，否则您不太可能看到或需要这个。

`volatile`告诉编译器一个值可能在其背后改变，并且应该每次查阅。

例如，编译器在内存中查看一个在幕后持续更新的地址，比如某种硬件定时器。

如果编译器决定优化并在寄存器中存储值很长时间，内存中的值将更新，并且在寄存器中不会反映出来。

通过声明一些内容为 `volatile`，您在告诉编译器：“嘿，这指向的东西可能随时会发生变化，原因在于这段程序代码之外。”

``` {.c}
volatile int *p;
```

### `_Atomic`

这是一个可选的C功能，我们将在[原子章节](#chapter-atomics)中谈论。

[i[`volatile` 类型限定符]>]
[i[类型限定符]>]

## 存储类说明符

[i[存储类说明符]<]

存储类说明符类似于类型限定符。它们为编译器提供关于变量类型的更多信息。

### `auto`

[i[`auto` 存储类]<]

您很少见到这个关键字，因为`auto`是块作用域变量的默认值。这是隐含的。

以下是相同的：

``` {.c}
{
    int a;         // auto是默认值...
    auto int a;    //因此这是多余的
}
```

`auto`关键字指示该对象具有 _自动存储期_。也就是说，它存在于定义它的作用域中，并且在退出作用域时会自动释放。

一个关于自动变量的注意事项是，直到你明确初始化它们，它们的值才是不确定的。我们说它们装着“随机”或“垃圾”数据，尽管这两种说法都不太让我满意。无论如何，除非你初始化它，否则你不会知道里面装的是什么。

总是在使用前初始化所有自动变量！

### `static` {#static}

这个关键词有两个含义，取决于变量是文件作用域还是块作用域。

让我们从块作用域开始。

#### `static` in Block Scope

在这种情况下，我们基本上是在说，“我只希望存在这个变量的单个实例，在调用之间共享”。

也就是说，它的值会在调用之间保留。

具有初始化程序的块作用域中的`static`只会在程序启动时初始化一次，而不是在每次调用该函数时都初始化。

让我们来看一个例子：

``` {.c .numberLines}
#include <stdio.h>

void counter(void)
{
    static int count = 1;  // 这个只初始化一次

    printf("This has been called %d time(s)\n", count);

    count++;
}

int main(void)
{
    counter();  // "This has been called 1 time(s)"
    counter();  // "This has been called 2 time(s)"
    counter();  // "This has been called 3 time(s)"
    counter();  // "This has been called 4 time(s)"
}
```

看到了吗，`count`的值在每次调用之间保持不变了吗？

需要注意的一点是，`static`块作用域变量默认初始化为`0`。

``` {.c}
static int foo;      // 默认的初始值是`0`...
static int foo = 0;  // 因此`0`赋值是多余的
```

最后，请注意，如果你在编写多线程程序，你必须确保不让多个线程同时处理同一个变量。

```c
// 在文件作用域

当你离开了代码块的范围，来到文件作用域时，意义就有些不同了。

文件作用域下的变量已经能够在函数调用之间保持持久，所以这种行为已经存在。

在这种情况下，`static`的意思是这个变量在这个特定的源文件之外不可见。有点像“全局”的意思，但仅限于这个文件内部。

关于如何使用多个源文件构建的内容，请看后面的部分。

// `extern`

`extern`存储类说明符为我们提供了一种引用其他源文件中的对象的方法。

举个例子，假设文件`bar.c`整个内容如下：

``` {.c .numberLines}
// bar.c

int a = 37;
```

就只有这么一句。在文件作用域下声明一个新的`int a`。

但如果我们有另一个源文件`foo.c`，想要引用`bar.c`中的`a`，该怎么办？

使用`extern`关键字很简单：

``` {.c .numberLines}
// foo.c

extern int a;

int main(void)
{
    printf("%d\n", a);  // 37，来自于bar.c！

    a = 99;

    printf("%d\n", a);  // 仍然是来自bar.c的“a”，但现在是99了
}
```

我们也可以在代码块中声明`extern int a`，它同样会引用`bar.c`中的`a`：

``` {.c .numberLines}
// foo.c

int main(void)
{
    extern int a;

    printf("%d\n", a);  // 37，来自于bar.c！

    a = 99;

    printf("%d\n", a);  // 仍然是来自bar.c的“a”，但现在是99了
}
```

如果`bar.c`中的`a`被标记为`static`，这个方法就行不通了。文件作用域下的`static`变量在该文件之外是不可见的。
```

关于函数上的 `extern` 的最后一点。对于函数，`extern` 是默认的，所以是多余的。如果你只希望在单个源文件中可见，你可以声明一个函数为 `static`。

### `register`

这是一个关键字，用于提示编译器这个变量经常被使用，应该尽可能地提高访问速度。编译器并不一定会同意。

现代的 C 编译器优化器非常有效地能够自行解决这个问题，所以现在很少见到这种情况了。

但如果你必须：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    register int a;   // 使 "a" 尽可能快速地使用。

    for (a = 0; a < 10; a++)
        printf("%d\n", a);
}
```

然而，这是有代价的。你不能获取一个寄存器的地址：

``` {.c}
register int a;
int *p = &a;    // 编译器错误！无法获取寄存器的地址
```

对于数组的任何部分也是一样的：

``` {.c}
register int a[] = {11, 22, 33, 44, 55};
int *p = a;  // 编译器错误！无法获取 a[0] 的地址
```

或是对数组的部分进行解引用：

``` {.c}
register int a[] = {11, 22, 33, 44, 55};

int a = *(a + 2);  // 编译器错误！获取了 a[0] 的地址
```

有趣的是，对于使用数组表示形式的等价情况，gcc 只会发出警告：

``` {.c}
register int a[] = {11, 22, 33, 44, 55};

int a = a[2];  // 编译器警告！
```

带有：

``` {.default}
warning: ISO C forbids subscripting ‘register’ array
```

```c
// Translate the following content

The fact that you can't take the address of a register variable frees
the compiler up to make optimizations around that assumption if it
hasn't figured them out already. Also adding `register` to a `const`
variable prevents one from accidentally passing its pointer to another
function that willfully ignore its
constness^[https://gustedt.wordpress.com/2010/08/17/a-common-misconsception-the-register-keyword/].

A bit of historic backstory, here: deep inside the CPU are little
dedicated "variables" called [flw[_registers_|Processor_register]]. They
are super fast to access compared to RAM, so using them gets you a speed
boost. But they're not in RAM, so they don't have an associated memory
address (which is why you can't take the address-of or get a pointer to
them).

But, like I said, modern compilers are really good at producing optimal
code, using registers whenever possible regardless of whether or not you
specified the `register` keyword. Not only that, but the spec allows
them to just treat it as if you'd typed `auto`, if they want. So no
guarantees.

[i[`register` storage class]>]

### `_Thread_local`

[i[`_Thread_local` storage class]<]

When you're using multiple threads and you have some variables in either
global or `static` block scope, this is a way to make sure that each
thread gets its own copy of the variable. This'll help you avoid race
conditions and threads stepping on each other's toes.

If you're in block scope, you have to use this along with either
`extern` or `static`.

Also, if you include `<threads.h>`, you can use the rather more
palatable `thread_local` as an alias for the uglier `_Thread_local`.

More information can be found in the [Threads section](#thread-local).

[i[`_Thread_local` storage class]>]
[i[Storage-Class Specifiers]>]
```