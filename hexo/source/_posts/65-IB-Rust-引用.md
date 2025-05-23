---
title: IB-Rust-引用
date: 2025-05-22 16:26:31
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson7](https://github.com/Zoella-w/IB-Rust/tree/main/7_reference)

## 引用的分类

- **不可变引用（Immutable Reference）**
  - 可以读取数据，但不能修改
  - 一个变量可以有多个不可变引用，但不能与可变引用共存
- **可变引用（Mutable Reference）**
  - 可以读取和修改数据
  - 一个变量在某一时刻只能有一个可变引用，且不能与不可变引用共存

## 借用的规则

- 同一时间内，一个变量只能有一个可变引用或多个不可变引用
- 引用必须总是有效（被引用的数据在其引用的生命周期内必须始终存在，且不能被销毁）

## move & borrow & slice

### move

当​堆数据被赋值给另一个变量或作为参数传递时，其所有权会 ​​转移​​（move）到新变量或函数中，原变量将失效

### borrow

借用是 Rust 中通过引用 ​​临时访问数据​​ 的逻辑概念

### slice

切片（slices）分为：

#### 字符串切片

```rust
let s = String::from("hello world");
let hello = &s[0..5]; // 引用 "hello"
let world = &s[6..11]; // 引用 "world"
```

#### 数组切片

```rust
let arr = [1, 2, 3, 4, 5];
let slice = &arr[1..3]; // 引用 [2, 3]
```

## 课后习题

通过编译器错误提示，修复并运行代码
```rust
#[test]
fn test_lifetime() {
    let large = longest("a", "ab");
    println!("large one is {large}");
    // expected named lifetime parameter
    fn longest(x: &str, y: &str) -> &str {
        if x.len() > y.len() {
          x
        } else {
          y
        }
    }
}
```

如果函数返回引用，且该引用依赖于输入参数的引用，必须显式标注生命周期参数
```rust
#[test]
fn test_lifetime() {
    let large = longest("a", "ab");
    println!("large one is {large}");
    // modified
    fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
        if x.len() > y.len() {
          x
        } else {
          y
        }
    }
}
