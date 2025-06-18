---
title: IB-Rust-测试
date: 2025-06-14 17:24:43
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson24.1](https://github.com/Zoella-w/IB-Rust/tree/main/24.1_test_course)

[Codes in lesson24.2](https://github.com/Zoella-w/IB-Rust/tree/main/24.2_test_task)

## Unit Test
### 测试结构
#### 基本单元测试

使用 `#[cfg(test)]` 属性标记测试模块：
```rust
#[cfg(test)]
mod tests {
    // 导入要测试的模块
    use super::*;
    
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    fn test_add() {
        assert_eq!(add(1, 2), 3);
    }
}
```

#### 常用测试宏
##### ​`​assert!​​`

验证布尔表达式为 true
```rust
#[test]
fn test_assert() {
    let result = true;
    assert!(result);
}
```

##### `​​assert_eq!​`​

比较两个值是否相等
```rust
#[test]
fn test_equality() {
    assert_eq!("hello", "hello");
}
```

##### `​​assert_ne!​​`

比较两个值是否不相等
```rust
#[test]
fn test_inequality() {
    assert_ne!("foo", "bar");
}
```

##### `​​should_panic​​`

测试是否如预期般 panic
```rust
#[test]
#[should_panic(expected = "除数为零")]
fn test_divide_by_zero() {
    1 / 0;
}
```

### 测试生命周期
#### 测试函数生命周期

使用 `#[test]` 标记的函数有独立的生命周期：
- 每个测试在自己的线程中运行
- 测试失败只会导致该线程崩溃，不会影响其他测试
- 返回值必须是 () 类型（单元类型）

#### 测试前准备与清理

使用 setup 和 teardown 函数进行测试环境的准备和清理：
```rust
#[cfg(test)]
mod tests {
    struct TestFixture {
        value: i32,
    }
    impl TestFixture {
        fn new() -> Self {
            println!("创建测试环境");
            TestFixture { value: 42 }
        }
    }
    impl Drop for TestFixture {
        fn drop(&mut self) {
            println!("清理测试环境");
        }
    }
    
    #[test]
    fn test_with_fixture() {
        let fixture = TestFixture::new();
        assert_eq!(fixture.value, 42);
    }
}
```

#### 多测试共享环境

使用生命周期更长的结构体：
```rust
#[cfg(test)]
mod tests {
    static mut TEST_VALUE: i32 = 0;
    
    #[test]
    fn test_one() {
        unsafe { TEST_VALUE = 42 };
        assert_eq!(unsafe { TEST_VALUE }, 42);
    }
    #[test]
    fn test_two() {
        assert_eq!(unsafe { TEST_VALUE }, 0);
    }
}
```

### 测试私有函数

Rust 允许在测试中访问私有函数：
```rust
mod my_module {
    pub fn public_func() -> i32 {
        private_func()
    }
    fn private_func() -> i32 {
        42
    }
    
    #[cfg(test)]
    mod tests {
        use super::*;
        #[test]
        fn test_private_func() {
            assert_eq!(private_func(), 42);
        }
    }
}
```

### 高级测试功能
#### 忽略测试

使用 `#[ignore]` 标记暂时不运行的测试：
```rust
#[test]
#[ignore = "功能尚未实现"]
fn expensive_test() {
    // 耗时的测试
}
```

#### 测试组合使用

组合多个属性：
```rust
#[test]
#[ignore]
#[should_panic(expected = "预期错误")]
fn complex_test() {
    // ...
}
```

#### 数据驱动测试

使用循环或宏创建多组测试数据：
```rust
#[test]
fn test_addition() {
    let test_cases = vec![
        (1, 1, 2),
        (2, 3, 5),
        (-1, 1, 0),
        (0, 0, 0),
    ];
    for (a, b, expected) in test_cases {
        assert_eq!(a + b, expected);
    }
}
```

### 测试运行与管理
#### 运行测试

```bash
# 运行所有测试
cargo test
# 运行特定测试（名称匹配）
cargo test test_add
# 运行被忽略的测试
cargo test -- --ignored
# 在测试通过时显示输出
cargo test -- --nocapture
# 单线程运行测试（避免状态共享问题）
cargo test -- --test-threads=1
```

#### 测试覆盖率

使用 tarpaulin 或 cargo llvm-cov 获取测试覆盖率：

```bash
# 安装 tarpaulin
cargo install cargo-tarpaulin
# 运行测试覆盖率
cargo tarpaulin --ignore-tests
```

## Integration Test

Rust 集成测试 用于测试库的公共 API，验证多个模块协同工作的情况。与单元测试关注内部实现不同，集成测试从外部用户的角度验证程序功能

### 集成测试核心概念
#### 位置与约定

默认放在项目根目录的 tests 目录中，每个测试文件都是独立的 crate，Cargo自动发现并运行这些测试

```text
project/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    ├── integration_test1.rs
    ├── integration_test2.rs
    └── common/
        └── mod.rs  # 共享辅助代码
```

注意：使用 common/mod.rs 而不是 common.rs

#### 基本特点

- 仅能访问公共API
- 测试完整工作流程
-  模拟外部依赖
- 可测试多个模块组合行为

### 创建集成测试
#### 基本结构

```rust
// tests/basic_integration.rs
use my_library;  // 导入被测试的库
#[test]
fn test_add() {
    assert_eq!(my_library::add(2, 3), 5);
}
```

#### 运行集成测试

```bash
cargo test --test integration_test_name  # 运行特定测试文件
cargo test                               # 运行所有测试（包括单元测试）
```

### 高级集成测试技巧
#### 共享辅助代码

在 tests/common/mod.rs 中创建共享功能：
```rust
// tests/common/mod.rs
pub fn setup() {
    println!("测试环境准备");
    // 数据库连接、网络模拟等
}
pub fn teardown() {
    println!("清理测试环境");
}
```

在测试文件中使用：
```rust
// tests/advanced_integration.rs
mod common;
use common::{setup, teardown};
use my_library;
#[test]
fn test_complex_operation() {
    setup(); // set up
    let result = my_library::complex_operation();
    assert!(result.is_ok());
    teardown(); // tear down
}
```

#### 测试模块组织

```rust
// tests/api/mod.rs - 核心测试模块
pub mod v1;
pub mod v2;
// tests/api/v1.rs - API版本1测试
#[test]
fn test_api_v1() { /* ... */ }
```

### 集成测试报告

```bash
# 生成 HTML 报告
cargo test --test integration_tests -- --format json | cargo2html -o report.html
# 使用 cargo-tarpaulin 获取覆盖率
cargo tarpaulin --integration-tests --out Html
```

## Doc Tests

文档测试通过允许将测试代码直接嵌入到文档注释中，来确保文档示例始终与代码行为保持一致，该机制极大增强了 Rust 生态系统的文档质量

### 基本概念
#### 什么是文档测试？

Rust 支持将可执行代码示例直接嵌入到文档注释中。当运行 `cargo test` 时，这些代码块会被提取、编译并作为测试运行，目标是确保文档中的示例始终正确有效

#### 核心价值

- ​​文档与代码同步​​：示例代码随功能变化自动更新
​- ​可信文档​​：用户可信任通过测试的文档示例
​- ​开发者便利​​：编写文档的同时就创建了测试用例

### 基础语法
#### 基本文档测试

```rust
/// 计算两数之和
///
/// # 示例
///
/// ```
/// let sum = my_crate::add(2, 3);
/// assert_eq!(sum, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

#### 模块级文档测试

```rust
//! # My Crate
//!
//! 提供数学运算功能
//!
//! ## 示例
//!
//! ```
//! use my_crate;
//!
//! let result = my_crate::multiply(3, 4);
//! assert_eq!(result, 12);
//! ```
pub fn multiply(a: i32, b: i32) -> i32 {
    a * b
}
```

### 文档测试工作机制
#### 编译与执行流程

1. ​​文档解析​​：rustdoc 提取文档中的代码块
​2. ​测试代码生成​​：
   - 自动添加 `extern crate` 语句
   - 将代码包装到 `fn main() { ... }` 中
   - 添加必要的 `use` 语句
​3. ​独立编译​​：每个文档测试作为独立 crate 编译
​​4. 测试执行​​：编译后作为普通测试运行

#### 生成的测试代码（示例）

原始文档注释：
```rust
let result = add(2, 3);
assert_eq!(result, 5);
```

实际执行的测试代码：
```rust
extern crate my_crate;
use my_crate::add;
fn main() {
    let result = add(2, 3);
    assert_eq!(result, 5);
}
#[test]
fn test_name() {
    main();
}
```

### 高级使用技巧
#### 隐藏辅助代码

使用 # 隐藏辅助代码（在文档中不显示，但在测试中包含）：
```rust
/// ```
/// # // 这行在文档中隐藏
/// # fn setup() -> String { String::from("world") }
/// let s = setup();
/// assert_eq!(s, "world");
/// ```
```

文档展示效果：
```rust
let s = setup();
assert_eq!(s, "world");
```

#### 测试错误处理

使用 `should_panic` 属性测试预期 panic：
```rust
/// ```should_panic
/// # struct NonPositive;
/// my_crate::divide(10, 0); // 应当 panic
/// ```
pub fn divide(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("除零错误");
    }
    a / b
}
```

#### 编译期测试

使用 `compile_fail` 确保代码无法编译：
```rust
/// ```compile_fail
/// // 类型不匹配示例
/// let s: String = 42; // 无法将整数转换为字符串
/// ```
```

#### 定制测试属性

通过内联属性控制测试行为：
```rust
/// ```no_run
/// // 这段代码会编译但不会运行
/// let result = fetch_data_from_internet();
/// ```
/// ```ignore
/// // 完全忽略此测试代码
/// not_implemented_yet();
/// ```
```

#### 使用返回值测试

直接在文档注释中返回值：
```rust
/// ```
/// # use my_crate::sqrt;
/// assert_eq!(sqrt(9.0)?, 3.0); // 使用 ? 返回错误
/// # Ok::<(), &'static str>(())
/// ```
```

### 文档测试属性控制
#### Crate 级属性配置

在 lib.rs 或 main.rs 顶部全局配置：

```rust
#![doc(test(attr(
    // 为所有文档测试启用 unstable 特性
    allow(unused_imports),
    feature(staged_api)
)))]
#![doc(test(attr(deny(warnings))))] // 将所有警告视为错误
```

### 文档测试最佳实践

- 测试文档覆盖原则
  - 每个公共 API 都应该有文档测试
  - 重点关注边界情况和错误处理
  - 避免过度测试简单 getter/setter
- 性能优化策略
  - 保持示例简洁（不超过 10-15 行）
  - 复杂逻辑拆分为多个测试

## Benches

Rust 的基准测试 (Benchmarks) 是用于评估代码性能的核心工具。Rust 提供了强大的基准测试框架，帮助开发者对代码性能进行定量分析和优化

### 基准测试概览

#### 基准测试的目的

- ​​性能度量​​：测量代码执行时间
​- ​性能对比​​：比较不同实现的速度差异
​- ​回归检测​​：防止性能退化
​​- 优化依据​​：确定优化点并量化效果

#### 基准测试的特点

- ​​精确测量​​：使用高精度计时器
​- ​统计分析​​：计算平均值、离群值等
​​- 内存分析​​：可集成内存分配器
​​- 编译器优化控制​​：通过 `black_box` 避免过度优化

### 原生基准测试框架（test crate）

Rust 标准库提供了内置基准测试支持，但需要 nightly 版本：
```rust
#![feature(test)]
extern crate test;
use test::Bencher;

#[bench]
fn bench_add(b: &mut Bencher) {
    b.iter(|| {
        (0..1000).sum::<u64>()
    });
}
```

#### 关键组件

- ​​Bencher​​：基准测试控制器
​- ​iter 方法​​：包含要测量的代码
​​- black_box​​：防止编译器优化

#### 编译与运行

```bash
rustup run nightly cargo bench
```

输出示例：
```bash
test bench_add ... bench:       1,234 ns/iter (+/- 123)
```

## Examples

Rust 中的示例测试是位于项目 examples/ 目录下的可执行文件，它们作为项目的高级文档，展示如何使用库的实际应用场景。与文档测试中的小代码片段不同，示例测试提供了更完整、更真实的用法演示

### 项目结构与约定
#### 标准目录结构

```text
my_crate/
├── Cargo.toml
├── src/
│   └── lib.rs
└── examples/
    ├── basic_usage.rs
    ├── advanced_usage.rs
    └── demo_setup/
        ├── main.rs
        └── helper.rs
```

#### 命名约定

- 文件: examples/some_example.rs
- 二进制名称: cargo run --example some_example

### 创建基础示例

```rust
// examples/basic_demo.rs
use my_crate::{add, multiply};
fn main() {
    // 展示基本功能
    println!("2 + 3 = {}", add(2, 3));
    println!("2 × 3 = {}", multiply(2, 3));
    // 包含错误处理
    match my_crate::divide(10, 2) {
        Ok(result) => println!("10 / 2 = {}", result),
        Err(e) => println!("Error: {}", e),
    }
}
```

### 测试与验证指令
#### 编译验证

```bash
cargo test --examples  # 尝试编译所有示例
cargo build --examples  # 构建所有示例
```

#### 运行测试

```bash
cargo run --example basic_demo
cargo run --example network_client
```

### 多文件示例

使用目录包含多个源文件：

```text
examples/
└── complex_demo/
    ├── main.rs
    ├── network.rs
    └── processors.rs
```

在 Cargo.toml 中配置：

```toml
[[example]]
name = "complex_demo"
path = "examples/complex_demo/main.rs"
```

## 目录结构规范

```text
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/  # 基准测试
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/  # 示例
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/  # 集成测试
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

## 课后习题

参考如下模版，创建并发布一个 crate 到 crates.io

**关注点：**
1. 单元测试
2. 继承测试
3. doc 测试
4. examples
5. 项目规范

!()[https://github.com/ibuidl/template-lib-crate]

### 进阶

修改 ci.yaml，通过 github actions 自动更新 crate
