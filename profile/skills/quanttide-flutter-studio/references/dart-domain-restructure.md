# Dart/Flutter 目录重排：按域切片 + 全量 import 重写（qtdata 2026-09-24 实证）

用户指令形如「lib 分解成 data project business 三个文件夹，每个文件夹下的分层和现在的规则一样」。
45 个文件搬迁、88 条相对 import + 12 个测试文件的 `package:` import，一次跑完、一次通过——**不要手工逐个改 import**。

## 0. 先摸清两件事（决定重写策略）

```bash
# ① import 是相对还是 package: 绝对？——决定重写算法
grep -rn "^import '" lib/ test/ | grep -v "package:flutter\|dart:" | head
#   实证：lib/ 88 条全是相对（'../models/project.dart'），test/ 却用 package:qtdata_studio/models/project.dart
#   → 两类都要处理，只写一类必挂一半
# ② 目标形态里哪些文件跨域？——按「被几个域用到」判定，不按名字猜
```

**跨域件判定先查引用者**（别按直觉把"看着通用"的都丢进 app/）：对每个候选文件列出 import 它的文件，按域归类，只被一个域用的留在那个域。实证：`progress_bar_widget` 只被 `project_card` 用 → 留 project/；`section_header` 被四域用 → app/。

## 1. 建映射表（唯一事实源）

写成 `{旧路径: 新路径}` 的 JSON 落 `/tmp/<repo>-map.json`，先跑三条校验：源文件都在、目标无冲突、条数对得上**清单**。

```python
missing = [o for o in M if not os.path.exists(os.path.join(root, o))]
dups    = [v for v in M.values() if list(M.values()).count(v) > 1]
```

## 2. 搬迁 + 重写相对 import（一条脚本）

```python
# 1) 先收集所有 dart 文件的原内容（跳过 build/ .dart_tool/ 平台目录/ doc/）
# 2) os.makedirs(dirname(new)) + git mv old new        ← git 会识别为 rename，历史保留（44 文件全 R）
# 3) 对【每个】文件（含未搬迁的 main.dart）重写 import：
#    tgt_old = normpath(join(dirname(rel_old), imp))    ← 按【老位置】解析
#    tgt_new = M.get(tgt_old, tgt_old)                  ← 映射到新位置
#    new_imp = relpath(tgt_new, dirname(rel_new))       ← 按【新位置】回算相对路径
#    if not new_imp.startswith('.'): new_imp = './' + new_imp
#    只在新旧不同时才写回（减少 diff 噪音）
# 4) 再单跑一遍 package: 开头的：把 'package:<pkg>/<旧lib相对路径>' 整体替换为新路径
```

- 两步必须都做：**相对 import 用重构算法，`package:` import 用字符串映射**（`package:` 不含位置信息，直接查表替换）
- 空目录要自己 `rmdir`（git mv 不删空目录，git 也不跟踪目录）
- 拆出一个类到新文件时（如 `Deliverable` 从 `project.dart` 拆到 `asset/models/deliverable.dart`）：先查**谁真的按名字用了这个类**——实证全仓只有定义处内部用，则新文件加 import 即可，其余文件零改动

## 3. 陷阱（本轮真踩到的）

1. **盘到 12 个测试文件却只映射了 11 个** → 漏掉的那个（`test/screens/dashboard_screen_test.dart`）在新树里孤立，`flutter analyze` 报 `uri_does_not_exist`。**搬迁前先 `find` 出完整文件清单并与映射表条数对账**，别用记忆里的数量
2. **`package:` import 只在 test/ 出现**，第一轮只处理相对 import → 一半测试文件指向老路径。**先 grep 两类 import 的分布再写脚本**
3. **`frontend` 平台目录（ios/android/...）与 `doc/` 别扫进去**（扫了会误改或误报），walk 时显式 skip
4. **文档里的路径一并过时**：改完 `grep -rn "lib/widgets\|lib/views\|lib/models" --include=*.md src/` 扫 README/STATUS/TODO/ROADMAP/CONTRIBUTING——实证要同步 6 个文件的路径引用；**CHANGELOG 的历史条目不动**（那是当时的事实）

## 4. 判"行为不变"：纯挪文件/拆件有四条便宜证据（比 worktree 对差快）

一条脚本跑完，前后各跑一次，数字直接写进提交信息：

| 证据 | 取法 | qtdata 实测 |
|---|---|---|
| 断言数 | `grep -c "expect("` 求和（lib 无、test 有） | 105 → 105 |
| 中文字面量集合 | 正则抽 `'…中文…'` 去重后做集合差 | 71 种，丢掉 0 / 新增 0 |
| 类名差集 | `^class \w+` / `^enum \w+` 集合差 | 只多出必要的私有→公开改名 |
| 最长文件 | 逐文件行数排序前 5 | 381 → 235 |

集合差**必须为空或能被"本次意图"逐条解释**（实证：新增 `ProjectList`／`MatrixDataCell` 正是拆件与撞名改名，属预期）。
再加门禁三连（`dart format --set-exit-if-changed` / `flutter analyze` / `flutter test`）+ `grep -rn "旧目录名" lib test` 清零。

⚠️ 本机跑 `flutter analyze` 会把 `pubspec.lock` 的 URL 改写成 `pub.flutter-io.cn` 镜像——**提交前 `git status` 确认它没被带上**。

## 5. 私有类跨文件后必须转公开

从 tab 里把 `_QuotationCard` 搬进 `views/` 后：类名去下划线转公开 + 按 `use_key_in_widget_constructors` 加 `super.key`。
**撞名要另起名**（实证 `_MatrixCell` → `MatrixDataCell`，因为模型类已占 `MatrixCell`）——这类"超出只改路径"的动作要在提交信息和 TODO 里点名，别混在搬迁里悄悄做。
