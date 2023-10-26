---
title: LeetCode刷题之160相交链表
date: 2023-10-25 14:39:47
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
    - 双指针
abbrlink: 231025143947
---

[160 相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists)：编写一个程序，找到两个无环单链表相交的起始节点，如果不存在交点则返回 `null`。要求时间复杂度为 `O(m+n)`，空间复杂度为 `O(1)`。

<!--more-->

```
A:          a1 → a2
                    ↘
                      c1 → c2 → c3
                    ↗
B:    b1 → b2 → b3
```

但是不会出现以下相交的情况，因为每个节点只有**一个** `next` 指针，也就只能有一个后继节点，而以下示例中节点 `c` 有两个后继节点。

```
A:          a1 → a2       d1 → d2
                    ↘  ↗
                      c
                    ↗  ↘
B:    b1 → b2 → b3        e1 → e2
```

## 暴力解法

两重循环，对于链表A的每一个节点，依次从头到尾遍历链表B的每一个节点，当链表A的当前节点与链表B的当前节点**首次**相同时，即为相交的起始节点，即可返回答案。

很显然，时间复杂度：`O(mn)`，空间复杂度：`O(1)`，不满足题意。

## 哈希表解法

遍历链表A时存储指针地址到哈希表中。遍历链表B时，判断当前节点是否在哈希表中，若在（**第一个**在的），即为相交的起始节点，即可返回答案。

很显然，时间复杂度：`O(m+n)`，空间复杂度：`O(m)`，不满足题意。

## 双指针解法

设链表 A 的长度为 a + c，链表 B 的长度为 b + c，其中 c 为尾部公共部分长度，**可知长度满足：a + c + b = b + c + a**。

当访问链表 A 的指针访问到链表尾部时，**令它从链表 B 的头部开始访问链表 B**；同样地，当访问链表 B 的指针访问到链表尾部时，**令它从链表 A 的头部开始访问链表 A**。

这样就能控制访问 A 和 B 两个链表的指针**能同时访问到相交点**。如果不存在交点，那么 a + b = b + a，以下实现代码中`nodeA`和`nodeB`会同时为`null`，从而退出循环。

时间复杂度：`O(m+n)`，空间复杂度：`O(1)`，满足题意。

C语言实现：

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode *getIntersectionNode(struct ListNode *headA, struct ListNode *headB) {
    struct ListNode *nodeA = headA, *nodeB = headB;
    while (nodeA != nodeB) {
        nodeA = (nodeA != NULL ? nodeA->next : headB);
        nodeB = (nodeB != NULL ? nodeB->next : headA);
    }
    return nodeA;
}
```

Python语言实现：

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None
class Solution:
    def getIntersectionNode(self, headA: ListNode, headB: ListNode) -> ListNode:
        nodeA, nodeB = headA, headB
        while(nodeA != nodeB):
            nodeA = headB if(nodeA == None) else nodeA.next
            nodeB = headA if(nodeB == None) else nodeB.next
        return nodeA
```
