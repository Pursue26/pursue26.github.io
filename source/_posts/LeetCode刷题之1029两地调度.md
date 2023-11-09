---
title: LeetCode刷题之1029两地调度
date: 2023-11-07 09:46:52
categories: 
    - LeetCode
    - 贪心算法
tags: 
    - LeetCode
    - 贪心算法
abbrlink: 231107094652
---

[1029 两地调度](https://leetcode-cn.com/problems/two-city-scheduling/)：公司计划面试 `2N` 人。第 `i` 个面试者飞往 A 市的费用为 `costs[i][0]`，飞往 B 市的费用为 `costs[i][1]`。返回将每个面试者都飞到某一座城市的最低费用，要求每个城市都有 `N` 个面试者抵达。

<!--more-->

```
输入：costs = [[10,20],[30,200],[400,50],[30,20]]
输出：110
解释：
第一个人去 a 市，费用为 10。
第二个人去 a 市，费用为 30。
第三个人去 b 市，费用为 50。
第四个人去 b 市，费用为 20。

最低总费用为 10 + 30 + 50 + 20 = 110，每个城市都有一半的人在面试。
```

## 排序+贪心算法

要求每个城市都有 `N` 个面试者抵达。我们可以选择 `N` 个面试者去 A 市，那么剩下的 `N` 个面试者自然就得去 B 市了。**那哪些人去 A 市呢**？

「每个人」只需要考虑自己选哪个城市更省钱，但必须要有 `N` 个人去 A 市，`N` 个人去 B 市。因此，我们只需要**比较「所有人」去 B 市与去 A 市的成本差——差值越大，说明去 A 市比去 B 市省出来的成本更多；差值越小，说明能省出来的成本就越少，这些人就去 B 市吧**。

时间复杂度：`O(nlogn)`，排序所需时间，空间复杂度：`O(1)`。

```c
int cmp(const void *pa, const void *pb) {
    const int *arr1 = *(const int **)pa;
    const int *arr2 = *(const int **)pb;
    // 按B市与A市成本差降序排序
    return (arr2[1] - arr2[0]) - (arr1[1] - arr1[0]);
}

int twoCitySchedCost(int** costs, int costsSize, int* costsColSize) {
    qsort(costs, costsSize, sizeof(costs[0]), cmp);
    int minCost = 0;
    for (int i = 0; i < costsSize; ++i) {
        // 前一半差值大的去A市, 后一半差值小的去B市
        if (i < (costsSize >> 1)) {
            minCost += costs[i][0];
        } else {
            minCost += costs[i][1];
        }
    }
    return minCost;
}
```
