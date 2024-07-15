<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72 -->


# 结构体 {#structs}

`struct` 关键字
在C里，我们有一种叫`struct`的东西，它是一种用户可定义的类型，可以容纳多个数据片段，可能是不同的类型。

这是一种将多个变量捆绑成一个变量的便利方式。这对于将变量传递给函数很有益处（这样你只需传递一个变量而不是很多个），并且对于组织数据并使代码更易读也很有用。

如果你来自另一种语言，你可能熟悉_类_和_对象_的概念。这两者在C中是不存在的，本地不存在[虽然在C中像`int`这样的单个内存项被称为“对象”，但它们不是面向对象编程意义上的对象。]。你可以将`struct`看作只有数据成员而没有方法的类。

## 声明结构体

`struct` 关键字
你可以这样在你的代码中声明一个`struct`：

``` {.c}
struct car {
    char *name;
    float price;
    int speed;
};
```

这通常在任何函数外全局范围内完成，以便全局可用。

当你这样做时，你正在创建一个新的类型。完整的类型名是`struct car`。（不只是`car`--那样行不通。）

尚未有该类型的任何变量，但我们可以声明一些：

``` {.c}
struct car saturn;  // 类型为“struct car” 的变量“saturn”
```

现在我们有一个未初始化的变量`saturn` [Saturn品牌是美国的一种热门经济车型，直到2008年的经济危机将其拖垮。我们这些粉丝对此感到遗憾。]，它的类型是 `struct car`。

我们应该初始化它！但我们如何设置这些各个字段的值呢？
```

和许多其他编程语言一样从C语言那里借鉴来的，我们将使用点运算符(`.`)来访问结构体中的各个字段。

``` {.c}
saturn.name = "Saturn SL/2";
saturn.price = 15999.99;
saturn.speed = 175;

printf("名称:           %s\n", saturn.name);
printf("价格（美元）:    %f\n", saturn.price);
printf("最高时速（公里/小时）: %d\n", saturn.speed);
```

在前几行中，我们设置了`struct car`中的值，然后在接下来的部分中，我们将这些值打印出来。
[i[`struct`关键词-->声明]>]

## 结构体初始化器 {#struct-initializers}

[i[`struct`关键词-->初始化器]<]
前一节中的例子有点笨重。肯定有一种更好的方法来初始化那个`struct`变量！

您可以使用初始值设定项，在定义变量时按照在`struct`中出现时的顺序为字段赋值（这在定义变量后不起作用---必须在定义时发生）。

``` {.c}
struct car {
    char *name;
    float price;
    int speed;
};

// 现在采用初始化器！与struct声明中的字段顺序相同：
struct car saturn = {"Saturn SL/2", 16000.99, 175};

printf("名称:      %s\n", saturn.name);
printf("价格:     %f\n", saturn.price);
printf("最高时速: %d 公里/小时\n", saturn.speed);
```

初始化器中字段需要按照相同顺序排列的事实有点怪异。如果有人改变了`struct car`中的顺序，可能会破坏所有其他代码！

我们可以更具体地使用初始化器：

``` {.c}
struct car saturn = {.speed=175, .name="Saturn SL/2"};
```

现在它与`struct`声明中的顺序无关了。这无疑是更安全的代码。

类似于数组初始化器，任何缺失的字段指示器都会被初始化为零（在这种情况下，应为`.price`，我已经省略了）。

## 将结构体传递给函数

你可以通过几种方式将结构体传递给函数。

1. 传递结构体本身。
2. 传递结构体的指针。

记住，当你将某些内容传递给函数时，函数会对这个内容进行_拷贝_，无论是指针、整数、结构体还是其他任何内容。

通常有两种情况需要传递结构体的指针：

1. 你需要函数能够对传入的结构体进行更改，并且让调用者能够看到这些更改。
2. 结构体相对较大，在栈上进行拷贝比仅仅拷贝一个指针要昂贵得多^[在64位系统上，指针通常是8字节大小]。

出于这两个原因，通常更常见的做法是将结构体的指针传递给函数，但将结构体本身传递给函数也并非不合法。

让我们尝试传递一个指针，创建一个函数，允许你设置`struct car`的`.price`字段：

``` {.c .numberLines}
#include <stdio.h>

struct car {
    char *name;
    float price;
    int speed;
};

int main(void)
{
    struct car saturn = {.speed=175, .name="Saturn SL/2"};

    // 传递结构体car的指针，并提供一个新的、更现实的价格：
    set_price(&saturn, 799.99);

    printf("Price: %f\n", saturn.price);
}
```

通过观察这里的参数类型，你可以轻松得出`set_price()`函数的函数签名。

`saturn`是一个`struct car`，所以`&saturn`必须是`struct car`的地址，也就是指向`struct car`的指针，也就是`struct car*`。

而`799.99`是一个`float`。

所以函数声明必须是这样的：

``` {.c}
void set_price(struct car *c, float new_price)
```

我们只需要编写函数体。一个尝试可能是：

``` {.c}
void set_price(struct car *c, float new_price) {
    c.price = new_price;  // 错误！
}
```

这不会起作用，因为点运算符只适用于`struct`……它不适用于指向`struct`的_指针_。

好的，所以我们可以对变量`c`进行解引用以解除指针以访问到`struct`本身。对`struct car*`进行解引用会得到指针指向的`struct car`，然后我们可以使用点运算符：

``` {.c}
void set_price(struct car *c, float new_price) {
    (*c).price = new_price;  // 有效，但不太优雅和惯用 :(
}
```

这也可以！但是输入所有这些括号和星号有点笨拙。C语言有一种叫做箭头操作符的语法糖，可以帮助简化操作。
[i[`struct`关键字-->传递和返回]>]

## 箭头操作符

[i[`->`箭头操作符]<]
箭头操作符帮助我们访问指向`struct`的指针中的字段。

``` {.c}
void set_price(struct car *c, float new_price) {
    // (*c).price = new_price;  // 有效，但不太优雅和惯用 :(
    //
    // 上面的代码完全等同于下面的代码：

    c->price = new_price;  // 就是这个！
}
```

所以在访问字段时，何时使用点号，何时使用箭头呢？

- 如果你有一个`struct`，使用点号（`.`）。
- 如果你有一个指向`struct`的指针，使用箭头（`->`）。
[i[`->`箭头操作符]>]

## 复制和返回`struct`

[i[`struct`关键字-->复制]<]
这是一个简单的例子！

只需从一个变量赋值给另一个变量！

``` {.c}
struct car a, b;

b = a;  // 复制结构体
```

而从函数返回`struct`（而不是指向它的指针）也会对接收变量进行类似复制。

这不是"深复制"。所有字段都按原样复制，包括指针指向的内容。

## 比较`struct`s

要安全地比较`struct`，只有一种方法：逐个比较每个字段。

你可能认为可以使用`memcmp()`，但这不处理可能存在的填充字节的情况。如果首先用`memset()`将`struct`清零，那么它可能有效，尽管可能出现奇怪的元素，可能比较结果与预期不同。