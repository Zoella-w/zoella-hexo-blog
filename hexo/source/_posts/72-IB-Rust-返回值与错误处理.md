---
title: IB-Rust-返回值与错误处理
date: 2025-06-02 21:59:06
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson14](https://github.com/Zoella-w/IB-Rust/tree/main/14_return_error)


## Option 返回值

```rust
let mut s = String::from("A");
let p1 = s.pop();
dbg!(p1); // Some('A')
let p2 = s.pop();
dbg!(p2); // None
```

### 解构 Option

```rust
enum Option<T> {
    Some(T),
    None,
}
```

### 匹配 `Option`

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}
fn main() {
    let five = Some(5);
    let six = plus_one(five); // Some(6)
    let none = plus_one(None); // None
}
```

### `Option<T>` 的辅助函数

>  Method overview https://doc.rust-lang.org/std/option/

#### 使用 unwrap

如果确定 Option 存在值，可以使用 `unwrap` 方法来获取该值；如果 Option 不存在值，则会触发 panic

```rust
let mut s = String::from("A");
let p1 = s.pop().unwrap(); // "A"
// let p2 = s.pop().unwrap(); // panic: called `Option::unwrap()` on a `None` value
```

#### 使用 is_some 和 is_none

可以使用 `is_some` 和 `is_none` 方法来判断 Option 中是否存在值。

```rust
let v = [10, 40, 30];
if v.get(1).is_some() {
    println!("{}", v[1]); // 40
}
```

#### 使用 unwrap_or 提供默认值

```rust
fn div(a: i32, b: i32) -> Option<f64> {
    if b != 0 {
        Some(a as f64 / b as f64)
    } else {
        None
    }
}
fn main() {
    let a = 10;
    let b = 0;
    let result = div(a, b).unwrap_or(0.0); // 0.0
}
```

## 错误处理

Rust 中的错误主要分为两类：

- **可恢复错误**，通常是从系统全局角度看可以接受的错误，例如处理用户的访问、操作等错误，这些错误只会影响某用户自身的操作进程，而不会影响系统的全局稳定性
- **不可恢复错误**，通常是全局性或者系统性的错误，例如数组越界访问，系统启动时发生了影响启动流程的错误等，这些错误对系统来说往往是致命的

- `Result<T, E>` 用于可恢复错误
- `panic!` 用于不可恢复错误

## 用 `panic!` 处理不可恢复的错误

### 触发

#### 被动触发

C 语言中，读取数据结构未定义的值，会得到对应数据结构中这个元素内存位置的值（这些内存可能并不属于该数据结构），这被称为**缓冲区溢出**，并会导致安全漏洞，比如攻击者可以通过操作索引，读取储存在数据结构之后不被允许的数据

为防范此类漏洞，如果读取索引不存在的元素，Rust 会停止执行，发生 panic

```rust
let v = vec![1, 2, 3];
v[99];
```

#### 主动调用

Rust 提供了 `panic!` 宏，当调用执行该宏时，程序会打印出错误信息，展开报错点之前的函数调用堆栈，并退出程序

```rust
panic!("crash and burn");
```

### backtrace 栈展开

```shell
RUST_BACKTRACE=1 cargo run
```

在栈展开（或栈回溯）中，函数调用按照逆序排列：最近调用的排在最上方

要获取栈回溯信息，还需要开启 `debug` 标志。当不使用 `--release` 参数运行 `cargo build` 或 `cargo run` 时， `debug` 标识会默认启用

### panic 的两种终止方式

- **栈展开（*unwinding*）（默认）**: Rust 会回溯栈上数据和函数调用，可以给出充分的报错和栈调用信息，便于问题复盘，不过也意味着更多的善后工作
- **直接终止（*abort*）**: 不清理数据就直接退出程序，善后工作交与操作系统负责

大多数情况，使用默认选择最好；当关心编译出的二进制可执行文件大小时，可以使用直接终止的方式，例如下面的 `Cargo.toml` 配置文件，可以实现在 `release` 模式下，遇到 `panic` 直接终止：

```toml
[profile.release]
panic = 'abort'
```
```shell
RUST_BACKTRACE=1 cargo run --release
```

## 用 `Result` 处理可恢复的错误

大部分错误并没有严重到需要程序完全停止执行，例如，如果因为打开一个并不存在的文件而失败，此时我们可能想创建这个文件，而不是终止进程

- `Ok(T)`：泛型参数 `T` 代表成功时存入的正确值的类型
- `Err(E)`：`E` 代表错误时存入的错误值，存放方式是

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 返回 `Result`

#### 例1

```rust
fn div(x: f64, y: f64) -> Result<f64, String> {
    if y == 0.0 {
        // 此操作将会失败，那么（与其让程序崩溃）不如把失败的原因包装在 Err 中并返回
        Err(("y is zero").to_string())
    } else {
        // 此操作是有效的，返回包装在 Ok 中的结果
        Ok(x / y)
    }
}
```

#### 例2

```rust
#[derive(Debug)]
pub enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
}
fn div(x: f64, y: f64) -> Result<f64, MathError> {
    if y == 0.0 {
        Err(MathError::DivisionByZero)
    } else {
        Ok(x / y)
    }
}
fn sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}
```

### 处理 `Result` 

```rust
use std::fs::File;
fn main() {
    let f = File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

#### 对返回的错误进行处理

对不同的错误原因采取不同的行为：

- 如果 `File::open `因为文件不存在而失败，我们希望创建这个文件并返回新文件的句柄
- 如果 `File::open` 因为任何其他原因失败，例如没有打开文件的权限，我们仍然 `panic!`

```rust
use std::fs::File;
use std::io::ErrorKind;
fn main() {
    let greeting_file_result = File::open("hello.txt");
    // `File::open` 返回的 `Err` 成员中的值类型 `io::Error` 是一个标准库中提供的结构体
    // 该结构体有一个返回 `io::ErrorKind` 值的 `kind` 方法可供调用
    // `io::ErrorKind` 是一个标准库提供的枚举，其成员对应 `io` 操作可能导致的不同错误类型
    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {e:?}"),
            },
            other_error => {
                panic!("Problem opening the file: {other_error:?}");
            }
        },
    };
}
```

### `Result<T, E>` 的辅助方法

`match` 能够胜任，不过可能有点冗长且不总能很好的表明其意图`Result<T, E>` 类型定义了很多辅助方法来处理各种情况

> Method overview: https://doc.rust-lang.org/std/result/index.html

#### unwrap

- 如果 `Result` 值是成员 `Ok`，`unwrap` 会返回 `Ok` 中的值
- 如果 `Result` 是成员 `Err`，`unwrap` 会调用 `panic!`

```rust
use std::fs::File;
fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

#### expect

`expect` 跟 `unwrap` 很像，也是遇到错误直接 `panic`, 但会有自定义的错误提示信息，相当于重载了错误打印的函数：

```rust
use std::fs::File;
fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

#### map

`Result<T, E>` -> `Result<U, E>`

```rust
let line = "1\n2\n3\n4\n";
for num in line.lines() {
    match num.parse::<i32>().map(|i| i * 2) {
        Ok(n) => println!("{n}"),
        Err(..) => {}
    }
}
```

#### map_err

`Result<T, E>` ->`Result<T, F>`

```rust
fn x() -> Result<(), String> {
    let f = File::open("hello.txt").map_err(|e: std::io::Error| format!("{e}") );
    match f{
        Err(e)=>Err(e),
        Ok(_)=>{
            Ok(())
        }
    }
}
```

### 传播错误

可以使用 `?` 运算符来简写

对于 Result

- 如果结果是 `Ok(T)`，则把 `T` 赋值给 `f`
- 如果结果是 `Err(E)`，则返回该错误

对于 Option

- 如果值是 `Some`，`Some` 中的值作为表达式的返回值同时函数继续
- 如果值是 `None`，此时 `None` 会从函数中提前返回

```rust
use std::fs::File;
use std::io::Read;
fn read_username_from_file() -> Result<String, std::io::Error> {
    // // 打开文件，f是`Result<文件句柄,io::Error>`
    // let f = File::open("hello.txt");
    // let mut f = match f {
    //     // 打开文件成功，将file句柄赋值给f
    //     Ok(file) => file,
    //     // 打开文件失败，将错误返回(向上传播)
    //     Err(e) => return Err(e),
    // };
    let mut f = File::open("hello.txt")?;

    // 创建动态字符串s
    let mut s = String::new();

    // // 从f文件句柄读取数据并写入s中
    // match f.read_to_string(&mut s) {
    //     // 读取成功，返回Ok封装的字符串
    //     Ok(_) => Ok(s),
    //     // 将错误向上传播
    //     Err(e) => Err(e),
    // }
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

### 链式方法调用

```rust
fn read_username_from_file() -> Result<String, std::io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

### 常见错误

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   arr.get(0)?
   // 正确
   // let a = arr.get(0)?;
   // return a;
}
```

这段代码无法通过编译：`?` 操作符需要一个变量来承载值，这个函数只会返回 `Some(&i32)` 或者 `None`。而事实上，只有错误值能直接返回，正确的值不行，因为如果数组存在 0 号元素，函数第二行使用 `?` 后的返回类型是 `&i32` 而不是 `Some(&i32)`

## Option 与 Result 的互相转换

### `Option` -> `Result`: `ok_or()`

-  `Some(v)` to `Ok(v)`
-  `None` to `Err(err)` 

```rust
fn first(arr: &[i32]) -> Result<&i32, &str> {
    arr.get(0).ok_or("out of index")
}
```

### `Result` -> `Opiton`: `err()`

- `Err(e)` to `Some(e)` 
- `Ok(v)` to `None`

```rust
let f = File::open("hello.txt").err();
if f.is_some() {
    println!("no file");
}
```

### `Result` -> `Opiton`: `ok()`

- `Ok(e)` to `Some(e)` 
- `Err(v)` to `None`

```rust
let f = File::open("hello.txt").ok();
if f.is_none(){
    println!("no file");
}
```

## 课后习题

```rust
// 修复 call 函数的错误
// 当 b 为 None 时，按默认值 1
fn call(a: i32, b: i32) -> Result<f64, String> {
    let r = divide(a, b); 
    let s = sqrt(r);
    
    Ok(s);
}

fn divide(a: i32, b: i32) -> Option<f64> {
    if b != 0 {
        Some(a as f64 / b as f64)
    } else {
        None
    }
}

pub enum MathError {
    DivisionByZero,
    NegativeSquareRoot,
}

fn sqrt(x: f64) -> Result<f64, MathError> {
    if x < 0.0 {
        Err(MathError::NegativeSquareRoot)
    } else {
        Ok(x.sqrt())
    }
}
```

#### 答案

```rust
fn call(a: i32, b: Option<i32>) -> Result<f64, String> {
    let r = divide(a, b.unwrap_or(1)).ok_or("Division by zero".to_string())?;
    let s = sqrt(r).map_err(|e: MathError| -> String {
        match e {
            MathError::DivisionByZero => {
                return "DivisionByZero".to_string();
            }
            MathError::NegativeSquareRoot => {
                return "NegativeSquareRoot".to_string();
            }
        }
    })?;
    return Ok(s);
}
```
