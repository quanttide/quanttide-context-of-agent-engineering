# 跨智能体配置管理分析

## 现状

当前通过 `data/profile` 管理三种工具的配置：

| 工具 | 跟踪的文件 | 用途 |
|------|-----------|------|
| **Zed** | `AGENTS.md` + `settings.json` | 系统提示词 + 编辑器/Agent 配置 |
| **OpenCode** | `opencode.json` | 自定义配置（插件引用） |
| **Hermes** | `config.yaml` | 运行时配置（模型/tools/terminal） |

辅助仓库 `data/library` 提供各工具的配置参考文档，用于对照官方文档维护 profile。

## 收益分层

### 高收益（跨机器直接复用，无需修改）

| 文件 | 原因 |
|------|------|
| `zed/AGENTS.md` | Ponytail 系统提示词，所有机器一致，cp 即用 |
| `zed/settings.json` 中的 `agent.default_model`、`language_models`、`theme`、`font_size` | 模型选择、UI 偏好跨机器一致 |
| `hermes/config.yaml` 中的 `model`、`toolsets`、`compression`、`checkpoints` | 运行时策略偏好，不依赖机器路径 |

### 中收益（跨机器需要小幅调整）

| 文件 | 调整点 |
|------|--------|
| `zed/settings.json` 中的 `agent.tool_permissions` | `always_allow` 的路径白名单是项目路径相关的 |
| `hermes/config.yaml` 中的 `terminal.cwd`、`sandboxes` | 工作目录相关 |
| `opencode/opencode.json` | `plugin` 路径已用 `~` 处理，但符号链接需要每台机器建 |

### 低收益（单机独有或可通过安装获取）

| 内容 | 原因 |
|------|------|
| `opencode/command/`（ponytail 命令） | 由 ponytail 仓库安装 |
| `hermes/SOUL.md` | 安装自带 |
| `hermes/.env` | 含 API 密钥，不纳入版本管理 |
| 运行时缓存/数据库（sessions, state.db 等） | 机器运行时生成 |

## 真正的跨智能体共享点

1. **Ponytail system prompt**（`zed/AGENTS.md`）— 所有 agent 共享相同的懒人开发模式指令
2. **模型选型策略** — main/flash 做日常、pro 修复杂问题、GLM 备选（详见 `data/report/models/index.md`）
3. **工具权限策略** — 什么路径 allow、什么命令 allow（目前只在 Zed 中有显式配置）

## 核心定位

当前 profile 的核心价值是**单机备份恢复**（cp 即用）。跨智能体统一管理的收益有限——三个工具各自的配置语法差异大，抽象层收益不高。

### 可能的演进方向

- 抽一个统一的 `model-providers.md` 记录各供应商 API 接入、模型规格、价格
- 各工具的配置对照该文档生成
- 但这增加抽象层，与"cp 即用"的简洁原则冲突
