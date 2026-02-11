---
title: IB-Solana-Defi项目拆解4
date: 2025-07-09 12:12:11
tags:
    - Solana
categories:
    - Solana
      - IB课程
---

[Codes in lesson7](https://github.com/Zoella-w/IB-Solana/tree/main/7_swap)

## User 命令详解（ `swap_base_input` & `swap_base_output`）

### `swap_base_input`（基于输入数量的代币交换）

src/instructions/lib.rs：
```rust
pub fn swap_base_input(
    ctx: Context<Swap>,
    amount_in: u64,
    minimum_amount_out: u64,
) -> Result<()> {
    instructions::swap_base_input(ctx, amount_in, minimum_amount_out)
}
```

src/instructions/swap_base_input.rs：
```rust
/// 交换操作的账户结构
#[derive(Accounts)]
pub struct Swap<'info> {
    /// 执行交换的用户（支付者）
    pub payer: Signer<'info>,

    /// 池权限账户（PDA），用于签名代币操作
    /// CHECK: 安全性由种子和 bump 保证
    #[account(
        seeds = [crate::AUTH_SEED.as_bytes()],
        bump,
    )]
    pub authority: UncheckedAccount<'info>,

    /// AMM 配置账户，读取协议费用参数
    #[account(address = pool_state.load()?.amm_config)]
    pub amm_config: Box<Account<'info, AmmConfig>>,

    /// 池状态账户（可变）
    #[account(mut)]
    pub pool_state: AccountLoader<'info, PoolState>,

    /// 用户的输入代币账户（可变）
    #[account(mut)]
    pub input_token_account: Box<InterfaceAccount<'info, TokenAccount>>,

    /// 用户的输出代币账户（可变）
    #[account(mut)]
    pub output_token_account: Box<InterfaceAccount<'info, TokenAccount>>,

    /// 输入代币金库账户（可变），必须是池的两个金库之一
    #[account(
        mut,
        constraint = input_vault.key() == pool_state.load()?.token_0_vault || 
                    input_vault.key() == pool_state.load()?.token_1_vault
    )]
    pub input_vault: Box<InterfaceAccount<'info, TokenAccount>>,

    /// 输出代币金库账户（可变），必须是池的两个金库之一
    #[account(
        mut,
        constraint = output_vault.key() == pool_state.load()?.token_0_vault || 
                    output_vault.key() == pool_state.load()?.token_1_vault
    )]
    pub output_vault: Box<InterfaceAccount<'info, TokenAccount>>,

    /// 输入代币程序（SPL Token 或 Token2022）
    pub input_token_program: Interface<'info, TokenInterface>,

    /// 输出代币程序（SPL Token 或 Token2022）
    pub output_token_program: Interface<'info, TokenInterface>,

    /// 输入代币铸币账户
    #[account(address = input_vault.mint)]
    pub input_token_mint: Box<InterfaceAccount<'info, Mint>>,

    /// 输出代币铸币账户
    #[account(address = output_vault.mint)]
    pub output_token_mint: Box<InterfaceAccount<'info, Mint>>,

    /// 预言机观测状态账户（可变）
    #[account(mut, address = pool_state.load()?.observation_key)]
    pub observation_state: AccountLoader<'info, ObservationState>,
}

/// 执行固定输入量交换的核心函数
pub fn swap_base_input(
    ctx: Context<Swap>, // 账户上下文
    amount_in: u64,      // 用户输入的代币数量
    minimum_amount_out: u64 // 用户可接受的最小输出量
) -> Result<()> {
    // 1. 获取当前时间戳
    let block_timestamp = solana_program::clock::Clock::get()?.unix_timestamp as u64;
    let pool_id = ctx.accounts.pool_state.key();
    // 2. 加载池状态
    let pool_state = &mut ctx.accounts.pool_state.load_mut()?;
    // 3. 验证交换条件
    if !pool_state.get_status_by_bit(PoolStatusBitIndex::Swap) || // 检查交换功能是否启用
       block_timestamp < pool_state.open_time // 检查池是否已开放
    {
        return err!(ErrorCode::NotApproved);
    }
    // 4. 计算输入代币转账费用
    let transfer_fee = get_transfer_fee(
        &ctx.accounts.input_token_mint.to_account_info(),
        amount_in
    )?;
    // 5. 计算实际进入池的输入量（扣除转账费）
    let actual_amount_in = amount_in.saturating_sub(transfer_fee);
    require_gt!(actual_amount_in, 0); // 确保实际输入量大于 0
    // 6. 确定交易方向及获取池状态数据
    let (
        trade_direction,               // 交易方向 (ZeroForOne 或 OneForZero)
        total_input_token_amount,       // 输入代币池总量
        total_output_token_amount,      // 输出代币池总量
        token_0_price_x64,              // token_0 价格
        token_1_price_x64               // token_1 价格
    ) =
    // 情况 1：用 token_0 换 token_1 (ZeroForOne)
    if ctx.accounts.input_vault.key() == pool_state.token_0_vault &&
           ctx.accounts.output_vault.key() == pool_state.token_1_vault 
    {
        let (input_amount, output_amount) = pool_state.vault_amount_without_fee(
            ctx.accounts.input_vault.amount,
            ctx.accounts.output_vault.amount
        );
        let (token0_price, token1_price) = pool_state.token_price_x32(
            ctx.accounts.input_vault.amount,
            ctx.accounts.output_vault.amount
        );
        (
            TradeDirection::ZeroForOne,
            input_amount,
            output_amount,
            token0_price,
            token1_price
        )
    }
    // 情况 2：用 token_1 换 token_0 (OneForZero)
    else if ctx.accounts.input_vault.key() == pool_state.token_1_vault &&
              ctx.accounts.output_vault.key() == pool_state.token_0_vault 
    {
        let (output_amount, input_amount) = pool_state.vault_amount_without_fee(
            ctx.accounts.output_vault.amount,
            ctx.accounts.input_vault.amount
        );
        let (token0_price, token1_price) = pool_state.token_price_x32(
            ctx.accounts.output_vault.amount,
            ctx.accounts.input_vault.amount
        );
        (
            TradeDirection::OneForZero,
            input_amount,
            output_amount,
            token0_price,
            token1_price
        )
    }
    // 无效的金库组合
    else {
        return err!(ErrorCode::InvalidVault);
    };
    // 7. 计算交换前的常数乘积（恒定乘积模型）
    let constant_before = u128::from(total_input_token_amount)
        .checked_mul(u128::from(total_output_token_amount))
        .unwrap();
    // 8. 调用曲线计算器计算交换结果
    let result = CurveCalculator::swap_base_input(
        u128::from(actual_amount_in), // 实际输入量
        u128::from(total_input_token_amount), // 输入代币池总量
        u128::from(total_output_token_amount), // 输出代币池总量
        ctx.accounts.amm_config.trade_fee_rate, // 交易费率
        ctx.accounts.amm_config.protocol_fee_rate, // 协议费率
        ctx.accounts.amm_config.fund_fee_rate // 基金费率
    ).ok_or(ErrorCode::ZeroTradingTokens)?; // 处理可能的错误
    // 9. 计算交换后的常数乘积
    let constant_after = u128::from(
        result.new_swap_source_amount.checked_sub(result.trade_fee).unwrap()
    ).checked_mul(u128::from(result.new_swap_destination_amount))
    .unwrap();
    // 10. 验证实际输入量与计算结果一致
    require_eq!(
        u64::try_from(result.source_amount_swapped).unwrap(),
        actual_amount_in
    );
    // 11. 计算输出代币转账费用
    let (input_transfer_amount, input_transfer_fee) = (amount_in, transfer_fee);
    let (output_transfer_amount, output_transfer_fee) = {
        let amount_out = u64::try_from(result.destination_amount_swapped).unwrap();
        let transfer_fee = get_transfer_fee(
            &ctx.accounts.output_token_mint.to_account_info(),
            amount_out
        )?;
        let amount_received = amount_out.checked_sub(transfer_fee).unwrap();
        // 滑点保护：确保实际收到量不低于用户设定的最小值
        require_gt!(amount_received, 0);
        require_gte!(amount_received, minimum_amount_out, ErrorCode::ExceededSlippage);
        (amount_out, transfer_fee)
    };
    // 12. 处理协议费用和基金费用
    let protocol_fee = u64::try_from(result.protocol_fee).unwrap();
    let fund_fee = u64::try_from(result.fund_fee).unwrap();
    match trade_direction {
        TradeDirection::ZeroForOne => {
            // 方向为 token_0 -> token1，费用计入 token_0
            pool_state.protocol_fees_token_0 = pool_state.protocol_fees_token_0.checked_add(protocol_fee).unwrap();
            pool_state.fund_fees_token_0 = pool_state.fund_fees_token_0.checked_add(fund_fee).unwrap();
        }
        TradeDirection::OneForZero => {
            // 方向为 token_1 -> token0，费用计入 token_1
            pool_state.protocol_fees_token_1 = pool_state.protocol_fees_token_1.checked_add(protocol_fee).unwrap();
            pool_state.fund_fees_token_1 = pool_state.fund_fees_token_1.checked_add(fund_fee).unwrap();
        }
    };
    // 13. 记录交换事件
    emit!(SwapEvent {
        pool_id,
        input_vault_before: total_input_token_amount,
        output_vault_before: total_output_token_amount,
        input_amount: u64::try_from(result.source_amount_swapped).unwrap(),
        output_amount: u64::try_from(result.destination_amount_swapped).unwrap(),
        input_transfer_fee,
        output_transfer_fee,
        base_input: true
    });
    // 14. 验证常数乘积模型（应保持不变或增加）
    require_gte!(constant_after, constant_before);
    // 15. 执行代币转账
    // 15.1 用户输入代币转移到池金库
    transfer_from_user_to_pool_vault(
        ctx.accounts.payer.to_account_info(),
        ctx.accounts.input_token_account.to_account_info(),
        ctx.accounts.input_vault.to_account_info(),
        ctx.accounts.input_token_mint.to_account_info(),
        ctx.accounts.input_token_program.to_account_info(),
        input_transfer_amount,
        ctx.accounts.input_token_mint.decimals,
    )?;
    // 15.2 池输出代币转移到用户账户
    transfer_from_pool_vault_to_user(
        ctx.accounts.authority.to_account_info(),
        ctx.accounts.output_vault.to_account_info(),
        ctx.accounts.output_token_account.to_account_info(),
        ctx.accounts.output_token_mint.to_account_info(),
        ctx.accounts.output_token_program.to_account_info(),
        output_transfer_amount,
        ctx.accounts.output_token_mint.decimals,
        &[&[crate::AUTH_SEED.as_bytes(), &[pool_state.auth_bump]]], // PDA 签名
    )?;
    // 16. 更新预言机状态
    ctx.accounts.observation_state.load_mut()?.update(
        oracle::block_timestamp(), // 当前时间戳
        token_0_price_x64, // 交换前的 token_0 价格
        token_1_price_x64  // 交换前的 token_1 价格
    );
    // 17. 更新池的最后操作 epoch
    pool_state.recent_epoch = Clock::get()?.epoch;
    Ok(())
}
```

#### 作用

基于输入数量的代币交换

#### 工作内容

- 输入参数
  - `amount_in`：输入代币数量
  - `minimum_amount_out`：最少输出代币数量（滑点保护）

- 账户
  - `payer`：调用者
  - `amm_config`：配置
  - `pool_state`：池子状态
  - `input_token_account` 和 `output_token_account`：用户的输入和输出账户
  - `input_vault` 和 `output_vault`：池子的输入和输出储备

- 逻辑
  1. 检查池子允许交易且开放时间已到
  2. 计算实际输入量（扣除转账费）
  3. 确定交易方向并计算储备和价格
  4. 用 `CurveCalculator::swap_base_input` 计算交易结果（含手续费）
  5. 检查滑点和常量乘积（k 不减少）
  6. 更新协议和资金费用
  7. 转入输入代币，转出输出代币
  8. 更新预言机和 `recent_epoch`

#### 原理

- 基于 x * y = k，输入增加 x，输出减少 y，保持 k 不变
- 手续费分为交易费、协议费和资金费

### `swap_base_output`（基于输出数量的代币交换）

src/instructions/lib.rs：
```rust
pub fn swap_base_output(ctx: Context<Swap>, max_amount_in: u64, amount_out: u64) -> Result<()> {
    instructions::swap_base_output(ctx, max_amount_in, amount_out)
}
```

src/instructions/swap_base_output.rs：
```rust
use crate::curve::calculator::CurveCalculator; // 曲线计算器核心
use crate::curve::TradeDirection; // 交易方向枚举
use crate::error::ErrorCode; // 自定义错误码
use crate::states::*; // 状态结构体
use crate::utils::token::*; // 代币工具函数
use anchor_lang::prelude::*; // Anchor 预导入
use anchor_lang::solana_program; // Solana 程序基础
use anchor_spl::token_interface::{Mint, TokenAccount, TokenInterface}; // 代币接口

pub fn swap_base_output(
    ctx: Context<Swap>,       // 账户上下文
    max_amount_in: u64,       // 最大可接受输入量（滑点保护）
    amount_out_less_fee: u64, // 期望接收的输出量（不含转账费）
) -> Result<()> {
    // 1. 基础验证：输出量必须大于 0
    require_gt!(amount_out_less_fee, 0);
    // 2. 获取当前时间戳并验证池状态
    let block_timestamp = solana_program::clock::Clock::get()?.unix_timestamp as u64;
    let pool_id = ctx.accounts.pool_state.key();
    let pool_state = &mut ctx.accounts.pool_state.load_mut()?;
    // 检查池是否允许交换且已开放
    if !pool_state.get_status_by_bit(PoolStatusBitIndex::Swap)
        || block_timestamp < pool_state.open_time
    {
        return err!(ErrorCode::NotApproved);
    }
    // 3. 计算输出代币的转账费用
    let out_transfer_fee = get_transfer_inverse_fee(
        &ctx.accounts.output_token_mint.to_account_info(),
        amount_out_less_fee,
    )?;
    // 实际从池中提取的输出量 = 期望接收量 + 转账费
    let actual_amount_out = amount_out_less_fee.checked_add(out_transfer_fee).unwrap();
    // 4. 确定交易方向并获取池状态
    let (
        trade_direction,           // 交易方向
        total_input_token_amount,  // 输入代币总量
        total_output_token_amount, // 输出代币总量
        token_0_price_x64,         // token_0 价格
        token_1_price_x64,         // token_1 价格
    ) = 
    // 情况 1: 用 token_0 换 token_1 (ZeroForOne)
    if ctx.accounts.input_vault.key() == pool_state.token_0_vault
        && ctx.accounts.output_vault.key() == pool_state.token_1_vault
    {
        let (input_amount, output_amount) = pool_state.vault_amount_without_fee(
            ctx.accounts.input_vault.amount,
            ctx.accounts.output_vault.amount,
        );
        let (token0_price, token1_price) = pool_state.token_price_x32(
            ctx.accounts.input_vault.amount,
            ctx.accounts.output_vault.amount,
        );
        (
            TradeDirection::ZeroForOne,
            input_amount,
            output_amount,
            token0_price,
            token1_price,
        )
    }
    // 情况 2: 用 token_1 换 token_0 (OneForZero)
    else if ctx.accounts.input_vault.key() == pool_state.token_1_vault
        && ctx.accounts.output_vault.key() == pool_state.token_0_vault
    {
        let (output_amount, input_amount) = pool_state.vault_amount_without_fee(
            ctx.accounts.output_vault.amount,
            ctx.accounts.input_vault.amount,
        );
        let (token0_price, token1_price) = pool_state.token_price_x32(
            ctx.accounts.output_vault.amount,
            ctx.accounts.input_vault.amount,
        );
        (
            TradeDirection::OneForZero,
            input_amount,
            output_amount,
            token0_price,
            token1_price,
        )
    } else {
        // 无效的金库组合
        return err!(ErrorCode::InvalidVault);
    };
    // 5. 计算交换前的常数乘积
    let constant_before = u128::from(total_input_token_amount)
        .checked_mul(u128::from(total_output_token_amount))
        .unwrap();
    // 6. 调用曲线计算器计算交换结果
    let result = CurveCalculator::swap_base_output(
        u128::from(actual_amount_out),             // 实际需要的输出量
        u128::from(total_input_token_amount),      // 输入代币总量
        u128::from(total_output_token_amount),     // 输出代币总量
        ctx.accounts.amm_config.trade_fee_rate,    // 交易费率
        ctx.accounts.amm_config.protocol_fee_rate, // 协议费率
        ctx.accounts.amm_config.fund_fee_rate,     // 基金费率
    )
    .ok_or(ErrorCode::ZeroTradingTokens)?;
    // 7. 计算交换后的常数乘积
    let constant_after = u128::from(
        result
            .new_swap_source_amount
            .checked_sub(result.trade_fee)
            .unwrap(),
    )
    .checked_mul(u128::from(result.new_swap_destination_amount))
    .unwrap();
    // 8. 日志记录（调试用）
    #[cfg(feature = "enable-log")]
    msg!(
        "source_amount_swapped:{}, destination_amount_swapped:{}, trade_fee:{}, constant_before:{},constant_after:{}",
        result.source_amount_swapped,
        result.destination_amount_swapped,
        result.trade_fee,
        constant_before,
        constant_after
    );
    // 9. 计算输入代币的转账费用并验证滑点
    let (input_transfer_amount, input_transfer_fee) = {
        // 计算实际需要的输入量
        let source_amount_swapped = u64::try_from(result.source_amount_swapped).unwrap();
        require_gt!(source_amount_swapped, 0);
        // 计算输入代币的转账费用
        let transfer_fee = get_transfer_inverse_fee(
            &ctx.accounts.input_token_mint.to_account_info(),
            source_amount_swapped,
        )?;
        // 用户需支付的总输入量 = 实际输入量 + 转账费
        let input_transfer_amount = source_amount_swapped.checked_add(transfer_fee).unwrap();
        // 滑点保护：确保总输入量不超过用户设置的最大值
        require_gte!(
            max_amount_in,
            input_transfer_amount,
            ErrorCode::ExceededSlippage
        );
        (input_transfer_amount, transfer_fee)
    };
    // 10. 验证输出量计算结果
    require_eq!(
        u64::try_from(result.destination_amount_swapped).unwrap(),
        actual_amount_out
    );
    // 11. 准备输出代币转账数据
    let (output_transfer_amount, output_transfer_fee) = (actual_amount_out, out_transfer_fee);
    // 12. 处理协议费用和基金费用
    let protocol_fee = u64::try_from(result.protocol_fee).unwrap();
    let fund_fee = u64::try_from(result.fund_fee).unwrap();
    match trade_direction {
        TradeDirection::ZeroForOne => {
            // token_0 -> token_1 方向，费用计入 token_0
            pool_state.protocol_fees_token_0 = pool_state
                .protocol_fees_token_0
                .checked_add(protocol_fee)
                .unwrap();
            pool_state.fund_fees_token_0 =
                pool_state.fund_fees_token_0.checked_add(fund_fee).unwrap();
        }
        TradeDirection::OneForZero => {
            // token_1 -> token_0 方向，费用计入 token_1
            pool_state.protocol_fees_token_1 = pool_state
                .protocol_fees_token_1
                .checked_add(protocol_fee)
                .unwrap();
            pool_state.fund_fees_token_1 =
                pool_state.fund_fees_token_1.checked_add(fund_fee).unwrap();
        }
    };
    // 13. 记录交换事件
    emit!(SwapEvent {
        pool_id,
        input_vault_before: total_input_token_amount,
        output_vault_before: total_output_token_amount,
        input_amount: u64::try_from(result.source_amount_swapped).unwrap(),
        output_amount: u64::try_from(result.destination_amount_swapped).unwrap(),
        input_transfer_fee,
        output_transfer_fee,
        base_input: false // 标记为固定输出量交换
    });
    // 14. 验证常数乘积模型（应保持不变或增加）
    require_gte!(constant_after, constant_before);
    // 15. 执行代币转账
    // 15.1 用户输入代币转移到池金库
    transfer_from_user_to_pool_vault(
        ctx.accounts.payer.to_account_info(),
        ctx.accounts.input_token_account.to_account_info(),
        ctx.accounts.input_vault.to_account_info(),
        ctx.accounts.input_token_mint.to_account_info(),
        ctx.accounts.input_token_program.to_account_info(),
        input_transfer_amount,
        ctx.accounts.input_token_mint.decimals,
    )?;
    // 15.2 池输出代币转移到用户账户
    transfer_from_pool_vault_to_user(
        ctx.accounts.authority.to_account_info(),
        ctx.accounts.output_vault.to_account_info(),
        ctx.accounts.output_token_account.to_account_info(),
        ctx.accounts.output_token_mint.to_account_info(),
        ctx.accounts.output_token_program.to_account_info(),
        output_transfer_amount,
        ctx.accounts.output_token_mint.decimals,
        &[&[crate::AUTH_SEED.as_bytes(), &[pool_state.auth_bump]]], // PDA 签名
    )?;
    // 16. 更新预言机状态
    ctx.accounts.observation_state.load_mut()?.update(
        oracle::block_timestamp(), // 当前时间戳
        token_0_price_x64,         // 交换前的 token_0 价格
        token_1_price_x64,         // 交换前的 token_1 价格
    );
    // 17. 更新池的最后操作 epoch
    pool_state.recent_epoch = Clock::get()?.epoch;
    Ok(())
}
```

#### 作用

基于输出数量的代币交换

#### 工作内容

- 输入参数
  - `max_amount_in`：最大输入代币数量（滑点保护）
  - `amount_out_less_fee`：期望输出代币数量（不含转账费）
  - 账户：同 `swap_base_input`

- 逻辑
  1. 检查池子允许交易
  2. 计算实际输出量（含转账费）
  3. 确定交易方向并计算储备和价格
  4. 用 `CurveCalculator::swap_base_output` 计算所需输入量
  5. 检查滑点（输入量不超过最大值）
  6. 更新费用，转账代币
  7. 更新预言机和 `recent_epoch`

#### 原理

- 与 swap_base_input 相反，先确定输出量，反推输入量，确保 k 不减少
