---
title: C语言之多维数组
date: 2023-09-22 14:30:03
categories:
    - 学习笔记
    - C语言
tags:
    - 多维数组
    - 数组排序
abbrlink: 230922143003
---

本文章主要介绍 C 语言创建多维数组的方式及其排序，包括静态多维数组、`malloc` 动态申请多维数组、指针数组的多维数组和多维结构体数组的创建。

<!--more-->

## 排序接口

`qsort` 函数是 C 语言标准库中的一个排序函数，用于对数组进行快速排序。它的函数原型如下：

```c
void qsort(
    void *base, 
    size_t nmemb, 
    size_t size, 
    int (*compar)(const void *, const void *)
);
```

参数说明：

-   `base`：指向需要排序的数组的第一个元素的指针。
-   `nmemb`：数组中元素的个数。
-   `size`：每个元素的大小（以字节为单位）。
-   `compar`：指向比较函数的指针，用于指定数组元素的比较规则。

`base` 指定了待排序数组的首地址，再结合 `nmemb * size` 可以确定要排序的数组的范围。

比较函数 `compar` 的原型如下：

```c
int compar(const void *a, const void *b);
```

参数说明：

-   `a` 和 `b`：指向待比较的两个元素的指针。

比较函数 `compar` 必须返回一个整数值，表示 `a` 和 `b` 的大小关系：

-   如果 `a` 小于 `b`，则返回一个负整数。
-   如果 `a` 等于 `b`，则返回零。
-   如果 `a` 大于 `b`，则返回一个正整数。

通过传入不同的比较函数，`qsort` 函数可以实现对不同类型的数组进行排序。

## 静态多维数组

### 多维数组创建

```c
int rows = 3, cols = 4;
int arr[rows][cols];
```

### 多维数组排序

按照二维数组第0列升序排序。

```c
int cmp(const void *a, const void *b) {
    return ((int *)a)[0] - ((int *)b)[0];
}

// 传入cmp函数指针的arr是一个一维指针
qsort(arr, rows, sizeof(arr[0]), cmp);  // 快速排序
```

## malloc动态申请多维数组

### 多维数组创建

```c
int rows = 3, cols = 4;

// 动态申请二维数组
int **arr = (int **)malloc(rows * sizeof(int *));
for (int i = 0; i < rows; i++) {
    arr[i] = (int *)malloc(cols * sizeof(int));
}

// 释放内存
for (int i = 0; i < rows; i++) {
    free(arr[i]);
}
free(arr);
```

释放 `malloc` 申请的多维数组时，不能直接 `free(arr)`，因为 `malloc` 和 `free` 执行次数要一致。

### 多维数组排序

按照二维数组第0列升序排序，若相等，则按第1列升序排序。

```c
int cmp(const void* a, const void* b) {
    // a 是一个指向二维数组首行的指针
    // 通过类型转换和解引用操作, ap指向了a所指向的整型数组的首地址
    const int* ap = *(int **)a;
    const int* bp = *(int **)b;
    if (ap[0] == bp[0]) {
        return ap[1] - bp[1];
    } else {
        return ap[0] - bp[0];
    }
}

// 传入cmp函数指针的arr是一个二维指针
qsort(arr, rows, sizeof(arr[0]), cmp);
```

## malloc动态申请多维结构体数组

### 多维结构体数组创建

```c
int rows = 3, cols = 4;

typedef struct node {
    int x;
    int y;
    int z;
} Node_t;

// 动态申请二维数组
Node_t **arr = (Node_t **)malloc(rows * sizeof(Node_t *));
for (int i = 0; i < rows; i++) {
    arr[i] = (Node_t *)malloc(cols * sizeof(Node_t));
}

// 释放内存
for (int i = 0; i < rows; i++) {
    // 如果结构体成员中有指针且申请了空间, 则需要先释放成员的空间再释放结构体的空间
    free(arr[i]);
}
free(arr);
```

### 多维结构体数组排序

按照二维数组第0列的结构体成员 `x` 降序排序。

```c
int cmp(const void *a, const void *b)
{
    // return (Node_t *)a->x - (Node_t *)b->x;   //错误写法
    return ((Node_t *)a)->x - ((Node_t *)b)->x;   //正确写法1
    // return (*(Node_t *)b).x - (*(Node_t *)a).x;     //正确写法2
}

qsort(arr, rows, sizeof(arr[0]), cmp);
```

- `->` 操作符的优先级高于 `()` 操作符。

## 指针数组的多维数组

使用指针数组的多维数组：

```c
int rows = 3, cols = 4;

int* arr[rows];
for (int i = 0; i < cols; i++) {
    arr[i] = (int *)malloc(4 * sizeof(int));
}

// 释放内存
for (int i = 0; i < rows; i++) {
    free(arr[i]);
}
```

这种方式创建的多维数组实际上是一个指针数组，每个指针指向一个一维数组，可以在运行时动态分配每个一维数组的大小（方便创建每行元素个数不同的多维数组）。

### 多维数组排序

按照二维数组第0列降序排序。

```c
int cmp(const void* a, const void* b) {
    const int* ap = *(int **)a;
    const int* bp = *(int **)b;
    return bp[0] - ap[0];
}

// 传入cmp函数指针的arr是一个二维指针
qsort(arr, rows, sizeof(arr[0]), cmp);
```
