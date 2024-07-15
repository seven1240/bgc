<!-- Beej's guide to C

# vim: ts=4:sw=4:nosi:et:tw=72
-->

# 练习

这些是各章节的练习。

**警告：**如果在尝试解答之前查看解决方案，你只能获得 10%^[根据我的非科学估计。但说真的，先自己动手做，然后再看答案是最好的学习方式。] 潜在学习成果的 10%。尽量多花些时间尝试解答，不要轻易放弃查看答案。

## 简介 {#ex-intro}

1. 修改 "Hello, world!" 示例，打印你的名字。[flsol[intro/hello_name.c]]

1. 修改 "Hello, world!" 示例，将你的名字打印在一行上，下一行是你最喜欢的水果。[flsol[intro/hello_fruit.c]]

## 变量和语句 {#ex-var-stat}

1. $\pi$ 的近似值是 $208341 / 66317$。编写一个计算这个值的程序，将结果存储在一个 `float` 变量中，并打印出来。提示：在至少一个数字后面加上 `.0`，以强制计算为 `float` 而不是整数。[flsol[varstat/pi_approx.c]]

1. 在脑海中选择一个介于 50 和 97 之间的随机整数。称其为 $x$。计算 $(x+10\times x-5)/2$ 的整数结果并打印出来。[flsol[varstat/formula1.c]]

1. 和前一个问题做同样的事情，只是除以 `2.0` 而不是 `2`，并将结果存储在一个 `float` 中。然后打印出来。[flsol[varstat/formula2.c]]

1. 假设有一个变量 `i = 2`。编写一个程序，同时将 `j = i + 7`（得到 `9`）和在同一行代码中递增 `i`。[flsol[varstat/postinc.c]]

1. 假设有一个变量 `i = 2`。编写一个程序，在同一行代码中递减 `i`，然后将 `j = i + 7`（得到 `8`）赋值。[flsol[varstat/predec.c]]

1. 选择一个整数变量`i`的值。编写程序，如果`i`大于17，则打印`"foo"`，否则打印`"bar"`。 [flsol[varstat/foo17.c]]

1. 如果你在上一个问题中使用了`if`-`else`，请使用三元运算符完成同样的操作。否则，进行相反操作。[flsol[varstat/foo17b.c]]

1. 使用while循环来确定在数字$20754371$上进行整数除以$7$的次数，直到结果为零。[flsol[varstat/whilediv7.c]]

1. 使用单个`for`循环和两个变量来打印:
   ```
   0 0
   1 1
   2 3
   3 6
   4 10
   5 15
   ```
   其中第二个数字是其前一个数字加上第一个数字的值。在左列打印到99。最后`j`的值将为`4950`。[flsol[varstat/twofor.c]]

## 函数

1. 编写一个函数，带有两个整数参数`x`和`y`，计算

   $x\times20+x\times y$

   然后返回结果。[flsol[functions/funcxy.c]]

1. 编写一个通过Gregory和Leibniz无限级数计算$\pi$的函数，计算到指定项数（其中第一项为$1/1$（也就是$1$），如下):

   $\displaystyle\pi=4(\frac{1}{1}-\frac{1}{3}+\frac{1}{5}-\frac{1}{7}+\frac{1}{9}-\cdots)$

   打印出1到5000项的级数结果。第5000项应产生$3.141397$。[flsol[functions/pi.c]]

## 指针

```c
// 1. 编写一个函数，该函数接受一个指向`int`和一个指向`float`的指针作为参数。它应将`int`加到`float`上，并从`int`中减去`float`。应当看起来像这两步骤同时发生，例如输入为`5`和`3.2`，则应得到`1`和`8.2`。
//    由于通过指针操作值，结果应该在调用者中可见。
void some_function(int *ptr_to_int, float *ptr_to_float) {
    *ptr_to_float = *ptr_to_float + *ptr_to_int;
    *ptr_to_int = *ptr_to_int - *ptr_to_float;
}

// 2. 编写一个函数，该函数接受两个`int*`参数并交换它们。
int a = 10, b = 20;

swap(&a, &b);

printf("%d, %d\n", a, b); // 应该输出 20, 10
```

## 数组

// 1. 声明并初始化一个包含以下值的`int`数组：
//    ``` {.default}
//    10, 5, 2, 30, 97, 64
//    ```
//    编写一个名为`sum()`的函数，接受数组和数组长度作为参数。使函数返回数组元素的总和（即`208`）。

// 2. 声明一个包含2048个元素的数组，通过初始化设置索引为`312`的元素为`3490`。所有其他元素应初始化为零。

// 3. 编写一个函数，检查一个[flw[tie-tac-toe|Tic-tac-toe]]板上是否有赢家。在$3\times3$的板上，`0`表示没有人在那里移动过。`1`表示“X”在那里移动过，`2`表示“O”移动过。返回`0`、`1`或`2`表示没有赢家，“X”赢了，或“O”赢了，分别。

## 字符串

// 1. 编写一个可以原地翻转字符串的函数。使其返回传入的相同指针。

// 2. 编写一个函数，如果字符串完全大写则返回true。提示：查看`isupper()`函数。
```

```c
// 写一个函数，返回字符串中第一个字母的指针。如果字符不在字符串中，则返回`NULL`。 [flsol[strings/my_strchr.c]]
```

## 结构体

```c
// 编写一个名为`struct point`的结构体，包含两个名为`x`和`y`的`double`成员，用于存储2D点的信息。编写一个函数，通过将它们的各个分量相加来将两个`struct point`相加。函数应该返回一个`struct point`作为结果。 [flsol[structs/point.c]]
```

```c
// 进行相同的操作，但是将指向`struct point`的指针传递到函数中。使函数修改第一个点，使其成为第一个点和第二个点的总和。 [flsol[structs/point_ptr.c]]
```

## 输入/输出

```c
// 编写一个程序，从`stdin`中最多读取50行，每行不超过80个字符，然后以相反顺序打印它们。提示：字符的二维数组可能有帮助。 [flsol[io/reverselines.c]]
```

```c
// 编写一个程序，从[fls[`input.txt`|io/input.txt]]中读取3D坐标，并计算平均3D点，然后将其打印出来。平均点是来自输入的X的平均值，然后是Y的平均值，最后是Z的平均值。对于`input.txt`的情况：

// ```
// 平均点 <46.296997,48.968998,49.845001>
// ```

// [flsol[io/point_avg.c]]
```

## 类型定义

```c
// 编写一个`typedef`，创建一个名为`vector3d`的新类型，用于以下`struct`：

// ``` {.c}
// struct {
//    float x, y, z;
// };
// ```

// 编写一个名为`dot3d`的函数，计算并返回两个`vector3d`的矢量点积。此函数应接受两个指向`vector3d`的指针作为参数，因此可以这样调用：

// ``` {.c}
// float result = dot3d(&v1, &v2);
// ```
```

```markdown
   两个向量的点积是它们的`x`、`y`和`z`分量的乘积之和：

   $v_1=(x_1,y_1,z_1)$\
   $v_2=(x_2,y_2,z_2)$

   $v_1\cdot v_2 = x_1\times x_2 + y_1\times y_2 + z_1\times z_2$

   [flsol[typedef/vector3d.c]]

## 指针 II

1. 编写`my_strchr()`函数，返回字符串中第一个字符的指针，类似于`strchr()`。使用指针算术来实现。
   [flsol[pointers2/my_strchr.c]]

1. 编写`my_strrchr()`函数，返回字符串中最后一个字符的指针，类似于`strrchr()`。使用指针算术来实现。
   [flsol[pointers2/my_strrchr.c]]

1. 编写一个函数`object_sum()`，计算由`void*`参数指向的对象的字节和。

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

1. 编写一个用于对`int`数组进行升序排序的比较函数，使用`qsort()`。再编写一个用于降序排序的比较函数。 [flsol[pointers2/int_qsort.c]]

## 手动内存分配

1. 编写一个程序，为一个包含10个`int`的数组分配空间，然后为每个索引填充索引号乘以10的数组元素。
   [flsol[manmem/tenints.c]]

1. 编写一个程序，将任意大小的文件读入内存，并返回指向数据的指针，而不需要事先知道文件大小。这里可以使用`fread()`函数。
   [flsol[manmem/readfile.c]]

## 作用域

1. 这段代码会打印什么？

   ``` {.c .numberLines}
   #include <stdio.h>

   int total = 0;
```

```c
void add(int x)
{
    total += x;
    printf("现在总数为 %d\n", total);
}

int main(void)
{
    for (int i = 0; i < 3; i++)
        add(i);
}
```

答案^[用数值`1`、`3`、`5`分别替换`Total is now`。`total`位于文件作用域，因此每次调用`add()`都会影响它。]。

1. 这段代码会打印什么内容？

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    int i = 0;

    if (1 < 2) {  // 始终为真
        int i = 0;
    
        i += 5;
    }

    i += 5;

    printf("%d\n", i);
}
```

答案^[`5`--`i`的第一次增加是在隐藏外部作用域变量上进行的]。

2. 这段代码会打印什么内容？

``` {.c .numberLines}
#include <stdio.h>

int main(void)
{
    for (int i = 0; i < 3; i++) {
        for (int i = 0; i < 2; i++) {
            printf("你好！\n");
        }
    }
}
```

答案^[打印了`Hi there`六次。内部声明的`i`隐藏了外部的一个]。

## 类型 II

1. 编写一个程序，将一个`unsigned char`初始化为`0`。然后在循环中，总共增加这个值260次，并在每次增加后打印出来。

当值达到255时会发生什么^[假设您的计算机使用的是8位字节]？

答案^[从255翻转到0，就像里程表一样。当值达到255时发生这种情况，因为255对应的二进制是`11111111`---所有8位均为`1`，所以再加1得到的是`100000000`。但这是9位数，太大了无法容纳在一个字节内。最高位被舍弃，留下的二进制是`00000000`。当无符号类型值从最大值递增时，所有无符号类型都会这样做]。 [flsol[types2/overflow.c]]

## 类型 III

1. 编写一个函数，打印出浮点数的整数部分，后面跟一个`+`符号，再后面跟着数字的小数部分。举个例子，给定一个`float`值`3.14159`，应该打印出：

   ``` {.default}
   3+0.14159
   ```

   尝试在没有任何字符串操作的情况下完成，只通过数值操作。

   [flsol[types3/cutfrac.c]]

1. 编写自己的函数，将字符串转换为整数，不使用任何内置的转换函数（如`atoi()`或`strtol()`）。

   ``` {.c}
   int my_atoi(char *s)
   {
        // TODO
   }

   printf("%d\n", my_atoi("3490"));  // 输出 3490
   ````

   要加分项：

   * 使其跳过前导空格---参考`isspace()`函数
   * 当遇到第一个非数字字符时停止转换---参考`isdigit()`函数
   * 处理负数

   [flsol[types3/my_atoi.c]]

## 类型 IV

1. 这将是一个由两个文件组成的项目。一个文件包含将数字添加到累加总和中的功能。另一个文件调用该功能。

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

   1. 一个名为`void add(int x)`的函数。这将加到从`0`开始的累计总和上。
   2. 一个名为`int total(void)`的函数，返回至今为止的累计总和。
   3. 一个名为`count`的共享全局变量，返回`add()`被调用的次数。

```c
   实现`total.c`中的功能，然后修改`runner.c`，使其能调用`add()`和`total()`函数并可以访问变量`count`。

   确保`total()`是唯一`runner.c`获取总和的方式。

   你需要利用`static`和`extern`来解决这个问题。

   通常情况下，这是我们会使用[头文件](#includes-func-protos)的场景，但我们还没有讨论过这个。

   你可以通过在命令行中同时指定两个文件来一起编译它们：

   ``` {.default}
   gcc -Wall -Wextra -o runner runner.c total.c
   ```

   [flsol[types4/runner.c]] [flsol[types4/total.c]]

## 多文件项目

1. 修改上面.Types IV的练习，使用头文件`total.h`。

   确保在头文件上使用`#ifdef`包装器。

   如果在链接所有文件之前将C文件编译为目标文件，则为加分项目。

   [flsol[multifile/runner.c]] [flsol[multifile/total.c]]
   [flsol[multifile/total.h]]

## 外部环境

1. 编写一个程序，打印出在命令行上指定的环境变量的值。

   如果变量未设置，则应打印出消息`no such variable`。

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

   （`HOME`、`LANG`和`PATH`是shell常常设置的变量，所以很可能它们已经存在了。）

   你需要使用`getenv()`来完成这个任务。

   [flsol[env/printenv.c]]

## C预处理器
```

```cpp
// 写一个宏，接受两个参数，将它们相乘，然后将当前源代码行号加到乘积中。宏的结果应该是整数类型。

#define MULTIPLY_AND_ADD_LINE(x, y) ((x) * (y) + __LINE__)

// 写代码，根据当前的C版本打印出`C89`、`C99`或者`C11或更高版本`。

#if __STDC_VERSION__ >= 201112L
    printf("C11或更高版本\n");
#elif __STDC_VERSION__ >= 199901L
    printf("C99\n");
#else
    printf("C89\n");
#endif

// 有一个程序通过`#define`为`X`定义一个数字值。使用条件编译，如果条件为真则打印`X大于30`。否则打印从`0`到`X-1`的值。

#define X 35

#if X > 30
    printf("X大于30\n");
#else
    for (int i = 0; i < X; i++) {
        printf("%d\n", i);
    }
#endif

```