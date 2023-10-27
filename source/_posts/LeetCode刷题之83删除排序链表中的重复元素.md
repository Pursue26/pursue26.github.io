---
title: LeetCode刷题之83删除排序链表中的重复元素
date: 2023-10-26 18:45:14
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
    - 双指针
    - 递归
abbrlink: 231026184514
---

[83 删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)：给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。

<!--more-->

```
输入: 1->1->2->3->3
输出: 1->2->3
```

## 双指针解法

我们可以使用**一前、一后两个指针，找到前、后相邻的两个值不相等的节点，将它们链接起来**。然后，将「前指针」重新指向「后指针」，「后指针」往后移动，继续找前、后相邻的两个值不相等的节点...直到后指针为空为止。

时间复杂度：`O(n)`，空间复杂度：`O(1)`。

```c
struct ListNode* deleteDuplicates(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }

    struct ListNode *left = head;
    struct ListNode *right = head->next;

    while (right != NULL) {
        if (left->val == right->val) {
            struct ListNode *temp = right;
            right = right->next;
            // 释放节点空间
            temp->next = NULL;
            free(temp);
        } else {
            // 将它们链接起来
            left->next = right;
            // 将前指针重新指向后指针，后指针往后移动
            left = right;
            right = right->next;
        }
    }
    left->next = NULL;
    return head;
}
```

为什么退出前还有一行 `left->next = NULL` 代码？
这是因为，如果原链表最后是以重复的元素退出 `while` 循环的，则 `left` 就是最后一个节点，但 `left->next` 此时还不是 `NULL`，所以需要这一行代码。

## 单指针解法

其实，我们也可以只使用一个指针，这个指针指向当前节点，只要「下一个节点」与当前节点的值相等，我们就更新当前节点的**下一个节点**为「再」下一个节点；遇到不相等时，才**更新当前指针**为「下一个节点」。

时间复杂度：`O(n)`，空间复杂度：`O(1)`。

```c
struct ListNode* deleteDuplicates(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }

    struct ListNode *cur = head;
    while (cur->next != NULL) {
        // 「下一个节点」与当前节点的值相等
        if (cur->val == cur->next->val) {
            struct ListNode *temp = cur->next;
            // 更新当前节点的下一个节点为「再」「下一个节点」
            cur->next = cur->next->next;
            // 释放节点空间
            temp->next = NULL;
            free(temp);
        } else {
            // 不相等时，才更新当前指针为「下一个节点」
            cur = cur->next;
        }
    }
    return head;
}
```

## 递归解法

假设，函数 `deleteDuplicates(head)` 已经了实现删除链表 `head` 中的重复节点，并返回删除后的链表的头节点。

那么，我们以 `head->next` 为头节点调用函数 `deleteDuplicates(head->next)`，将实现以 `head->next` 为头节点的**子链表**的去重操作，这时只需要将 `head` 的下一个节点指向这个去重的链表，就可以实现对**完整链表**的去重操作了。

但是，可能会遇到 `head` 和它的下一个节点 `head->next` 的值相等的时候，这是就需要**修正头节点的地址** —— 抛弃前面的节点。

递归的终止条件就是，链表为空或者只有一个节点。因为这时不需要进行去重操作。

时间复杂度：`O(n)`，空间复杂度：`O(n)`。

一种写法：先递归处理下一个节点，然后再判断下一个节点的值是否与当前节点的值相等，如果相等则删除当前节点，重新指定头节点。

```c
struct ListNode* deleteDuplicates(struct ListNode* head) {
    if (head == NULL || head->next == NULL) {
        return head;
    }

    head->next = deleteDuplicates(head->next);

    if (head->next->val == head->val) {
        struct ListNode *temp = head;
        head = head->next;  // 修正头节点的地址
        temp->next = NULL;
        free(temp);
    }
    return head;
}
```

另一种写法：先判断下一个节点的值是否与当前节点的值相等，如果相等则先删除当前节点，然后递归地处理下一个节点，并更新头节点为删除重复节点之后的头节点；否则，说明相邻节点不重复，递归地处理下一个节点。

```c
struct ListNode* deleteDuplicates(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }
    
    if (head->next->val == head->val) {
        struct ListNode *temp = head->next;
        head->next = NULL;
        free(head);
        head = deleteDuplicates(temp);
    } else {
        head->next = deleteDuplicates(head->next);
    }
    return head;
}
```

> 不管哪种写法，针对链表中的重复节点，都是删除前面的节点，保留下最后一个节点。
