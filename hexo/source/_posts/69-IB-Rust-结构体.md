---
title: IB-Rust-结构体
date: 2025-05-26 23:22:16
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson11](https://github.com/Zoella-w/IB-Rust/tree/main/11_struct)


## 普通结构体

### 定义结构体

- 结构体的定义的位置没有要求，实例化的作用域在定义的范围内即可
- 大括号中，定义每一部分数据的名字和类型，称为字段（field）

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

### 创建结构体实例

初始化实例时，每个字段都需要进行初始化，没有初始值

```rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

#### 使用字段初始化简写语法

变量名与字段名完全相同（类似 js）

```rust
let email = String::from("someone@example.com");
let username = String::from("someusername123");

let user1 = User {
    email,
    username,
    active: true,
    sign_in_count: 1,
};
```

#### 从其他实例创建实例

- 结构体更新语法允许从一个实例中创建一个新实例，同时保留部分字段值
- 最后没有逗号

```rust
let user2 = User {
    email: String::from("another@example.com"),
    active: user1.active,
    username: user1.username,
    sign_in_count: user1.sign_in_count,
};
```

```rust
let user2 = User {
    email: String::from("another@example.com"),
    ..user1 // 简写
};
```

### 修改结构体字段（可变性）

整个实例必须是可变的，不允许只将某个字段标记为可变

```rust
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
user1.email = String::from("anotheremail@example.com");
```

## 特殊的结构体

- 元组结构体（Tuple Struct）
- 单元结构体（Unit-like Struct）

### 元组结构体

字段没有名称的结构体，这种结构体长得像元组，因此被称为元组结构体

例如 `Point` 元组结构体，是 `(x, y, z)` 形式的坐标点

- 使用 `struct` 关键字，接着是圆括号，然后是各字段类型
- 实例化时使用圆括号

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
let black = Color(0, 0, 0); // 访问
let origin = Point(0, 0, 0);
```

### 单元结构体

如果定义一个类型，但是不关心该类型的内容, 只关心其行为时，就可以使用 `单元结构体`

- 使用 `struct` 关键字，接着是名称
- 实例化不需要花括号或圆括号

```rust
struct AlwaysEqual;
let subject = AlwaysEqual;
// 不关心 AlwaysEqual 的字段数据，只关心其行为
// 因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual { }
```

## 所有权

### 实现 Copy 特征的类型

实现了 Copy 特征的类型无需所有权转移，可以直接在赋值时进行数据拷贝

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    let active = user1.active;
    println!("{}", user1.active); // true
    print_username(user1); // someusername123
}
fn print_username(user: User) {
    println!("{}", user.username);
}
```

### 没有实现 Copy 特征的类型

- 字段所有权发生了移动，但其他字段不受影响
- 结构体整体也无法再被使用
- 结构体更新语法同样适用

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    let name = user1.username;
    println!("{}", user1.email); // someone@example.com
    println!("{}", user1.username); // error
    print_username(user1); // error 
}
fn print_username(user: User) {
    println!("{}", user.username);
}
```

### 结构体更新语法

```rust
fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
    // let user2 = User {
    //     email: String::from("another@example.com"),

    //     active: user1.active,
    //     username: user1.username,
    //     sign_in_count: user1.sign_in_count,
    // };
    println!("{}",user1.active);
    println!("{}",user1.username); // 报错
    print_username(user1); // 报错
}
fn print_username(user: User) {
    println!("{}", user.username);
}
```

### 结构体中的借用

可以让 `User` 结构体从其它对象借用数据，不过需要引入生命周期

生命周期能确保结构体的作用范围要比它所借用的数据的作用范围要小

```rust
struct User<'a> {
    username: &'a str,
    email: &'a str,
    sign_in_count: u64,
    active: bool,
}
let user1 = User {
    email: "someone@example.com",
    username: "someusername123",
    active: true,
    sign_in_count: 1,
};
```

## 方法

### 定义方法

方法与函数不同，方法的第一个参数是 `self`，代表调用该方法的结构体实例

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
}
```

一般使用 `&self` 替代 
-  `self: &Self`
- `rectangle: &Rectangle`

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    // fn area(self: &Self) -> u32 {
    //     self.width * self.height
    // }
    // fn area(self: &Rectangle) -> u32 {
    //     self.width * self.height
    // }
}

```

---

`self` 依然有所有权的概念：

- `self` 表示 `Rectangle` 的所有权转移到该方法中，这种形式较少
- `&self` 表示对 `Rectangle` 的不可变借用
- `&mut self` 表示可变借用

#### 方法名跟结构体字段名相同

在外部的包里，用户只能通过方法获取字段，而不能直接访问，作用类似 getter

```rust
pub struct Rectangle {
    width: u32,
    height: u32,
}
impl Rectangle {
    // getter
    pub fn width(&self) -> bool {
        self.width > 0
    }
}
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```

### 关联函数

在 `impl` 块中定义的函数被称为**关联函数**（associated functions），因为它们与 `impl` 后面命名的类型相关

在 `String` 类型上定义的 `String::from()` 也是这样的函数

- 关联函数没有 `self`
- 不能用 `.` 的方式来调用，需要用 `::` 来调用，例如 `let sq = Rectangle::new(3, 3);`
- 一般使用 `new` 来作为构造器的名称

```rust
pub struct Rectangle {
    width: u32,
    height: u32,
}
impl Rectangle {
    fn new(width: u32, height: u32) -> Rectangle {
        Rectangle { width, height }
    }
}
fn main() {
    let rect1 = Rectangle::new(30, 50);
}
```

### 多个 impl 定义

允许为一个结构体定义多个 `impl` 块，目的是提供更多的灵活性和代码组织性

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

## 实现trait

### 为类型实现特征

如果不同的类型具有相同的行为，可以定义一个特征，然后为这些类型实现该特征

**定义特征**是把一些方法组合在一起，目的是定义一个实现某些目标所必需的行为的集合

```rust
trait Shape {
    fn area(&self) -> f64;
}
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
impl Shape for Rectangle {
    fn area(&self) -> f64 {
        (self.width * self.height) as f64
    }
}
fn print_area(shape: &impl Shape) {
    println!("{}", shape.area());
}
```

## 打印结构体的信息

### ❌ println!("{}", r) 

结构体默认没有实现 `Display` 特征

> 结构体为什么不默认实现 `Display` 特征呢？
> 
> 原因在于结构体较为复杂，例如：想要逗号对字段进行分割吗？需要括号吗？等等
> 因此如果要用 `{}` 的方式打印结构体，那就自己实现 `Display` 特征

### ❌ println!("{:?}", r) 

结构体默认没有实现 `Debug` 特征

### ✅  #[derive(Debug)]

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    println!("rect1 is {:?}", rect1);  // rect1 is Rectangle { width: 30, height: 50 }
}
```

#### println!("{:#?}", r) 

```rust
println!("rect1 is {:#?}", rect1);
// rect1 is Rectangle {
//     width: 30,
//     height: 50,
// }
```

#### dbg!(&r)

代码所在的文件名、行号、表达式以及表达式的值

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };
    dbg!(&rect1);
    // [src/rectangle/main.rs:97:16] 30 * scale = 60
    // [src/rectangle/main.rs:100:5] &rect1 = Rectangle {
    //     width: 60,
    //     height: 50,
    // }
}
```

### ✅ 自己实现 `Display` 特征

```rust
impl std::fmt::Display for Rectangle {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        for _ in 0..self.height {
            let mut s = String::new();
            for _ in 0..self.width {
                s.push('#');
            }
            write!(f, "{}\n", s);
        }
        return Ok(());
    }
}
```

## 课后作业

### ✅ 自己实现 `Debug` 特征

```rust
impl std::fmt::Debug for Rectangle {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        for _ in 0..self.height {
            let mut s = String::new();
            for _ in 0..self.width {
                s.push('#');
            }
            write!(f, "{}\n", s);
        }
        return Ok(());
    }
}
```