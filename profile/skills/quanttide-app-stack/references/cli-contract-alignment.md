# CLI 端的契约对齐三件套（2026-09-12 实录，qtcloud-work/src/cli）

用户指令：studio 那套做完后接一句「**类似的，cli 也需要一套**」——同一批三件套（STATUS/TODO/ROADMAP），按端换段划分与判据命令。产出：`apps/qtcloud-work/src/cli/{STATUS,TODO,ROADMAP}.md`。

## 体检数据（写进 STATUS 的可比数字）

| 项 | 数 |
|---|---|
| `src/` | 11 个 `.rs`，**平铺无目录层**，2 971 行 |
| 最长文件 | `task.rs` **945**、`cli.rs` **541**、`workflow.rs` 459、`catalog.rs` 276、`material.rs` 240 |
| 其余 | `audit.rs` 147、`artifact.rs` 147、`prompts.rs` 92、`help.rs` 89、`paths.rs` 19、`main.rs` 16 |
| `tests/` | 11 个文件，**按用例组织**（agent_step / criteria / criteria_matrix / definition_check / state_machine / task_start / run_context / defaults / contract / human_step / material_intake）+ `common/` |
| `docs/` | 三轴齐：api-references 9 篇、dev-guide 5 篇、user-guide 3 篇 |
| `scripts/` | `validate-version.sh`、`validate-changelog.sh`、`validate-usecases.sh` |
| CI | `cargo fmt --check` / `cargo test --locked` / `cargo clippy --locked` / `cargo publish --dry-run`；tag `cli/*` → 校验版本与 CHANGELOG → 三平台构建 |
| `data/` | 已在 `.gitignore`（本地工作区数据不入库）✓ |

## 命令面 → 交付模块（从 `help` 页四组反推，未实现的也占位）

| 命令面组 | 内容 | 归类 |
|---|---|---|
| 工作区 | `find`（→ `search`）/ `catalog` / `audit` / `material` | 聚合 `catalog`、`material`；服务 `search`、`audit` |
| 工作流（定义侧） | `workflow --list / --new / <名> / --check / --export / --import` | 聚合 `workflow/` |
| 任务（执行侧） | `task --list / --new / <名> / --next / --done / --journal` | 聚合 `task/`（走一步 = 现 `task_run` 那部分） |
| 其他 | `health`（`GET /health`）、`help` | 适配（`health` 现住 `cli.rs`，待分出） |

**另加两个来自代码而非命令面的模块**：`artifact/`（资产表 = **定义型聚合**，标准产物定义 `journal`/`report`/`profile`… 即规范）、`workspace/`（三处位置与落点，**未立**——`workspace_root`/`data_dir` 散在 `cli.rs`，`paths.rs` 只管路径显示）。

**判据命令**：`cargo fmt --check && cargo clippy --locked && cargo test --locked` + `sh scripts/validate-usecases.sh`；**行为不变是硬要求**（用例对账 + 测试全绿）。

## 五段（与 studio 七段的差别：没有「先分层」这一段）

1. **正名与归类**：`find` → `search`（与 studio 同批；还要改 `docs/api-references/find.md` → `search.md`、`help.rs` 分组、README 命令表）；把三类归类写成文
2. **立目录**：聚合 / 服务 / 适配分家；`workspace` 立聚合、`search` 从 `catalog.rs` 分出（服务依赖聚合）、`health` 从 `cli.rs` 分出
3. **拆长文件**：`task.rs` 945 → 任务聚合内多件；`cli.rs` 541 → 只留 clap 定义 + 定位 + 发射；`workflow.rs` 459 → 拆；约定集中到 `CONVENTIONS`
4. **契约对齐**：两侧工具库版本同步、依赖许可清单、IaC 归属
5. **文档对齐**：`layers.md` 重写、api-references 分命令与概念、`prompts` 两侧说法一致、`validate-usecases.sh` 扫描范围随目录调整

**CLI 缺的正是 studio 已走过的「先分层」阶段**：它从一开始就按领域对象分文件（`task.rs`/`workflow.rs`/…），只差「成目录 + 拆行数 + 分出服务与适配」——所以段更短，且**不需要经过分层**。

## 跨端四账（体检一端时必查另一端）

1. **工具库版本错开**：cli `quanttide-work = "0.1.0-beta.4"` vs studio `quanttide_work: ^0.1.0-beta.5`——对表脚本要求两边算出同一个结果，**版本必须对齐**
2. **共用件两侧各一份**：`cli/src/prompts.rs` 与 `studio/lib/repositories/local/prompts.dart`，头注释互相打脸（各说「只是命令行/工作台这一侧的说法」）——要么合一，要么写明差异
3. **结构文档过时**：`docs/dev-guide/layers.md` 说 `outcome.rs` 还在 `src/`（实际已抽进工具箱，只留 `paths.rs`）、没提 `paths.rs`/`prompts.rs`、整批写着「拟建」
4. **`api-references/` 混两类**：命令参考（`task.md`/`workflow.md`/`find.md`…）与概念参考（`workspace.md` 讲布局与资产表，不是命令）

## 提交链（本次实测）

`qtcloud-work`（`src/cli|studio/*.md`）→ `quanttide-work`（`apps/qtcloud-work` 指针）→ `quanttide` 根（`domains/quanttide-work` 指针）。每层 push 后核对 `git log origin/main --oneline -1`。
