---
title: C语言之typedef实践
date: 2023-09-20 17:28:49
categories:
    - 学习笔记
    - C语言
tags:
    - typedef
abbrlink: 230920172849
---

在 [C 语言之 typedef 自定义类型基础](https://pursue26.github.io/posts/230918174223.html) 中介绍了 `typedef` 的使用，下面介绍一些 `typedef` 的实践。

<!--more-->

## 两大陷阱

### 不是简单的字符串替换

`typedef` 是定义了一种类型的新别名，不同于宏，它不是简单的字符串替换。

当 `const` 与 `typedef` 相结合时，如定义：

```c
typedef char* PSTR;
int mystrcmp(const PSTR, const PSTR);
```

`const PSTR` 实际上相当于指针常量 `const char*` 吗？

不是的，它实际上相当于 `char* const`。这是因为 `const` 给予了整个指针本身以常量性，也就是形成了常量指针。简单来说，记住当 `const` 和 `typedef` 一起出现时，`typedef` 不会是简单的字符串替换就行。

### 与其它存储类关键字不共存

`typedef` 在语法上是一个**存储类关键字**，虽然它并不真正影响对象的存储特性，但变量只能被一种储类的关键字修饰。

```c
typedef static int SINT_t;
typedef auto int AINT_t;
extern typedef int INT_t;
```

上述代码中，有两个存储类关键字，会编译将失败，报错 error: multiple storage classes in declaration specifiers

```c
typedef int INT_t;
static INT_t a = 10;
```

这种写法是合法的。

> 其它存储类关键字有 `auto`、`extern`、`mutable`、`static`、`register` 等。

## typedef、const与define结合

示例1：通常讲，`typedef` 要比 `define` 要好，**特别是在有指针的场合**。

```c
#define pStr2 char*
typedef char* pStr1;

pStr1 s1, s2;  
pStr2 s3, s4;
```

在上述的变量定义中，`s1`、`s2`、`s3`都被定义为`char *`，而`s4`则定义成了`char`，不是我们所预期的指针变量，根本原因就在于`define`只是简单的字符串替换而`typedef`则是为一个类型起新名字。


示例2：当`const`和`typedef`一起出现时，`typedef`不会是简单的字符串替换。

```c
typedef char* pStr;
char s[4] = "abc";
const char *p1 = s;
const pStr p2 = s; // 实际为 char* const p2 = s;

p1++;  // 指针常量的指针可变
p2++;  // 编译报错, 常量指针的指针不可变
*p1 = 'm';  // 编译报错, 指针常量的值不可变

s[0] = 'd';
s[1] = 'e';

printf("s = %s, %c\r\n", s, *p1, *p2);  //dec, e, d
*p2 = 'm';
printf("s = %s, %c\r\n", s, *p1, *p2);  //mec, e, m
```

示例中，
- `const char *p1`是限定数据类型为`char *`的指针变量可变，指针变量指向的对象不可变，所以`p1++`正确；
- `p2++`出错了，这个问题再一次提醒我们：`typedef`和`define`不同，它不是简单的文本替换；
- `const pStr p2`并不等于`const char *p2`，`const pStr p2`和`const long x`本质上没有区别，都是对变量进行只读限制，只不过此处变量`p2`的数据类型是我们自己定义的，而不是系统固有类型。因此，`const pStr p2`的含义是：限定数据类型为`char *`的指针变量`p2`为只读，因此`p2++`错误。

## 抑制劣质代码

人们常常使用 `typedef` 来编写更美观和可读的代码。所谓美观，意指 `typedef` 能隐藏笨拙的语法构造以及平台相关的数据类型，从而增强可移植性以及未来的可维护性。在编程中使用 `typedef` 目的一般有两个，一个是给变量一个易记且意义明确的新名字，另一个是简化一些比较复杂的类型声明。

### 定义易于记忆的类型名

`typedef` 使用最多的地方是创建易于记忆的类型名，用它来归档程序员的意图。类型出现在所声明的变量名字中，位于 `typedef` 关键字右边。例如：

```c
typedef int size;
```

此声明定义了一个 `int` 的同义字，名字为 `size`。注意 `typedef` 并不创建新的类型，它仅仅为现有类型添加一个同义字。因此，你可以在任何需要 `int` 的上下文中使用 `size`：

```c
void measure(size *pSize);
size array[4];
size len = file.getlength();
```

`typedef` 还可以掩饰复合类型，如指针和数组。例如，你不用像下面这样重复定义有 81 个字符元素的数组：

```c
char line[81];
char text[81];
```

定义一个 `typedef`，每当要用到相同类型和大小的数组时，可以这样：

```c
typedef char Line[81];
Line text, secondline;
getline(text);
```

同样，也可以像下面这样隐藏指针语法：

```c
typedef const char *pstr_t;
int mystrcmp(pstr_t, pstr_t);
```

**记住**，不管什么时候，只要为指针声明 `typedef`，都要在最终的 `typedef` 名称中加一个 `const`，比如：

```c
typedef const char *cpstr_t;  // 指针常量, 使得该指针可变, 而指针指向的对象不可变（只读）
typedef char* const cpstr_t;  // 常量指针, 使得该指针不可变（只读）, 但指针指向的对象可变的
```

### 简化代码

上面讨论的 `typedef` 行为有点像 `define` 宏，用其实际类型替代同义字。不同点是 `typedef` 在编译时被解释，因此可以让编译器来应付超越预处理器能力的文本替换。例如：

```c
typedef int (*PF) (const char *, const char *);
```

这个声明引入了 `PF` 类型作为函数指针的同义字，该函数有两个 `const char *` 类型的参数以及一个 `int` 类型的返回值。如果要使用下列形式的函数声明，那么上述这个 `typedef` 是不可或缺的：

```c
PF Register(PF pf);
```

`Register()` 的参数是一个 `PF` 类型的回调函数，返回某个函数的地址，其署名与先前注册的名字相同。下面我展示一下如果不用 `typedef`，我们是如何实现这个声明的：

```c
int (*Register (int (*pf)(const char *, const char *))) (const char *, const char *); 
```

很少有程序员理解它是什么意思，更不用说这种费解的代码所带来的出错风险了。显然，这里使用 `typedef` 不是一种特权，而是一种必需。

持怀疑态度的人可能会问：“OK，还会有人写这样的代码吗？”，快速浏览一下揭示 `signal()` 函数的头文件 `<csinal>`，有一个同样接口的函数：

```c
// signal原型
void (*signal(int sig, void (*func)(int)))(int);

// typedef优化后
typedef void (*pFun)(int);
pFun signal(int sig, pFun func);
```

> 参考链接：https://www.cnblogs.com/a-s-m/p/10995722.html
