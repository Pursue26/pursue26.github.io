---
title: LeetCode刷题之234回文链表
date: 2023-11-01 09:42:19
categories: 
    - LeetCode
    - 链表
tags: 
    - LeetCode
    - 链表
    - 快慢指针
abbrlink: 231101094219
---

[234 回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/)：请判断一个链表是否为回文链表。所谓回文链表就是以链表中间为中心点两边对称。**进阶**：你能否用 `O(n)` 时间复杂度和 `O(1)` 空间复杂度解决此题？

<!--more-->

```
输入: 1->2->3->2->1; 输出: true
输入: 1->2->3->4->1; 输出: false
```

## 多次遍历

我们可以遍历链表中的每个节点，依次把它们存储在数组中，最后从数组的两头往中间判断链表是否是回文数组。

这就需要对链表进行两次遍历。第一次遍历，计算链表的长度，用于申请合适的数组空间；第二次遍历，向数组中存储链表节点。最后，我们需要对数组进行一次遍历，来判断链表是否是回文链表。

时间复杂度：`O(n)`，空间复杂度：`O(n)`。

```c
bool isPalindrome(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return true;
    }

    struct ListNode *node = head;
    int size = 0;
    // 计算链表的长度
    while (node) {
        size++;
        node = node->next;
    }
    
    struct ListNode *nums[size];
    node = head;
    size = 0;
    // 向数组中存储链表节点
    while (node) {
        nums[size++] = node;
        node = node->next;
    }

    // 从数组的两头往中间判断链表是否是回文链表
    int left = 0, right = size - 1;
    while (left < right) {
        if (nums[left++]->val != nums[right--]->val) {
            return false;
        }
    }
    return true;
}
```

## 快慢指针+反转链表

> 你能否用 `O(n)` 时间复杂度和 `O(1)` 空间复杂度解决此题？

可以使用快慢指针找到链表的中点，然后断开链表，并将前一段链表进行反转，然后同时遍历两段链表，来判断原链表是否是回文链表。

这样，就不需要额外的申请内存空间（但是会修改入参的内容）。

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

bool isPalindrome(struct ListNode* head){
    if (head == NULL || head->next == NULL) {
        return true;
    }

    // 快慢指针寻找链表中点两侧的节点
    // 1->2->2->1, 1->2->3->2->1
    //    ↑  ↑        ↑     ↑
    struct ListNode *fast = head, *slow = head;
    while (fast && fast->next) {
        fast = fast->next->next;
        if (fast && fast->next) {
            slow = slow->next;
        }
    }

    // 重新指定快指针的位置
    if (fast) {
        fast = slow->next->next;
    } else {
        fast = slow->next;
    }

    // 断开链表, 并反转链表
    slow->next = NULL;
    struct ListNode *reverseHead = reverseList(head);

    // 判断链表是否是回文链表
    while (fast) {
        if (fast->val != reverseHead->val) {
            return false;
        }
        fast = fast->next;
        reverseHead = reverseHead->next;
    }
    return true;
}
```
