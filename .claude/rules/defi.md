---
paths:
  - src/vault/**
  - src/modules/vault/**
  - src/core/vault/**
  - src/router/**
  - src/modules/dex/core/**
  - src/modules/dex/periphery/**
  - src/dex/core/**
  - src/dex/periphery/**
  - src/dex/**
  - src/masterchef/**
  - src/modules/masterchef/**
  - src/staking/**
  - src/domain/staking/**
  - src/modules/staking/**
  - src/launchpad/**
  - src/modules/launchpad/**
  - src/domain/tier/**
  - src/bridge/**
  - src/**/lending/**
  - src/**/amm/**
  - src/**/oracle/**
---

# DeFi 横切规则（定价 / 预言机 / 滑点 / 记账）

适用于涉及 swap、借贷、份额、预言机或跨模块资金结算的合约。业务流程细则见 `products/` 下对应文件。

## 定价与预言机

- 不直接使用 spot price 作为关键定价依据，优先 TWAP、Chainlink 或多源校验
- 预言机结果必须考虑精度、decimals、更新延迟、staleness 与异常值
- 明确价格失效时的降级或暂停策略

## 用户保护与 MEV

- 面向资金进出或兑换的入口，显式考虑 `minAmountOut`、`maxAmountIn`、`deadline` 等参数
- 可被 flash loan、sandwich、oracle manipulation、MEV 放大的路径，先说明攻击面与防护假设

## 代币与记账

- 不假设 ERC20 转账金额等于实际到账；fee-on-transfer / rebase 代币用前后余额差
- 份额、利率、清算、汇率、手续费须明确舍入方向，检查极端边界与累计误差
- 评估资金锁死、坏账、记账不一致、Gas DoS、批量循环爆炸

## 借贷（若适用）

- 抵押率、健康因子、清算阈值、利息累计须覆盖边界与精度
- 清算与坏账处理路径可测试、可审计
