---
title: IB-Rust-trait特性
date: 2025-06-10 22:00:54
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson20](https://github.com/Zoella-w/IB-Rust/tree/main/20_trait)

## trait 定义

trait 定义了某个特定类型拥有可能与其他类型共享的功能

>  *trait* 类似于其他语言中 **接口**（*interfaces*）的功能

## trait 实现

### 普通实现

- 使用 `trait` 关键字声明一个特征
- 在大括号中定义该特征的所有方法
- 只定义特征方法的签名，而不进行实现，此时方法签名结尾是 `;`，而不是 `{}`

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

#### 为类型实现特征

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct Post {
    pub title: String, // 标题
    pub author: String, // 作者
    pub content: String, // 内容
}
impl Summary for Post {
    fn summarize(&self) -> String {
        format!("文章{}, 作者是{}", self.title, self.author)
    }
}

pub struct Weibo {
    pub username: String,
    pub content: String
}
impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

#### 特征定义与实现的位置（孤儿规则）

如果你想要为类型 `A` 实现特征 `T`，那么 `A` 或者 `T` 至少有一个是在当前作用域中定义的

- 可以为上面的 `Post` 类型实现标准库中的 `Display` 特征，这是因为 `Post` 类型定义在当前的作用域中
- 可以在当前包中为 `String` 类型实现 `Summary` 特征，因为 `Summary` 定义在当前作用域中
- 无法在当前作用域中，为 `String` 类型实现 `Display` 特征，因为它们俩都定义在标准库中，而不是当前的作用域

### 默认实现

#### 1、其它类型无需再实现该方法，也可以选择重载该方法

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

Post 选择了默认实现，而 Weibo 重载了该方法

```rust
impl Summary for Post {}

impl Summary for Weibo {
    fn summarize(&self) -> String {
        format!("{}发表了微博{}", self.username, self.content)
    }
}
```

#### 2、默认实现 允许调用 相同特征中的其他方法，哪怕这些方法没有默认实现

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

impl Summary for Weibo {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
println!("1 new weibo: {}", weibo.summarize());
```

### 带泛型的 trait

- 可以对同一个目标类型，多次 impl 此 trait，每次提供不同的泛型参数
- 在具体方法调用的时候，必须通过**类型标注**明确使用的是哪一个具体的实现

```rust
// 将元组结构体中的数据，转换为类型 T
trait Converter<T> {
    fn convert(&self) -> T;
}

struct MyInt(i32);

impl Converter<String> for MyInt {
    fn convert(&self) -> String {
        // self.0 访问元组结构体的第 0 个元素
        self.0.to_string()
    }
}
impl Converter<f32> for MyInt {
    fn convert(&self) -> f32 {
        self.0 as f32
    }
}

fn main() {
    let my_int = MyInt(42);
    let output: String = my_int.convert();
    println!("output is: {}", output);
    let output: f32 = my_int.convert();
    println!("output is: {}", output);
}
```

### 关联类型

关联类型是 trait 定义中的类型占位符，定义时不指定其具体的类型，在实现（impl）该 trait 时，才为这个关联类型赋予确定的类型

**关联类型方式只允许对目标类型实现一次**

```rust
trait Converter {
    type Output;
    fn convert(&self) -> Self::Output;
}
impl Converter for MyInt {
    type Output = String;
    fn convert(&self) -> Self::Output {
        self.0.to_string()
    }
}
```

### 默认泛型类型参数

#### 默认泛型的例子

```rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
// 相当于 fn add<T: std::ops::Add<T, Output = T>>(a: T, b: T) -> T {
    a + b
}
println!("add i8: {}", add(2i8, 3i8));
println!("add f64: {}", add(1.23, 1.23));
```

#### 默认泛型类型参数

加法之所以设置默认参数​，​是因为相同类型相加是最常见的情况

```rust
trait Add<Rhs=Self> {
    type Output;
    fn add(self, rhs: Rhs) -> Self::Output;
}
```

`<Rhs=Self>` 定义了加法操作的​​右操作数类型​​：
- `Rhs` 是 "Right-hand side"（右操作数）的缩写
- `=Self` 表示默认情况下右操作数与左操作数相同
- 这是一个​​默认泛型参数​​语法，允许在实现时不显式指定类型

`type Output;` 定义加法操作的​​结果类型​​，具体类型在实现时确定

`fn add(self, rhs: Rhs) -> Self::Output;`
- 接受两个参数：左操作数（self）和右操作数（rhs）
- 返回 Output 关联类型定义的类型
使用 move 语义转移所有权

#### 使用默认泛型类型参数

```rust
use std::ops::Add;
struct Point {
    x: i32,
    y: i32,
}
impl Add for Point {
    type Output = Point;
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
```

#### 自定义 Rhs 类型而不使用默认类型

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;
    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

## 使用特征作为函数参数

### impl Trait 语法

可以使用任何实现了 `Summary` 特征的类型作为该函数的参数

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

除了单个约束条件，还可以通过 `+` 语法指定多个约束条件

```rust
pub fn notify(item: &(impl Summary + Display)) {}
```

### Trait Bound 语法

- `impl Trait` 适用于短小的例子，它是 trait bound 语法的语法糖
- 更长的 trait bound 则适用于更复杂的场景
- 除了单个约束条件，我们还可以通过 `+` 语法指定多个约束条件

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

pub fn notify<T: Summary + Display>(item: &T) {}
```

#### Trait Bound 能做到而 impl Trait 做不到的能力

```rust
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

这适用于 `item1` 和 `item2` 允许是不同类型的情况（只要都实现了 `Summary`）。如果希望它们都是相同类型，就只能使用 trait bound 才能实现：

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

### 通过 `where` 简化 trait bound

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}
```

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### 使用特征约束有条件地实现方法或特征
#### 实现方法

可以有条件地只为那些实现了特定 trait 的类型实现方法

只有那些为 `T` 类型实现了 `PartialOrd` trait（允许比较）和 `Display` trait（允许打印）的 `Pair<T>` 才会实现 `cmp_display` 方法

```rust
use std::fmt::Display;
struct Pair<T> {
    x: T,
    y: T,
}
// impl Pair<i32> {
//     fn new(x: i32, y: i32) -> Self {
//         Self { x, y }
//     }
// }
impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

## 返回实现了 trait 的类型
###  `impl Trait` 语法

在返回值中使用 `impl Trait` 语法，来返回实现了某个 trait 的类型

```rust
fn returns_summarizable() -> impl Summary {
    Weibo {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
    }
}
```

### `dyn trait` 语法

返回实现了某个 trait（动态 trait） 的类型

#### 动态 trait 对象

指定某种指针例如 `&` 引用或 `Box<T>` 智能指针，还有 `dyn` keyword，以及指定相关的 trait 来创建 trait 对象

```rust
fn returns_summarizable(switch: bool) -> Box<dyn Summary>  {
    if switch {
        Box::new(Post {
            title: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        })
    } else {
        Box::new(Weibo {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
        })
    }
}
```

## 通过 `derive` 派生特征

形如 `#[derive(Debug)]` 的代码是一种特征派生语法，被 `derive` 标记的对象会自动实现对应的默认特征代码，继承相应的功能

- `Debug` 特征有一套自动实现的默认代码，当给一个结构体标记后，就可以使用 `println!("{:?}", s)` 的形式打印该结构体的对象
- `Copy` 特征也有一套自动实现的默认代码，当标记到一个类型上时，可以让这个类型自动实现 `Copy` 特征，进而可以调用 `copy` 方法，进行自我复制

```rust
#[derive(Debug, Clone)]
struct Point {
    x: i32,
    y: i32,
}
fn main() {
    let p1 = { x: 0, y: 1};
    let p2 = p1;
    println!("p1: {:?}", p1); // 可以打印，因为实现了 Copy 特征
}
```

### 用于程序员输出的 `Debug`

`Debug` trait 用于开启格式化字符串中的调试格式，允许以调试目的来打印一个类型的实例，可以在程序执行的特定时间点观察其实例

例如，在使用 `assert_eq!` 宏时，`Debug` trait 是必须的。如果等式断言失败，这个宏就把给定实例的值作为参数打印出来，就能看到两个实例为什么不相等

### 默认值的 `Default`

`Default` trait 会创建一个类型的默认值，`Default` 派生的实现调用了类型每部分的 `default` 函数，这意味着类型中所有的字段或值必须实现了 `Default` 才能派生 `Default` 

```rust
#[derive(Default)]
pub struct Post {
    pub title: String,   // 标题
    pub author: String,  // 作者
    pub content: String, // 内容
}
let post1 = Post::default();
```

在 `Option<T>` 实例上使用 `unwrap_or_default` 方法时，`Default` trait 是必须的。如果 `Option<T>` 是 `None`，`unwrap_or_default` 将返回存储在 `Option<T>` 中 `T` 类型的 `Default::default` 的结果

```rust
let post2 = None.unwrap_or_default();
```

## 练习题

#### trait 的定义与实现

```rust
use std::fmt::Display;

// 不要需改 Item 的定义
trait Item<T = String> {
    type Output: Display;
    fn summarize(&self) -> Self::Output;
}

// 不要需改 Apple 结构的定义
struct Apple {
    name: String,
}

impl Item for Apple {
    type Output = String; // 增加
    fn summarize(&self) -> String {
        self.name.to_string()
    }
}

// 不要需改 weibo 结构的定义
struct Weibo {
    author: String,
    content: String,
}

impl Item for Weibo {
    type Output = String; // 增加
    fn summarize(&self) -> String {
        format!("@{}:{}", self.author, self.content)
    }
}

pub struct Container {
    // items: Vec<Item>,
    items: Vec<Box<dyn Item<Output = String>>>, // 修改
}

impl Container {
    pub fn iterator(&self) {
        // for item in self.items {
        for item in &self.items { // 修改
            println!("{}", item.summarize());
        }
    }
}

fn main() {
    let apple = Apple {
        name: "Apple".to_string(),
    };
    let w = Weibo {
        author: "weibo".to_string(),
        content: "hello".to_string(),
    };
    let container = Container {
        // items: vec![apple, w],
        items: vec![Box::new(apple), Box::new(w)], // 修改
    };
    container.iterator();
}
```
