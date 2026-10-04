# Bloc 家法目录与命名（官方源核实，2026-09-24）

用途：被问「按 Bloc 的最佳实践该怎么切/该叫什么」时，别凭印象——下面是官方仓库实测结果与核实命令。

## 一、家法长什么样（官方示例真实 tree）

`felangel/bloc` 的两个官方示例（架构文档直接引用）：

```
examples/flutter_weather/lib/
├── main.dart  app.dart  flutter_weather_bloc_observer.dart
├── search/{search.dart, view/search_page.dart}
├── settings/{settings.dart, view/settings_page.dart}
└── weather/
    ├── cubit/weather_cubit.dart  weather_state.dart  weather_cubit.g.dart
    ├── models/models.dart  weather.dart  weather.g.dart
    ├── view/weather_page.dart
    ├── widgets/weather_empty.dart  weather_error.dart  weather_loading.dart  weather_populated.dart  widgets.dart
    └── weather.dart                    ← 桶文件
```

```
examples/flutter_todos/lib/
├── app/{app.dart, app_bloc_observer.dart}  bootstrap.dart  main_*.dart
├── l10n/  theme/
├── home/{cubit/home_cubit.dart, view/home_page.dart, view/view.dart, home.dart}
├── edit_todo/{bloc/{edit_todo_bloc,event,state}.dart, view/edit_todo_page.dart, view/view.dart, edit_todo.dart}
├── stats/{bloc/…, view/stats_page.dart, stats.dart}
└── todos_overview/
    ├── bloc/todos_overview_{bloc,event,state}.dart
    ├── models/models.dart  todos_view_filter.dart
    ├── view/todos_overview_page.dart  view.dart
    ├── widgets/todo_list_tile.dart  todos_overview_filter_button.dart  widgets.dart
    └── todos_overview.dart
```

**规律**：顶层按功能切（feature-first）；功能内 `bloc/`（或 `cubit/`）+ `models/` + `view/`（页面，单数）+ `widgets/`（部件，复数）；每层一个桶文件（`view.dart`/`widgets.dart`/`models.dart`），功能再一个桶（`<功能>.dart`），外部只 import 桶。非功能的横切放顶层：`app/`、`theme/`、`l10n/`。

## 二、计数证据（谁才是家法里的名词）

官方仓库全量 paths 统计（`git/trees/master?recursive=1`）：

| 目录名 | 命中路径数 |
|---|---|
| `bloc/` | 245 |
| `view/` | 63 |
| `widgets/` | 22 |
| `views/` | **0** |
| `screens/` | 0（示例里页面就是 `view/*_page.dart`） |

教程（flutter-weather / flutter-todos / flutter-counter / github-search）出现的目录名同样只有 `bloc`、`cubit`、`models`、`view`、`widgets`。

**结论**：界面**页面**层叫 `view/`，**部件**层叫 `widgets/`；**没有 `views/`，也没有 `screens/`**。

## 三、核实方法（一条命令定性，别靠训练印象）

GitHub API 直连可用（`api.github.com` 通常直连通，不必走代理）：

```bash
# 官方示例的真实目录
curl -s "https://api.github.com/repos/felangel/bloc/contents/examples/flutter_todos/lib"
# 全仓 paths（拿计数）：git/trees 递归列表，再按目录名过滤
curl -s "https://api.github.com/repos/felangel/bloc/git/trees/master?recursive=1"
# 文档原文（base64）：contents/docs/src/content/docs/{architecture.mdx,naming-conventions.mdx}
```

`raw.githubusercontent.com` 走 curl 可能取不到内容（本次实测空响应），**改用 `api.github.com/.../contents/<path>` + base64 解码**即可。

## 四、架构文档说的是「层」，不是「目录名」

`architecture.mdx` 只定义三层：**Presentation / Business Logic / Data（Repository + Data Provider）**，并给出仓库/数据提供者的职责。目录名（`bloc/ models/ view/ widgets/`）来自官方示例的实践——引用家法时要分清引的是哪一头，别把「层」当作「目录名」的权威。

`naming-conventions.mdx` 另有命名约定（可选，但大型项目推荐）：事件用**过去式**（`BlocSubject + Noun + Verb`，首次加载 `BlocSubjectStarted`）；状态用**名词**——多子类时 `BlocSubject + Verb + State`（Initial/Success/Failure/InProgress），单类时 `BlocSubject + State` + 枚举 `BlocSubject + Status`。

## 五、落进量潮 studio 的实际形态（2026-09-24 用户拍板并已实施——**别再按家法"纠正"回去**）

用户看过上面原文后的裁决：**域切分照家法（feature-first），域内分层维持量潮自己的规矩**（不采纳 `view/` + `widgets/` 两档）：

```
lib/
├── main.dart              装配（留根）
├── app/                   跨域共用（用户拍板：跨域那层叫 app/，**不叫 shared/**）
├── project/  data/  business/  asset/    四域，每域内部 models/ views/ screens/
└──（asset 域收交付物 Deliverable 与交付矩阵「三域 × 五阶段」；Tab 归本域 screens/，撤掉 tabs/ 子层）
```

- 界面部件那层叫 **`views/`**——这是**本项目显式约定**（`src/studio/CONTRIBUTING.md` 已写明"Bloc 官方示例是 `view/` + `widgets/` 两档，我们合成一层"）。**不要再按家法把它改回 `widgets/`**：换名等于推翻用户裁决，且组织级契约案例目前写的也是 `views/`
- `test/` 同名同构（`test/{域}/{层}/`）；跨域测试基础设施留 `test/helpers/`、装配留 `lib/main.dart` —— 同一原则：**跨域的留根，属域的进域**
- `domains/quanttide-code/.../contract.md` 那条案例（"界面件该叫 views/"）在官方材料里站不住（全仓 0 处 `views/`）——我们照旧用 `views/`，但**理由是项目约定，不是家法**；契约案例本身待修（在别的仓，未动）
- **教训（本节就是它的产物）**：二段照契约案例把 `widgets/` 改成 `views/`，方向反了。引「家法／惯例／最佳实践」这类权威结论前先核原文——尤其这类会被下一个项目照抄的目录命名；核完若与内部契约冲突，**把冲突摆给用户裁决**，别自行选一边

## 六、按域重排 lib 怎么搬（2026-09-24 qtdata 实证，44 文件）

域切分的搬迁是机械活但易漏，完整配方（可复制脚本 + 三个坑）见 `roadmap-stage-execution` 的 `references/bulk-dir-restructure.md`。三条最容易踩的：① **lib 用相对 import、test 用 `package:` 绝对 import**——两类都要重写；② 映射表要覆盖**全部**文件，`flutter analyze` 的报错是漏项探测器（本次漏了一个测试文件）；③ 验证靠静态对比（断言数 / 中文字面量集合 / 类名差集 / 最长文件），不靠"看起来对"。
