---
title: LeetCode刷题之725分隔链表
date: 2023-11-01 13:52:28
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
abbrlink: 231101135228
---

[725 分隔链表](https://leetcode-cn.com/problems/split-linked-list-in-parts/)：给定一个头结点为 `root` 的链表, 编写一个函数以将链表分隔为 k 个连续的部分。每部分的长度应该尽可能的相等: 任意两部分的长度差距不能超过 1，也就是说可能有些部分为 `null`。这 k 个部分应该按照在链表中出现的顺序进行输出，并且排在前面的部分的长度应该大于或等于后面的长度。返回一个由上述 k 部分组成的数组。

<!--more-->

```
输入：head = [1,2,3], k = 5
输出：[[1],[2],[3],[],[]]
输入：head = [1,2,3,4,5,6,7,8,9,10], k = 3
输出：[[1,2,3,4],[5,6,7],[8,9,10]]
```

函数接口：
```c
struct ListNode** splitListToParts(struct ListNode* head, int k, int* returnSize);
```

## 两趟遍历

该题就是正常的遍历，然后划分即可。首先，一次遍历计算链表的长度 `size`；然后，根据链表长度 `size` 和 `k` 值，计算划分的链表的长度，前 `size % k` 段的长度为 `size / k + 1`，后 `size - size % k` 段的长度为 `size / k`。

时间复杂度：`O(n)`，两趟遍历，空间复杂度：`O(1)`，不算输出占用的空间。

```c
struct ListNode** splitListToParts(struct ListNode* head, int k, int* returnSize){
    struct ListNode *node = head;
    int size = 0;

    // 计算链表长度
    while (node) {
        size++;
        node = node->next;
    }
    node = head;

    // 申请返回的空间, 并全部初始化为NULL
    struct ListNode **ans = (struct ListNode **)malloc(k * sizeof(struct ListNode *));
    memset(ans, 0, k * sizeof(struct ListNode *));
    
    int a = size / k, b = size % k;
    int splitSize = 0, idx = 0;
    bool save = true;
    while (node) {
        // 存储划分的每段链表的头节点
        if (save) {
            ans[idx++] = node;
            save = false;
        }

        splitSize++;
        // 前b次划分的链表长度为a+1, 后size-b次划分的链表长度为a
        if (splitSize < a + (b > 0 ? 1 : 0)) {
            node = node->next;
        } else {
            splitSize = 0;
            b = fmax(--b, 0);  // b减到0停止
            save = true;
            struct ListNode *tmp = node;
            node = node->next;
            tmp->next = NULL;
        }
    }
    *returnSize = k;
    return ans;
}
```
