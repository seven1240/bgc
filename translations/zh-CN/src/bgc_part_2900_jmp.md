<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 使用 `setjmp` 和 `longjmp` 进行长跳转 {#setjmp-longjmp}

[长跳转]<

我们已经看到了`goto`，它在函数范围内进行跳转。但是`longjmp()`允许您跳回到执行过程中的早期点，回到调用此函数的函数的位置。

这个功能有很多限制和注意事项，但对于从调用堆栈的深处返回到较早状态时非常有用。

在我看来，这种功能极少被使用。

## 使用 `setjmp` 和 `longjmp`

[`setjmp()` 函数]<
[`longjmp()` 函数]<

我们要做的操作是使用`setjmp()`在执行过程中设置一个书签。稍后，我们将调用`longjmp()`，它会跳回到我们用`setjmp()`设置书签的早期执行点。

而且，即使您调用了子函数，它也可以做到这一点。

这里有一个快速演示，我们会在函数中调用几层深度，然后跳出它。

我们将使用一个文件作用域变量`env`来保存我们调用`setjmp()`时的状态，以便稍后在调用`longjmp()`时将它们恢复。这是我们记住“位置”的变量。

变量`env`的类型是`jmp_buf`，这是在`<setjmp.h>`中声明的不透明类型。

``` {.c .numberLines}
#include <stdio.h>
#include <setjmp.h>

jmp_buf env;

void depth2(void)
{
    printf("进入深度 2\n");
    longjmp(env, 3490);           // 跳出
    printf("离开深度 2\n");       // 不会发生
}

void depth1(void)
{
    printf("进入深度 1\n");
    depth2();
    printf("离开深度 1\n");       // 不会发生
}
```

``` {.default}
在调用函数中，setjmp() 返回 0
返回至主函数，setjmp() 返回 3490
```

在内部，这相当直截了当。通常，_堆栈指针_ 负责跟踪存储局部变量的内存位置，_程序计数器_ 则负责跟踪当前执行指令的地址^["堆栈指针"和"程序计数器"与底层架构和C实现相关，不是规范的一部分]。

因此，如果我们想跳回到之前的函数，基本上只需将堆栈指针和程序计数器恢复为保留在 [i[`jmp_buf` type]] `jmp_buf` 变量中的值，并确保返回值设置正确。然后执行将在那里恢复。

但有各种因素使这变得更加复杂，导致了大量未定义行为陷阱。

### 局部变量的值

[i[`setjmp()` 函数]<]

如果希望在调用 `setjmp()` 的函数中，当发生 [i[`longjmp()`]] `longjmp()` 后，自动（非`static`和非`extern`）局部变量的值保持不变，必须将这些变量声明为 [i[`volatile` 类型限定符-->与 `setjmp()` 一起使用]<] `volatile`。

从技术上讲，它们只需要在 `setjmp()` 调用和 `longjmp()` 调用之间发生改变时才需要是 `volatile`^[这里的理由是程序可能在处理变量时在 _CPU 寄存器_ 中临时存储一个值。在这段时间内，寄存器保存正确的值，而堆栈上的值可能已过时。然后稍后寄存器的值将被覆盖，该变量的更改将丢失]。

例如，如果运行以下代码：

``` {.c}
int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

然后稍后返回`longjmp()`，`x` 的值将无法确定。

如果要修复此问题，`x` 必须是 `volatile`：

``` {.c}
volatile int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

现在在[i[`longjmp()`]]之后，值将正确变为`30`。
`longjmp()`会使程序返回到此行。

### 保存了多少状态？

当你使用`longjmp()`时，执行会从对应的`setjmp()`处恢复。就是这样。

[i[`setjmp()`]<]
[i[`longjmp()`]<]

规范指出，这就好像你跳回函数的那一点，局部变量的值会回到`longjmp()`调用时的值。

[i[`setjmp()`]>]

不会复原的包括，引用规范的说法：

* 浮点状态标志
* 打开的文件
* 抽象机器的其他任何组件

### 不能把任何东西命名为`setjmp`

你不能用名称`setjmp`来定义任何`extern`标识符。或者，如果`setjmp`是一个宏，你也不能取消宏定义。

这两种行为都是未定义的。

### 不能在更大的表达式中使用`setjmp()`

[i[`setjmp()`-->在表达式中]<]

也就是说，你不能这样做：

``` {.c}
if (x == 12 && setjmp(env) == 0) { ... }
```

这对于规范来说太复杂了，因为在展开堆栈等操作时会遇到很多问题。我们不能`longjmp()`回到一个部分执行过的复杂表达式中。

所以对该表达式的复杂度有着限制。

* 它可以是条件控制表达式的整个部分。

  ``` {.c}
  if (setjmp(env)) {...}
  ```

  ``` {.c}
  switch (setjmp(env)) {...}
  ```

* 可以作为关系或相等表达式的一部分，只要另一个操作数是整数常量。整个事情都是条件控制表达式。

  ``` {.c}
  if (setjmp(env) == 0) {...}
  ```

* 逻辑非(`!`)操作的操作数，即整个控制表达式。

  ``` {.c}
  if (!setjmp(env)) {...}
  ```

* 独立的表达式，可能转型为 `void`。

  ``` {.c}
  setjmp(env);
  ```
  ``` {.c}
  (void)setjmp(env);
  ```

### 什么情况下不能使用 `longjmp()`?

[i[`lonjmp()`]<]

如果出现以下情况，则属于未定义行为：

* 之前没有调用过 `setjmp()`
* 在另一个线程中调用了 `setjmp()`
* 在可变长度数组（VLA）的作用域内调用了 `setjmp()`，并且在调用 `longjmp()` 之前执行已经离开了该VLA的作用域
* 包含 `setjmp()` 的函数在调用 `longjmp()` 之前就退出了

关于最后一个情况，“退出”包括了函数的正常返回，以及如果另一个 `longjmp()` 跳回了调用栈中比该函数更早的位置。

### 不能将 `0` 传递给 `longjmp()`

如果试图将值 `0` 传递给 `longjmp()`，它会将该值悄悄地改为 `1`。

因为 `setjmp()` 最终返回这个值，并且 `setjmp()` 返回 `0` 有特殊意义，所以禁止返回 `0`。

### `longjmp()` 和可变长度数组

如果在VLA的作用域内使用 `longjmp()` 跳出，那么分配给VLA的内存可能会泄露^[也就是说，直到程序以无法释放的方式结束，该内存将被保留]。

如果您通过 `longjmp()` 回到任何仍在作用域内具有 VLA 的较早函数，同样的事情会发生。

这是关于可变长度数组（VLAs）的一件让我十分困扰的事情——你可以编写完全合法的C代码，却浪费了内存。但是，嘿——我不负责规范。