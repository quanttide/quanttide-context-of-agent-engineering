# Studio 界面数据（seed JSON）换成真实案例（2026-09-24 qtdata 实证）

**触发**：用户贴一段真实业务素材/案例后说「使用这个最新案例替换 qtdata 当前的模拟数据」。

原则：**这是数据替换，不是重构**——不动模型与字段结构，把新案例填进现有形状；改动面压到最小，并保留「行为可比」的信号。

## 一、先摸清耦合面（三步，别先动 JSON）

1. **结构**：通读 `assets/data/seed_projects.json`。本例顶层 `{"projects":[...]}`，单项目字段：`id / _comment / name / client / created / updated / status / currentPhase / contractAmount / business{costBased,marketBased,pricingNote,contractDate,payments[]} / deliverables[{name,status}] / matrix{rows[],columns[],cells{row_key_col_key:{name,status}}} / blueprint{steps[],exceptions[{label,strategy}]} / phases[{name,status,items[{name,desc,hasDoc,type}]}]`
2. **模型**：逐域读 `lib/<域>/models/*.dart` 的 `fromJson`，记下**必填 vs 可空**（本例 `business` 可空、`Payment.date` 默认空串、`contractAmount` 必填 double）+ 注释里的单位语义（金额=万元）
3. **断言**：`grep -rn "seed" lib test` 找读取入口——本例是 `test/helpers/seed.dart`（`File(...).readAsStringSync()` 同步读包根，绕开 rootBundle 的 future 悬挂）× 7 个测试文件，外加 `dashboard_screen_test` 走 rootBundle 的独立入口。再 `grep -rn "<旧项目名>\|<旧客户名>" test lib` 确认 **lib 里没有硬编码案例内容**（本例干净，全部来自 seed）。把这些内容耦合点逐条列出来：项目名／客户／更新日期／交付物计数／完成度%／阶段名与每阶段项数／矩阵格名／蓝图关键词／结款文案

## 二、设计新 seed：让「结构数值」保持不变

这是缩小改动面的关键手法——**新案例按老骨架填，让数值型断言原样成立**：

- 交付物仍 4 项（3 done + 1 active）→ 完成度仍是 75%
- 时间线仍 6 阶段，项数分布仍 2/3/2/2/2/3、`hasDoc` 仍 11 项 → 「2 项 ×4」「3 项 ×2」「查看资料 ×11」全不动
- 矩阵仍 3 维（项目/数据/商务）× 5 阶段（调研/谈判/实施/验收/复盘）= 15 格，只是每格名字换成新业务的真实动作
- 列状态与格状态要自洽（项目已到实施 → 谈判列应为 done，验收/复盘列 todo）

结果：**只有文案型断言要改**，断言条数不变（本轮 105 处不变）——这是「行为没被削弱」的可比信号，汇报时给出来。

## 三、脱敏与「示例数字」处理

- 客户名不写实名 → 「某医疗器械企业」这类（对齐 journal/内容的脱敏口径）
- 未知的商务数字（成本法/市场法/合同额/结款）**填示例值，并在两处显式标注**：JSON 的 `pricingNote` + 顶层 `_comment`（「示例数据，非实际报价，待商务确认后替换」）。studio 是公开部署的，示例值必须能让读者认出是示例
- 同步落两份文档：`TODO.md` 补一条**带可跑判据**的欠账（例：`grep -n "示例数据，非实际报价" src/studio/assets/data/seed_projects.json` 无输出；影响面写清 `test/**` 里依赖数字的断言）；`STATUS.md` 加「界面数据」小节记来源／内容／示例数字／断言同步情况
- 另可加一条「案例内容随案例库同步」的待办：seed 的交付物/矩阵格/阶段名要能对上 `docs/gallery/<产品>/` 的案例页

## 四、门禁与上线（两件事都要说）

```bash
cd src/studio
dart format --set-exit-if-changed lib test assets   # 期望 0 changed
flutter analyze                                     # 期望 No issues found
flutter test                                        # 本例 21/21
```

- 逐条 diff 测试文件，确认没有把**数值型断言**顺手改成写死的新值（那就失去了可比性）
- CI 的 `deploy-studio.yml` **只在 `studio/*` tag 上部署** → 分支推送只跑门禁，**线上仍是旧数据**。报告里必须点明「要上线须打 tag 发版，版本号用户拍板」，别让用户以为推完就换了

## 五、别做的事

- 不改模型/字段结构去迁就新案例——新案例按现有字段填；字段不够用说明素材不足，写进欠账
- 不把旧案例数据留成 JSON 里的「第二个项目」——用户说的是替换
- 不在 seed 里写真实价格或客户实名（除了脱敏，还因为 seed 随 Web 构建一起对外可下载）
