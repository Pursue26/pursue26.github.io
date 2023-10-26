---
title: LeetCode刷题之206反转链表
date: 2023-10-25 18:59:47
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
    - 双指针
    - 递归
abbrlink: 231025185947
---

[206 反转链表](https://leetcode-cn.com/problems/reverse-linked-list/)：反转一个单链表。例如，输入：`1->2->3->4->5->NULL`；输出：`5->4->3->2->1->NULL`。

<!--more-->

## LIFO堆栈解法

可以使用堆栈的后进先出特性来实现链表的反转。遍历链表中的节点，并存储在堆栈中；遍历完后，依次弹出堆栈中的节点，并将它与上一个节点链接起来。

时间复杂度：`O(n)`，空间复杂度：`O(n)`。

## 双指针解法

可以借助一前一后两个指针，在遍历链表的过程中，将两个前后两个节点的指向关系反转。

```
1  2  3  4  5  NULL
 -> -> -> -> ->
 <- <- <- <- <-
```

时间复杂度：`O(n)`，空间复杂度：`O(1)`。

```c
struct ListNode* reverseList(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }

    struct ListNode *pre = head, *cur = head->next;  // 双指针
    struct ListNode *suff = NULL, *tail = NULL;  // 临时指针和结果指针
    pre->next = NULL;
    while (cur != NULL) {
        if (cur->next == NULL){
            tail = cur;  // 记录最后一个节点
        }
        suff = cur->next;  // 记录下一个节点
        cur->next = pre;  // 转向关系反转
        pre = cur;  // 更新上一个节点
        cur = suff;  // 更新当前节点
    }
    return tail;
}
```

## 头插法

对于链表中的每一个节点，我们**都将它插在新链表的头结点之前**，对于链表 `1->2->3->4->5->NULL`：
1. 遍历前新链表为 `NULL`；
2. 遍历第一个节点时，将其插入在新链表 `NULL` 的头结点之前，更新后的链表为 `1->NULL`;
3. 遍历第二个节点时，将其插入在新链表 `1->NULL` 的头结点之前，更新后的链表为 `2->1->NULL`;
4. ...
5. 遍历最后一个节点时，将其插入在新链表 `4->3->2->1->NULL` 的头结点之前，更新后的链表为 `5->4->3->2->1->NULL`;

时间复杂度：`O(n)`，空间复杂度：`O(1)`。

```c
struct ListNode* reverseList(struct ListNode* head){
    struct ListNode* newHead = NULL;

    while (head != NULL) {
        struct ListNode *cur = head;  // 当前节点
        head = head->next;  // 先找到下一个节点
        cur->next = newHead;  // 当前节点指向新链表的头
        newHead = cur;  // 更新新链表的头
    }
    return newHead;
}
```

> 双指针解法与头插法有什么区别？

双指针解法是反转前后两个节点的指向关系；头插法是初始化一个新的空链表，依次将待反转链表的节点，追加在新链表的最前面。

## 递归解法

假设我们要反转链表 `1->2->3->4->5->NULL`，可以先反转节点 `1` 后面**更短的链表**，即 `2->3->4->5->NULL`，这会得到 `5->4->3->2->NULL`。最后，再将节点 `1` 链接到上面反转的链表的最后一个非空节点（即节点 `2`）和 `NULL` 节点之间。

递归的终止条件就是链表无需反转了，即链表没有节点或只有一个节点。

时间复杂度：`O(n)`，空间复杂度：`O(n)`，递归深度可能达到 `n` 层。

```c
struct ListNode* reverseList(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }
    struct ListNode* cur = reverseList(head->next);
    head->next->next = head;
    head->next = NULL;
    return cur;
}
```

这段代码是用递归方式实现单链表的反转。让我们逐行解析代码的实现原理：

1. 首先，检查头指针 `head` 是否为空或者链表只有一个节点（即 `head->next` 为空）。如果是的话，直接返回头指针。
2. 如果链表有多个节点，递归调用 `reverseList` 函数，传入 `head->next` 作为参数，**目的是将 `head->next` 节点及之后的节点进行反转**。
3. 在递归调用之后，我们得到了反转后的链表的头指针 `cur`，它的尾结点是 `head->next`。
4. 然后，我们**将 `head` 节点插入到反转后的链表的末尾**，即 `head->next->next = head`。这一步是将 `head` 节点的下一个节点（`head->next`）指向 `head`，实现反转。
5. 最后，将 `head` 的下一个节点指向 `NULL`，以确保链表的末尾指向 `NULL`。
6. 返回反转后的链表的头指针 `cur`。

这段代码的核心思想是通过递归来实现链表的反转。通过不断地调用 `reverseList` 函数，**每次反转两个节点**，最终实现整个链表的反转。

