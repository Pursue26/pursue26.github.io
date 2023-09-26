---
title: C语言之uthash哈希表基础
date: 2023-09-22 17:26:26
categories:
    - 学习笔记
    - C语言
tags:
    - 哈希表
abbrlink: 230922172626
---

由于 C 语言本身不存在哈希，但当需要使用哈希表的时候，自己构建又会异常复杂。因此，我们可以调用开源的第三方库 `uthash.h`，**这只是一个头文件**。我们需要做的就是将头文件通过 `#include "uthash.h"` 引入到自己的项目中。由于 `uthash` 仅是头文件，因此没有可链接的库代码。

<!--more-->

## uthash简介

使用 `uthash` 添加、查找和删除通常是常数时间的操作，此哈希库的目标是简约、高效。它大约有 1000 行 C 代码，它会*自动内联*，因为它是作为宏实现的。

`uthash` 还包括三个额外的头文件，主要提供链表、动态数组和动态字符串。
-   `utlist.h` 为 C 结构提供了链接列表宏。
-   `utarray.h` 使用宏实现动态数组。
-   `utstring.h` 实现基本的动态字符串。

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
} HashItem;

HashItem *hashTbl = NULL;    /* important! initialize to NULL */
```

注意：结构中一定要包含 `UT_hash_handle hh`（`hh`不需要初始化）。它可以命名为任何名称，但是一般都命名为 `hh`。

### 添加

- `HASH_ADD_INT`：表示添加的键值为整型，参数为 `(head, keyfield_name, item_ptr)`；

- `HASH_ADD_STR`：表示添加的键值为字符串类型，参数为 `(head, keyfield_name, item_ptr)`；

- `HASH_ADD_PTR`：表示添加的键值为指针类型，参数为 `(head, keyfield_name, item_ptr)`；

- `HASH_ADD`：表示添加的键值可以是任意类型，参数为 `(hh_name, head, keyfield_name, key_len, item_ptr)`。

这些常见的宏的参数可以看[这里](https://troydhanson.github.io/uthash/userguide.html#_convenience_macros)，一般性的宏（如`HASH_ADD`）的参数可以看[这里](https://troydhanson.github.io/uthash/userguide.html#_general_macros)。

```c
void add_user(int user_id, char *name) {
    HashItem *hashNode = NULL;

    /* 添加前先进行重复性检查，因为当把两个相同key值的结构体添加到哈希表中时会报错 */
    HASH_FIND_INT(hashTbl, &user_id, hashNode);  /* id already in the hash? */

    /* 只有在哈希中不存在ID的情况下，才创建该项目并将其添加；否则，只修改已经存在的结构 */
    if (hashNode == NULL) {
      hashNode = (HashItem *)malloc(sizeof(HashItem));
      hashNode->id = user_id;
      HASH_ADD_INT(hashTbl, id, hashNode);  /* id: name of key field */
    }
    strcpy(hashNode->name, name);
}
```

在 `HASH_ADD_INT` 函数中，

-   第一个参数 `hashTbl` 是哈希表；
-   第二个参数 `id` 是键字段的名称；
-   第三个参数 `hashNode` 是指向要添加的结构的指针。

### 将指向哈希指针的指针传递给函数（重要）

在上面的例子中 `hashTbl` 是一个全局变量，但是如果调用者想将哈希指针传递给函数 `add_user` 怎么办？乍一看，您似乎可以简单地将 `hashTbl` 作为参数传递，但这行不通。

```c
/* bad */
void add_user(HashItem *obj, int user_id, char *name) {
  // ...
  HASH_ADD_INT(obj, id, hashNode);
}
```

正确地，你需要**传递一个指向哈希指针的指针**（a pointer to the hash pointer）。

```c
/* good */
void add_user(HashItem **obj, int user_id, char *name) {
  ...
  HASH_ADD_INT(*obj, id, hashNode);
}
```

**必须处理「指向哈希指针的指针」的原因很简单：散列宏会修改它（换句话说，它们修改指针地址本身，而不仅仅是它指向的内容）**。

The reason it’s necessary to deal with a pointer to the hash pointer is simple: the hash macros modify it (in other words, they modify the _pointer itself_ not just what it points to).

> 假设哈希表 `hashTbl` 的指针（地址）是 `0x7fffd69b9a10`，那么通过运算符 `&` 可以得到存放该地址的地址，假如为 `0x8defd69b9a26`，那么后续散列宏修改了哈希指针后，我们还可以通过地址 `0x8defd69b9a26` 指向的内容（哈希表的地址）来获取最新的哈希表地址。

### 查找

```c
HashItem *find_user(int user_id) {
    HashItem *hashNode = NULL;
    HASH_FIND_INT(hashTbl, &user_id, hashNode);  /* hashNode: output pointer */
    return hashNode;
}
```

在 `HASH_FIND_INT` 函数中，

-   第一个参数 `hashTbl` 是哈希表；
-   第二个参数是 `user_id` 的地址（一定要传递地址）；
-   第三个参数 `hashNode` 是输出变量。

当可以在哈希表中找到相应键值时，返回给定键的结构到 `hashNode`，当找不到时返回 `NULL` 到 `hashNode`。也就是说可以通过判断返回值是否为 `NULL` 来判断查找的键值是否存在于哈希表中。

### 替换

```c
void replace_user(HashItem *obj, HashItem *newHashNode) {
    HashItem *oldHashNode = find_user(newHashNode->id);
    if (oldHashNode) {
        HASH_REPLACE_INT(hashTbl, id, newHashNode, oldHashNode);
    }
}
```

`HASH_REPLACE` 宏等效于 `HASH_ADD` 宏，`HASH_REPLACE` 会尝试查找和删除项目，如果找到并删除了一个项目，它将返回该项目的指针作为输出参数。

### 删除

```c
void delete_user(HashItem *hashNode) {
    HASH_DEL(hashTbl, hashNode);  /* user: pointer to delete */
    free(hashNode);             /* optional; it's up to you */
}
```

1. 要从哈希表中删除结构，必须具有指向它的指针（如果只有键值，应该先执行 `HASH_FIND` 以获取结构的指针）。

2. 这里 `hashTbl` 是哈希表，`hashNode` 是指向我们要从哈希表中删除的结构的指针。删除结构只是将其从哈希表中删除，并非 `free`，何时释放结构的选择完全取决于自己，`uthash` 永远不会释放您的结构。

### 迭代删除

上面的「删除」只能从哈希表中删除指定的一个结构，如果想将所有结构从哈希表中删除，可以使用迭代删除操作。

`HASH_ITER` 宏是一个删除安全的迭代构造，它扩展为一个简单的 `for` 循环。

```c
void delete_all() {
    HashItem *curr = NULL, *tmp = NULL;
    HASH_ITER(hh, hashTbl, curr, tmp) {
        HASH_DEL(hashTbl, curr);  /* delete; users advances to next */
        free(curr);               /* optional- if you want to free  */
    }
}
```

#### 一次性删除

如果你只想删除所有的结构（哈希结点），而不释放它们的内存空间或进行任何逐个元素的清理操作，你可以使用单个操作更高效地完成这个任务。

```c
HASH_CLEAR(hh, hashTbl);
```

之后，列表头（这里是 `hashTbl`）将被设置为 `NULL`。

> 在 uthash 中，ut 是 unordered tables，即无序表。
> 
> 上面全大写的宏，为了方便，有些也被我叫成了函数。
> 
> 上面的代码，未做充分的指针为空判断，实际使用指针前，要先进行不为空判断。

## uthash的实践

以 [LeetCode两数之和](https://leetcode.cn/problems/two-sum/)为例，介绍 `uthash.h` 哈希表的使用。

题目大意：从一个数组中找出两个索引不同的数，使得它们的和等于目标值，并返回这两个数。

```c
#include "uthash.h"

typedef struct {
    int key;
    int val;
    UT_hash_handle hh;
} HashItem;

HashItem *hashFindItem(HashItem **obj, int key) {
    HashItem *pEntry = NULL;
    HASH_FIND_INT(*obj, &key, pEntry);
    return pEntry;
}

void hashAddItem(HashItem **obj, int key, int val) {
    HashItem *pEntry = hashFindItem(obj, key);
    if (pEntry == NULL) {
        pEntry = (HashItem *)malloc(sizeof(HashItem));
        pEntry->key = key;
        HASH_ADD_INT(*obj, key, pEntry);
    }
    pEntry->val = val;
}

void hashFree(HashItem **obj) {
    HashItem *curr = NULL, *tmp = NULL;
    HASH_ITER(hh, *obj, curr, tmp) {
        HASH_DEL(*obj, curr);  
        free(curr);
    }
}

int *twoSum(int* nums, int numsSize, int target, int* returnSize){
    HashItem *hashTbl = NULL;

    for (int i = 0; i < numsSize; ++i) {
        // 重要：传入的是指向哈希指针的指针
        HashItem *hashNode = hashFindItem(&hashTbl, nums[i]);
        if (hashNode == NULL) {
            hashAddItem(&hashTbl, nums[i], i);
        }
    }

    for (int i = 0; i < numsSize; ++i) {
        HashItem *hashNode = hashFindItem(&hashTbl, target - nums[i]);
        if (hashNode && hashNode->val != i) {
            *returnSize = 2;
            int *ans = (int *)malloc((*returnSize) * sizeof(int));
            ans[0] = i, ans[1] = hashNode->val;
            return ans;
        }
    }
    *returnSize = 0;
    return NULL;
}
```

上面代码中，我们定义了哈希表的查找、添加、释放接口，接口中的第一个参数均为指向哈希表指针的指针（二级指针），这是必须的。

1. 首先，在进行哈希操作前，先定义了一个哈希表 `hashTbl`，他是一个 `HashItem` 类型的指针；
2. 然后，将数组中的键-值对（索引对应的值-索引）依次添加到哈希表中，并使用操作符 `&` 获取指向哈希指针的指针；
3. 最后，经过查找后，不再使用哈希表，这时通过释放接口，将哈希表中的所有哈希结点迭代地删除并释放对应的内存空间。

如果哈希表的键值不是整形，而是字符类型呢，应该怎么使用呢？可以参考[这里](https://pursue26.github.io/posts/230925185057.html#%20哈希表实现的字典树)的一个例子。

> 源码地址：https://github.com/troydhanson/uthash

> 英文原版用户指导：https://troydhanson.github.io/uthash/userguide.html

