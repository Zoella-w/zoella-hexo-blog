---
title: IB-Rust-生命周期
date: 2025-06-05 23:40:26
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson18](https://github.com/Zoella-w/IB-Rust/tree/main/18_lifetime)


## 创建生命周期

⽣命周期主要通过⽣命周期注解来创建和使⽤。⽣命周期注解是⼀种显式声明引⽤有效时间的⽅式，通常⽤'a、'b 这样的符号表示

## 生命周期类别

### fn

'a ⽣命周期注解 表明返回值的⽣命周期与输⼊参数的⽣命周期相同：

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

### struct

```rust
struct Example<'a> {
    part: &'a str,
}
```

### enum

```rust
enum StringOption<'a> {
    Some(&'a str),
    None,
}
```

## 生命周期消除（Lifetime Elision）

> rust 编译期自动推理，无需手动重复添加

- 每个引⽤参数都有⾃⼰的⽣命周期参数
- 如果只有⼀个输⼊引⽤参数，那么它的⽣命周期会被赋予所有输出引⽤
- 如果有多个输⼊⽣命周期参数，但其中⼀个是 `&self` 或 `&mut self`，那么 `self` 的⽣命周期会被赋予所有输出引⽤

```rust
fn _first_word(s: &str) -> &str {
    &s[..1]
}
// 返回值不包含引⽤，⽆需标注
fn _add(a: &i32, b: &i32) -> i32 {
    *a + *b
}
// 只包含⼀个 输入引⽤参数，输出引用的⽣命周期与其⼀致
fn _identity(a: &i32) -> &i32 {
    a
}
```

## 特殊生命周期标注

`'static` ⽣命周期表示整个程序运⾏期间都有效的⽣命周期，通常⽤于**全局变量**或**字符串字⾯量**

```rust
 let s: &'static str = "hello";

const SOME_COORDINATE: (i32, i32) = (7, 4);
let static_reference: &'static (i32, i32) = &SOME_COORDINATE;

struct Counter<'a> {
    counter: &'a mut i32,
}
// 方法中没有需要显示指定的生命周期，可以用占位符默认匹配
impl Counter<'_> {
    fn increment(&mut self) {
        *self.counter += 1;
    }
}
```

## 生命周期约束

生命周期注解可以用来约束多个引用之间的关系

'b: 'a 表示⽣命周期'b 必须不短于'a：

```rust
fn example<'a, 'b>(x: &'a str, y: &'b str) -> &'a str
where
    'b: 'a,
{
    x
}
```

## ⽣命周期⼦类型和协变

⽣命周期可以有⼦类型关系，较短的⽣命周期可以被视为较⻓⽣命周期的⼦类型。这在协变（convariance）中尤为重要

```rust
fn _example1<'a, 'b>(x: &'a str, y: &'b str) -> &'a str
where
    'a: 'b,
{
    x
}
```

## 课后习题

修改如下代码，使得编译通过，并解释为什么？

进阶： 还有⼏种解法可以实现，你能通过修改⼀个⽣命周期参数实现吗？

```rust
fn test_lifetime_multiple() {
    fn insert_value<'a, 'b>(my_vec: &'a mut Vec<&'a i32>, value: &'b i32) {
        my_vec.push(value)
    }
    let mut my_vec: Vec<&i32> = vec![];
    let val1 = 1;
    let val2 = 2;
    let a = &mut my_vec;
    insert_value(a, &val1);
    println!("a is {:?} ", a);
    let b = &mut my_vec;
    insert_value(b, &val2);
    println!("b is {:?}", b);
    println!("{my_vec:?}");
}
```

### 解决方法：


`&'a mut Vec<&'a i32>` 是错误的，`my_vec` 和 `其中存储的元素引用` 的生命周期不能相同，因为有如下冲突：
- 第一次调用后，向量要求所有元素具有生命周期'α1
- 但第二次调用要求元素具有生命周期'α2
- 'α1和'α2是不同的具体生命周期实例
- 向量不能存储具有不同生命周期的引用

```rust
let a = &mut my_vec;  // [1] 借用开始，设为生命周期'α1
insert_value(a, &val1); // [2] 要求val1生命周期 ≥ 'α1
                        // 并强制存储元素的生命周期为'α1
println!("a is {:?} ", a); // [3] a最后一次使用

let b = &mut my_vec;  // [4] 新借用开始，设为生命周期'α2
insert_value(b, &val2); // [5] 要求存储元素生命周期为'α2
```

#### 正确代码：

```rust
fn _test_lifetime_multiple() {
    fn insert_value<'a>(my_vec: &mut Vec<&'a i32>, value: &'a i32) {
        my_vec.push(value)
    }
    let mut my_vec: Vec<&i32> = vec![];
    let val1 = 1;
    let val2 = 2;
    let a = &mut my_vec;
    insert_value(a, &val1);
    println!("a is {:?} ", a);
    let b = &mut my_vec;
    insert_value(b, &val2);
    println!("b is {:?}", b);
    println!("{my_vec:?}");
}
```
