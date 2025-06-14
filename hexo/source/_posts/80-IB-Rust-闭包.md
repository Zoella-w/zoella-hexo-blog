---
title: IB-Rust-闭包
date: 2025-06-11 23:46:23
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson22](https://github.com/Zoella-w/IB-Rust/tree/main/22_closure)


## 闭包介绍

- 什么是闭包？
  - 闭包是一个可以捕获所在环境中的变量的匿名函数
  - 闭包通过 `||` 符号定义，可以像普通函数一样调用，但与函数不同，闭包可以访问外部作用域的变量

- 闭包的特点
  - 可以捕获周围作用域的变量
  - 支持作为参数传递给其他函数
  - 可以返回闭包作为函数的返回值
  - 闭包通常通过类型推断来确定参数和返回值的类型

## 闭包定义

```rust
let add_one = |x: i32| -> i32 { x + 1 };
println!("{}", add_one(5));  // 输出：6
```

### 省略类型的闭包

Rust 可以推断闭包的参数和返回值类型，因此在很多情况下可以省略类型声明

```rust
let add_one = |x| x + 1;
println!("{}", add_one(5));  // 输出：6
```

## 闭包的使用

### 作为函数参数

闭包可以作为函数的参数传递，从而实现更灵活的代码结构

```rust
fn apply_to_3<F>(f: F) -> i32
where
    F: Fn(i32) -> i32,
{
    f(3)
}
let double = |x| x * 2;
println!("{}", apply_to_3(double));  // 输出：6
```

### 捕获环境变量

闭包可以捕获并使用其定义所在环境中的变量

```rust
let x = 4;
// // can't capture dynamic environment in a fn item
// fn equal_to_x(z: i32) {
//     z = x;
// }
let equal_to_x = |z| z == x;
let y = 4;
assert!(equal_to_x(y));
```

---

**闭包的三种捕获方式**
- **按值捕获（move 语义）**：将环境变量的所有权移入闭包
- **按引用捕获**：通过引用捕获环境变量
- **按可变引用捕获**：通过可变引用捕获环境变量

```rust
let mut num = 5;
// 默认按引用捕获，获取 num 的引用
let add_num = |x: i32| x + num;
println!("{}", add_num(3));  // 输出：8
// 按可变引用捕获，获取 num 的可变引用
let mut change_num = |x: i32| num += x;
change_num(5);
println!("{}", num);  // 输出：10
```

## 闭包的原理

- **自动实现的函数类型**
  - `Fn`、`FnMut` 和 `FnOnce` 是 Rust 提供的三种函数闭包类型，分别表示按引用捕获、按可变引用捕获和按值捕获

- **闭包的类型推断**
  - Rust 能够根据闭包的使用上下文推断出闭包的具体类型
  - `Fn`、`FnMut` 和 `FnOnce` 是闭包在不同情况下自动实现的 trait

- **生命周期与闭包**
  - 闭包可以捕获引用，但需要保证引用的生命周期超过闭包的生命周期

```rust
let s = String::from("hello");
let closure = || println!("{}", s);
closure();  // 正常运行，因为 s 在closure 之前有效
```

---

创建闭包时，通过闭包对环境值的使用，Rust 能推断出具体使用哪个 trait:
- 所有的闭包都实现了 `FnOnce`
- 没有 move 捕获变量的实现了 `FnMut`
- 无需可变访问捕获变量的闭包实现了 `Fn`

## move 关键字

在参数列表前使用 move 关键字，可以强制闭包取得它所使用用的环境值的所有权。一般用于：将闭包传递给新线程以移动数据使其归新线程所有

```rust
let x = vec![1, 2, 3];
let equal_to_x = move |z| z = x; // drop x
println!("can't use x here: {:?}", x);
let y = vec![1,2,3];
equal_to_x(y); // drop y
println!("can't use y here: {:?}", y);
```

## 课后习题

假设你正在开发一个博客系统，其中每个用户可以查看不同的文章页面。页面的渲染是一个计算密集型的过程，可能涉及数据库查询、模板渲染等操作。因此，为了优化性能，你决定在服务器端实现一个缓存系统。

要求：
- 实现 PageCache 结构体：
  - 该结构体应缓存根据 用户ID 和 文章ID 渲染的页面。
  - 你需要为该结构体实现一个 get_page 方法，该方法接受 用户ID 和 文章ID，并返回渲染后的页面内容。
  - 如果相同的 用户ID 和 文章ID 已经渲染过，则 get_page 应直接返回缓存的页面，而不是重新渲染。

通用性：
  - PageCache 应支持任意类型的 用户ID（例如，u32 或 String）和 文章ID。
  - 缓存的内容应为渲染后的 HTML页面（String 类型）。

示例：
```rust
fn main() {
    let mut page_cache = PageCache::new(|user_id: &str, article_id: u32| -> String {
        println!("Rendering page for user {} and article {}", user_id, article_id);
        format!("Rendered HTML content for user {} and article {}", user_id, article_id)
    });

    // 第一次调用，会执行页面渲染
    println!("{}", page_cache.get_page("user1", 42)); // 输出 "Rendering page for user1 and article 42" 和 "Rendered HTML content for user user1 and article 42"
    
    // 第二次调用，直接返回缓存结果
    println!("{}", page_cache.get_page("user1", 42)); // 仅输出 "Rendered HTML content for user user1 and article 42"，不再渲染

    // 不同用户查看同一文章，会重新渲染
    println!("{}", page_cache.get_page("user2", 42)); // 输出 "Rendering page for user2 and article 42" 和 "Rendered HTML content for user user2 and article 42"
}
```

```rust
use std::collections::HashMap;
use std::hash::Hash;
struct PageCache<K, V> {
    cache: HashMap<(K, V), String>,
}

impl<K, V> PageCache<K, V>
where
    K: Clone + Eq + Hash,
    V: Clone + Eq + Hash,
{
    fn new() -> Self {
        PageCache {
            cache: HashMap::new(),
        }
    }

    fn get_page<F>(&mut self, user_id: K, article_id: V, render: F) -> String
    where
        F: FnOnce(&K, &V) -> String, // FnOnce 确保闭包只能被调用一次
    {
        // 缓存键
        let cache_key = (user_id.clone(), article_id.clone());
        // 检查缓存是否存在
        if let Some(content) = self.cache.get(&cache_key) {
            // 缓存命中
            content.clone()
        } else {
            // 缓存未命中：调用渲染函数生成新内容
            let content = render(&user_id, &article_id);
            // 将新内容存入缓存（使用原始数据避免克隆开销）
            self.cache.insert((user_id, article_id), content.clone());
            content
        }
    }
}

fn main() {
    // 创建空缓存实例
    let mut page_cache = PageCache::new();
    // 第一次调用：执行渲染并缓存结果
    println!(
        "{}",
        page_cache.get_page("user1", 42, |user_id, article_id| {
            println!(
                "Rendering page for user {} and article {}",
                user_id, article_id
            );
            format!(
                "Rendered HTML content for user {} and article {}",
                user_id, article_id
            )
        })
    );
    // 第二次调用：相同用户和文章 - 直接返回缓存
    println!(
        "{}",
        page_cache.get_page("user1", 42, |_, _| {
            // 这个闭包在缓存命中时不会执行
            unreachable!("This should never be called when cache exists");
        })
    );
    // 不同用户：重新执行渲染
    println!(
        "{}",
        page_cache.get_page("user2", 42, |user_id, article_id| {
            println!(
                "Rendering page for user {} and article {}",
                user_id, article_id
            );
            format!(
                "Rendered HTML content for user {} and article {}",
                user_id, article_id
            )
        })
    );
}
```