# 跨智能体语义共享分析

## 问题

跨智能体管理，管的是什么？

不是管配置文件的语法结构（每个工具语法各异，抽象层收益低），而是管**语义**——同一份概念在不同工具中以各自原生方式表达。

## 当前共享的语义

### 1. Ponytail 身份与行为模式

所有 agent（Zed Agent、OpenCode、Hermes）共享同一套身份定义：

- **Ponytail 系统提示词** — "lazy senior dev mode"，定义 agent 行为准则（YAGNI、标准库优先、最小代码、ponytail: 注释标记等）
- 在各工具中的表达方式：
  - **Zed**：`~/.config/zed/AGENTS.md` → 符号链接指向 ponytail 仓库的 `AGENTS.md`（单一源头，不维护副本）
  - **OpenCode**：通过 ponytail 插件从同一仓库的 `ponytail-instructions.js` 注入提示词
  - **Hermes**：暂未集成

同一份语义（身份指令），源头在 ponytail 仓库，各工具通过各自机制引用。

### 2. Ponytail 插件行为

ponytail 插件（`ponytail.mjs`）为 OpenCode 提供了一套命令：

- `ponytail audit`、`ponytail review`、`ponytail gain` 等
- 这些命令封装了 Ponytail 工作流中重复出现的操作模式

插件的语义是"Ponytail 工作流的 terminal 入口"，OpenCode 通过 `plugin` 配置加载它，Zed Agent 通过 `AGENTS.md` 加载同一套逻辑。未来 Hermes 也可能通过 skills 机制加载类似功能。

### 3. 模型选型策略

| 层级 | 模型 | 用途 |
|------|------|------|
| 主力 | DeepSeek V4 Flash | 日常编码，性价比高 |
| 上位替代 | DeepSeek V4 Pro | 复杂问题，DeepSeek 自己解决不了时 |
| 备选 | GLM 5.2 / 5V-Turbo | DeepSeek 生态外的后备方案 |

各工具表达方式：
- **Zed**：`settings.json` 中 `agent.default_model` + `language_models` 配置
- **OpenCode**：`opencode.json` 中 `agents.coder.model`
- **Hermes**：`config.yaml` 中 `model.default` + `model.provider`

同一份策略，各工具各自配置。

### 4. 工具权限策略

哪些操作需要确认、哪些可以直接执行：

- `terminal` → 默认允许
- `fetch` → 默认允许
- `write_file` → 特定路径白名单 always_allow，其余允许

目前只在 Zed 中有显式配置，其他工具尚未统一。

## 语义管理的收益

真正有收益的跨智能体管理，不是"统一配置文件格式"，而是：

1. **一处决策，多处同步** — 模型选型换了，各工具的配置对照着改
2. **身份一致性** — 无论用哪个 agent，Ponytail 的行为模式一致
3. **插件生态复用** — ponytail 命令写一次，OpenCode 和未来的 agent 都能用

## 管理的边界

不管理的：

- 每个工具独有的功能（Zed 的 UI 主题、Hermes 的 sandbox/checkpoint）
- 通过安装获取的内容（默认提示词、内置命令）
- 含密钥的文件（`.env`）
- 运行时生成的数据（缓存、数据库）
