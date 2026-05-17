---
paths:
  - src/router/**
  - src/modules/dex/periphery/**
  - src/modules/router/**
  - src/dex/periphery/**
  - src/dex/**
  - src/**/*Router*.sol
  - src/**/*Aggregator*.sol
---

# Router / Aggregator

## 适用范围

多跳 swap、聚合路由、ETH/WETH 包装。DEX 推荐 `src/modules/dex/periphery/router/`；兼容扁平 `src/router/`。

## 设计检查清单

- 路径中每个 hop 的 token 与 pool 校验；禁止任意 `call` 目标
- `amountOutMin`、`deadline`、recipient；部分成交行为明确
- ETH 收发与 WETH `deposit`/`withdraw` 一致性
- 回调（uniswapV3 等）：`msg.sender` 与池地址校验
- `Permit2` / `approve`：最小授权、可撤销
- 聚合器：多源报价失败、滑点保护、残留代币处理
- MEV 与 sandwich 面（见 `defi.md`）

## 权限与资金

- 不持有用户长期余额；扫残留代币权限可控
- 费率收款地址变更可审计

## 外部依赖

- `defi.md` 横切规则；ERC20 见 `security.md`
- Pool 逻辑在 `modules/dex/core/pool/` 时，定价不变量见池合约与 `defi.md`

## 测试要点

- 过期 deadline、滑点不足 revert
- 恶意路径、错误 pool 地址 revert
