```c
#include <stdio.h>

int main() {
    int num = 10; // Define an integer variable named 'num' and assign a value of 10
    int* ptr = &num; // Define a pointer variable named 'ptr' and assign the address of 'num' to it

    printf("The value of num is: %d\n", num); // Print the value of 'num'
    printf("The address of num is: %p\n", (void*)&num); // Print the address of 'num'
    printf("The value of num using a pointer is: %d\n", *ptr); // Print the value of 'num' using the pointer

    return 0;
}
```

// Translate the comments in this C code block into Simplified Chinese.

现在，并不是所有的数据类型都只使用一个字节。例如，一个`int`通常是四个字节，`float`也是一样，但实际取决于系统。您可以使用`sizeof`运算符来确定某种类型使用多少字节的内存。

``` {.c}
// %zu是用于类型size_t的格式说明符

printf("一个int使用 %zu 字节的内存\n", sizeof(int));

// 这对我来说打印出"4"，但在不同系统中可能会有所不同。
```

> **内存有趣知识**：当您有一个使用超过一个字节的内存的数据类型（比如您典型的`int`）时，构成数据的字节在内存中总是相邻的。有时它们的顺序是您期望的顺序，有时则不是。虽然C语言不保证任何特定的内存顺序（它是依赖于平台的），但通常仍然可以编写与平台无关的代码，您甚至无需考虑这些令人讨厌的字节顺序问题。

所以，**总之**，如果我们能继续下去，为指针的定义做些鼓舞和一些预示性的音乐，_指针是一个保存地址的变量_。想象一下这一点2001太空奥德赛的古典乐谱。巴·巴姆·巴姆·巴姆·巴《嘭》！

嗯，也许有点夸张了，是吧？指针并没有太多神秘之处。它们就是数据的地址。就像一个`int`变量可以保存值`12`一样，一个指针变量可以保存数据的地址。

```c
这意味着所有这些东西都表示相同的意思，即表示内存中某个位置的数字：

* 索引到内存（如果你把内存看作一个大数组的话）
* 地址
* 位置

我会把它们混用。是的，我刚刚加上了“位置”，因为永远不会有太多意思相同的词。

指针变量保存着那个地址数字。就像一个 `float` 变量可能保存 `3.14159` 一样。

想象你有一堆用地址编号顺序标号的便利贴®。 （第一个标号为 `0`，下一个为 `1`，以此类推。）

除了数字代表它们位置外，你还可以在每张便利贴上写另一个你选择的数字。 它可以是你拥有的狗的数量。 或者火星周围的卫星数量...

...或者，_它可以是另一张便利贴的索引！_

如果你写下了你拥有的狗的数量，那就是一个普通变量。 但如果你在那里写下了另一张便利贴的索引，_那就是一个指针_。它指向另一张便利贴！

另一个类比可能是房子地址。 你可以有一个带有某些特质的房子，院子、金属屋顶、太阳能等。 或者你可以有那个房子的地址。 地址和房子本身并不相同。一个是完整的房子，另一个只是几行文字。 但房子的地址是指向那个房子的 _指针_。 它不是房子本身，但它告诉你在哪里找到它。

在计算机中，我们也可以用数据做同样的事情。 你可以有一个保存某个值的数据变量。 而那个值位于某个地址的内存中。 你可以有一个不同的 _指针变量_ 拥有那个数据变量的地址。

它不是数据变量本身，但就像房子地址一样，它告诉我们在哪里找到它。
```

当我们有了这个，我们说我们有一个指向该数据的"指针"。然后我们可以跟着这个指针访问数据本身。

(尽管看起来现在没什么用，但当与函数调用一起使用时，这一切都变得不可或缺。请耐心等到那一步。)

所以，如果我们有一个 `int`，然后我们想要一个指向它的指针，我们想要的是某种方法来获取那个 `int` 的地址，对吧？毕竟，指针只是保存着数据的_地址。你猜我们会用什么运算符来找到那个 `int` 的_地址？

[i[`&` 取地址运算符]]嗯，让我来告诉你一个令人震惊的秘密，温和的读者，我们使用`地址运算符`（恰好是一个&符号）来找到数据的地址。&符号。

所以举个快速的例子，我们引入一个新的_格式说明符_给`printf()`，这样你就可以打印一个指针。你已经知道`%d`会打印一个十进制整数，对吗？嗯，`%p`[i[printf()函数-->带指针]]会打印一个指针。现在，这个指针看起来可能是一个垃圾数字（而且可能是用十六进制^[即，基数为16，包含数字0、1、2、3、4、5、6、7、8、9、A、B、C、D、E和F。]而不是十进制)，但实际上它只是数据存储在内存中的索引。（或者是数据的首字节存储在内存中的索引，如果数据是多字节的。）在几乎所有情况下，包括这个，被打印的数字的实际值对你来说都不重要，我只是为了展示`地址运算符`才在这里展示它。

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 10;

    printf("The value of i is %d\n", i);
    printf("And its address is %p\n", (void *)&i);
```

```c
// %p 期望参数是一个指向void的指针
// 我们将其转换以让编译器满意。
```

在我的电脑上，这会打印出：

``` {.default}
i的值为10
它的地址是0x7ffddf7072a4
```
[i[`&`取址运算符]>]

如果你感兴趣，这个十六进制数字在十进制中是140,727,326,896,068（就像奶奶以前用的十进制）。这是变量`i`的数据存储的内存索引。这是`i`的地址。这是`i`的位置。这是`i`的指针。

它是一个指针，因为它告诉你在内存中`i`的位置。就像写在一张纸上的家庭地址告诉你去哪里找到特定的房子一样，这个数字告诉我们在内存中哪里可以找到`i`的值。它指向`i`。

再次强调，通常我们并不关心地址的具体数字是多少。我们只关心它是`i`的指针。

## 指针类型 {#pttypes}

[i[指针类型]<] 这个我们知道了。你现在可以成功地取一个变量的地址并将其打印到屏幕上。这对简历来说是一个小技巧，对吧？接下来你可以揪着我的衣领礼貌地问指针有什么用。

很好的问题，我们将在赞助商的广告后马上回答。

> `ACME ROBOTIC HOUSING UNIT CLEANING SERVICES. YOUR HOMESTEAD WILL BE
> DRAMATICALLY IMPROVED OR YOU WILL BE TERMINATED. MESSAGE ENDS.`

欢迎回到Beej's Guide的下一个章节。上次我们讨论了如何使用指针。现在我们要做的是将一个指针存储在一个变量中，以便以后使用。你可以通过在变量名和类型之间加上星号（`*`）来识别_指针类型_：
```

``` {.c .numberLines}
int main(void)
{
    int i;  // i的类型是"int"
    int *p; // p的类型是"指向int的指针"，或者"int指针"
}
```

嘿，这里有一个变量，它是一个指针类型，可以指向其他的`int`变量。也就是说，它可以保存其他`int`变量的地址。我们知道它指向`int`，因为它的类型是`int*`（读作"int指针"）。

当你将一个值赋给一个指针变量时，赋值符号右边的类型必须与指针变量的类型相同。幸运的是，当你取一个变量的地址时，结果的类型就是指向该变量类型的指针，所以如下的赋值是完美的：

``` {.c}
int i;
int *p;  // p是一个指针，但尚未初始化且指向垃圾值

p = &i;  // p被赋予i的地址--p现在"指向"i
```

在赋值号的左边，我们有一个类型为指向`int`的指针（`int*`），而在右边，我们有一个指向`int`的表达式，因为`i`是一个`int`（因为`int`的地址得到的是一个指向`int`的指针）。一个东西的地址可以存储在指向该东西的指针中。

懂了吗？我知道这还是有点难以理解，因为你还没有看到指针变量的真正用途，但我们正在小步快走，以免有人迷失方向。所以现在，让我们介绍你认识反引用操作符。这有点像在反派世界中`地址取值`会是什么感觉。

## 解引用 {#deref}

指针变量可以被视为通过指向它而“引用”另一个变量。在C语言中很少有人谈论“引用”或“引用”，但我提出来是为了让这个操作符的名称更容易理解一些。

当你有一个指向变量的指针（大致上是“对变量的引用”）时，可以通过对指针进行*解引用*来使用原始变量。（你可以把这看作是“去指针化”指针，但没有人会说“去指针化”）。

回到我们的类比，这有点像查看一个家庭地址然后去那栋房子。

现在，我所说的“访问原始变量”是什么意思呢？嗯，如果你有一个名为`i`的变量，并且有一个指向`i`的指针叫做`p`，你可以像使用原始变量`i`一样使用*解引用的指针*`p`！

[你几乎掌握足够的知识来处理例子了。你最后需要知道的一点是：什么是解引用操作符？实际上它被称为*间接操作符*，因为你是通过指针间接访问值。而且还是那个星号，再次出现：`*`。不要把这个与你之前在指针声明中使用的星号搞混了。它们是同一个字符，但在不同的上下文中有不同的含义^[这还不是全部！它也用在了`/*注释*/`、乘法运算以及在带有可变长度数组的函数原型中！它都是同一个`*`，但上下文赋予了它不同的含义]。

下面是一个完整的示例：

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i;
    int *p;  // 这不是一个解引用--这是一种类型“int*”

    p = &i;  // 现在p指向i，p持有i的地址

    i = 10;  // i现在是10
    *p = 20; // p指向的东西（即i！）现在是20！

    printf("i is %d\n", i);   // 打印"20"
    printf("i is %d\n", *p);  // “20”！解引用-p就等同于i！
}
```

请记住，`p` 存储的是 `i` 的地址，正如你在第 8 行对 `p` 赋值的地方所看到的。间接运算符的作用是告诉计算机**使用指针指向的对象**，而不是使用指针本身。通过这种方式，我们将 `*p` 转换为 `i` 的一种别名。

好了，但_为什么_？为什么要这样做呢？

## 作为参数传递指针 {#ptpass}

现在你可能会想，你对指针有很多了解，但实际应用却几乎为零，对吧？我的意思是，如果你可以直接使用 `i`，那么 `*p` 有什么用呢？

好吧，朋友，指针的真正力量在于当你开始将它们传递给函数时会显现出来。这有什么了不起的呢？也许你还记得以前可以将各种参数传递给函数，它们会被忠实地复制到参数中，然后你可以在函数内部操作这些变量的本地副本，最后你可以返回一个单一值。

如果你想从函数中返回多个数据呢？我的意思是，你只能返回一个东西，对吧？如果我用另一个问题来回答这个问题呢？...嗯，两个问题？

当你将指针作为参数传递给函数时会发生什么？指针的副本会被放入相应的参数中吗？_当然会。_记得我之前不厌其烦地说过**每个单一参数**都会被复制到参数中，函数会使用参数的一个副本。同样道理也适用于这里，函数将得到指针的一个副本。

```c
#include <stdio.h>

void increment(int *p)  // 请注意它接受一个指向int的指针
{
    *p = *p + 1;        // 给p指向的东西加一
}

int main(void)
{
    int i = 10;
    int *j = &i;  // 请注意取址运算符；将其转换为指向i的指针

    printf("i is %d\n", i);        // 输出 "10"
    printf("i is also %d\n", *j);  // 输出 "10"

    increment(j);                  // j是int*--指向i

    printf("i is %d\n", i);        // 输出 "11"！
}
```

好的！这里有几个要注意的地方... 最重要的是`increment()`函数接受一个`int*`作为参数。我们通过使用`取址运算符`将`int`变量`i`改为`int*`，在调用中传递了一个`int*`给它。(记住，指针保存地址，我们通过运行它们通过`取址运算符`来创建变量的指针。)

```c
// The `increment()` function gets a copy of the pointer.
// Both the original pointer `j` (in `main()`) and the copy of that pointer `p` (the parameter in `increment()`) point to the same address, namely the one holding the value `i`. (Again, by analogy, like two pieces of paper with the same home address written on them.) Dereferencing either will allow you to modify the original variable `i`! The function can modify a variable in another scope! Rock on!

// The above example is often more concisely written in the call just by using address-of right in the argument list:
printf("i is %d\n", i);  // prints "10"
increment(&i);
printf("i is %d\n", i);  // prints "11"!

// As a general rule, if you want the function to modify the thing that you're passing in such that you see the result, you'll have to pass a pointer to that thing.

## The `NULL` Pointer

// Any pointer variable of any pointer type can be set to a special value called `NULL`. This indicates that this pointer doesn't point to anything.
int *p;
p = NULL;

// Since it doesn't point to a value, dereferencing it is undefined behavior, and probably will result in a crash:
int *p = NULL;
*p = 12;  // CRASH or SOMETHING PROBABLY BAD. BEST AVOIDED.

// Despite being called [flw[the billion dollar mistake by its creator|Null_pointer#History]], the `NULL` pointer is a good [flw[sentinel value|Sentinel_value]] and a general indicator that a pointer hasn't yet been initialized.
// (Of course, like other variables, the pointer points to garbage unless you explicitly assign it to point to an address or `NULL`.)

## A Note on Declaring Pointers

// The syntax for declaring a pointer might seem a bit strange. Let's look at this example:
```

``` {.c}
int a;
int b;
```

我们可以将其缩减为一行，对吧？

``` {.c}
int a, b;  // 同样的意思
```

所以 `a` 和 `b` 都是 `int`。没问题。

但是这个呢？

``` {.c}
int a;
int *p;
```

我们能将其写在一行吗？可以的。但是`*`应该放在哪里呢？

规则是，`*`应该放在任何指针类型的变量前面。也就是说，在这个例子中，`*` 不是 `int` 的一部分，而是变量 `p` 的一部分。

考虑到这一点，我们可以写成这样：

``` {.c}
int a, *p;  // 同样的意思
```

值得注意的是，下面的代码行并不声明两个指针：

``` {.c}
int *p, q;  // p 是指向 int 的指针；q 只是一个 int。
```

如果程序员写下以下（有效）的代码行，可能会看起来特别隐蔽，但其功能与上面的代码行相同。

``` {.c}
int* p, q;  // p 是指向 int 的指针；q 只是一个 int。
```

现在看看这个示例，并确定哪些变量是指针，哪些不是：

``` {.c}
int *a, b, c, *d, e, *f, g, h, *i;
```

我会在脚注中放上答案^[指针类型的变量是 `a`、`d`、`f` 和 `i`，因为它们在前面有 `*` 符号。]。

## `sizeof` 和 指针

只有一点语法可能会让人困惑，你可能会不时看到。

记住 `sizeof` 是基于表达式的_类型_。

``` {.c}
int *p;

// 打印 'int' 的大小
printf("%zu\n", sizeof(int));

// p 是类型 'int *'，所以打印的是 'int*' 的大小
printf("%zu\n", sizeof p);

// *p 是类型 'int'，所以打印的是 'int' 的大小
printf("%zu\n", sizeof *p);
```

你可能在代码中看到最后面有`sizeof`。只要记住，`sizeof`关注的是表达式的类型，而不是表达式本身中的变量。