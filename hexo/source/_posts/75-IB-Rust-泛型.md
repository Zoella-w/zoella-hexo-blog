---
title: IB-Rust-泛型
date: 2025-06-05 22:21:27
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson17](https://github.com/Zoella-w/IB-Rust/tree/main/17_generics)

## 泛型的例子

用同一功能的函数处理不同类型的数据，例如两个数的加法，无论是整数、浮点数、还是自定义类型，都能支持。在不支持泛型的编程语言中，需要为每种类型编写一个函数：

```rust
fn add_i8(a:i8, b:i8) -> i8 {
    a + b
}
fn add_i32(a:i32, b:i32) -> i32 {
    a + b
}
fn add_f64(a:f64, b:f64) -> f64 {
    a + b
}
fn main() {
    println!("add i8: {}", add_i8(2i8, 3i8));
    println!("add i32: {}", add_i32(20, 30));
    println!("add f64: {}", add_f64(1.23, 1.23));
}
```

### 泛型

当使用泛型定义函数时，本来在函数签名中指定参数和返回值的类型的地方，会改用泛型来表示。这种技术能使代码适应性更强，从而为函数调用者提供更多的功能，同时也避免了代码重复

```rust
fn add<T>(a:T, b:T) -> T {
    a + b
}
```

上面代码的 `T` 就是**泛型参数**，泛型参数的名称可以为任意，但是惯例一般用 `T`作为首选。该名称越短越好，除非需要表达含义

不是所有 `T` 类型都能进行相加操作，因此需要用 `std::ops::Add<Output = T>` 对 `T` 进行限制：

```rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
    a + b
}
fn main() {
    println!("add i8: {}", add(2i8, 3i8));
    println!("add i32: {}", add(20, 30));
    println!("add f64: {}", add(1.23, 1.23));
}
```

## 在函数定义中使用泛型

不是所有的类型都能进行比较，使用标准库中定义的 `std::cmp::PartialOrd` trait 可以实现类型的比较功能，限制 `T` 只对实现了 `PartialOrd` 的类型有效（`i32` 和 `char`）

```rust
fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is {}", result);
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

## 结构体定义中的泛型

- 结构体名称后面的尖括号中声明泛型参数的名称，结构体定义中可以指定具体数据类型的位置
- x 和 y 是相同的类型

```rust
struct Point<T> {
    x: T,
    y: T,
}
fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
    // let wont_work = Point { x: 5, y: 4.0 }; // error: mismatched types
}
```

定义一个 `x` 和 `y` 有不同类型且仍是泛型的 `Point` 结构体：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

## 枚举中使用泛型

```rust
enum Option<T> {
    Some(T),
    None,
}
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## 方法中使用泛型

其他 `T` 不是 `i32` 类型的 `Point<T>` 实例没有定义此方法

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl Point<i32> {
    fn x(&self) -> &i32 {
        &self.x
    }
}
```

#### `impl` 之后声明泛型 `T`

泛型参数可以与结构体定义中声明的泛型参数不同

```rust
struct Point<T> {
    x: T,
    y: T,
}
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

#### 方法使用了与结构体定义中不同类型的泛型

```rust
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}
impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };
    let p3 = p1.mixup(p2); // {x: 5, y: c}
}
```

## const 泛型（Rust 1.51 版本引入的重要特性）

`[i32; 3]` 和 `[i32; 2]` 确实是两个完全不同的类型，因此无法用同一个函数调用

```rust
fn display_array(arr: [i32; 3]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);
    let arr: [i32; 2] = [1,2];
    // display_array(arr); // error: mismatched types
}
```

### 方式1: 切片+泛型

- 使用数组切片，传入 `arr` 的不可变引用
- `i32` 改成所有类型T的数组
- 需要对 `T` 加一个限制 `std::fmt::Debug`，表明 `T` 可以用在 `println!("{:?}", arr)` 中，因为 `{:?}` 形式的格式化输出需要 `arr` 实现该特征

```rust
fn display_array<T: std::fmt::Debug>(arr: &[T]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(&arr);

    let arr: [i32;2] = [1,2];
    display_array(&arr);
}
```

### 方式2: const 泛型

`const N: usize`，表示 const 泛型 `N` 基于的值类型是 `usize`

定义一个类型为 `[T; N]` 的数组：其中的 `N` 是一个基于值的泛型参数，因为它用来替代数组的长度

```rust
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);
    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

## 泛型代码的性能

> Rust 通过在编译时进行泛型代码的 **单态化**（*monomorphization*）来保证效率

对于标准库中的 `Option` 枚举：

```rust
let integer = Some(5);
let float = Some(5.0);
```

编译器生成的单态化版本的代码看起来像这样（名字为假想的）：

```rust
enum Option_i32 {
    Some(i32),
    None,
}
enum Option_f64 {
    Some(f64),
    None,
}
fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

泛型 `Option<T>` 被编译器替换为了具体的定义。Rust 会将每种情况下的泛型代码编译为具体类型，使用泛型没有运行时开销。代码运行的执行效率和手写每个具体定义的重复代码一样。单态化正是 Rust 泛型在运行时极其高效的原因

但是，Rust 在编译期为泛型对应的多个类型，生成各自的代码，因此损失了编译速度，且增大了最终生成文件的大小
