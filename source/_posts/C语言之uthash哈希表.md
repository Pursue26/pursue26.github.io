---
title: C语言之uthash哈希表
date: 2023-09-22 17:26:26
categories:
    - 学习笔记
    - C语言
tags:
    - 哈希表
abbrlink: 230922172626
---

由于 C 语言本身不存在哈希，但是当需要使用哈希表的时候自己构建哈希会异常复杂。因此，我们可以调用开源的第三方库 `uthash.h`，**这只是一个头文件**。我们需要做的就是将头文件通过 `#include "uthash.h"` 引入到自己的项目中。由于 `uthash` 仅是头文件，因此没有可链接的库代码。

<!--more-->

## uthash简介

使用 `uthash` 添加、查找和删除通常是常数时间的操作，此哈希库的目标是简约高效。它大约有 1000 行 C 代码，它会*自动内联*，因为它是作为宏实现的。

`uthash` 还包括三个额外的头文件，主要提供链表、动态数组和动态字符串。
-   `utlist.h` 为 C 结构提供了链接列表宏。
-   `utarray.h` 使用宏实现动态数组。
-   `utstring.h` 实现基本的动态字符串。

uthash 中的 ut 是 unordered tables，即无序表。

> 内联是一种编译器优化，它将「函数调用」替换为「函数体的代码」。这样可以避免函数调用时所需的开销，从而提高性能。

## uthash的使用

### 初始化

这里我们将 `id` 作为一个索引值，也就是键 key，将 `name` 作为值 value，它可以是任意复杂结构。

```c
#include "uthash.h"

typedef struct {
    int id;                    /* key */
    char name[10];
    UT_hash_handle hh;         /* makes this structure hashable */
} HashNode;

HashNode *hashTbl = NULL;    /* important! initialize to NULL */
```

注意：结构中一定要包含 `UT_hash_handle hh`（`hh`不需要初始化）。它可以命名为任何名称，但是一般都命名为 `hh`。

### 添加

- `HASH_ADD_INT`：表示添加的键值为整型；

- `HASH_ADD_STR`：表示添加的键值为字符串类型；

- `HASH_ADD_PTR`：表示添加的键值为指针类型；

- `HASH_ADD`：表示添加的键值可以是任意类型。

```c
void add_user(int user_id, char *name) {
    HashNode *hashNode = NULL;

    /*添加前先进行重复性检查，因为当把两个相同key值的结构体添加到哈希表中时会报错*/
    HASH_FIND_INT(hashTbl, &user_id, hashNode);  /* id already in the hash? */

    /*只有在哈希中不存在ID的情况下，才创建该项目并将其添加。否则，只修改已经存在的结构*/
    if (hashNode == NULL) {
      hashNode = (HashNode *)malloc(sizeof(HashNode));
      hashNode->id = user_id;
      HASH_ADD_INT(hashTbl, id, hashNode);  /* id: name of key field */
    }
    strcpy(hashNode->name, name);
}
```

`HASH_ADD_INT` 函数中，

-   第一个参数 `hashTbl` 是哈希表
-   第二个参数 `id` 是键字段的名称
-   第三个参数 `hashNode` 是指向要添加的结构的指针

### 将指向哈希指针的指针传递给函数

在上面的例子中 `hashTbl` 是一个全局变量，但是如果调用者想将哈希指针传递给函数 `add_user` 怎么办？乍一看，您似乎可以简单地将 `hashTbl` 作为参数传递，但这行不通。

```c
/* bad */
void add_user(HashNode *obj, int user_id, char *name) {
  // ...
  HASH_ADD_INT(obj, id, hashNode);
}
```

正确地，你需要**传递一个指向哈希指针的指针**（a pointer to the hash pointer）。

```c
/* good */
void add_user(HashNode **obj, int user_id, char *name) {
  ...
  HASH_ADD_INT(*obj, id, hashNode);
}
```

请注意，我们也取消了 `HASH_ADD` 中的指针引用。

**必须处理指向哈希指针的指针的原因很简单：散列宏修改它（换句话说，它们修改指针地址本身，而不仅仅是它指向的内容）**。

The reason it’s necessary to deal with a pointer to the hash pointer is simple: the hash macros modify it (in other words, they modify the _pointer itself_ not just what it points to).

### 查找
```c
HashNode *find_user(int user_id) {
    HashNode *hashNode;
    HASH_FIND_INT(hashTbl, &user_id, hashNode);  /* hashNode: output pointer */
    return hashNode;
}
```
HASH_FIND_INT函数中，第一个参数users是哈希表，第二个参数是user_id的地址（一定要传递地址）。最后s是输出变量。当可以在哈希表中找到相应键值时，s返回给定键的结构，当找不到时s返回NULL。

### 替换
```c
void replace_user(HashHead *head, HashNode *newNode) {
    HashNode *oldNode = find_user(*head, newNode->id);
    if (oldNode) {
        HASH_REPLACE_INT(*head, id, newNode, oldNode);
    }
}
```
HASH_REPLACE宏等效于HASH_ADD宏，HASH_REPLACE会尝试查找和删除项目，如果找到并删除了一个项目，它将返回该项目的指针作为输出参数。

### 删除
```c
void delete_user(HashNode *user) {
    HASH_DEL(hashTbl, user);  /* user: pointer to delete */
    free(user);             /* optional; it'hashNode up to you */
}
```
要从哈希表中删除结构，必须具有指向它的指针（如果只有键，请先执行HASH_FIND以获取结构指针）。这里users是哈希表，user是指向我们要从哈希中删除的结构的指针。删除结构只是将其从哈希表中删除，并非free，何时释放结构的选择完全取决于自己；uthash永远不会释放您的结构。

> 源码地址：https://github.com/troydhanson/uthash
> 
> 英文原版用户指导：https://troydhanson.github.io/uthash/userguide.html

