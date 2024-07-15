``` {.c}
typedef：定义新类型

[i[`typedef` 关键字]<]
嗯，并不是创建_新的_类型，而是为现有类型取个新名字。乍一看似乎有点无聊，但我们确实可以利用这个功能让我们的代码更清晰。

## 理论上的 `typedef`

基本上，你拿一个已有的类型，用 `typedef` 为它创建一个别名。

就像这样：

``` {.c}
typedef int antelope;  // 为 "int" 创建一个别名 "antelope"

antelope x = 10;       // "antelope" 类型就和 "int" 一样
```

你可以对任意已有类型这样做。甚至可以用逗号分隔创建多个别名：

``` {.c}
typedef int antelope, bagel, mushroom;  // 这些都是 "int"
```

是不是很方便？可以用 `mushroom` 替代 `int`？你一定对这个功能_兴奋不已_！

好了，讽刺教授——马上我们就会讨论一些更常见的应用方式。

### 作用域

[i[`typedef` 关键字-->作用域规则]<]
`typedef` 遵循常规的[作用域规则](#scope)。

因此，在文件作用域（"全局"）下发现 `typedef` 是相当常见的，这样所有函数都可以随意使用新类型。

## 实际应用中的 `typedef`

所以将 `int` 重命名为其他名字其实并不那么有趣。让我们看看在哪些地方 `typedef` 常常出现。

### `typedef` 和 `struct` {#typedef-struct}

[i[`typedef` 关键字-->结构体]<]
有时为了避免一遍又一遍地输入 `struct` 这个词，会将 `struct` 收到新名字。

``` {.c}
struct animal {
    char *name;
    int leg_count, speed;
};

//  原名            新名
//            |         |
//            v         v
//      |-----------| |----|
typedef struct animal animal;
```

```c
struct animal y; // 这样也可以
animal z; // 这个也可以，因为"animal"是一个别名

就个人而言，我不太喜欢这样的做法。我更喜欢在类型前加上`struct`这个词，这样代码更清晰，程序员知道他们得到了什么。但这个做法非常普遍，所以我在这里包括了它。

现在我想以一种你经常看到的方式运行完全相同的例子。我们将把`struct animal` _放进_`typedef`中。你可以像这样把它们全部结合在一起：

// 原始名称
//           |
//           v
//      |-----------|
typedef struct animal {
    char *name;
    int leg_count, speed;
} animal; // <-- 新名称

struct animal y; // 这样也可以
animal z; // 这个也可以，因为"animal"是一个别名

这与先前的示例完全相同，只是更简洁。

但这还不是全部！还有另一个常见的快捷方式，你可能在使用匿名结构的代码中看到，它们被称为_匿名结构_^[我们稍后会详细讨论这个]。事实证明，在许多地方你其实并不需要给结构命名，其中`typedef`就是其中之一。

让我们使用匿名结构做同样的例子：

// 匿名结构！它没有名字！
//        |
//        v
//      |----|
typedef struct {
    char *name;
    int leg_count, speed;
} animal; // <-- 新名称

//struct animal y; // 错误：这样将不再起作用-- 没有这样的结构！
animal z; // 这个可以，因为"animal"是一个别名

作为另一个例子，我们可能会发现类似于这样的东西：

typedef struct {
    int x, y;
} point;

point p = {.x=20, .y=40};
```

```c
printf("%d, %d\n", p.x, p.y);  // 20, 40
```

### `typedef` 和其他类型

使用 `typedef` 与 `int` 这样的简单类型并不是完全无用...它帮助你抽象类型，使以后更容易更改它们。

例如，如果你的代码中到处都是 `float`，如果以后因为某些原因必须将它们全部更改为 `double`，那么将会很痛苦。

但是如果你稍微准备一下：

``` {.c}
typedef float app_float;

// 还有

app_float f1, f2, f3;
```

然后，如果以后想要更改为另一种类型，比如 `long double`，你只需要更改 `typedef`：

``` {.c}
//        voila!
//      |---------|
typedef long double app_float;

// 不需要更改这一行：

app_float f1, f2, f3;  // 现在这些都是 long doubles
```

### `typedef` 和指针

你可以创建一个指针类型。

``` {.c}
typedef int *intptr;

int a = 10;
intptr x = &a;  // "intptr" 是 "int*" 类型
```

我真的不喜欢这种做法。它隐藏了 `x` 是指针类型的事实，因为在声明中看不到 `*`。

在我看来，最好明确显示出你在声明一个指针类型，这样其他开发人员可以清楚地看到 `x` 是指针类型，不会误解 `x` 具有非指针类型。

但最后统计的人数，比如说，832,007 人有不同的看法。

### `typedef` 和大写

我见过各种大小写形式的 `typedef`。

``` {.c}
typedef struct {
    int x, y;
} my_point;          // 小写蛇形命名

typedef struct {
    int x, y;
} MyPoint;          // 驼峰命名
```

```c
typedef struct {
    int x, y;
} 点;          // Leading uppercase

typedef struct {
    int x, y;
} MY_POINT;          // UPPER SNAKE CASE
```

C11规范并未明确规定哪种方式更好，并展示了所有大写和小写的示例。

K&R2更倾向于使用首字母大写，但也展示了一些大写和蛇形命名（带`_t`）的示例。

如果你正在使用风格指南，请坚持。如果没有，请查阅一个然后坚持。

## 数组和`typedef`

[i[`typedef`关键字-->数组]<]
语法有点奇怪，在我的经验中很少见到，但你可以`typedef`一个有一定数量项目的数组。

``` {.c}
// 将五个int类型的数组类型化为five_ints
typedef int five_ints[5];

five_ints x = {11, 22, 33, 44, 55};
```

我不太喜欢它，因为它隐藏了变量的数组特性，但这是可以做到的。
[i[`typedef`关键字-->数组]>]
[i[`typedef`关键字]>]
```