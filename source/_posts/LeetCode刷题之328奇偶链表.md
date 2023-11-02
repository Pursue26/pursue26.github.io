---
title: LeetCode刷题之328奇偶链表
date: 2023-11-02 12:07:08
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
    - 双指针
abbrlink: 231102120708
---

[328 奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list/)：给定一个单链表，把所有的奇数位节点和偶数位节点分别链接在一起，并链接这两个链表（即奇数位的尾结点的下一个节点是偶数位的首节点）。请尝试使用原地算法完成。你的算法的空间复杂度应为 `O(1)`，时间复杂度应为 `O(n)`。

<!--more-->

```
输入: head = [1,2,3,4,5]
输出: [1,3,5,2,4]
输入: head = [2,1,3,5,6,4,7]
输出: [2,3,6,7,1,5,4]
```


## 数组+遍历

可以使用两个数组，一个数组按序存储奇数位节点，一个数组按序存储偶数位节点。最后，先遍历奇数数组，再遍历偶数数组，即可。

时间复杂度：`O(n)`，三趟遍历，空间复杂度：`O(n)`，用于临时存储链表中的节点。

```c
struct ListNode* oddEvenList(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }

    struct ListNode *node = head;
    int size = 0;
    while (node) {
        size++;
        node = node->next;
    }
    node = head;

    // 分配两个数组的空间, 使用位运算计算
    struct ListNode *odd[(size >> 1) + (size & 1)];
    struct ListNode *even[(size >> 1)];
    int oddIdx = 0, evenIdx = 0;
    size = 0;
    while (node) {
        if ((size & 1) == 0) {
            // 一个数组按序存储奇数位节点
            odd[oddIdx++] = node;
        } else {
            // 一个数组按序存储偶数位节点
            even[evenIdx++] = node;
        }
        size++;
        node = node->next;
    }

    // 先遍历奇数数组
    for (int i = 0; i < oddIdx; ++i) {
        if (i < oddIdx - 1) {
            odd[i]->next = odd[i + 1];
        } else {
            // 奇数位的尾结点的下一个节点是偶数位的首节点
            odd[i]->next = even[0];
        }
    }
    // 再遍历偶数数组
    for (int i = 0; i < evenIdx; ++i) {
        if (i < evenIdx - 1) {
            even[i]->next = even[i + 1];
        } else {
            even[i]->next = NULL;
        }
    }
    
    return head;
}
```

## 双指针交替遍历

可以使用奇指针和偶指针两个指针来进行模拟。首先，奇指针 `odd` 指向链表第一个节点，偶指针 `even` 指向链表第二个节点。那么，偶指针的下一个节点 `even->next` 就是下一个奇数节点 `odd = even->next`，这个奇数节点的下一个节点就是下一个偶数节点 `even = odd->next`。然后，我们将前后相邻的奇数位、偶数位节点分别链接起来。最后，再将最后一个奇数位节点链接到第一个偶数位节点即可。

```
     ------- 
    |       |
    1---2---3---4---5
     -/- -/-
        |       |
         -------

             -------
            |       |
    1---2---3---4---5
             -/- -/-
                |       |
                 -------
```

时间复杂度：`O(n)`，一趟遍历，空间复杂度：`O(1)`。

```c
struct ListNode* oddEvenList(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }

    struct ListNode *odd = head, *even = head->next;
    struct ListNode *evenHead = even;  // 记录第一个偶数位节点, 否则会因为破坏链表的链接关系, 无法找到这个节点

    while (odd->next && even->next) {
        // 临时记录奇数位和偶数位节点
        struct ListNode *oddTmp = odd;
        struct ListNode *evenTmp = even;
        // 找到下一个奇数位和偶数位节点
        odd = even->next;  // odd = odd->next->next;
        even = odd->next;  // even = even->next->next;
        // 分别链接前后相邻的奇数位和偶数位节点
        oddTmp->next = odd;
        evenTmp->next = even;
    }
    // 将最后一个奇数位节点链接到第一个偶数位节点
    odd->next = evenHead;
    
    return head;
}
```

在 `while` 循环中，`odd->next && even->next` 表示**至少还有未链接到奇数链表末尾的奇数位节点**，这个条件等价于 `odd->next && odd->next->next`。
