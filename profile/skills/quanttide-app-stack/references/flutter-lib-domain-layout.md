# Flutter studio 的 lib/ 按域重排 + 核查「框架家法」的取证法

触发：给 Studio 的 `lib/` 按域（或职能）分文件夹；或有人（含自己）拿「某某框架惯例」当理由要求改名。

实证：qtdata studio，2026-09-24（四域重排，35 → 36 个 dart 文件、45 个文件搬迁）。

## 一、先取证，再谈「家法」

**不许凭印象说「某框架惯例是 X」**。取证路径（官方仓库的示例目录树就是权威）：

```bash
# 官方示例的真实目录（api.github.com 直连可用；raw.githubusercontent 若被墙就用 contents API）
curl -s "https://api.github.com/repos/felangel/bloc/contents/examples/flutter_todos/lib"
# 全仓统计某个目录名出现次数（自己数，别猜）
curl -s "https://api.github.com/repos/felangel/bloc/git/trees/master?recursive=1" \
  | python3 -c "import json,sys;t=[i['path'] for i in json.load(sys.stdin)['tree']];print(len([p for p in t if '/views/' in p]))"
```

**Bloc 官方（2026-09-24 实测）**：

| 目录 | 全仓示例出现次数 |
|---|---|
| `bloc/` | 245 |
| `view/`（页面，`<feature>_page.dart`） | 63 |
| `widgets/`（可复用部件） | 22 |
| `views/` | **0** |

结构（`examples/flutter_weather`、`examples/flutter_todos` 一致）：

```
lib/
  main.dart / bootstrap.dart / app/        ← 装配与非功能（l10n/ theme/）
  <feature>/
    bloc/ 或 cubit/   状态
    models/           该功能的模型（模型在功能内部，不在顶层）
    view/<feature>_page.dart   页面（单数 view）
    widgets/          该功能的部件（复数）
    <feature>.dart    桶文件（统一出口）
```

官方**没有 `screens/`，也没有 `views/`**；两个教程（flutter-weather / flutter-todos）提到的目录名正好是 `bloc`、`cubit`、`models`、`view`、`widgets`。

**结论对契约的影响**：`quanttide-code` 契约那条案例（2026-09-12「`qtcloud-work/studio` 界面件那层该叫 `views/`——按 Bloc 惯例」）**站不住**：官方是 `view/`（页面）+ `widgets/`（部件）两档。qtdata 在 2026-09-24 前半天照那条案例把 `widgets/` 改成了 `views/`——**方向反了，责任在办事的人（改之前没取证）**。

用户裁决（2026-09-24）：**维持我们自己的规范（单层 `views/`）**，但**语义上必须诚实**——写成「本项目约定，偏离家法的点是合成一层」，不许挂在家法名下。已在 qtdata 的 `CONTRIBUTING.md` / `STATUS.md` 这样落。契约正文的修订属另一仓（`domains/quanttide-code`），未获指令不动。

## 二、qtdata 定稿的域布局（用户拍板）

```
lib/
  main.dart              装配，留根
  app/                   跨域共用：侧栏/断点、区块标题、状态徽章、阶段标签、
                         提示、资料弹窗、详情页外壳、总览 Tab
  project/  data/  business/  asset/      四个域
                          每域内部：models/  views/  screens/
  test/{app,data,project,business,asset}/{models,views,screens}/
  test/helpers/          跨域测试基础设施，留根
```

规矩（写进 CONTRIBUTING）：

- 一个文件只属于一个域；**进 `app/` 前先确认它真被两个以上域用到**（只被一个域用就留在那个域）——本轮据此把 `progress_bar_widget.dart` 留在 `project/`，其余 9 件（侧栏、断点、区块标题、状态徽章、阶段标签、toast、资料弹窗、详情页外壳、总览 Tab）进 `app/`
- 域判据来自数据形状，不靠感觉：交付资产矩阵的行键就是 `project / data / business`（三域 × 五阶段 = 15 格），矩阵与交付物归 `asset/`
- 单文件 ≤250 行、测试同名同构等旧规矩照旧
- 撤掉 `screens/tabs/` 子目录（每域只剩一个 Tab，单文件子目录没意义）——**这类顺手的结构调整要在报告里点出来**，用户可能推翻

## 三、迁移配方（可脚本化，逐条都是踩过才知道）

1. **先建全量映射表再动手**：从 `git ls-files` 列全部 dart 文件 → 写 `旧路径 → 新路径` → 校验「源文件都在」「目标无冲突」。**漏一个文件要到 `flutter analyze` 才暴露**（本轮漏了 `test/screens/dashboard_screen_test.dart`，报出 61 个错）。
2. `git mv` 搬（保历史——45 个文件 git 全部识别为重命名），先 `mkdir -p` 目标目录。
3. **重写 import 两类都要管**：
   - `lib/` 内是**相对** import（本轮 88 条）；`test/` 常用 **`package:<pkg>/...`** 绝对 import（12 个文件）。只处理一类 → 大批 `uri_does_not_exist`。
   - 相对 import 的算法：按**旧位置** resolve 目标 → 查映射表 → 按**新位置** `os.path.relpath` 回算；路径没变就不动（减少 diff 噪声）。
   ```python
   tgt_old = os.path.normpath(os.path.join(os.path.dirname(rel_old), imp))
   tgt_new = M.get(tgt_old, tgt_old)
   newimp  = os.path.relpath(tgt_new, os.path.dirname(rel_new))
   ```
4. **把一个类拆出去**（本轮 `Deliverable` 从 `project/models/project.dart` → `asset/models/deliverable.dart`）：新文件建好；**原文件加对新文件的 import**（它仍要 `List<Deliverable>`）；只有**点名叫那个类**的文件才需要额外 import——先 grep 一遍引用面，往往只有定义处本身（本轮 `Deliverable` 仅在该文件内出现 6 次，零外部使用者）。
5. 清空目录（`rmdir`；git 不跟踪目录，留着是噪音）。
6. 门禁三连定位残余：`flutter analyze`（`uri_does_not_exist` 直接指出漏改）→ `dart format --set-exit-if-changed` → `flutter test`。

## 四、「行为不变」的四个可比信号（自做或委派 pi 后复验都用这套）

挪文件的重构，别只看「测试绿了」，拿四组数当证据：

| 信号 | 本轮实测（改前 → 改后） |
|---|---|
| 测试文件数 / `expect(` 计数 | 12 个 / 105 → 12 个 / 105（**断言未削弱**） |
| `lib/` 中文字符串字面量种类 | 71 → 71，无增无减 |
| 类名差集 | 只有预期的私有→公开改名（`_QuotationCard`→`QuotationCard`、`_MatrixCell`→`MatrixDataCell` 撞模型类） |
| 文件行数分布 / 最长文件 | 最长 381 → 235（全部 <250） |

一条命令式的做法：用 `git show HEAD:<path>` 取旧内容、读新文件，取集合做差。**字面量种类一致**是很强的「行为没动」信号——界面文案、颜色、提示语一个都没变。

## 五、文档同步清单（改完结构必查）

`README.md`（结构块与形态表）、`CONTRIBUTING.md`（结构与规矩）、`STATUS.md`（规模表 + 结构契约逐条判定）、`TODO.md`（该段标为已落地 + 影响路径）、`ROADMAP.md`（分段表 / 现在在哪 / 待决里的路径）、仓库级 `<app>/STATUS.md`（组件小节）。

**`CHANGELOG.md` 里的历史条目不改**（旧路径是当时的事实）——要说明另加，别改历史。

## 六、Pitfalls

1. **把「框架惯例」当权威去覆盖用户规范**：先取证（第一节）；用户明确「维持我的规范」时，别再拿家法劝第二次——只把偏离写成显式约定。
2. **只想改一类 import**（相对 vs `package:`）——两类都要。
3. **漏文件**：映射表要对全量文件，别凭印象挑；analyze 是兜底但不是第一道。
4. **顺手做的结构调整（撤子目录、留根的文件）要显式报出来**，别让它藏在 diff 里。
5. **拆类的连带 import**：忘了给原文件加 import，analyze 会报 `Undefined class`。
6. **本机门禁会改 `pubspec.lock`**：`flutter pub get` / `analyze` 会把锁文件的 URL 改写成 `PUB_HOSTED_URL` 镜像（如 `pub.flutter-io.cn`）——提交前 `git status` 看一眼，别把它夹带进去。
