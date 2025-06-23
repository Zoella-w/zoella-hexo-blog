---
title: IB-Solana-基础&环境搭建
date: 2025-06-18 17:29:21
tags:
    - Solana
categories:
    - Solana
      - IB课程
---

## Sol 源码概览 & 区块链应用

- Solana 区块链的工作原理
  - 共识协议
  - 高性能架构
  - 高并发处理
  - 低延迟
- Runtime 模块
  - 管理账户状态
  - 智能合约执行
- Programs 模块
  - Token Program Stake Program
- Rust 和 区块链

## Sol 课程目标 & 在区块链中的地位

- 掌握 Sol 的基础
- Sol 开发的 8 大核心概念
- Sol 的原生开发、Anchor 框架
- Sol 的优势
- 开发者趋势

## Sol 历史、未来、以及发展

- 公链核心问题
  - 开发者
  - 用户
  - 资本
- Sol 的叙事
- Sol 的运转轨道：人的力量
  - SBF
  - 开发者、黑客松
  - meme
- 生态系统
  - Star Atlas, Saber, Audius, gmt
  - jto, metaora, Kamino, orca, rad, mango, jup
  - BONKbot, Drift, blink
  - 稳定币与商业合作
  - 涉及硬件 => depin, AI
  - solfare => gas mev
- Sol 的价值是如何积累的
- Sol 的牛、熊市案例
- Sol 的未来

## Sol 的架构和常见名词概念

### 常见概念

- accounts
- 合约、验证器（validator）
- 区块浏览器
- address
- transfer
- dApp
- token（代币）
- fee
- block（区块高度）
- 租约（管理费）
- 签名
- 钱包
  - phantom（幻影）
  - solflare（gas 优先）
  - backpack

### 8 大核心技术

- POH
- Tower BFT
- Sealevel
- Cloudbreak
- Gulf Stream
- Turbine
- Pipelining
- Archivers

#### PoH (Proof of History，历史证明)

PoH 不是共识算法，而是一种 ​​**加密时钟机制​​**
- 通过可验证延迟函数（VDF）生成不可篡改的时间序列。
- 每个节点使用 SHA-256 哈希函数创建连续的哈希链：前一个哈希值作为下一个输入，形成有序的时间戳序列。

​技术优势​​：
- ​​时间同步去中心化​​：节点无需通信即可验证事件顺序，减少共识开销。 
- ​并行交易处理基础​​：交易按 PoH 序列预排序，使后续并行执行成为可能。
- 确认速度提升：交易顺序在共识前已确定，对比传统 PBFT（拜占庭容错），区块确认速度提升 ​​10 倍​​以上。

#### Tower BFT（塔式拜占庭容错）

Tower BFT 是在 PoH 基础上优化的 BFT 共识算法，通过 **​​投票锁定机制**​ ​实现快速终局性（Finality）：
- 验证者投票支持区块，锁定时间呈指数增长（1 Slot → 2 Slots → 4 Slots）。
- 当超过 ​​2/3 验证者​​ 投票支持时，区块立即确认。
​
​技术优势：
- ​降低通信复杂度​​：PoH 提供全局时钟，投票消息减少至常数级（传统 PBFT 需 O(N²) 通信）。
- ​抗分叉能力​​：锁定机制强制节点对早期投票负责，减少恶意分叉风险。

#### Sealevel（并行智能合约引擎）

Sealevel 是首个支持 ​**​分片内并行执行** ​​的运行时环境，通过交易预声明状态依赖实现：
- 交易需显式声明读写账户（如 Account A (读), B (写)）。
- 无冲突交易分配至多线程并行执行；冲突交易按序处理。
​​
技术优势：​​
​- ​GPU 多核利用率​​：支持数千个 GPU 核心同时处理交易，吞吐量提升百倍。
​- ​动态资源调度​​：结合 Cloudbreak 数据库，实现状态访问的并发优化。

#### Cloudbreak（水平扩展状态数据库）

Cloudbreak 是 SSD 优化的数据结构，支持 ​​**32 线程并发读写​**​
- 账户数据分片存储于多个 SSD，通过内存映射文件（mmap）加速访问。
- 顺序写入优化：采用追加写入（append-only）减少随机操作，提升 I/O 效率。
​
#### Gulf Stream（无内存池交易转发）

Gulf Stream 通过预知领导者轮换顺序（PoS 机制），将交易 ​**​直接推送给未来领导者​**​：
- 客户端交易引用最近确认的区块哈希，有效期约 24 秒。
- 验证者提前执行交易，失败则丢弃，减轻内存压力。
​​
技术优势：​​
​- ​零内存池阻塞​​：传统链（如以太坊）内存池常达 10 万笔，Solana 仅需处理几秒内的交易。
​- ​领导者切换加速​​：新区块生产无需等待未确认交易，切换时间降至 800 毫秒。

#### Turbine（区块传播协议）

Turbine 是受 BitTorrent 启发的分片传输方案：
- 将区块拆分为 ​**​64 KB 数据包**​​，附加 Reed-Solomon 纠删码（容错率 33%）。
- 领导者按权益权重将数据包分发给顶级验证者，后者逐层广播至全网。
​​
技术优势​​：
- 带宽效率提升​​：传播复杂度从 O(N) 降至 O(log N)，支持 4 万节点在 400 毫秒内同步。 
- ​抗网络攻击​​：纠删码确保即使 1/3 数据包丢失，区块仍可完整恢复。

#### Pipelining（流水线交易验证）

Pipelining 借鉴 CPU 流水线设计，四阶段硬件并行处理：

​1. ​数据获取​​（网卡） → 2. ​​签名验证​​（GPU） → 3. ​​银行处理​​（CPU） → 4. ​​写入账本​​（磁盘）

​技术实现：
- GPU 并行验证数万签名，TPU（交易处理单元）单机处理 50,000 TPS。
- 验证者同时运行 TPU（区块生产）和 TVU（区块验证）双流水线。

#### Archivers（分布式账本存储）

Archivers 将历史数据分片存储于轻量级节点（归档器）：
- 验证者将账本拆分为碎片，通过纠删码分发至归档器网络。
- 归档器硬件门槛低（普通硬盘即可），通过存储证明（PoRep）确保数据完整性。
​​
生态应用：​​
- 与 Filecoin 合作（Old Faithful 计划），实现 PB 级历史数据去中心化存储。
- 新节点同步速度提升 ​​10 倍​​，降低参与门槛。

#### 总结

Solana 八大技术通过紧密协作解决区块链“不可能三角”：
- ​​可扩展性​​：`PoH` + `Sealevel` + `Turbine` 实现 65,000 TPS。
- 去中心化​​：`Archivers` 降低存储门槛，全球节点超 2,000 个。​​
- 安全性​​：`Tower BFT` 确保 1–2 秒终局性，抗 33% 拜占庭节点。

### Sol 三种类型账户

- 数据账户存储数据
  - 系统拥有的账户
  - PDA（程序派生地址）账户
    - 存储智能合约 => 状态
- 程序账户存储可执行程序
- 原生账户

## Solana 技术基础

### 区块链

区块链是由⼀系列按时间顺序链接的区块组成的。每个区块包含三部分：
- 区块头（Block Header）：
  - 前⼀个区块的哈希（Previous Block Hash）：链接到链上前⼀个区块的加密哈希值，确保数据不可篡改。
  - 时间戳（Timestamp）：记录创建区块的时间。
  - 默克尔根（Merkle Root）：所有交易数据的哈希值，确保数据完整性。
- 区块体（Block Body）：
  - 交易列表（Transactions）：包含区块中所有的交易记录。
- 区块哈希（Block Hash）：
  - 当前区块的哈希值：通过对区块头数据进⾏哈希计算⽣成，作为下⼀个区块
的前⼀个区块哈希。

### 交易流程

区块链中的交易流程可以简化为以下⼏个步骤：
1. 交易创建：⽤户 创建⼀笔交易，将资⾦或数据发送给⽤户 。
2. 交易⼴播：该交易被⼴播到区块链⽹络中，所有节点接收到交易信息。
3. 交易验证：⽹络中的矿⼯或验证节点对交易进⾏验证，确保交易的合法性。
4. 交易打包：验证通过的交易被打包进区块中，并由矿⼯竞争挖矿（在 PoW 的情况
下）。
5. 交易确认：新创建的区块被添加到区块链上，交易得到确认。

### 共识机制

共识机制是区块链⽹络中所有节点就数据达成⼀致的⽅式。以常⻅的⼯作量证明（
Proof of Work, PoW）为例：
- ⼯作量证明（Pow）：矿⼯通过计算⼯作量来解决⼀个复杂的数学问题，第⼀个解出问题的矿⼯可以将其区块添加到区块链中。

### 智能合约

智能合约是部署在区块链上的⾃执⾏代码，能根据预定条件⾃动执⾏交易

### 去中⼼化⽹络（P2P）

区块链是由分布在全球的节点组成的去中⼼化⽹络，这些节点共同维护和更新区块链的状态。每个节点都有⼀份完整的区块链副本。

## 环境搭建

### xcode

```bash
xcode-select -p # check if installed
xcode-select --install # install
```

### Solana-cli

安装：
```bash
sh -c "$(curl -sSfL https://release.anza.xyz/stable/install)" # new version (recommend)
sh -c "$(curl -sSfL https://release.solana.com/stable/install)" # old version
```

升级：
```bash
agave-install update # new version
solana-install update # old version
```

检查：
```bash
solana -V
```

### Anchor

安装：
```bash
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force
```

如碰到失败（版本不兼容）可尝试如下⽅案：
```bash
cargo install --git https://github.com/coral-xyz/anchor --tag v0.30.1
anchor-cli --force
```
```bash
avm install latest
avm use latest
```

验证：
```bash
anchor -V
```

### 创建钱包

#### 命令行

> 默认钱包地址为 `~/.config/solana/id.json`

```bash
solana-keygen new # --force overwrite
solana-keygen new -o ~/.config/solana/id2.json # 创建多个钱包
solana config set -k ~/.config/solana/id.json # 设置为默认钱包
```

#### Playground

通过 Playground 创建：https://beta.solpg.io/
- 右上角 -> wallet -> Add
> 创建后会自动 airdrop 5 SOL

### 空投代币

```bash
solana airdrop 5
```
or 通过 faucet 领取 https://faucet.solana.com/

### 查看账户交易

通过区块浏览器查看账户交易：
`https://explorer.solana.com/address/[钱包地址]?cluster=devnet`

例如：https://explorer.solana.com/address/9XCHWjLS6vhQXubtRCP8cfZKG7JT44HSbJuuW6cLwWVp

### 本地部署验证节点

```bash
solana-test-validator
```
```bash
solana config get # get config
solana config set --url localhost # 配置 本地主机验证器
solana config set --url testnet # 配置 test net
```

### solana-cli 常见用法

```bash
# 获取当前环境
solana config get
# 配置环境
solana config set --url <net>
# 创建钱包
solana-keygen new
# 获取当前账户地址
solana address
# 获取账户余额
solana balance <pub_key>
# 转账
solana transfer <recipient_public_key> <amount> --from <sender_keypair_path>
# 举例
solana transfer 9k3V7trvz5sT2o4wat2JhpiURujfLV9JcB4268XGp3W8 0.1 --from ~/.config/solana/id1.json
```

## 课后练习

练习 solana-cli 的常⻅⽤法：
1. 切换到 devnet
2. 创建多个钱包
3. 领取空投
4. 通过 cli 实现各个钱包之间转账交易
5. 查看账户余额
6. 通过区块浏览器查看交易记录
