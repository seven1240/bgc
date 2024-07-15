``` {.c .numberLines}
#include <stdio.h>
#include <setjmp.h>

jmp_buf env;

void depth2(void)
{
    printf("Entering depth 2\n");
    longjmp(env, 3490);           // Bail out
    printf("Leaving depth 2\n");  // This won't happen
}

void depth1(void)
{
    printf("Entering depth 1\n");
    depth2();
    printf("Leaving depth 1\n");  // This won't happen
}
```

```c
int main(void)
{
    switch (setjmp(env)) {
      case 0:
          printf("Calling into functions, setjmp() returned 0\n");
          depth1();
          printf("Returned from functions\n");  // This won't happen
          break;

      case 3490:
          printf("Bailed back to main, setjmp() returned 3490\n");
          break;
    }
}
```

在运行时，输出为:

``` {.default}
Calling into functions, setjmp() returned 0
Entering depth 1
Entering depth 2
Bailed back to main, setjmp() returned 3490
```

如果尝试将该输出与代码进行匹配，就能明显看出有一些非常独特的事情正在发生。

其中一个最引人注目的是，`setjmp()` 返回**两次**。这到底是什么操作？

其实情况是这样的：如果 `setjmp()` 返回 `0`，这意味着你已经成功地在那一点设置了"书签"。

如果它返回非零值，这表示你刚刚回到了之前设置的"书签"。(而返回值就是你传递给`longjmp()`的值。)

这样你就可以区分设置"书签"和以后返回到它之间的区别。

因此，当上述代码第一次调用`setjmp()`时，`setjmp()`会将状态存储在`env`变量中并返回`0`。之后，当我们使用相同的`env`调用`longjmp()`时，它会恢复状态并返回`longjmp()`传递的值。

[i[`setjmp()`函数]>]
[i[`longjmp()`函数]>]

## 陷阱

```c
// Under the hood, this is pretty straightforward. Typically the _stack
// pointer_ keeps track of the locations in memory that local variables are
// stored, and the _program counter_ keeps track of the address of the
// currently-executing instruction^[Both "stack pointer" and "program
// counter" are related to the underlying architecture and C
// implementation, and are not part of the spec.].

// So if we want to jump back to an earlier function, it's basically only a
// matter of restoring the stack pointer and program counter to the values
// kept in the [i[`jmp_buf` type]] `jmp_buf` variable, and making sure the
// return value is set correctly. And then execution will resume there.

// But a variety of factors confound this, making a significant number of
// undefined behavior traps.

// ### The Values of Local Variables

// [i[`setjmp()` function]<]

// If you want the values of automatic (non-`static` and non-`extern`)
// local variables to persist in the function that called `setjmp()` after
// a [i[`longjmp()`]] `longjmp()` happens, you must declare those variables
// to be [i[`volatile` type qualifier-->with `setjmp()`]<] `volatile`.

// Technically, they only have to be `volatile` if they change between the
// time `setjmp()` is called and `longjmp()` is called^[The rationale here
// is that the program might store a value temporarily in a _CPU register_
// while it's doing work on it. In that timeframe, the register holds the
// correct value, and the value on the stack might be out of date. Then
// later the register values would get overwritten and the changes to the
// variable lost.].

// For example, if we run this code:

int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

`longjmp()` back later, the value of `x` will be indeterminate.

If we want to fix this, `x` must be `volatile`:
```

``` {.c}
volatile int x = 20;

if (setjmp(env) == 0) {
    x = 30;
}
```

[p[`setjmp()`函数]>]

[p[`volatile`类型限定符-->与`setjmp()`一起使用]>]

现在在[i[`longjmp()`]]后值将变为正确的`30`。
`longjmp()`会把我们带回到这一点。

### 保存了多少状态？

[p[`setjmp()`函数]<]
[p[`longjmp()`函数]<]

当你`longjmp()`时，执行将恢复到对应`setjmp()`的点。就是这样。

[p[`setjmp()`函数]>]

规范指出，就好像你跳回到该函数的那一点，局部变量保持在`longjmp()`调用时的值.

[p[`longjmp()`函数]>]

不会恢复的东西包括（引用规范的话）：

* 浮点状态标志
* 打开的文件
* 抽象机器的任何其他组件

### 不能将任何东西命名为`setjmp`

不能有任何名为`setjmp`的`extern`标识符。或者，如果`setjmp`是一个宏，你不能取消定义它。

这两者都是未定义行为。

### 不能在较大的表达式中使用`setjmp()`

[p[`setjmp()`-->在表达式中]<]

也就是说，你不能这样做：

``` {.c}
if (x == 12 && setjmp(env) == 0) { ... }
```

由于必须在展开堆栈时进行的机制，规范不允许这种太过复杂的操作。
我们不能`longjmp()`回到仅部分执行的复杂表达式中。

因此，对该表达式的复杂度有一定限制。

* 它可以是条件控制表达式的整体。

  ``` {.c}
  if (setjmp(env)) {...}
  ```

  ``` {.c}
  switch (setjmp(env)) {...}
  ```

``` {.c}
// 在条件语句中可以作为关系运算或相等运算的一部分，只要另一个操作数是整数常量。整个表达式是条件语句的控制表达式。
``` 

``` {.c}
// 作为逻辑非 (`!`) 运算的操作数，整个表达式是控制表达式。
``` 

``` {.c}
// 作为独立表达式，可能需要转换为 `void`。
setjmp(env);
```
``` {.c}
(void)setjmp(env);
```

### 何时不能使用 `longjmp()`？

```c
// 如果出现以下情况，则属于未定义行为：
```

```c
// 之前没有调用过 `setjmp()`
// 从另一个线程调用 `setjmp()`
// 在变长数组（VLA）的作用域内调用了 `setjmp()`，并且在调用 `longjmp()` 之前退出了该 VLA 的作用域。
// 包含 `setjmp()` 的函数在调用 `longjmp()` 之前已经返回。
```

```c
// 对于最后一种情况，“返回”包括函数的正常返回，以及如果另一个 `longjmp()` 跳回到调用栈中比较“早”的位置，早于所讨论的函数。
```

### 不能向 `longjmp()` 传递 `0`

```c
// 如果尝试向 `longjmp()` 传递值 `0`，它会悄悄地将该值更改为 `1`。
```

```c
// 由于 `setjmp()` 最终返回这个值，而且 `setjmp()` 返回 `0` 有特殊含义，所以禁止返回 `0`。
```

### `longjmp()` 和变长数组

```c
// 如果在变长数组的作用域内并且通过 `longjmp()` 跳出该作用域，那么为变长数组分配的内存可能会泄漏。
```

```c
// 如果通过 `longjmp()` 返回到之前仍处于作用域内的具有变长数组的任何早期函数，同样会发生内存泄漏。
```

```c
// 这是关于VLA的一件事情真的让我很烦恼---你可以编写完全合法的C代码，但却浪费了内存。但是，嘿---我不负责规范。

[i[`lonjmp()`]>]
[i[长跳转]>]
```