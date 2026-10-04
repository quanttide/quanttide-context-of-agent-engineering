---
name: quanttide-app-stack
description: 开发量潮云应用三端（Rust CLI / Go Provider / Flutter Studio）时触发。
---

# 量潮云应用三端开发（cli / provider / studio）

量潮云产品应用（qtcloud-delib、qtcloud-econ 等）的标准结构为 `src/{cli, provider, studio}` 三端。参考实例：qtcloud-delib（2026-08-08）、qtcloud-econ（2026-08-09 建立）。Flutter 客户端细节见 `quanttide-flutter-studio`；本技能覆盖 CLI 与 Provider 端及三端数据关系。

## 三端标准结构（+site 四端变体）

```
<app-repo>/src/
├── cli/        Rust CLI（命令 <app>，如 qtcloud-econ）
├── provider/   Go 服务端（HTTP API）
├── studio/     Flutter 客户端（见 quanttide-flutter-studio）
└── site/       React + Vite 官网（可选——qtfounder/qthealth 有，纯后台型如 secret 无）
```

site 端：`npm create vite@latest . -- --template react-ts` + `npm install` + `npm run build`；新模板的 `/vite.svg` import 可能不存在（public/ 无此文件）——骨架页删掉 logo import 即可；`.gitignore` 补 `/src/site/node_modules/` `/src/site/dist/`。

### 四端角色与依赖规则（2026-08-26 qtcloud-crowd/qtcrowd 双 app 教训）

同一产品家族两个 app（管理云 + 参与端）时，四端角色要分清——**初始化前先问"这个端给谁用"**（用户两次纠正："qtcrowd studio 是给众包参与人员使用的"、"site 是介绍功能的，不应该有数据模型"）：

| 端 | 角色 | 依赖 |
|----|------|------|
| site | ① 功能介绍页（管理云自身——无数据模型，静态）② 数据展示站（对外信息展示——可拉 provider 只读） | 静态无依赖；展示真实数据时走 provider API（只读） |
| studio | **管理端**（人财事：审核/认证/结算）或 **参与端**（认领/交付/结算查看）——同 app 家族两端可并存 | 管理端走 provider API；参与端本地数据（my-tasks.json）不写管理端数据 |
| cli | 管理方本机工具 | 直接读写同构 JSON（QTCLOUD_*_DATA 覆盖），部署后可切 provider API |
| provider | 唯一服务端数据入口 | 数据文件（store 抽象：local/oss） |

- **site 不涉及数据模型**：功能介绍页 = hero + 功能卡片（一句话/卡），不展开管理动作/数据字段（用户纠正过）
- **数据边界**：参与端本地数据（我的认领/结算）≠ 管理端数据（任务/执行方/台账）——参与端不写管理端数据；两端通过 provider 联动
- **同名 app 家族**（qtcrowd 参与端 + qtcloud-crowd 管理端）共用 provider 时：qtcrowd 是独立 app 仓库，调 qtcloud-crowd provider API（跨 app 服务端依赖 OK——provider 是唯一入口）
- **依赖方向硬约束（2026-08-26 定稿）**："**前台可以依赖后台，反之不行**"——后台（管理端 provider）不知道前台存在；前台依赖：OSS（读）+ 后台 API（写回）。推式同步（后台调前台 API）= 后台依赖前台 = 违反约束
- **OSS 共享数据层架构（2026-08-26 qtcrowd/qtcloud-crowd 定稿——前后台同家族）**：
  ```
  后台 provider（qtcloud-crowd——私有桶：审核/认证/结算完整数据）
      ▲ 上架查询（GET /api/tasks?status=published——前台主动拉取）
      ▲ 认领/交付写回（前台 provider 转发）
      │
  qtcrowd-provider（前台唯一服务端——自己的桶 qtcrowd-provider）
      ├── 上架：调后台 API（published 任务）→ 写自己的桶（黄页快照 public/tasks/{id}.json）
      ├── 数据 API：GET /api/tasks（site/studio 全经它——不再直读 OSS/CDN）
      └── 写操作：claim/deliver 转发后台（状态码透传）
      ▲ site/studio 只认 qtcrowd-provider（读+写）
  ```
  - **架构演进链（用户逐步纠正到定稿）**：① 后台建公开桶前台直读 → 被否（"后台不应该直接往 site 里提交"——公开层不在后台图纸）；② 后台投递前台已有桶 → 被否（"只删除不新建"+ 前台自治）；③ **定稿：前台 provider 上架模式**——qtcrowd-provider 调后台 API 拉取 published 任务写入**自己的桶**，再以数据 API 供给 site/studio。核心：**前台完全自治（自己的存储+自己的 API），后台只响应调用（零投递）**
  - **依赖方向硬约束**："前台可以依赖后台，反之不行"——上架（前台调后台 API）+ 转发（前台调后台 API）都符合；后台从不主动调前台
  - **注册模式本质**：前台写自己的桶 = 资源注册（写对象即注册、读桶即发现）——比推式/拉式/Outbox 都轻（OSS 即注册表——不需要注册中心）；多租户 = 路径加维度（`{tenant}/tasks/`——设计留缝代码不做）
  - **"只删除不新建"纪律**：图纸外的桶/资源不做不建（曾自建 qtcrowd-public 被要求删除；后台公开桶 qtcloud-crowd-public 删桶 + terraform state rm 双清理）
  - **实施前核对图纸**：初始化/实施基础设施前先问"这个端给谁用/存储归谁/图纸上有吗"——用户两次纠正（"qtcrowd studio 是给参与人员用的"、"site 是介绍功能的"）
- 参与端数据源优先本地文件（`data/my-*.json` 原子写 + 环境变量覆盖）——Web 平台自动降级 InMemory

## 新应用仓库初始化（2026-08-16 qthealth/quanttide-devops 实践）

- **gh 建仓默认进个人账户**：`gh repo create qthealth` 建到 `Guo-Zhang/qthealth`——必须显式 `gh repo create quanttide/<name> --public --license apache-2.0 --description "..."` 指定组织；建错后删除需 `gh auth refresh -h github.com -s delete_repo` 授权
- **初始化顺序**：先建根 `.gitignore`（`/src/cli/target/`、`/src/provider/server`、`/src/site/node_modules/`、`/src/site/dist/` 等构建产物）**再** build——cargo new/flutter create 后立即 `git add -A` 会把 `target/` 和 go 编译二进制一起提交（实测踩坑两次：41 文件清理）；**新增端时用追加（append）而非覆盖 .gitignore**——site 加入时覆盖导致 target/server 二次误入（本会话又踩一次）
- 三端脚手架：`cargo new cli --name <app>-cli` + `flutter create studio --project-name studio --platforms=web,windows,linux,macos,android,ios --org com.quanttide` + `go mod init github.com/quanttide/<app>/provider`（provider 骨架参考 qtcloud-secret：cmd/server + internal/{model,handler}）
- **studio 骨架最小模式**（qtcloud-secret/devops/qthealth 一致）：main.dart（App 壳 + AppState 注入）+ app_state.dart（ChangeNotifier + 占位动作）+ ui/ 单页（ListenableBuilder 监听——免 provider 依赖）+ widget_test（渲染断言）
- **验证三连**：cli `cargo build && cargo run -- <cmd>` / provider `go build ./... && go vet ./... && gofmt -l .` / studio `flutter analyze && flutter test`
- **提交链**：`<app>` → 领域父仓库（`chore: update <app> submodule`）→ quanttide 根

### 领域仓库加子模块（2026-08-17 quanttide-product/docs/brochure 实践）

在领域仓库（quanttide-product 等）挂新子模块（文档仓库）的坑：

- **先建仓再 add**：`gh repo create quanttide/<name> --public` 后新仓库为空，`git submodule add` 报"无法检出子模组/没有检出一个提交"——先 clone 新仓库 + 建初始提交（README）再 add；仍失败（本环境"您位于一个尚未初始化的分支"）则**手动方式**：`git clone <url> docs/<name>` → `rm -rf docs/<name>/.git` → `git add docs/<name>` → 手动往 `.gitmodules` 追加 `[submodule "docs/<name>"]` + `path`/`url` 三段 → 提交
- **远端仓库名必须用 `quanttide-<name>` 前缀规范**（用户纠正：`brochure-of-product-development` 错，应为 `quanttide-brochure-of-product-development`）——与其他子模块（quanttide-context-of-…/quanttide-journal-of-…）命名一致；建错后：建新仓库 → push 内容 → 修 remote + 修 `.gitmodules` → 删旧仓（`gh repo delete` 需 `delete_repo` scope，`gh auth refresh -h github.com -s delete_repo` 授权）
- **gh repo rename 语法**：`gh repo rename <new-name>`（单参数），不接受 `<old> <new>` 双参数
- **手动方式创建的子模块**：remote 需手动 `git remote set-url origin <正确 url>`（clone 后 .git 删除再 add 时 remote 可能指向父仓库）

## 应用间功能迁移（企业后台/个人前台区分，2026-08-16 qtcloud-health→qthealth）

用户把个人功能（情绪日记/量表）从企业后台迁到个人前台应用的模式：

1. **复制**：`cp -r` 源 studio 的 lib/ 全目录（models/views/services/data/blocs/screens 等）+ test/ 到目标 studio（含 `lib/*.dart` 根文件）；删目标骨架残留（app_state/ui 占位页）
2. **包名替换**：测试文件 import `package:<旧包名>/...` → `package:<新包名>/...`（`sed -i 's/package:qtcloud_health_studio\//package:studio\//g' test/**/*.dart`）——lib 内相对导入不受影响
3. **依赖补齐**：pubspec 加源项目的依赖（flutter_bloc/go_router/shared_preferences/fl_chart/equatable 等）——**注意 pubspec 缩进**（dependencies 块不能嵌套进自身，写坏用 read_file 全文重写）
4. **验证**：`flutter pub get && flutter analyze && flutter test`（本实例 58 测试全过）+ build web
5. **源侧移除**：`git rm -r src/studio` + `src/README.md` 标注迁移去向（保留企业后台部分：provider/manifests/docs）
6. **提交链**：两个应用仓库各提交推送 → 父仓库指针（含两个 gitlink）→ quanttide 根

前后台区分原则：个人数据/功能（日记、量表）→ 个人前台（qthealth）；企业/租户能力（provider、manifests、多租户设计）→ 企业后台（qtcloud-health）。

## 种子数据：各自维护（用户明确要求，勿共享）

三端各自维护自己的机制/领域种子数据副本，不共享同一文件：

```
src/cli/data/mechanisms.json         # CLI 独立
src/provider/data/mechanisms.json    # Provider 独立
src/studio/assets/data/mechanisms.json  # Studio 独立（pubspec 注册 assets/data/ 目录级）
```

改任一端的种子数据不自动影响其他端——需要时手动同步（cp 到各端）。

### 换种子内容（模拟数据 → 真实案例）的耦合面清点（2026-09-24 qtdata studio 实证）

用户说「用这个最新案例替换当前的模拟数据」时，真正的成本在**清点被内容吊住的断言**，不在写新 JSON：

- **两条读取路径都要查，别只看 helper**：`test/helpers/seed.dart`（同步读 `assets/data/seed_projects.json`，绕过 rootBundle）＋ **直接吃 rootBundle 的屏**（`dashboard_screen_test.dart` 不 import helper，第一轮就漏了它，跑到最后才炸）
- **耦合点清单（逐个 grep 后先列出来再动手）**：项目名／客户文案／status 与 currentPhase 的词／更新日期／交付物计数与完成度／阶段名与每阶段项数／矩阵单元格名／蓝图步骤与异常关键词／结款文案（`已收 X / Y 万`，格式在 `payment_card.dart`）／由 `contractAmount` 推出的值（确认收入 `toStringAsFixed(1)`——**选数字时避开 round 边界**，2.25 这种别用自己的心算当预期）
- **保持数值型断言稳定，只改文案层**：新数据仍做成 4 个交付物（3 done + 1 active → 75%）、6 阶段 14 项（其中 11 项 hasDoc）——「行为不变」的可比信号不丢，改动面缩到文案
- **脱敏与诚实**：客户名写成「某医疗器械企业」这类；没有真实值的经济字段填**示例值并在 `pricingNote` 与 JSON `_comment` 里注明「非实际报价」**，同时开一条 TODO 记「换真实数字」（判据＝那句注明消失）——别让演示数据看起来像真实报价
- 收尾：门禁三连（`dart format --set-exit-if-changed` / `flutter analyze` / `flutter test`）＋ 在 studio STATUS 加「界面数据」小节（来源／内容／示例数字／断言同步情况）＋ TODO 补「案例内容随案例库同步」一条（判据＝seed 与 `docs/gallery/<业务线>/` 案例页逐条对得上）

## Rust CLI 模式（src/cli）

参考：qtcloud-devops/src/cli、qtcloud-delib/src/cli。命令名 = `<app>`（bin），包名 = `<app>-cli`（lib+bin 分离）。

- **Cargo.toml**：`[lib] name = "<app>_cli"` + `[[bin]] name = "<app>"`（lib+bin 分离，测试走 lib）；依赖 clap(derive)/serde/serde_json；`[dev-dependencies] assert_cmd`
- **src/lib.rs**：`pub mod <domain>;`（领域模块，测试放这里）
- **src/<domain>.rs**：领域模型（serde Deserialize，对齐种子数据 JSON 字段，如 `#[serde(rename = "player_id")]`）+ `load(path)` 加载 + 摘要函数 + `#[cfg(test)]` 单元测试
- **src/main.rs**：clap derive——`Cli`（全局 `--path` 参数，`#[arg(long, global = true)]`）+ `Commands`（如 `Mechanism { #[command(subcommand)] action }`）+ `MechanismAction`（List/Show{id}）；main 里 match 分发，错误 `process::exit(1)`
- **.gitignore**：`/target`
- **验证**：`cargo fmt && cargo build && cargo test`（`cargo test` 输出多 target 段，`tail` 只看到 0 tests 是误导——用 `grep "test result"` 看全部）
- 运行：`./src/cli/target/debug/<app> mechanism list`（默认数据路径 `src/cli/data/<domain>s.json`，相对仓库根）
- **CHANGELOG.md** 补 v0.1.0 记录（rust 组完整）

### LLM 驱动命令（clarify/design 模式，2026-08-09 qtcloud-econ）

数据工程流程命令（需求澄清 → 机制设计）用 LLM 把 Markdown/JSON 转为结构化输出（参考 qtdata CLI 的 LLM 转换模式）：

```bash
qtcloud-econ clarify --input 描述.md [--output 需求.json]   # 模糊业务描述 → 结构化需求
qtcloud-econ design --input 需求.json [--output 规格.json]  # 需求 → 机制规格（对齐领域模型）
```

- **src/llm.rs**：DeepSeek API（OpenAI 兼容）blocking 调用——`reqwest`(blocking,json) POST `https://api.deepseek.com/chat/completions`，`response_format = {"type":"json_object"}`，temperature 0.1；key 从 `DEEPSEEK_API_KEY` 环境变量取；返回 JSON 字符串
- **命令设计依据 context 文档**（重要）：命令规格写在领域仓库的 `data/context/`（如 `default/clarify-models.md`、`default/design-models.md`——思维模式描述）。实现前先读 context，把思维模式（如"识别直觉语言下的机制设计骨架"）落成 system prompt 的结构化输出契约
- 结构化输出对齐领域模型（design 输出 = Mechanism 四要素 players/strategies/rules/objectives，可被 list/show 直接消费）
- 结构：每命令一个模块（`src/clarify.rs`/`src/design.rs`），`SYSTEM_PROMPT` 常量 + 输入输出结构体（Serialize+Deserialize）+ roundtrip 单元测试；main.rs 加 `Clarify{--input,--output}`/`Design{...}` 子命令，`write_output` 辅助（写文件 + stdout）
- **运行需 `export DEEPSEEK_API_KEY`**；无 key 时构建/测试仍全绿（roundtrip 测试不调 LLM），端到端留待有 key 时跑

### 命令归属设计决策（用户确认，2026-08-09）

clarify/design 等**流程命令独立顶层**（不挂在 mechanism 领域子命令下）：

```bash
qtcloud-econ clarify    # 流程轴：需求澄清（全局）
qtcloud-econ design     # 流程轴：规格设计（--type mechanism/game/contract 支持未来领域）
qtcloud-econ mechanism  # 领域轴：实体管理（list/show）
```

理由（用户认可）：
- **两轴分离**：clarify/design 是流程轴（数据工程阶段：澄清→设计→实现→交付），mechanism 是领域轴（实体）——不同维度不嵌套
- **未来拓展**：新领域（course/pricing 博弈）复用同一流程命令，流程可扩展（simulate/analyze/library）
- **qtdata 先例**：流程命令在顶层（Blueprint/Scope/Quotation/Delivery）

### context 文档结构（用户纠正："不应该写 README，重写"）

领域仓库 `data/context/` 的 README.md **只做入口**（领域定位/数据工程流程/文档导航表），长内容放独立文档：

```
data/context/
├── README.md                    # 入口 + 文档导航
└── default/                     # 内容文档（clarify-models/design-models/deliverables）
```

交付物边界等扩展内容不要写进 README——建 `default/deliverables.md` 独立承载，README 加导航行即可。

### 聚合目录四块形态：models / events / services / repos.rs（2026-09-21 qtcloud-work 定稿方向）

用户提议聚合内部按四块切。**它照规范三块长出来的**——规范每个实体就是 `## 领域属性` / `## 领域事件` / `## API端点`（工单多一节 `## 命令行工具API端点`），第四块是规范故意不写的那半（\"位置不进模型，由平台给\"）：

| 块 | 装什么（判据） | 不许出现什么 |
|---|---|---|
| `models/` | 领域属性：字段、身份，以及**模型自己的规矩**（校验、推导、落点算式） | 不碰盘、不发事件、不编排 |
| `events/` | 领域事件：事件名 + 负载形状（逐字对规范·领域事件） | 不落盘（落盘统一由根 `events.rs` 做） |
| `services/` | 这个聚合能端出的事（API 端点）：一个动作一件，编排 models + repos + 发事件，返回 `Outcome` | 不定义新字段、不直接读文件 |
| `repos.rs` | 这个聚合的存取：拿 `locate` 给的位置，搬字节与行 | 不判业务规矩（规矩在 `models/`） |
| `mod.rs` | 只有类型与出口 | 不放实现 |

依赖方向随之定死（可测）：`models ← events ← services → repos`；**models 不认识盘、不认识端点**。这条正好补上 STATUS 长期记的\"规则只活在人的记忆里\"——可写成契约测试断言（`models/` 下不许出现 `crate::locate` / `repos` / `services` / `events`）。

三条边界（用户当场圈定的，容易收过头）：

- **`services` 一词撞名**：本库\"领域服务\"已专指跨聚合只做一件事的那类（顶层 `search/`、`audit/`，见 CONTRIBUTING 三类）。聚合内第三块再叫 `services` 就是同词两义——建议叫 `actions/`（本库现名，代码与文档里\"动作\"已立住）或照规范原文叫 `endpoints/`；**命名归用户，先问再动**。
- **四块是判据清单，不是建目录要求**：定义型聚合（如 `artifact/`）只有 `models/` 有内容——不为凑齐造空壳（`go`/`python`/`ts` 三个空壳的账还在）。写成\"一件东西先问是哪块，没有就不建\"。
- **仓储边界：装载给位置、仓储管读写**。属于\"这台机器上这本账\"的规则（`workspace_key()` / `account()` / 账本 XDG 缺省）留在 `locate/`，不进 `repos.rs`——那是装载的事，不是聚合内容。

整理顺序（每步搬文件 + 一次提交 + 双二进制对照）：① 判据落进 CONTRIBUTING 与 dev-guide 落点图（拍板后才动）→ ② 试点 `workflow/`（四块齐、拆法典型）→ ③ `artifact/` → `workspace/` → `order/` → ④ 契约测试从按文件列清单改成**按块的依赖断言** + STATUS 复检。**唯一要拆逻辑的是 `order/`**：`Order` 现在既装字段又落盘（`save`/`ensure` 在 `impl Order` 里），得切成 `models/`（纯字段与判定）+ `repos.rs`（读写）。

逐聚合的现有文件映射（含 `workspace/` 的四件定位与 `locate/` 里该收回的部分）：`references/cli-aggregate-layout.md`。

## Go Provider 模式（src/provider）

参考：qtcloud-delib/src/provider（模型单一事实源 + Repository + GORM 双引擎）。最小骨架（种子数据文件加载，无需 DB）：

```
src/provider/
├── go.mod                      # module github.com/quanttide/<app>-provider; go 1.26
├── cmd/server/main.go          # HTTP 服务：ServeMux + PathValue（Go 1.22+ 路由模式）
├── internal/<domain>/          # 领域包（对齐种子数据 JSON 字段）
│   ├── model.go                # 模型（json tag 与种子数据一致）
│   ├── repository.go           # Repository 接口 + FileRepository（从 data/mechanisms.json 加载）
│   └── handler.go              # HTTP 处理器（List/Show，404 处理）
└── data/mechanisms.json        # 独立种子数据
```

- **main.go**：`mux.HandleFunc("GET /api/<domain>s", h.List)` + `mux.HandleFunc("GET /api/<domain>s/{id}", h.Show)`；端口用环境变量覆盖（如 `QECON_ADDR`，默认 :8080）
- **handler**：`writeJSON` 辅助（Content-Type + status + json.Encoder）
- **验证**：`go mod tidy && go build ./... && go vet ./...`；实测：`(QECON_ADDR=:18080 go run ./cmd/server &)` + `curl` 检查 200/404（**lint 单文件 vet 报 undefined 是同包文件未就绪的误报，go build 通过即 OK**）
- README.md（API 表/结构/运行）+ CHANGELOG.md（v0.1.0）
- 演进方向（参考 qtcloud-delib）：Repository + GORM 双引擎（开发 SQLite / 生产 PostgreSQL）、cmd/server 依赖组装

### Provider 存储抽象（Store 接口，2026-08-21 qtcloud-agent 新增）

外部数据源型 provider 演进：不直接 os 操作，抽 `internal/store` 包——**数据源可插拔**（本地文件系统默认 + OSS 预留），Repository 只依赖接口：

```go
// internal/store/store.go —— 接口（read/write/list，按相对路径）
type Store interface {
    ReadFile(path string) ([]byte, error)
    WriteFile(path string, data []byte) error
    ListDir(path string) ([]Entry, error)   // Entry{Name, IsDir, Size, ModTime}
    IsDir(path string) bool
}
// local.go: NewLocalStore(root) 文件系统实现（默认）
// oss.go:   OSSStore——阿里云 OSS 实现（2026-08-21 已完整实现，非占位）
```

- **运行切换**：环境变量 `QTCLOUD_AGENT_STORE`（local 默认 / oss）——main.go 组装处 switch 选择 store 实现，Repository 代码零改动
- **OSSStore 实现要点**（引 `github.com/aliyun/aliyun-oss-go-sdk/oss`）：目录语义用 `ListObjectsV2(Prefix=p+"/", Delimiter="/")` 模拟——CommonPrefixes=子目录、Objects=文件（TrimPrefix 去前缀）；ReadFile/WriteFile=GetObject/PutObject（同 key 覆盖）；IsDir=前缀探测；凭证从环境变量（ALIYUN_OSS_BUCKET/ENDPOINT/ACCESS_KEY_ID/SECRET）
- **SDK 下载坑**：proxy.golang.org 超时 → `GOPROXY=https://goproxy.cn,direct go get ...`
- **main.go 组装**：`agentStore := store.NewLocalStore(registry)` → `agent.NewRepository(agentStore)`；切换数据源只改这一行（OSS 时 `store.NewOSSStoreFromEnv()`）
- **测试注入**：`NewRepository(store.NewLocalStore(t.TempDir()+"/..."))`——临时目录造数据，无需 mock 接口；本地模式即测试模式
- **坑：store.Entry 是字段不是方法**——用 `e.Name`（非 `e.Name()`，os.DirEntry 风格会写错）
- 递归大小（dirSize）走数据源 `ListDir`（测试 `repo.dirSize("")` 测整棵树）——不在 Repository 留裸 os 调用
- 与 qtfounder 直读文件系统版的关系：Store 抽象是它的接口化——同一"数据源即文件系统/仓库"思想，多一个可插拔位

### Provider 数据存储：FC 无持久化 → OSS 每表单对象（2026-08-26 qtcloud-course 实证）

**FC 3.0 容器无持久化**（disk 512MB 临时盘——实例回收数据丢失）——provider 持久化**不能用本地 SQLite/文件**。解法：**OSS 对象存储**（容器可读写、数据持久）：

- **存储设计：每表一个对象**（`{table}.json`——如 `programs.json` 全量实体列表）——读时 Get 整个对象解析、写时全量覆盖原子写；对齐 qtcloud-crowd 的 OSS store 模式（单文档全量读写）
- **懒加载缓存**：首次读时从 OSS 拉取 → 内存 map + 自增 seq（续号——重启后 seq 从现有数据最大值续，不重置）；写时同步回 OSS
- **配置**：`QTCLOUD_<APP>_STORE=oss|memory`（默认 memory——测试）+ `QTCLOUD_OSS_BUCKET/ENDPOINT/ACCESS_KEY_ID/SECRET`（对齐 QTCLOUD_OSS_* 前缀惯例；bucket 变量无默认值——部署时 tfvars/CI 提供，命名对齐 `<app>-provider`）
- **seed 适配**：`cmd/seed` 幂等 + SetID 固定 ID——Dockerfile 启动链 `seed → seed-catalog → server`（全 `STORE=oss`——部署即数据就绪）
- **IaC**：`oss.tf` 数据桶（私有）+ FC 环境变量注入（AK/SK 明文入 tfstate——注释说明与 crowd 现状一致；生产后续可用 FC 密钥管理）
- **测试**：mock OSS server（校验签名头 + 404 XML——对齐 crowd oss_test）；重启恢复测试（懒加载 + seq 续号）；store 覆盖率 80%+
- **升级文档**（docs/dev-guide/upgrade.md）：升级分**代码升级**（tag → deploy）与**内容升级**（profile 创作源头 → provider 种子数据 → seed 导入）两类——内容升级依赖持久化解决（OSS 后可靠）；写文档时明确"当前缺口/需要解决"如实标注

### Provider 两种形态

1. **种子数据型**（qtcloud-econ）：读内置 `data/<domain>s.json`（模型对齐种子字段），三端各自维护副本
2. **外部数据源型**（qtfounder 2026-08-15）：读外部仓库文件系统（fiction 章节 + memory 文档），为 Studio **Web 端提供真实数据通道**（Web 无文件系统）
   - 数据源环境变量与 Studio **共用同一变量**（`QTFOUNDER_FICTION_PATH`/`QTFOUNDER_MEMORY_PATH`）——单一配置，README 同步记录
   - 环境变量缺一即 `log.Fatal`（启动即报错，不静默降级）
   - 列表接口返回结构化条目（如章节 id/title/path，id 需数值排序——见 pitfalls）
   - 测试用真实数据源 + `t.Skipf` 区分"数据源不可用"与"读到空"

### Go Provider Pitfalls（qtfounder 2026-08-15 踩坑）

- **字符串排序 ≠ 数值排序**：章节/编号列表按字典序排序时 `"10_1" < "1_1"`（'1'='1' 后 '0'<'1'）——需解析编号为数值对再比较（`fmt.Sscanf(id, "%d_%d")` 拆两数，先比前再比后）
- **测试相对路径层级**：从 `src/provider/internal/<domain>/` 到仓库根逐级数（internal→provider→src→\<app\>→父仓库 = **5 级到 apps/、6 级到父仓库根**）；算错时可用 `python3 -c "import os; print(os.path.abspath(os.path.join('..','..',...)))"` 快速验证
- **单目录容错掩盖路径错误**：`os.ReadDir` 失败 `continue`（单目录缺失不阻断）会让列表函数静默返回空——测试里数据源路径错误时 `ListChapters` 报 err（可见）但 `ListMemoryDocs` 静默空（难查）。测试应对真实数据源用 `t.Skipf` 显式跳过（区分"数据源不可用"与"读到空"）
- 数据源环境变量缺一即 `log.Fatal`（启动即报错，不静默降级）——qtfounder 模式：`QTFOUNDER_FICTION_PATH`/`QTFOUNDER_MEMORY_PATH` 必填

## 页面设计原则：结构即界面（用户两次纠正"过度复杂"，2026-08-15）

qtfounder Studio 资产页/写作页设计中用户两次纠正"过度复杂了。直接按照现有的结构来写页面就可以"——核心原则：

- **页面结构 = 数据源目录结构的直接镜像**：资产页（小说/记忆）就是目录树（一级文件夹→二级文件夹→文件），不发明任何抽象视图。被否定的发明：漏斗、缺口、矩阵、素材池、当前焦点、方法速查、检索/过滤（"不发明视图：不搞漏斗/缺口/检索等抽象——目录结构本身就是视图"）
- **资产页只读 + 发现**：浏览/阅读/状态可见；一切写动作（编辑/创建/改稿）只属于写作职能页——资产页可以提供跳转引导，不承载创作动作
- **工作流要显性化**（创作页）：不是"同一棵树+可写"，而是**流程视图**——阶段条（0_日志→1_灵感→2_脚本→3_初稿→4_改稿，含语义+计数）+ 文件挂载 + 推进动作（移动到下一阶段）。"显性化"= 把流程本身呈现出来，文件位置即流程位置（真实目录状态，不发明状态字段）
- **结构规则准确化**（用户给的资产结构规则）：一级文件夹=一本小说、二级文件夹=一个阶段、文件编号=排序、无编号=未排序（素材池特征）、同名"2"后缀=同排序位置的不同版本——页面照实显示各线实际目录名（校园言情用 1_素材/2_提纲 也不同），不统一命名
- **契约驱动自动适配**：新增目录（如 0_日志 阶段）引擎零改动自动出现——这正是"结构即界面"的机制保障

### 导航栏三职能 = 业务闭环三环（qtfounder/qthealth 2026-08-16）

导航栏整理模式：**三职能平级（无分组），每职能对应业务闭环的一环**，各回答一个问题：

| 应用 | 职能 | 回答 |
|------|------|------|
| qtfounder | 资产（有什么）/ 写作（做）/ 思考（想） | 结构 / 动作 / 状态 |
| qthealth | 记录（观测）/ 状态（反馈）/ 练习（干预） | 发生什么 / 怎么样 / 该做什么 |
| qtcloud-agent | 会话（观测）/ 技能（制作）/ 智能体（治理） | 看过什么 / 沉淀什么 / 管什么 |

- **导航项 = 用户旅程环节，不是功能列表**：qtcloud-health 的"首页/写日记/心理测试"（工具视角）→ qthealth 的"记录/状态/练习"（旅程视角）；用户确认方向后直接改 app_shell 的 destinations + router（initialLocation 改新首页职能）
- **导航项统一名词（领域对象）**：用户纠正"概念上不统一，有动词有名词"（记录/状态/练习 中"记录""练习"动词感）——统一为全名词（日志/状态/练习——但"日志"又被否：**概念覆盖面要准**，日志只含日记型，定期量表也是观测数据 → 用"记录"（records：日记+行为+量表结果=观测一切）；准确 > 词性观感，名词性是消除动词感的手段而非目的
- **迁移时首页看板常是"反馈环"雏形**：home_screen（概览看板）git mv 并入"状态"职能（类名替换），不要保留"首页"概念
- **全屏/专注场景不占导航位**：危机干预、量表作答（全屏路由，安全网/专注路径）；量表列表可作为状态页 AppBar 二级入口
- 练习等新职能页先用占位卡片页（列表 + SnackBar 占位），后续填内容
- 改导航后同步更新 widget_test 的导航项断言（文本/工具提示）

### Studio 按职能归位（代码结构 = 产品结构，2026-08-16 qthealth）

导航确定三职能后，lib/ 从功能分层（screens/blocs/views 平铺）整理为**按职能分包**——目录直接映射产品三环：

```
lib/
├── main / router / theme / constants     # 入口层
├── shell/app_shell.dart                  # 应用壳
├── record/     # 观测：表单 + cubit + state + card
├── status/     # 反馈：看板 + 量表页 + cubit + charts
├── practice/   # 干预：练习页 + 危机页
├── models/ + sources/ + services/        # 领域模型 / 本地存储 / 加工引擎（不按职能分）
```

- **归位法**：`git mv`（保历史）+ **脚本批量重写相对 import**——对每个移动文件：解析 `import 'rel';` 行 → 基于旧位置 resolve 目标 → 基于新位置重算相对路径（os.path.relpath）→ 替换；未移动文件（router/main）与移动后失效的引用手动修（`screens/x → record|status|practice/x`、`views/app_shell → shell/app_shell`、`../blocs/x → ./x 或 ../record/x`）
- **测试文件同步**：`package:studio/blocs/...` 等包路径引用也要批量改（注意 record 相关 cubit 指向 `record/`、assessment 指向 `status/`——sed 一把梭会把 record 的也改成 status，需二次修正）
- 归位后用 `flutter analyze` 找残余失效 import（报 uri_does_not_exist），全部有效后再跑测试
- 新功能（行为打卡）顺势补在对应职能目录（models/practice_log.dart + sources/practice_store.dart + practice 页"完成练习→记时长"）——功能归位而非另起目录

### Studio 按域分文件夹（2026-09-24 qtdata 定稿：四域 + app）

用户拍板的域布局：`lib/` 下 `project/ data/ business/ asset/` 四域 + 跨域 `app/`（原 shared，用户改名），每域内部保持 `models/ views/ screens/` 三层，`main.dart` 与 `test/helpers/` 留根；域判据来自数据形状（交付资产矩阵的行键就是 project/data/business，矩阵与交付物归 `asset/`）。规矩：**一个文件只属一个域，进 `app/` 前先确认它真被两个以上域用到**（本轮据此把 `progress_bar_widget` 留在 project，9 件跨域件进 app）。

两条最容易踩的：

- **import 两类都要重写**：`lib/` 内是相对 import，`test/` 常用 `package:<pkg>/...` ——只处理一类会炸出一片 `uri_does_not_exist`；且映射表要按全量文件建（漏一个 `dashboard_screen_test.dart`，报 61 个错才暴露）
- **拿「框架家法」当理由改名之前先取证**：Bloc 官方示例是 `view/`（页面）+ `widgets/`（部件），全仓 **0 处 `views/`**——我们契约那条「按 Bloc 惯例该叫 views/」不成立，`views/` 是**本项目约定**。用户裁决：维持自己规范，但**偏离要显式成文，不许挂在家法名下**；被要求「维持我的规范」后不要再拿家法劝第二次

迁移配方（映射表 → `git mv` → 相对/package 双轨 import 重写 → 拆类的连带 import → 清空目录 → 门禁定位残余）、「行为不变」的四组可比信号（`expect(` 计数 / 中文字面量集合 / 类名差集 / 最长行数）、文档同步清单与 pitfalls：`references/flutter-lib-domain-layout.md`

**搬迁 + 门禁 + CI 校验这类多步作业要中途报进度**——用户会在沉默过久时直接催「怎么还没好」（2026-09-24 实录）；每完成一步（搬迁完 / 门禁绿 / CI 绿）给一行状态，别攒到最后一次性汇报。

### Studio 独立数据层（qtcloud-agent 2026-08-21，用户要求"不要依赖 cli，独立管理数据"）

外部数据源型 studio 的两种数据接入方式，**优先独立数据层**（用户纠正了先做的 CLI 调用版）：

- ❌ **调 CLI**：`Process.run('<app>', args)` 解析输出——重复解析层 + 依赖 CLI 安装（首个版本做了，用户否决"不要依赖 cli，独立管理数据"）
- ✅ **独立数据层**：models + repositories 直读文件系统（与 provider 同一数据源、同一解析逻辑），环境变量可覆盖：

```
lib/
├── models/          # 领域模型（agent.dart: AgentStatus/AgentFile 状态枚举；session.dart: SessionGroup/SessionItem）
├── repositories/    # 独立数据层（agent_repository.dart / session_repository.dart——不调 CLI，直读文件系统）
└── main.dart        # 壳 + 三职能导航 + 各职能页
```

- **数据源环境变量**（与 provider/CLI 共用同一变量）：`QTCLOUD_AGENT_REGISTRY`（注册中心，默认 data/profile）、`DSH_HOME`（DSH 数据，默认 ~/.dsh）
- **解析逻辑必须对齐 CLI 实现**（读 CLI 源码而非猜）：agent_local_dir 映射（zed→~/.config/zed 等）、skip_file（README/LICENSE/.gitkeep）、JSONC 清理 + JSON 展平对比、`decode_workspace`（`--tmp--`→临时工作区、`--path--`→strip 双横线后 '-'→'/'，**无前导斜杠**）、projcache 路径（`storages/session_projcache.json` + `/tables/sessions/{sid}/identity/cwd`）——Dart/Go 各自重实现但逻辑一致
- 页面即数据展示：会话页（工作区分组列表）、智能体页（ExpansionTile 展开 ok/diff/missing）、技能页（占位卡片列最小循环步骤）
- 独立数据层的好处：本地测试注入临时目录、Web 端可跑、不依赖 CLI 二进制

### 纯 Dart 包模块划分（toolkit 2026-09-26，两轮纠正后定稿）

非 Flutter 的纯 Dart 库（如 quanttide-founder-toolkit）分层按 Bloc 两原则：**feature-first（域优先）+ 域内数据与逻辑分离**：

```
lib/src/
├── core/            跨域共享（parse/rules/engine/llm）——域依赖 core，core 不依赖域
├── <域>/（memory/ fiction/…）
│   ├── models.dart      域模型（纯数据，无 I/O）
│   ├── repository.dart  装载与发现（I/O）
│   └── bloc.dart        业务逻辑（事件进 → 状态出）
assets/
├── artifacts/       名词定义（YAML：一个名词的字段、从哪提取）
└── workflows/       动词定义（YAML：用引擎动词组装的任务 + 判断规则）
```

引擎动词（workflow 的 steps 只认这几个，写 docs 不建目录）：`scan`（关键词定位，代码）/ `judge`（LLM 判断）/ `merge`（策略合并，代码）——加名词=加 artifact YAML，加任务=加 workflow YAML，加动词=改引擎。规则是数据不是代码。

三个被纠正的坑：

- **分析词汇 ≠ 代码结构**：把分析用的分类学（六类计算）直接做成六个类＝「模块划分很奇怪」。分析里的动词才是代码里的类/函数，名词（artifacts）和任务（workflows）进 YAML
- **小包别炸目录**：十来个文件拆五个目录 =「这什么玩意」。一文件一角色、够装一组同角色文件才建目录；`models/` `repos/` 值得建目录（被依赖的层），parse/rules/compute 单文件即可
- **纯 Dart 包没有应用壳**：跨域共享层叫 `core/` 不叫 `app/`（`app/` 是 Flutter 应用壳的名字，照搬被问「app 文件夹为什么会需要」）；Bloc 语境里这层就叫 core/shared

完整的 artifacts/workflows 两层资产设计、引擎动词契约、以及坑（import 路径、私有成员跨文件）：`references/dart-toolkit-architecture.md`

### 多语言 monorepo 发布：scope tag + 包级 CHANGELOG（2026-09-27 toolkit 实证）

一个仓库装多语言包（dart / rust / go 各自 `packages/<lang>/`）时发布纪律：

- **tag 按包打 scope**：`dart/v0.1.0-alpha.1`——不是根级 `v0.1.0-alpha.1`（用户纠正："是 dart/v0.1.0-alpha.1，我等会还要写 Rust 和 go 的"）。`qtcloud-devops release audit/publish -v dart/vX.Y.Z` 直接支持 scope 格式
- **CHANGELOG 跟包走**：`packages/dart/CHANGELOG.md`（audit 找的是 scope 对应目录下的 CHANGELOG，根 CHANGELOG 不算）；**根 CHANGELOG 只做索引**（一行指向各包 CHANGELOG），不装具体版本内容（用户："改啊"）
- **打错 tag 的清理三连**：`git push origin :refs/tags/<tag>` + `gh release delete <tag> -y` + `git tag -d <tag>`，再用正确 scope 重发
- **pubspec version 同步**：audit 检查「配置文件一致性」，`packages/<lang>/` 的版本声明（pubspec.yaml / Cargo.toml / go.mod）要与 tag 的版本号一致

### Bloc 业务逻辑文档：doc/<域>.md（2026-09-27 toolkit 实证）

bloc 装域特有编排逻辑，文档按域拆：`packages/dart/doc/memory.md`（MemoryBloc 的四步流程/三问路由/证据分级/合并策略/边界）、`doc/fiction.md`（FictionBloc 的三步提炼/编号轴/阶段流转/包装文案/边界）。写法：每节「逻辑名 + 表格/步骤 + 边界」，把 Skill 提示词里的自然语言规则落成可实现的规格。通用逻辑归 toolkit `docs/`，域特定逻辑归 `packages/<lang>/doc/`——两层不重复，互相引用。

### 契约对齐三件套：STATUS / TODO / ROADMAP（2026-09-12 studio 实录）

既有代码仓按软件工程契约整理时的标准产出。用户指令模式：「1. 根据我们的软件工程契约写 STATUS.md。2. 根据张力写 TODO.md，详细列举需要修改的每一个细节，可以分阶段走。3. 根据 TODO 总结 ROADMAP.md 给人类理解和核对。**最后我只看 ROADMAP**。」

- **先定位尺子（硬前置）**：契约原型 `domains/quanttide-code/data/insight/code-agent/contract.md`（结构契约 / 依赖契约 / 阶段条款）＋ 平台契约 `domains/quanttide-code/docs/gallery/categories/platform/index.md`（角色技术栈 / 桶与域名 / IaC / 门禁）＋ 工作流 `domains/quanttide-work/docs/gallery/workflows/code-implement*.yaml`。**先读这三份再列条款，不凭记忆列**；`docs/dev-guide/stories/` 一类是需求故事（需求来源），不是工程契约——用户问「拿哪份契约当尺子」时默认答案就是上面三条，与 qtcloud-work/studio 那份同源、横向可比
- **三份文档各有读者**：STATUS = 数据（逐条契约 × 现状 × ✓/✗/—，每行带可比数字与出处）；TODO = 执行（按张力求段，每条「改什么／判据（命令行可跑的）／影响哪些文件」）；ROADMAP = 人——**因为「只看 ROADMAP」，它必须自足**（为什么做／目标／分段表：每段一句话 + 怎么算完／每段跑什么／现在在哪），细节靠链接下探另两份
- **张力 = 现状与契约的差**，逐条从契约条款反查：结构契约（命名随家法／混用／单文件行数／角色唯一／依赖单向／组装分离／测试同构／横切集中）、依赖契约（版本号非 path／官方 SDK／默认选型／许可清单／坑写进契约）、平台契约（角色技术栈／桶与域名／IaC 目录／门禁／可观测）
- **分段顺序**：零风险先做（正名 widgets→views）→ 工作流原定目标（收敛两套实现＝「交接」）→ 结构性重构（转聚合）→ 对外对齐 → 文档与工作流对齐 → 发布线
- **每段判据三连**：`flutter analyze && flutter test` + 对表脚本（`scripts/parity.sh`，24 条必须仍一致）——**行为不变是硬要求**
- **跑门禁前先确认工具链版本**（2026-09-24 qtdata 实证，同一次会话内先判反又纠正）：`bash -ic 'which flutter; flutter --version'`——同一台机器可能**并存两套 SDK**，`.bashrc` 可能指向旧的那套，而 **agent 自己 export 的 PATH ≠ 用户的 PATH**（拿它当「本地版本」会把「本地 vs CI 谁落后」判反）。正确收口顺序：**先用某套 SDK 把门禁跑绿（把版本写进评测记录）→ 把 CI 的 `flutter-version` pin 升到同一版 → 同步改 `.bashrc` 指向该版本 → `bash -ic 'which flutter'` 复验**（只改文件不验证等于没改）。非登录 shell 不读 `.bashrc`，命令写全路径；旧 SDK 占盘大，清理前先问用户
- **加严 `analysis_options.yaml` 不必怕，先跑再修**（同日实证）：`strict-casts` / `strict-inference` / `strict-raw-types` + `prefer_single_quotes` / `unawaited_futures` / `always_declare_return_types` 上到一个 21 文件 2 833 行的 studio 后，**首跑只命中 2 处推断失败**（`MaterialPageRoute` 与 `showDialog` 缺显式类型参数，补 `<void>` 即零告警），`dart format` 零改动——命中就改代码、不关规则；事前扫一遍裸泛型与无类型空集合能把预判说准大半
- **顺手查工作流自身的账**：工作流判据常与现状脱节（studio 的 `code-implement-studio.yaml` 交接判据查 `lib/cli`——该目录不存在，实际那层是 `repositories/client.dart` + `runner.dart`），体检时一并核对并在 TODO 里开一条
- **ROADMAP 必含「交付哪些模块」一节**（用户纠正：「ROADMAP 似乎没有明确交付哪些聚合文件夹」）——人只读 ROADMAP，所以**交付物的目录名要写死**：**分三类列**（聚合 / 领域服务 / 适配处 `platform/`；判据与命名规矩见 `quanttide-domain-design` 的「三类模块并列」节）＋ **界面侧不参与划分**（views/screens 按界面职责分，三类模块只约束软件内部那一侧）＋ 验收（目录都在、`local/` 下无散落单件、单文件 ≤250 行）。TODO 对应段（转聚合）的靶子与判据同步写同一份清单
- **模块清单从命令面反推，但命令面只给候选、不给归类**：读 CLI 的 `help` 页拿候选（工作区组 `find`/`catalog`/`audit`/`material`；工作流组 → `workflow/`；任务组 → `task/`；`health`/`help` 归 `platform/`），**未实现的命令也先占位**（`dispatch` 如实报「还没搬：<命令>」的照样列）；归类按判据定——**有自己的定义 = 聚合**（`catalog` 有定义：条目、名字索引、快照语义）、只跨聚合做一件事 = 领域服务（`search`、`audit`）、边界外 = 适配器。**别按词性归**（`catalog` 名动同形但取名词义 = 聚合）；**别造 `-er` 目录**（`-er` 是类名惯例，不是目录名惯例）
- **命令改名要同时开一条 TODO**：命名有争议时看承诺的行为（`find` 报位置 vs `search` 给集合；带 `--show` = 检索 → 改名 `search`，同时改 CLI 分发 / `help` 分组 / `api-references` / studio 的 `dispatch` 与模块目录名）；发布线已开 = 破坏性变更（版本号 + CHANGELOG），**未搬一侧还在时改最便宜**
- **同一套三件套适用于 CLI 端**（2026-09-12 用户下一句就是「类似的，cli 也需要一套」）——按端换的只有段划分与判据命令：studio 七段 + `flutter analyze && flutter test` + `scripts/parity.sh`；**cli 五段**（正名与归类 → 立目录 → 拆长文件 → 契约对齐 → 文档对齐）+ `cargo fmt --check && cargo clippy --locked && cargo test --locked` + `scripts/validate-usecases.sh`。**CLI 的体检形态与 studio 不同**：`src/` 是**平铺一层**（11 个 `.rs`），聚合件（`task` `workflow` `catalog` `artifact` `material`）、服务件（`catalog.rs` 里寄生的 search、`audit`）、适配件（`cli` `help` `prompts` `paths`，以及住在 `cli.rs` 里的 `health` 与工作区落点）**全部同层**——这是那条硬禁的另一种形态（不是"层名与聚合名混用"，而是"三类同层"）；另有两个聚合**没立**：`workspace`（`workspace_root`/`data_dir` 散在 `cli.rs`）、`search`（寄生在 `catalog.rs`）
- **CLI 侧顺手能给 studio 抄的三件**（体检时要对比两端，别只查一端）：① **用例对账** `validate-usecases.sh`——文档用例号（`docs/user-guide/*.md` 的 `## 用例 N`）与测试出处（`tests/*.rs` 的 `// 用例：N`）**两边集合必须相等**，正是契约"保证必须可判"的落地；② **发布纪律** `validate-version.sh` + `validate-changelog.sh` + `cargo publish --dry-run` 全在 CI；③ **门禁齐** fmt/clippy/test 三条。studio 那侧缺的 `dart format` 门禁与发布线照这套抄
- **两端体检时必查的四类跨端账**（cli 侧 2026-09-12 实测四条，§别的端同样适用）：① **工具库版本两侧对齐**（cli `0.1.0-beta.4` vs studio `^0.1.0-beta.5` 错开——而对表脚本要求两边算出同一个结果，版本必须同步）；② **两侧各有一份的共用件**（`prompts.rs` 与 `prompts.dart` 头注释互相打脸：各说"只是自己这一侧的说法"，要么合一要么写明差异）；③ **结构文档过时**（`docs/dev-guide/layers.md` 还说 `outcome.rs` 在 `src/`——实际已抽进工具箱，且整批写着"拟建"）；④ **`api-references/` 混两类**（命令参考 `task.md`/`workflow.md` 与概念参考 `workspace.md`）
- 完整检查清单（结构/依赖/平台/工作流四组）+ 分段细则 + 判据命令 + 聚合靶子清单与命令面结构 + studio 实测数据：`references/studio-contract-alignment.md`；**cli 侧实测数据、平铺形态、五段清单与跨端四账**：`references/cli-contract-alignment.md`
- **仓库级快照 vs 组件级三件套**：`<app>/STATUS.md`（仓库级：业务定位 / Scope 状态表 / 战略差距分析 / ROADMAP 进度 / 文档覆盖 / 已知不一致）与 `<app>/src/<组件>/STATUS.md`（组件级：条款 × 现状 × ✓/✗/—）是两份不同的东西。刷新流程、契约出处、过时点核法（tag 可能根本不存在、docs 目录可能已删、**差距分析的尺子可能已随新业务边界换掉**）、取数命令与 pitfalls：`references/status-doc-refresh.md`（2026-09-24 qtdata 实录）
- **存量件盘点（哪个可以删／归档／迁出）的纪律**：用户问「哪些过时可以删或归档、哪些不适合继续在这个文件夹维护」时，交的是**候选 + 在用证据 + 裁决去向**，不是删除建议——qtdata 那轮提的三个「可以删」被三个全否（平台目录要保留并完善、空 `integration_test/` 要维护、`doc/` 是页面分解产物不能删）。判死一个对象之前先找「谁在用它」的证据（`scripts/*.sh`、CI 构建了哪个平台、被谁 import、依赖声明在不在）。取证命令、死代码与孤儿测试的对表法、过时命名（现行代码 vs CHANGELOG 历史）的分法、没工具链时「能改的改、不能验证的落成 TODO」、以及裁决要写回三件套：`references/repo-inventory-survey.md`
- **三件套写完就直接委派执行**（用户下一句就是「分别启动两个 pi 进程，执行 cli 和 studio 的重构」）：TODO 的**段就是 pi 的任务书**（pi 无会话记忆，提示词点名先读 TODO/STATUS/ROADMAP 即可，不必复述细节）；两端各起一个 pi 进程并行，**各改各的子目录、互不越界**；每段门禁按端分（cli `cargo fmt/clippy/test` + `validate-usecases.sh`；studio `flutter analyze && flutter test`）；**跨侧对表（`parity.sh`）不让 pi 当门禁**——一端正被另一进程改时会误判，只跑 `--report`，全量对表由 Hermes 在两端都停下后跑；门禁与提交由 Hermes 收尾（pi 沙箱 gitdir 只读）。流程与提示词骨架见 `roadmap-stage-execution` 的「2c. 并行双端」节与 `templates/pi-refactor-prompt.md`

### 资产契约模式（qtfounder 2026-08-15，用户意图）

统一资产管理：**资产契约（yaml 配置文件承载语义规则）+ 资产目录（通用引擎）**——语义规则从代码移入配置，新增资产=写一个契约 yaml，零代码：

```yaml
# src/studio/assets/contracts/fiction.yaml
asset: fiction
label: 小说
root: fiction
ignore: [README.md, CHANGELOG.md, myst.yml]   # 仓库级文件不进树
levels:                                        # 层级语义（逐级目录）
  - key: novel
    label: 小说
  - key: stage
    label: 阶段
naming:                                        # 文件命名规则（编号=排序）
  pattern: '^(\d+)_(\d+)(?:_(.*?))?(?: (\d+))?$'
  title: '$3'
  sortKey: ['$1', '$2']
  version: '$4'
  unsorted: last                               # 无编号文件排末尾
```

- **memory.yaml 对照**：levels=[category, folder(optional)]（二级仅 journal 有）+ naming.pattern: null（自由命名）+ unsorted: natural
- **引擎实现两个真实坑**（asset_catalog_engine.dart）：
  1. **optional 级缺省**：下一级 optional 且当前目录无子目录 → 文件直接挂当前节点（注意检查**当前目录**内部的子目录，不是父目录的 children）
  2. **sortKey 数值比较**：`"1_1" > "10_1"`（字典序陷阱）——按 `_` 分段 int 比较（`_compareSortKey`）
- **测试**：widget 测试用 loader 注入（见下），引擎测试用真实数据源 + `TestWidgetsFlutterBinding.ensureInitialized()`（rootBundle 加载契约需要 binding）；路径从 src/studio 到父仓库根=4 级
- **扩展性边界**（用户确认的洞察，记录在 memory/insight）：契约覆盖"本地文件系统+目录树+文件名语义"的资产域——数据源形态/文件内容/非目录形态/写操作是边界，域内扩展性=配置性，不预埋未出现的需求（YAGNI）

### Flutter widget 测试与真实文件 IO（qtfounder 2026-08-15 踩坑）

- **pumpAndSettle 超时**：widget 里 initState 触发真实文件 IO（Directory.list）时，FakeAsync 不推进真实异步 → `pumpAndSettle` 超时（"timed out"）
- **标准解法：依赖注入**——页面接受 loader 参数（如 `CreativeDesk({chaptersLoader, memoryLoader})`），测试注入 fake；生产默认真实加载。这是测试约束催生的架构决策，比 runAsync 稳定
- 兜底：`tester.runAsync(() => Future.delayed(...))` + 有限 `pump`；验证"页面挂载"（标题或 CircularProgressIndicator 存在）而非"数据完成"
- 引擎单元测试（非 widget）：`TestWidgetsFlutterBinding.ensureInitialized()` 后 rootBundle 可用；Flutter 测试 cwd=项目根（相对路径从 src/studio 到父仓库=4 级，与 Go 的 6 级不同）

## 应用部署（CI + IaC，2026-08-16 qthealth 首次上线）

qtcloud-* 应用工程化闭环 = CI（.github/workflows）+ IaC（manifests/terraform）：

- **CI 四件**：`ci-verify.yml`（push/PR：四端 cargo/go/flutter/npm 验证）+ `deploy-provider.yml`（provider/* tag：镜像 + terraform apply）+ `deploy-studio.yml`（studio/* tag：build web + OSS + CDN）+ `deploy-site.yml`（site/* tag）——部署触发都按 tag 前缀（`provider/**`、`studio/**`、`site/**`），避免组件 Release 互误触发
- **IaC**：完整 terraform 声明（providers/oss/cdn/fc/platform/locals/variables/outputs + tfvars.example）——复制 qtcloud-secret 模板适配（改名 + 域名 + **删 JWT/密钥相关**——无鉴权 provider 不需要）；`terraform fmt -check` 通过 + 残留 grep 校验
- **首次部署用 `-target`** 只建本应用资源（OSS 桶/CDN/DNS），绕过跨应用共享 RAM 角色冲突（EntityAlreadyExists）
- **多站部署**（同应用 site+studio 各自域名）：cdn.tf 复制域名块生成新资源组（独立 configs + DNS 记录）+ workflow 补 terraform apply -target 步骤；两层子域（studio.health.…）需 acme.sh 单域名签发（`--dns dns_ali`，account.conf 已有阿里 DNS 凭证）——见 reference 多站节
- **批量替换域名的 DNS rr 坑**（qtcloud-agent 2026-08-21 双站）：sed 全局替换 `health.example.com → agent.cloud.example.com` 时，**DNS 记录的独立 rr 值（"health"/"studio.health"）不会被完整域名串替换覆盖**——terraform apply 建了 CDN 域名但 DNS 记录 rr 仍是旧前缀（NXDOMAIN）；需手工改 rr（"agent.cloud"/"studio.agent.cloud"）后重新 apply。替换后 grep `rr =` 核对
- **site 桶资源缺失坑**（同上）：模板只有 studio 桶——site 的 `-target` 不含桶 → 上传 NoSuchBucket；补 `site-bucket.tf` + deploy-site target 加 `alicloud_oss_bucket.site`
- **CDN 源站桶名必须与上传目标一致**（模板复制易错：源站 `<app>-studio` vs 上传 `<app>-site` → 404）
- 证书：acme.sh 泛域名证书 `*.example.com` 匹配**单层**子域；`aliyun cdn SetCdnDomainSSLCertificate` 绑定（PEM 路径含字面星号需引号保护）
- 完整排障链（4 轮 CI 实录：setup-ossutil 版本 / terraform 缺失 / apply 交互卡死 / RAM 冲突 / 404 源站错桶）：`references/ci-iac-deploy.md`
- **门禁进 CI：一个文件两个作业，且必须读 CI 真跑结果**（2026-09-24 qtdata 落地）——`quality-gates` 在分支/PR 上跑 `dart format --set-exit-if-changed` + `flutter analyze` + `flutter test`，`build-and-deploy` 用 `needs` 串它、再用 `if: startsWith(github.ref, 'refs/tags/studio/')` 只在 tag 上部署（先例 qtcloud-work 的 `release-studio.yml`；qtdata 沿用 `deploy-studio.yml` 这个名字，**别改名**）。三条本地绿**不算完**：去 GitHub 读真跑结果——公开仓的 Actions 走未认证 API 就读得到，逐作业逐步骤看到门禁全 success、部署作业 `skipped` 才算过（`gh` token 过期不影响这条）。workflow 骨架、`paths` 过滤对 tag 同样生效的坑、轮询与判读脚本、以及「先例缺 `dart format` 半条门禁」这类跨仓不一致（要问用户别自行动手）：`references/ci-iac-deploy.md` 的「门禁进 CI」「验证 CI 真跑了」两节

## 应用仓库文档体系（qtfounder 模式，2026-08-15）

应用仓库（非领域仓库）的文档分层——用户认可的组织方式：

```
<app-repo>/
├── README.md        # 入口：定位 + 模块索引 + 快速开始 + 文档导航（瘦身，不承载详细内容）
├── STATUS.md        # 状态报告：模块版本/数据源/验证/数据流（时效性，随提交更新）
├── CONTRIBUTING.md  # 贡献规范：提交约定/验证门禁/分层提交/文档同步（规则，不承载配置细节）
├── AGENTS.md        # AI 工作指南：模块结构/工作原则/验证命令/提交规范
├── CHANGELOG.md     # 版本记录
└── src/<module>/ROADMAP.md  # 每模块规划（目标→任务清单→验证标准）
```

- **消灭重复**：详细配置收敛到 dev-guide（数据源配置是开发环境搭建的一部分），README 不重复（用户接受瘦身——曾因 README 塞长内容被纠正"不应该写 README"）
- STATUS.md 参考 quanttide-data 的 STATUS.md 格式（更新日期 + 模块版本表 + 状态列）；**刷新流程与过时点核法**见 `references/status-doc-refresh.md`——旧 STATUS 里的版本号、tag、文档目录都不可信，逐条从 git 与工程文件核；且「过时就更新」这类指令要先报告过时点再改
- 优先级：先补缺的（AGENTS.md + 缺 ROADMAP 的模块），再做 README 去重
- 四模块变体：qtfounder = site（React）/ cli / studio / provider——同文档体系适用

### docs/ 三分类（用户指定，2026-08-15）

根目录 `docs/` 按**读者**分三类，各写 index.md：

```
docs/
├── user-guide/index.md       # 用户指南：产品入口 + 使用步骤 + 数据源说明
├── dev-guide/index.md        # 开发指南：模块结构 + 配置 + 验证命令 + 工作流
└── api-reference/index.md    # API 参考：总体构成导航
```

- 每目录 index.md 是入口（概览表 + 指向详情文件），README 文档导航表注册
- **api-reference 按"服务/工具"分解**（用户指定）：index.md 只做总体构成（Provider HTTP API + CLI 命令两类概览），详情拆独立文件（`provider.md` / `cli.md`）——不要把所有端点塞进一个 index
- cli.md 参考：命令 + 参数表（默认值/说明）+ 输出结构（JSON 字段表）

### 文档叙事：按读者问题序，禁自造概念（2026-09-27 toolkit 三轮纠正）

写 toolkit/产品文档时用户三次纠正：「从开发者视角切换到读者视角」「整体叙事逻辑有什么缺点」「自造口语化概念不要」。定型：

- **叙事按读者问题序**（问题→洞察→解法→验证→落地），不按架构序（功能→管线→规则→包）——最有价值的洞察（如「memory 与 fiction 结构相同」）放论据位置，不当附录
- **禁自造口语化概念**：「同构」「三层管线」「读懂层」被否——用平实说法（「同一套组织方式」「分三步」「语法解析/结构切分/语义提取」）
- **本质≠手段**：「让 AI 理解语义」是手段，「人机交互框架：人写进去，AI 读得懂、算得动」才是本质——先问「这解决什么问题」再写「怎么做」
- **通用逻辑与特定逻辑分层**：toolkit `docs/` 记通用逻辑（任何语言实现都适用），`packages/<lang>/doc/` 记特定逻辑（目录结构/类职责/命令）——两层互相引用不重复
- **一句话定位法**：用户要求「一句话给目标读者介绍」时，先给手段级答案再被推到本质级——提前想清楚「本质是什么」（人机交互框架 > 读懂层 > 解析工具）

### 三类文档的读者定位（用户两次纠正，2026-08-15）

写 docs 前先明确"读者是谁"，三类文档用不同语言：

| 文档 | 读者 | 语言与结构 |
|------|------|-----------|
| user-guide | **使用者**（非开发者） | 平实场景语言（"你能用它做什么"）→ 打开步骤 → 看到什么 → FAQ；避免 dart-define/环境变量等术语（纠正"不够友好"：命令表/数据源布局是开发者视角，使用者看不懂） |
| dev-guide | **开发者** | 模块结构 + 环境搭建（前置要求 + 数据源配置）+ 开发命令 + 调试提示（开发知识，怎么做） |
| CONTRIBUTING | **贡献者** | 贡献规范（规则，必须遵守）：提交约定/验证门禁/分层提交/文档同步要求（纠正"区分不开"：CONTRIBUTING=规则，dev-guide=知识） |

**CONTRIBUTING vs dev-guide 职责划分**（用户纠正"区分不开"）：CONTRIBUTING = 规则（提交约定、验证门禁、分层提交、环境变量命名一致），dev-guide = 知识（结构、环境、命令、调试）。**数据源配置归 dev-guide**（开发环境搭建的一部分），CONTRIBUTING 开头一句话区分定位 + 指向 dev-guide，不重复配置内容；README 导航措辞同步（CONTRIBUTING 描述改为"贡献规范"）。

### 仓库内 SKILL 的位置与结构（用户纠正，2026-08-17）

给量潮仓库（应用/领域/data 子仓库）写可复用技能时：

- **位置**：`<repo>/.agents/skills/<skill-name>/SKILL.md`——不是仓库根（用户纠正：SKILL 正确位置是 .agents/skills/<name>/ 目录，git mv 保留历史即可）；写进仓库 = 随仓库分发，Agent 在该仓库工作时 AGENTS.md 指引加载
- **结构：三段论**（用户明确要求"SKill 三段论：目标、流程、验收标准"）——目标（一句话使命）/ 流程（编号步骤，可拆阶段）/ 验收标准（必须满足 + 质量检查 + 反模式表）
- 流程可再拆：如用户故事地图 SKILL 把"任务"与"故事/细节"拆两个阶段（任务是概括一句话，故事是"用户要…"细节展开，一个任务下多个故事）
- 验收标准要含**主线清晰**（读者能理解"产品解决什么问题"）+ **符合目标用户视角**（不是开发者/系统视角）
- 范本：quanttide-product/data/profile/.agents/skills/product-requirement/SKILL.md（产品云为完整示例）

### 意图/规划类文档的叙事化（memory/intention 教训，2026-08-15）

用户要求"梳理叙事逻辑，增强文档对人类的可读性"后，第一版叙事化被纠正"不够准确地表达意图"——叙事化时焦点偏移的教训：

- **叙事弧线不能牺牲意图准确性**：把规划文档写成"个人的故事"（起点=情绪/疗愈）会丢失"规划"的意图性。正确的叙事起点是**规划前提**（如"创作需要系统承载"——可持续/可积累/可表达），不是情感动机
- **"长出来的" vs "有意的系统设计"**：说工具是"长出来的"（被动演化）削弱了规划的主动性——用户确认的规划意图要用"有意的系统设计"（每层解决一个明确意图）
- 叙事化结构（用户认可）：前提 → 核心（架构=有意的设计）→ 分点意图（每层回答什么）→ 闭环 → 归宿/同构——用"为什么规划/规划了什么/意图是什么"组织，而非"作者的故事"
- 叙事化的目的是让读者跟着弧线走，但**每个节点的措辞必须忠于意图本身**（第一次重写把"疗愈故事"当起点就是偏移）；分点意图用动作短语（可观测/可积累/可感知/可表达）

## 讲代码/模块给用户听（2026-09-21 实证：一句「看不懂，重说」）

用户是作者本人，问「X 模块是做什么的」时他要知道的是**这东西干嘛用**，不是它的架构定位。第一版用抽象标签（「静态核对」「纯字符串计算」）＋贴函数签名讲 `place`/`check`，被回「看不懂，重说」。改法：

- **先跑一次真命令、贴真输出**（用真实夹具跑一遍 CLI），再逐行读那段输出——真例子比定义快十倍；
- 一句话说它是干嘛的（「算文件名的」「挑毛病的」），紧跟一个真实例子（`report/review-ui-shot.md`）——**不贴代码片段当解释**；
- 讲真输出时要敢说它**打错了一半**（如启发式的误报），把输出的可疑处当结论讲，不照本宣科；
- 收尾给「这功能的问题在哪」（如判据在规范里没出处），比补一串建议有用；
- **用户收窄范围时严格跟着收窄**（「只看 workspace，其他忽略不计」）——别再铺全局方案。被问「应该写什么」时答三段：判据（每块装什么/不许装什么）＋ 逐件映射（现有文件去哪）＋ 边界（别收过头），别扩成其他聚合的清单。

## 三端数据流

```
种子数据（各自维护）→ cli（命令行展示）/ provider（HTTP API）/ studio（页面）
```

三端模型字段对齐同一份种子数据 JSON 结构（id/name/description + 领域字段），改动模型先对齐种子数据。

## 关联

- Flutter Studio 端（create/测试/CI/IaC）：`quanttide-flutter-studio`
- 双轴架构挂载（领域轴+平台轴）：`quanttide-git-ops`
- 治理数据流水线与 mock 边界：`quanttide-delib-governance`
- qtcloud-code 重设计方向：**约束驱动生成**（constraint-driven generation）——约束先行→AI 生成→audit（对齐审计）→问题清单反馈→AI 修正。与传统"事后校验"的区别：约束是生成任务的规格输入（AI 在约束下写），不是旁观的门禁（AI 写完才查）。audit 校验代码↔测试↔文档三边对齐；review 校验质量；reflect/refactor 降级/移除。详细见 `references/code-doc-code-loop.md`
