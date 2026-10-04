# quanttide-delib 议事领域架构速览（2026-08 状态）

处理 delib 相关任务（契约迭代、决议子系统开发、roadmap 更新）前先读本速览，避免重读全部文档。

## 领域边界

- **quanttide-delib** 只承载**议事规则**（议题怎么走流程）：契约、流程、议事云产品、案例、档案
- **quanttide-org** 承载**企业治理实际情况**：章程（王—两院三权并列）、设计意图（君主立宪制）、治理手册（周会/周报/目标管理）
- 用户拍板过的边界：delib 不考虑治理实际情况；治理机构演进（书记处/秘书处/执行副总）不属于 delib 的过时问题

## 契约 v0.0.7（specification/contract.yaml）核心

- **七节点固定骨架**（fixed_skeleton，不可变）：议题创建 → 动议提出 → 附议扩散 → 辩论 → 投票 → 决议形成 → 归档沉淀
- **四独立实体**（不混入议题类型）：
  - 议事规则 Rule：常驻规则配置（版本化/修订记录/生效状态），修订经提案议题产出
  - 议程 Agenda：会议骨架（时间/参与者/议程项[关联议题]），会议编排
  - 议题 Issue：两类模式（见下），走七节点生命周期
  - 决议 Resolution：议事结果，独立档案统一治理
- **两类议题模式**（stage_requirements 配置 required/optional/skipped）：
  - 研讨：附议=参与确认、辩论=必经（开放深度讨论）、投票=skipped、决议=optional
  - 提案：附议=封闭表态（附议/反对/弃权）、辩论=optional（冲突时）、投票=required、决议=required

## 决议可见性三通道（解决"看不到决议"的最大管理困难）

1. **治理视图**：决议总览（状态/责任人/时间）——04-resolution 组件核心
2. **决议播报**：会议开场自动播报未结决议——02-meeting 联动
3. **周会决议汇总**：周会材料自动生成决议汇总——07-weekly

## 产品与开发分期（roadmap/qtcloud-delib.md）

- 定位：以**决议为中心**、以议题为过程的议事管理 SaaS
- **MVP（决议子系统，先行）**：00 foundation → 04 resolution → 05 archive（独立闭环，不依赖议事流程）
- **V1（议事子系统）**：01 issue → 02 meeting → 03 vote + 06 role + 07 weekly（决议从议题决议形成节点自动汇入档案）
- **V2**：08 audit + 09 classroom + 10 ai-advisor（AI 三角色模拟辩论，理念见 intention/delib-cloud.md）
- 产品意图：intention/qtcloud-delib.md（定位/核心问题/数据策略/实施路径）

## 数据策略（自建系统）

- 档案库 = 单一事实源（决议 YAML 一决议一文件 + index.yaml）
- **飞书单向导入**：导出 → 字段映射 → 落档案；不反向写飞书
- 实验区：`data/context/laboratory/`（schema.yaml + 示例决议 + scripts/gen_index.py：逾期推导 + 治理视图）
- 逾期为推导状态（due < today 且未完成），不显式存储
- 导入脚本：`laboratory/scripts/import_feishu.py`（飞书导出 md → 决议 YAML；运行 `uv run --with pyyaml`）

### 飞书真实数据发现（导入实验，2026-08-04）

飞书"议事档案"知识库（space 7564357713069883393）：首页 / 议题（12 个旧类型节点）/ 决议（含"执行管理"）。议题文档真实形态：标题 + 动议区（带 @用户）+ `<poll>` 投票。

| 发现 | 对 schema 的影响 |
|------|------------------|
| 飞书决议/提案**无完成期限**字段 | `due` 缺失 → 逾期推导跳过（gen_index 需容错） |
| 状态用 **【已废止】标题前缀** + 删除线表达 | 导入时映射为 status=已废止 |
| 投票是 `<poll>` 块 | 映射为 `vote.present` |
| 导出标题格式不一（`# ` vs `<title>`，正文有 `# 动议区` 干扰） | 解析先匹配 `<title>` 再匹配 `# ` |

导出命令：`lark-cli drive +export --token <obj_token> --doc-type docx --file-extension markdown`（详见 lark-cli 技能）。

## 文档地图（data/context/）

```
README.md（索引）· index.md（子领域/设计规则）· workflow.md（七节点流程）
specification/contract.yaml（契约 v0.0.7）
intention/qtcloud-delib.md（产品意图）· intention/delib-cloud.md（AI 三角色理念）
roadmap/qtcloud-delib.md + components/（11 组件定义）
gallery/（案例）· handbook/（流程手册）· profile/（议事档案分类）
laboratory/（实验区）· CHANGELOG.md
```

## 迭代习惯

- 契约版本升级：`spec_version` 字段 + 头部注释 + CHANGELOG 记录（v0.0.3→v0.0.7 演进：固定骨架→决议独立→规则/议程独立→两类模式→七节点）
- 每次契约变更**分层同步**：contract → workflow → index → roadmap 主文档 + 组件 → README → CHANGELOG
- 改完 `grep -rn "旧概念"` 清理残留（曾清理"九种类型/五节点/无辩论区"）
