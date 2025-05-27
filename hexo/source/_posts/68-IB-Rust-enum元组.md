---
title: IB-Rust-enum元组
date: 2025-05-24 21:02:46
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson10](https://github.com/Zoella-w/IB-Rust/tree/main/10_enum)

## 不同语言 enum 对比

### typescript

``` ts
enum Direction {
    North,
    East,
    South,
    West
}
let dir: Direction = Direction.North;
```

### c++

``` c++
enum Direction {
    North,
    East,
    South,
    West
}
int main() {
    Direction dir = North;
}
```

### rust

``` rust
enum Direction {
    North,
    East,
    South,
    West
}
let dir: Direction = Direction::North;
```

## enum 语法

### field-less enum

只定义类型，但没有定义值

``` rust
enum Fieldless {
    Tuple(),
    Struct {},
    Unit,
}
```

### unit-only enum

全都是 unit 类型

``` rust
enum UnitOnlyEnum {
    Foo = 0,
    Bar = 1,
    Baz = 2,
}
```

## enum 规范

### Pascal Case

enum 名和 enum 值采用 Pascal Case

``` rust
enum PascalCase {
    PascalCase1,
    PascalCase2,
    PascalCase3,
}
```

### snake_case

函数或关联函数采用 snake_case

``` rust
// 方法
impl Pets {
    // 入参是 mut 或 &mut
    fn snake_case1(&self) {
        println!("hi");
    }
}
// 关联函数
impl Pets {
    fn snake_case2(name: String) {
        println!("name is {name}");
    }
}
```

## enum 用法

### match

match 必须能匹配所有条件

``` rust
enum Pets {
    Cat,
    Dog,
}
let cat = Pets::Cat;
let dog = Pets::Dog;
match cat {
    Pets::Cat => {
        println!("is cat");
    }
    Pets::Dog => {
        println!("is dog");
    }
    _ => {}
}
```

### if let

``` rust
if let cat = Pets::Cat {
    println!("is cat");
}
```

## option & result

### option

- 表示一个值可能为 Some（存在）或 None（不存在）
- 替代其他语言中的 null 或 undefined，强制开发者处理可能的空值

``` rust
let num = Some(1);
let none: Option<usize> = None; // Option<T>
match num {
    Some(val) => {}
    None => {}
}
```

### result

- 表示一个操作可能成功（Ok）或失败（Err）
- 替代其他语言中的异常（Exception），强制开发者处理可能的错误

``` rust
fn main() -> Result<(), ()> {
    let num: Result<usize, ()> = Ok(1);
    match num {
        Ok(val) => {}
        Err(_) => {}
    }
    // Err(())
    Ok(())
}
```

## option 和 result 的转换

### option -> result: `ok_or`

当 Option 的 None 需要携带具体的错误信息时（例如在函数中需要返回 Result，但内部逻辑依赖 Option）

``` rust
let opt: Option<i32> = Some(42);
let result: Result<i32, &str> = opt.ok_or("error");
let none: Option<i32> = None;
let result: Result<i32, &str> = none.ok_or("error");
```

### result -> option : `ok()`, `err()`

当错误无关紧要，只需关注操作是否成功时（例如日志记录或快速判断）

``` rust
let res: Result<i32, &str> = Ok(42);
let opt: Option<i32> = res.ok();
let res: Result<i32, &str> = Err("error");
let opt: Option<i32> = res.ok();
```

## 常用 API

### `Option` API

#### `.map()` `.unwrap()`

`map()` 对 Option 中的值应用一个闭包，返回一个新的布尔值 Option。若原 Option 是 None，直接返回 None

`.unwrap()` 取值，如果是 None 则会 panic

``` rust
let opt: Option<i32> = Some(1);
let a1 = opt.map(|num| num > 0); // Some(true)
assert!(a1.unwrap()); // true
```

#### `.and_then()`

`.and_then()` 对 Option 中的值应用一个闭包，闭包返回新的 Option，实现链式调用。若原 Option 是 None，直接返回 None

``` rust
let opt: Option<i32> = Some(1);
let b1 = opt.and_then(|val| Some(val + 1)); // Some(2)
assert_eq!(b1, Some(2));
```

#### `.or_else()`

`.or_else()` 在 Option 为 None 时调用闭包生成一个替代的 Option。若原 Option 是 Some，直接返回原值

``` rust
let opt: Option<i32> = Some(1);
let c1 = opt.or_else(|| Some(2)); // Some(1)；如果不是 Some，返回 Some(2)
assert_eq!(c1, Some(1));
```

### Result API

#### `.map()`

`.map()` 对 Result 中的 Ok 值应用一个闭包，返回新的 Result。若原 Result 是 Err，直接返回原 Err

``` rust
let ret: Result<i32, &str> = Ok(1);
let a2 = ret.map(|val| val > 0); // Ok(true)
assert!(a2.unwrap()); // true
```

#### `.and_then()`

`.and_then()` 对 Result 中的 Ok 值应用一个闭包，闭包返回新的 Result，实现链式调用。若原 Result 是 Err，直接返回原 Err

``` rust
let ret: Result<i32, &str> = Ok(1);
let b2 = ret.and_then(|val| Ok((val + 1))); // Ok(2)
assert_eq!(b2, Ok(2));
```

#### `.or_else()`

`.or_else()` 在 Result 为 Err 时调用闭包生成一个替代的 Result。若原 Result 是 Ok，直接返回原值

``` rust
let ret: Result<i32, &str> = Ok(1);
let c2 = ret.or_else(|str| Err(3)); // Ok(1)；如果不是 Ok，返回 Err(3)
assert_eq!(c2, Ok(1));
```

## enum 内存占用

每个 enum 包含标签（tag）​​和​​数据（data）​​两部分：
- ​​标签​​：用于区分不同变体，大小取决于变体数量
  - 比如：1 byte = 8 bits，能表示 2^8=256 个变体
​- ​数据​​：存储变体中的具体值，大小由最大的变体决定
  - 因为每次当枚举变量被赋值为某个变体时，只有该变体的数据会被写入内存，而旧数据会被销毁

## 课后习题

``` rust
enum MyEnum {
    A(u8, u8), // 2
    B,
    C {},
}
// 标签1，内存2
// 1 + 2 = 3
println!("size of MyEnum: {}", size_of::<MyEnum>());

enum EnumA {
    A = 255,
}
// 当枚举只有一个变体时，Rust 编译器会将其优化为 ​​零大小类型（ZST）​​，即不占用任何内存
// 即使显式指定判别值，判别值仅在 ​​编译时存在​​，不会在运行时存储
// 标签0，内存0
println!("size of EnumA: {}", size_of::<EnumA>());

enum EnumB {
    A = 255,
    B, // 256 -> 2
}
// 当定义枚举时，如果某个变体​​没有显式指定判别值（discriminant）​​
// Rust 会默认将其判别值设为 ​​前一个变体的判别值加 1​​
// 判别值的整数值直接用于标识变体，无需额外标签
// 标签0，内存2
// 0 + 2 = 2
println!("size of EnumB: {}", size_of::<EnumB>());
```
