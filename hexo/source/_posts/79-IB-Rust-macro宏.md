---
title: IB-Rust-macro宏
date: 2025-06-11 16:26:12
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson21](https://github.com/Zoella-w/IB-Rust/tree/main/21_macro)

## 常用宏

```rust
println!("Hello, world!"); // 常用
println! {"Hello, world!"};
println!["Hello, world!"];
let v = vec![1, 2, 3, 4, 5];
assert_eq!(1, 10);
panic!("Something went wrong!");
```

## 声明宏

声明宏（Declarative Macros），也称为 `macro_rules!`，是最常⻅的宏类型。它们允许通过模式匹配来⽣成代码

```rust
#[macro_export]
macro_rules! say_hello {
    // 表示 0 个参数的情况
    () => {
        println!("Hello, world!");
    };
}
fn main() {
    say_hello!();
    say_hello! {};
    say_hello![];
}
```

### 实现变长参数

宏比起函数，可以实现变长参数

变长参数是通过一种叫“重复模式”的机制实现的，它由三部分组成：
- ​​捕获组​​ `$( ... )`：这表示一组可以重复的内容
​- ​分隔符​​ `,`：用来分隔多个参数（可以是其他符号）
​- ​重复指示器​ `*` 或 `+` `-`​：表示这组内容可以重复多少次

```rust
macro_rules! my_macro {
    // 模式部分
    // 捕获组 分隔符 重复指示器
    ($($arg:expr),*) => {
        // 宏体中的重复代码
        $(
            // 这里处理每个 $arg
            println!("Got argument: {}", $arg);
        )*
    };
}
```

- `​​$($arg:expr)`​​：捕获一个表达式并命名为 `$arg`
  - `$arg` 是变量名（可以自定义）
  - `:expr` 表示匹配的表达式类型
​- ​`,`​​：每个参数之间的分隔符
​​- `*`​​：表示这个模式可以匹配0次或多次
  - `+`：表示匹配1次或多次（至少一个参数）

#### 处理不同的参数类型

表达式（expressions）

```rust
$($arg:expr),*  // 匹配任意表达式，如：1, "text", 2+3, function_call()
```

标识符（identifiers）

```rust
$($name:ident),*  // 匹配变量名，如：x, my_var, data
```

类型（types）

```rust
$($t:ty),*  // 匹配类型名，如：i32, String, Vec<u8>
```

任意令牌（tokens）

```rust
$($token:tt),*  // 匹配任何语法元素，如：>, ?, ->, +=
```

#### 多种分隔符

自定义分隔符，且支持不同分隔符的多个模式

```rust
macro_rules! print_multiple {
    // 空格分隔的模式
    ($($arg:expr) *) => {
        $(
            println!("Arg: {}", $arg);
        )*
    };
    // 逗号分隔的模式
    ($($arg:expr), *) => {
        $(
            println!("Arg: {}", $arg);
        )*
    };
    // 分号分隔的模式
    ($($arg:expr); *) => {
        $(
            println!("Arg: {}", $arg);
        )*
    };
}
print_multiple![1 2 3];
print_multiple![4, 5, 6];
print_multiple![7; 8; 9];
```

## 过程宏

过程宏（Procedural Macros）允许使⽤函数⽣成代码，分为三种类型：
- 派生宏
- 属性宏（Attribute-like macro）
- 函数宏（Function-like macro）

过程宏在​​编译时​​执行，需要：
- 作为独立的 crate 编写
- 在 toml 文件中特殊标记（proc-macro = true）
- 接收 TokenStream 作为输入
- 输出新的 TokenStream

### 过程宏配置

Cargo.toml 文件内容：

```toml
[lib]
proc-macro = true
[dependencies]
syn = "2.0"     # Rust语法解析器
quote = "1.0"   # 代码生成器
proc-macro2 = "1.0" # 过程宏工具
```

项目结构：

```
my_macros/
├── Cargo.toml
└── src/
    ├── main.rs          # 主入口
    ├── derive_macros.rs # 派生宏
    ├── attr_macros.rs  # 属性宏
    └── fn_macros.rs    # 函数式宏
```

### 派生宏

派生宏 `#[derive(MyMacro)]` 用于自动为结构体和枚举实现 trait

#### `​​#[derive(Debug)]`

自动实现 std::fmt::Debug trait，用于调试输出

```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }
println!("{:?}", Point { x: 1, y: 2 }); // Point { x: 1, y: 2 }
```

#### `​​#[derive(Clone)]`

自动实现 Clone trait，允许值复制

```rust
#[derive(Clone)]
struct Data {
    content: String,
}
fn main() {
    let data = Data { content: "Hello".to_string() };
    let cloned_data = data.clone();
}
```

#### `​​#[derive(Copy)]`

将类型标记为复制语义（需同时实现 Clone）

```rust
#[derive(Copy, Clone)]
struct Position {
    x: u8,
    y: u8,
}
fn main() {
    let pos1 = Position { x: 5, y: 10 };
    let pos2 = pos1; // 复制而非移动
}
```

#### `​​#[derive(PartialEq, Eq)]`

实现相等比较操作符 == 和 !=

```rust
#[derive(PartialEq, Eq)]
struct UserId(u64);
fn main() {
    let id1 = UserId(1001);
    let id2 = UserId(1002);
    assert!(id1 != id2);
}
```

#### `​​#[derive(PartialOrd, Ord)]`

实现比较操作符 <, >, <=, >=

```rust
#[derive(PartialOrd, Ord, PartialEq, Eq)]
struct Priority(u8);
fn main() {
    let p_low = Priority(1);
    let p_high = Priority(10);
    assert!(p_low < p_high);
}
```

#### `​​#[derive(Default)]`

自动生成默认值实现

```rust
#[derive(Default)]
struct Settings {
    timeout: u32,
    retry_count: u8,
}
fn main() {
    let settings = Settings::default(); // timeout=0, retry_count=0
}
```

### 属性宏

属性宏 `#[my_macro]` 用于自定义属性处理

#### `#[cfg]`

用于条件编译

```rust
#[cfg(target_os = "windows")]
fn get_system_info() -> String {
    "Windows system".to_string()

#[cfg(target_os = "linux")]
fn get_system_info() -> String {
    "Linux system".to_string()
}
fn main() {
    println!("Running on: {}", get_system_info());
}
```

#### `#[test]`

用于单元测试

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_addition() {
        assert_eq!(2 + 2, 4);
    }
    #[test]
    #[should_panic(expected = "out of bounds")]
    fn test_panic() {
        vec![1, 2, 3][10];
    }
}
```

#### `#[allow]` / `#[warn]` / `#[deny]`

用于代码检查控制

```rust
#[allow(unused_variables)] // 允许未使用变量
fn unused_example() {
    let x = 5; // 不会产生警告
}

#[warn(missing_docs)] // 文档缺失警告
pub struct NoDocumentation;

#[deny(deprecated)] // 拒绝使用已弃用项目
fn no_deprecated() {
    // 使用已弃用方法将导致编译错误
}
```

#### `#[repr]`

用于内存表示控制

```rust
#[repr(C)] // C语言兼容内存布局
struct CCompatible {
    x: u32,
    y: u32,
}

#[repr(packed)] // 无填充紧凑布局
struct PackedData {
    flag: u8,
    value: u32,
}
```

### 函数宏

函数宏 `my_macro!()` 是类似函数调用的宏

#### `println!` / `format!`

用于格式化输出

```rust
let name = "Alice";
let age = 30;
// 标准输出
println!("Hello, {}! You are {} years old.", name, age);
// 创建格式化字符串
let greeting = format!("Hello, {}! Age: {}", name, age);
```

#### `vec!`

用于向量创建

```rust
// 空向量
let empty: Vec<i32> = vec![];
// 指定元素的向量
let numbers = vec![1, 2, 3, 4, 5];
// 重复元素的向量
let zeros = vec![0; 10]; // 10个0
```

#### `assert!` / `assert_eq!` / `assert_ne!`

用于测试断言

```rust
fn calculate(a: i32, b: i32) -> i32 {
    a + b
}
fn main() {
    let result = calculate(2, 3);
    assert!(result == 5);
    assert_eq!(result, 5);
    assert_ne!(result, 10);
    // 带自定义消息
    assert!(
        result > 0, 
        "Result must be positive, got {}", 
        result
    );
}
```

#### `panic!`

用于触发恐慌

```rust
fn divide(numerator: f64, denominator: f64) -> f64 {
    if denominator == 0.0 {
        panic!("Division by zero!");
    }
    numerator / denominator
}
```

#### `file!`, `line!`, `column!`

用于展示代码位置信息

```rust
fn log(message: &str) {
    println!(
        "[{}:{}:{}] {}", 
        file!(), 
        line!(), 
        column!(),
        message
    );
}
fn main() {
    log("Starting program"); // [src/main.rs:12:5] Starting program
}
```

#### `concat!`

用于连接文字

```rust
const VERSION_STRING: &str = concat!(
    "App v", 
    env!("CARGO_PKG_VERSION"), 
    " built at ", 
    env!("BUILD_TIMESTAMP")
);
fn main() {
    println!("{}", VERSION_STRING);
}
```

## 课后习题

通过 `macro_rules!` 实现对应的 macro，并通过测试 case

```rust
assert_eq!(repeat!("x",3) ,"xxx");
assert_eq!(sum!(1,2,3,4,5), 15);
assert_eq!(max_value!(1,8,9), 9);
```

```rust
#[macro_export]
macro_rules! repeat {
    // 基本情况：重复0次
    ($item:expr, 0) => {
        ""
    };
    // 基本实现：重复n次
    ($item:expr, $n:expr) => {{
        // 创建足够大的字符串空间以容纳所有重复项
        let mut result = String::with_capacity($item.len() * $n);
        for _ in 0..$n {
            result.push_str($item);
        }
        result
    }};
}

macro_rules! sum {
    // 基本情况：单个元素
    ($x:expr) => { $x };
    // 基本实现：递归处理多个元素
    // $($y:expr),+ 匹配 除了第一个参数后面的所有参数
    ($x:expr, $($y:expr),+) => {{
        // $ 说明后面的 ($y),+ 是新的匹配参数
        $x + sum!($($y),+)
    }};
}

/// 查找多个值中的最大值的宏
macro_rules! max_value {
    // 基本情况：两个值比较
    ($x:expr, $y:expr) => {
        if $x > $y { $x } else { $y }
    };
    // 递归处理多个值
    ($x:expr, $($y:expr),+) => {
        max_value!($x, max_value!($($y),+))
    };
}
```
