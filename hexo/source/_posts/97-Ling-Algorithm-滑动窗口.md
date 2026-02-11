---
title: Ling-Algorithm-滑动窗口
date: 2026-02-11 20:00:43
tags:
    - Algorithm
categories:
    - Algorithm
      - Window
---

## [209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/description/)

### 最优解法

**时间复杂度 O(n)（左指针和右指针各移动 n），空间复杂度 O(1)**

> `right` 从左到右移动，当 `sum-nums[left]>=target` 时，`left` 循环向左移动，且 `sum-=nums[left]`
> 如果 `sum>=target` 符合条件，则判断是否需要更新答案。
> 注意：`right-left+1` 是子数组长度

``` javascript
/**
 * @param {number} target
 * @param {number[]} nums
 * @return {number}
 */
var minSubArrayLen = function(target, nums) {
    const len = nums.length;
    let ans = len + 1, sum = 0, left = 0;
    // 右端点右移
    for (let right = 0; right < len; ++right) {
        sum += nums[right];
        // 循环判断是否可以左移左端点
        while (sum - nums[left] >= target) {
            sum -= nums[left];
            left++;
        }
        // 如果符合条件，则判断是否更新答案
        if (sum >= target) {
            ans = Math.min(ans, right - left + 1);
        }
    }
    return ans <= len ? ans : 0;
};
```

## [713. 乘积小于 K 的子数组](https://leetcode.cn/problems/subarray-product-less-than-k/)

### 最优解法

**时间复杂度 O(n)，空间复杂度 O(1)**

> `right` 从左向右移动，循环 `prod/nums[left]` 直到 `prod<k`，
> 此时计算以当前右端点结尾的所有子数组的数量：`right-left+1` （`[l,r], [l+1,r] ... [r,r]`）

``` javascript
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var numSubarrayProductLessThanK = function(nums, k) {
    // 排除特殊情况
    if (k <= 1) return 0;
    const len = nums.length;  
    let left = 0, prod = 1, ans = 0;
    // 右端点右移
    for(let right = 0; right < len; ++right) {
        prod *= nums[right];
        while (prod >= k) {
            prod /= nums[left];
            left++;
        }
        // 以当前右端点结尾的所有子数组的数量
        ans += right - left + 1;
    }
    return ans;
};
```

## [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/description/)

### 最优解法

**时间复杂度 O(n)，空间复杂度 O(1)（map 的长度最大为 26）**

> 维护一个 `map`，记录字符出现的次数。
> `right` 从左到右移动，如果当前字符出现次数大于1，则循环右移 `left`（删除左侧字符），直到当前字符第一次出现。
> `right-left+1` 即为当前子串长度，判断是否更新答案

``` javascript
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    const map = new Map();
    let left = 0, ans = 0;
    // 右端点右移
    for (let right = 0; right < s.length; ++right) {
        let c = s[right];
        // 更新字符出现次数
        // map.get(c) ?? 0 使用了空值合并运算符 ??
        // 如果 map.get(c) 为 null 或者 undefined，则设置为 0
        map.set(c, (map.get(c) ?? 0) + 1);
        // 如果当前字符出现次数大于1，则循环删除左侧字符
        while(map.get(c) > 1) {
            map.set(s[left], map.get(s[left]) - 1);
            left++;
        }
        ans = Math.max(ans, right - left + 1);
    }
    return ans;
};
```
