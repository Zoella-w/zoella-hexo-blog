---
title: IB-Rust-数组与切片
date: 2025-05-22 17:23:49
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson8](https://github.com/Zoella-w/IB-Rust/tree/main/8_array_slice)

## 数组介绍

[T; N] 表示 N 个值的数组，每个值的类型为 T。数组的大小是在编译期就已确定的常量，并且是类型的一部分，不能追加新元素或缩小数组

任意一种类型的值 v，表达式 v.len() 会给出 v 中的元素数，v[i] 引用的是 v 的第 i 个元素（i 的类型必须是 usize）。Rust 会检查 i 是否在 0～v.len()-1 的范围内，如果没在则会出现 panic

## 数组切片介绍

&[T] 和 &mut [T] 可称为 T 的共享切片和 T 的可变切片，它们是对一系列元素的引用，这些元素是某个其他值（如：数组或向量）的一部分

可以将切片视为指向其第一个元素的指针，以及从该点开始允许访问的元素数量的计数

可变切片 &mut [T] 允许读取和修改元素，但不能共享；共享切片 &[T] 允许在多个读取者之间共享访问权限，但不允许修改

## 数组

- 数组是一段分配的**连续的相同数据类型的**内存块

- 数组是静态的，一旦定义和初始化（为数组中每个元素赋值），则长度不可更改

- 数组的元素有相同的数据类型，每个元素都独占数据类型大小的内存块，即数组的内存大小等于数组长度乘数组的数据类型大小

- 数组中每个元素都按顺序依次存储，数组下标既代表元素的存储位置，也是数组元素的唯一标识

- 可以更新或修改数组元素的值，但不能删除数组元素。如果要删除，可将值赋为表示删除的值（比如 0）

## 切片

切片（Slice）表示从包含多个元素的容器中取得局部数据，Rust 有三种数据类型支持 Slice 操作：String、Array 和 Vec

Rust 中的切片操作只允许获取一段连续的局部数据，该数据称作切片数据。而有些语言可以取得离散元素，甚至有些语言可以对 hash 结构进行切片操作

## 切片常用函数

- len()：slice 元素个数
- is_empty()：判断 slice 是否为空
- contains()：判断是否包含某个元素
- repeat()：重复 slice 指定次数
- reverse()：反转 slice
- join()：将各元素压平（flatten）并通过指定的分隔符连接起来
- swap()：交换两个索引处的元素
- windows()：以指定大小的窗口进行滚动迭代
- starts_with()：判断 slice 是否以某个 slice 开头

## 课后习题

> 给定一个整数数组 nums，返回一个数组 answer ，使得 answer[i] 等于 nums 除之外 nums[i] 的所有元素的乘积。
> 任何前缀或后缀的乘积 nums 都保证适合 32 位整数。
> 您必须编写一个能够及时运行 O(n) 且无需使用除法运算的算法。
> 
> 示例 1：
> 输入： nums = [1,2,3,4]
> 输出： [24,12,8,6]
> 
> 示例 2：
> 输入： nums = [-1,1,0,-3,3]
> 输出： [0,0,9,0,0]
> 
> 限制：
> 2 <= nums.length <= 105
> -30 <= nums[i] <= 30
> 任何前缀或后缀的乘积 nums 都保证适合 32 位整数。
> 
> 进阶： 你能以 O(1) 额外空间复杂度解决这个问题吗？（输出数组不算作空间复杂度分析的额外空间。）

``` rust
fn main() {
    let nums = [1, 2, 3, 4];
    let answer = get_answer(nums.to_vec());
    println!("answer is {:?}", answer); // [24, 12, 8, 6]
}

fn get_answer(nums: Vec<i32>) -> Vec<i32> {
    let n = nums.len();
    let mut answer = vec![1; n];

    // 计算前缀乘积：answer[i]
    for i in 1..n {
        // nums[1, n-1] 的前缀乘积（nums[0] 没有前缀）
        answer[i] = answer[i - 1] * nums[i - 1];
    }

    // 计算后缀乘积，并更新 answer[i]
    let mut right = 1; // right 表示当前元素右侧所有元素的乘积
    for i in (0..n).rev() {
        // 从 nums[n-1..0] 的乘积
        answer[i] *= right; // 前缀乘积 * 后缀乘积
        right *= nums[i]; // 更新后缀乘积
    }

    answer
}
```