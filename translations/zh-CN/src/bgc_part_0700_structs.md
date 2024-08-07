<!-- Beej的C语言指南 -->

# 结构体 {#structs}

[i[`struct`关键字]<]
在C语言中，我们有一种被称为`struct`的东西，它是一种用户自定义的类型，可以容纳多个数据片段，可能是不同类型的数据。

这是一种将多个变量捆绑成一个的便捷方式。这对于将变量传递给函数（这样您只需要传递一个变量而不是多个变量）是有益的，并且有助于组织数据和使代码更易读。

如果您来自另一种语言，您可能熟悉_类_和_对象_的概念。这些概念在C中不存在本地[^是的，在C中像`int`这样的内存中的单独项目被称为"对象"，但它们并不是面向对象编程意义上的对象。]。你可以将`struct`看作是一个只有数据成员而没有方法的类。

## 声明结构体

[i[`struct`关键字-->声明]<]
您可以在代码中像这样声明一个`struct`：

``` {.c}
struct car {
    char *name;
    float price;
    int speed;
};
```

这通常是在全局范围声明，不在任何函数的内部，以便`struct`是全局可用的。

这样做，您就创建了一个新的_类型_。完整的类型名称是`struct car`。（不只是`car`---那样是行不通的。）

还没有任何该类型的变量，但我们可以声明一些：

``` {.c}
struct car saturn;  // 类型为 "struct car" 的变量"saturn"
```

现在我们有一个未初始化的变量`saturn`[^Saturn是美国一款经济车的热门品牌，但在2008年的经济危机中遭到倒闭，对于我们的粉丝来说是个遗憾。]，类型为`struct car`。

我们应该对其进行初始化！但我们如何设置这些单独字段的值？

就像许多其他从C语言中借鉴的语言一样，我们将使用点运算符（`.`）来访问各个字段。

``` {.c}
saturn.name = "Saturn SL/2";
saturn.price = 15999.99;
saturn.speed = 175;

printf("名称：     %s\n", saturn.name);
printf("价格（美元）：%f\n", saturn.price);
printf("最高速度（公里）：%d\n", saturn.speed);
```

在第一行，我们在`struct car`中设置了值，然后在接下来的部分中，我们打印了这些值出来。

## 结构体初始化器 {#struct-initializers}

在前一部分的示例中有点繁琐。初始化`struct`变量肯定有更好的方法！

你可以使用初始化器通过在定义变量时按照`struct`中字段的顺序放入值来完成（在定义时进行，定义后无法再使用，定义时就要完成）。

``` {.c}
struct car {
    char *name;
    float price;
    int speed;
};

// 现在用一个初始化器！与`struct`声明中的字段顺序相同：
struct car saturn = {"Saturn SL/2", 16000.99, 175};

printf("名称：      %s\n", saturn.name);
printf("价格：     %f\n", saturn.price);
printf("最高速度： %d 公里\n", saturn.speed);
```

初始化器中的字段需要按照相同顺序有点奇怪。如果有人更改了`struct car`中的顺序，可能会造成所有其他代码的破坏！

我们可以更具体地使用我们的初始化器：

``` {.c}
struct car saturn = {.speed=175, .name="Saturn SL/2"};
```

现在与`struct`声明中的顺序无关了。这绝对是更安全的代码。

类似于数组初始化器，任何缺失的字段标识都会被初始化为零（在这种情况下，我省略了`.price`）。

## 将结构体传递给函数

您可以通过以下两种方式将结构体传递给函数。

1. 传递结构体本身。
2. 传递指向结构体的指针。

请记住，当您将某样东西传递给函数时，函数会为其创建一个 _副本_ 以供操作，无论它是指针的副本、整数、结构体还是其他内容。

通常有两种情况下您想传递指向结构体的指针：

1. 您需要函数能够更改传入的结构体，并且希望这些更改在调用方中显示。
2. 结构体相对较大，在栈上复制较昂贵，只需复制一个指针就更经济^[指针在64位系统上可能为8个字节]。

出于这两个原因，将指针传递给函数远比传递结构体本身更常见，尽管传递结构体本身并不是非法的。

让我们尝试传递一个指针，创建一个函数，允许您设置“struct car”的`.price`字段：

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

    // 传递一个指向这个“struct car”的指针，以及一个新的，
    // 更实际的价格：
    set_price(&saturn, 799.99);

    printf("Price: %f\n", saturn.price);
}
```

看着那里的参数类型，您应该能够想出`set_price()`的函数签名。

`saturn` 是一个 `struct car`，所以 `&saturn` 必须是 `struct car` 的地址，也就是指向 `struct car` 的指针，即 `struct car*`。

而 `799.99` 是一个 `float`。

因此函数声明必须如下所示：

``` {.c}
void set_price(struct car *c, float new_price)
```

我们只需要编写其函数体。一种尝试可能是：

``` {.c}
void set_price(struct car *c, float new_price) {
    c.price = new_price;  // 错误！！
}
```

这样是行不通的，因为点运算符只能用于 `struct`……不能用于指向 `struct` 的_指针_。

好的，我们可以对变量 `c` 进行解引用以取消指针，以便访问 `struct` 本身。对 `struct car*` 进行解引用会得到指针指向的 `struct car`，我们应该可以在其上使用点操作符：

``` {.c}
void set_price(struct car *c, float new_price) {
    (*c).price = new_price;  // 可行，但是有点笨拙和不符惯用法 :(
}
```

这样可以！但是键入所有那些括号和星号有点笨拙。C语言有一些语法糖，称为 _箭头操作符_，可以帮助解决这个问题。
[i[`struct` 关键字-->传递和返回]>]

## 箭头操作符

[i[`->` 箭头操作符]<]
箭头操作符帮助引用指向 `struct` 的指针中的字段。

``` {.c}
void set_price(struct car *c, float new_price) {
    // (*c).price = new_price;  // 可行，但是不符惯用法 :(
    //
    // 上面的这行100%等价于下面的这行：

    c->price = new_price;  // 这就是它！
}
```

所以在访问字段时，何时使用点号，何时使用箭头呢？

* 如果你有一个 `struct`，使用点号（`.`）。
* 如果你有一个指向 `struct` 的指针，使用箭头（`->`）。
[i[`->` 箭头操作符]>]

## 复制和返回 `struct`

[i[`struct` 关键字-->复制]<]
这里有一个简单的示例给你！

只需将一个赋值给另一个！

``` {.c}
struct car a, b;

b = a;  // 复制结构
```

从函数返回`struct`（而不是指向它的指针）也会使接收变量进行类似的复制。

这并不是一种"深复制"。所有字段都按原样复制，包括指向其他值的指针。
[i[`struct`关键字-->复制] >]

## 比较`struct`s

[i[`struct`关键字-->比较]<]
只有一种安全的方法来做：逐个比较每个字段。

你可能认为可以使用
[fl[`memcmp()`|https://beej.us/guide/bgclr/html/split/stringref.html#man-strcmp]],
但这并不能处理可能存在的[填充字节](#结构填充字节)情况。

如果先使用[fl[`memset()`|https://beej.us/guide/bgclr/html/split/stringref.html#man-memset]]将`struct`清零，那么有可能会有效，尽管可能存在一些奇怪的元素，[无法如您所预期那样进行比较|https://stackoverflow.com/questions/141720/how-do-you-compare-structs-for-equality-in-c]。
[i[`struct`关键字-->比较]>] [i[`struct`关键字]>]