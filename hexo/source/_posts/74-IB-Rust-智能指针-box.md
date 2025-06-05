---
title: IB-Rust-智能指针&box
date: 2025-06-04 13:18:38
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson16](https://github.com/Zoella-w/IB-Rust/tree/main/16_smart_pointer_box)

## 智能指针

### 智能指针概述

智能指针（Smart Pointers）是一类数据结构，不仅包含一个指针，还附带一些额外的元数据和功能。智能指针实现了 `Deref` 和 `Drop` 两个 trait，使得它们可以像指针一样解引用并在离开作用域时自动清理资源

### 智能指针作用

- 资源管理
  - 自动管理资源的分配和释放，避免内存泄漏
- 所有权与借用
  - Rust 的所有权系统通过智能指针来确保内存安全，避免数据竞争和悬垂指针
- 复杂数据结构
  - 通过智能指针可以构建复杂的数据结构，如递归结构、共享数据等

## `BOX<T>`

- `Box<T>` 将类型 T 的值分配在堆上，而不是栈上
- 当 Box 被销毁时，堆上的数据也会被销毁

### Box 的底层实现

- 底层原理
  - `Box<T>` 实际上是一个智能指针，内部包含一个指向堆上分配内存的裸指针
  - 当 `Box<T>` 被销毁时，其 `Drop` trait 会被调用，释放堆上的内存
- 内存分配
  - Rust 使用系统的全局分配器（如 malloc 和 free）来管理堆内存
  - Box::new 分配内存，Drop 释放内存
- 安全性
  - Rust 的所有权系统确保 `Box<T>` 的内存安全。所有权转移时，堆内存的生命周期也会随之变化

### Box 的使用场景

#### 堆分配
   
Box 最常见的用途是将数据分配在堆上，而不是栈上。这在处理较大数据结构、或数据结构的大小在编译时不确定时尤为重要

```rust
let b = Box::new(5);
println!("b = {}", b);
```

#### 动态大小类型（DST）

Box 允许处理动态大小类型，如 `str` 和 `[T]`

```rust
let s: Box<str> = "Hello, world!".into();
println!("s = {}", s);
let arr: Box<[i32]> = vec![1, 2, 3, 4, 5].into_boxed_slice();
println!("arr = {:?}", arr);
```

#### 递归数据结构

递归数据结构需要指针类型引用自身，而 Box 提供了这一功能

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
use List::{Cons, Nil};
fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

#### 类型擦除

`Box<dyn Trait>` 用于类型擦除，允许在运行时决定类型

```rust
trait Animal {
    fn speak(&self);
}
struct Dog;
struct Cat;
impl Animal for Dog {
    fn speak(&self) {
        println!("Woof!");
    }
}
impl Animal for Cat {
    fn speak(&self) {
        println!("Meow!");
    }
}
fn main() {
    let animals: Vec<Box<dyn Animal>> = vec![Box::new(Dog), Box::new(Cat)];
    for animal in animals.iter() {
        animal.speak();
    }
}
```

### `dyn` 关键字

`dyn` 关键字用于指定动态分发的类型，允许在运行时决定具体类型，可用于实现动态分发的 trait 对象

### 内存管理和性能优化

通过使用 Box，可以控制内存的分配和释放，从而优化性能和内存使用。例如，将大型数据结构放在堆上，而不是栈上，从而避免栈溢出

```rust
let large_array = Box::new([0u8; 1_000_000]);
println!("Large array allocated on the heap.");
```

## Box 的优缺点

- 优点
  - 提供堆内存分配，支持复杂数据结构
  - 与 Rust 的所有权系统完美集成，确保内存安全
  - 动态分配对象，实现类型擦除
- 缺点
  - 需要堆内存分配和释放，可能带来性能开销
  - 不适合需要频繁分配和释放的场景

## Drop、Derefb 和 DerefMut

- `Drop` Trait
  - `Drop` trait 定义了当一个值离开作用域时应该执行的操作
  - 例如：`Box<T>` 在超出作用域时会自动调用其 Drop trait，释放堆上的内存
- `Deref` Trait
  - `Deref` trait 定义了如何将一个类型转换为引用
  - 例如：`Box<T>` 实现了 Deref，所以可以通过 * 运算符解引用获取其内部数据

### `Drop` Trait

`Drop` trait 用于自定义当值离开作用域时执行的代码，通常用于释放资源（例如内存、文件句柄、网络连接等）

#### 定义和实现

`Drop` trait 定义了一个 drop 方法，当值被释放时，Rust 会自动调用这个方法

```rust
pub trait Drop {
  fn drop(&mut self);
}
```

```rust
struct Resource {
    name: String,
}
// 正确实现标准库的 Drop trait
impl Drop for Resource {
    fn drop(&mut self) {
        println!("{} is dropped", self.name);
    }
}
fn box_study() {
    let _r1 = Resource {
        name: "r1".to_string(),
    };
    {
        let _r2 = Resource {
            name: "r2".to_string(),
        };
    }  // r2 在这里离开作用域
}  // r1 在这里离开作用域
fn main() {
    box_study();
}
```

### `Deref` Trait

`Deref` trait 用于重载解引用运算符（*），允许定义自定义指针类型的解引用行为

#### 定义和实现

`Deref` trait 定义了一个 deref 方法，该方法返回指向目标类型的引用

```rust
pub trait Deref {
  type Target: ?Sized;
  fn deref(&self) -> &Self::Target;
}
```
```rust
use std::ops::Deref;
fn main() {
  let x = 5;
  let y = MyBox::new(x);
  println!("x = {}", x);
  // 触发解引用运算符重载
  // 等价于 *(y.deref())（编译器自动转换）
  println!("y = {}", *y);
}
struct MyBox<T>(T);
// 构造函数实现
impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
impl<T> Deref for MyBox<T> {
  // 关联类型
  type Target = T;
  fn deref(&self) -> &T {
    println!("deref called");
    // 访问元组结构体的第一个字段
    &self.0
  }
}
```

### `DerefMut` Trait

与 Deref 类似，DerefMut 用于重载可变解引用运算符（*），允许对自定义类型进行可变解引用

#### 定义和实现

```rust
pub trait DerefMut: Deref {
  fn deref_mut(&mut self) -> &mut Self::Target;
}
```

```rust
impl<T> DerefMut for MyBox<T> {
  fn deref_mut(&mut self) -> &mut Self::Target {
    println!("deref called");
    // 访问元组结构体的第一个字段
    &mut self.0
  }
}
```

## 课后作业

### 作业 1: 内存管理和性能优化

创建一个大型数组并将其分配到堆上，然后测量和比较分配在堆和栈上的性能差异。

创建一个包含 1_000_000 个元素的数组，分别将其分配在堆和栈上。使用 `std::time::Instant` 来测量分配和访问时间。

```rust
use std::time::Instant;

fn main() {
  test_stack_allocation();
  test_heap_allocation();
}

fn test_stack_allocation() {
  // 栈上分配测试 (1,000,000 个整数)
  let n = 1_000_000;
  // 分配
  let start = Instant::now();
  let mut arr: [u32; 1_000_000] = [0; 1_000_000]; // 改用u32避免负数问题
  let duration = start.elapsed();
  println!("分配时间: {:?}", duration); // 分配时间: 58.75µs
  // 写入
  let start = Instant::now();
  for i in 0..n {
      arr[i] = i as u32;
  }
  let duration = start.elapsed();
  println!("写入时间: {:?}", duration); // 写入时间: 7.7795ms
  // 读取和求和
  let start = Instant::now();
  let mut sum: u64 = 0; // 使用u64来容纳大数字
  for i in 0..n {
      sum += arr[i] as u64; // 将每个元素转为u64
  }
  let duration = start.elapsed();
  println!("读取时间: {:?}\n", duration); // 读取时间: 7.601417ms
}

fn test_heap_allocation() {
  // 堆上分配测试 (1,000,000 个整数)
  let n = 1_000_000;
  // 分配
  let start = Instant::now();
  let mut arr = Box::new([0u32; 1_000_000]); // 明确类型为u32
  let duration = start.elapsed();
  println!("分配时间: {:?}", duration); // 分配时间: 421.75µs
  // 写入
  let start = Instant::now();
  for i in 0..n {
      arr[i] = i as u32;
  }
  let duration = start.elapsed();
  println!("写入时间: {:?}", duration); // 写入时间: 6.979958ms
  // 读取和求和
  let start = Instant::now();
  let mut sum: u64 = 0;
  for i in 0..n {
      sum += arr[i] as u64;
  }
  let duration = start.elapsed();
  println!("读取时间: {:?}", duration); // 读取时间: 7.15825ms
}
```

### 作业2：实现一个简单的文件系统模拟

- 目标
实现一个简单的文件系统模拟，其中包含文件和文件夹的概念。文件夹可以包含文件和其他文件夹。使用 Box 来管理内存，并实现对文件系统的基本操作（如创建文件、创建文件夹、列出文件和文件夹）。

- 作业要求
  - 定义 `FileSystem` trait 和 Node 枚举
    - `FileSystem` trait 包含 `create_file`、`create_folder` 和 `list_contents` 方法。
    - `Node` 枚举包含 `File` 和 `Folder` 变体。
  - 实现 `FolderNode` 结构体
    - `FolderNode` 实现 `FileSystem` trait，包含 `name` 和 `contents` 字段。
    - 使用 `Box` 管理 `contents` 中的子节点。
  - 实现文件系统的基本操作
    - `create_file` 方法在文件夹中创建文件。
    - `create_folder` 方法在文件夹中创建子文件夹。
    - `list_contents` 方法列出文件夹的所有内容。
  - 测试文件系统的操作
    - 创建根文件夹并添加文件和文件夹。
    - 创建子文件夹并添加文件。
    - 列出文件夹的内容并输出文件系统结构。
- 提示
使用 `Box` 来管理 `Folder` 中的子节点。
使用递归方法来遍历和列出文件和文件夹的内容。
考虑使用 `Vec` 来存储文件夹的子节点。

```rust
trait FileSystem {
    fn create_file(&mut self, name: &str) -> Result<(), String>;
    fn create_folder(&mut self, name: &str) -> Result<(), String>;
    fn list_contents(&self, indent: usize);
}
// 定义 Node 枚举，包含：文件节点 和 文件夹节点
enum Node {
    // File 和 Folder 是变体的名称
    // (FileNode) 和 (FolderNode) 是变体关联的数据类型
    File(FileNode),
    Folder(FolderNode),
}
// 文件节点结构
struct FileNode {
    name: String,
}
// 文件夹节点结构
struct FolderNode {
    name: String,
    contents: Vec<Box<Node>>, // 使用 Box<Node> 存储子节点
}
impl FileSystem for FolderNode {
    fn create_file(&mut self, name: &str) -> Result<(), String> {
        // 检查是否同名节点已存在
        // *node 对 &Box<Node> 解引用得到 Box<Node>
        // **node 对 Box<Node> 解引用得到 Node 值
        // &**Node 取得对 Node 的引用（&Node），为了避免所有权的移动
        if self.contents.iter().any(|node| match &**node {
            Node::File(f) => f.name == name,
            Node::Folder(f) => f.name == name,
        }) {
            return Err("Name already exists".to_string());
        }
        // 创建新文件
        self.contents.push(Box::new(Node::File(FileNode {
            name: name.to_string(),
        })));
        Ok(())
    }

    fn create_folder(&mut self, name: &str) -> Result<(), String> {
        // 检查是否同名节点已存在
        if self.contents.iter().any(|node| match &**node {
            Node::File(f) => f.name == name,
            Node::Folder(f) => f.name == name,
        }) {
            return Err("Name already exists".to_string());
        }
        // 创建新文件夹
        self.contents.push(Box::new(Node::Folder(FolderNode {
            name: name.to_string(),
            contents: Vec::new(),
        })));
        Ok(())
    }

fn list_contents(&self, indent: usize) {
        // 递归列出所有内容
        // .iter() 遍历 self.contents 中的每个元素（&Box<Node> 类型）
        // .enumerate() 会将迭代器转换为新的迭代器，新迭代器产生元组 (index, item)
        for (i, node) in self.contents.iter().enumerate() {
            let is_last = i == self.contents.len() - 1;
            let prefix = if is_last { "└──" } else { "├──" };
            let item_indent = indent + 2;

            match &**node {
                // {:indent$} 功能：创建指定数量的空格缩进
                // indent$ 是一个 ​​命名参数占位符
                // indents 的值为 item_indent
                Node::File(file) => println!(
                    "{:indent$}{} {} (File)",
                    "",
                    prefix,
                    file.name,
                    indent = item_indent
                ),
                Node::Folder(folder) => {
                    // 打印子文件夹名称作为父文件夹的子项
                    println!(
                        "{:indent$}{} {} (Folder)",
                        "",
                        prefix,
                        folder.name,
                        indent = item_indent
                    );
                    folder.list_contents(item_indent + 2);
                }
            }
        }
    }
}

fn main() {
    // 创建根文件夹
    let mut root = FolderNode {
        name: "Root".to_string(),
        contents: Vec::new(),
    };
    // 在根目录添加文件和文件夹
    // 使用 .unwrap() 使得出现错误后 painc 退出程序
    root.create_file("document.txt").unwrap();
    root.create_folder("Pictures").unwrap();
    root.create_folder("Music").unwrap();
    // 在 Pictures 文件夹中添加文件
    // .iter_mut() 获取集合的​​可变引用迭代器
    // matches! 宏检查是否匹配模式（名字为 Pictures 的文件夹
    // .map() 将 &mut Box<Node> 类型转为 &mut Node 类型
    // pictures 的类型为 &mut FolderNode
    if let Some(Node::Folder(pictures)) = root
        .contents
        .iter_mut()
        .find(|n| matches!(&***n, Node::Folder(f) if f.name == "Pictures"))
        .map(|n| &mut **n)
    {
        pictures.create_file("photo1.jpg").unwrap();
        pictures.create_file("photo2.jpg").unwrap();
    }
    // 在 Music 文件夹中添加文件
    if let Some(Node::Folder(music)) = root
        .contents
        .iter_mut()
        .find(|n| matches!(&***n, Node::Folder(f) if f.name == "Music"))
        .map(|n| &mut **n)
    {
        music.create_file("song1.mp3").unwrap();
        music.create_folder("Classical").unwrap();
    }
    // 列出整个文件系统结构
    println!("File System Structure:");
    root.list_contents(0);
}
```
