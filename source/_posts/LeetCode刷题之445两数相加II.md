---
title: LeetCode刷题之445两数相加II
date: 2023-10-31 16:31:33
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
abbrlink: 231031163133
---

[445 两数相加 II](https://leetcode-cn.com/problems/add-two-numbers-ii/)：给你两个非空链表来代表两个非负整数。**数字最高位位于链表开始位置**。它们的每个节点只存储一位数字。将这两数相加会返回一个新的链表。你可以假设除了数字 0 之外，这两个数字都不会以零开头。进阶：如果输入链表不能修改该如何处理？换句话说，你不能对列表中的节点进行翻转。

<!--more-->

```
输入：l1 = [7,2,4,3], l2 = [5,6,4]
输出：[7,8,0,7]
```

## 反转链表再遍历

其实我们可以利用 [LeetCode刷题之206反转链表](https://pursue26.github.io/posts/231025185947.html)和 [LeetCode刷题之2两数相加](https://pursue26.github.io/posts/231031120718.html)完成该题。

首先，将链表 `l1` 和链表 `l2` 反转，然后调用两数相加的代码，最后再对返回的链表再次反转即可。

时间复杂度：`O(max(m, n))`，空间复杂度：`O(1)`，不考虑输出占用的空间；递归调用栈的空间复杂度：`O(max(m, n))`，如果采用迭代的方式反转链表，则为常数空间。

```c
struct ListNode* reverseList(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return head;
    }
    struct ListNode* tail = reverseList(head->next);
    head->next->next = head;
    head->next = NULL;
    return tail;
}

struct ListNode* addTwoNumbers_i(struct ListNode* l1, struct ListNode* l2) {
    struct ListNode dummy;
    struct ListNode *cur = &dummy;
    int curSum = 0, carry = 0;

    while (l1 || l2 || carry > 0) {  // 把最后一次加法是否有进位放在这里, 简化代码
        // 创建节点并指向NULL
        struct ListNode *node = (struct ListNode *)malloc(sizeof(struct ListNode));
        node->next = NULL;

        // 计算当前两节点相加的和, 加上进位
        curSum = carry;
        if (l1) {
            curSum += l1->val;
            l1 = l1->next;
        }
        if (l2) {
            curSum += l2->val;
            l2 = l2->next;
        }

        // 计算本次加法的进位和相加值
        carry = curSum > 9 ? 1 : 0;
        curSum = curSum % 10;
        
        // 将下一个节点指向新节点, 并修改其值为相加值
        node->val = curSum;
        cur->next = node;
        cur = cur->next;
    }
    return dummy.next;
}

struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2){
    l1 = reverseList(l1);
    l2 = reverseList(l2);
    struct ListNode* head = addTwoNumbers_i(l1, l2);
    return reverseList(head);
}
```

## LIFO堆栈

> 进阶：如果输入链表不能修改该如何处理？换句话说，你不能对列表中的节点进行翻转。

**记住：如果需要考虑链表反转，首先考虑栈（后进先出）**。

因此，我们可以申请两个栈空间，依次遍历两个链表，将节点按遍历顺序放入各自的堆栈中。然后，依次同时从两个堆栈中各弹出一个节点，进行两数相加，将相加形成的新节点，放入第三个堆栈中，直到相加操作结束。最后，依次弹出第三个堆栈中的节点，并将它们链接在一起，即为答案。

堆栈的实现：我们可以使用[数据结构之堆栈（数组实现）](https://pursue26.github.io/posts/231016161508.html)或[数据结构之堆栈（链表实现）](https://pursue26.github.io/posts/231016184406.html)中实现的堆栈数据结构。但是，为了简便，这里使用了普通数组来模拟堆栈。

时间复杂度：`O(max(m, n))`，空间复杂度：`O(max(m, n))`，即堆栈占用的空间，不考虑输出占用的空间。

```c
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2){
    struct ListNode *stack1[101], *stack2[101], *stack3[101];

    struct ListNode *node1 = l1;
    struct ListNode *node2 = l2;
    int pos1 = 0, pos2 = 0, pos3 = 0;
    while (node1) {
        stack1[pos1++] = node1;
        node1 = node1->next;
    }
    while (node2) {
        stack2[pos2++] = node2;
        node2 = node2->next;
    }
    pos1--;  // 变成索引值
    pos2--;

    int curSum = 0, carry = 0;
    while (pos1 >= 0 || pos2 >= 0 || carry > 0) {
        // 创建节点并指向NULL
        struct ListNode *node = (struct ListNode *)malloc(sizeof(struct ListNode));
        node->next = NULL;

        // 计算当前两节点相加的和, 加上进位
        curSum = carry;
        if (pos1 >= 0) {
            curSum += stack1[pos1]->val;
            pos1--;
        }
        if (pos2 >= 0) {
            curSum += stack2[pos2]->val;
            pos2--;
        }

        // 计算本次加法的进位和相加值
        carry = curSum > 9 ? 1 : 0;
        curSum = curSum % 10;

        // 相加和入第三个堆栈
        node->val = curSum;
        stack3[pos3++] = node;
    }

    struct ListNode *ans = stack3[--pos3];
    // 依次弹出第三个堆栈中的节点
    while (pos3 > 0) {
        stack3[pos3]->next = stack3[pos3 - 1];
        pos3--;
    }
    stack3[pos3]->next = NULL;
    
    return ans;
}
```
