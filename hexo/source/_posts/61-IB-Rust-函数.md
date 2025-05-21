---
title: IB-Rust-函数
date: 2025-05-21 13:48:11
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson3](https://github.com/Zoella-w/IB-Rust/tree/main/3_func)

## 函数组成

- 声明函数的关键字 `fn`
- 函数名 `add()`
- 参数 `i` 和 `j`和参数类型`i32`
- 返回值类型`i32`
- 函数体`i + j` <!--注意这里没有return -->

注意：函数可以在任意位置定义

### 函数名

开头是字符/下划线，后面是数字，下划线，字母（不能仅有下划线）

函数名和变量名使用[蛇形命名法（snake case）](https://course.rs/practice/naming.html)，如：`fn add_two() -> {}`

### 函数参数

Rust 是强类型语言，需要为所有函数参数标识具体类型

### 函数返回

函数返回值是函数体最后一条表达式的返回值，也可以使用 `return` 提前返回，初学者只需记住两种形态：

```rust
// 没有return + 没有分号
fn add(i: i32, j: i32) -> i32 {
    i + j
}

// return + 分号
fn add(i: i32, j: i32) -> i32 {
    return i + j;
}
```

#### 特殊返回类型

##### 1、无返回值`()`

如果一个函数没有返回值，就返回`()`

```rust
fn print(i: i32) {
    println!("{}", i);
}

fn print(i: i32) -> () {
    println!("{}", i);
}
```

##### 2、函数永不返回

函数返回类型为`!`时，表示该函数永不返回（diverge function），常用做会导致程序崩溃的函数：

```rust
fn dead_end() -> ! {
  panic!("崩溃");
}
```

下面的函数创建了一个无限循环，也永不返回：

```rust
fn forever() -> ! {
  loop {
    //...
  };
}
```

## 练习题

https://practice-zh.course.rs/basic-types/functions.html
