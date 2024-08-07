``` {.c .numberLines}
#include <stdio.h>

int plus_one(int n)  // 函数的“定义”
{
    return n + 1;
}
```  

`int` 在 `plus_one` 前表示返回类型。

`int n` 表示该函数接受一个 `int` 参数，保存在名为“参数” `n` 的特殊局部变量中。参数是一种特殊类型的本地变量，其中将复制参数。

我要强调这一点，这里的参数复制了参数的值。如果你知道参数是参数值的_复制品_，很多 C 语言中的事情会更容易理解。稍后会详细介绍。

将程序分析到 `main()`，我们可以看到调用函数的地方，在这里我们将返回值赋给局部变量 `j`：

``` {.c .numberLines startFrom="8"}
int main(void)
{
    int i = 10, j;
    
    j = plus_one(i);  // 调用

    printf("i + 1 is %d\n", j);
}
```

在我忘记之前，请注意我在使用函数之前已经定义了它。
如果我没有这样做，编译器在编译`main()`时还不知道这个函数，它会产生一个未知函数调用的错误。有一种更合适的方式去编写上面的代码，可以使用_函数原型_，但我们稍后会讨论这个。

还要注意，`main()`[i[`main()` 函数]]是一个函数！

它返回一个`int`。

但是这里的`void`[i[`void` 类型]]是什么？这是一个关键字，用于表示函数不接受任何参数。

您还可以返回`void`来表明函数不返回值：

```c
#include <stdio.h>

// 这个函数不接受任何参数，也不返回任何值：

void hello(void)
{
    printf("Hello, world!\n");
}

int main(void)
{
    hello();  // 输出 "Hello, world!"
}
```

## 传值调用 {#passvalue}

[i[按值传递]()]我之前提到过，当你将参数传递给一个函数时，会产生参数的一个副本，并存储在相应的参数中。

如果参数是一个变量，会产生该变量的值的副本并存储在参数中。

更一般地，整个参数表达式会被计算并确定其值，然后将该值复制到参数中。

无论如何，参数中的值是独立的。它与您在调用函数时所使用的参数值或变量无关。

让我们来看一个例子。仔细研究它，看看能否在运行代码之前确定输出：

```c
#include <stdio.h>

void increment(int a)
{
    a++;
}

int main(void)
{
    int i = 10;

    increment(i);

    printf("i == %d\n", i);  // 这将打印什么？
}
```

乍一看，看起来 `i` 是 `10`，然后我们将其传递给函数 `increment()`。 在那里，值被递增，所以当我们打印它时，它应该是 `11`，对吗？

> "习惯失望吧。"  
> --- Dread Pirate Roberts，《公主新娘》

但它不是 `11` --- 它打印的是 `10`！怎么回事？

这一切都取决于你传递给函数的表达式会被 _复制_ 到相应的参数上。该参数是一个副本，而不是原始值。

所以在 `main()` 中，`i` 是 `10`。然后我们将其传递给 `increment()`。在那个函数中，对应的参数被称为 `a`。

这种复制就像是一种赋值。泛泛地说，`a = i`。所以在那一点上，`a` 是 `10`。而在 `main()` 中，`i` 也是 `10`。

然后我们将 `a` 递增为 `11`。但我们根本没有改变 `i`！它依然是 `10`。

最后，函数执行完毕。所有局部变量都被丢弃（再见，`a`！），我们返回到 `main()`，此时 `i` 仍然是 `10`。

然后我们打印它，得到 `10`，任务完成。

这就是为什么在之前的例子中，使用 `plus_one()` 函数，我们`返回`了本地修改的值，这样我们才能在 `main()` 中看到它。

看起来有点受限制，对吧？好像你只能从函数中获取一个数据，这是你的想法。然而，还有一种获取数据的方法；C 程序员称之为 _传引用_，这是一种我们以后会讲述的故事。

但没有花哨的名字会让你忽视这个事实：_你向函数传递的一切参数，_毫无例外_，都会被复制到相应的参数中，并且函数将在该本地副本上操作，_不管怎样_。请记住这一点，即使我们在谈论所谓的传引用时。

所以，如果你回忆一下前面几节讨论的冰河时代，我提到过在使用函数之前必须先定义它，否则编译器事先不知道该函数的存在，会报错。

这并不完全准确。你可以提前告诉编译器，你将会使用某种类型的函数，带有特定的参数列表。这样函数就可以在任何地方被定义（甚至是在另一个文件中），只要在调用该函数之前已经声明了该函数的_函数原型_。

幸运的是，函数原型非常简单。它只是函数定义的第一行的简单复制，结尾加上一个分号以示完整。例如，以下代码调用稍后定义的函数，因为之前已经声明了一个原型：

``` {.c .numberLines}
#include <stdio.h>

int foo(void);  // 这是原型！

int main(void)
{
    int i;
    
    // 我们可以在这里调用 foo()，即使它的定义在后面，因为
    // 关键是原型已经被声明了！

    i = foo();
    
    printf("%d\n", i);  // 3490
}

int foo(void)  // 这是定义，与原型相同！
{
    return 3490;
}
```

如果你在使用函数之前没有声明它（不是用原型也不是定义），那么你就在执行一种称为_隐式声明_的操作。这在第一个 C 标准（C89）中是允许的，并且那个标准有关于它的规则，但现在已经不被允许了。而且在新代码中依赖它没有正当理由。

我们可能会注意到我们一直在使用的示例代码中有一个情况...那就是，我们一直在使用老旧的 `printf()` 函数，没有定义或声明原型！我们是如何逃脱这种无序状态的呢？实际上并没有逃脱。有一个原型；就在我们用 `#include` 包含的那个头文件 `stdio.h` 中，还记得吗？所以我们还是合法的，警官！

## 空参数列表

有时候你可能会在旧代码中看到这样的写法，但在新代码中你不应该再这样编码。应该始终使用 `void` 来表示函数不带参数。在现代代码中永远^【永远不要说“永远”】不要忽略这一点。

如果你很容易记住在函数和原型中使用 `void` 来表示空参数列表，你可以跳过这部分的其余内容。

此问题有两个背景：

- 在函数定义中省略所有参数
- 在原型中省略所有参数

首先让我们看一个潜在的函数定义：

``` {.c}
void foo()  // 这里应该真的有一个 `void`
{
    printf("Hello, world!\n");
}
```

虽然规范规定在这种情况下的行为_好像_你指定了 `void`（C11 §6.7.6.3¶14），但 `void` 类型有其存在的理由。要使用它。

但在函数原型的情况下，使用 `void`[i[`void` 类型-->在函数原型中]]和不使用有一个_明显_的区别：

``` {.c}
void foo();
void foo(void);  // 不同！
```

在原型中省略 `void` 表示向编译器表明关于函数参数没有其他信息。这实际上关闭了所有的类型检查。

为原型**一定**使用 `void` 当你有空的参数列表。