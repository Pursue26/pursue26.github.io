---
title: LeetCode刷题之117填充每个节点的下一个右侧节点指针II
date: 2023-11-03 12:12:09
categories: 
    - LeetCode
    - 树遍历
tags: 
    - LeetCode
    - 树遍历
    - 层序遍历
    - 链表
abbrlink: 231103121209
---

[117 填充每个节点的下一个右侧节点指针 II](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node-ii/)：给定一个包含指针 `next` 成员的二叉树：填充它的每个 `next` 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 `next` 指针设置为 `NULL`。初始状态下，所有 `next` 指针都被设置为 `NULL`。

<!--more-->

树结构体：
```c
struct Node {
    int val;
    Node *left;
    Node *right;
    Node *next;
}
```

示例图：
![](https://assets.leetcode.com/uploads/2019/02/15/117_sample.png)

## 队列+层序遍历

根据题目描述，「填充它的每个 `next` 指针，让这个指针指向（同层的）其下一个右侧节点」，我们很容易想到使用「层序遍历」解决这个问题。而层序遍历通常需要借助 FIFO 队列实现。

**关于借助队列的树的层序遍历**：首先要入队根节点；然后，记录队列中的节点个数 `size`；随后，弹出 `size` 个节点，在弹出的过程中入队弹出节点的左、右节点；最后，在这 `size` 个节点弹出后，队列中剩下的节点，就是刚刚入队的树的下一层节点。只要队列不为空，我们就反复进行上述操作，直到队列为空为止（二叉树的叶子层没有左、右节点，便不会再有节点入队，队列也就为空了）。

时间复杂度：`O(n)`，一趟遍历，空间复杂度：`O(n)`，需要使用队列保存二叉树中的节点。

> 关于队列，我们可以使用[数据结构之队列（链表实现）](https://pursue26.github.io/posts/231017105123.html)中实现的接口。但是，为了简单、清晰，我在这里直接使用数组模拟 FIFO 队列。

```c
typedef struct Node TreeNode_t;
struct Node* connect(struct Node* root) {
    if (root == NULL) {
        return NULL;
    }
    
	TreeNode_t *queue[6000];
    int front = 0, rear = 0;  // 队列的头尾指针
    queue[rear++] = root;  // 树不为空, 首先入队根节点

    while (rear - front > 0) {  // 队列不为空
        int size = rear - front;  // 当前队列的数据大小
        TreeNode_t *curNode = NULL, *preNode = NULL;  // 当前节点和上一个节点

        // 弹出这几个数据（某一层的所有节点）
        for (int i = 0; i < size; ++i) {
            curNode = queue[front++];  // 首指针后移
            // 链接前后两个节点, 并更新上一个节点为当前节点
            if (preNode) {
                preNode->next = curNode;
            }
            preNode = curNode;
            
            // 入队左右子树的节点
            if (curNode->left) {
                queue[rear++] = curNode->left;
            }
            if (curNode->right) {
                queue[rear++] = curNode->right;
            }
        }
        // 每层的最后一个节点指向空地址
        curNode->next = NULL;
    }
    return root;
}
```

## 利用已建立的链表构建下一层链表

根据题目描述，「填充它的每个 `next` 指针，让这个指针指向（同层的）其下一个右侧节点」。这样填充后，二叉树的每一层都将是一个单链表，链表的头节点是每层最左侧的树节点。这样，我们就可以根据链表中的节点，获得它的下一层的左、右孩子节点。

因此，我们可以利用当前层的链表 `currLayer` 来构建下一层的链表 `nextLayer`：
- 下一层的构建过程就是：不断的访问 `currLayer` 中的节点，通过它访问下一层的左、右孩子节点，将这些孩子链接起来，就是 `nextLayer`；
- 构建的下一层的链表时，要记录下一层的头结点，以便更新当前层为下一层。
    - 下一层的头结点，就是 `currLayer` 中的节点里，第一个有孩子节点（若有左、右孩子，左孩子优先）。

最后，当下一层为叶子结点层时，便不会有下一层链表，当前层链表会更新为 `NULL`，标志着每层的链表构建完成。

时间复杂度：`O(n)`，一趟遍历，空间复杂度：`O(1)`。

```c
struct Node* connect(struct Node* root) {
    struct Node *currLayer = root;  // 首层的链表为树的根节点

    // 只要当前层不为空, 就可以试探下一层
    while (currLayer) {
        struct Node *node = currLayer;  // 获取当前层的头节点, 用于遍历
        struct Node *nextLayer = NULL;  // 记录下一层的头节点
        bool isFirst = true;

        currLayer = NULL;  // 置空当前层

        while (node) {  // 遍历当前层构成的链表
            struct Node *data[2] = {node->left, node->right};
            // 遍历当前节点的左右节点
            for (int i = 0; i < 2; ++i) {
                if (data[i]) {
                    // 判断是否是下一层的第一个节点
                    if (isFirst) {
                        nextLayer = data[i];
                        currLayer = nextLayer;  // 更新当前层为下一层头节点, 用于下一轮最外层while循环
                        isFirst = false;
                    } else {
                        nextLayer->next = data[i];  // 链接下一层的节点
                        nextLayer = nextLayer->next;
                    }
                }
            }
            node = node->next;
        }
    }
    return root;
}
```

> 评论区用户@M9988：方法二确实比方法一巧妙。但是，作为面试准备的话，我更推崇练习方法一。因为方法一才是层序（BFS）遍历的通用模式。把这个BFS模板练熟以后，面试方可信手拈来（再在里面做附加逻辑）。方法二，技巧性强，可以提高，但是面试的话还是把场景套上模板的熟练能力。
