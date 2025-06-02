---
title: IB-Rust-模式匹配
date: 2025-05-27 17:57:20
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson12](https://github.com/Zoella-w/IB-Rust/tree/main/12_match)


## 概述

模式匹配可以检查数据的结构并进行相应操作，提高代码的可读性和简洁性、减少错误，尤其是在处理复杂数据结构时

## 基础模式匹配

### `match` 表达式

基本用法：match 语句可以用于模式匹配数字、字符串、枚举等

```rust
let number = 13;
match number {
    1 => println!("One!"),
    2 => println!("Two!"),
    3 => println!("Three!"),
    _ => println!("Something else!"),
}
```

## 模式匹配的各种模式

### 字面值模式

```rust
let x = 1;
let y = "Hello";
match x {
    1 => println!("One"),
    2 => println!("Two"),
    _ => println!("Other"),
}
match y {
    "Hello" => println!("Greeting"),
    "Goodbye" => println!("Farewell"),
    _ => println!("Other"),
}
```

### 变量模式

将 x 赋值给 var，注意 x 所有权的转移

```rust
let x = 42;
match x {
    var => println!("The value is: {}", var),
}
```

### 通配符模式

```rust
match x {
    _ => println!("Any value"),
}
```

### 结构模式

```rust
struct Point { x: i32, y: i32 }
let p = Point { x: 0, y: 7 };
match p {
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y } => println!("On neither axis: ({}, {})", x, y),
}
```

### 元组模式、枚举模式、解构模式

类似的，用于解构元组、枚举和其他复杂数据结构

## 守卫和绑定

守卫：在模式匹配中，可以使用守卫来添加额外的条件判断

**先匹配模式，再检查条件**

```rust
let x = 5;
match x {
    n if n % 2 == 0 => println!("Even"),
    n => println!("Odd"),
}
```

绑定：在模式匹配中，可以使用绑定来将模式中的值绑定到变量上

**同时匹配模式并绑定变量**

```rust
let x = 3
match x {
    var @ 1..=5 => println!("Value in range: {}", var),
    _ => println!("Out of range"),
}
```

## 模式匹配的应用场景

- 处理错误
- 解析命令行参数
- 解析配置文件
- 解析数据包
- 解析 XML 或 JSON 等数据格式

### 处理错误

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(a / b)
    }
}

match divide(4, 2) {
    Ok(result) => println!("Result is {}", result),
    Err(e) => println!("Error: {}", e),
}
```

## 高级模式匹配技巧

### 嵌套模式

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
let msg = Message::ChangeColor(0, 160, 255);
match msg {
    // 解构
    Message::ChangeColor(r, g, b) => {
        println!("Change the color to red {}, green {}, and blue {}", r, g, b)
    }
    _ => (),
}
```

### 模式匹配与迭代器： 结合 iter 和 match 使用

```rust
let vec1 = vec![1, 2, 3];
let vec2 = vec![1, 2, 3];
for (a, b) in vec1.iter().zip(vec2) {
    println!("{} + {} = {}", a, b, a + b);
}
```

### if let 和 while let： 简化单个模式匹配

```rust
let opt = Some(5);
if let Some(x) = opt {
    println!("Matched {:?}", x);
}
let mut iter = vec![1, 2, 3].into_iter();
while let Some(x) = iter.next() {
    println!("Matched {:?}", x);
    // Matched 1
    // Matched 2
    // Matched 3
}
```

### ref 和 ref mut

#### 使用场景

- 借用数据而不转移所有权：在某些情况下，你只需要借用数据而不是转移其所有权。例如在递归数据结构中，借用数据可以避免所有权转移带来的复杂性
- 对数据进行修改：使用 ref mut 可以在模式匹配时对数据进行修改，而无需转移所有权

```rust
let x = String::from("Hello");
match x {
    // 借用 x 的所有权，x 所有权没有转移
    ref r => println!("Got a reference to a value: {:?}", r),
}
println!("x is still accessible: {}", x);

let x = String::from("Hello");
match x {
    ref mut r => {
        *r += String::from("world")
    },
}
```

## 课后习题

编写一个使用模式匹配解析 JSON 字符串的程序

- 作业目标
  - 理解如何使用 Rust 的模式匹配功能解析 JSON 数据。
  - 学会使用 serde_json 库进行 JSON 处理。
  - 练习在实际应用场景中使用模式匹配。
- 作业要求
  - 使用 serde_json 库解析 JSON 字符串。
  - 使用模式匹配提取 JSON 对象中的不同字段。
  - 处理不同类型的数据（字符串、数字、数组、嵌套对象等）。
- 作业示例
假设你有一个包含用户信息的 JSON 字符串：

```json
{
  "name": "Alice",
  "age": 30,
  "email": "alice@example.com",
  "address": {
    "street": "123 Main St",
    "city": "Wonderland"
  },
  "phone_numbers": ["123-456-7890", "987-654-3210"]
}
```

``` rust
use serde_json::{Result, Value};
use std::collections::HashMap;

fn main() -> Result<()> {
    // 原始 JSON 数据
    // 如果字符串包含双引号，可以在开头和结尾加 #
    let json_str = r#"
    {
        "name": "Alice",
        "age": 30,
        "email": "alice@example.com",
        "address": {
            "street": "123 Main St",
            "city": "Wonderland"
        },
        "phone_numbers": ["123-456-7890", "987-654-3210"]
    }
    "#;

    // 步骤 1：解析 JSON 字符串为动态类型 Value
    // serde_json::from_str 返回 Result<Value, Error>
    let v: Value = serde_json::from_str(json_str)?;

    // 步骤 2：使用模式匹配处理 JSON 结构
    match v {
        Value::Object(obj) => {
            let mut name = String::from("Unknown");
            let mut age = 0;
            let mut address = HashMap::new();
            let mut phone_numbers = Vec::new();
            for (key, value) in obj {
                match key.as_str() {
                    "name" => {
                        if let Value::String(s) = value {
                            name = s;
                        }
                    }
                    "age" => {
                        if let Value::Number(n) = value {
                            // as_i64()​​：将 JSON 数值转换为 i64 类型，返回 Option<i64>
                            // unwrap_or(0)：解包 Option<i64>，如果值为 None，则使用默认值 0
                            age = n.as_i64().unwrap_or(0) as i32;
                        }
                    }
                    "address" => {
                        if let Value::Object(addr_obj) = value {
                            let mut street = String::from("Unknown");
                            let mut city = String::from("Unknown");
                            for (addr_key, addr_value) in addr_obj {
                                match addr_key.as_str() {
                                    "street" => {
                                        if let Value::String(s) = addr_value {
                                            street = s;
                                        }
                                    }
                                    "city" => {
                                        if let Value::String(s) = addr_value {
                                            city = s;
                                        }
                                    }
                                    _ => {}
                                }
                            }
                            // 插入 HashMap 键值对
                            address.insert("street", street);
                            address.insert("city", city);
                        }
                    }
                    "phone_numbers" => {
                        if let Value::Array(arr) = value {
                            for num in arr {
                                if let Value::String(s) = num {
                                    phone_numbers.push(s.clone());
                                }
                            }
                        }
                    }
                    _ => {}
                }
            }
            println!("Name: {}", name);
            println!("Age: {}", age);
            println!("Address: {:?}", address);
            println!("Phone Numbers: {:?}", phone_numbers);
        }
        _ => println!("Invalid JSON structure"),
    }
    Ok(())
}
```