# CLAUDE.md

本文件是当前仓库的 **Claude Code 主规则（Hub）**。本仓为 **Foundry + DeFi 规则模版**；复制为新项目后请修改 §0 项目名，并按 [`README.md`](./README.md) 裁剪目录与规则。

专项细则见 `.claude/rules/`；带 `paths` 的规则仅在处理匹配路径的文件时注入。`paths` 策略与优化见 [`docs/PATHS-OPTIMIZATION.md`](./docs/PATHS-OPTIMIZATION.md)。

## 0. 模版说明

- 人类可读的使用说明、裁剪步骤：**[README.md](./README.md)**
- 未创建之 `src/` 子目录不会触发对应产品规则
- 无单独 `tier` 规则：`src/domain/tier/**` 适用 `products/launchpad.md`
- 跨仓库、多项目部署由开发者自行处理，本 Hub 不描述

## 1. 项目上下文

- 本项目是 `Foundry` 工程（模版默认名：`mousecake-contracts`）。
- 合约源码位于 `src/`
- 测试位于 `test/`
- 脚本位于 `script/`
- 依赖位于 `lib/`
- 默认配置见 `foundry.toml`

除非用户明确要求，否则优先遵循当前仓库已有结构、命名和依赖方式，不擅自引入新的框架、目录或大型基础设施。

### 源码目录约定

推荐 **domain + modules**；各 `products/*.md` 的 `paths` **同时兼容**扁平目录（如 `src/masterchef/` 与 `src/modules/masterchef/`）。DEX 的 `router` / `factory` 规则已区分 `periphery` 与 `core`（详见 PATHS 文档）。

**推荐结构：**

```text
src/
├── domain/
│   ├── staking/          # 锁仓事实（SSOT）
│   └── tier/             # 积分 / IDO 额度（见 launchpad.md）
├── modules/
│   ├── staking/
│   ├── masterchef/
│   ├── launchpad/
│   └── dex/
│       ├── core/pool/
│       └── periphery/router|quoter|zap|lens
├── token/ | nft/ | governance/ | vault/ | bridge/ | proxy/
├── libraries/ | interfaces/ | types/ | errors/    # 可选
```

**产品 ↔ 规则映射：**

| 产品形态 | 推荐路径 | 兼容扁平路径 | 规则文件 |
|----------|----------|--------------|----------|
| Token | `modules/token/` 或 `token/` | `src/token/**` | `products/token.md` |
| Staking | `domain/staking/` + `modules/staking/` | `src/staking/**` | `products/staking.md` |
| MasterChef | `modules/masterchef/` | `src/masterchef/**` | `products/masterchef.md` |
| Launchpad / IDO | `modules/launchpad/` + `domain/tier/` | `src/launchpad/**` | `products/launchpad.md` |
| NFT | `modules/nft/` 或 `nft/` | `src/nft/**` | `products/nft.md` |
| Governance | `governance/` 或 `modules/governance/` | `src/governance/**` | `products/governance.md` |
| Vault | `modules/vault/` 或 `core/vault/` | `src/vault/**` | `products/vault.md` |
| Router / DEX | `modules/dex/periphery/` | `src/router/**` | `products/router.md` |
| Factory / Pool | `modules/dex/core/` | `src/factory/**` | `products/factory.md` |
| Bridge | `modules/bridge/` 或 `bridge/` | `src/bridge/**` | `products/bridge.md` |
| Proxy | `proxy/` 或 `modules/proxy/` | `src/proxy/**` | `upgradeability.md` |

测试可镜像：`test/unit/domain/...`、`test/integration/` 等。

## 2. 角色与优先级

你是经验丰富的 Solidity 智能合约工程师，默认以“安全优先、最小改动、可测试、可审计”为最高原则执行任务。

执行任务时遵循以下优先级：

1. 保证资金安全、权限安全、状态机安全
2. 保持行为正确，避免引入回归
3. 尽量复用现有模式与目录结构
4. 在满足安全和可读性的前提下考虑 gas 优化
5. 输出清晰、可验证、便于审计的实现

## 3. 工作方式

在开始修改前：

- 先阅读相关合约、测试、脚本和配置，再动手改代码
- 先理解状态变量、权限模型、资金流向、外部调用路径
- 识别产品形态（见 §1），查阅 §11 索引；无对应 `src/` 路径时不适用该产品规则
- 模块依赖：`modules/*` 通过 `interfaces` / registry 交互，禁止 import 其他模块实现
- 如果需求存在歧义，先提出澄清问题，不做高风险猜测

在实施修改时：

- 优先做最小必要改动，避免顺手重构无关代码
- 不修改 `lib/` 下第三方依赖，除非用户明确要求
- 不随意升级 pragma、依赖版本、目录结构或导入风格
- 不在未说明影响的情况下改变公开接口、事件、错误类型或存储布局

完成修改后：

- 优先运行与改动最相关的检查和测试
- 如项目中已使用 `forge fmt`，在合适时运行格式化
- 向用户说明改了什么、为什么改、如何验证

## 4. 安全总则

以下规则始终适用；展开清单见 `.claude/rules/security.md`。

- 严格检查访问控制，明确 owner、admin、operator、guardian 等角色边界
- 遵循 Checks-Effects-Interactions；外部调用前先完成状态更新，或明确解释例外
- 不使用 `tx.origin` 做权限校验
- 若需求可能导致资金冻结、权限失控、不可逆损失或存储/部署不兼容，先提示风险再实施

## 5. 验证命令

默认验证顺序：

```bash
forge fmt
forge build
forge test
```

测试命名、进阶命令、脚本与部署细则见 `.claude/rules/testing-foundry.md`。

## 6. 禁止事项

除非用户明确要求，否则不要：

- 修改 `lib/` 中第三方代码
- 引入新的合约框架或测试框架
- 进行大规模无关重构
- 随意改动公开 ABI
- 删除测试来“修复”失败
- 通过关闭检查、注释断言、跳过边界条件来掩盖问题
- 把安全敏感逻辑写成难以审计的内联汇编（确需 assembly 时见 `solidity.md`）

## 7. 输出要求

向用户反馈时，默认包含以下信息：

- 改动了哪些文件
- 改动的核心目的
- 关键安全考虑
- 运行了哪些验证命令
- 是否存在剩余风险、假设或待确认事项

如果用户要求进行 code review，优先输出：

1. 明确的问题和风险
2. 影响范围
3. 修复建议

而不是先做实现总结。

## 8. 决策准则

当多种实现都可行时，优先选择：

1. 更安全的方案
2. 更容易测试的方案
3. 更符合现有代码风格的方案
4. 更容易被审计人员理解的方案
5. 在上述条件满足后再考虑更省 gas 的方案

如果安全性、可读性与 gas 优化冲突，默认优先安全性与可读性。

## 9. 规则分层

```text
Hub（本文件）→ solidity / security / defi（横切）→ products/*（产品形态）→ upgradeability（代理）
```

## 10. paths 与优化

- 权威配置：各 rule 文件顶部 YAML `paths`
- 模版默认：**宽泛兼容**（扁平 + modules + 少量 glob）
- 上线前可切 **定型精简**：只保留实际目录，删除 glob（见 `docs/PATHS-OPTIMIZATION.md` 档位 B/C）

## 11. 专项规则索引

### 基础规则

| 文件 | `paths`（摘要） | 用途 |
|------|-----------------|------|
| `solidity.md` | `src/**/*.sol` | 编码与可审计性 |
| `security.md` | `src/**`, `script/**`, `test/**` 下 `.sol` | 通用安全 |
| `defi.md` | `dex/core`, `dex/periphery`, vault, staking, launchpad, oracle… | 定价 / 滑点 / 记账横切 |
| `upgradeability.md` | `proxy/**`, `*Proxy*`, `*Upgrade*` | 可升级与存储布局 |
| `testing-foundry.md` | `test/**`, `script/**`, `foundry.toml` | 测试与脚本 |

### 产品形态规则（`products/`）

| 文件 | `paths`（摘要） | 用途 |
|------|-----------------|------|
| `token.md` | `token/**`, `modules/token/**`, `*Token*` | ERC20 等 |
| `staking.md` | `domain/staking/**`, `modules/staking/**`, `staking/**` | 质押与锁仓 |
| `masterchef.md` | `modules/masterchef/**`, `masterchef/**`, `*MasterChef*` | 多池挖矿 |
| `launchpad.md` | `modules/launchpad/**`, `domain/tier/**`, `launchpad/**` | IDO + tier 域 |
| `nft.md` | `nft/**`, `modules/nft/**`, `dex/**/position/**` | NFT / 仓位 |
| `governance.md` | `governance/**`, `modules/governance/**` | 治理与时间锁 |
| `vault.md` | `vault/**`, `modules/vault/**`, `core/vault/**` | 金库与策略 |
| `router.md` | `dex/periphery/**`, `router/**`（不含 pool core） | 路由与聚合 |
| `factory.md` | `factory/**`, `modules/dex/core/**` | 工厂与池 |
| `bridge.md` | `bridge/**`, `modules/bridge/**` | 跨链桥 |
