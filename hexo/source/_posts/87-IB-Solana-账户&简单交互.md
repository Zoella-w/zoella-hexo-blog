---
title: IB-Solana-账户&简单交互
date: 2025-06-21 19:21:19
tags:
    - Solana
categories:
    - Solana
      - IB课程
---

[Codes in lesson3](https://github.com/Zoella-w/IB-Solana/tree/main/3_account_interaction)

## 账户

- 数据账户，用来存储数据
  - 系统所有账户
  - 程序派生账户（PDA）
- 程序账户，用来存储可执行程序
- 原生账户，指 Solana 上的原生程序，例如：System，Stake，以及 Vote

### 账户结构体

- Account
- AccountInfo

#### Account

```rust
#[derive(Deserialize, PartialEq, Eq, Clone, Default)]
#[serde(rename_all = "camelCase")]
pub struct Account {
    /// lamports in the account -- Sol 余额，lamports 是最小单位
    pub lamports: u64,
    /// data held in this account -- 存储数据，和合约或用户相关
    #[serde(with = "serde_bytes")]
    pub data: Vec<u8>,
    /// the program that owns this account. If executable, the program that loads this account.  -- 所有者的数据和权限
    pub owner: Pubkey,
    /// this account's data contains a loaded program (and is now read-only)  -- data 中存放的是否为可执行代码
    pub executable: bool,
    /// the epoch at which this account will next owe rent -- 租金，存储程序在 Sol 上需要收费
    pub rent_epoch: Epoch,
}
```

#### AccountInfo

```rust
/// Account information
#[derive(Clone)]
#[repr(C)]
pub struct AccountInfo<'a> {
    /// Public key of the account -- 公钥，账户唯一标识
    pub key: &'a Pubkey,
    /// The lamports in the account.  Modifiable by programs. -- Sol 余额
    pub lamports: Rc<RefCell<&'a mut u64>>,
    /// The data held in this account.  Modifiable by programs. -- 同 Account
    pub data: Rc<RefCell<&'a mut [u8]>>,
    /// Program that owns this account -- 同 Account
    pub owner: &'a Pubkey,
    /// The epoch at which this account will next owe rent -- 同 Account
    pub rent_epoch: Epoch,
    /// Was the transaction signed by this account's public key? -- 当前交易是否被该账户的公钥签名了
    pub is_signer: bool,
    /// Is the account writable? -- 是否能被修改
    pub is_writable: bool,
    /// This account's data contains a loaded program (and is now  read-only) -- 同 Account
    pub executable: bool,
}
```

#### Account & AccountInfo 对比

- AccountInfo：
  - 更**轻量级**，包含对区块链上现有账户数据的引用。
  - 用于在 Solana **程序（智能合约）内部**处理账户。
  - 适合在**链上**处理和操作账户数据。
- Account：
  - 更**完整**的账户表示，包含账户的所有数据副本。
  - 常用于**客户端或测试环境**中，用于模拟或获取完整的账户状态。
  - 适合**离线**处理或全局管理账户数据。

#### 要点

- 账户是用来存放数据的
- 每个账户都有一个独一无二的地址
- 每个账户大小不能超过 10MB
- 账户大小是静态的
- 账户数据存储需要付租金
- 默认的账户所有者是“系统程序”

### 程序派生账户（PDA）

相关概念：https://solana.com/zh/docs/core/pda

#### PDA 注意事项

1. **不能直接签名交易**
   - 限制: PDA 账户没有私钥，因此无法像普通账户那样签名交易。
   - 影响: 这意味着 PDA 无法自主发起交易，它只能被相关的 **智能合约程序** 用作 **数据存储或执行** 操作。这确保了 PDA 只能在程序的控制下使用。
2. **地址碰撞的可能性**
   - 限制: 在理论上，虽然非常罕见，使用相同的程序 ID 和相同的种子值可以生成相同的 PDA 地址。
   - 影响: 这意味着在设计智能合约时，开发者必须谨慎选择种子值，以确保不会产生地址碰撞。一般来说，通过使用唯一的种子（比如用户的公钥和其他独特的数据），可以避免这种问题。
3. **PDA 地址的最大长度**
   - 限制: PDA 的种子值组合在一起不能超过 32 字节（bytes）。
   - 影响: 如果数据太大，可能无法直接作为种子使用，可能需要对数据进行哈希处理或其他方式来适应这个限制。
4. **生成 PDA 的计算成本**
   - 限制: PDA 是通过哈希函数计算生成的，这个过程消耗计算资源。
   - 影响: 在性能敏感的应用中，频繁生成 PDA 可能增加链上计算的成本，影响程序的执行效率。因此，在设计程序时需要平衡性能和安全性。
5. **单一程序的访问**
   - 限制: PDA 账户是由一个特定的程序生成和控制的，只有这个程序可以操作该 PDA 账户。
   - 影响: 虽然这提供了很强的安全性，但也意味着不能轻易地跨程序共享 PDA 账户。如果多个程序需要访问相同的数据，可能需要复杂的设计或数据复制。
6. **内存账户的使用**
   - 限制: 如果 PDA 被用作 Solana 上的内存账户（即需要存储较多的数据），这些账户的大小是有限制的，超过一定大小需要支付更高的费用来增加内存租金。
   - 影响: 需要考虑 PDA 账户的数据量，避免不必要的存储开销，或者拆分数据存储到多个 PDA 账户中。

#### PDA 应用场景

1. 用户状态管理：只有智能合约可以访问，保障用户状态数据隐私性
2. 去中心化金融（DeFi）协议
3. NFT 元数据存储
4. DAO（去中心化自治组织）投票系统
5. 时间锁合约
6. 多签（Multisig）钱包
7. 去中心化身份验证

## 开发使用的 Rust 库

- [solana_client（客户端）](https://docs.rs/solana-client/latest/solana_client/)
- [solana_sdk（操作 Sol）](https://docs.rs/solana-sdk/latest/solana_sdk/)
- [solana_program（合约）](https://docs.rs/solana-program/latest/solana_program/)

## 实战

- 启动本地环境：`solana-test-validator`
- 更改 solana 配置，链接到本地开发环境
- 创建本地账户
- 给新建账户空投 sol
- 使用 SDK
  - 空投 sol
  - 获取账号信息
  - 转移 sol
- 通过 JsonRpc 获取账户信息

```bash
curl http://127.0.0.1:8899 -s -X \
  POST -H "Content-Type: application/json" -d '
  {
    "jsonrpc": "2.0",
    "id": 1,
    "method": "getAccountInfo",
    "params": [
      "mjvFkAaysJHDbEAvcJVd6Q2ZKb9kTeGPx6rAqVVQcUR",
      {
        "encoding": "base58"
      }
    ]
  }
'
```
```json
{"jsonrpc":"2.0","result":{"context":{"apiVersion":"2.2.17","slot":100523},"value":{"data":["","base58"],"executable":false,"lamports":9999995000,"owner":"11111111111111111111111111111111","rentEpoch":18446744073709551615,"space":0}},"id":1}
```
