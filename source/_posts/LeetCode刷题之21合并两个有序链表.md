---
title: LeetCode刷题之21合并两个有序链表
date: 2023-10-26 10:12:39
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
    - 双指针
    - 递归
abbrlink: 231026101239
---

[21 合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)：输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

<!--more-->

```
示例：
    链表 A：1->2->4
    链表 B：1->3->4
    合并链表：1->1->2->3->4->4
```

## 双指针解法

使用两个指针遍历链表，分别指向链表 A 和链表 B 的头结点，然后比较两个指针所指向的节点的大小关系，将较小的节点链接到合并链表的末尾，并后移对应的指针；直到其中一个指针为空时，将另一个非空指针指向的剩余链表拼接到合并链表的末尾。

时间复杂度：`O(m+n)`，空间复杂度：`O(1)`。

```c
struct ListNode* mergeTwoLists(struct ListNode* list1, struct ListNode* list2){
    if (list1 == NULL) {
        return list2;
    } else if (list2 == NULL) {
        return list1;
    }

    struct ListNode *head = NULL;

    // 根据大小关系确定合并链表的头结点, 并确定每个链表的起始位置
    if (list1->val <= list2->val) {
        head = list1;
        list1 = list1->next;
    } else {
        head = list2;
        list2 = list2->next;
    }

    struct ListNode *cur = head;

    while (list1 && list2) {
        // 比较两个指针所指向的节点的大小关系，将较小的节点链接到合并链表的末尾
        if (list1->val <= list2->val) {
            cur->next = list1;
            list1 = list1->next;
        } else {
            cur->next = list2;
            list2 = list2->next;
        }
        cur = cur->next;
    }
    // 将非空指针指向的剩余链表拼接到合并链表的末尾
    cur->next = (list1 == NULL ? list2 : list1);
    
    return head;
}
```

> 这里的双指针直接复用的两个链表指针变量。

上面代码虽然清晰，但是有点长了。其实，我们可以**利用一个临时的「哑结点」作为合并链表的头结点的上一个节点**，来简化操作。

这样，就不需要上面代码中 `while` 循环前的「根据大小关系确定合并链表的头结点, 并确定每个链表的起始位置」判断了。

```c
struct ListNode* mergeTwoLists(struct ListNode* list1, struct ListNode* list2){
    if (list1 == NULL) {
        return list2;
    } else if (list2 == NULL) {
        return list1;
    }

    struct ListNode dummy;  // 哑结点
    struct ListNode *cur = &dummy;

    while (list1 && list2) {
        // 比较两个指针所指向的节点的大小关系，将较小的节点链接到合并链表的末尾
        if (list1->val <= list2->val) {
            cur->next = list1;
            list1 = list1->next;
        } else {
            cur->next = list2;
            list2 = list2->next;
        }
        cur = cur->next;
    }
    // 将非空指针指向的剩余链表拼接到合并链表的末尾
    cur->next = (list1 == NULL ? list2 : list1);
    
    return dummy.next;  // 返回哑结点的下一个节点, 即为合并链表的头结点
}
```

## 递归解法

我们同样可以利用递归实现合并两个链表。

```c
struct ListNode* mergeTwoLists(struct ListNode* list1, struct ListNode* list2){
    if (list1 == NULL) {
        return list2;
    } else if (list2 == NULL) {
        return list1;
    }

    if (list1->val <= list2->val) {
        list1->next = mergeTwoLists(list1->next, list2);
        return list1;
    } else {
        list2->next = mergeTwoLists(list1, list2->next);
        return list2;
    }
}
```

上面这种写法，跟双指针解法中的第一个代码块相似，直接在原有的链表上进行操作，并直接返回较小节点的头节点作为合并后的链表头。

同样地，我们也可以像双指针解法中的第二个代码块那样，利用一个哑结点，将哑结点的下一个节点指向链表 A 和链表 B 中较小的节点 N，并递归地将较小的节点 N 指向后续合并链表的头结点。

```c
struct ListNode* mergeTwoLists(struct ListNode* list1, struct ListNode* list2){
    if (list1 == NULL) {
        return list2;
    } else if (list2 == NULL) {
        return list1;
    }

    struct ListNode dummy;
    struct ListNode *cur = &dummy;

    if (list1->val <= list2->val) {
        cur->next = list1;
        list1->next = mergeTwoLists(list1->next, list2);
    } else {
        cur->next = list2;
        list2->next = mergeTwoLists(list1, list2->next);
    }
    cur = cur->next;

    return dummy.next;
}
```
