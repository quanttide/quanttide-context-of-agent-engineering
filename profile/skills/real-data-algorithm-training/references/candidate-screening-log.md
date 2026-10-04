# 候选筛选实录（2026-10-01）

用户问「找一个实际案例做最简单的算法训练」时扫过的全部候选、实测数字与淘汰理由。**下次先读这份，别重扫。**

## 逐个候选

| 候选 | 数据与标签 | 实测 | 结论 |
|---|---|---|---|
| GHTorrent PR 面板 | 524,485 行 / 34,761 作者 / 2011–2021；标签＝当天有无 PR 被合并 | 四档 AUC 0.678（全部）/ 0.777（自有仓库）/ 0.852（组织仓库）/ 0.769（外部仓库，正例 3.8%，准确率 0.960 **低于**基线 0.962） | **淘汰**——标签是观测结果，没有标准答案可对，用户原话「这个算法结果没有办法直接验证」 |
| 四类记忆分类 | `quanttide-think/apps/qtcloud-think/tests/fixtures/journal` 的 `category` 字段：episodic 16 / semantic 15 / self 14 / procedural 8 | **0.265 < 基线 0.302** | **淘汰**——无信号，标签是作者的写作意图，`title`+`description` 里没有可学的差异 |
| 提交类型分类 | 20 个仓 4221 commits；chore 3313（78.5%）、docs 471、feat 254、refactor 102、fix 65、ci 7、test 5 | 全猜 chore = 78.5% | **淘汰**——严重失衡，第一课会教出坏直觉 |
| 文档板块分类 | 各仓 `data/<层>/<板块>/`，12 个板块 | 0.474（基线 0.378） | **淘汰**——类太杂、噪声太大 |
| 文档分层分类 | 各仓 `data/<层>/` 目录名即标签 | 单仓 230 篇 7 类 **0.583**（基线 0.300）；全仓 1210 篇 8 类 **0.656**（基线 0.321） | **中选** |

## 中选者的画像

八层分布：profile 388 / journal 271 / context 159 / insight 140 / intention 85 / report 77 / roadmap 60 / brochure 30 = **1210 篇**（跨全部领域仓；只扫 quanttide-tech 时是 5 类 195 篇，类别数随搜索范围变化）。

三档难度台阶（同一份数据切粗细分层）：

- 日志 vs 洞察：2 类 411 篇 → **0.898**，基线 0.659
- 内容流向四层（日志/洞察/意图/路线图）：4 类 556 篇 → **0.793**，基线 0.487
- 八层全量：8 类 1210 篇 → **0.656**，基线 0.321

混淆矩阵几乎全是干净对角线，唯一成规模的错误在 **journal ↔ profile**（9 例）——档案本来就由日志提炼而来，它们同源。这条「错误集中在语义真正相邻的两类」是可讲解性最强的证据。

## 两个顺手发现

- **私有侧不同构**：`quanttide-tech-private` 有 1027 篇 md，但按业务线/职能/人名组织（qtadmin-private 432 / qtclass-private 242 / qtdata-private 154 / qtrecurit-private 74 / qtcloud-private 64 / qtconsult-private 51），**没有层目录**，所以层分类实验扩不过去。私有侧自己的标签轴是职能，但素材含人名，不适合放进公开实验室。
- **标签会漂移**：文档会在层之间搬家（草稿定稿后归位），所以分类器只能当建议器，不能当裁判。这本身是教学点。

## 落档位置

`quanttide-algorithm` 域：实验记录在 `data/context/laboratory/`（README + 可复现脚本，三档数字已实测复现）；资产档案在 `data/profile/quanttide-asset/asset-classifier/`（index / requirement / specification / implementation / evaluation / feedback 六件套）；候选入口判据在 `data/insight/candidate-selection.md`；域意图在 `data/intention/incubation/index.md`。
