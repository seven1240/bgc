<!-- Beej的C语言指南

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 练习

这些是各个章节的练习。

**警告:** 如果你在尝试解决问题之前就查看答案，你只能获得 10%^[根据我的非科学估计。但是真的，先自己动手做再看答案是最好的学习方式。] 的潜在学习效果。在放弃并查看答案之前，请多花点时间思考。

## 简介 {#ex-intro}

1. 修改“Hello, world!”示例，打印你的名字。[flsol[intro/hello_name.c]]

1. 修改“Hello, world!”示例，将你的名字和你喜欢的水果分别打印在一行。[flsol[intro/hello_fruit.c]]

## 变量和语句 {#ex-var-stat}

1. $\pi$ 的一个近似值是 $208341 / 66317$。编写一个程序计算这个值，将结果存储在一个 `float` 类型的变量中，并将其打印出来。提示: 在至少一个数字后面加上 `.0`，强制计算为 `float` 而不是整数。[flsol[varstat/pi_approx.c]]

1. 在心中选择一个介于 50 和 97 之间的随机整数。称它为 $x$。计算 $(x+10\times x-5)/2$ 作为整数并打印出来。[flsol[varstat/formula1.c]]

1. 做和上一个问题一样的事情，但是将其除以 `2.0` 而不是 `2`，并将结果存储在一个 `float` 类型的变量中。然后打印出来。[flsol[varstat/formula2.c]]

1. 假设你有一个变量 `i = 2`。编写一个程序，将 `j = i + 7`（得到 `9`）赋给 `j`，同时在同一行的代码中增加 `i`。[flsol[varstat/postinc.c]]

1. 假设你有一个变量 `i = 2`。编写一个程序，在同一行的代码中将 `i` 减小并赋值 `j = i + 7`（得到 `8`）。[flsol[varstat/predec.c]]

1. 为整型变量 `i` 选择一个值。编写一个程序，如果 `i` 大于17，则打印 `"foo"`, 否则打印 `"bar"`。

1. 如果你在上一个问题中使用了 `if`-`else`，请使用三元运算符做同样的事情。否则，做相反的操作。

1. 使用 while 循环来确定对数字 $20754371$ 进行整数除以 $7$ ，直到结果为零为止。

1. 使用单个带有两个变量的 `for` 循环来打印:
   ```
   0 0
   1 1
   2 3
   3 6
   4 10
   5 15
   ```
   其中第二个数字是前一个数字加上第一个数字的值。在左列打印到99。`j` 的最后一个值将为 `4950`。

## 函数

1. 编写一个函数，有两个整型参数 `x` 和 `y`，计算

   $x\times20+x\times y$

   并返回结果。

1. 编写一个函数，通过 Gregory 和 Leibniz 无限级数计算 $\pi$，直到指定次数的项（第一项为 $1/1$（也就是 $1$））：

   $\displaystyle\pi=4(\frac{1}{1}-\frac{1}{3}+\frac{1}{5}-\frac{1}{7}+\frac{1}{9}-\cdots)$

   打印出第 1 到第 5000 项的级数结果。第 5000 次迭代应产生 $3.141397$。

1. 编写一个函数，它接受一个指向 `int` 和一个指向 `float` 的指针作为参数。它应该将 `int` 加到 `float` 上，并从 `int` 中减去 `float`。它应该看起来好像这两者同时发生，例如输入 `5` 和 `3.2` 应该得到 `1` 和 `8.2`。

   由于通过指针操作值，结果应该在调用者处可见。

1. 编写一个函数，它接受两个 `int*` 参数并交换它们。

   ``` {.c}
   int a = 10, b = 20;

   swap(&a, &b);

   printf("%d, %d\n", a, b); // 应该输出 20, 10
   ```

## 数组

1. 声明并初始化一个包含以下值的 `int` 数组：

   ``` {.default}
   10, 5, 2, 30, 97, 64
   ```

   编写一个接受数组和其长度作为参数的函数 `sum()`。使函数返回数组中元素的总和（为 `208`）。

1. 声明一个包含 2048 个元素的数组，并使用初始化器将索引为 `312` 的元素设置为 `3490`。所有其他元素应初始化为零。

1. 编写一个函数，检查一个[井字棋](Tic-tac-toe)棋盘是否有获胜者。$3\times3$ 棋盘上的 `0` 表示尚无人在那里移动。`1` 表示 "X" 已在那里移动，`2` 表示 "O" 已在那里移动。分别返回 `0`、`1` 或 `2`，表示无胜者、"X" 赢了或者 "O" 赢了。

## 字符串

1. 编写一个函数，反转一个字符串并原地修改。使其返回传入的相同指针。

1. 编写一个函数，如果一个字符串完全是大写字母则返回 true。提示：查看 `isupper()` 函数。

1. 编写一个函数，返回字符串中第一个字母的指针。如果字符不在字符串中，则返回 `NULL`。

## 结构体

1. 编写一个名为 `struct point` 的结构体，拥有两个名为 `x` 和 `y` 的 `double` 成员，用于存储二维点。编写一个函数，将两个 `struct point` 相加，通过相加它们各自的分量。函数应当返回一个 `struct point` 作为结果。

2. 做同样的事情，只是将 `struct point` 的指针传递到函数中。函数应当修改第一个点，使其成为第一个点和第二个点之和。

## 输入/输出

1. 编写一个程序，从 `stdin` 中最多读取50行，每行最多80个字符，然后以相反顺序打印它们。提示：二维字符数组可能会有所帮助。

2. 编写一个程序，从 [链接为 `input.txt` 的位置](io/input.txt) 中读取三维坐标，并计算三维点的平均值，将其打印出来。平均点是输入的 X、Y、Z 的平均值。对于 `input.txt`：

   ```
   平均点为 <46.296997,48.968998,49.845001>
   ```

## Typedef

1. 编写一个 `typedef`，为此 `struct` 创建一个名为 `vector3d` 的新类型：

   ``` {.c}
   struct {
      float x, y, z;
   };
   ```

   编写一个名为 `dot3d` 的函数，计算并返回两个 `vector3d` 的矢量点积。此函数应接受两个指向 `vector3d` 的指针作为参数，因此可以这样调用：

   ``` {.c}
   float result = dot3d(&v1, &v2);
   ```

两个向量的点积是它们的`x`、`y`和`z`分量的乘积之和：

$v_1=(x_1,y_1,z_1)$\
$v_2=(x_2,y_2,z_2)$

$v_1\cdot v_2 = x_1\times x_2 + y_1\times y_2 + z_1\times z_2$

指针 II

1.编写`my_strchr()`函数，该函数返回字符串中第一个字符的指针，类似于`strchr()`。使用指针算术来实现这个功能。
[flsol[pointers2/my_strchr.c]]

2.编写`my_strrchr()`函数，该函数返回字符串中最后一个字符的指针，类似于`strrchr()`。使用指针算术来实现这个功能。
[flsol[pointers2/my_strrchr.c]]

3.编写一个函数`object_sum()`，计算`void*`参数指向的对象的字节之和。

``` {.c}
int object_sum(void *p, size_t count)
```

``` {.c}
int t;
float x = 3.14159;
int y = 3490;

t = object_sum(&x, sizeof x);
t = object_sum(&y, sizeof y);
```

[flsol[pointers2/object_sum.c]]

4.编写一个比较函数，使用`qsort()`按升序对`int`数组进行排序。再编写另一个比较函数，按降序排序它。
[flsol[pointers2/int_qsort.c]]

手动内存分配

1.编写一个程序，为一个包含10个`int`的数组分配空间，然后用索引号乘以10来填充该数组。
[flsol[manmem/tenints.c]]

2.编写一个程序，从文件中读取任意大小的数据到内存，无需提前知道文件大小，并返回数据的指针。`fread()`函数可能有所帮助。
[flsol[manmem/readfile.c]]

## 作用域

1.这段代码会打印出什么内容？

``` {.c .numberLines}
#include <stdio.h>

int total = 0;

```c
   void add(int x)
   {
       total += x;
       printf("Total is now %d\n", total);
   }

   int main(void)
   {
       for (int i = 0; i < 3; i++)
           add(i);
   }
```

答案：`Total is now` 分别为 `1`, `3`, `5`。`total` 是文件作用域，因此每次调用 `add()` 都会受到影响。

1. 这段代码会打印什么？

```c
   #include <stdio.h>

   int main(void)
   {
       int i = 0;

       if (1 < 2) {  // 总是为真
           int i = 0;
        
           i += 5;
       }

       i += 5;

       printf("%d\n", i);
   }
```

答案：`5` - `i` 的第一次递增作用于隐藏在外部作用域中的变量。

2. 这段代码会打印什么？

```c
   #include <stdio.h>

   int main(void)
   {
       for (int i = 0; i < 3; i++) {
           for (int i = 0; i < 2; i++) {
               printf("Hi there!\n");
           }
       }
   }
```

答案：会打印六次 `Hi there`。内部声明的 `i` 覆盖了外部声明的 `i`。

## 类型 II

1. 编写一个程序，将一个 `unsigned char` 初始化为 `0`。然后在循环中，将该值递增总计 260 次，并每次打印出来。

当超过 255 会发生什么^[假设您的计算机使用的是 8 位字节。]？

答案：从 255 回绕到 0，就像里程表一样。这是因为 255 的二进制表示为 `11111111`---所有 8 位都是 `1`，因此再加一变成 `100000000`。但这是 9 位，太大了无法存储在一个字节中。最高位被丢弃，留下的是二进制 `00000000`。所有无符号类型在从其最大值递增时都会这样。].  [flsol[types2/overflow.c]]

## 类型 III
```

1. 编写一个函数，该函数打印浮点数的整数部分，后跟一个`+`符号，再跟着数字的小数部分。例如，给定`float`值`3.14159`，应该打印：

   ``` {.default}
   3+0.14159
   ```

   尝试在不使用任何字符串操作的情况下实现这一点，仅使用数值计算。

   [flsol[types3/cutfrac.c]]

2. 编写自己的函数，将一个字符串转换为整数，而不使用任何内置转换函数（如`atoi()`或`strtol()`）。

   ``` {.c}
   int my_atoi(char *s)
   {
        // 待办事项
   }

   printf("%d\n", my_atoi("3490"));  // 打印 3490
   ````

   额外加分项目：

   * 使其跳过前导空格---参见`isspace()`函数
   * 使其在遇到第一个非数字字符时停止转换---参见`isdigit()`函数
   * 使其处理负数

   [flsol[types3/my_atoi.c]]

## 类型 IV

1. 这将是一个由两个文件组成的项目。一个文件包含将数字添加到累加的功能，另一个文件调用该功能。
   
   主文件`runner.c`将包含以下内容：

   ``` {.c .numberLines}
   // runner.c

   #include <stdio.h>

   int main(void)
   {
      for (int i = 0; i < 10; i++)
         add(i);

      printf("Total is %d over %d calls\n", total(), count);
   }
   ```

   文件`total.c`将包含三个可从其他源文件调用的内容：

   1. 一个名为`void add(int x)`的函数。这将增加从`0`开始的累加总数。
   2. 一个名为`int total(void)`的函数，用于返回到目前为止的总数。
   3. 一个名为`count`的共享全局变量，用于返回调用`add()`的次数。

在 `total.c` 文件中实现功能，然后修改 `runner.c` 文件，使其可以调用函数 `add()` 和 `total()` 并可以看到变量 `count`。

确保 `total()` 是 `runner.c` 可以获取总数的唯一方式。

您将需要使用 `static` 和 `extern` 来解决这个问题。

通常情况下，这是我们会使用头文件的情况，但我们还没有讨论过头文件。

您可以通过在命令行上指定它们来一起编译两个文件：

``` {.default}
gcc -Wall -Wextra  -o runner runner.c total.c
```

## 多文件项目

1. 修改上面 Types IV 的练习，使用头文件 `total.h`。

确保在头文件上使用 `#ifdef` 包裹器。

如果有条件编译 C 文件为目标文件然后再将它们链接在一起，将会得到额外的加分。

[flsol[multifile/runner.c]] [flsol[multifile/total.c]] [flsol[multifile/total.h]]

## 外部环境

1. 编写一个程序，打印出在命令行中指定的环境变量的值。

如果该变量没有被设置，应该打印出消息 `no such variable`。

例如：

``` {.default}
$ ./printenv HOME LANG FOOBAR PATH
```
``` {.default}
HOME=/Users/beej
LANG=en_US.UTF-8
FOOBAR: no such variable
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

（`HOME`、`LANG` 和 `PATH` 是shell通常设置的变量，所以它们很可能已经存在于您的系统中。）

您将需要使用 `getenv()` 来完成这个任务。

[flsol[env/printenv.c]]

## C预处理器

1. 编写一个宏，接受两个参数，将它们相乘，然后将当前源代码行号加到乘积中。宏的结果应该是整数类型。

   [flsol[cpp/macro2.c]]

2. 编写代码，根据当前的C版本输出`C89`、`C99`或`C11或更高版本`。

   [flsol[cpp/cversion.c]]

3. 让程序使用`#define`为`X`定义一个数字值。使用条件编译，如果条件为真，则打印`X is greater than 30`，否则打印从`0`到`X-1`的值。

   [flsol[cpp/condcomp.c]]