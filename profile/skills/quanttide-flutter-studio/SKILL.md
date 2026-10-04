---
name: quanttide-flutter-studio
description: 开发量潮 Flutter Studio 客户端（qtdata/qtclass/qtcloud-econ/qtfounder/qtcloud-agent）时触发。
---

# 量潮 Flutter Studio 客户端开发与部署

量潮各云产品客户端（qtdata_studio、qtcloud_econ_studio、qtfounder_studio 等）均为 Flutter 应用，位于 `<app-repo>/src/studio/`。参考实例：qtdata/src/studio、qtcloud-econ/src/studio（2026-08-09 建立）、qtfounder/src/studio（2026-08-14 建立，量潮创始人工作台·创作现场）、qtcloud-secret/src/studio 与 qtcloud-devops/src/studio（2026-08-16，pi agent / Hermes 参考建立）。

### qtcloud-secret 骨架模式（2026-08-16 参考实例）

qtcloud-secret/src/studio（pi agent 维护，v0.1.0-alpha.7）的分层是轻量客户端的可抄结构——用户明确\"参考 qtcloud-secret\"初始化 qtcloud-devops：

```
src/studio/
├── pubspec.yaml      # name: studio（简单名惯例，非 <app>_studio）；version 0.1.0-alpha.N
├── lib/
│   ├── main.dart         # <App>App(state: AppState()) 入口，注释写设计思路指向 docs/index.md
│   ├── app_state.dart    # 应用状态（ChangeNotifier：登录态/数据；页面由状态驱动，不依赖导航栈）
│   ├── ui/               # 页面（login_page/secret_list_page/secret_edit_page/backup_page…）
│   ├── store/            # 本地缓存
│   ├── api/              # provider 客户端（对接服务端 API）
│   └── auth/             # 会话管理
└── test/widget_test.dart
```

- 页面由 AppState 状态驱动（未登录→登录页；已登录→列表页）——**无导航栈依赖**（参考 qtcloud-secret main.dart 注释）
- 简单场景可用 Flutter 内置 `ListenableBuilder(listenable: state)` 免 provider 依赖（qtcloud-devops 实测）
- qtcloud-devops/src/studio 骨架 = main + app_state + ui/scan_page（扫描卡片 + 按钮，占位逻辑待接 CLI）

## 项目初始化（标准流程）

**手工骨架不够，必须 `flutter create` 补全平台目录**（android/ios/web/windows/linux/macos、.gitignore、.metadata）：

```bash
# 1. 先手工写 pubspec.yaml + lib/main.dart（应用名 <app>_studio、主题参考 qtdata：紫色 0xFF4F46E5 + F1F5F9 背景）
# 2. 有 flutter SDK 时补全标准结构（保留现有 pubspec/lib/analysis_options，只补缺失平台目录）
export PATH="$HOME/flutter/bin:$PATH"
flutter create . --platforms=web,windows,linux,macos,android,ios --org com.quanttide
# 3. 修复 create 生成的模板测试：test/widget_test.dart 引用 MyApp，改为实际应用类（如 EconApp）
# 4. 三连验证：flutter analyze（零问题）→ flutter test → flutter build web
```

- flutter 首次下载中断可续传（"Resuming transfer from byte position N"），重跑 `flutter --version` 即续。
- 目录惯例：`lib/{models,screens,widgets}`（**widgets 不是 components**——用户明确要求 Flutter 惯例）；种子数据统一 `assets/data/`，pubspec 注册**目录级** `- assets/data/`（新增种子自动打包）。

## 开发模式（Mechanism 模块实例，2026-08-09）

1. **模型**：`lib/models/<domain>.dart`——实体 + `fromJson`（对缺失字段容错：`?? ''`、`as List? ?? []`）
2. **种子数据**：`assets/data/<domain>s.json`——真实业务数据（如招聘博弈机制提炼自 journal 08-09），不是凭空编造
3. **页面**：`lib/screens/<domain>_screen.dart`——列表页（DefaultAssetBundle 加载种子）+ 详情页；`lib/widgets/<domain>_card.dart` 卡片组件
4. **接入**：`main.dart` 壳（品牌侧边栏 80px + 内容区），页面作为首页
5. **测试分层**：`test/models/`（fromJson 解析/容错）+ `test/widgets/`（卡片渲染/统计/点击）+ `test/screens/`（种子加载/详情结构）
6. 先写 ROADMAP（src/studio/ROADMAP.md）定目标，再实现——目标形式"实现 X 模型 + 配套展示页面，支持 Y 模块"

### Flutter 测试 Pitfalls

- **数字匹配歧义**：`find.text('1')` 在多个统计同为 1 时 findsOneWidget 失败 → 用 `findsNWidgets(3)` 或只对唯一数字断言
- **const 构造 lint**：`prefer_const_constructors`——测试辅助构造器用 `=> const Mechanism(...)`（列表内元素不必逐个 const）
- **widget 测试加载 asset**：异步加载需 `pumpAndSettle()` 等完成；pubspec 注册的 asset 在 flutter test 环境可用
- **真实文件 IO 与 FakeAsync 不兼容**（qtfounder 踩坑）：页面在 initState 里 `Directory.list()` 读文件系统时，widget test 的 FakeAsync zone 不推进真实 IO，`pumpAndSettle()` 超时（`tester.runAsync` 也不可靠）。解法=**依赖注入**：StatefulWidget 加可空 loader 参数（`final Future<List<T>> Function()? chaptersLoader`），`_load()` 用 `widget.chaptersLoader ?? 默认加载器`；测试注入 fake loader 返回示例数据
- **无法注入时的备选**（页面经 Shell 导航挂载、无注入点时）：widget 测试只验证**挂载**而非加载完成——tap 导航后 `pump()` 一次，断言"标题出现 **或** CircularProgressIndicator 存在"（`find.text(x).evaluate().isNotEmpty || find.byType(CircularProgressIndicator).evaluate().isNotEmpty`），不等待 IO 完成（避免 pumpAndSettle 超时）；真实加载正确性由引擎级测试（注入 dataSourceRoot 的目录读取测试）覆盖
- `withValues(alpha:)` 替代已废弃的 withOpacity

## 数据源环境变量配置（qtfounder 模式，2026-08-14）

工作台以外部仓库为数据源（如 fiction/memory），路径通过 dart-define 环境变量配置：

```bash
flutter run \
  --dart-define=QTFOUNDER_FICTION_PATH=$HOME/repos/quanttide-founder/assets/fiction \
  --dart-define=QTFOUNDER_MEMORY_PATH=$HOME/repos/quanttide-founder/assets/memory
```

- `lib/config.dart`：`String.fromEnvironment('VAR')` 读注入值（编译期常量，跨平台一致）；空时回退默认路径——桌面端 `Platform.environment['HOME']` 拼 `~/repos/...`，**Web 端无文件系统返回空**（`kIsWeb` 判断）
- `lib/data/<domain>_repository.dart`：桌面端 dart:io 读目录（`Directory.list()` 过滤 `.md` 排序）；Web 端回退内置示例数据
- 平台行为：桌面=真实文件系统；Web=内置示例（真实数据需 API 转发）；测试=依赖注入（不碰文件系统）
- **README.md + CONTRIBUTING.md 必须记录配置方法**（变量表/默认值/数据源布局约定/平台行为）——用户明确要求

## studio 独立数据层（不依赖 CLI，2026-08-21 qtcloud-agent 纠正）

**用户明确纠正："不要依赖 cli，独立管理数据"**——studio 不得用 `Process.run('qtcloud-agent', ...)` 调 CLI 拿数据，而是**直读同一数据源文件系统**，自己实现解析逻辑（models + repositories 分层，qtfounder 模式）。CLI 与 studio 是同一数据的两个消费端，各自独立实现、行为对齐：

```
lib/
├── models/<domain>.dart                    # 领域模型（AgentStatus/AgentFile、SessionGroup/SessionItem）
└── repositories/<domain>_repository.dart   # 直读文件系统（不调 CLI）
    ├── agent_repository.dart    # 注册中心：读 registry/agents/ 目录 + 本地配置对比
    └── session_repository.dart  # DSH 数据：读 $DSH_HOME/sessions/ 按工作区分组
```

- **数据源用环境变量**（运行时 `Platform.environment`，非 dart-define 编译期——因为是外部数据目录不是打包资源）：`QTCLOUD_AGENT_REGISTRY`（注册中心，默认 data/profile）、`DSH_HOME`（默认 ~/.dsh）
- **对齐 CLI 解析逻辑但独立实现**（读 CLI 源码 main.rs 逐个对齐，别猜）：
  - agent_local_dir 映射：zed→~/.config/zed、opencode→~/.config/opencode、hermes→~/.config/hermes；未知 agent 报 unknown
  - 配置对比 = JSONC 清理（去 `//` 行注释 + 去尾逗号）→ JSON 展平（key.path → 字符串）→ 键值对比（一致/差异/缺失）
  - decode_workspace：`--tmp--`→临时工作区；`--path--`→去双横线、`-` 还原 `/`（**不是 base64**，曾猜错）
  - projcache 路径：`$DSH_HOME/storages/session_projcache.json`，结构 `/tables/sessions/{sid}/identity/cwd`（不是根目录的 session_projcache.json，曾写错）
- **页面**：三职能导航（会话/技能/智能体——与 qtfounder 资产/写作/思考同构，职能平级无分组）；会话页按工作区分组列表（大小/时间/cwd）；智能体页 ExpansionTile 展开对比（图标分色：绿=一致/橙=差异/红=缺失）
- 技能页（未实现功能）= 设计意图占位卡片（列出待建步骤），不硬编
- 验证：`flutter analyze` 零问题 + `flutter test`（导航三职能存在性）+ `flutter build web`

## lib 结构（qtcloud-think studio，2026-08-22 三轮纠正定稿）

用户三轮纠正：**"data/不应该存在"** → **"仓储指的是领域驱动的仓储"** → **"那应该叫 models/"** + **"和 repos/"**。最终定稿 = **对齐现有仓库惯例的三件套，不发明层**：

```
lib/
├── models/                       # 领域模型（thought.dart——models/ 是惯例名，用户明确"那应该叫 models/"）
├── repositories/                 # 仓储（对齐 qtcloud-agent 的 repos/ 惯例）
│   ├── thought_repository.dart   #   想法仓储（本地文件读写）
│   └── clarify_repository.dart   #   AI 加工访问（调 provider——判断/澄清/结构化）
├── screens/                      # 页面（perceive_screen/understand_screen…）
└── main.dart
```

- **仓储 = DDD Repository**（封装数据访问，领域层依赖接口、实现可换）——不是"用户数据存放位置"
- ⚠️ 弯路实录：`data/`（被否——非惯例名）→ `domain/ + infrastructure/`（被否——过度分层，用户"那应该叫 models/"；仓储应并入 repositories/）→ `services/`（被否——AI 加工也是数据访问，并入 repositories/ 为 clarify_repository）
- **规则：目录名对齐仓库现有惯例（models+repositories+screens），不发明新层**（services/domain/infrastructure 都不引入，除非用户明确要求）
- 种子数据（示例，git 跟踪）在 `assets/data/`——是数据来源，与仓储概念无关；用户数据（仓储实现写的位置）在用户目录（环境变量可覆盖），git 忽略

### 实施顺序（用户要求：分两步，2026-08-22 qtcloud-think）

用户明确："**分两步，先写只需要单元测试的，再写需要组件测试的。旧代码可以全部丢弃**"：

1. **第一步（纯单测）**：models + repositories——无 UI 依赖，纯 Dart 测试
2. **第二步（widget 测试）**：screens + main 导航壳——依赖注入假仓储后测交互
3. 旧方案代码（旧 models/screens）**直接 git rm 丢弃**，不兼容共存；main.dart 先写最小壳（占位），第二步再实现导航

### 单元测试模式（qtcloud-think 分解 1 验证，16 测试全绿）

- **模型**：JSON 往返无损（toJson→fromJson 全字段，含空/非空记录）+ 状态流转合法性（非法回退 `throwsStateError`）+ 默认值
- **仓储（本地文件）**：`Directory.systemTemp.createTemp()` 注入临时目录（不碰真实用户目录）——写入读回/更新持久化（**新实例模拟重启**读回）/按状态过滤/损坏文件→空列表不崩溃
- **仓储（网络/AI 访问）**：`MockClient`（http 包 testing 库）拦截请求——断言 URL/body 解析正确 + **失败降级必测**（500/超时→抛明确错误，不造假结果——设计文档明确）
- **领域对象最小接口**：仓储依赖抽象（如 `ThoughtLike` 接口：content/clarifyLog）而非具体模型——解耦仓储与模型
- ⚠️ **改 Dart 用 patch 工具，不用 sed**：sed 改字符串插值/引号会把 jsonEncode 改坏（`ambiguous_set_or_map_literal_both`/`undefined_identifier` 连环错）——analyze 报错后 read_file 定位再 patch；测试辅助函数命名不要下划线开头（`no_leading_underscores_for_local_identifiers` lint）

## CI/IaC 部署全家桶（每应用三件套，5 实例验证）

```
.github/workflows/deploy-studio.yml   # CI：打 studio/* tag 触发
manifests/terraform/                  # IaC：OSS 桶 + CDN 域名（Terraform 声明）
scripts/run-studio-linux.sh           # 本地运行：flutter build linux → 运行 bundle
```

### deploy-studio.yml 要点
- 可直接复制的已知良好版：`templates/deploy-studio.yml`（改 3 参数：bucket/域名/tag）
- **两作业形态（2026-09-24 qtdata 起）**：`quality-gates`（`on: push branches: '**'` + `pull_request`，`paths: src/studio/**` + 本 workflow 文件；跑 `dart format --set-exit-if-changed --output=none .` + `flutter analyze` + `flutter test`；`concurrency: group: <wf>-${{ github.ref }}`）与 `build-and-deploy`（`needs: quality-gates` + `if: startsWith(github.ref, 'refs/tags/studio/')`）——**分支/PR 只跑门禁、tag 才部署**，家族里 qtcloud-work 的 `release-studio.yml` 同构
- 触发 `push tags: studio/*`；subosito/flutter-action v2（flutter-version 固定）→ `flutter build web`
- **复核 CI 真的跑了：用公开 REST API，不需要 gh token**（`gh` 认证过期也能查）——
  ```bash
  curl -s "https://api.github.com/repos/<org>/<repo>/actions/runs?per_page=2" | python3 -c "import json,sys; [print(r['name'], r['head_sha'][:7], r['status'], r['conclusion'], r['html_url']) for r in json.load(sys.stdin)['workflow_runs']]"
  ```
  分支推送后 `sleep 45` 再轮询到 `completed`；**要连作业一起核**（`.../runs/<id>/jobs`）——门禁 success + 部署 job `skipped`（非 tag）= 按设计生效。docs-only 改动只要落在 `paths` 里也会触发，等于顺手再验一次
- **域名惯例**：studio 用 `studio.{产品}.example.com`、site 用 `{产品}.example.com`（qtclass=studio.class、qthealth=studio.health、qtdata=studio.data）——两级子域 `*.example.com` 泛域名证书盖不住，**要单独签单域名证书**；换域名的完整命令链与四个坑（新域名根路径 403「forbidden to list buckets」→ 必须加根改写、`oss_pri_buckets` 参数不被接受要用 `l2_oss_key`、配置从 configuring 到 success 要等 5–10 分钟、续期钩子必须显式装）见 `aliyun-static-site-deploy`
- OSS 部署：`ossutil cp build/web/ oss://<app>-studio/ -r -f`（长缓存 max-age=31536000）
- **入口文件单独 no-cache**：manifest.json / index.html / flutter_bootstrap.js（长缓存导致浏览器拿不到新版）
- CDN 刷新：aliyun-python-sdk-cdn，RefreshObjectCaches（Directory 级）

### Terraform IaC 要点（参考 qtcloud-data/manifests/terraform）
- 文件：main.tf（OSS bucket + CDN domain）、variables.tf、outputs.tf、terraform.tfvars.example、bucket_policy_cdn.json、README.md
- **BlockPublicAccess 坑**：新 OSS 桶默认开桶级 BlockPublicAccess → 必须先关闭才能设 `acl = "public-read"`；同时开静态网站托管（根路径返回 index.html）
- bucket 名/域名必须与 deploy-studio.yml 一致（不一致静默失败）
- bucket_policy_cdn.json：CDN 回源授权（RAM 角色 AliyunCDNRoleForOssPrivateAuth）

### 发布预检（打 tag 前）
- `gh secret list`——**ALIYUN_ACCESS_KEY_ID/SECRET 必须已配置**（缺失则 CI 部署必然失败）
- bucket/域名已建（terraform apply 过或手动确认）；CHANGELOG 有对应版本记录；tag 未打

### 四端完整 CI 矩阵（qthealth 2026-08-16，新应用标配）

一个 verify + 三个 deploy（tag 前缀隔离触发）：

```
.github/workflows/
├── ci-verify.yml        # push/PR 触发：四端构建测试（cargo build+test / go build+vet+test /
│                        #   flutter pub get+analyze+test / npm ci+build）——每端一个 job
├── deploy-provider.yml  # tag provider/* → Docker 镜像 → Terraform apply → FC
├── deploy-studio.yml    # tag studio/* → flutter build web → OSS → CDN 刷新
└── deploy-site.yml      # tag site/* → npm build → OSS（<appA>-site 桶）→ CDN 刷新
```

- verify 用各自官方 action（dtolnay/rust-toolchain、actions/setup-go、subosito/flutter-action、actions/setup-node），defaults.run.working-directory 指向各端
- deploy-site 与 deploy-studio 同构（React dist/ 替代 build/web）

### IaC 完整模板复制适配法（qtcloud-secret → qthealth，2026-08-16）

新应用部署 IaC 最快路径 = **复制成熟仓库的 manifests/terraform 全目录适配**（qtcloud-secret 是完整版：providers/oss/cdn/fc/platform/locals/variables/outputs/versions + tfvars.example，10 文件含 FC 函数部署——比 qtcloud-data 的 6 文件 studio 版更全）：

1. 复制全部 `*.tf` + tfvars.example → 脚本批量替换：仓库名（`qtcloud-secret`→`qthealth`，含连字符/下划线两种形态）、CDN 域名（`secret.cloud.example.com`→`health.cloud.example.com`）
2. **移除源应用特有配置**：本应用无鉴权 → 删 JWT 相关（variables.tf 的变量块正则删除 + fc.tf 的环境变量行）——残留检查 `grep -rn "旧名\|dummy"` 清零
3. DNS 记录同步改（cdn.tf 的 `rr = "secret.cloud"` → `"health"`）
4. `terraform fmt -recursive`（复制文件常带格式问题，fmt 后 `fmt -check` 验证）；`validate` 需先 init（无 key 时跳过）
5. 提交信息列出完整清单（ci + iac + 域名）

## 设计文档族（src/studio/doc/，qtfounder 模式 2026-08-15）

用户会要求把设计决策**单独成文**（"导航栏怎么设计的，专门写一篇文档"、"再写一篇路由规划设计 router.md"）。Studio 的 `doc/` 目录（注意是单数 `doc/`）放设计思路文档，与 ROADMAP（规划/做什么）分开：

```
src/studio/doc/
├── index.md        # 整体设计思路：定位（创作现场）→ 架构（数据源→仓库层→UI）→ 关键决策（每条含"为什么"）→ 与平台意图对应表 → 演进方向
├── navigation.md   # 导航栏设计：定位 → ASCII 结构图 + 区域尺寸表 → 设计决策（每条 ①为什么…）→ 交互 → 演进（未来导航项候选 + 扩展规则）
├── router.md       # 路由规划：当前状态（无路由是合理起步）→ 规划目标 → 路由表（一级壳内切换/二级详情 push 带参数）→ 技术选型（GoRouter：Web URL 同步 + StatefulShellRoute 匹配壳）→ 壳集成 → Web 行为 → 演进步骤
├── screens/        # 页面设计（用户要求按文件夹整理，git mv 保留历史）
│   ├── index.md          # 页面总览（原 screens.md：总览表 → 每页定位/数据源/ASCII 示意/设计要点 → 共同原则 → 演进）
│   └── fiction-asset.md  # 单页专门设计（如小说资产页：资产结构规则 → 目录树镜像 → 交互 → 边界）
└── models/         # 数据模型（用户要求 screens/ 与 models/ 分文件夹）
    ├── fiction.md  # 小说资产数据模型（Novel/Stage/FictionFile/Sequence，编号=排序）
    ├── memory.md   # 记忆资产数据模型（MemoryCategory/MemoryFolder，自由命名+日期）
    └── asset-contract.md  # 资产契约与资产目录（统一模式，见下）
```

要点：
- **设计决策必须写"为什么"**：每个决策一条（如"为什么 80px 窄栏：内容密集型界面让位""为什么 loader 注入：widget test 的 FakeAsync 与文件 IO 不兼容"）——决策即踩坑史的正面记录
- **结构图用 ASCII**：导航栏/壳布局画 ASCII 图 + 区域尺寸表（比纯文字直观）
- **互相引用**：router.md 引用 navigation.md（扩展规则一致），结尾列"关联文档"；移动文件后同步全部相对引用（`../screens/...`）
- 路由选型给出理由（GoRouter 的 StatefulShellRoute ⇄ 导航栏项一一对应是选型核心，不是赶时髦）

### 模块级设计文档扩展（qtcloud-think，2026-08-22）

用户要求设计文档**按模块分文件夹**（"doc/screens/，每个页面一个文档"、"参考 screens/ 分解其他文件夹"）：

```
src/studio/doc/
├── index.md            # 模块设计总览：定位 → 模块结构（lib/ ASCII）→ 数据模型 → 数据流 → 实施分解（分解 1..N，每项含验收）
├── screens/            # 页面设计：每页一个文档
│   ├── perceive_screen.md     # 定位 → 功能区（ASCII）→ 交互行为 → 数据 → 验收
│   └── understand_screen.md
├── models/             # 领域模型（thought.md：状态流转/序列化/验收）
└── repositories/       # 仓储设计（thought_repository.md + clarify_repository.md——对齐 repos/ 惯例）
```

- 每个模块文档同格式：**定位 → 设计/接口 → 数据 → 验收**（对齐 screens/ 模式）
- 实施分解写进 doc/index.md（分解 1：模型+仓储…分解 N：导航壳），每分解含任务清单 + 验收
- 页面级文档含 ASCII 功能区布局 + 交互行为表 + 数据表（对齐"功能区/交互/数据/验收"四段）
- ⚠️ doc 子目录命名随 lib 惯例走（models/repositories/screens）——曾用 data//domain//services/ 均被用户否；.gitignore 的 `data/` 规则会误伤 doc/data 文档（git check-ignore -v 排查）——用精确路径或移动文档目录

### 领域模型收敛流程（qtcloud-execute 执行云，2026-08-23）

用户通过**连续追问收敛领域模型**——"X 是什么/做什么用"= 概念诚实检查，说不清就砍：

- **聚合最小化**：事件风暴产出 4 聚合（JournalEntry/Business/ProfileItem/Distillation）被否——"聚合应该只需要 Task 和 TaskList 足够了"；TaskList = 业务实体（一个业务一个清单）；**分组概念最终整体取消**（见下）——不枚举约束，业务自定义
- **字段逐个过审**：BlockReason（"是什么"→砍——阻塞=status 一种）、source（"是做什么的"→砍——无操作价值）、content→**description**（任务领域惯例 title+description）；枚举由用户明确：status=未开始/进行中/评审中/已完成（四态只前进）、priority=紧急/高/中/低（AI 建议+人确认）
- **命名随惯例不造词**：Section→Group（"分组为什么是 Section"）、content→description；事件风暴写 specification.md（领域事件/命令/聚合/角色/事件流/设计约束——DDD 建模）
- **doc 四层**：models/（领域模型）+ screens/（页面）+ views/（组件——弹窗/卡片）+ states/（Bloc Cubit）——views 与 screens 分层：详情用**弹窗组件**（"task-detail 做个组件就可以，然后清单里弹窗点开"），不配独立页面/路由
- **清单切换器数据驱动**（"不应该对清单做静态假设，而是应该有切换器"——不硬编码三业务）
- **看板演进三连（重要：中间态会被推翻）**：① 用户"二维看板，根据分组和状态"→ 我建议列=状态（看板惯例）→ 用户纠正"**列是分组，行是状态**"→ 我做成固定网格矩阵 → 用户"BoardView 只是一个表格，不是真正的看板"→ 反思（丢了两轮方向：先推责给布局，再归因"列=分组"本身错——**真正原因：我把数据投影当看板，从数据结构出发而非看板交互**）→ ② 用户确认"列=分组泳道、行=列内状态分段"（我理解成矩阵是错的）→ ③ 用户自己发现"**需要把状态调整到列更合适**"→"分组意义有限还干扰"→ **最终定稿：列=状态泳道（四列）+ 拖拽跨列=状态推进；分组取消→Task.category（String?，业务自定义分类，不枚举约束）**；BoardProjection 从"分组×状态矩阵"改"状态列→任务流"
- **⚠️ BoardView 表格化教训（设计文档即契约）**：doc 写"矩阵/单元格/交叉定位"→ pi 忠实实现成 Table——**文档语言决定实现形态**；要看板必须写"泳道/堆叠/纵向流动/拖拽跨列"，不能写"矩阵/格子"（数据投影视角）。设计文档评审先问"描述的是目标交互范式还是数据结构投影"
- **ROADMAP 四阶段重构**：领域层（模型+仓储）→ 状态层（Bloc）→ 组件层（views）→ 页面收敛+删旧代码——验收=旧代码全删 + analyze/test/build 全绿。委派 pi 执行时任务必须自包含（引用 ROADMAP/doc 路径+文件清单+约束+验证命令，注明"git 提交由我处理"），background+notify_on_complete，完成后 Hermes 验证再分层提交；pi 与 Hermes 同改一文件时以 Hermes 真实数据版为准
- **种子数据按 profile 档案制作**：从 data/profile 业务档案提取真实任务（3 清单×status/priority 覆盖），不是编造——强化"种子=真实业务数据"原则；⚠️ pi 会自编种子 + 基于自编数据写测试断言——替换为真实种子后必须同步修断言（8+ 测试挂），且断言任务 id/数量时以真实种子为准

### 数据模型文档模式（models/fiction.md，2026-08-15）

模型文档**独立成文**（"设计一个对应的数据模型，单独写一个文档来描述"）：
- **模型 = 仓库结构的镜像，不发明字段**：资产结构规则直接建模（Novel=一级文件夹 / Stage=二级文件夹 / FictionFile=文件（sequence 可空=未排序，version="2"后缀）/ Sequence=编号解析）
- 编号解析规则（正则）：`^(\d+)_(\d+)(?:_(.*?))?(?: (\d+))?\.md$` → sequence/title/version；不匹配 → sequence=null（未排序），title=全名
- 排序规则：已排序（chapter,scene,version 升序）在前，未排序按文件名在后；仓库级文件（README/CHANGELOG/myst.yml）不进模型
- **跨端契约**：模型是约定（文档即契约），Dart（studio）/ Go（provider）各自实现，JSON 字段命名对齐（列 JSON 示例）
- 关系：`FictionAssetTree → Novel(1..n) → Stage(1..n, order) → FictionFile(0..n)`

## 资产契约与资产目录（统一资产管理模式，2026-08-15 定稿）

两个资产（fiction/memory）规划完成后用户问"可否抽象出一套共同的资产管理模式？"——最终定稿为**资产契约 + 资产目录**：

- **资产契约（Asset Contract）**：**配置文件**（`src/studio/assets/contracts/<asset>.yaml`）承载语义规则——levels（层级语义+optional）、naming（pattern/title/sortKey/version/unsorted）、ignore（仓库级文件）
- **资产目录（Asset Catalog）**：通用引擎——读契约 → 遍历目录（跳过 ignore）→ 逐级建节点 → 文件按命名解析 → 排序 → 输出通用 `CatalogTree`（`CatalogNode`（name/label/children/files）+ `CatalogFile`（name/path/title/sortKey/version））
- 页面：通用 `AssetCatalogPage` 按契约渲染任何资产（label 由契约提供）；`/assets` 聚合页 = 契约注册表
- **新增资产 = 写一个契约 yaml，零代码**（引擎/页面/模型通用）

```yaml
# fiction.yaml 示例
levels: [{key: novel, label: 小说}, {key: stage, label: 阶段}]
naming: {pattern: '^(\d+)_(\d+)(?:_(.*?))?(?: (\d+))?$', title: '$3', sortKey: ['$1','$2'], version: '$4', unsorted: last}
# memory.yaml 示例：levels 二级 optional；naming.pattern: null（自由命名），unsorted: natural
```

关键决策：
- **契约 = 消费方视角，放 studio 端**（`assets/contracts/`）——fiction/memory 是内容仓库，不被应用配置污染
- **语义不进引擎**：引擎只做通用遍历/解析/排序，小说/记忆语义由契约 label 承载
- 排序策略内置三种：`sequence-first`（有键升序在前/无键名称在后）/ `natural` / `date`
- ⚠️ **教训**：用户说抽象时，量潮偏好 = **声明式配置（契约文件承载规则），不是代码级抽象**（策略类/接口）。我曾答抽象 NamingStrategy 类被用户纠正为「我的意图是抽象成资产契约和资产目录，用资产契约配置文件承载语义规则」——规则进配置，代码保持通用

### 实现要点（2026-08-15 实现验证，qtfounder 首个实例）

- **optional 级缺省**：memory 契约的 folder 级可选（仅 journal 有 default/）——目录无子目录时文件直接上挂当前节点。⚠️ 判断必须检查**当前目录**是否有子目录（`Directory(e.path).list()`），曾误检查父目录的 children 导致 optional 分支永不触发
- **sortKey 数值比较**（非字典序）：Dart `_compareSortKey`——`split('_')` 分段 `int.parse` 循环比较（1_1 < 10_1）。与 provider Go 的 `fmt.Sscanf("%d_%d")` 同一规则，两端各自实现、行为一致
- **契约测试两个坑**：① 引擎测试读 rootBundle（契约 yaml）需 `TestWidgetsFlutterBinding.ensureInitialized()` 作为 main() 第一行，否则报 "Binding has not yet been initialized"；② 测试 cwd = 项目根（src/studio），到父仓库根的相对路径是 **4 级**——Go provider 在 internal/creative 深处是 6 级，**两端路径级数不同**，先验证（os.path abspath + exists 探针）再写死
- 页面消费 AssetCatalog：CatalogNode 卡片（可展开/计数/版本标记 v2）→ 文件条目 → 阅读占位（正文阅读后续实现）
- **契约驱动自动适配新结构**：作者新增阶段目录（如 0_日志）或章节（19→20 章）时，契约 levels 通用、**无需改契约自动出现**——实证资产契约的价值；⚠️ 反过来说，**引擎测试断言阶段数/章节数会过时**（曾断言 4 阶段/19 章，作者推进后失败），测试断言用 containsAll 而非精确数量，或按需更新

### 扩展性边界（用户确认的评估，2026-08-15 沉淀 memory/insight）

域内（本地文件系统 + 目录树 + 文件名语义 + 排序组织）扩展性 = **配置性**（新资产 = 新契约 yaml，零代码）；**域外需新机制，不硬塞进契约**：

| 边界 | 说明 |
| --- | --- |
| 数据源形态 | 契约隐含 local-fs——远端/API/数据库源不适用 |
| 文件内容 | 契约只解析文件名，不解析内容（frontmatter/元数据） |
| 非目录形态 | 数据库表/API 集合/图谱——完全不同的结构 |
| 操作语义 | 当前只读——写操作是另一维度 |

预埋扩展点（`source: local-fs` / `contentParser: null` / `operations: [read]`）**不提前实现**（YAGNI）——真实需求出现再加字段。判断：边界不是缺陷，是设计克制（该域就是量潮资产管理的全部真实需求）。

## 导航栏职能设计（qtfounder 最终定稿，2026-08-15 多轮纠正后）

导航栏设计经历多轮用户纠正，**最终定稿：三个职能平级，无分组**：

```
┌────────────┐
│     量潮     │  ← 品牌区（两字 logo，16px/w700/主色——用户从单字"量"改为两字）
│            │
│  (资产)    │  ← 资产职能（聚合入口）
│  (写作)    │  ← 写作职能
│  (思考)    │  ← 思考职能
│            │
│  (弹性区)  │
└────────────┘
```

- **资产是职能而非组**（用户明确纠正"'资产'变成一个职能，记忆、小说这些都是资产的一部分"）：资产页是**聚合入口**（/assets → 小说/记忆等资产卡片 → 各资产页），资产内部按类型导航
- 三个职能回答三个问题：**有什么（资产）/ 怎么做（写作）/ 怎么想（思考）**
- 路由：`/assets` → `/assets/fiction` `/assets/memory`（二级）→ 阅读页（三级）；`/write`（写作）、`/think`（思考）
- ⚠️ **命名修正（最后定稿）**：用户把 `/create 创作` 改为 **`/write 写作`**、`/emotion 情绪` 改为 **`/think 思考`**（"改路由命名 /create 改为/write 写作，/emotion 情绪改为/think 思考"）。**"思考"不是简单改名**——对应 08-14 journal"情绪结构化处理器 = 思考云原型"（区分事实/感受/行动，作为思考的垫脚石）——情绪职能的实质是**思绪结构化**。新建/已有 Studio 直接采用 资产/写作/思考 命名
- 扩展规则：新资产（journal/report）→ 资产职能内部加卡（导航栏不变）；新职能（分析）→ 导航栏加项
- **⚠️ 曾走过的弯路**：最初的"资产组/领域组"分组设计（资产=fiction/memory 两导航项、领域=创作/情绪）被用户推翻——资产从"组"降级为"项"，分组概念消失。不要在导航栏设计中重新引入分组

## 结构即界面（最强设计原则，2026-08-15 两次纠正）

用户对资产类页面的核心要求——**"过度复杂了。直接按照现有的结构来写页面就可以"**：

- **页面结构 = 数据源仓库目录结构**：资产长什么样，页面就长什么样（如 /assets/fiction = 职场言情/校园言情/重生言情 → 1_灵感/2_脚本/4_改稿 目录树，展开/收起 + 计数 + 点击阅读）
- **不发明视图**：漏斗、缺口展示、三线统计卡片、跨数据源联合等抽象视图**全部被否**——"不搞漏斗/缺口/检索等抽象——目录结构本身就是视图"
- **镜子原则**（用户"你觉得我的目的是什么"的答案）：系统是**不加扭曲的观测**——添加 AI 的抽象/解释层，用户看到的就不是自己，是加工过的自己。可观测的前提是观测不被扭曲
- 交互只有三个：展开/收起、点击阅读（只读）、目录旁计数

### 资产结构规则（用户明确纠正，2026-08-15）

解读资产仓库结构前先对齐规则（用户："更准确的规则是..."）：

```
一级文件夹 = 一本小说      职场言情 · 校园言情 · 重生言情
二级文件夹 = 一个阶段      0_日志 → 1_灵感 → 2_脚本 → 3_初稿 → 4_改稿（职场言情 2026-08-15 起为五阶段，0_日志=创作日志）
                          校园另用 1_素材 → 2_提纲；阶段目录照实显示，不假定固定数量
文件编号   = 排序          1_1_咖啡厅重逢 = 排序 1_1；6_2_海边散步 = 排序 6_2
无编号     = 未排序        偷看睡觉 = 等待排序的素材
```

- **编号即排序**：无"正式/候选"之分，哪个阶段都有编号；"2"后缀 = 同排序位置的不同版本；11_1 在 3_初稿 = "编号在、阶段未到"（曾误判为"11_x 缺口"）
- 三线阶段命名不同（职场 灵感/脚本 vs 校园 素材/提纲）——页面照实显示，不统一
- 页面目录树 = 一级文件夹（小说卡片）→ 二级文件夹（阶段，照实命名）→ 文件（编号即排序显示）

### 资产页边界原则（用户明确要求）

**资产类页面只挖掘资产结构、支持发现工作流，不做具体创作活动**：
- 资产页允许：浏览/阅读/状态可见/跳转详情
- 资产页禁止：写作/编辑/创建/改稿——创作活动唯一归属职能页（/create，唯一创作承载）
- 资产页发现缺口（如 11_x 规划中）只做**跳转引导**到职能页，不承载创作动作

## 创作职能页 = 工作流显性化（2026-08-15 三次纠正定稿）

创作职能页（/write，曾名 /create）经历与资产页同样的"过度复杂"纠正，最终定稿为**流程视图**：

- ❌ 第一次（过度复杂）：发明"当前焦点 / 素材池 / 方法速查 / 写作入口 / 四阶段进度"——被否（"也还是过度复杂了。和之前问题一样"）。注意：**素材池 = 2_脚本 真实目录、方法速查 = memory 文档**——都是已存在结构，在创作页再发明概念 = 重复发明
- ❌ 第二次（不足）："同结构可写视图（新建/编辑）"——用户补充"**创作工作流要显性化**"
- ✅ 定稿：**流程视图**——fiction 阶段作为工作流显性呈现：
  - 流程条：`0_日志 → 1_灵感 → 2_脚本 → 3_初稿 → 4_改稿`（0_日志=创作日志，2026-08-15 作者新增；每阶段：语义 + 文件数）
  - 各阶段挂载文件（复用资产目录引擎数据，同资产页数据源）
  - **推进动作**：文件从当前阶段移动到下一阶段（脚本→初稿→改稿）——"写好了脚本"这个动作显式化
  - 新建（指定阶段创建）+ 文件点击 → 写作视图（编辑 + 保存写回）
- 核心：**文件位置 = 流程位置**（真实目录状态即流程状态，不发明状态字段）；工作流显性化 = 流程条 + 推进动作

**泛化原则**："结构即界面/不发明抽象"对**所有职能页**成立——资产页 = 目录镜像只读；创作页 = 流程视图可写可推进。两者都禁止发明概念（漏斗/缺口/焦点/素材池/速查等），真实目录/文档就是视图。

## 思考职能页（/think，2026-08-15 规划定稿）

思绪结构化（思考云原型）——依据 08-14 journal 作者意图（情绪结构化处理器：区分事实/感受/行动；"先做到日志里面"；"不代替思考，而是思考的垫脚石"；"每天不把想法处理干净，很难睡得着"）：

- **日志即输入**：不发明输入界面——思绪天然存在 `memory/journal/default/YYYY-MM-DD.md`，思考页 = 日志的结构化工作台（作者原话"把它先做到日志里面"）
- **三区结构化**：事实（发生了什么）/ 感受（情绪如何）/ 行动（要做什么）——思考云原型的骨架
- **结构化沉淀**：`## 结构化（思考云）`段写回当日日志（不动日志正文叙事）
- **每日清理**：处理状态 = 当天是否完成结构化（对应睡眠动力）
- **边界**：不代替思考（垫脚石——摆出来，不自动结论）；本页写操作仅结构化段
- 演进：日志列表（复用资产目录引擎，memory 契约）→ 手动三区 → 状态标记 → LLM 辅助（复用 cli 情绪维度模式）

## 文档体系（用户偏好，2026-08-15 多次纠正后定稿）

用户对项目文档有明确的结构偏好，改动前先对齐：

### 根文档职责（一次纠正："dev-guide 和 CONTRIBUTING 区分不开"）
- **README.md**：只做入口（项目是什么 + 模块索引 + 快速开始 + 文档导航）——**详细配置收敛到 CONTRIBUTING，README 不重复**（"README 瘦身去重"）
- **STATUS.md**：项目状态（模块版本/数据源/验证，时效性，随提交更新）
- **CONTRIBUTING.md**：**贡献规则**（提交约定 Conventional Commits/验证门禁/分层提交/文档同步）——"必须遵守"类
- **docs/dev-guide/index.md**：**开发知识**（模块结构/环境搭建/命令/调试）——"如何做"类
- **docs/user-guide/index.md**：**面向使用者**（"user guide 不够友好"纠正——写"这是什么/你能做什么/怎么打开/打开后看到什么/FAQ"，非技术语言，不用 dart-define 等术语）
- **AGENTS.md**：AI 工作指南（工作原则/验证命令/提交规范）
- **docs/api-reference/**：index.md（总体构成）+ provider.md + cli.md（按接口拆分，不是单文件）

### 内容组织（一次纠正："不应该写 README，重写"）
- 内容文档放**独立文件**（如 context/default/deliverables.md），README 只做入口 + 文档导航表
- 配置方法**单一权威**：详细配置只在 CONTRIBUTING（或 dev-guide），其他地方引用不复制

### 设计文档（screens.md 模式）
页面级设计单独成文（src/studio/doc/screens.md）：页面总览表 → 每页（定位/数据源/ASCII 界面示意/设计要点/意图对齐）→ 页面共同原则 → 演进顺序。单页专门设计（如小说资产页）可再单独成文 fiction-asset.md。

## 用户偏好（明确纠正）

1. **IaC 不能偷懒**（"iac 严重偷懒了"）：IaC = 基础设施声明（Terraform：bucket/CDN/策略/变量），`run-studio-linux.sh` 只是本地运行脚本不算 IaC。参考要参考全——qtcloud-data/manifests/terraform 是完整模板（6 文件），不是只抄 CI
2. **文档先行**：先 ROADMAP 定目标再实现；页面/模型改动前确认边界（曾因改 main.dart/新增页面被撤回——见 quanttide-delib-governance 的"只允许改 mock 数据"）
3. **发布节奏**：CHANGELOG 先行 → 预检（secrets/资源/tag）→ 打 tag；不可直接打 tag 期望 CI 成功
4. **产品规划验证先行**（2026-08-22 qtcloud-think roadmap 确认）："用出来的才算数"——最小闭环先行（只做高频路径，一周内真实用起来）→ 真实使用 1-2 周验证（**主动用/缺了难受=值得做**）→ 砍掉验证（砍掉不受影响=不值得做）→ 数据驱动（上线后按使用频率砍死功能深化活功能）。高级功能（预测/决策类）无数据支撑就是空壳——延迟到实例/数据积累后再判断。ROADMAP 结构：确定要做的（勾选清单）vs 延迟验证（暂不排期）分开列

## 模板收敛

5 实例（qtdata/qtclass/qtcloud-data/qtcloud-delib/qtcloud-econ）重复同一套三件套 → 已提 issue 收敛为参数化模板（qtcloud-devops#20）：新建客户端 = 复制模板 + 改 3 参数（bucket/域名/tag 前缀）。

## lib 分层 = Bloc 家法（2026-09-24 用官方源核实，纠正一条错判）

**家法 = feature-first**：顶层按功能切，非功能的放 `app/`、`theme/`、`l10n/`；功能内固定四类 + 桶文件：

```
lib/
├── main.dart
├── app/                装配（app.dart、observer）——不属于任何功能
├── l10n/  theme/
└── <功能>/
    ├── bloc/  或 cubit/     状态        （官方全仓示例 245 处）
    ├── models/              该功能的模型
    ├── view/    <功能>_page.dart  页面（单数 view，63 处）
    ├── widgets/             该功能的可复用部件（复数，22 处）
    └── <功能>.dart          桶文件（统一出口）
```

- ⚠️ **`views/` 不是家法**——官方全仓示例 `views/` **0 处**。契约原型里那条案例（2026-09-12「界面件按 Bloc 惯例该叫 `views/`」）与官方材料不符；据此把 qtdata 的 `widgets/` 改名 `views/`（2026-09-24 二段）是错的方向，家法里部件层就是 `widgets/`（本 skill 早前记的「widgets 不是 components」才是对的）
- **架构文档只定层，不定目录名**：三层是 Presentation / Business Logic / Data（Repository + Data Provider）；目录名来自官方示例的实践——引用时别把「层」和「目录名」混成一句
- **别凭印象答「家法怎么切」**：核实方法与官方目录原文见 `references/bloc-house-rules.md`（GitHub API 拉官方 example 真实 tree + 计数，一次调用定性）

## qtdata 四域结构（2026-09-24 用户拍板并当天落地）

```
lib/
├── main.dart                        装配（留根）
├── app/                             跨域共用件（用户把拟名 shared/ 改定为 app/）
└── data/  project/  business/  asset/    四域，域内 models/ views/ screens/
```

- ⚠️ **域内分层用项目自己的规范 `models/ views/ screens/`**（用户原话\"其他维持我的规范\"）——**不是**家法的 `view/ widgets/`。`views/` 这个层名已写进 `CONTRIBUTING.md` 并标注\"**本项目约定**（Bloc 官方示例是 `view/` + `widgets/` 两档，我们合成一层）\"。**用户否掉\"按家法纠正\"后不要再反复推销家法**——按用户规范执行，把偏离如实记成\"本项目约定/项目自定\"，不挂在\"按 Bloc 惯例\"名下（本条与上节不矛盾：上节纠正的是\"别谎称家法叫 views/\"，本节是\"用户有权自定\"）
- 三域从**数据与界面**看出来：seed JSON 矩阵三行键 `project/data/business`；第四域 `asset` = 矩阵（三域 × 五阶段）× 交付物 `Deliverable`（从 `project/models/project.dart` 拆出归此）
- **界面 5 Tab ≠ 三域**：总览是聚合视图、资产 Tab 另有观测入口性质
- **跨域件判定：进 `app/` 前先确认真被两个以上域用到**——只被一个域用就留在那个域。盘查实录：`progress_bar_widget` 只被 `project_card` 用 → 留 `project/`；`phase_tag`／`status_badge`／`section_header` 跨域 → `app/`；`doc_dialog`（资料弹窗，资产矩阵的格与项目 Tab 共用）→ `app/`
- `test/` 同名同构（`test/{域}/{views,screens}`）；跨域测试基础设施留 `test/helpers/`（与 `main.dart` 同理：装配/跨域的东西留根）
- ⚠️ 三域此时**没有枚举/常量表**，只活在 seed JSON 的键里——要变成结构概念，先给域轴一个定义（枚举或契约）
- 完整搬迁配方（脚本化 git mv + 两类 import 重写 + 判\"行为不变\"的静态对比）见 `references/dart-domain-restructure.md`

## 关联

- git 操作/子模块/历史脱敏：`quanttide-git-ops`
- 治理数据流水线与 mock 数据边界：`quanttide-delib-governance`
- 三端结构（Rust CLI / Go Provider 与本端并列，种子数据各自维护）：`quanttide-app-stack`
