---
title: LeetCode刷题之2两数相加
date: 2023-10-31 12:07:18
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
abbrlink: 231031120718
---

[2 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)：给出两个**非空**的链表用来表示两个非负的整数。其中，它们各自的位数是按照**逆序**的方式存储的，并且它们的每个节点只能存储**一位**数字。如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

<!--more-->

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

## 一次遍历（哑结点）

对于待相加的两个链表中的节点，它们相加可能出现进位（相加的值大于 9）。这时在下次相加时就要考虑这个带进位的结果；同时，最后一次相加的结果可能有进位，也可能没有进位，这也是不能忽略的。

每次相加都要创建一个链表节点，用于存储相加的值，并把它们链接起来，形成题目要求的返回的新链表。

时间复杂度：`O(max(m, n))`，空间复杂度：`O(1)`，不考虑输出占用的空间。

```c
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
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
    // // 判断最后一次加法是否有进位, 有责增加一个节点存储进位值
    // if (carry > 0) {
    //     struct ListNode *node = (struct ListNode *)malloc(sizeof(struct ListNode));
    //     node->next = NULL;
    //     node->val = carry;
    //     cur->next = node;
    //     cur->next->next = NULL;
    // }
    return dummy.next;
}
```

