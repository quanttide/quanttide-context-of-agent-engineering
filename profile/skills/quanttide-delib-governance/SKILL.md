---
name: quanttide-delib-governance
description: 处理量潮议事数据→决议档案→治理视图→周会报告（qtadmin-delib 私有仓库）时触发。
---

# 量潮议事治理数据流水线

处理量潮议事数据（飞书）→ 决议治理信息（档案/视图/报告）的完整流程。位置：私有仓库 `qtadmin-delib`（quanttide-tech-private/data/qtadmin-private/data/qtadmin-delib）。

## 领域模型（用户建模，已被纠正确认）

- **议题 Issue**：过程实体，两类模式——研讨（开放深度讨论）/ **提案 Proposal（=议题的提案模式，非独立实体）**：动议→附议（封闭表态）→辩论（冲突时）→投票→决议
- **决议 Resolution**：结果实体，独立管理（统一治理）；owner 必填（"未指定"=责任缺口）
- **议程 Agenda**：会议骨架；**纪要=议程的记录形态，非独立实体**
- 状态机：显式（待执行/执行中/已完成）+ 推导（逾期 = due < today 且未完成，不存储）

## 仓库结构（qtadmin-delib）

```
assets/            飞书导出语料（议题/提案、会议档案/周会纪要、工作报告/简报）——单一事实源
src/
├── config.py      路径常量
├── importer.py    Stage 2 导入：提案/纪要 → 决议 YAML
├── governance.py  Stage 3：索引 + 逾期推导 + 治理视图
└── reporter.py    Stage 4：周会汇总 + 质量指标（编号错位/辩论记录/决议落档）
data/resolutions/ 一决议一文件 + index.yaml
data/report/      周会决议汇总、实验基线报告
docs/domain-models.md  领域模型文档（开发/设计/治理三层契约）
examples/         质量审计实验（诊断碎片，非交付物）
```

## 流水线命令

```bash
cd ~/repos/quanttide-tech-private/data/qtadmin-private/data/qtadmin-delib
uv run --with pyyaml python3 -m src.importer          # 导入：assets → data/resolutions/
uv run --with pyyaml python3 -m src.governance --today 2026-08-06   # 治理视图
uv run --with pyyaml python3 -m src.reporter --week 2026-W33        # 周会报告
```

注意：系统 python 无 pyyaml（PEP 668）——一律 `uv run --with pyyaml`。若 `uv run --with` 报依赖解析失败（如镜像 403 Forbidden），改用仓库 .venv 直接装：`uv pip install --python .venv/bin/python pyyaml --index-url https://pypi.org/simple`，之后 `.venv/bin/python -m src.xxx`。

治理视图产物：`src.governance` 除 index.yaml 外输出 `data/resolutions/governance.json`（Studio 客户端消费的治理视图数据：stats/overdue/by_owner/resolutions）。

## 用户偏好（四次明确纠正，必须遵守）

1. **完整流水线，不是散落实验**（"实验方向很偏，我要你组织完整流程"）：任务组织为 采集→导入→治理→输出→回流 一条链；质量审计实验是诊断辅助，不是交付物
2. **私有先行，勿反复提回灌公开契约**（"你为什么老是抓着公开契约不放"）：qtadmin-delib 的系统建设是终点本身；公开 delib 契约同步只做用户明确要求
3. **场景要真实业务性，非展示性**（"场景还不够真实"）：数据/服务必须服务真实业务问题（如"看不到决议"），不做给人看的展示场景
4. **先理解整体蓝图，勿碎片化执行构想**（"反思为什么总是搞不对"→"不对"）：用户每次给方向是在**勾勒一个产品构想**（如"量潮科技数字化"整体项目、开发 qtdata Studio 客户端），不是一个个孤立任务。把每个词合理化后立即规划执行 = 反复被纠正。正确姿势：先复述整体蓝图对齐（用户说"yes/符合"≠ 开始执行，是设计继续），他明确说做什么就直接做，不问"改造还是新建"之类暴露没理解的问题
5. **客户端开发边界：只允许改 mock 数据**（"studio 代码改偏了 撤回"→"只允许你改 mock 数据"）：qtdata Studio 的页面/导航/模型/pubspec 代码是既定结构，未获明确指示不得改动。案例内容通过**替换 `lib/mock_data.dart` 的数据**进入客户端——Project 结构（name/client/deliverables/matrix/blueprint/phases）原样保留，只换数据内容（客户项目 → "量潮科技数字化"项目，议事决议数据映射进各字段）。动手前先确认改动边界

## 整体项目上下文（"量潮科技数字化"）

- 定位：量潮自身数字化 = qtdata 数据工程的首个完整案例，**支撑 qtdata Studio 客户端开发**（吃自己的狗粮）
- 需求点（子项目）：1 议事决议数据（当前）→ 2 开源研发数据 → 3 课程学习数据 → 4 招聘数据 → 5 交付数据（脱敏）
- 每个需求点走数据工程标准：数据需求 → 数据规格 → 数据契约 → 实现 → 交付
- Studio 数据链路：`governance.json`（本流水线产出）→ qtdata/src/studio；**当前唯一合法改动 = `lib/mock_data.dart`**（案例数据进 mock：把客户项目替换为"量潮科技数字化"，议事决议数据映射进 deliverables/matrix/blueprint/phases）；页面/导航/模型代码不动（曾因改 main.dart、新增治理页面被撤回）

### 内部定价与项目形态（用户拍板）

- **内部项目是战略投资，不是生意**（"太复杂了。这个是亏本也必须做的事情"）：量潮科技数字化（议事决议数据）的价值 = 支撑 qtdata 开发 + 量潮自身数字化 + 知识资产沉淀，**不看单项目盈亏**——不要主动挂事业部核算/损益（用户明确搁置，勿再提）
- **不要过度断言管理设计**（"为什么事业部核算可以从这里开始"）：从局部（内部定价）直接跳到更大体系（事业部核算）暴露没理解；中间隔着成本数据/分摊规则/报表形态，缺了就是断言。管理体系的联系由用户自己确认，不替他连
- **内部定价三法**（用户要求"先估计一下"时才做）：成本法（数据工程标准工作分解 × 人天成本）→ 市场法（外部行情）→ 内部价 = 市场价 × 折扣（7 折≈免获客成本）；估算进 mock 时 `contractAmount` 填有依据的数字（万元单位）+ 注释记录完整估算依据
- **进度估计进 mock**：更新 phases/deliverables/matrix 状态反映真实进度（如"单周示例上线→done，批量整理→active"），`updated` 日期同步；自动完成度 = contractAmount × done/total

## 飞书导出解析 Pitfalls

- 标题两种格式：`# 标题` 或 `<title>`（优先匹配 `<title>`，否则误取 `# 小节`）
- `<cite user-name>`=@提及人（owner 候选）；`<poll>`=投票标记；`【已废止】`前缀=status 废止
- 飞书决议无 due 字段——逾期推导需容错跳过
- 决议 id 生成必须加序号（多条提取同 id 会互相覆盖）
- 生成 YAML 用 `yaml.safe_dump` 序列化，勿手写字符串（`- ` 开头等会解析报错）

## 工作报告分类工作流（qtadmin-execute → docs/profile，2026-08-22）

同一私有仓库族（qtadmin-private）的第二个数据源：**qtadmin-execute**（工作报告数据，`data/qtadmin-execute/assets/工作报告/简报Brief/<部门>/<周>简报.md`——CEO办/COO办/数据工程部）。用户指令模式："阅读报告，根据事项分类，写到 data/profile"。

工作流：

1. 读全部简报（`grep -A30 "本周工作\|同步消息"` 逐份提取；数据工程部格式不同——`# 三、工作总结` 段落）
2. 按**对象/业务线**分类（对齐现有档案结构，不发明新分类体系）：量潮数据（按老师/客户/平台对象，如某客户=结项/复盘、另一客户=复现、平台=云服务）、量潮课堂（实训基地招聘/课堂创新）、商务行政（发票模板/邮件工作流）
3. **增量追加**到 `qtadmin-private/docs/profile/index.md`：保留原有条目，新增事项带**周次来源标注**（如"（CEO 33周）"）——便于回溯
4. 事项提取颗粒度：同步消息表（事项/负责人/状态）+ 本周工作列表——按对象聚合，不逐条照搬

注意：qtadmin-private 是私有仓库（不回灌公开契约），docs/profile/index.md 是工作档案（按对象/业务线分类的事实聚合）。

## 工作报告脱敏公开流程（qtadmin-execute → 公开 quanttide-execute，2026-08-23）

用户确认\"目前公司的平台都开源的\"——报告事项可**脱敏后提交公开仓库**。原则：

- **可公开**：产品/技术事实（上线/版本/阻塞/产品规划）——平台都开源，代码公开了上线事实也该公开
- **保留私有**：人事（候选人数/个体信息/成员分工）、客户（客户名/验收状态）、治理决策（决策权归属——脱敏为\"决策机制分层\"）
- 脱敏手法：去人名（真名→\"决策机制\"）、去客户名（客户名→\"某客户项目\"）、去人数（32 人/4 人淘汰→\"招聘全流程\"）

公开提交路径（`quanttide-execute` 领域仓库）：

```
data/profile/            # 档案（状态视图）：业务 × 职能
├── index.md             # 整体方向 + 各业务重点
├── qtdata/index.md + business.md（商务复盘）+ product.md（产研研发）
├── qtclass/index.md + operation.md（招聘运营）+ product.md（课堂创新）
└── qtcloud/index.md + product.md
data/journal/            # 日志（时间轴）：业务文件夹 × 日期
├── qtclass/2026-08-23.md
├── qtcloud/2026-08-23.md
└── qtdata/2026-08-23.md
```

- 每个业务 index.md 格式：**定位（一句话）→ 重点（3 条）→ 当前状态 → 职能档案索引**——用户\"给各个业务增加 index.md 展开\"
- 职能文件划分依据用户指令：qtdata=复盘是商务环节/研发是产研环节 → business/product；qtclass=运营/产研 → operation/product；qtcloud 全产研 → 仅 product
- journal 与 profile **同构但不同维度**：journal=事实（业务×日期，时间组织）、profile=状态（业务×职能，结构组织）——journal 是原料，profile 是提炼（日志→档案单向演进）
- 此双视图结构是**执行云产品原型**（设计思路已写入 qtcloud-execute/docs/dev-guide/design.md：双视图 + AI 提炼管道——AI 自动做 journal→profile 提炼、优先级在档案层、与思考云 4D 同构）
- 日志日期用**当天**（对话跨天时以 `date +%Y-%m-%d` 为准，用户\"今天 23\"纠正过）；不同业务文件夹同日文件独立

## 关联

- 飞书数据采集：见 `lark-feishu-cli` 技能（绑定/授权/导出）
- 契约与流程：公开 `quanttide-delib/data/context`（contract.yaml v0.0.7 七节点）
