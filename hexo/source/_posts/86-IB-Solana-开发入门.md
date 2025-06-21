---
title: IB-Solana-开发入门
date: 2025-06-19 16:38:56
tags:
    - Solana
categories:
    - Solana
      - IB课程
---

[Codes in lesson2.1](https://github.com/Zoella-w/IB-Solana/tree/main/2.1_startup_native)
[Codes in lesson2.2](https://github.com/Zoella-w/IB-Solana/tree/main/2.2_startup_anchor)
[Codes in lesson2.3](https://github.com/Zoella-w/IB-Solana/tree/main/2.3_startup_anchor_todo)

## Native Rust 搭建

```bash
cargo new --lib <project_name> # new project
cargo add solana-program # 添加依赖
```

### 修改编译配置

在 Cargo.toml 文件中增加动态链接库：
```bash
[lib]
crate-type = ["cdylib", "lib"]
```

### Build & Deploy

```bash
solana config get
cargo build-sbf # 构建合约
solana program deploy <xxx.so>
solana program close <program_id>
```

## Playground 搭建

https://beta.solpg.io/

## Anchor 搭建

```bash
anchor init <project_name>
anchor test
anchor deploy
```

### Deploy Failed

```bash
solana program show --buffers
solana program close --buffers
```

## 课后作业

通过 Anchor 框架实现⼀个 项⽬，部署到 devnet
1. 新建 todo
2. 查看 todo
3. 删除 todo item
