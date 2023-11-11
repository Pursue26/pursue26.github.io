---
title: LeetCode刷题之435无重叠区间
date: 2023-11-09 15:08:55
categories: 
    - LeetCode
    - 贪心算法
tags: 
    - LeetCode
    - 贪心算法
abbrlink: 231109150855
---

[435 无重叠区间](https://leetcode-cn.com/problems/non-overlapping-intervals/)：给定一个区间的集合 `intervals`，其中 `intervals[i] = [start_i, end_i]`。返回需要移除区间的最小数量，使剩余区间互不重叠。注意，区间 `[1,2]` 和 `[2,3]` 的边界相互接触，但没有相互重叠。

<!--more-->

```
输入: intervals = [[1,4],[2,3],[4,6]]
输出: 1
解释: 移除 [1,4] 后，剩下的区间没有重叠。
```

## 右区间排序 + 贪心算法

既然说，移除最少的区间，使剩下区间互相不重叠。对于示例，因为 `[1,4]` 和 `[2,3]` 这两个区间重叠了，但都没跟 `[4,6]` 区间重叠，而 `[1,4]` 区间的右边界太大了，可能会影响到后续区间的最小值，有造成更多区间被移除的风险。所以，我们才移除右边界较大的那个区间。

因此，我们可以按照区间的右边界升序排序，然后使用一前、一后两个指针，指向待比较的两个区间，判断这两个区间是否需要移除一个：
- 前指针指向的区间的右边界不大于后指针指向的区间的左边界，则不需要移除：
    - 前指针指向后区间，后指针后移一个区间
- 前指针指向的区间的右边界大于后指针指向的区间的左边界，则需要移除：
    - 直接移除后区间（区间已经排序，留下右边界小的区间），前指针保持固定，后指针后移一个区间

时间复杂度：`O(nlogn)`，排序所需时间，空间复杂度：`O(1)`。

```c
int cmp(const void *pa, const void *pb) {
    const int *arr1 = *(const int **)pa;
    const int *arr2 = *(const int **)pb;
    return arr1[1] - arr2[1];
}

int eraseOverlapIntervals(int** intervals, int intervalsSize, int* intervalsColSize){
    qsort(intervals, intervalsSize, sizeof(intervals[0]), cmp);

    int pre = 0, cur = 1;
    int ans = 0;
    while (cur < intervalsSize) {
        // 不需要移除
        if (intervals[pre][1] <= intervals[cur][0]) {
            pre = cur;  // 前指针指向后一个区间
        } else {  // 需要移除
            ans++;
        }
        cur++;  // 后指针后移一个区间
    }
    return ans;
}
```

## 左区间排序 + 贪心算法

当然，我们也可以按照区间的左边界升序排序。但在需要移除时，需要进一步判断是移除前一个区间，还是后一个区间；也就是，对上一节中的「直接移除后区间（区间已经排序，留下右边界小的区间）」的判断。

时间复杂度：`O(nlogn)`，排序所需时间，空间复杂度：`O(1)`。

```c
int cmp(const void *pa, const void *pb) {
    const int *arr1 = *(const int **)pa;
    const int *arr2 = *(const int **)pb;
    return arr1[0] - arr2[0];
}

int eraseOverlapIntervals(int** intervals, int intervalsSize, int* intervalsColSize){
    qsort(intervals, intervalsSize, sizeof(intervals[0]), cmp);

    int pre = 0, cur = 1;
    int ans = 0;
    while (cur < intervalsSize) {
        // 不需要移除
        if (intervals[pre][1] <= intervals[cur][0]) {
            pre = cur;  // 前指针指向后一个区间
        } else {  // 需要移除
            // 进一步判断是移除前一个区间，还是后一个区间
            if (intervals[pre][1] >= intervals[cur][1]) {
                pre = cur;  // 移除前一个区间
            }
            ans++;
        }
        cur++;  // 后指针后移一个区间
    }
    return ans;
}
```

> 这个题目，其实是**预定会议的一个问题**，给定若干个会议时间（开始时间-结束时间），然后去预定会议，那么能够成功预定的最大会议数是多少？其核心是找出最大不重叠区间的个数。
