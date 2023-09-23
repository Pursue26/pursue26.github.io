---
title: C语言之main函数参数
date: 2023-09-16 17:19:29
categories:
    - 学习笔记
    - C语言
tags:
    - main函数参数
abbrlink: 230916171929
---

## 指令行操作

C 语言支持从 CLI（指令行）传入参数给 main() 函数，多个命令行参数之间用空格分隔，但是如果参数本身带有空格，那么就使用双引号或单引号。

C语言的 main 函数是程序的入口函数，它可以有两种形式的参数：

<!--more-->

1. 无参数形式：main 函数的原型可以是`int main(void)`。这表示不接受任何参数，程序执行时不需要从命令行传递参数给 main 函数。

2. 带参数形式：main 函数的原型可以是`int main(int argc, char *argv[])`。其中，`argc` 表示命令行参数的个数，而 `argv` 是一个字符串指针数组，每个元素都是一个命令行参数的字符串。

   - `argc` 标识传入的参数个数。如果没有提供任何参数，`argc` 将被设置为 1；否则，`argc` 将被设置为传入的参数个数加 1。
   - `argv[0]` 通常是程序的名称或路径的字符串。
   - `argv[1]` 到 `argv[argc - 1]` 是命令行传递给程序的参数，以空格分隔。

值得注意的是，也可以使用 `char **argv` 来代替 `char *argv[]` 形参，两者具有相同的类型含义。因为数组变量名 `argv` 就是指向 `argv[]` 数组的第一个元素的指针，同时又因为第一个元素的数值就是一个指针，所以此时的数组变量名 `argv` 的本质就是一个指针的指针（双重指针）。

下面是一个例子，演示如何使用带参数的 main 函数来接收命令行参数：

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("number of params: %d\n", argc);
    
    for (int i = 0; i < argc; i++) {
        printf("param-%d: %s\n", i, argv[i]);
    }
    
    return 0;
}
```

如果编译并运行这个程序，并在命令行中输入参数，例如 `./a.out arg1 arg2 3 arg4`，则输出如下：

```text
number of params: 5
param-0: ./a.out
param-1: arg1
param-2: arg2
param-3: 3
param-4: arg4
```

参考链接：https://is-cloud.blog.csdn.net/article/details/105347737
