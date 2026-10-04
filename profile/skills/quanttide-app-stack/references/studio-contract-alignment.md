# Studio 契约对齐实录（qtcloud-work/studio，2026-09-12）

用户指令模式：**「1. 根据我们的软件工程契约写 STATUS.md。2. 根据张力写 TODO.md，详细列举需要修改的每一个细节，可以分阶段走。3. 根据 TODO 总结 ROADMAP.md 给人类理解和核对。最后我只看 ROADMAP。」**

落点：`apps/<app>/src/studio/{STATUS,TODO,ROADMAP}.md`（ROADMAP 已存在则重写，保留其中仍然有效的内容如「现在能用的 / 接口还没有的 / 文档欠账 / 发布线」）。

## 三份文档的读者分工

| 文件 | 读者 | 形态 |
|---|---|---|
| STATUS.md | AI / 数据 | 逐条契约 × 现状 × ✓/✗/—，每行带可比数字（文件数、行数、条数）与出处；末尾「与工作流的对照」核对工作流判据是否与现状一致 |
| TODO.md | 执行者 | 按张力求**段**，每条写清：**改什么 / 判据（尽量是命令行可跑的） / 影响哪些文件**；开头写通用判据三连 |
| ROADMAP.md | 人（用户只看这份） | **必须自足**：为什么做 / 目标 / 分段表（每段一句话 + 怎么算完）/ 每段跑什么 / 现在在哪 / 与结构无关的功能欠账。细节靠链接下探 STATUS 与 TODO |

## ROADMAP 必含「交付哪些模块」一节（用户纠正：\"ROADMAP 似乎没有明确交付哪些聚合文件夹\"）

人只读 ROADMAP，所以**交付物的目录名必须写死在那份里**——只写\"按聚合重排\"等于没写。该节四件：

1. **模块表（分三类列）**：聚合 ｜ 领域服务 ｜ 适配处——各行「目录 ｜ 管什么 ｜ 现状（已有／命令面已有未搬／已有待拆）」（初版只列一张\"聚合\"表，被用户两处纠正：`catalog` 是聚合、`audit` 是服务）
2. **职能件收进一处**：`platform/`——命令面（`dispatch`）、导览（`help`）、判据执行（`rules`）、给智能体的话术（`prompts`）、YAML 解析、平台读写（`fs/`、`host/`）
3. **界面侧不按聚合**：`views/`（可复用的块）与 `screens/`（三屏）按界面职责分——**聚合约束的是软件内部那一侧，不是界面**
4. **验收**：上述目录都在、`local/` 下不再有散落单件、单文件 ≤250 行

TODO 里「转聚合」那段的靶子与判据写同一份清单，两处保持一致。

### 模块清单怎么定：命令面给候选，归类按判据（2026-09-12 用户两次纠正后的正确形态）

读 CLI 的 `help` 页拿**候选**（它按用途分组），但**命令面只给候选、不给归类**——每条按「有没有自己的定义」定类：

| help 分组 | 命令 | 归类 | 落成目录 |
|---|---|---|---|
| 工作区 | `search`（按名找文档，**原名 `find`**）/ `catalog`（按资产种类清点）/ `audit`（审计资产表与工作区）/ `material`（材料的四字段与阶段） | `catalog`、`material` **有自己的定义 → 聚合**；`search`、`audit` 只跨聚合做一件事 → **领域服务** | 聚合 `catalog/`、`material/`；服务 `search/`、`audit/` |
| 工作流（定义侧） | `--list` / `--new` / `<名字>` / `--check` / `--export` / `--import` | 有身份有生命周期 → 聚合 | `workflow/` |
| 任务（执行侧） | `--list` / `--new` / `<名字>` / `--next` / `--done` / `--journal` | 聚合（「走一步」= 现 `task_run.dart`，同属 task） | `task/` |
| 其他 | `health`（探活 provider）/ `help`（导览） | 边界外/职能 → 适配处 | `platform/` |

**最终靶子（分三类列）**：聚合 `workflow/` `task/` `catalog/` `material/` `workspace/`；领域服务 `search/` `audit/`；适配处 `platform/`（命令面 dispatch、导览 help、判据执行 rules、给智能体的话术 prompts、YAML 解析、平台读写 fs/host）。

四条纪律：

- **未实现的命令也先占位**：`dispatch` 如实回「还没搬：<命令>」的（studio 侧 `search`/`catalog`/`audit`/`material`）照样列——清单是靶子，不是现状
- **归类看定义、不看词性**：`catalog` 跨工作区且只读，但它有自己的定义（条目、名字索引、快照语义）→ 聚合；`audit`/`search` 没有自己的定义 → 服务。`catalog`、`search`、`audit` 全属名动同形，**名字形态不决定归类**
- **不造 `-er` 目录**：`searcher/` 不合惯例（`-er` 是类名惯例：Fowler 的 Finder、Lucene 的 `IndexSearcher`）；目录用能力名
- **依赖单向**：服务可依赖聚合，聚合不得依赖服务（`search` 用 `catalog` 的名字索引）
- **`search` 与 `catalog` 同属「目录层」**，共用一份名字索引（`catalog.rs` 的 `build()`）：`catalog` = 按资产种类清点（快照，仓库变了要重扫），`search` = 按名字定位一份（输出 `[种类] 相对路径` 两列，`--show` 带出内容）。索引**同时收文件名与篇内标题**——因为命名规则是「英文文件名定路径、中文标题定概念，二者不互译」，不收两边就找不到
- **别把界面当成业务模块**：`views/`、`screens/` 不参与划分

## 体检检查清单（对任一 Studio 仓按契约逐条量）

结构契约：

- [ ] 命名随框架家法（⚠️ **2026-09-24 拿官方源纠正**：家法里**页面层是 `view/<功能>_page.dart`、部件层是 `widgets/`**——官方示例全量统计 `views/` **0 处**、`view/` 63、`widgets/` 22、`bloc/` 245。早前「界面件应叫 `views/`」那条是错判，据此改的仓要正回来；核实方法与原文见 `quanttide-flutter-studio/references/bloc-house-rules.md`）
- [ ] 同一目录不得混用层名与聚合名（硬禁）——列出 `local/*.dart` 逐个归类为聚合名／职能名
- [ ] 单文件行数越界（>250 行即触发转聚合）
- [ ] 角色唯一：是否并存两套实现（`client.dart` 起命令行 + `local/` 自己算 = 同一件事两份）
- [ ] 依赖单向（端侧引用工具箱、无本地 models 层）
- [ ] 组装与实现分离（`main.dart` 装配 / 命令面一处实现）
- [ ] 测试跟着分层（`test/` 与 `lib/` 同构）
- [ ] 横切约束集中一处（有无所约定文件；散在 README 与注释里 = 缺）

依赖契约：

- [ ] 自研包按版本号引（`^0.1.0-beta.5`）而非 path
- [ ] 官方 SDK 优先（用 `yaml` 包而非自写解析）
- [ ] 默认选型成文（Bloc ✓ / Material ✓ / `go_router` ✗ —— 现为自切屏 + `Navigator`）
- [ ] 依赖许可清单（一处列明已许可依赖 + 新增依赖先入清单）
- [ ] 外部依赖的坑写进契约（CanvasKit 自托管：构建命令里是否有 `--dart-define=FLUTTER_WEB_CANVASKIT_URL=/canvaskit/`）

平台契约：

- [ ] 角色技术栈（Flutter Web + Bloc）
- [ ] 桶命名 `{产品线}-{用途}` / 域名 `{产品}.cloud.example.com`
- [ ] IaC 目录 `manifests/terraform/`（Studio 仓常无——记明归属或豁免）
- [ ] 门禁：`dart format` + `flutter analyze`（CI 常只跑后两条）
- [ ] 门禁本地从严与 CI 一致
- [ ] 可观测与安全（无后端则不适用，显式豁免）

工作流对照：

- [ ] 工作流的判据引用的路径是否真实存在（studio 实录：交接判据查 `lib/cli`，该目录不存在，实际是 `repositories/client.dart` + `runner.dart`）
- [ ] 判据 grep 的占位文案（「命令行还没给」类）是否已清
- [ ] 对表脚本可跑（`scripts/parity.sh`）

## 分段原则（TODO 的段）

1. **正名**（零风险）：⚠️ 目标名**以家法为准**（页面 `view/<功能>_page.dart`、部件 `widgets/`，见 `quanttide-flutter-studio/references/bloc-house-rules.md`）——**别再照旧稿写成 `lib/views/`**：qtdata 二段曾照这条把 `widgets/` 改成 `views/`，方向反了；`test/` 同步，改 import 与 README 目录图
1b. **命令改名**（同为零风险的对外正名）：`find` → `search`——判据是承诺的行为（带 `--show` = 检索不是定位；VS Code 惯例 Find=文件内 / Search=全项目）。一起改：CLI 分发、`help` 分组、`api-references`、studio 的 `dispatch`、模块目录名；发布线已开 = 破坏性变更（版本号 + CHANGELOG），**studio 侧还没搬时改最便宜**
2. **清混用**（硬禁）：职能件下移一层（`local/platform/`）或聚合件上移一层（`local/{task,workflow,run}/`），做完只留一种维度；聚合名单写进约定文件
3. **收敛两套实现**（=工作流原定的「交接」）：删起命令行那层，界面全走 `local/`；错误类型随之改（`CliFailure` → `LocalFailure`）；同步删对应测试与 README 里「命令行那份仍留着」的说法
4. **转聚合**（触发已命中）：拆长文件、按聚合重排；建 `lib/CONVENTIONS.md` 收横切约定（界面只认传进来的模型与回调、错误怎么说、路径怎么显示、聚合名单、views 命名）
5. **契约对齐**：路由（改 go_router 或申请例外）、CI 补 `dart format --set-exit-if-changed .`、CanvasKit 实测与自托管、依赖许可清单、IaC 归属
6. **文档与工作流对齐**：补 `doc/` 欠账、重定 `doc/models/`（lib 已无 models 层 → 改称「契约形状」或迁 toolkit）、工作流判据改到实际路径
7. **发布线**：定「一条线还是各发各的」+ 开 `studio/v*`（版本号由创始人拍板）

## 通用判据三连（每段做完，缺一不算完）

```bash
cd apps/<app>/src/studio
flutter analyze && flutter test
sh scripts/parity.sh            # 对表：命令行与 studio 各跑一次，结果必须仍一致
```

行数判据：`find lib -name "*.dart" | xargs wc -l | awk '$1>250'` 为空。
命名判据：按家法应为 `test -d lib/widgets`（部件层）且页面在 `view/*_page.dart`——**不要**用 `test -d lib/views` 当正名判据（那是错判目标，见 bloc-house-rules）。
收敛判据：`test ! -f lib/repositories/client.dart`。

## studio 实测数据（2026-09-12，供下次对比）

- `lib/` 35 个 Dart 文件、3 675 行；最长四件：`repositories/local/tasks.dart` 378、`states/workbench_bloc.dart` 349、`repositories/local/workflows.dart` 316、`repositories/local/task_run.dart` 279
- `test/` 与 `lib/` 同构（repositories / states / screens / widgets + support + fixtures）
- `parity.sh` 24 条命令两侧结果一致（比 `ok`/`columns`/`rows`/`data`，`lines` 不比——给人看的话各写各的）
- CI：`.github/workflows/release-studio.yml`——分支/PR 触达 `src/studio/**` 只跑门禁（analyze + test），推 `studio/**` tag 才构建 Web 版并上传 OSS `qtcloud-work-studio` + 刷 CDN `work.cloud.example.com`；缓存策略：哈希资源长缓存、固定名入口文件 no-cache
- 已命中触发：单文件行数越界（4 件 >250 行）+ 目录混用
- 依赖：`quanttide_work ^0.1.0-beta.5`（工具箱，版本号非 path）、`flutter_bloc ^9.1.0`、`yaml ^3.1.4`、`cupertino_icons`、dev `flutter_lints ^6.0.0`

## qtdata studio 实测数据（2026-09-24，第二个 studio 样本）

- `lib/` 21 个文件 2 833 行；最长四件：`screens/tabs/business_tab.dart` 381、`models/project.dart` 377、`screens/dashboard_screen.dart` 335、`widgets/cards/matrix_card.dart` 303 → 同样命中「单文件越界」
- `test/` 12 文件 21 用例；`flutter analyze` 零告警、`dart format --set-exit-if-changed` 无差异、`flutter test` 全绿（三条实测）
- 运行时依赖只有 `flutter` + `cupertino_icons`——**无网络、无状态管理、无本地存储**；数据来自 `assets/data/seed_projects.json`，且 `test/helpers/seed.dart` 直接读这个 JSON（换数据源时 11 个测试一起坏）
- 平台判据：**web 是产品形态**（CI `build web` → OSS `qtdata-studio` + 刷 CDN `data.example.com`）、**linux 是本地调试形态**（`scripts/run-studio-linux.sh` 里就是 `flutter build linux`）——判「平台目录能不能删」必须先看这两个脚本与 CI，别按文件数下结论。qtdata 那轮把 android/ios/macos/windows 判成可删，被用户全否（要保留并完善）；空 `integration_test/` 也被否（要维护），`doc/`（页面分解产物）同样不能删
- 与 qtcloud-work 那侧不同的两处缺口：**无路由表**（平台契约要求 studio 统一 `go_router`，这里是 `MaterialApp(home:)` 直挂）、**无 `repositories/` 与 `states/` 两层**
- **加严实录**：`analysis_options.yaml` 上 `strict-casts` / `strict-inference` / `strict-raw-types` + `prefer_single_quotes` / `unawaited_futures` / `always_declare_return_types` 后首跑**只命中 2 处推断失败**（`MaterialPageRoute` 与 `showDialog` 缺显式类型参数），补 `<void>` 即转零告警；`dart format` 对本仓零改动 → **加严不必怕，先跑再修；命中就改代码，不关规则**。（裸泛型与无类型空集合各 0 处，事前扫一遍能预判大半）
- **本机工具链：并存两套 Flutter SDK，别急着判「本地与 CI 不一致」**（2026-09-24，同一次会话内先判错又纠正）——`~/repos/flutter`（3.44.8，gitee 镜像，7/29）与 `~/flutter`（3.44.9，github，8/8）同时在，而 `.bashrc` 当时指向**旧的那套**；CI pin 的 3.44.8 恰好等于用户 PATH 里的旧版。四条纪律：
  - **先查实测版本再下结论**：`bash -ic 'which flutter; flutter --version'`（只有交互式 shell 读 `.bashrc`）。agent 自己 `export PATH` 出来的路径**不等于用户的 PATH**——拿它当「本地版本」会把结论判反
  - 收口顺序：**先用某套 SDK 把门禁跑绿（把版本写进评测记录）→ 再把 CI 的 `flutter-version` pin 升到同一版 → 同时改 `.bashrc` 指向该版本 → 用 `bash -ic 'which flutter'` 复验解析结果**（改了文件不验证等于没改）
  - agent 的非登录 shell 不读 `.bashrc`：命令写全路径 `~/flutter/bin/flutter` 或先 `export PATH="$HOME/flutter/bin:$PATH"`，但**别把这个 export 当成环境现状写进仓库文档**
  - 两套并存占盘（1.7G + 1.2G）：清理旧 SDK 前先问用户，不擅自动手

## 二段实录：正名与拆分（widgets→views，pi 执行 + Hermes 复验，2026-09-24）

- 正名：`lib/widgets/{cards,common,dialogs}` 12 件**平铺**进 `lib/views/`（撤掉按形状切的子目录），`test/widgets` → `test/views`（测试跟着分层）
- 拆越界件（同层单文件 ≤250 行即越界）：`models/project.dart` 377 → 6 件、`tabs/business_tab.dart` 381 → 7 件（外壳 + 6 张卡）、`screens/dashboard_screen.dart` 337 → 3 件、`views/matrix_card.dart` 303 → 2 件；**最长文件 381 → 235**
- 唯一超出「只改路径」的命名：跨文件搬出的私有类必须转公开（`_QuotationCard` → `QuotationCard`，并按 `use_key_in_widget_constructors` 补 `super.key`）、`_MatrixCell` → `MatrixDataCell`（与模型类 `MatrixCell` 撞名）
- **提示词要写「不许发明抽象」**：拆模型只按现有类聚集拆，不开新文件、不改字段名、不改 JSON 结构（模型来源没定之前不重构语义）
- 拆完的 `models/` 分布：`project`（Project + Deliverable）、`project_status`（两个枚举）、`project_phase`、`project_matrix`、`blueprint`、`business_info`

## 判「行为不变」的三处静态对比（Flutter 版，比 worktree 对差便宜）

纯挪文件／拆件的重构，一条脚本跑完比跑测试更能说明「没动行为」——数字对得上，才敢说「只动了位置」：

- **断言数**：改前 `git ls-tree -r --name-only HEAD src/studio/test` 逐件 `grep -c 'expect('` 求和 vs 改后——**少了就是削弱了断言**
- **中文字面量集合**：遍历全 `lib/`（改前用 `git show HEAD:<file>`）取中文串集合再比——**丢了/多了都要问清楚**（qtdata 那轮 71 种一一对应 = 界面文案一字未动）
- **类名差集 + 最长文件**：`class|enum` 名前后差集应只剩可解释的改名；最长文件给前后数字（381 → 235）
- **提交前查 `pubspec.lock`**：本机 `flutter analyze` 会把锁文件 URL 从 `pub.dev` 改写成 `pub.flutter-io.cn` 镜像（pi 每次都还原，自己也要查）——别带进提交；想根治给环境设固定 `PUB_HOSTED_URL`

## 按域分文件夹（lib/{域}/{层}，2026-09-24 用户拍板）

用户给的形态：`lib/` 分成**域文件夹**，每个域内**照现在的分层**（`models/` `views/` `screens/`），`test/` 同名跟着分。qtdata 的域是 **data / project / business + asset**——**asset 收「三域 × 五阶段」的交付资产矩阵与 `Deliverable` 交付物**（用户明确把矩阵和交付物都归 asset，不归 shared）。

**「域」现在的真实状态**：三个域只活在 seed JSON 里——矩阵三行的键 `project` / `data` / `business`，`cells` 的键就是 `{域}_{阶段}`（3×5=15 格）；代码里**没有枚举、没有常量表、没有按域切的目录**（全 `lib/` 里 `'business'` 只出现一次，还是 `json['business']`）。界面侧的对应物是详情页三个 Tab（data=`BlueprintCard`、project=`TimelineCard`、business=报价/合同/交付/付款/流程五张卡）；另外两个 Tab（总览、资产）**不是域**，是观测入口——「5 个 Tab」≠「3 个域」，别混。要让域成为结构概念，得先给它定义（枚举或契约）。

**动结构之前先盘 import 图**（这次靠它把 shared/ 从 12 件缩到 9 件，并抓出两件「其实不跨域」）：逐文件算出「对外接口（类/函数）+ 被哪些文件 import」，按域归组。

判据：**被 ≥2 个域引用**才进 `shared/`；只被一个域引用的（如 `progress_bar_widget` 只被 `project_card` 用）留那个域；`section_header`（跨四域）是真共享，`phase_tag`（只 project + 详情页外壳）属边界——**拿引用证据说话，别按「听起来通用」归**。

最终 shared/ 清单（9 件）：`models/project_status`（两个枚举被 8 处引用）、`views/{section_header,status_badge,sidebar,responsive,toast,doc_dialog}`、`screens/project_detail_screen`（详情页外壳，组装三域）、`screens/tabs/overview_tab`（总览 = 项目摘要 + 交付物明细，跨 project+asset）；`main.dart` 留 `lib/` 根。

**先给映射表再动手**：域划分不是干净的三分法——共用件、聚合视图、外壳页都要单独交代去向。用户会追问「**要看到底有什么**」，所以清单必须带**证据列**（里面有什么类/函数 + 被谁引用），只给结论会被打回；边界项单独列出来说「我倾向 X，你也可以 Y」。

### 最终形态（2026-09-24 用户裁决并已实施——**别再按家法改回去**）

用户看过家法原文后的三点裁决：**① 跨域那层叫 `app/`（不叫 `shared/`）② 域内维持量潮自己的分层（`models/ views/ screens/`），不采纳家法的 `view/` + `widgets/` 两档 ③ 其余照原方案**。已实施（44 文件搬迁，git 全程识别为 rename）：

```
lib/
├── main.dart         装配（留根）
├── app/              跨域 10 件：project_status（两个枚举）+ views/{section_header,status_badge,
│                     sidebar,responsive,toast,doc_dialog,phase_tag} + screens/{project_detail_screen,overview_tab}
├── project/  9 件 1103 行（含 progress_bar_widget——只被 project_card 用，留域内）
├── data/     3 件 222 行      business/  8 件 479 行
└── asset/    5 件 432 行（project_matrix + matrix_card + matrix_cells + assets_tab + Deliverable）
```

- `Deliverable` 从 `project/models/project.dart` 拆出 → `asset/models/deliverable.dart`
- Tab 归**本域** `screens/`（**撤掉 `screens/tabs/` 子层**：每域只剩一个 Tab 页，单文件子目录没意义——这类"顺手合并一层"要在方案里写明）
- `test/{app,data,project,business,asset}/{views,screens}` 同名同构；`test/helpers/` 与 `main.dart` 同理**留根**（同一原则：**跨域的留根，属域的进域**）
- 界面部件层叫 `views/` 是**本项目显式约定**（`CONTRIBUTING.md` 写明"Bloc 官方示例是 `view/` + `widgets/` 两档，我们合成一层"）——**别再正回 `widgets/`**（换名 = 推翻用户裁决）。组织级契约那条案例站不住、但属别的仓，未动
- 搬迁配方（映射表校验 / 两类 import 都要重写 / 门禁当漏项探测器 / 收尾清单）：`roadmap-stage-execution` 的 `references/bulk-dir-restructure.md`
