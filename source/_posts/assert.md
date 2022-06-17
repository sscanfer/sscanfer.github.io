---
title: assert
date: 2021-10-21 06:09:12
tags: C
---

# 断言（assert, _Static_assert）

参考资料：

1. C Primer Plus 6th 16.12 断言库
2. [断言(assert)的用法 - thisway_diy - 博客园 (cnblogs.com)](https://www.cnblogs.com/thisway/p/5558914.html)
3. [C 语言编程 — 使用 assert 断言进行程序设计 - 不言不语技术 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hzcya1995/p/13309246.html)

## 简介

*assert.h* 头文件支持的断言库是一个用于辅助调试程序的小型库。它由`assert()`宏组成，接受一个整型表达式作为参数。如果表达式求值为假（非零），`assert()`宏就在标准错误流`stderr`中写入一条错误信息，并调用`abort()`函数终止程序（`abort()`函数的原型在 *stdlib.h* 头文件中）。

assert()宏是为了标识出程序中某些条件为真的关键位置，如果其中的一个具体条件为假，就用 `assert()`语句终止程序。通常，`assert()`的参数是一个条件表达式或逻辑表达式。如果 `assert()`中止了程序，它首先会显示失败的测试、包含测试的文件名和行号。

## assert的用法

以下程序演示了一个使用assert的小程序。在求平方根之前，该程序断言z是否大于或等于0。程序还错误地减去一个值而不是加上一个值，故意让z得到不合适的值。

```c
/* 程序清单1
 * assert.c -- 使用 assert()
 */
#include <stdio.h>
#include <math.h>
#include <assert.h>

int main(void)
{
    double x, y, z;

    puts("Enter a pair of numbers ( 0 0 to quit):");
    while (scanf("%lf %lf", &x, &y) == 2 && (x != 0 || y != 0))
    {
        z = x * x - y * y;  /* 本来要用+ */
        assert(z >= 0);
        printf("answer is %f\n", sqrt(z));
        puts("Next pair of numbers: ");
    }
    puts("Done.");

    return 0;
}
```

运行结果如下图：
![assert.c运行结果](assert_c.png)

**分析：**
*具体的错误提示因编译器而异*。这条消息可能不是指明`z >= 0`，而是指明没有满足`z >=0`的条件。
用`if`语句也能完成类似的任务：

```c
if (z < 0) {
	puts("z less than 0");
	abort();
}
```

但是，使用`assert()`有几个好处：它不仅能自动标识文件和出问题的行号，还有一种无需更改代码就能开启或关闭`assert()`的机制。

如果认为已经排除了程序的 bug，就可以把下面的宏定义写在包含 *assert.h* 的位置前面：

```c
#define NDEBUG
```

并重新编译程序，这样编译器就会禁用文件中的所有`assert()`语句。如果程序又出现问题，可以移除这条`#define`指令（或者把它注释掉），然后重新编译程序，这样就重新启用了`assert()`语句。

## 注意事项

1. 每个`assert()`只检验一个条件，因为同时检验多个条件时,如果断言失败，无法直观的判断是哪个条件失败。

    ```c
    // 反例
    assert(nOffset>=0 && nOffset+nSize<=m_nInfomationSize);

    // 正例
    assert(nOffset >= 0);
    assert(nOffset+nSize <= m_nInfomationSize);
    ```

2. **ASSERT只有在Debug版本中才有效，如果编译为Release版本则被忽略**。换言之，不能在断言中使用改变环境的语句，因为`assert`只在 *DEBUG* 时生效，如果这么做，会使用程序在真正运行时遇到问题。

    ```c
    // 反例
    assert(i++ < 100);

    // 正例
    assert(i < 100);
    i++;
    ```

   对于上面程序清单中的反例，如果执行出错（例如初始`i > 100`)，则`i++`就不会执行。

3. **不能用断言来检查运行时错误：**断言是用来处理内部编程或设计是否符合假设；对于可能会发生的且必须处理的情况要写防错程序，而不是断言。

    ```c
    // 反例
    #include <stdio.h>
    #include <stdlib.h>
    #incldue <assert.h>

    #define MAX 100

    int main(int argc, char **argv)
    {
        int *result = NULL;
        result = (int*)malloc(MAX * sizeof(int));

        assert(result != NULL);  // 错误用法

        for (int i = 0; i < MAX; i++) {
            result[i] = i;
        }

        return 0;
    }
    ```

## _Static_assert (C11)

> `assert()`表达式是在运行时进行检查。C11新增了一个特性：`_Static_assert`，可以在编译时检查`assert()`表达式。
>
> 因此，`assert()`可以导致正在运行的程序中止，而`_Static_assert()`可以导致程序无法通过编译。

`_Static_assert()`接受两个参数。第1个参数是**整型常量表达式**，第2个参数是一个字符串。如果第 1 个表达式求值为 0（或_False），编译器会显示字符串，而且不编译该程序。

```c
/* 程序清单2
 * statasrt.c -- 使用_Static_assert()
 */
#include <stdio.h>
#include <limits.h>
_Static_assert(CHAR_BIT == 16, "16-bit char falsely assumed");

int main(void)
{
    puts("char is 16 bits.");
    return 0;
}
```

编译结果如下图所示：

![statasrt.c编译结果](statasrt_c.png)

**分析：**

根据语法，`_Static_assert()`被视为声明。因此，它可以出现在函数中，或者在这种情况下出现在函数的外部。

`_Static_assert`要求它的第1个参数是整型常量表达式，这保证了能在编译期求值（`sizeof`表达式被视为整型常量）。

**注意：**

在程序清单1中，不能用`_Static_assert`代替`assert`，因为`assert`中作为测试表达式的`z > 0`不是常量表达式，要到程序运行时才求值。当然，可以在程序清单的`main()`函数中使用`assert(CHAR_BIT == 16)`，但这会在编译和运行程序后才生成一条错误信息，很没效率。
