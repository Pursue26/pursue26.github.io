---
title: LeetCode刷题之19删除链表的倒数第N个节点
date: 2023-10-27 15:02:43
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 快慢指针
abbrlink: 231027150243
---

[19 删除链表的倒数第 N 个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)：给定一个链表，删除链表的倒数第 N 个节点，并且返回链表的头节点。进阶：你能尝试使用一趟扫描实现吗？

<!--more-->

```
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

## 两趟遍历

删除倒数第 `n` 个，即需要统计链表长度 `L`，然后删除正数第 `L-n+1` 个即可。

第一趟遍历：统计链表长度；第二趟遍历：删除节点。

时间复杂度：`O(L)`，遍历了两遍链表，空间复杂度：`O(1)`。

```c
struct ListNode* removeNthFromEnd(struct ListNode* head, int n){
    struct ListNode *node = head;
    int size = 0;
    while (node) {
        node = node->next;
        size++;
    }
    node = head;

    // 删除的是第一个节点
    if (n == size) {
        struct ListNode *newHead = head->next;
        head->next = NULL;
        free(head);
        return newHead;
    }

    for (int i = 0; i < size - n - 1; ++i) {
        node = node->next;
    }
    struct ListNode *temp = node->next;
    node->next = node->next->next;
    temp->next = NULL;
    free(temp);
    
    return head;
}
```

## 快慢指针（一趟遍历）

进阶：你能尝试使用一趟扫描实现吗？

我们可以使用两个指针（快慢指针），让快指针先走 `n` 步，然后快指针和慢指针同时、同速度走，当快指针走完时，慢指针刚好走了 `L-n` 步。

时间复杂度：`O(L)`，遍历了一遍链表，空间复杂度：`O(1)`。

```c
struct ListNode* removeNthFromEnd(struct ListNode* head, int n){
    struct ListNode *fast = head;
    struct ListNode *slow = head;
    
    for (int i = 0; i < n; ++i) {
        fast = fast->next;
    }

    // 删除的是第一个节点
    if (fast == NULL) {
        struct ListNode *newHead = head->next;
        head->next = NULL;
        free(head);
        return newHead;
    }

    while (fast->next) {
        fast = fast->next;
        slow = slow->next;
    }
    struct ListNode *temp = slow->next;
    slow->next = slow->next->next;
    temp->next = NULL;
    free(temp);
    
    return head;
}
```
