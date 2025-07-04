---
title: IB-Solana-token解析
date: 2025-06-23 15:33:54
tags:
    - Solana
categories:
    - Solana
      - IB课程
---

## solana 上的代币

代币是代表对各种资产所有权的数字资产。代币化使得财产权的数字化成为可能，是管理 可替代和不可替代资产 的基本组成部分
- **可替代代币** 代表同类型和同价值的可互换和可分割资产（例如 USDC）
- **不可替代代币（NFT）** 代表不可分割资产的所有权（例如艺术品）
- **半可替代代币**（如股权代币）

## SPL（Solana Program Library）

SPL Token 是 Solana 上的一套智能合约标准，类似于以太坊的 ERC-20 和 ERC-721 标准，但针对 Solana 的高性能架构进行了优化

SPL Token 它定义了在 Solana 上创建、管理和交易代币的规则和接口，包含：
- ​​代币程序​​：核心逻辑合约
- ​​账户标准​​：定义代币存储结构
​​- 指令集​​：标准化的操作接口

### 核心组件

- 代币程序（Token Program）包含与网络上的代币（包括可替代和不可替代）交互的所有指令逻辑。
- 铸币账户（Mint Account）代表一种特定类型的代币，并存储关于代 币的全局元数据，如总供应量和铸造权限（有权创建新代币单位的地址）。
- 代币账户（Token Account）跟踪特定地址拥有的特定类型代币（铸 造账户）的单位数量。
- 关联代币账户（Associated Token Account）

### Token Program（代币程序）

Token Program（代币程序）是 Solana 上的智能合约，负责处理所有代币操作的核心逻辑。它提供了一套标准化的接口，支持创建和管理各种类型的代币

#### 核心功能

​​账户初始化​​：创建铸币账户和代币账户
​​代币铸造​​：增发新的代币单位
​​代币转移​​：在账户间转移代币
​​代币销毁​​：减少代币供应量
​​权限管理​​：控制铸币、冻结等权限
​​委托操作​​：允许第三方临时管理代币


### Mint Account（铸币账户）

Mint Account（铸币账户）是代币的“定义文件”，存储代币的全局元数据。每个铸币账户代表一种独特的代币类型。

#### 属性

- Decimals（小数位数）：定义代币的最小单位，通常是 0 到 9（创建后不可更改）
- Supply（供应量）：代币的当前总供应量
- Mint Authority（铸币权限）：可以铸造新的代币的账户，用于控制增发
- Freeze Authority（冻结权限）：可以冻结或解冻代币账户的权限（可选）
- ​​唯一标识​​：每个铸币账户地址对应一种代币

#### 功能

- 铸造代币：当 Mint Authority 执行铸币操作时，新的代币会增加到总供应量中，并分配给指定的 Token Account
- 销毁代币：减少总供应量

### Token Account（代币账户）

Token Account（代币账户）存储特定用户持有的特定代币的数量和状态信息。每个代币账户：
- 关联一个铸币账户（代币类型）
- 关联一个所有者（用户或程序）
- 记录当前余额

#### 属性

- Amount（余额）：账户中持有的代币数量
- Owner（账户拥有者）：控制该账户的用户或合约地址
- Mint（铸币账户关联）：该账户与哪个 Mint Account 相关联
- Delegate（代理账户）：可以被授权管理该账户的其他账户（可选）
- State（状态）：账户是否处于冻结状态

#### 功能

- 接受和发送代币：Token Account 可以接受其他账户的代币，并通过转账指令将其发送给其他账户
- 代理权限管理：可以设置 Delegate 来授权第三方管理该账户的代币

### Associated Token Account（关联代币账户）

Associated Token Account（ATA）是 Token Account 的一种特殊类型，简化了 SPL Token 的账户管理

ATA 是自动与一个钱包地址绑定的账户，每个代币账户 Token Account 和每个铸币账户 Mint Account 对应一个唯一的 ATA，该账户类型极大简化了代币管理

#### 传统代币账户的问题

假设 Alice 想给 Bob 发送 USDC：

> Alice 需要知道 Bob 的 USDC 账户地址
> 如果 Bob 还没有 USDC 账户，Alice 无法发送
> Bob 必须手动创建账户并告诉 Alice 地址
> 整个过程繁琐且容易出错
> ATA 如何解决问题

#### 使用 ATA 解决问题

> Alice 通过公式计算 Bob 的 USDC 邮箱地址
> 如果邮箱不存在，Alice 可以顺便创建它
> 直接发送 USDC 到计算出的地址
> Bob 自动收到 USDC，无需任何操作

#### 特点

- 唯一性：每个钱包地址只能有一个和某个 Mint 关联的 ATA
- 自动生成：Solana 提供了工具来自动生成 ATA，处理钱包地址与代币账户之间的关联，方便用户管理不同的 SPL Token

## Token 命令

### 创建 Token

```shell
spl-token create-token
```

```shell
spl-token create-token --decimals 2 --name "My Token" --symbol "MTK" --url "https://my.token.url"
```

### 查看 Token 账户

```shell
spl-token account-info --address <token_address>
```

```text
SPL Token Mint
  Address: HHfsG1zwz175BTtRiL8LqXhzgjUFXNBf7YtXsjfyMMvJ # TOKEN_MINT_ADDRESS
  Program: TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA
  Supply: 0
  Decimals: 9
  Mint authority: WoFtMc6fw3C4EkJr2JYfH4kEAd75sChvsz5HR5Q3joU # wallet pubkey
  Freeze authority: (not set)
```

### 创建 Associated Token Account

```shell
spl-token create-account <TOKEN_MINT_ADDRESS>
```

```text
Creating account 91HoZZtpRoGrHnFTRqu4Yf2VAZVDymEzRBKnVGT7f4s7 # ATA
Signature: 3CFByMJ7mH5rUgE5FZYSCjj7J5Zd2PavG1vWvTCzVKs1n3QiHvvrd2CPPaFZiHFzsjp953Ta4kBpf3g4X3bDmzMP
```

### 创建 Token Account

```shell
spl-token create-account <TOKEN_MINT_ADDRESS> <ACCOUNT_KEYPAIR>
```

```text
Creating account DbXozEQpYYFJGnUNvwfYbYvP8iqiqMV7N8BwDf59vpro # TOKEN_ACCOUNT_ADDRESS
Signature: 5sRBKd8fTQZmZyNkvVYgqDmbU4CCDqxf9o2pbEn7ZPC7JwqbYVLNtRjTuDErF2sjTsCob9fjLQ7xmhqQHkMNr2xz
```

### mint

#### 自动 mint 到 ATA（关联代币账户）中

```shell
spl-token mint <TOKEN_MINT_ADDRESS> <amount>
```

```text
Minting 100 tokens
  Token: HHfsG1zwz175BTtRiL8LqXhzgjUFXNBf7YtXsjfyMMvJ
  Recipient: 91HoZZtpRoGrHnFTRqu4Yf2VAZVDymEzRBKnVGT7f4s7
Signature: 4TC95YXBhhY5WUGBDs622yyWiE3hG7aRU3dZkWxWrQsXFhjr3DpMB4AtX4q2jsudMV8fr3gogabQLvzkaPbh81vu
```

#### mint 到 Token Account 中

```shell
spl-token mint <TOKEN_MINT_ADDRESS> <amount> -- <TOKEN_ACCOUNT_ADDRESS>
```

```text
Minting 100 tokens
  Token: HHfsG1zwz175BTtRiL8LqXhzgjUFXNBf7YtXsjfyMMvJ
  Recipient: DbXozEQpYYFJGnUNvwfYbYvP8iqiqMV7N8BwDf59vpro
Signature: 5STxsFQh5PQfN1zSjVYiTmzcgzwwXzGjpU4gQbTYjrMGT5JgDCQCcXqCVZGeEUnr3WDSKMSAqv68SQqLBoBEuWB7
```
