---
title: Ling-Algorithm-三数之和
date: 2026-02-08 08:46:21
tags:
    - Algorithm
categories:
    - Algorithm
      - Sum
---

## [167. 两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/description/)

用获取信息量来衡量一个算法的效率

### 暴力解法

**时间复杂度：O(n^2)**，没有利用数据有序的信息

### 最优解法

**时间复杂度：O(n)，空间复杂度：O(1)**

> 根据当前指针两个元素相加值判断指针移动，每次判断能得到 n 个信息（大于说明左指针右侧的值都不行，小于说明右指针左侧的值都不行）。

``` javascript
/**
 * @param {number[]} numbers
 * @param {number} target
 * @return {number[]}
 */
var twoSum = function(numbers, target) {
    const len = numbers.length;
    let left = 0, right = len - 1;
    while (left < right) {
        const s = numbers[left] + numbers[right];
        if (s === target) return [left + 1, right + 1];
        // 左指针右移
        if (s < target) left++;
        // 右指针左移
        else right--;
    }
};
```

## [15. 三数之和](https://leetcode.cn/problems/3sum/description/)

### 最优解法

排序 O(nlgn)，循环 O(n^2)

**时间复杂度：O(n^2)，空间复杂度：O(1)**

> 先升序排序，固定 `x=nums[i]` 的值，然后左指针指向 `nums[j]` (j=i+1)，右指针指向 `nums[k]` (k=len-1)，相加大于0 说明左指针右侧的值都不行，相加小于0 说明右指针左侧的值都不行。
>
> 注意去重：如果 `x+nums[j]+nums[k]===0`，循环判断 `if(nums[j+1]===nums[j])`、`if(nums[k-1]===nums[k])`。
>
> 「优化点一」：如果 `nums[i]+nums[i+1]+nums[i+2]>0`，那么往后所有的组合都大于0，直接 break 提前结束
> 「优化点二」：如果 `nums[i]+nums[len-1]+nums[len-2]<0`，那么 `nums[i]` 和其他任意的组合都小于0，continue 从更大的 `nums[i+1]` 开始判断

``` javascript
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var threeSum = function(nums) {
    nums.sort((a, b) => a - b);
    const len = nums.length;
    const ans = [];
    for (let i = 0; i < len - 2; i++) {
        const x = nums[i];
        if (i > 0 && x === nums[i - 1]) continue;
        if (x + nums[i + 1] + nums[i + 2] > 0) break; // 优化一
        if (x + nums[len - 1] + nums[len - 2] < 0) continue; // 优化二
        let j = i + 1, k = len - 1;
        while (j < k) {
            const s = x + nums[j] + nums[k];
            if (s === 0) {
                ans.push([x, nums[j], nums[k]]);
                for (j++; j < k && nums[j] === nums[j - 1]; j++); // 跳过重复数字
                for (k--; k > j && nums[k] === nums[k + 1]; k--); // 跳过重复数字
            }
            else if (s < 0) j++;
            else k--;
        }
    }
    return ans;
};
```