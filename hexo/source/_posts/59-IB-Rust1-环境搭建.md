---
title: IB-Rust1-环境搭建
date: 2025-05-20 12:17:51
tags:
    - Rust
categories:
    - Rust
      - IB课程
---

[Codes in lesson1.1](https://github.com/Zoella-w/IB-Rust/tree/main/1.1_hello_world)

[Codes in lesson1.2](https://github.com/Zoella-w/IB-Rust/tree/main/1.2_hello_cargo)

## 一、Rust 介绍

### 1、内存安全

不允许空指针和悬空指针，可预防 C++ 中的许多类型错误

### 2、静态类型

编译器必须在编译期知道所有变量的类型

在编译器能推导变量类型的情况下，不需要手动为变量指定类型

### 3、并发编程

使开发者能编写高效、安全的多线程程序，避免数据竞争等并发问题

## 二、环境搭建

### 1、安装

通过 `rustup` 下载 Rust

```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

检查是否正确安装 Rust

```console
$ rustc --version
```

### 2、rustup 命令

#### （1）升级 rust 工具链和 rustup 本身

```rustup update```

#### （2）卸载 rust

```rustup self uninstall```

#### （3）打开离线文档

```rustup doc```

## 三、vscode 配置

1、rust-analyzer

2、even-better-toml

3、crates（已无法使用，换成了 Dependi）

## 四、hello world

rust 文件以 .rs 结尾，如 main.rs

```rust
fn main() {
    println!("Hello, world!");
}
```

### 1、编译

```shell
rustc main.rs
```

编译成功后输出一个二进制可执行文件 main

### 2、运行

rust 是 **预编译静态类型**（*ahead-of-time compiled*）语言，即一旦拥有了编译后的可执行文件，不需要安装 Rust 即可运行

```shell
./main
```

## 五、cargo

### 1、初始化项目

```shell
cargo new hello_cargo
cd hello_cargo
```

其中的 Cargo.toml 文件包含：

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

**[package]**

- 项目名称
- 项目版本
- Rust 版本

**[dependencies]**

项目依赖，Rust 中的代码包为 crates

### 2、构建并运行

#### （1）构建

```bash
cargo build
```

创建可执行文件 *target/debug/hello_cargo*

#### （2）运行

```bash
./target/debug/hello_cargo
```

#### （3）构建并运行

```bash
cargo run
```

同时编译并运行生成的可执行文件，更方便

#### （4）发布构建

```shell
cargo build --release
```

在 *target/release* 下生成可执行文件，编译优化让 Rust 代码运行的更快，但消耗更长的编译时间

**两种构建对比：**

- `cargo build` 用于开发，因为经常需要重新构建

- `cargo build --release` 用于为用户构建最终程序，不会经常重新构建，并希望程序运行得更快

### 3、添加依赖

crate 是一个 Rust 代码库，可以包含任意能被其他程序使用的代码，但不能自执行

添加一个随机数的库（crate）：

```toml
[dependencies]
rand = "0.8.5"
```

然后重新构建项目

---

当增加了新依赖，Cargo 会从 [Crates.io](https://crates.io/) 获取依赖，并将指定的依赖版本写入 Cargo.lock 文件

> Cargo.lock 文件
> 
> 确保任何人在任何时候重新构建代码，都会产生相同的结果，因为 Cargo 只会使用指定的依赖版本（类似 package-lock.json）

---

如需升级 crate，`cargo update` 会忽略 *Cargo.lock* 文件，并计算出所有符合 *Cargo.toml* 声明的最新版本，接着将其写入 *Cargo.lock*。

但是 Cargo 只会寻找 `0.8.x` 的版本，假设 `rand` crate 发布了 `0.8.6` 和 `0.9.0` 两个新版本，运行 `cargo update` 后会升级到 `0.8.6` 而不是 `0.9.0`。这是因为 `0.9.0` 相对于 `0.8.5` 主版本发生了变化。

如需升级到 `0.9.x` 版本，更新 *Cargo.toml* 文件：

```toml
[dependencies]
rand = "0.9.0"
```

### 4、配置国内镜像

为了使用 `crates.io` 之外的注册服务，需要对 `$HOME/.cargo/config.toml` 文件进行配置，添加新的服务提供商

> **cargo v1.68 开始支持稀疏索引**：不需要完整克隆 crates.io-index 仓库，加快获取包的速度

> 协议推荐使用 git，如配置 git 协议后无法正常获取和编译 crate，可以换 https 协议试试

两种实现方式：增加新的镜像地址 和 覆盖默认的镜像地址

#### （1）增加新的镜像地址

如果支持稀疏索引：

```toml
### https，任选一种即可
[registries]
ustc = { index = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/" }
[registries.ustc]
index = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"

### git，任选一种即可
[registries]
ustc = { index = "sparse+git://mirrors.ustc.edu.cn/crates.io-index/" }
[registries.ustc]
index = "sparse+git://mirrors.ustc.edu.cn/crates.io-index/"
```

---

这种方式只会新增一个新的镜像地址，因此在引入依赖的时候，需要指定该地址，例如在项目中引入 `time` 包，你需要在 `Cargo.toml` 中使用以下方式引入:

```toml
[dependencies]
rand = {  registry = "ustc" }
```

#### （2）【推荐】覆盖默认的镜像地址

这种方式不需要修改 `Cargo.toml` 文件，因为它会直接使用新注册服务替代默认的 `crates.io`。

在 `$HOME/.cargo/config.toml` 添加以下内容：

```toml
[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "https://mirrors.ustc.edu.cn/crates.io-index/"
```

创建一个新的镜像源 `[source.ustc]`，然后将默认的 `crates-io` 替换成新的镜像源：`replace-with = 'ustc'`

---

**可用镜像列表：**

```toml
# 中科大
"https://mirrors.ustc.edu.cn/crates.io-index/"
"git://mirrors.ustc.edu.cn/crates.io-index"
# 清华
"https://mirrors.tuna.tsinghua.edu.cn/git/crates.io-index.git/"
# 字节
"https://rsproxy.cn/crates.io-index/"
```
