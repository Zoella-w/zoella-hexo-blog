---
title: IB-Rust-字符串
date: 2025-05-23 14:37:53
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson9](https://github.com/Zoella-w/IB-Rust/tree/main/9_string)

## 字符串的定义

字符串是由字符组成的连续集合

Rust 中的字符是 Unicode 类型，因此每个字符占据 4 字节内存空间，但字符串是 UTF-8 编码，也就是字符串中的字符所占的字节数是变化的（1～4）

比如，对于“hello 中国”来说，utf8 编码为：

```shell
h    e   l   l   o   _   中           国
[104 101 108 108 111 32  228,184,173  229,155,189]
```

### str

str 是 Rust 的一个基础类型，本质是一个字节数组`[u8]`

`str` 或 `[u8]` 类型的值存放在内存：可能是堆，可能是栈，还可能硬编码进可执行程序

#### 字符串字面量

字符串字面量是 `str` 类型，在编译时就知道其内容，其字面值文本被直接硬编码进可执行程序

在存储了该字符串之后，需要通过切片引用 `&str` 来访问它

- `&str`是一种不可变引用，所以它没有所有权
- `str` 类型是硬编码进可执行文件，无法被修改

``` rust
let s: &str = "hello world";
```

### String

String 字符串是在程序运行得过程中动态生成的，其在 rust 中是一个复合数据类型，定义如下：

```rust
pub struct String {
    vec: Vec<u8>,}
```

这说明 `String` 是可改变的，并且拥有所有权

### 其他

除了 `String` 类型的字符串，Rust 的标准库还提供了其他类型的字符串，例如 `OsString`， `OsStr`， `CsString` 和` CsStr` 等

#### &str vs String

`&str`: 这是一个字符串切片，它是固定大小的，并且不能改变。

`String`: 这是一个可增长的、可改变的、拥有所有权的、UTF-8 编码的字符串类型。它通常用于需要改变或者增长字符串的情况。

## 类型转换

### &str 的转换

#### 字节数组 `[u8]`|`Vec<u8>`->`&str`

```rust
use std::str;
fn main() {
    let b = [104, 101, 108, 108, 111, 32, 228, 184, 173, 229, 155, 189];
    // 实现了从 Vec<u8> 到 [u8] 的隐式类型转换
  	// let b = vec!(104, 101, 108, 108, 111, 32, 228, 184, 173, 229, 155, 189);
    let s = str::from_utf8(&b).unwrap();
}
```

#### 字符串字面量->`[u8]`

```rust
let s = "hello 中国";
let b = s.as_bytes();
dbg!("{}", b);
```

#### `String`->`&str`

String 转变为 &str 是无损的（性能无损，不会造成重写 malloc 或者数据移动）

`&String` 可以自动转换为 `&str`，因此在函数中，如果接收参数是 `&String`，通常会采用 `&str` 作为入参，提升数据兼容性

字符串是 UTF-8 编码，因此需要保证索引的字节刚好落在字符的边界，因为有些字符的长度超过 1 个字节

``` rust
let s = String::from("hello, world");
// 方法一、使用 &String 直接转换
some_func(&s);
// 方法二、使用 .as_str() 转换
some_func(s.as_str());
```

### String的转换

#### 字节数组 `Vec<u8>`->`String`

```rust
fn main() {
    let b = vec!(104, 101, 108, 108, 111, 32, 228, 184, 173, 229, 155, 189);

    let s = String::from_utf8(b).unwrap();

    println!("{}", s);
}
```

#### `&str`->`String`

从 &str 获得 String 是低效的，因为要重新 malloc 数据

```rust
let s = String::from("hello 世界");
```

### Overview

```none
&str    -> String  | String::from(s) or s.to_string() or s.to_owned()
&str    -> &[u8]   | s.as_bytes()
&str    -> Vec<u8> | s.as_bytes().to_vec() or s.as_bytes().to_owned()
String  -> &str    | &s if possible* else s.as_str()
String  -> &[u8]   | s.as_bytes()
String  -> Vec<u8> | s.into_bytes()
&[u8]   -> &str    | s.to_vec() or s.to_owned()
&[u8]   -> String  | std::str::from_utf8(s).unwrap(), but unsafe
&[u8]   -> Vec<u8> | String::from_utf8(s).unwrap(), but unsafe
Vec<u8> -> &str    | &s if possible* else s.as_slice()
Vec<u8> -> String  | std::str::from_utf8(&s).unwrap(), but unsafe
Vec<u8> -> &[u8]   | String::from_utf8(s).unwrap(), but unsafe
```

## 操作字符串

可变字符串 `String` 的修改，添加，删除等常用方法，且如果执行这些操作会修改原字符串，则字符串必须是 mut 的

### 追加 (Push)

- 使用 `push()` 方法追加字符 `char`
- 使用 `push_str()` 方法追加字符串字面量
- 两个方法都是**在原有的字符串上追加**，需要用 `mut` 修饰

```rust
let mut s = String::from("Hello ");
s.push_str("rust");
s.push('!');
```

### 插入 (Insert)

- 使用 `insert()` 方法插入单个字符 `char`
- 使用 `insert_str()` 方法插入字符串字面量
- 这俩方法需要传入两个参数，第一个参数是字符（串）插入位置的索引，索引从 0 开始计数；第二个参数是要插入的字符（串），注意不要越界
- 两个方法都是**在原有的字符串上追加**，需要用 `mut` 修饰

```rust
    let mut s = String::from("Hello rust!中文");
    s.insert(5, ',');
    s.insert_str(6, " I like");
```

### 替换 (Replace)

#### `replace`

- `replace()` 方法接收两个参数，第一个参数是要被替换的字符串，第二个参数是新的字符串
- 该方法会替换所有匹配到的字符串
- 该方法返回一个新的字符串，而不是操作原来的字符串，所以不需要 mut 修饰

```rust
let s = String::from("I like rust. Learning rust is my favorite!");
let new_string_replace = s.replace("rust", "RUST");
```

#### `replacen`

方法接收三个参数，前两个参数与 `replace()` 方法一样，第三个参数则表示替换的个数

```rust
let s = "I like rust. Learning rust is my favorite!";
let new_string_replacen = s.replacen("rust", "RUST", 1);
```

#### `replace_range`

- `replace_range` 接收两个参数，第一个参数是要替换字符串的范围（Range），第二个参数是新的字符串
- 该方法是直接操作原来的字符串，需要使用 `mut` 修饰
- 如果range的范围大于/小于新字符串长度会怎样？
- 参数的位置需要在合法的字符边界

```rust
  let mut s = String::from("hello rust 中国");
  s.replace_range(7..8, "R");
```

### 删除 (Delete)

####  `pop` 

- 删除并返回字符串的最后一个字符
- 其返回值是一个 `Option` 类型，如果字符串为空，则返回 `None`
- 该方法直接操作原来的字符串，需要用 `mut` 修饰

```rust
let mut s = String::from("中文!");
let p1 = s.pop(); // 中
let p2 = s.pop(); // 文
let p3 = s.pop(); // ！
let p4 = s.pop(); // `None`
```

####  `remove` 

- 其返回值是删除位置的字符串
- 接收一个参数，表示该字符起始索引位置
- 参数的位置需要在合法的字符边界
- 该方法直接操作原来的字符串，需要用 `mut` 修饰

```rust
fn main() {
    let mut s = String::from("测试remove方法");
    println!(
        "string_remove 占 {} 个字节",
        std::mem::size_of_val(s.as_str())
    );
    // 删除第一个汉字
    s.remove(0);
    // 下面代码会发生错误
    // s_remove.remove(1);
    // 直接删除第二个汉字
    // s_remove.remove(3);
    dbg!(s);
}
```

####  `truncate` 

- 删除字符串中从指定位置开始到结尾的全部字符
- 参数的位置需要在合法的字符边界
- 该方法是直接操作原来的字符串，需要用 `mut` 修饰

```rust
let mut s = String::from("测试truncate");
s.truncate(3);
```

####  `clear` 

- 清空字符串
- 该方法直接操作原来的字符串，需要用 `mut` 修饰

```rust
let mut s = String::from("string clear");
s.clear();
```

### 连接 (Concatenate)

#### 使用 Add()

`add()` 方法的第二个参数必须为字符串切片引用（Slice）类型

```rust
use std::ops::Add;
fn main() {
    let s = String::from("hello ");
    let string_rust = String::from("rust");
    let result = s.add(&string_rust);
}
```

#### 使用 + 或者 += 连接字符串

之所以可以直接 + 是因为 Add 实现了 + 的 trait

```rust
let s = String::from("hello ");
let string_rust = String::from("rust");
let result = s + &string_rust; // &string_rust 会解引用为 &str
let mut result = result + "!";
result += "!!!";
```

##### 关于自动解引用

`std::String` -> `&str` 没有任何成本，反过来则需要重新申请空间

所以在定义使用字符串的函数的时候，优先定义成字符串切片的类型

##### 关于所有权移动

- 无论是 + 还是 add 都发生了所有权移动

```rust
    let s = String::from("hello ");
    let string_rust = String::from("rust");
    let result = s + &string_rust; // &string_rust 会解引用转为 &str
    // println!("{}", s); // 报错
    let mut result = result + "!";
    result += "!!!";
    println!("连接字符串 + -> {}", result);
```

#### 使用 `format!` 连接字符串

- `format!` 的用法与 `println!` 的用法类似

```rust
let s1 = "hello";
let s2 = String::from("rust");
let s = format!("{} {}!", s1, s2);
println!("{}", s);
```

### Rust 手册 String 相关的函数

https://doc.rust-lang.org/std/string/struct.String.html#

## 字符串转义

可以通过转义的方式 `\` 输出 ASCII 和 Unicode 字符。

```rust
// 通过 \ + 字符的十六进制表示，转义输出一个字符
let byte_escape = "I'm writing \x52\x75\x73\x74!";
println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape); // What are you doing? (\x3F means ?) I'm writing Rust!

// \u 可以输出一个 unicode 字符
let unicode_codepoint = "\u{211D}";
let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";
println!(
    "Unicode character {} (U+211D) is called {}",
    unicode_codepoint, character_name
); // Unicode character ℝ (U+211D) is called "DOUBLE-STRUCK CAPITAL R"

// 换行了也会保持之前的字符串格式
// 使用\忽略换行符
let long_string = "String literals
                    can span multiple lines.
                    The linebreak and indentation here ->\
                    <- can be escaped too!";
println!("{}", long_string); // The linebreak and indentation here -><- can be escaped too!
```

### 原样输出

```rust
println!("{}", "hello \\x52\\x75\\x73\\x74");
let raw_str = r"Escapes don't work here: \x3F \u{211D}";
println!("{}", raw_str);
// 如果字符串包含双引号，可以在开头和结尾加 #
let quotes = r#"And then I said: "There is no escape!""#;
println!("{}", quotes);
// 如果还是有歧义，可以继续增加，没有限制
let longer_delimiter = r###"A string with "# in it. And even "##!"###;
println!("{}", longer_delimiter);
```
