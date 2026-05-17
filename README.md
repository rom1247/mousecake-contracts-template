# mousecake-contracts

Foundry 智能合约 **模版仓库**，内置 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 项目规则（`CLAUDE.md` + `.claude/rules/`），覆盖常见 DeFi 产品形态的检查清单。

复制本仓即可开始新项目；按需裁剪目录与规则文件，并修改本文与 `CLAUDE.md` 中的项目名。

## 包含内容

| 组件 | 路径 | 说明 |
|------|------|------|
| Foundry 工程 | `foundry.toml`, `src/`, `test/`, `script/` | 构建 / 测试 / 部署 |
| Claude Hub | [`CLAUDE.md`](./CLAUDE.md) | 全局约束、目录约定、规则索引 |
| 专项规则 | [`.claude/rules/`](./.claude/rules/) | 按 `paths` 条件加载（见 [`docs/PATHS-OPTIMIZATION.md`](./docs/PATHS-OPTIMIZATION.md)） |
| 示例合约 | `src/Counter.sol` | 验证工具链；可删除或替换 |

## 快速开始

```bash
# 1. 使用模版创建仓库后 clone
git clone <your-repo-url>
cd mousecake-contracts

# 2. 安装依赖（含 submodule 时）
git submodule update --init --recursive
forge build
forge test

# 3. 全局替换项目名（README、CLAUDE.md §0/§1）
# 4. 按下方「裁剪」删除不用的产品与空目录
# 5. 若固定一种目录风格，可微调 .claude/rules 中的 paths（见 PATHS 文档）
```

## 推荐源码结构

默认推荐 **domain（共享事实）+ modules（用户入口）**；规则 `paths` **同时兼容**扁平目录（如 `src/masterchef/`）。

```text
src/
├── domain/
│   ├── staking/          # 锁仓时长、金额等（SSOT）
│   └── tier/             # 积分 / IDO 额度（规则见 launchpad.md，无单独 tier 文件）
├── modules/
│   ├── staking/
│   ├── masterchef/
│   ├── launchpad/
│   └── dex/              # 可选
│       ├── core/pool/
│       └── periphery/router|quoter|zap|lens/
├── token/ | nft/ | governance/ | vault/ | bridge/ | proxy/   # 根级或 modules/ 下均可
├── libraries/ | interfaces/ | types/ | errors/               # 可选横切
```

`test/` 建议：

```text
test/
├── unit/
├── integration/          # 如 staking → tier → ido
├── invariant/
├── fork/
└── helpers/
```

`script/` 建议：`deploy/`、`upgrade/`、`interact/`。

## 产品形态与规则映射

| 产品 | 推荐路径 | 兼容扁平路径 | Claude 规则 |
|------|----------|--------------|-------------|
| Token | `token/`、`modules/token/` | `src/token/**` | `products/token.md` |
| Staking | `domain/staking/` + `modules/staking/` | `src/staking/**` | `products/staking.md` |
| MasterChef | `modules/masterchef/` | `src/masterchef/**` | `products/masterchef.md` |
| Launchpad / IDO | `modules/launchpad/` + `domain/tier/` | `src/launchpad/**` | `products/launchpad.md` |
| NFT | `nft/`、`modules/nft/` | `src/nft/**` | `products/nft.md` |
| Governance | `governance/` | `src/governance/**` | `products/governance.md` |
| Vault | `modules/vault/`、`core/vault/` | `src/vault/**` | `products/vault.md` |
| Router / DEX | `modules/dex/periphery/` | `src/router/**` | `products/router.md` |
| Factory / Pool | `modules/dex/core/` | `src/factory/**` | `products/factory.md` |
| Bridge | `bridge/`、`modules/bridge/` | `src/bridge/**` | `products/bridge.md` |
| Proxy | `proxy/` | `src/proxy/**` | `upgradeability.md` |

横切：`defi.md`（定价 / 预言机 / 滑点）、`security.md`、`solidity.md`、`testing-foundry.md`。

### 常见业务组合（参考）

- **锁仓 + 挖矿 + IDO**：`domain/staking` → `modules/masterchef`；`domain/tier` → `modules/launchpad`；积分逻辑不要散落在 launchpad 实现里。
- **Fork Uniswap V3**：池子在 `modules/dex/core/pool/`，路由在 `modules/dex/periphery/router/`。
- **仅借贷等独立协议**：可复制本仓 rules 子集到新仓库，目录自定（本模版不强制借贷目录）。

## 裁剪模版

| 不做 | 建议操作 |
|------|----------|
| 不用 Bridge | 删除 `src/bridge/`（若有）；可选删除 `products/bridge.md` |
| 不做 DEX | 不建 `modules/dex/`；可删 `router.md` / 收窄 `defi.md` paths |
| 不做 NFT | 删除 `products/nft.md` 与对应目录 |
| 全栈 | 保留全部 rules；未建目录时对应规则 **不会加载** |

仅删除空目录时，可 **保留** rule 文件以减少噪音；删 rule 可减小 Claude 扫描面（见 [PATHS 优化](./docs/PATHS-OPTIMIZATION.md)）。

## 自定义 Claude 规则

1. 修改 [`CLAUDE.md`](./CLAUDE.md) §1 目录表与项目描述。  
2. 编辑 `.claude/rules/**/*.md` 顶部 YAML `paths`，与真实 `src/` 一致。  
3. 新增产品：复制 `products/token.md` 改名，并在 Hub §10 增加索引行。  
4. 路径策略（宽泛兼容 vs 精简加载）：见 **[`docs/PATHS-OPTIMIZATION.md`](./docs/PATHS-OPTIMIZATION.md)**。

## Forge 常用命令

```bash
forge fmt
forge build
forge test
forge test --match-path "test/integration/*" -vvv
forge coverage
forge snapshot
```

部署（示例，按实际脚本名修改）：

```bash
forge script script/deploy/Deploy.s.sol --rpc-url $RPC_URL --broadcast
```

环境变量：`PRIVATE_KEY`、`RPC_URL` 等勿写入源码，见 `testing-foundry.md`。

## 文档索引

| 文档 | 用途 |
|------|------|
| [CLAUDE.md](./CLAUDE.md) | Claude Code 主规则（Hub） |
| [docs/PATHS-OPTIMIZATION.md](./docs/PATHS-OPTIMIZATION.md) | `paths` 设计、重叠与精简方案 |
| [Foundry Book](https://book.getfoundry.sh/) | 工具链官方文档 |

## License

按项目需要补充（如 MIT）。
