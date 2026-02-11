---
title: Ling-Algorithm-接雨水
date: 2026-02-08 14:20:22
tags:
    - Algorithm
categories:
    - Algorithm
      - Water
---

## [11. 盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)

### 最优解法

**时间复杂度：O(n)，空间复杂度：O(1)**

> 左指针指向 `height[left]` (left=0)，右指针指向 `height[right]` (right=len-1)，
> 哪条边矮，则将该条边向内侧移动，看是否有更大值（因为如果有更大值，那么一定不会由矮边构成）。
>
> 优化点：如果矮边内侧的边比矮边还矮，就继续向内移动并循环判断

``` javascript
/**
 * @param {number[]} height
 * @return {number}
 */
var maxArea = function(height) {
    const len = height.length;
    let ans = 0;
    let left = 0, right = len - 1;
    while (left < right) {
        const area = (right - left) * Math.min(height[left], height[right]);
        ans = Math.max(ans, area);
        if (height[left] < height[right]) {
            // left++;
            // 优化
            const cur = height[left];
            while (height[left] <= cur && left < right) {
                left++;
            }
        }
        else {
            // right--;
            // 优化
            const cur = height[right];
            while (height[right] <= cur && right > left) {
                right--;
            }
        }
    }
    return ans;
};
```

## [42. 接雨水](https://leetcode.cn/problems/trapping-rain-water/)

### 最优解法一

**时间复杂度：O(n)，空间复杂度：O(n)**

> 当前格子i能接的雨水为：左边板子的最大值 `pre_max[i]`、右边板子的最大值 `suf_max[i]` 中的最小值，再减去当前格子的高度 `height[i]`。
> 从左到右遍历计算每个格子左边板子的最大值，从右到左遍历计算每个格子右边板子的最大值，
> 然后遍历每个格子，计算当前格子的接水量，累积相加。

``` javascript
/**
 * @param {number[]} height
 * @return {number}
 */
var trap = function(height) {
    const len = height.length;
    let ans = 0;
    let pre_max = [], suf_max = [];
    pre_max[0] = height[0];
    suf_max[len - 1] = height[len - 1];
    for (i = 1; i < len; ++i) {
        pre_max[i] = Math.max(pre_max[i - 1], height[i]);
        suf_max[len - i - 1] = Math.max(suf_max[len - i], height[len - i - 1]);
    }
    // for (i = len - 2; i >= 0; --i) {
    //     suf_max[i] = Math.max(suf_max[i + 1], height[i]);
    // }
    for (i = 0; i < len; ++i) {
        ans += Math.min(pre_max[i], suf_max[i]) - height[i];
    }
    return ans;
};
```

### 最优解法二

**时间复杂度：O(n)，空间复杂度优化：O(1)**

> 在解法一的基础上，使用 `left=0` 和 `right=len-1` 双向指针。循环时，先更新当前的 `pre_max` 和 `suf_max`。
> 如果 `pre_max<suf_max`，说明 `left` 指向的格子水量为 `pre_max-height[left]`，接着右移 `left`；如果 `pre_max>suf_max`，说明 `right` 指向的格子水量为 `suf_max-height[right]`，接着左移 `right`。

``` javascript
/**
 * @param {number[]} height
 * @return {number}
 */
var trap = function(height) {
    const len = height.length;
    let ans = 0, pre_max = 0, suf_max = 0;
    let left = 0, right = len - 1;
    // 当left===right时，是最高的柱子，接不了水
    while (left < right) {
        pre_max = Math.max(pre_max, height[left]);
        suf_max = Math.max(suf_max, height[right]);
        if (pre_max < suf_max) {
            ans += pre_max - height[left];
            left++;
        } else {
            ans += suf_max - height[right];
            right--;
        }
    }
    return ans;
};
```
