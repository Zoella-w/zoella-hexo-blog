---
title: IB-Rust-模块化
date: 2025-06-03 17:05:46
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson15](https://github.com/Zoella-w/IB-Rust/tree/main/15_modulization)

## Package & Crate

### 定义和作用

- ​​package 是包含一个或多个 crate 的集合​​：由 Cargo.toml 文件定义
- ​​项目组织单位​​：对应于文件系统中的一个目录
​​- 依赖管理​​：通过 Cargo.toml 管理外部 crate 依赖
​​- 构建管理​​：Cargo 根据包定义构建所有 crate

### 关键规则

- ​​一个 package 最多包含一个 library​​ crate
- ​​一个 package 可以包含任意数量的 binary crate
​- ​至少包含一个 crate​​（library crate or binary crate）

``` none
my_package/
├── Cargo.toml       # 包定义文件
└── src/
    ├── lib.rs       # library crate（可选）
    ├── main.rs      # 默认 binary crate
    └── bin/         # 额外 binary crate
        ├── tool1.rs
        └── tool2.rs
```

Cargo.toml 额外 binary crate

``` toml
[[bin]]
name = "tool1"
path = "src/bin/tool1.rs"
```

## Visibility

### `private` (default)

默认私有：

``` rust
struct PrivateStruct;
fn private_function() {}
```

### `pub(crate)`

- 在当前 crate 内的任何位置可见
- 对外部 crate 不可见
- 适合内部共享的实用工具（utils）

``` rust
pub(crate) struct CrateVisible; // 当前 crate 内可见
```

### `pub(in path)`

- 限定在指定的模块路径内可见
- 比 pub(crate) 更精确的可见性控制
- 路径必须是当前crate内的模块路径

``` rust
pub(in crate::module) struct ModuleVisible; // 特定模块路径内可见
```

### `pub use`

- 创建简化的公共 API
- 在不暴露内部结构的情况下公开功能
- 组合来自不同模块的相关项

``` rust
pub use some_module::SomeType; // 简化重新导出到当前作用域
```

## Path

- 绝对路径：`crate`
- 相对路径：`super` `self`

``` rust
// 绝对路径访问
crate::a::echo(); 

// 相对路径访问
a::echo(); 
self::a::echo();
use super::echo1;
echo1();
```

## Workspace

工作区是一组共享同一个 Cargo.lock 文件和 target 输出目录的包（packages），这些包通常：

- 位于同一个代码仓库中
- 相互关联（如 library crate + binary crate）
- 需要共享依赖项版本

### 典型目录结构

```
my_workspace/
├── Cargo.toml          # 工作区配置文件
├── Cargo.lock          # 共享的依赖锁文件
├── target/             # 共享的构建输出目录
│
├── crate1/             # 第一个成员包
│   ├── Cargo.toml
│   └── src/
│
├── crate2/             # 第二个成员包
│   ├── Cargo.toml
│   └── src/
│
└── tools/              # 工具包（可选）
    ├── Cargo.toml
    └── src/
```

### 创建工作区

#### 创建根目录配置

`my_workspace/Cargo.toml`:

``` toml
[workspace]
members = [
    "crate1",       # library crate
    "crate2",       # binary crate
    "tools/*",      # 通配符支持
]
resolver = "2"      # 使用新版解析器（Cargo 2021+）
```

#### 创建成员包

``` shell
# 创建库crate
cargo new --lib crate1

# 创建二进制crate
cargo new crate2

# 创建工具包
mkdir -p tools/tool1
cargo init --bin tools/tool1
```

#### 工作区操作命令

``` shell
# 构建所有成员
cargo build --workspace

# 测试特定成员
cargo test -p crate1

# 运行二进制成员
cargo run -p crate2 -- --args

# 检查所有依赖更新
cargo update --workspace
```

## Test

### unit test

- 位置​​：与源码在同一文件（src/ 目录下）
​- ​作用​​：测试函数或模块内部的实现细节
​​- 标记​​：`#[cfg(test)] mod tests { }` + `#[test]`

``` rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

### integration test

- ​​位置​​：单独的 tests/ 目录（与 src/ 同级）
​- ​作用​​：测试库的公共接口
​- ​特点​​：每个测试文件都是独立的 crate

``` rust
// project/tests/integration_test.rs
use my_crate;
#[test]
fn it_adds_two() {
    assert!(my_crate::add(2, 2), 4);
}
```
