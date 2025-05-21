---
title: IB-Rust-流程控制
date: 2025-05-21 14:12:36
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson4](https://github.com/Zoella-w/IB-Rust/tree/main/4_flow_control)

## 条件控制

### `if` 表达式

Rust 不会自动将非布尔值转换为布尔值，必须显式使用布尔值作为 `if` 的条件

### 使用 `else if` 处理多重条件

只会执行第一个条件为 `true` 的代码块

### 在 `let` 语句中使用 `if`

声明的变量将会绑定到表示 `if` 表达式结果的值上

- `if` 的每个分支可能的返回值必须是相同类型
- 代码块的值是其最后一个表达式的值，即不需要分号


## 循环

Rust 有三种循环：`loop`、`while` 和 `for`

### `loop`

运行时会出现反复打印的 `again!`，直到手动停止程序：

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

#### break & continue

`break` 关键字告诉程序停止循环

`continue` 关键字告诉程序继续循环

#### 从循环返回值

和 if 类似，loop 也能赋值

```rust
fn main() {
    let mut counter = 0;
    let result = loop {
        counter += 1;
        if counter == 10 {
            break counter * 2;
        }
    };
    println!("The result is {result}");
}
```

#### 循环标签

break + 循环标签 可以退出外层循环

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;
        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }
        count += 1;
    }
    println!("End count = {count}");
}
```

### `while` 条件循环

条件为 `true` 执行循环，条件不为 `true` 停止循环

```rust
fn main() {
    let mut number = 3;
    while number != 0 {
        println!("{number}!");

        number -= 1;
    }
    println!("LIFTOFF!!!");
}
```

#### 用 loop 实现

```rust
fn main() {
    let mut number = 3;
    loop {
        if number == 0 {
            break;
        }
        println!("{number}!");
        number -= 1;
    }
    println!("LIFTOFF!!!");
}
```

### `for`

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a {
        println!("the value is: {element}");
    }
}
```

倒序输出 [1, 2, 3]，结果为 3, 2, 1：

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
}
```

#### 所有权转移

> 对于实现了 `copy` 特征的数组（如 [i32; 10]）而言，`for item in arr` 不会转移 `arr` 的所有权，而是对其进行了拷贝，因此循环之后仍可以使用 `arr` 

| 使用方法  | 等价使用方式 | 所有权 |
| -- | -- | -- |
| `for item in collection` | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
| `for item in &collection` | `for item in collection.iter()` | 不可变借用 |
| `for item in &mut collection` | `for item in collection.iter_mut()` | 可变借用 |

## 练习题

https://practice-zh.course.rs/flow-control.html
