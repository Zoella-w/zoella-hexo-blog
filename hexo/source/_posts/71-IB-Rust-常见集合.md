---
title: IB-Rust-常见集合
date: 2025-05-28 22:24:21
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson13](https://github.com/Zoella-w/IB-Rust/tree/main/13_collection)

Rust 中的常见集合有 Vector 和 HashMap

## Vector

### 什么是 Vec

Vec 是一个动态数组，可以根据需要动态增长和缩小，适用于需要按顺序存储数据的场景

### Vec 的基本操作

#### 创建和初始化

```rust
let v: Vec<i32> = Vec::new();
let v = vec![1, 2, 3];
```

#### 添加元素

```rust
// let mut v = vec![1, 2, 3];
v.push(1);
```

#### 访问元素

```rust
let v = vec![1, 2, 3];

// 直接引用访问
let third = &v[2];
println!("third: {third}");

// 使用 get 访问
match v.get(2) {
    Some(val) => println!("value: {val}"),
    None => println!("Index error"),
};
```

#### 修改元素

```rust
let mut v = vec![1, 2, 3];
v[0] = 0;
```

#### 遍历元素

```rust
// 如果不用引用，v 的所有权会被转移
for i in &v {
    println!("i: {i}");
}
```

### Vec 的进阶用法

#### 使用枚举存储多种类型

```rust
enum SpreadSheetCell {
    Int(i32),
    Float(f64),
    Test(String),
}
let row = vec![
    SpreadSheetCell::Int(3),
    SpreadSheetCell::Float(10.12),
    SpreadSheetCell::Test(String::from("Hello")),
];
```

#### 容量与重新分配

```rust
let mut v: Vec<i32> = Vec::with_capacity(10);
println!("capacity: {}", v.capacity()); // 10
v.push(1);
println!("capacity: {}", v.capacity()); // 10
```

### Vec 的常见陷阱

#### 不安全的索引访问

用 if 判断索引是否合法，或者使用 match

#### 可变引用与不可变引用的混用

```rust
// let mut v = vec![1, 2, 3];
// let first = &v[0]; // 不可变借用
// v.push(4); // 可变借用，error: mutable borrow occurs here
// println!("The first element is : {}", first);

let mut v = vec![1, 2, 3];
{
    let first = &v[0]; // 不可变借用
    println!("The first element is : {}", first);
}
v.push(4); // 可变借用，error: mutable borrow occurs here
println!("v: {:?}", v);
```

## HashMap

### 什么是 HashMap

HashMap 是一个键值对（key-value）存储的数据结构，适用于需要快速查找数据的场景

### HashMap 的基本操作

#### 创建和初始化

```rust
let mut scores: HashMap<_, _> = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("yellow"), 50);
```

#### 访问元素

```rust
let team_name = String::from("Blue");
let score = scores.get(&team_name);
match score {
    Some(s) => println!("The {} score is {}", team_name, s),
    None => println!("No score for this team"),
};
```

#### 遍历元素

```rust
// 使用引用防止所有权转移
for (team, score) in &scores {
    println!("{}: {}", team, score);
}
```

### HashMap 的进阶用法

#### 更新哈希表

```rust
scores.insert(String::from("Blue"), 25);
scores.entry(String::from("Yellow")).or_insert(100);

let entry = scores.entry(String::from("Red")).or_insert(30);
println!("entry: {entry}"); // 30
*entry += 10;
println!("entry: {entry}"); // 40
println!("{:?}", scores);
```

#### 合并哈希表

```rust
let mut map1 = HashMap::new();
map1.insert("a", 1);
map1.insert("b", 2);
let mut map2 = HashMap::new();
map2.insert("b", 3);
map2.insert("c", 4);
for (k, v) in map2 {
    // map1.insert(k, v);
    map1.entry(k).or_insert(v);
}
```

#### 生成哈希 key

```rust
use std::hash::{DefaultHasher, Hash, Hasher};

fn calculate_hash<T: Hash>(t: &T) -> u64 {
    let mut s = DefaultHasher::new();
    t.hash(&mut s);
    s.finish()
}

fn hashmap_study() {
    let key1 = String::from("key1");
    let key2 = String::from("key2");
    println!(
        "hash of key1: {}, and key2: {}",
        calculate_hash(&key1),
        calculate_hash(&key2)
    );
}
```

### HashMap 的常见陷阱

#### 哈希冲突

使用 `.entry().or_insert()`

#### 值的所有权问题

```rust
let field_name = String::from("color");
let field_value = String::from("Blue");
let mut map = HashMap::new();
// map.insert(field_name, field_value);
// println!("field_name: {}", field_name); // error: value borrowed here after move
map.insert(field_name.clone(), field_value.clone());
println!("field_name: {}", field_name);
```

## 练习题
### 练习 1
使用 Vec 实现一个简单的栈
实现一个简单的栈（后进先出，LIFO）数据结构，支持 push、pop 和 peek 操作。

``` rust
struct Stack<T> {
    elements: Vec<T>,
}

impl<T> Stack<T> {
    fn new() -> Self {
        {
            Stack {
                elements: Vec::new(),
            }
        }
    }

    fn push(&mut self, item: T) {
        self.elements.push(item);
    }

    fn pop(&mut self) -> Option<T> {
        self.elements.pop()
    }

    fn peek(&self) -> Option<&T> {
        self.elements.last()
    }
}

fn main() {
    let mut stack = Stack::new();

    // 压入元素
    stack.push(1);
    stack.push(2);
    stack.push(3);

    // 查看栈顶
    assert_eq!(stack.peek(), Some(&3));

    // 弹出元素
    assert_eq!(stack.pop(), Some(3));
    assert_eq!(stack.peek(), Some(&2));

    // 继续弹出和压入
    assert_eq!(stack.pop(), Some(2));
    stack.push(4);
    assert_eq!(stack.peek(), Some(&4));

    // 清空栈
    assert_eq!(stack.pop(), Some(4));
    assert_eq!(stack.pop(), Some(1));
    assert_eq!(stack.pop(), None); // 空栈时返回 None
    assert_eq!(stack.peek(), None); // 空栈时返回 None
}
```

### 练习 2
使用 HashMap 实现一个字频统计器
编写一个程序，统计一个字符串中每个单词出现的频率。

``` rust
use std::collections::HashMap;

fn word_frequency(text: &str) -> HashMap<String, u32> {
    let mut frequency_map = HashMap::new();
    for word in text.split_whitespace() {
        // 清理单词并转换为 String（拥有所有权）
        let cleaned_word = word
            .chars() // 将单词分解为字符迭代器
            .filter(|c| c.is_alphanumeric()) // 保留字母和数字字符，过滤标点符号
            .collect::<String>() // 将过滤后的字符收集为 String
            .to_lowercase();
        if cleaned_word.is_empty() {
            continue;
        }
        // 使用 String 作为键（拥有所有权）
        *frequency_map.entry(cleaned_word).or_insert(0) += 1;
    }
    frequency_map
}

fn main() {
    let text = "Hello world! Hello Rust. Rust is awesome. World says hello to Rust.";
    let frequency = word_frequency(text);

    println!("Word frequency:");
    for (word, count) in &frequency {
        println!("{}: {}", word, count);
    }

    // 验证结果
    assert_eq!(frequency.get("hello"), Some(&3));
    assert_eq!(frequency.get("rust"), Some(&3));
    assert_eq!(frequency.get("world"), Some(&2));
    assert_eq!(frequency.get("is"), Some(&1));
    assert_eq!(frequency.get("says"), Some(&1));
    assert_eq!(frequency.get("to"), Some(&1));
    assert_eq!(frequency.get("awesome"), Some(&1));
}
```

### 练习3
综合练习：使用 Vec 和 HashMap 实现一个简单的书籍库存管理系统
实现一个书籍库存管理系统，可以添加书籍、查询库存、更新库存以及删除书籍。

``` rust
use std::collections::HashMap;
use std::fmt;

// 书籍结构体
#[derive(Debug, Clone)]
struct Book {
    id: u32,        // 唯一标识符
    title: String,  // 书名
    author: String, // 作者
    price: f64,     // 价格
    quantity: u32,  // 库存数量
}

impl Book {
    // 创建新书籍
    fn new(id: u32, title: &str, author: &str, price: f64, quantity: u32) -> Self {
        Book {
            id,
            title: title.to_string(),
            author: author.to_string(),
            price,
            quantity,
        }
    }
}

// 为 Book 实现 Display trait 以便打印
impl fmt::Display for Book {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(
            f,
            "ID: {}, Title: '{}', Author: '{}', Price: ${:.2}, Quantity: {}",
            self.id, self.title, self.author, self.price, self.quantity
        )
    }
}

// 库存管理系统
struct InventoryManager {
    books_by_id: HashMap<u32, Book>,           // 按 ID 快速查找
    books_by_title: HashMap<String, Vec<u32>>, // 按书名查找 ID 列表
    next_id: u32,                              // 下一个可用 ID
}

impl InventoryManager {
    // 创建新的库存管理系统
    fn new() -> Self {
        InventoryManager {
            books_by_id: HashMap::new(),
            books_by_title: HashMap::new(),
            next_id: 1,
        }
    }

    // 添加新书
    fn add_book(&mut self, title: &str, author: &str, price: f64, quantity: u32) {
        let id = self.next_id;
        self.next_id += 1;
        let book = Book::new(id, title, author, price, quantity);
        // 添加到 ID 索引
        self.books_by_id.insert(id, book.clone());
        // 添加到书名索引
        self.books_by_title
            .entry(title.to_lowercase())
            .or_insert_with(Vec::new)
            .push(id);
        println!("Added book: {}", book);
    }

    // 按 ID 查找书籍
    fn find_by_id(&self, id: u32) -> Option<&Book> {
        self.books_by_id.get(&id)
    }

    // 按书名查找书籍
    fn find_by_title(&self, title: &str) -> Vec<&Book> {
        let title_lower = title.to_lowercase();
        self.books_by_title
            .get(&title_lower)
            .map(|ids| {
                ids.iter()
                    .filter_map(|id| self.books_by_id.get(id))
                    .collect()
            })
            .unwrap_or_else(Vec::new)
    }

    // 更新库存数量
    fn update_quantity(&mut self, id: u32, delta: i32) -> Result<(), String> {
        if let Some(book) = self.books_by_id.get_mut(&id) {
            let new_quantity = book.quantity as i32 + delta;
            if new_quantity < 0 {
                return Err(format!(
                    "Cannot update quantity for book ID {}. Negative quantity not allowed.",
                    id
                ));
            }
            book.quantity = new_quantity as u32;
            println!("Updated book ID {}: new quantity = {}", id, book.quantity);
            Ok(())
        } else {
            Err(format!("Book with ID {} not found", id))
        }
    }

    // 删除书籍
    fn remove_book(&mut self, id: u32) -> Result<(), String> {
        if let Some(book) = self.books_by_id.remove(&id) {
            // 从书名索引中移除
            if let Some(ids) = self.books_by_title.get_mut(&book.title.to_lowercase()) {
                ids.retain(|&book_id| book_id != id);
                // 如果该书名下没有其他书籍，移除整个条目
                if ids.is_empty() {
                    self.books_by_title.remove(&book.title.to_lowercase());
                }
            }
            println!("Removed book: {}", book);
            Ok(())
        } else {
            Err(format!("Book with ID {} not found", id))
        }
    }

    // 列出所有书籍
    fn list_all_books(&self) {
        println!("\n--- Inventory Report ---");
        if self.books_by_id.is_empty() {
            println!("No books in inventory");
            return;
        }
        for book in self.books_by_id.values() {
            println!("{}", book);
        }
        println!("Total books: {}", self.books_by_id.len());
    }
}

fn main() {
    let mut inventory = InventoryManager::new();

    // 添加书籍
    inventory.add_book("The Rust Programming Language", "Steve Klabnik", 39.99, 10);
    inventory.add_book("Programming Rust", "Jim Blandy", 49.99, 5);
    inventory.add_book("Rust in Action", "Tim McNamara", 44.99, 8);
    inventory.add_book("The Rust Programming Language", "Carol Nichols", 39.99, 15); // 同名不同作者

    // 查询书籍
    println!("\nSearching for books by title 'Rust':");
    for book in inventory.find_by_title("Rust") {
        println!("- {}", book);
    }
    println!("\nSearching for book ID 2:");
    if let Some(book) = inventory.find_by_id(2) {
        println!("- {}", book);
    }

    // 更新库存
    println!("\nUpdating stock:");
    inventory.update_quantity(1, -3).unwrap(); // 卖出3本
    inventory.update_quantity(1, 5).unwrap(); // 进货5本

    // 尝试无效更新
    match inventory.update_quantity(1, -20) {
        Ok(_) => {}
        Err(e) => println!("Error: {}", e),
    }

    // 删除书籍
    println!("\nRemoving book ID 3:");
    inventory.remove_book(3).unwrap();

    // 列出所有书籍
    inventory.list_all_books();

    // 尝试删除不存在的书籍
    match inventory.remove_book(99) {
        Ok(_) => {}
        Err(e) => println!("\nError: {}", e),
    }
}
```