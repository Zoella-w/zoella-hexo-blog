---
title: IB-Solana-Defi项目拆解2
date: 2025-07-09 11:22:44
tags:
    - Solana
categories:
    - Solana
      - IB课程
---

[Codes in lesson7](https://github.com/Zoella-w/IB-Solana/tree/main/7_swap)

## Admin 命令详解

### `create_amm_config`（创建 AMM 配置）

src/instructions/lib.rs：
```rust
pub fn create_amm_config(
    ctx: Context<CreateAmmConfig>,
    index: u16,
    trade_fee_rate: u64,
    protocol_fee_rate: u64,
    fund_fee_rate: u64,
    create_pool_fee: u64,
) -> Result<()> {
    // 验证费率参数有效性
    // 1. 确保费率不超过 FEE_RATE_DENOMINATOR_VALUE（通常是 1e6 或 100%）
    assert!(trade_fee_rate < FEE_RATE_DENOMINATOR_VALUE);
    assert!(protocol_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    assert!(fund_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    // 2. 确保 protocol_fee_rate + fund_fee_rate 不超过 100%
    assert!(fund_fee_rate + protocol_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    // 调用实际实现
    instructions::create_amm_config(
        ctx,
        index,
        trade_fee_rate,
        protocol_fee_rate,
        fund_fee_rate,
        create_pool_fee,
    )
}
```

src/instructions/admin/create_config.rs：
```rust
#[derive(Accounts)]
#[instruction(index: u16)]
// 「账户上下文」结构体
pub struct CreateAmmConfig<'info> {
    /// 签名者账户：该账户将被授予协议所有者的权限
    #[account(
        mut,  // 标记账户为​​可变
        // address = <预期地址> @ <错误类型>
        address = crate::admin::ID @ ErrorCode::InvalidOwner  // 必须是预定义的管理员地址
    )]
    pub owner: Signer<'info>, // 签名者账户

    /// 初始化配置状态账户，存储协议所有者地址和费率
    #[account(
        init,  // 创建并初始化新账户
        seeds = [  // PDA 种子
            AMM_CONFIG_SEED.as_bytes(),  // 固定种子
            &index.to_be_bytes()  // 索引的大端字节表示
        ],
        bump,  // 自动寻找有效的 bump 值
        payer = owner,  // 指定支付账户为账户所有者
        space = AmmConfig::LEN  // 指定账户空间大小
    )]
    pub amm_config: Account<'info, AmmConfig>, // AMM 配置账户

    pub system_program: Program<'info, System>, // 系统程序
}

pub fn create_amm_config(
    ctx: Context<CreateAmmConfig>, // 账户上下文
    index: u16,                    // 配置索引
    trade_fee_rate: u64,           // 交易费率
    protocol_fee_rate: u64,        // 协议费率
    fund_fee_rate: u64,            // 基金费率
    create_pool_fee: u64,          // 创建池费用
) -> Result<()> {
    // 获取可写的 amm_config 账户引用
    let amm_config = ctx.accounts.amm_config.deref_mut();
    // 设置配置字段
    amm_config.protocol_owner = ctx.accounts.owner.key(); // 设置协议所有者
    amm_config.bump = ctx.bumps.amm_config; // 存储 bump 值
    amm_config.disable_create_pool = false; // 启用池创建
    amm_config.index = index; // 配置索引
    amm_config.trade_fee_rate = trade_fee_rate; // 交易费率
    amm_config.protocol_fee_rate = protocol_fee_rate; // 协议费率
    amm_config.fund_fee_rate = fund_fee_rate; // 基金费率
    amm_config.create_pool_fee = create_pool_fee; // 创建池费用
    amm_config.fund_owner = ctx.accounts.owner.key(); // 基金所有者

    Ok(()) // 返回成功
}
```

#### 作用

创建 AMM 的全局配置，定义交易费率、协议费率、资金费率等参数。这是整个系统的基础设置，通常由管理员在部署时调用一次

#### 工作内容

- 输入参数
  - `index`：配置的唯一索引
  - `trade_fee_rate`：交易手续费率（例如 0.25%，即 2500 表示 2500/1e6）
  - `protocol_fee_rate`：协议费率（从交易费中分配给协议所有者的部分）
  - `fund_fee_rate`：资金费率（从交易费中分配给资金所有者的部分）
  - `create_pool_fee`：创建池子的费用

- 账户
  - `owner`：调用者，必须是管理员（`crate::admin::id()`）
  - `amm_config`：新创建的配置账户，使用种子（`AMM_CONFIG_SEED` 和 `index`）生成
  - `system_program`：用于支付账户创建费用

- 逻辑
  1. 验证调用者是管理员
  2. 初始化 `amm_config` 账户，设置以下字段：
    - `protocol_owner` 和 `fund_owner` 设置为调用者
    - `trade_fee_rate`, `protocol_fee_rate`, `fund_fee_rate`, `create_pool_fee` 设置为输入值
    - `disable_create_pool` 默认 false
    - `bump` 和 `index` 记录


### `collect_protocol_fee`（收取流动池中累积的协议费）

src/instructions/lib.rs：
```rust
pub fn collect_protocol_fee(
    ctx: Context<CollectProtocolFee>,
    amount_0_requested: u64,
    amount_1_requested: u64,
) -> Result<()> {
    instructions::collect_protocol_fee(ctx, amount_0_requested, amount_1_requested)
}
```

src/instructions/admin/collect_protocol_fee.rs：
```rust
#[derive(Accounts)]
pub struct CollectProtocolFee<'info> {
    /// 费用收集者：必须是协议所有者或预定义管理员
    #[account(
        constraint = (owner.key() == amm_config.protocol_owner || owner.key() == crate::admin::ID) 
        @ ErrorCode::InvalidOwner // 权限验证失败时返回错误
    )]
    pub owner: Signer<'info>, // 签名者账户（必须签名）

    /// 池权限账户（PDA）：用于签名代币转账
    /// CHECK: 安全性由种子和 bump 保证
    #[account(
        seeds = [crate::AUTH_SEED.as_bytes()], // 固定种子
        bump, // 自动计算 bump 值
    )]
    pub authority: UncheckedAccount<'info>, // 未检查账户（PDA）

    /// 池状态账户（可变）：存储累积的协议费用
    #[account(mut)] // 标记为可变
    pub pool_state: AccountLoader<'info, PoolState>, // 账户加载器（用于加载状态）

    /// AMM 配置账户：存储协议所有者信息
    #[account(address = pool_state.load()?.amm_config)] // 验证地址匹配池状态中的配置
    pub amm_config: Account<'info, AmmConfig>, // 配置账户

    /// token_0 金库账户（可变）：存放 token_0 的池资金
    #[account(
        mut, // 可变
        constraint = token_0_vault.key() == pool_state.load()?.token_0_vault // 验证地址匹配池状态
    )]
    pub token_0_vault: Box<InterfaceAccount<'info, TokenAccount>>, // 代币账户接口

    /// token_1 金库账户（可变）：存放 token_1 的池资金
    #[account(
        mut, // 可变
        constraint = token_1_vault.key() == pool_state.load()?.token_1_vault // 验证地址匹配池状态
    )]
    pub token_1_vault: Box<InterfaceAccount<'info, TokenAccount>>, // 代币账户接口

    ///  token_0 铸币账户：定义 token_0 的属性
    #[account(address = token_0_vault.mint)] // 验证地址匹配金库的铸币地址
    pub vault_0_mint: Box<InterfaceAccount<'info, Mint>>, // 铸币账户接口
    ///  token_1 铸币账户：定义 token_1 的属性
    #[account(address = token_1_vault.mint)] // 验证地址匹配金库的铸币地址
    pub vault_1_mint: Box<InterfaceAccount<'info, Mint>>, // 铸币账户接口
    ///  token_0 接收账户（可变）：接收收集的 token_0 协议费用
    #[account(mut)] // 可变
    pub recipient_token_0_account: Box<InterfaceAccount<'info, TokenAccount>>, // 代币账户接口
    ///  token_1 接收账户（可变）：接收收集的 token_1 协议费用
    #[account(mut)] // 可变
    pub recipient_token_1_account: Box<InterfaceAccount<'info, TokenAccount>>, // 代币账户接口
    /// 标准 SPL 代币程序：用于标准代币转账
    pub token_program: Program<'info, Token>, // 代币程序
    /// Token2022 程序：用于扩展代币转账
    pub token_program_2022: Program<'info, Token2022>, // Token2022 程序
}

/// 收集协议费用的核心函数
pub fn collect_protocol_fee(
    ctx: Context<CollectProtocolFee>, // 账户上下文
    amount_0_requested: u64, // 请求收集的 token_0 数量
    amount_1_requested: u64, // 请求收集的 token_1 数量
) -> Result<()> {
    // 声明局部变量：实际提取数量和权限 bump 值
    let amount_0: u64;
    let amount_1: u64;
    let auth_bump: u8;
    // 使用独立作用域限制池状态可变借用的生命周期
    {
        // 加载并获取池状态的可变引用
        let mut pool_state = ctx.accounts.pool_state.load_mut()?;
        // 计算实际可提取的 token_0 数量（不超过请求量和可用量）
        amount_0 = amount_0_requested.min(pool_state.protocol_fees_token_0);
        // 计算实际可提取的 token_1 数量（不超过请求量和可用量）
        amount_1 = amount_1_requested.min(pool_state.protocol_fees_token_1);
        // 更新池状态中的 token_0 协议费用余额（安全减法）
        pool_state.protocol_fees_token_0 = pool_state
            .protocol_fees_token_0
            .checked_sub(amount_0)
            .unwrap(); // 不会 panic 因为前面 min 保证了不会下溢
        // 更新池状态中的 token_1 协议费用余额（安全减法）
        pool_state.protocol_fees_token_1 = pool_state
            .protocol_fees_token_1
            .checked_sub(amount_1)
            .unwrap(); // 不会 panic 因为前面 min 保证了不会下溢
        // 获取权限 bump 值（用于后续签名）
        auth_bump = pool_state.auth_bump;
        // 更新最近操作 epoch（记录操作时间）
        pool_state.recent_epoch = Clock::get()?.epoch;
    } // 池状态可变借用在此结束
    
    // 执行 token_0 转账：从池金库到接收账户
    transfer_from_pool_vault_to_user(
        ctx.accounts.authority.to_account_info(), // 权限账户（PDA）
        ctx.accounts.token_0_vault.to_account_info(), // 来源金库账户
        ctx.accounts.recipient_token_0_account.to_account_info(), // 目标接收账户
        ctx.accounts.vault_0_mint.to_account_info(), // token_0 铸币账户
        // 动态选择代币程序：根据铸币账户所有者判断
        if ctx.accounts.vault_0_mint.to_account_info().owner == ctx.accounts.token_program.key {
            ctx.accounts.token_program.to_account_info() // 标准代币程序
        } else {
            ctx.accounts.token_program_2022.to_account_info() // Token2022 程序
        },
        amount_0, // 转账数量
        ctx.accounts.vault_0_mint.decimals, // 代币精度（小数位数）
        &[&[crate::AUTH_SEED.as_bytes(), &[auth_bump]]], // PDA签名种子
    )?;
    
    // 执行 token_1 转账：从池金库到接收账户
    transfer_from_pool_vault_to_user(
        ctx.accounts.authority.to_account_info(), // 权限账户（PDA）
        ctx.accounts.token_1_vault.to_account_info(), // 来源金库账户
        ctx.accounts.recipient_token_1_account.to_account_info(), // 目标接收账户
        ctx.accounts.vault_1_mint.to_account_info(), // token_1 铸币账户
        // 动态选择代币程序：根据铸币账户所有者判断
        if ctx.accounts.vault_1_mint.to_account_info().owner == ctx.accounts.token_program.key {
            ctx.accounts.token_program.to_account_info() // 标准代币程序
        } else {
            ctx.accounts.token_program_2022.to_account_info() // Token2022 程序
        },
        amount_1, // 转账数量
        ctx.accounts.vault_1_mint.decimals, // 代币精度（小数位数）
        &[&[crate::AUTH_SEED.as_bytes(), &[auth_bump]]], // PDA 签名种子
    )?;

    Ok(())
}
```

#### 作用

提取流动性池中累积的协议费用（`protocol_fees_token_0` 和 `protocol_fees_token_1`），由协议所有者或管理员调用

#### 工作内容

- 输入参数
  - `amount_0_requested`: 希望提取的 token_0 最大数量
  - `amount_1_requested`: 希望提取的 token_1 最大数量

- 账户
  - `owner`：调用者，必须是 `amm_config.protocol_owner` 或管理员
  - `authority`：池子的权限账户（种子生成）
  - `pool_state`：池子状态，记录累积费用
  - `amm_config`：配置账户，用于验证所有者
  - `token_0_vault` 和 `token_1_vault`：池子的代币储备
  - `recipient_token_0_account` 和 `recipient_token_1_account`：接收费用的账户
  - `token_program` 和 `token_program_2022`：代币转账程序

- 逻辑
  1. 验证调用者权限
  2. 从 `pool_state` 加载费用数据
    - `amount_0 = min(amount_0_requested, pool_state.protocol_fees_token_0)`
    - `amount_1 = min(amount_1_requested, pool_state.protocol_fees_token_1)`
  - 更新 `pool_state`
    - 减去提取的费用
    - 更新 `recent_epoch`
  - 执行转账
    - 从 `token_0_vault` 和 `token_1_vault` 转账到接收账户，使用 `authority` 签名

#### 原理

- 协议费用来源于交易手续费的一部分（由 `protocol_fee_rate` 决定），累积在池子中
- 提取时确保不超过实际累积量，防止超额提取


### `collect_fund_fee`（收取流动池中累积的基金费）

代码与 `collect_protocol_fee` 相近，只是将 `protocol` 换为 `fund`

#### 作用

提取流动性池中累积的资金费用（`fund_fees_token_0` 和 `fund_fees_token_1`），由资金所有者或管理员调用

#### 工作内容

- 输入参数、账户、逻辑
  - 与 `collect_protocol_fee` 相同，只是权限检查基于 `amm_config.fund_owner`

#### 原理、逻辑

- 资金费用是交易费的另一部分（由 `fund_fee_rate` 决定），与协议费用分离，用于不同的受益人。
- 实现逻辑与 `collect_protocol_fee` 几乎相同，仅权限和字段不同

### `update_amm_config`（更新 AMM 配置）

src/instructions/lib.rs：
```rust
pub fn update_amm_config(ctx: Context<UpdateAmmConfig>, param: u8, value: u64) -> Result<()> {
    instructions::update_amm_config(ctx, param, value)
}
```

src/instructions/admin/update_config.rs：
```rust
/// 更新 AMM 配置的账户结构
#[derive(Accounts)]
pub struct UpdateAmmConfig<'info> {
    /// 配置更新权限账户：必须是预定义管理员
    #[account(
        address = crate::admin::ID @ ErrorCode::InvalidOwner // 验证调用者地址是否为管理员
    )]
    pub owner: Signer<'info>, // 签名者账户（必须签名）
    
    /// 待更新的 AMM 配置账户（可变）
    #[account(mut)] // 标记为可变，因为配置将被修改
    pub amm_config: Account<'info, AmmConfig>, // AMM 配置账户
}

/// 更新 AMM 配置的核心函数
pub fn update_amm_config(
    ctx: Context<UpdateAmmConfig>, // 账户上下文
    param: u8,                     // 参数类型标识
    value: u64,                    // 新值
) -> Result<()> {
    // 获取 AMM 配置账户的可变引用
    let amm_config = &mut ctx.accounts.amm_config;
    // 根据参数类型执行不同的更新操作
    match param {
        // 0: 更新交易费率
        0 => update_trade_fee_rate(amm_config, value),
        // 1: 更新协议费率
        1 => update_protocol_fee_rate(amm_config, value),
        // 2: 更新基金费率
        2 => update_fund_fee_rate(amm_config, value),
        // 3: 更新协议所有者
        3 => {
            // 从剩余账户中获取新所有者地址
            let new_protocol_owner = *ctx.remaining_accounts.iter().next().unwrap().key;
            // 调用设置新协议所有者函数
            set_new_protocol_owner(amm_config, new_protocol_owner)?;
        }
        // 4: 更新基金所有者
        4 => {
            // 从剩余账户中获取新基金所有者地址
            let new_fund_owner = *ctx.remaining_accounts.iter().next().unwrap().key;
            // 调用设置新基金所有者函数
            set_new_fund_owner(amm_config, new_fund_owner)?;
        }
        // 5: 更新创建池费用
        5 => amm_config.create_pool_fee = value,
        // 6: 更新禁用创建池标志
        6 => amm_config.disable_create_pool = if value == 0 { false } else { true },
        // 其他值：返回无效输入错误
        _ => return err!(ErrorCode::InvalidInput),
    }

    // 返回成功
    Ok(())
}

/// 更新协议费率的内部函数
fn update_protocol_fee_rate(
    amm_config: &mut Account<AmmConfig>, // 可变的 AMM 配置引用
    protocol_fee_rate: u64,              // 新的协议费率值
) {
    // 验证：协议费率不能超过分母值
    assert!(protocol_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    // 验证：协议费率 + 基金费率不能超过分母值（总和不能超过 100%）
    assert!(protocol_fee_rate + amm_config.fund_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    // 更新配置中的协议费率
    amm_config.protocol_fee_rate = protocol_fee_rate;
}

/// 更新交易费率的内部函数
fn update_trade_fee_rate(
    amm_config: &mut Account<AmmConfig>, // 可变的 AMM 配置引用
    trade_fee_rate: u64,                 // 新的交易费率值
) {
    // 验证：交易费率必须小于分母值（不能大于等于 100%）
    assert!(trade_fee_rate < FEE_RATE_DENOMINATOR_VALUE);
    // 更新配置中的交易费率
    amm_config.trade_fee_rate = trade_fee_rate;
}

/// 更新基金费率的内部函数
fn update_fund_fee_rate(
    amm_config: &mut Account<AmmConfig>, // 可变的 AMM 配置引用
    fund_fee_rate: u64,                  // 新的基金费率值
) {
    // 验证：基金费率不能超过分母值
    assert!(fund_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    // 验证：基金费率 + 协议费率不能超过分母值（总和不能超过 100%）
    assert!(fund_fee_rate + amm_config.protocol_fee_rate <= FEE_RATE_DENOMINATOR_VALUE);
    // 更新配置中的基金费率
    amm_config.fund_fee_rate = fund_fee_rate;
}

/// 设置新协议所有者的内部函数
fn set_new_protocol_owner(
    amm_config: &mut Account<AmmConfig>, // 可变的 AMM 配置引用
    new_owner: Pubkey,                   // 新的协议所有者公钥
) -> Result<()> {
    // 验证：新所有者地址不能是默认零地址
    require_keys_neq!(new_owner, Pubkey::default());
    // 条件编译：如果启用了日志功能，记录所有者变更
    #[cfg(feature = "enable-log")]
    msg!(
        "AMM配置更新：旧协议所有者={}，新协议所有者={}",
        amm_config.protocol_owner.to_string(), // 旧所有者
        new_owner.to_string()                  // 新所有者
    );
    // 更新配置中的协议所有者
    amm_config.protocol_owner = new_owner;
    // 返回成功
    Ok(())
}

/// 设置新基金所有者的内部函数
fn set_new_fund_owner(
    amm_config: &mut Account<AmmConfig>, // 可变的 AMM 配置引用
    new_fund_owner: Pubkey,              // 新的基金所有者公钥
) -> Result<()> {
    // 验证：新基金所有者地址不能是默认零地址
    require_keys_neq!(new_fund_owner, Pubkey::default());
    // 条件编译：如果启用了日志功能，记录基金所有者变更
    #[cfg(feature = "enable-log")]
    msg!(
        "AMM 配置更新：旧基金所有者={}，新基金所有者={}",
        amm_config.fund_owner.to_string(), // 旧基金所有者
        new_fund_owner.to_string()         // 新基金所有者
    );
    // 更新配置中的基金所有者
    amm_config.fund_owner = new_fund_owner;
    // 返回成功
    Ok(())
}
```

#### 作用

更新 AMM 配置的某个参数（例如费率或所有者），由管理员调用

#### 工作内容

- 输入参数
  - param：要更新的参数（0: 交易费率，1: 协议费率，2: 资金费率，3: 新协议所有者，4: 新资金所有者，5: 创建池子费用，6: 是否禁用创建池子）
  - value：新值

- 账户
  - owner：必须是管理员
  - amm_config：要修改的配置账户

- 逻辑
  1. 验证调用者是管理员。
  2. 根据 param 执行更新
    - `0`：更新 `trade_fee_rate`，确保小于 `FEE_RATE_DENOMINATOR_VALUE`
    - `1`：更新 `protocol_fee_rate`，确保总和不超过 100%
    - `2`：更新 `fund_fee_rate`，同上
    - `3`：设置新 `protocol_owner`（从 `remaining_accounts` 获取）
    - `4`：设置新 `fund_owner`
    - `5`：更新 `create_pool_fee`
    - `6`：设置 `disable_create_pool`（0 为 false，其他为 true）

#### 原理

- 提供动态调整配置的能力，确保系统灵活性
- 费率检查防止配置错误（如超额分配）

### `update_pool_status`（更新流动池的状态）

src/instructions/lib.rs：
```rust
pub fn update_pool_status(ctx: Context<UpdatePoolStatus>, status: u8) -> Result<()> {
    instructions::update_pool_status(ctx, status)
}
```

src/instructions/admin/update_pool_status.rs：
```rust
/// 更新池状态的账户结构
#[derive(Accounts)]
pub struct UpdatePoolStatus<'info> {
    /// 权限账户：必须是预定义管理员才能更新池状态
    #[account(
        address = crate::admin::ID // 严格限制只有管理员可以调用
    )]
    pub authority: Signer<'info>, // 签名者账户（必须签名）
    /// 池状态账户（可变）：需要更新的池状态
    #[account(mut)] // 标记为可变
    pub pool_state: AccountLoader<'info, PoolState>, // 池状态账户加载器
}

/// 更新池状态的核心函数
pub fn update_pool_status(
    ctx: Context<UpdatePoolStatus>, // 账户上下文
    status: u8,                     // 新的池状态值
) -> Result<()> {
    // 验证状态值有效性：必须 ≤ 255（u8 最大值）
    require_gte!(255, status); // 等价于 require!(status <= 255)
    // 获取池状态的可变引用
    let mut pool_state = ctx.accounts.pool_state.load_mut()?;
    // 设置新的池状态
    pool_state.set_status(status);
    // 更新最近操作 epoch（记录状态变更时间）
    pool_state.recent_epoch = Clock::get()?.epoch;
    // 返回成功
    Ok(())
}
```

#### 作用

更新流动性池的状态（例如启用/禁用存款、交易等），由管理员调用

#### 工作内容

- 输入参数
  - `status`：新状态值（8位掩码，0-255）

- 账户
  - `authority`：必须是管理员
  - `pool_state`：要修改的池子状态

- 逻辑
  1. 验证调用者是管理员
  2. 更新 `pool_state.status` 为输入值
  3. 更新 `recent_epoch`

#### 原理

- 状态通过位掩码控制多种功能（例如存款、取款、交易）
- 管理员可以动态启用/禁用池子操作
