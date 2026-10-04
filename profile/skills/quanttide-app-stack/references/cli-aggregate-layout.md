# 聚合目录四块形态：逐件映射（qtcloud-work cli 实测，2026-09-21）

配套 `quanttide-app-stack` SKILL.md 的「聚合目录四块形态」节。数据来自 `apps/qtcloud-work/src/cli`（当时约 5 000 行、24 个模块、42 个非入口件）。

## 法源两条

1. **规范**：每个实体页就是 `## 领域属性` / `## 领域事件` / `## API端点`（`process/work-order.md` 多一节 `## 命令行工具API端点`）——三块照抄，第四块（`repos.rs`）是规范故意不写的那半：「位置不进模型，由平台在装载时给」。
2. **同族家法**：Dart studio 侧早就是 `lib/{models,services,repositories}`（qtclass / qtcrowd / qtcloud-pay 一致）。`qtcloud-pay/src/cli`（Rust）走的是客户端形态 `{api,commands,models}`——不是本形态，别拿它当先例。

## 逐聚合映射

### order/（最大，且唯一要拆逻辑）

| 现在 | 去处 | 理由 |
|---|---|---|
| `model.rs`（`WorkOrder` 封面 + `WorkRecord` 流水一笔） | `models/mod.rs`（必要时拆两件） | 领域属性 |
| `record.rs`（`validate_yaml` / `validate` / `validate_records` / `append_check`——同 id、跳号、倒流） | `models/record.rs` | 模型的规矩 |
| `progress.rs`（走过哪几步、下一步、完结，只推导不落字段） | `models/progress.rs` | 模型的推导 |
| `events.rs`（`WorkOrderCreated` / `WorkRecorded`） | `events.rs` | 领域事件 |
| `actions.rs`（开单/走一步/人记一笔/日志/删白纸）、`execute.rs`、`inspect.rs`、`journal.rs`、`ai.rs` | `services/`（一动作一件） | API 端点 |
| `mod.rs` 的 `Order::save` / `ensure` + `open` / `listing` / `create` / `delete` / `workflow_by_id` | `repos.rs` | 仓储（读写的归读写） |
| `mod.rs` 的 `Order` 类型与出口 | 留 `mod.rs` | 门面 |

**真拆点**：`Order` 现在既装字段（`payload` / `workflow` / `file`）又落盘（`save` / `ensure`），按判据切成 `models::Order`（纯字段 + 判定）+ `repos`（读写）。这是整批整理里唯一动结构逻辑的地方，对照时要额外盯 `events.jsonl` 与工单落盘全文。

### workflow/（四块齐，最典型的样板，建议首个试点）

| 现在 | 去处 |
|---|---|
| `model.rs`（`Workflow` / `Step`） | `models/mod.rs` |
| `read.rs`（`of` 读法 + `validate` 语法校验，纯逻辑） | `models/read.rs` |
| `check.rs`（定义核对：判据路径在不在区内、描述点到的小节有没有覆盖） | `services/check.rs` |
| `actions.rs`（写/看/列/核对/导出/导入） | `services/actions.rs` |
| `events.rs`（`WorkflowCreated`） | `events.rs` |
| `yaml.rs`（`load` / `dump` / schema 门禁）+ `WorkflowFile` 的文件那半（`create` / `write` / `open` / `listing`） | `repos.rs` |

### artifact/（定义型聚合：只有 models 有内容）

`mod.rs`（资产表二十格 + `STATED`）→ `models/mod.rs`；`model.rs`（`Artifact` 实例：名字 + 规格）→ `models/artifact.rs`；`place.rs`（落点算式 `<类别>/<工单名>.md`）→ `models/place.rs`。
`events/` 与 `services/` 与 `repos.rs` 都空着——**不建空壳**，等真内容来。

### workspace/（用户单独圈的样板；2026-09-21 实测）

现状三件 51 行：`mod.rs`(10) / `model.rs`(27，`identity()` 造六字段) / `events.rs`(14，`WorkspaceCreated`)——`models/` 与 `events/` 已就位。缺席的两块住在 `locate/`：

| 今天在 `locate/` | 位置 | 是什么 |
|---|---|---|
| `IDENTITY` 常量 + `identity_file()` | mod.rs:27 / :85 | 身份文件名与路径（**留装载**） |
| `ensure()` 里的"身份缺则生成 + 写盘 + 发 `WorkspaceCreated`" | mod.rs:115–120 | **一个工作区动作**（开工作区） |
| `workspace_id()`（先 `ensure`、再读文件、再取 `id`） | mod.rs:133–137 | 读身份内容 |

调用方两处：`workflow/mod.rs:79`（凭证派生认工作区 id）、`events.rs:20`（事件公共字段）。即**装载层今天既给位置又解析工作区内容**——这就是那一截该回本家的证据。

目标形态与四处改动：

```
workspace/
├── mod.rs            类型与出口（已有）
├── models/mod.rs     身份六字段 + identity()            ← 今天 model.rs
├── events.rs         WorkspaceCreated（Updated 等动作）  ← 今天 events.rs
├── services/open.rs  开工作区：身份缺则生成 → 写 → 发事件 ← locate::ensure() 的工作区那半
└── repos.rs          身份读 / 写 / 取 id                 ← locate::{读取, workspace_id}
```

- `locate::ensure()` 剩什么：只管建目录（账本、工作流目录）——那是装载的事；
- `locate::workspace_id()` → `workspace::repos::id(locate)`（先 `open` 再读）；
- `IDENTITY` / `identity_file()` 留 `locate`，`repos.rs` 用它拿路径。

**两条别收过头**（用户当场圈定）：`workspace_key()` / `account()` / 账本 XDG 缺省是"这台机器上这本账"的规矩，留装载；`services/` 现在别建空件，等"改工作区信息"的动作来了再建。

## 判据自检（四句话，都能在现成代码上对）

`models/` 不碰盘 · `repos.rs` 只搬字节（位置由装载给）· `services/` 编排并发事件 · `events.rs` 只定义。

## 整理顺序

① 判据落进 `CONTRIBUTING`（三类的下一层）+ `dev-guide/index.md` 落点图改目录级（**拍板后才动**）→ ② `workflow/` 试点 → ③ `artifact/` → `workspace/` → `order/` → ④ 契约测试的层清单从 42 条按文件改成**按块的依赖断言**，再 STATUS 复检。每步照旧：门禁 + 改动前后双二进制逐条对照（见 `quanttide-repo-ops` 的「重构行为不变对照台」节）。
