<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 类型部分 V: 复合字面量和通用选择

这是关于类型的最后一章节！我们将讨论两件事情：

- 如何拥有“匿名”未命名的对象以及它们的用途。
- 如何生成依赖类型的代码。

它们之间并没有特别的关联，但也不值得各自单独成章。所以我像个叛逆一样把它们塞到了这里！

## 复合字面量

[i[复合字面量]<]

这是该语言的一个很棒的特性，允许您在不将其分配给变量的情况下即时创建某种类型的对象。您可以创建简单类型、数组、`struct`，随心所欲。

其中一个主要用途是在您不想创建临时变量来保存值时，将复杂参数传递给函数。

创建复合字面量的方式是将类型名称放在括号中，然后在其后放置一个初始化列表。例如，一个未命名的`int`数组可能如下所示：

``` {.c}
(int []){1,2,3,4}
```

现在，这行代码本身并没有做任何事情。它创建了一个包含 4 个`int`的未命名数组，然后在不使用它们的情况下丢弃它们。

我们可以使用指针来存储对数组的引用...

``` {.c}
int *p = (int []){1 ,2 ,3 ,4};

printf("%d\n", p[1]);  // 2
```

但这似乎有点冗长来拥有一个数组。我是说，我们本来可以直接这样做^[这并不完全相同，因为它是一个数组，而不是指向`int`的指针。]：

``` {.c}
int p[] = {1, 2, 3, 4};

printf("%d\n", p[1]);  // 2
```

因此，让我们看一个更有用的示例。

### 向函数传递未命名对象

[i[复合字面量-->传递给函数]<]

假设我们有一个用于对`int`数组求和的函数：

``` {.c}
int sum(int p[], int count)
{
    int total = 0;

    for (int i = 0; i < count; i++)
        total += p[i];

    return total;
}
```

如果我们想调用它，通常需要这样做，声明一个数组并将值存储在其中以传递给函数：

``` {.c}
int a[] = {1, 2, 3, 4};

int s = sum(a, 4);
```

但是，无名对象为我们提供了一种通过直接传递它来跳过变量的方法（上面列出了参数名称）。看一下---我们将用一个无名数组来替换变量`a`，作为第一个参数传递：

``` {.c}
//                   p[]         count
//           |-----------------|  |
int s = sum((int []){1, 2, 3, 4}, 4);
```

相当巧妙！

### 无名`struct`s

我们也可以用`struct`做类似的事情。

首先，让我们不使用无名对象来操作。我们将定义一个`struct`来保存一些`x`/`y`坐标。然后，我们将定义一个，将值传递给它的初始化程序。最后，我们将将其传递给一个函数来打印这些值：

``` {.c .numberLines}
#include <stdio.h>

struct coord {
    int x, y;
};

void print_coord(struct coord c)
{
    printf("%d, %d\n", c.x, c.y);
}

int main(void)
{
    struct coord t = {.x=10, .y=20};

    print_coord(t);   // 打印 "10, 20"
}
```

足够简单明了吧？

让我们修改一下，使用一个无名对象来代替传递给`print_coord()`的变量`t`。

我们只需要把`t`拿出来，用一个无名`struct`来替换它：

``` {.c .numberLines startFrom="7"}
    //struct coord t = {.x=10, .y=20};

    print_coord((struct coord){.x=10, .y=20});   // 打印 "10, 20"
```

仍然能正常工作！
```

[i[`struct`关键字-->复合字面量]>]
[i[复合字面量-->使用`struct`]>]

### 指向未命名对象的指针

[i[复合字面量-->指针]<]

你可能已经注意到在上一个例子中，即使我们使用了`struct`，我们传递给`print_coord()`的是`struct`的副本，而不是指向`struct`的指针。

事实证明，我们可以像往常一样，通过`&`来获取未命名对象的地址。

这是因为一般情况下，如果某个运算符对于该类型的变量起作用，你也可以在该类型的未命名对象上使用该运算符。

让我们修改上面的代码，这样我们就可以传递指向未命名对象的指针

``` {.c .numberLines}
#include <stdio.h>

struct coord {
    int x, y;
};

void print_coord(struct coord *c)
{
    printf("%d, %d\n", c->x, c->y);
}

int main(void)
{
    //     注意 &
    //          |
    print_coord(&(struct coord){.x=10, .y=20});   // 输出 "10, 20"
}
```

此外，这也是将指针传递给简单对象的好方法：

``` {.c}
// 传递一个值为 3490 的 int 指针
foo(&(int){3490});
```

就是这么简单。

[i[复合字面量-->指针]>]

### 未命名对象和作用域

[i[复合字面量-->作用域]<]

未命名对象的生命周期在其作用域结束时结束。这可能会给你带来一些麻烦，如果你创建一个新的未命名对象，获取其指针，然后离开对象的作用域。在这种情况下，指针将指向一个无效的对象。

因此，以下情况产生未定义行为：

``` {.c}
int *p;

{
    p = &(int){10};
}

printf("%d\n", *p);  // 无效: (int){10} 已经超出作用域
```

同样，你不能从函数中返回指向未命名对象的指针。对象在超出作用域时将被释放：

``` {.c .numberLines}
#include <stdio.h>

```c
int *get3490(void)
{
    // 别这样做
    return &(int){3490};
}

int main(void)
{
    printf("%d\n", *get3490());  // 无效：(int){3490} 超出作用域
}
```

```c
    char *s = _Generic(i,
                    int: "that variable is an int",
                    float: "that variable is a float",
                    default: "that variable is some type"
                );

    printf("%s\n", s);
}
```

看看从第9行开始的`_Generic`表达式。

当编译器看到它时，会查看第一个参数的类型。
（在这个示例中，是变量`i`的类型。）然后它会查找匹配这种类型的情况。然后将参数代替整个`_Generic`表达式。

在这种情况下，`i`是一个`int`，所以匹配到该情况。然后字符串被代入表达式中。因此，当编译器看到它时，这行变成了这样：

``` {.c}
    char *s = "that variable is an int";
```

如果编译器在`_Generic`中找不到类型匹配，它会查找可选的`default`情况并使用那个情况。

如果找不到类型匹配且没有`default`，就会出现编译错误。第一个表达式**必须**匹配其中一种类型或`default`。

因为一遍又一遍地写`_Generic`太麻烦了，通常被用来制作一个宏的主体，以便可以轻松重复使用。

我们来创建一个名为`TYPESTR(x)`的宏，它接受一个参数并返回一个带有参数类型的字符串。

因此，`TYPESTR(1)`将返回字符串`"int"`，例如。

让我们来看看：

``` {.c}
#include <stdio.h>

#define TYPESTR(x) _Generic((x), \
                        int: "int", \
                        long: "long", \
                        float: "float", \
                        double: "double", \
                        default: "something else")

int main(void)
{
    int i;
    long l;
    float f;
    double d;
    char c;
```

```c
printf("i 的类型是 %s\n", TYPESTR(i));
printf("l 的类型是 %s\n", TYPESTR(l));
printf("f 的类型是 %s\n", TYPESTR(f));
printf("d 的类型是 %s\n", TYPESTR(d));
printf("c 的类型是 %s\n", TYPESTR(c));
}
```

这将输出：

``` {.default}
i 的类型是 int
l 的类型是 long
f 的类型是 float
d 的类型是 double
c 的类型是 其他
```

这并不奇怪，因为，就像我们说的那样，`main()`函数中的代码在编译时被替换为以下内容：

``` {.c}
printf("i 的类型是 %s\n", "int");
printf("l 的类型是 %s\n", "long");
printf("f 的类型是 %s\n", "float");
printf("d 的类型是 %s\n", "double");
printf("c 的类型是 %s\n", "其他");
```

这正是我们看到的输出。

让我们再做一个例子。我在这里添加了一些宏，这样当你运行：

``` {.c}
int i = 10;
char *s = "Hello!";

PRINT_VAL(i);
PRINT_VAL(s);
```

你会得到输出：

``` {.default}
i = 10
s = Hello!
```

我们需要利用一些宏魔术来实现这个目的。

``` {.c .numberLines}
#include <stdio.h>
#include <string.h>

// 为类型提供一个格式说明符的宏
#define FMTSPEC(x) _Generic((x), \
                        int: "%d", \
                        long: "%ld", \
                        float: "%f", \
                        double: "%f", \
                        char *: "%s")
                        // TODO: 添加更多类型
                        
// 打印变量形如 "name = value" 的宏
#define PRINT_VAL(x) do { \
    char fmt[512]; \
    snprintf(fmt, sizeof fmt, #x " = %s\n", FMTSPEC(x)); \
    printf(fmt, (x)); \
} while(0)

int main(void)
{
    int i = 10;
    float f = 3.14159;
    char *s = "Hello, world!";

    PRINT_VAL(i);
    PRINT_VAL(f);
    PRINT_VAL(s);
}
```

``` {.default}
i = 10
f = 3.141590
s = Hello, world!
```

我们本可以把所有东西都塞进一个大宏中，但我把它分成两个部分，以避免眼睛出血。

[i[通用选择]>]
```