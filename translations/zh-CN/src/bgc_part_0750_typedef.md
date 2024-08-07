`typedef`：创建新类型

[`typedef`关键字]  
实际上并不是创建_新_类型，而是为现有类型取一个新名字。从表面上看听起来有点毫无意义，但我们可以利用这一点使我们的代码更清晰。

## 理论上的`typedef`

基本上，你可以使用`typedef`为现有类型创建别名。

像这样：
``` {.c}
typedef int antelope;  // 为"int"取一个别名"antelope"

antelope x = 10;       // 类型"antelope"就等同于类型"int"
```

你可以对任何现有类型进行这样的操作。你甚至可以用逗号列表创建多种类型：

``` {.c}
typedef int antelope, bagel, mushroom;  // 这些都是"int"
```

这真的很有用，对吧？你可以输入`mushroom`而不是`int`？你一定对这个功能_非常激动_！

好吧，讽刺教授---我们马上会讨论到这个功能更常见的应用。

### 作用域

[`typedef`关键字-->作用域规则]  
`typedef`遵循常规[作用域规则](#scope)。

因此，很常见在文件作用域("全局")中查找`typedef`，这样所有函数都能自由使用新类型。

## 实践中的`typedef`

因此，将`int`重命名为其他名称并不那么令人激动。让我们看看`typedef`通常出现在哪些地方。

### `typedef`和`struct`s

[`typedef`关键字-->使用`struct`s]  
有时，`struct`会被重命名为一个新名称，这样你就不必一遍又一遍地输入`struct`这个单词了。

``` {.c}
struct animal {
    char *name;
    int leg_count, speed;
};

// 原始名称          新名称
//            |         |
//            v         v
//      |-----------| |----|
typedef struct animal animal;

``` {.c}
struct animal y;  // 这能运作
animal z;         // 这也可以，因为"animal"是一个别名
```

就我个人而言，我不喜欢这种做法。我喜欢当你在类型前面加上`struct`这个词时代码更加清晰；程序员知道他们得到了什么。但这种做法非常普遍，所以我在这里包含了它。

现在我想以一种常见的方式运行完全相同的示例。我们将把`struct animal` _放在_`typedef`里面。你可以这样混在一起：

``` {.c}
//  原始名字
//            |
//            v
//      |-----------|
typedef struct animal {
    char *name;
    int leg_count, speed;
} animal;                         // <-- 新名字

struct animal y;  // 这能运作
animal z;         // 这也可以，因为"animal"是一个别名
```

这与前面的示例完全相同，只是更简洁。

[i[`typedef`关键字-->使用匿名`struct`s]<]
不仅如此！还有另一个常见的快捷方式，你可能在代码中看到，它使用了所谓的_匿名结构体_^[我们稍后会更多地讨论这些。]。事实证明，实际上在各种地方不需要为结构体命名，其中包括`typedef`。

让我们用一个匿名结构体做同样的示例：

``` {.c}
//  匿名结构体！它没有名字！
//         |
//         v
//      |----|
typedef struct {
    char *name;
    int leg_count, speed;
} animal;                         // <-- 新名字

//struct animal y;  // 错误：这不再起作用--没有这样的结构体！
animal z;           // 这可以，因为"animal"是一个别名
```

作为另一个例子，我们可能会发现类似这样的东西：

``` {.c}
typedef struct {
    int x, y;
} point;

point p = {.x=20, .y=40};
```

printf("%d, %d\n", p.x, p.y);  // 20, 40

### `typedef` 和其他类型

并不是使用 `typedef` 与简单的类型如 `int` 完全毫无用处…… 它可以帮助你抽象类型，以便以后更容易地对其进行更改。

例如，如果你的代码中到处都是 `float`，要在后期发现需要将它们全部更改为 `double` 将会很麻烦。

但是，如果你稍微准备一下：

``` {.c}
typedef float app_float;

// 和

app_float f1, f2, f3;
```

那么如果以后你想要更改为另一种类型，比如 `long double`，你只需要更改 `typedef`：

``` {.c}
//        voila!
//      |---------|
typedef long double app_float;

// 并且不需要更改这一行：

app_float f1, f2, f3;  // 现在这里都是 long doubles
```

### `typedef` 和指针

你可以创建一个指针类型。

``` {.c}
typedef int *intptr;

int a = 10;
intptr x = &a;  // "intptr" 是类型 "int*"
```

我真的不喜欢这种做法。它隐藏了 `x` 是指针类型这一事实，因为在声明中看不到 `*`。

在我看来，最好明确显示你正在声明一个指针类型，以便其他开发人员能清晰地看到它，不会误将 `x` 误认为非指针类型。

但是，根据最近的统计，比如说，有 832,007 人有不同的看法。

### `typedef` 和大写

我见过各种各样的 `typedef` 大写风格。

``` {.c}
typedef struct {
    int x, y;
} my_point;          // 下划线命名法

typedef struct {
    int x, y;
} MyPoint;          // 驼峰命名法

``` {.c}
typedef struct {
    int x, y;
} Mypoint;          // 首字母大写

typedef struct {
    int x, y;
} MY_POINT;          // 全部大写蛇形命名法
```

C11规范没有规定哪种方式更好，且展示了全大写和全小写的示例。

K&R2主要使用首字母大写，但也展示了一些全大写和蛇形命名法的示例（带下划线`_t`）。

如果有所遵循的编码风格指南，请坚持。如果没有，请选一份使用并坚持。

## 数组和`typedef`

[i[`typedef`关键字-->带有数组]<]
语法有点奇怪，在我看来很少见，但你可以`typedef`一个包含一定数量项的数组。

``` {.c}
// 定义类型five_ints为包含5个int的数组
typedef int five_ints[5];

five_ints x = {11, 22, 33, 44, 55};
```

我不喜欢它，因为它隐藏了变量的数组特性，但是确实是可行的。
[i[`typedef`关键字-->带有数组]>]
[i[`typedef`关键字]>]
```