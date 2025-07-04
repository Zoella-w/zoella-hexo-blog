---
title: IB-Solana-SPLToken合约创建
date: 2025-06-26 16:17:13
tags:
---

[Codes in lesson5.1](https://github.com/Zoella-w/IB-Solana/tree/main/5.1_token_contract)
[Codes in lesson5.2](https://github.com/Zoella-w/IB-Solana/tree/main/5.2_token_cli)

## 代币创建（Create Token） 与 铸造（Mint）

### 核心概念

#### 账户模型基础

- 数据账户
  - 原生账户、代币账户、状态账户
  - 铸币账户
- 程序账户

如何判断一个账户的类型？​​
```text
​步骤 1​​：检查 executable（判断是程序账户 or 数据账户）
如果为 true -> ​​程序账户​​
如果为 false -> 数据账户

​步骤 2​​：检查 owner（判断是哪一种 数据账户）
11111111... -> ​​原生账户​​（系统程序拥有的数据账户）
Tokenkeg... -> ​​代币账户​​（SPL 程序拥有的数据账户）
其他程序 ID -> ​​状态账户​​（其他程序拥有的数据账户）

步骤 3：检查数据内容（判断是哪种代币账户）
Mint 结构体 -> 铸币账户
Account 结构体 -> 代币持有账户（即狭义上的代币账户）
```

**1、数据账户（Data Account）**

数据账户（Data account）是存储状态的“容器”，是 Solana 区块链上的​​基础存储单元

每个账户需要支付​​租金​​以维持数据存储，余额低于阈值可能导致账户被​​回收清除​

只有​​所有者程序​​可以修改账户数据，其他程序只能读取（除非特别授权）

**2、程序账户（Program Account）**

程序账户（Program account）存储可执行代码的“智能合约”，是 Solana 上的“智能合约容器”

程序账户是特殊类型的数据账户（executable=true），基于 ​​Berkeley Packet Filter（BPF）​​虚拟机

部署流程：
```plaintext
1. 创建普通账户（executable=false）
2. 向账户写入已编译的 BPF 字节码
3. 设置 executable=true 并更改所有者
```

**3、原生账户（Native Account）**

- 原生账户（Native account）是存储和管理 SOL，由系统程序（1111111...）拥有
- 每个公钥自动拥有一个原生账户：当用户生成密钥对时，账户即存在（余额为 0）
- 简单操作：
  - SOL 转账
  - 支付交易费用
  - 质押参与网络共识

**4、代币账户**

- 广义上（代币账户体系）
  - 代币账户分为：铸币账户 和 代币持有账户
  - 由 SPL Token 程序（Tokenkeg...）拥有

- 狭义上（代币持有账户，Token Account）
  - 代币持有账户 存储和管理 特定用户对某种 SPL 代币（如 USDC 这类自定义代币）的余额
  - 代币持有账户 必须关联一个 铸币账户（多对一）
  - 关联代币账户（ATA）是特殊的 Token Account

**5、关联代币账户（Associated Token Account）**

- 关联代币账户（ATA）是 Token Account 的一种特殊类型，简化了 SPL Token 的账户管理
- ATA 会自动与一个钱包地址绑定，每个代币账户（Token Account）和每个铸币账户（Mint Account）对应一个唯一的 ATA
- ​​首次接收代币时，系统会自动生成 ATA 并支付租金（存储费用）
- ​钱包/交易所可通过算法推导用户的所有 ATA 地址，统一管理资产

**5、铸币账户（Mint Account）**

- 铸币账户定义一种特定 SPL 代币的基本属性和全局信息
- 一个铸币账户可以关联多个代币账户（代币持有账户）

数据结构：
```rust
pub struct Mint {
    pub mint_authority: Option<Pubkey>, // 铸币权限（谁能创建新代币）
    pub supply: u64,                   // 代币总供应量
    pub decimals: u8,                  // 小数位数（决定代币最小单位）
    pub is_initialized: bool,          // 是否已初始化
    pub freeze_authority: Option<Pubkey> // 冻结权限（谁能冻结账户）
}
```

**6、状态账户（State Account）**

- 状态账户存储​​应用程序特定的状态数据
- ​​​由​​自定义程序拥有（既不是系统程序，也不是 SPL Token 程序）
- 是 Solana 账户系统中最灵活的数据存储形式

常见 owner：
- 自定义程序（最常见）
  - ​由​开发者部署的智能合约​​
  - 比如：投票程序、游戏程序、DeFi 协议
- 中间件程序
- DAO 治理程序

#### 核心组件

- 铸币账户 (Mint Account)
- ​​代币账户 (Token Account)​​
- 关联令牌账户 (Associated Token Account, ATA)

### 创建代币（Create Token / Create Mint Account）的流程

#### 底层流程

1. 创建账户容器：在区块链上分配空间
2. 设置所有权：将新账户的所有权赋予 SPL Token 程序
3. 初始化代币参数：写入代币的元数据

#### 完整代码

```rust
use solana_client::nonblocking::rpc_client::RpcClient;
use solana_sdk::{
    commitment_config::CommitmentConfig, native_token::LAMPORTS_PER_SOL, program_pack::Pack,
    signature::Keypair, signer::Signer, system_instruction::create_account,
    transaction::Transaction,
};
use spl_token::{instruction::initialize_mint2, state::Mint, ID as TOKEN_PROGRAM_ID};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // 1.1 建立 RPC 客户端连接
    let client = RpcClient::new_with_commitment(
        String::from("http://127.0.0.1:8899"), // 本地 Solana 节点
        CommitmentConfig::confirmed(),          // 确认级别
    );

    // 1.2 创建必要的密钥对
    let authority_keypair = Keypair::new(); // 既是 支付账户，也是 权限账户
    let mint_account = Keypair::new();     // 新代币的铸币账户

    // 2.1 确定铸币账户大小
    let mint_account_len = Mint::LEN; // 固定 82 字节

    // 2.2 计算最小租金
    // 当前约为 0.002 SOL（租全年的费用）
    let mint_account_rent = client
        .get_minimum_balance_for_rent_exemption(mint_account_len)
        .await?;

   // 3.1 创建系统指令
   let create_mint_account_ix = create_account(
       &authority_keypair.pubkey(),    // 支付账户：从该账户扣除租金和交易费
       &mint_account.pubkey(),        // 新账户地址：新创建的铸币账户地址
       mint_account_rent,            // 租金金额：代币账户存储所需租金
       mint_account_len as u64,       // 账户数据空间大小（82字节）
       &TOKEN_PROGRAM_ID,             // 账户所有者（SPL Token 程序，是一个固定值！）
   );

   // 4.1 创建初始化指令
   let initialize_mint_ix = initialize_mint2(
       &TOKEN_PROGRAM_ID,               // SPL Token 程序 ID
       &mint_account.pubkey(),         // 要初始化的铸币账户
       &authority_keypair.pubkey(),    // 铸币权限账户（可创建新代币）
       Some(&authority_keypair.pubkey()),// 冻结权限账户（可选，设为 None 表示无冻结权限）
       9,                              // 精度（9位小数）
   )?;

   // 5.1 请求空投
   // 为支付账户添加 5 SOL（测试网/开发网专有）
   let transaction_signature = client
       .request_airdrop(&authority_keypair.pubkey(), 5 * LAMPORTS_PER_SOL)
       .await?;

   // 5.2 等待空投确认
   loop {
    // 确保资金到位后才进行下一步
       if client.confirm_transaction(&transaction_signature).await? {
           break;
       }
   }

    // 6.1 创建交易
   let mut transaction = Transaction::new_with_payer(
       // 两个指令打包到同一交易：创建空账户容器 + 初始化代币参数
       &[create_mint_account_ix, initialize_mint_ix],
       // 支付账户
       Some(&authority_keypair.pubkey()),
   );

   // 6.2 交易签名
   transaction.sign(
       // 签名者：支付账户（支付租金和交易费）+ 铸币账户（证明对新账户的控制权）
       &[&authority_keypair, &mint_account],
       // 最新区块哈希（防止重放攻击的交易唯一标识）         
       client.get_latest_blockhash().await?,
   );

   // 6.3 发送并确认交易
   match client.send_and_confirm_transaction(&transaction).await {
       Ok(signature) => println!("Transaction Signature: {}", signature),
       Err(err) => eprintln!("Error sending transaction: {}", err),
   }

    Ok(())
}
```

### 铸造代币（Mint）到 关联令牌账户（ATA）的流程

#### 核心概念

- 铸造（Mint）：创建新的代币单位并添加到流通中
- 代币分配：将新创建的代币分配给特定用户

#### 完整代码

```rust
use solana_client::nonblocking::rpc_client::RpcClient;
use solana_sdk::{
    commitment_config::CommitmentConfig, native_token::LAMPORTS_PER_SOL, program_pack::Pack,
    signature::Keypair, signer::Signer, system_instruction::create_account,
    transaction::Transaction,
};
use spl_associated_token_account::{
    get_associated_token_address, instruction::create_associated_token_account_idempotent,
};
use spl_token::{
    instruction::{initialize_mint2, mint_to_checked},
    state::Mint,
    ID as TOKEN_PROGRAM_ID,
};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // =====================================================================
    // 1.0 初始化环境
    // =====================================================================

    // 1.1 建立 RPC 客户端连接
    let client = RpcClient::new_with_commitment(
        String::from("http://127.0.0.1:8899"), // 本地 Solana 节点
        CommitmentConfig::confirmed(),          // 确认级别
    );

    // 1.2 创建必要的密钥对
    let authority_keypair = Keypair::new(); // 既是 支付账户，也是 权限账户
    let mint_account = Keypair::new();     // 新代币的铸币账户

    // 1.3 计算关联令牌账户（ATA）地址
    // 使用确定性算法计算用户对该代币的专属地址
    let associated_token_account =
        get_associated_token_address(&authority_keypair.pubkey(), &mint_account.pubkey());

    // =====================================================================
    // 2.0 准备阶段
    // =====================================================================
    
    // 调用 setup 函数 创建代币（Create Token / Create Mint Account）和 ATA
    setup(&client, &authority_keypair, &mint_account).await?;

    // =====================================================================
    // 3.0 铸造阶段 - 铸造代币到ATA（核心流程）
    // =====================================================================

    // 3.1 获取代币精度（小数位数）
    let mint_decimals = client
        .get_token_account_balance(&associated_token_account)
        .await?
        .decimals;

    // 3.2 计算铸造数量
    // 1个完整代币
    // 公式: 数量 = 完整代币数量 × 10^小数位数
    let amount_to_mint = 1 * 10_u64.pow(mint_decimals as u32);

    // 3.3 构建铸造指令
    let mint_to_ix = mint_to_checked(
        &TOKEN_PROGRAM_ID,              // SPL 代币程序地址
        &mint_account.pubkey(),         // 铸币账户地址
        &associated_token_account,      // 目标 ATA 地址
        &authority_keypair.pubkey(),    // 铸币权限账户
        // 签名者列表，提供多签支持（这里只有铸币权限账户）
        &[&authority_keypair.pubkey()],
        amount_to_mint,                 // 铸造数量
        mint_decimals,                  // 代币精度（用于验证数量有效性）
    )?;

    // 3.4 创建交易
    let mut transaction = Transaction::new_with_payer(
        // 仅包含铸造指令
        &[mint_to_ix],
        // 支付账户
        Some(&authority_keypair.pubkey()),
    );

    // 3.5 获取最新区块哈希（防止重放攻击的交易唯一标识）   
    let latest_blockhash = client.get_latest_blockhash().await?;

    // 3.6 交易签名
    // 铸币权限账户（authority_keypair）授权操作
    transaction.sign(&[&authority_keypair], latest_blockhash);

    // 3.7 发送并确认交易
    match client.send_and_confirm_transaction(&transaction).await {
        Ok(signature) => println!("铸造成功! 交易签名: {}", signature),
        Err(err) => eprintln!("铸造失败: {}", err),
    }

    Ok(())
}

// =====================================================================
// 2.0 准备阶段
// - 创建代币（Create Token / Create Mint Account），同上
// - 创建关联令牌账户（ATA）
// =====================================================================

async fn setup(...) -> anyhow::Result<()> {
    // 2.1 空投 SOL 作为初始资金
    let transaction_signature = client
        .request_airdrop(&authority_keypair.pubkey(), 5 * LAMPORTS_PER_SOL)
        .await?;
    
    // 2.2 等待空投确认
    loop {
        if client.confirm_transaction(&transaction_signature).await? {
            break;
        }
    }
    
    // 2.3 设置代币参数
    let decimals = 9; // 代币小数位数
    
    // 2.4 计算铸币账户所需租金
    let mint_account_len = Mint::LEN; // 铸币账户固定大小（82字节）
    let mint_account_rent = client
        .get_minimum_balance_for_rent_exemption(mint_account_len)
        .await?;
    
    // 2.5 创建铸币账户容器
    let create_mint_account_ix = create_account(
        &authority_keypair.pubkey(), // 支付账户
        &mint_account.pubkey(),     // 新账户地址
        mint_account_rent,          // 租金金额
        mint_account_len as u64,    // 账户大小
        &TOKEN_PROGRAM_ID,          // 所有者：SPL代币程序
    );
    
    // 2.6 初始化铸币账户
    let initialize_mint_ix = initialize_mint2(
        &TOKEN_PROGRAM_ID,              // SPL 代币程序
        &mint_account.pubkey(),         // 目标铸币账户
        &authority_keypair.pubkey(),    // 铸币权限账户
        Some(&authority_keypair.pubkey()), // 冻结权限账户（可选）
        decimals,                       // 小数位数
    )?;
    
    // 2.7 创建关联令牌账户（ATA）
    let create_ata_ix = create_associated_token_account_idempotent(
        &authority_keypair.pubkey(), // 支付账户：从该账户扣除租金和交易费
        &authority_keypair.pubkey(), // 钱包地址（代币所有者）
        &mint_account.pubkey(),      // 铸币账户地址
        &TOKEN_PROGRAM_ID,           // 账户所有者（SPL Token 程序，是一个固定值！）
    );
    
    // 2.8 打包并发送准备交易
    let mut transaction = Transaction::new_with_payer(
        // 三个指令打包到同一交易：创建空账户容器 + 初始化代币参数 + 创建ATA账户
        &[create_mint_account_ix, initialize_mint_ix, create_ata_ix],
        // 支付账户
        Some(&authority_keypair.pubkey()),
    );
    
    // 2.9 交易签名
    transaction.sign(
        // 签名者：支付账户（支付租金和交易费）+ 铸币账户（证明对新账户的控制权）
        &[authority_keypair, mint_account],
        // 最新区块哈希（防止重放攻击的交易唯一标识）   
        client.get_latest_blockhash().await?,
    );
    
    // 2.10 发送并确认交易
    client.send_and_confirm_transaction(&transaction).await?;
    
    Ok(())
}
```

## Solana 项目结构

核心文件：
- processor.rs：核心业务逻辑，处理指令
- state.rs：定义账户的状态和扩展字段
- instruction.rs：定义各种代币操作的指令
- error.rs：定义了程序可能抛出的错误
- lib.rs：程序入口点，汇总各个模块

## 实现 Token 交互合约

### 项目初始化

#### 新建项目

新建合约项目 contract 和 调用合约的项目 cli

#### 项目配置

contract 依赖添加：
```shell
cargo add solana-program
cargo add borsh spl-token spl-associated-token-account
```

contract Cargo.toml 文件：
```toml
[package]
name = "token"
version = "0.1.0"
edition = "2024"

[dependencies]
borsh = "1.5.7"
solana-program = "2.3.0"
# 忽略入口文件
spl-associated-token-account = { version = "7.0.0", features = [
    "no-entrypoint",
] }
# 忽略入口文件
spl-token = { version = "8.0.0", features = ["no-entrypoint"] }

[lib]
crate-type = ["cdylib", "lib"]

[features]
no-entrypoint = []
```

cli 依赖添加：
```shell
cargo add spl-token borsh spl-associated-token-account solana-program solana-sdk solana-client
```

### 编译部署

编译：
```shell
cargo build-sbf
```

部署：
```shell
solana program deploy ./target/sbpf-solana-solana/release/token.so
```
