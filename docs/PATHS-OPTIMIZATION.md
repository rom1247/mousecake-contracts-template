# Claude Rules：`paths` 优化说明

本文说明本模版中 `.claude/rules/*.md` 的 `paths` 设计、已知重叠与可选精简方案。  
Claude Code 行为：**无 `paths` 的规则在会话启动时全量加载**；**有 `paths` 的规则在读取匹配文件时注入**（见 [官方文档](https://docs.anthropic.com/en/docs/claude-code/claude-md#path-specific-rules)）。

## 1. 设计目标（模版仓）

| 目标 | 做法 |
|------|------|
| 复制即用 | 同时支持 `src/masterchef/` 与 `src/modules/masterchef/` |
| 目录演进 | 推荐 domain + modules，不强制一次迁完 |
| 控制噪音 | 避免过宽 glob；DEX 区分 core / periphery |
| 无 tier 规则文件 | `src/domain/tier/**` 归入 `launchpad.md` |

## 2. 当前加载矩阵（摘要）

编辑下列文件时，**除 Hub + solidity + security** 外，典型额外规则：

| 编辑路径 | 产品规则 | 常同时加载 |
|----------|----------|------------|
| `src/modules/dex/core/pool/Foo.sol` | `factory.md` | `defi.md` |
| `src/modules/dex/periphery/router/Bar.sol` | `router.md` | `defi.md` |
| `src/domain/tier/Tier.sol` | `launchpad.md` | `defi.md` |
| `src/modules/masterchef/Chef.sol` | `masterchef.md` | `defi.md` |
| `src/token/Token.sol` | `token.md` | — |
| `src/Counter.sol` | — | 仅基础规则 |

## 3. 已实施的优化（相对初版）

| 文件 | 调整 | 原因 |
|------|------|------|
| `router.md` | `src/modules/dex/**` → `src/modules/dex/periphery/**`（及 `src/dex/periphery/**`） | 编辑 Pool 时不应加载 Router 清单 |
| `factory.md` | 保持 `src/modules/dex/core/**` | 与 router 分工 |
| `defi.md` | `src/modules/dex/**` 拆为 `core/**` + `periphery/**` | 与上同，仍覆盖池子定价场景 |
| `launchpad.md` | 含 `src/domain/tier/**` | 无单独 tier.md |

## 4. 仍存在的重叠（可接受 vs 可优化）

### 可接受的重叠

- **`defi.md` + 产品 rule**：横切（预言机、滑点）与业务流程（IDO 额度）并存，编辑 dex/staking 时常同时出现。
- **`security.md` + 产品 rule**：安全总则与场景清单互补。

### 可选手动精简

| 项 | 说明 | 建议 |
|----|------|------|
| `*Token*`、`*Stake*` 等 glob | 可能误匹配 `MockStakeToken.sol` 等 | 目录稳定后 **删除 glob**，只保留目录 paths |
| `defi.md` 列举全部产品目录 | 与产品 rules 重复触发 defi | 固定布局后改为只列 `**/oracle/**`、`modules/dex/**`、`domain/**` |
| 未使用产品的 rule 文件 | 永不触发 | 裁剪模版时 **删除对应 md** |
| `solidity.md` 的 `src/**/*.sol` | 任意 sol 都加载 | 保留（编码规范宜全局） |

## 5. 三种 paths 配置档位

### 档位 A：模版默认（当前）

- 扁平 + modules + domain 双路径
- 保留少量 `*Product*` glob
- **适合**：新项目、目录未定型

### 档位 B：定型精简（推荐上线前）

示例（MasterChef）：

```yaml
paths:
  - src/modules/masterchef/**
```

删除 `src/masterchef/**` 与 `*MasterChef*` glob。其他产品同理。

### 档位 C：严格域隔离

- `defi.md` 仅保留：

```yaml
paths:
  - src/modules/dex/core/**
  - src/modules/dex/periphery/**
  - src/**/oracle/**
  - src/**/lending/**
```

- 质押 / IDO 的定价细节 **不** 通过 defi 在 `domain/tier` 加载，仅靠 `launchpad.md` / `staking.md`（若 tier 无 swap 逻辑可接受）

## 6. 不建议的做法

| 做法 | 原因 |
|------|------|
| 在 `CLAUDE.md` 里 `@import` 全部 rules | 启动时全量加载，失去 paths 意义 |
| 所有产品共用一个 `products/all.md` + `paths: src/**` | 改 Counter 也加载 IDO 规则 |
| `paths: src/**` 写在 defi / security 以外的产品文件 | 上下文膨胀、遵循率下降 |

## 7. 维护检查清单

复制模版并开发一段时间后：

- [ ] 删除未使用产品对应的 `products/*.md`
- [ ] 删除不再使用的扁平 `paths` 行
- [ ] 评估是否移除 `*Glob*` 兜底
- [ ] 确认 `router` / `factory` 仅匹配 periphery / core
- [ ] 新增目录时同步改 YAML 与 `CLAUDE.md` §10、`README` 映射表
- [ ] 跑 `forge test` + 在 Claude 中打开典型文件，确认加载规则符合预期

## 8. 与 Hub 的关系

- **README**：给人看的目录与裁剪说明  
- **CLAUDE.md §10**：给 AI 的索引摘要  
- **本文件 + 各 rule 顶部 YAML**：paths 的权威来源（以 YAML 为准）

修改 paths 时：**先改 rule 文件 frontmatter，再改 CLAUDE §10 摘要**，避免两处不一致。
