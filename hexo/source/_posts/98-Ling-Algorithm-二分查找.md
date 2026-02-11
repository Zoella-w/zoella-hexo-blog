---
title: Ling-Algorithm-二分查找
date: 2026-02-11 21:57:16
tags:
    - Algorithm
categories:
    - Algorithm
      - Binary search
---

## [34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)

### 最优解法

**时间复杂度 O(logn)，空间复杂度 O(1)**

> 答案所在范围使用左闭右开区间 `[left,right)`（取不到 `right`，所以是开区间），为了使区间不为空，循环条件应为 `left<right`。
> - 如果 `nums[mid]>=target`，令 `right=mid` `[left,mid)`，可得 `nums[right]>=target`（循环不变量一），
> - 如果 `nums[mid]<target`，令 `left=mid+1` `[mid+1,right)`，可得 `nums[left-1]<target`（循环不变量二）。
> 
> 当 `left=right` 时结束循环，此时 `nums[left]>=target` 且 `nums[left-1]<target`，所以 `left` 和 `right` 同时指向答案。


``` javascript
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */

var lower_bound = function(nums, target) {
    let left = 0, right = nums.length; // [left, right)
    while (left < right) { // 区间不为空
        // 循环不变量：
        // nums[left-1] < target
        // nums[right] >= target
        const mid = Math.floor((left + right) / 2);
        if (nums[mid] >= target) {
            right = mid; // [left,mid) => nums[right]>=target
        } else {
            left = mid + 1; // [mid+1,right) => nums[left-1]<target
        }
        // 循环结束后 left=right
        // 循环不变量：nums[left-1]<target 且 nums[left]=nums[right]>=target
        // 所以 left 是第一个 >= target 的元素下标
    }
    return left;
};


var searchRange = function(nums, target) {
    const start = lower_bound(nums, target);
    if (start === nums.length || nums[start] !== target) {
        return [-1, -1];
    }
    // end 是第一个 > target 的元素下标 -1
    const end = lower_bound(nums, target + 1) -1;
    return [start, end];
};
```