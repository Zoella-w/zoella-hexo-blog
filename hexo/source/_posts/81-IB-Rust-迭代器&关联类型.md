---
title: IB-Rust-迭代器
date: 2025-06-14 15:28:32
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson23](https://github.com/Zoella-w/IB-Rust/tree/main/23_iter)

## 什么是迭代器？

- 迭代器模式：对一系列项执行某些任务
- 迭代器负责：
  - 遍历每个项
  - 确定序列（遍历）何时完成
- Rust 的迭代器：
  - 懒惰的：除非调用消费迭代器的方法，否则迭代器本身没有任何效果

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
// rust 的迭代器是懒惰的，除非调用消费迭代器的方法，否则本身没有效果
for val in v1_iter {
    println!("got: {val}");
}
```

## 迭代器的实现

迭代器是一个能够逐一生成元素的对象。它提供了一个统一的接口，用于遍历容器中的元素，同时保证了类型安全和内存安全

在 Rust 中，迭代器是实现了 `Iterator` trait 的对象。该 trait 定义了一个 `next` 方法，用于返回下一个元素

所有迭代器都实现了 `Iterator Trait`，其定义于标准库，大致如下：

```rust
pub trait Iterator{
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // methods with default implementations elided
}
```

`type Item` 和 `Self::item` 定义了与此该 trait 关联的类型。实现 Iterator trait 需要定义一个 Item 类型，用于 next 方法的返回类型（迭代器的返回类型）

## 关联类型

关联类型是 `Trait` 中的类型占位符，它可以用于 `Trait` 的方法签名中，定义出包含某些类型的 `Trait`，而在实现前无需知道这些类型是什么

### 关联类型与泛型的区别

- 范型
  - 每次实现 Trait 时标注类型
  - 可以为一个类型多次实现某个 `Trait`（不同的泛型参数）
- 关联类型
  - 无需标注类型
  - 无法为单个类型多次实现某个 `Trait`

```rust
struct Counter {}

pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
    // methods with default implementations elided
}
impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<u32> {
        None
    }
}
// // conflicting implementations of trait `Iterator` for type `Counter`
// impl Iterator for Counter {
//     type Item = String;
//     fn next(&mut self) -> Option<String> {
//         None
//     }
// }

pub trait Iterator2<T> {
    fn next(&mut self) -> Option<T>;
}
impl Iterator2<u32> for Counter {
    fn next(&mut self) -> Option<u32> {
        None
    }
}
impl Iterator2<String> for Counter {
    fn next(&mut self) -> Option<String> {
        None
    }
}
```

## 简单迭代器

```rust
let numbers = vec![1, 2, 3, 4, 5];
let mut iter = numbers.iter(); // 创建迭代器（必须使用 mut）
while let Some(num) = iter.next() { // 使用迭代器逐一获取元素
    println!("{}", num);
}
```

- `numbers.iter()` 创建了一个迭代器，该迭代器按顺序返回 `numbers` 向量中的每个元素
- `iter.next()` 返回迭代器中的下一个元素。如果没有更多元素，返回 `None`

## 几个迭代的方法

- `.iter()`：在不可变引用上创建迭代器
- `.into_iter()`：创建的迭代器会获得所有权
- `.iter_mut()`：迭代可变的引用

## 消耗迭代器的方法

在标准库中，`Iterator trait` 有一些带默认实现的方法
- 调用 next 的方法叫做“消耗型适配器”
  - 因为调用它们会把迭代器消耗尽
- `sum` 方法 也会耗尽迭代器
  - 取得迭代器的所有权
  - 通过反复调用 next，遍历所有元素
  - 每次迭代，把当前元素添加到一个总和里，迭代结束，返回总和

```rust
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
let total: i32 = v1_iter.sum();
println!("total: {total}");
```

## 产生其他迭代器的方法

定义在 `Iterator trait` 上的另外一些方法叫做“迭代器适配器”，可以把迭代器转换为不同种类的迭代器

可以通过链式调用使用多个迭代器适配器来执行复杂的操作，这种调用可读性较高
  
## `map` 方法

`.map()` 允许对迭代器中的每个元素应用一个函数，并返回一个新的迭代器

```rust
let numbers = vec![1, 2, 3, 4, 5];
let squares: Vec<_> = numbers.iter().map(|x| x * x).collect();
println!("{:?}", squares);
```

- `.map()` 将一个闭包应用于每个元素，这里是计算平方
- `.collect()` 是消耗型适配器，将迭代器的结果收集到一个容器中，这里是 `Vec<i32>`

## `filter` 方法

`.filter()` 允许根据条件筛选元素，并返回满足条件的元素的迭代器

```rust
let numbers = vec![1, 2, 3, 4, 5];
let even_numbers: Vec<_> = numbers.iter().filter(|&x| x % 2 == 0).collect();
println!("{:?}", even_numbers);
```

- `.filter()` 筛选出所有满足闭包条件（偶数）的元素

## 自定义迭代器

可以通过实现 `Iterator` trait 来创建自己的迭代器。必须实现 `next` 方法，定义迭代器如何产生下一个元素

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count <= 5 {
            Some(self.count)
        } else {
            None
        }
    }
}

fn main() {
    let mut counter = Counter::new();
    while let Some(count) = counter.next() {
        println!("{}", count);
    }
}
```

- `Counter` 结构体实现了 `Iterator` trait，`next` 方法每次返回下一个计数值，直到 5 为止

## 课后作业：实现自定义迭代器

请你实现一个自定义迭代器，用于生成斐波那契数列。你的迭代器应该支持无限生成斐波那契数，直到用户停止迭代。
- 任务要求：
  - 实现一个结构体 `Fibonacci`，并为它实现 `Iterator` trait。
  - 在 `next` 方法中生成下一个斐波那契数。
  - 编写一个测试函数，输出前 10 个斐波那契数。
- 提示：
  - 你可以使用两个字段来存储当前和前一个斐波那契数。
  - `take` 方法是一个迭代器适配器，用于限制生成的数量。
  - 注意边界条件，例如处理第一个和第二个斐波那契数。

### 示例：
```rust
struct Fibonacci {
    // 在这里定义所需的字段
}
impl Fibonacci {
    fn new() -> Self {
        // 初始化结构体
    }
}
impl Iterator for Fibonacci {
    type Item = u64;
    fn next(&mut self) -> Option<Self::Item> {
        // 实现斐波那契数生成逻辑
    }
}
fn main() {
    let fib = Fibonacci::new();
    for number in fib.take(10) {
        println!("{}", number);
    }
}
```

### 额外挑战（可选）：
- 修改迭代器，使其可以接受一个上限参数，当生成的斐波那契数超过这个上限时停止生成。
- 实现一个 `into_vec` 方法，将生成的斐波那契数列转换为一个 `Vec<u64>`。

```rust
struct Fibonacci {
    cur: u64,
    next: u64,
    count: u64,
    limit: u64,
}
impl Fibonacci {
    fn new(limit: u64) -> Self {
        Fibonacci {
            cur: 1,
            next: 1,
            count: 1,
            limit: limit,
        }
    }
    fn into_vec(&mut self) -> Vec<u64> {
        let mut v = vec![];
        while let Some(count) = self.next() {
            v.push(count);
        }
        v
    }
}
impl Iterator for Fibonacci {
    type Item = u64;
    fn next(&mut self) -> Option<Self::Item> {
        let cur = self.cur;
        self.cur = self.next;
        self.next = cur + self.next;
        if (self.count <= self.limit) {
            self.count += 1;
            Some(cur)
        } else {
            None
        }
    }
}
fn main() {
    let mut fib = Fibonacci::new(5);
    // for number in fib.take(10) {
    //     println!("{}", number);
    // }
    for number in fib.into_vec().iter() {
        println!("{}", number);
    }
}
```
