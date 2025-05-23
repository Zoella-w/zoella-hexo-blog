---
title: IB-Rust-所有权
date: 2025-05-22 14:32:11
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson6](https://github.com/Zoella-w/IB-Rust/tree/main/6_ownership)

## 内存回收

Rust 的设计目的是确保内存安全并防止数据竞争，而不依赖垃圾回收器，这种内存安全性主要通过所有权（ownership）来实现

主流编程语言的内存回收机制：
- 静态语言
  - 在编译时对变量类型进行检查和确认
  - C, C++, Rust
- 动态语言
  - 在运行时进行类型检查和确认
  - Javascript, Python

| 特性 | 静态语言 | 动态语言 |
| -- | -- | -- |
| 类型检查时间 | 编译时 | 运行时 | 
| 类型安全 | 更安全，减少运行时类型错误 | 较灵活，但类型错误可能在运行时出现
| 性能 | 通常更高效，编译器优化 | 通常较低，运行时类型检查 |
| 灵活性 | 较低，需明确声明类型 | 较高，允许在运行时改变类型 |
| 代码简洁性 | 需要显式类型声明，代码相对冗⻓ | 通常更简洁，适合快速开发 |
| 开发工具支持 | 更强大的静态分析和重构工具 | 开发工具支持有限，但在快速开发上占优势

### C/C++

- 内存管理方式：手动管理
- 特点
  - 程序员通过 malloc 和 free (C) 或 new 和 delete (C++) 释放内存
  - 没有内置的垃圾回收机制
- 优点：高效灵活，适用于对性能要求极高的系统级编程
- 缺点：容易出现内存泄漏、悬垂指针和缓冲区溢出等问题

``` C
// 释放分配的内存
free(ptr);
ptr = NULL; // 将指针设为 NULL，避免悬空指针 
// 动态分配一个数组的内存
int n = 5;
int *arr = (int *)malloc(n * sizeof(int));
```

### Javascript

- 内存管理方式：垃圾回收
- 特点
  - 浏览器和 Node.js 环境中均使用垃圾回收器（如 V8 引擎的垃圾回收器）
  - 采用标记-清除、标记-压缩等算法
- 优点：自动内存管理，适合快速开发和运行在多平台上的应用
- 缺点：垃圾回收机制在某些情况下可能导致性能问题，如线程停顿

### Rust

- 内存管理方式：所有权系统
- 特点
  - 使用所有权系统进行内存管理，编译器在编译时通过静态分析来确保内存安全
  - 每个值都有一个所有者，且任何时候只能有一个有效的所有者
  - 通过借用（引用）机制来共享数据，避免数据竞争和悬垂指针
- 优点
  - 在编译时保证内存安全，没有运行时开销
  - 避免了数据竞争和悬垂指针
- 缺点：需要程序员理解和遵循所有权和借用规则，学习曲线较陡

## 所有权规则

Rust 所有权系统的三个基本规则：
- 每一个值都有一个所有者（owner）
- 在任一时刻，值只能有一个所有者
- 当所有者离开作用域（scope），值会被丢弃（drop）

## 课后习题
用 2 种方式实现，确保 s1 s2 都能正常打印出来
``` rust
// TODO:
fn take_ownership(s: String) -> String {
    s
}
let s1 = String::from("Hello");
let s2 = take_ownership(s1);
// 如下代码不能修改
println!("{}", s1);
println!("{}", s2);
```

### 方式一：借用
``` rust
fn take_ownership(s: &String) -> String {
    s.clone()
}
fn main() {
    let s1 = String::from("Hello");
    let s2 = take_ownership(&s1);
    // 如下代码不能修改
    println!("{}", s1);
    println!("{}", s2);
}
```

### 方式二：调用时 clone
``` rust
fn take_ownership(s: &String) -> &String {
    s
}
fn main() {
    let s1 = String::from("Hello");
    let s2 = take_ownership(&s1);
    // 如下代码不能修改
    println!("{}", s1);
    println!("{}", s2);
}
```
