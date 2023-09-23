---
title: C语言之const关键字
date: 2023-09-13 20:07:29
categories:
    - 学习笔记
    - C语言
tags:
    - const关键字
    - 下限定
    - 上限定
abbrlink: 230913200729
---


宏定义 `#define` 和 `const` 关键字在 C 语言中有不同的用途：

1. `#define` 用于定义宏常量，它在编译前进行简单的文本替换，没有类型检查和作用域限制。宏常量的值可以是任意的表达式，包括函数调用和运算符操作，但容易导致代码的可读性和可维护性降低。

2. `const` 关键字用于定义常量变量，具有类型检查和作用域限制的特性。常量变量的值在定义后不能被修改，提高了代码的可读性和可维护性。常量变量的作用域仅限于定义它的代码块内，其他代码块无法访问该常量。

3. `const` 关键字还可以用于定义指向常量的指针、常量指针以及指向常量的常量指针。这些用法可以在编译阶段进行类型检查，避免了在运行时可能出现的错误。

4. 编译器通常不会为*普通的* `const` 常量分配存储空间，而是将它们保存在符号表中，成为编译期间的常量。这样可以避免存储和读内存的操作，提高程序的执行效率。

综上所述，虽然 `#define` 可以用于定义常量，但 `const` 关键字更加推荐，因为它提供了类型检查和作用域限制，提高了代码的可读性和可维护性，同时能够进行编译期间的优化。

<!--more-->

## 修饰常量变量（Constant Variables）

```c
const int var = 100;
int const var = 100;
```

这两种写法是等价的，都表示变量 `n` 的值不能被修改了。需要注意的是，用 `const` 修饰变量时，一定要给变量初始化，否则后续就不能赋值了。

```c
// C program to demonstrate that constant variables can not be modified
int main()
{
    const int var = 100;

    // Compilation error: assignment of read-only variable 'var'
    var = 200;

    return 0;
}
```

## 修饰常量字符串

```c
const char *str = "Hello, World!";
```

**字符指针指向的字符串是不能修改的**：在 C 语言中，字符串常量是以**只读方式**存储的，编译器会将其**存储在只读的数据段**中。而使用 `const` 关键字修饰的指针变量，会告诉编译器不允许通过该指针修改指向的数据。

如果没有 `const` 的修饰，我们可能会在后面有意无意的写 `str[0]='x'` 这样的语句，这样会导致对只读内存区域的赋值，然后程序会立刻异常终止。但是，有了 `const`，这个错误就能在程序被编译的时候被检查出来，这就是 `const` 的好处，让逻辑错误在编译期被发现。

## 指针常量（Pointer to Constant）

```c
const int* ptr;
int const *ptr;
```

这两种写法是等价的。指针常量可以更改指针 `ptr` 的地址，以指向任何其他同类型的变量，但不能更改指针 `ptr` 指向的对象的值。指针常量中的指针被存储在读写区域（read-write area），在本例中为堆栈，所指向的对象可能位于只读区域或读写区域。

> 简言之：指针（的地址）可变；指针地址不变的前提下，其指向的内容不可变。

变量 `i` 是变量：

```c
// C program to demonstrate that the pointer to point to any other integer variable, 
// but the value of the object (entity) pointed can not be changed

int main(void)
{
    int i = 10;
    int j = 20;
    /* ptr is pointer to constant */
    const int* ptr = &i;

    printf("ptr: %d\n", *ptr);
    /* error: object pointed cannot be modified using the pointer ptr */
    /* error: assignment of read-only location '*ptr' */
    *ptr = 100;

    ptr = &j; /* valid */
    printf("ptr: %d\n", *ptr);

    return 0;
}
```

变量 `i` 本身就是常量：

```c
// C program to demonstrate that the pointer to point to any other integer variable, 
// but the value of the object (entity) pointed can not be changed

int main(void)
{
    /* i is stored in read only area*/
    int const i = 10;
    int j = 20;

    /* pointer to integer constant. Here i
    is of type "const int", and &i is of
    type "const int *". And ptr is of type
    "const int *", types are matching no issue */
    int const* ptr = &i;

    printf("ptr: %d\n", *ptr);

    /* error: assignment of read-only location '*ptr' */
    *ptr = 100;

    /* valid. We call it up qualification. In
    C/C++, the type of "int *" is allowed to up
    qualify to the type "const int *". The type of
    &j is "int *" and is implicitly up qualified by
    the compiler to "const int *" */

    ptr = &j;
    printf("ptr: %d\n", *ptr);

    return 0;
}
```

> 在C++中，不允许进行下限定（down qualification）操作，而在C中进行此操作可能会导致警告。下限定是指将已经限定了类型的变量赋值给没有限定类型的变量的情况。
> 
> 在C++和C中，上限定（up qualification）是允许的，不会引发警告或错误。上限定是指将没有限定类型的变量赋值给已经限定了类型的变量的情况。这意味着可以将一个没有限定类型的变量赋值给一个已经限定了类型的变量，而编译器会自动进行类型转换。上限定操作可以用于提升变量的类型精度或实现类型转换。例如，将一个整数赋值给一个浮点数变量，编译器会自动将整数转换为浮点数类型。


下限定操作示例：

```c
// C program to demonstrate the down qualification

int main(void)
{
    int i = 10;
    int const j = 20;

    /* ptr is pointing an integer object */
    int* ptr = &i;

    printf("*ptr: %d\n", *ptr);

    /* The below assignment is invalid in C++, results in
    error In C, the compiler *may* throw a warning, but
    casting is implicitly allowed */
    ptr = &j;  /* warning: assignment discards ‘const’ qualifier from pointer target type [-Wdiscarded-qualifiers] */

    /* In C++, it is called 'down qualification'. The type
    of expression &j is "const int *" and the type of ptr
    is "int *". The assignment "ptr = &j" causes to
    implicitly remove const-ness from the expression &j.
    C++ being more type restrictive, will not allow
    implicit down qualification. However, C++ allows
    implicit up qualification. The reason being, const
    qualified identifiers are bound to be placed in
    read-only memory (but not always). If C++ allows
    above kind of assignment (ptr = &j), we can use 'ptr'
    to modify value of j which is in read-only memory.
    The consequences are implementation dependent, the
    program may fail
    at runtime. So strict type checking helps clean code.
    */

    printf("*ptr: %d\n", *ptr);

    return 0;
}
```

## 常量指针指向变量（Constant Pointer to Variable）

```c
int* const ptr;
```

上面的声明是一个指向整型变量的常量指针，这意味着我们可以改变该指针所指向的对象的值，但不能改变该指针指向另一个变量。

> 简言之：指针（地址）是常量，不可变；指针指向的变量可变。

```c
// C program to demonstrate that the value of object pointed
// by pointer can be changed but the pointer can not point to another variable

int main(void)
{
    int i = 10;
    int j = 20;

    /* constant pointer to integer */
    int* const ptr = &i;
    printf("ptr: %d\n", *ptr);

    *ptr = 100; /* valid */
    printf("ptr: %d\n", *ptr);

    /* error: assignment of read-only variable 'ptr' */
    ptr = &j;

    return 0;
}
```

## 常量指针指向常量（Constant Pointer to Constant）

```c
const int* const ptr;
```

上面的声明是一个指向常量变量的常量指针，这意味着我们不能更改指针所指向的值，也不能将指针指向其他变量。

```c
// C program to demonstrate that value pointed by the
// pointer can not be changed as well as we cannot point the
// pointer to other variables

int main(void)
{
    int i = 10;
    int j = 20;

    /* constant pointer to constant integer */
    const int* const ptr = &i;
    printf("ptr: %d\n", *ptr);

    ptr = &j; /* error: assignment of read-only variable 'ptr' */
    *ptr = 100; /* error: assignment of read-only location '*ptr' */

    return 0;
}
```

| Type                                  | Declaration             | Pointer Value Change | Pointing Value Change |
|---------------------------------------|-------------------------|----------------------|-----------------------|
| Pointer to Variable                   | `int *ptr`              | Yes                  | Yes                   |
| Pointer to Constant                   | `const int *ptr`        | No                   | Yes                   |
| Pointer to Constant                   | `int const *ptr`        | No                   | Yes                   |
| Constant Pointer to Variable          | `int* const ptr`        | Yes                  | No                    |
| Constant Pointer to Constant          | `const int* const ptr`  | No                   | No                    |

注：
1.  Pointer Value Change（指向的值的改变），如 `*ptr = 100`

2.  Pointing Value Change（指针的值的改变），如 `ptr = &a`

3. 如果 `const` 在 `*` 的左边，那么 `const` 可以与类型互换位置，如 `const int *ptr` 等价于 `int const *ptr`

4.  被 `const` 修饰的变量，地址不可变还是值不可变，傻傻分不清？观察 `const` 靠近普通类型还是靠近指针类型：
    -   如果 `const` 靠近的是普通类型，那么常量变量的值不可变，指针地址可变，如 `const int *ptr`，`int const *ptr`
    -   如果 `const` 靠近的是指针类型，那么指针指向的基本类型的值可变，指针地址不可变，如 `int* const ptr`
5.  **指针常量与常量指针记忆方法**：只保留 `const` 和 `*`，从右往左读，如下：
    -   ~~int~~ *~~ptr~~指针变量
    -   const ~~int~~ *~~ptr~~指针常量
    -   ~~int~~ const *~~ptr~~指针常量
    -   ~~int~~* const ~~ptr~~常量指针
    -   const ~~int~~* const ~~ptr~~常量指针指向常量变量

> 参考链接：https://www.geeksforgeeks.org/const-qualifier-in-c/
