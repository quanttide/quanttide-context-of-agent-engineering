# QuantTide 仓库地图（2026-08-29 更新）

## 根仓库 `repos/quanttide/`
- 定位：量潮第二大脑根仓库，git 子模块架构
- 关键文档：README.md、CONTRIBUTING.md、AGENTS.md（含 CLI 工具索引：qtcloud-devops release、qtcloud-knowl extract）
- 子模块三级路径约定：
  - `domains/{name}` — 领域知识仓库，核心事实源（quanttide-finance、quanttide-course、quanttide-delib、quanttide-org 等）
  - `default/{name}` — 默认实现/配置/基础设施（quanttide-tech、quanttide-founder）
  - `assets/{name}` — 资产仓库，其中 quanttide-journal / quanttide-intention 是**聚合容器**（内部按业务实体挂子模块）

## assets/ 聚合容器结构（2026-08）
```
assets/quanttide-intention/            # 意图聚合（子仓库在容器内部 assets/ 下）
├── assets/default/company → quanttide-intention-of-business-entity（公司意图 v0.2.6）
├── assets/data/           → quanttide-intention-of-data-engineering
├── assets/pay/            → quanttide-intention-of-payment-engineering
├── assets/human/          → quanttide-intention-of-human-resources
└── README.md / docs/（MyST 导航）/ AGENTS.md / CHANGELOG.md

assets/quanttide-journal/              # 日志聚合
└── default/company        → quanttide-journal-of-business-entity
```
模式：容器只放导航文档 + .gitmodules；子仓库独立维护；公司实体 = `default/company`，其他按领域名（data/pay/human）。

## `default/quanttide-tech`（量潮科技第二大脑）
- 方法论：数据驱动的产品研发闭环（工作档案 → 数据建模 → 半结构化契约 → 各端独立开发 → 新数据回流档案）
- 关键子模块：
  - `data/journal` → quanttide-journal-of-business-entity：工作日志，按业务实体分类目录（default/qtadmin/qtclass/qtcloud/qtconsult/qtdata/qtrecurit），文件 `YYYY-MM-DD.md`
  - `data/intention` → quanttide-intention-of-business-entity：战略意图文档站（myst.yml 驱动，v0.2.6）
  - `data/roadmap` → quanttide-roadmap-of-business-entity：工作蓝图。**intro/（公司级层）2026-08-29 停用**——跨领域主题全景迁归属领域意图仓库（实训基地 → org intention/training-base.md）；领域侧（qtclass/qtrecurit/.../training-base.md）保留，与全景同名同构
  - `data/profile/brand/founder.md` — 品牌档案：量潮创始人（qtfounder）→ 拟更名量潮控股（qthold）的升级路线，组织档案的权威信息源
  - `docs/bylaw|specification|handbook|tutorial|essay|gallery` 等文档子模块
  - `apps/qtadmin|qtdata|qtclass|qtweb|qtcloud|qtconsult|qtrecurit` 应用子模块
- `.quanttide/` 有契约文件（asset/contract.yaml、docs/contract.yaml）
- qtdata 差距分析三角：`data/journal/qtdata/`（业务日记）vs `data/intention/qtdata/`（战略意图）vs `apps/qtdata/src/cli/`（当前实现）

## `default/quanttide-founder`（创始人第二大脑）
- 定位：基于认知科学记忆分类框架的组织知识库
- `assets/memory/journal/YYYY-MM-DD.md` — 个人日记（注意与 quanttide-tech 的 data/journal 工作日志是两套东西）
- `.agents/skills/`：devops-commit、devops-release、devops-submodule、devops-review

## `domains/quanttide-delib`（议事领域，2026-08 边界重构后）
- **只承载议事规则**：specification/contract.yaml（v0.0.7）、workflow.md（七节点）、intention/delib-cloud.md（AI 三角色）、intention/qtcloud-delib.md（产品意图）、roadmap/（11 组件）、gallery/、laboratory/（决议子系统实验）
- **企业治理实际情况（章程/设计意图/治理手册）已迁出** → `domains/quanttide-org/data/context`

## `domains/quanttide-org`（组织治理领域，2026-08 初始化；2026-08-29 data 层补齐）
- `data/context`：index.md、intention/design-intention.md（君主立宪制）、bylaw/（bylaw.md 章程 + audit.md）、handbook/playbook.md（周会/周报/目标管理）
- `data/intention` → quanttide-intention-of-organization-management：index.md（组织云建设意图）+ **training-base.md（实训基地公司级全景，2026-08-29 自 tech roadmap/intro 迁入）**
- `data/profile` → quanttide-profile-of-organization-management（2026-08-29 新建挂载）：**组织档案——「组织 + 人物」两组结构（终稿，全部落成）**：
  - `orgs/` 下六个组织目录：`qttech/` 量潮科技（现行经营主体）｜`qthold/` 量潮控股（拟设）｜`qtalliance/` 量潮创新联盟｜`qtfounder/` 量潮创始人｜`qtinstitute/` 量潮研究院（拟设）｜`qtacademy/` 量潮实训基地
  - `people/<person-slug>/index.md`（**一个文件夹一个人物**——用户中途纠正"people/<person-slug>/index.md 这样"，从平铺文件改为目录式，与组织侧同构）：姓名 + 职务表（组织×头衔，跨组织身份一处维护）+ 角色定位 + 关联。**人物是一等实体**；九人齐：<person-slug> ×9 + people/index.md 总索引
  - **人物档可含学术履历**（2026-08-29 先例 <person-slug>）：某高校副教授，研究领域与项目、代表论文如实落档——档案结构 = 学术背景（职位+领域）/ 学术成果（条目式）两节；其他人物有可公开履历时按此模板补
  - 配套文档同轮完成：README.md（两组结构说明+组织名单+约定）、AGENTS.md（重写为两组规范：目录结构/人物一等实体约定/信息披露规则保留）、CHANGELOG.md（补齐全部历史，原只有 init 一行）、根 index.md 档案节改双指针（orgs/+people/）
  - **职务单一事实源仍在各组织 title.md**（people/index.md 明示"引用不复制，职务变动两处同步"）——与 title.md 吸收说法不一致，以此为准
  - **与站点解耦**：profile 重构与 qtorg site 无关（用户"暂时不动网站"），站点 people.ts 自主维护
- `data/journal`：quanttide-journal-of-organization-management
- 定位 = "谁有权、怎么管"；delib = "议题怎么走流程"

## `assets/quanttide-bylaw`（章程聚合容器，组织结构权威源）
- 聚合各法人主体章程（default/company → quanttide-bylaw-of-business-entity）与各领域章程（domains/asset|data|delib|devops|...）
- `default/company/organization/`：position/secretary.md（秘书三层制度：部门秘书→秘书处→秘书长）、position/company-representative.md（公司代表）、department/deliberation-institution.md（创始人-上议院-下议院三权并列：上议院=股东代表大会/所有权/合伙人委员会立宪权司法权，下议院=公司代表大会/经营权/书记处+执行委员会+技术委员会；书记处≠秘书处——书记处属下议院内设）、rank.md（五序列职级 G/P/F/T/M）
- `default/company/qtdata/support.md`：课堂 BU、数据 BU、秘书长（中台）——事业部的现行实证
- 结构档案（org/profile/*/structure.md）的组织结构信息源：qttech 秘书处/事业部/共享中心/两院制逐条对应以上章程与 qtdata intention 的事业部制
- 子模块 default/company 常未初始化（`-` 前缀），先 `git submodule update --init default/company`

## data/profile 口径分型（2026-08-29，动手前先确认目标属于哪型）
| 类型 | 仓库 | 结构 | 信息源 |
|------|------|------|--------|
| 产品档案 | quanttide-product/data/profile | 一个文件夹一个产品：index.md（产品级思考五维度）/ requirement.md（用户故事地图）/ evaluation.md（评估档案） | journal / roadmap / intention |
| 组织档案 | quanttide-org/data/profile | **两组结构（终稿）**：`orgs/<org>/`（index.md 定位/目标/形态/关联 + 可选 structure/institution/culture；qtacademy 的 structure/institution 为目录）+ `people/<slug>/index.md`（一人一档，目录式）；README = 结构说明与名单 | 品牌/意图/roadmap/**章程**（assets/quanttide-bylaw/default/company） |
| 交互设计档案 | quanttide-design/data/profile | 一个文件夹一个产品，仅 index.md，**记理想不记现状** | 设计 context 分析 |

## data/intention 文档站结构（v0.2.6，myst.yml toc）
- 转型叙事 intro/：from-project-to-product.md（项目→产品→生态）、from-heart-to-designer.md（心脏→设计者）
- 量潮科技管理 qtadmin/：index.md（治理思想平台化，2026-08 重写）、asset.md（第二大脑资产管理）、strategy.md（"发现希望"）、delib.md（两院制设计意图）
- 量潮云 qtcloud/：qtcloud-meta.md（元工程：意图=AI 生成+人类整理）、qtcloud-asset.md、qtcloud-execute.md（迁移方法论/成熟度）、qtcloud-support.md、qtcloud-innovate.md（创新管理）、qtcloud-write.md（写作云分层元数据）
- 量潮数据 qtdata/：product.md（观测视角/事业部制）、customer.md（客户资产，最高优先级）、connect.md、dataops.md（数据工程标准）、growth.md（发券增长）
- 量潮课堂 qtclass/：index.md（双云架构/三内容形态）、self-learning-platform.md（自学平台）、curriculum.md（课程体系；**原 strandard.md 更名，注意拼写**）
- 量潮招聘 qtrecurit/：index.md（序列 T0/T1/T2 + 轨道 A/B）
- 发布历史：v0.2.2 → v0.2.6（2026-07-29 ~ 08-01），全部 patch 版本

## 关键认知（2026-08）
- qtadmin 新定位 = 企业治理思想和制度的平台化，封装独特思考方式，秘书处管理视角，生产工具全部剥离给量潮云
- 内部资产三层建模：assets（平台原始数据）/ data（人工分析）/ docs（总结知识），每人一个 workspace；公开第二大脑 = 私有第二大脑的元认知
- 优惠券/代金券 = 未来代币前身，用交付和知识资产作底牌发债
- delib 议事框架：七节点固定骨架（创建→动议→附议→辩论→投票→决议→归档）+ 研讨/提案两类模式 + 议事规则/议程/决议独立实体；决议独立管理 = 统一治理（治理视图/播报/周会汇总），解决用户"看不到决议"的最大管理困难
- **档案可先于应用存在**（2026-08-29 qtacademy 实证）：profile 约定目录名与 apps/ 子模块一致，但路线图阶段的产品可先建档案（预留架构声明），汇报时说明边界即可
- **evaluation.md 是 profile 标准档案类型**（2026-08-29 定型）：按业务目标分解评估维度 → 评估问题+验证方法 → 检查清单+结论判定；跨仓库同步 = 纯复制（源保留，非迁移）
- **structure.md 是组织档案标准文件**（2026-08-29 qttech/qtalliance/qtacademy 三实例）：组织 index.md 的"组织形态"只留一句话 + 指针，架构细节全在 structure.md；用户给机构名骨架 → 从章程找原文充实 → 无原文的按治理惯例组织框架并标注归纳/占位
- **institution.md / culture.md 是组织档案标准文件**（2026-08-29 三组织同批）：institution 记制度体系与核心约定（章程原文逐条对应 bylaw 出处；章程未立项的写"已定框架+待立制度清单"，不编造制度）；culture 记文化气质（子组织显式写与母文化的传承变体关系）。qtacademy 组织形态终稿：内包子公司 + 传统职能部门编组（技术/产品/市场/职能），跨部门项目组承接任务、完队回部门
- **顾问决策讨论的格式铁律（2026-08-29 qtacademy 编组讨论实证）**：结论前置（第一句即结论），理由压缩；用户打断"结论是什么"= 格式否决，砍到结论+一句话理由；用户对结论说"no"后**停止辩护原方案，等用户自己的模型输入**——用户连给约束（"必须编进部门"）和类比（"参考内包子公司"）逐步收敛方案，每个输入都是权威修正，方案演进链记入档案防回退
- **缺席 ≠ 不存在（2026-08-29 文化一致性讨论实证）**：分析组织/文化/机制状态时，**资料没写 ≠ 现实没有**——制度层（institution/bylaw）与实践层（roadmap/journal）已有的东西（下议院周会、显性表决、自组队竞争）不因 culture.md 未记录而不存在。曾从"culture.md 无民主条目"推断"公司民主文化薄、基地民主被讲丢"，被纠正"不对。这些只是没有体现在现有资料里"。结论：文化文档落后于现实 ≠ 现实缺机制；补记录时只写实然（制度与实践已有的），不先于制度编造（qtalliance 留白待章程）；用户拍板的民主实践增量（基地考核评审权让渡给活跃成员）= 实然新增，institution 记框架+待立细则、culture 记体感条目，直接落档不再论证
- **qtacademy 档案结构已演进为目录**（2026-08-29 用户侧重构）：institution/ 是目录（index.md + rank.md 职级表 L-3→L-2→L-1→L0→L1+ + department.md），structure/ 也是目录（index.md + department.md）——patch 前先 ls 看实际结构，勿按单文件路径盲写。**命名一致性陷阱**：概览 index.md 曾写"职能部"而详档 department.md 写"综合部"（同轮设计迭代命名变了概览没跟上）——已统一为"综合部"（详档为准）；改部门/机构命名时必须同会话 grep 全档同步，不能只改一处。注意 profile/AGENTS.md 的信息披露规则（不公开=具体人员岗位）：人物档（people/）目前只写职务不写岗位细节，扩写履历前先核对该规则
- **profile 大规模 git mv 重组**：组织目录迁入 orgs/ 用 `git mv <dir> orgs/<dir>` 批量执行——git 识别 100% rename（status 显示 R），历史无损；迁移后 grep 全库 `profile/qt` 引用路径清理（站点 docs、AGENTS.md、其他仓库引用），引用清理要等用户对"站点引用"拍板（2026-08-29 用户选择全部删除而非更新路径）
- **重构任务中途被新指令改向时，先盘点当前状态再继续**（2026-08-29 profile 重构实证）：用户在 people/ 平铺文件建到一半时插入"people/<person-slug>/index.md 这样"（目录式）——先 git mv 已建文件进目录（`mkdir -p <slug> && mv <slug>.md <slug>/index.md`），补齐剩余人物（<slug>.md→<slug>/index.md），再建 people/index.md 总索引，最后一次性收尾（README/AGENTS/CHANGELOG 改写+提交）——不要丢弃已完成的工作重头再来
- **"propr"类乱码/极短输入 = "继续"信号**：用户发出残缺词（"propr"="propel/继续"）时，检查 git status 与上下文找进行中的任务，直接续做收尾，不要反问语义
- **会话外重构检测（2026-08-31 quanttide-learn 实证）**：跨会话回到某仓库时，即使昨天刚整理过，也要先 `git log --oneline -3` + `ls` 核对实际结构——用户/其他 agent 可能在会话外大改（learn 的 profile 已从 qtclass/qttech 分组重构为 learners/schedules/tasks 三目录、journal 平铺化）。SKILL.md 里记的"上次结构"只保证到上次推送为止，不是当前事实；发现结构变了先改 repo-map 再继续任务，不要按记忆里的旧路径操作

## `domains/quanttide-learn`（学习管理领域，2026-08-30/31 完整链条建立；**08-31 已被用户会话外大规模重构，勿按 08-30 路径找**）

- `data/journal` → quanttide-journal-of-learning-management：**顶层平铺 `{学习者ID}/YYYY-MM-DD.md`**（Jerry/、iGuo/——qtclass/qttech 分组已废）；内容源 = 飞书教学群聊（lark-cli im 导出），**脱敏纪律严格**：真名→代号（文件夹名即代号）、来源渠道痕迹（平台/群名/时间段/链接/邮箱）一律不入档也不入 README 映射——首篇日志当天两轮 git-filter-repo 的教训；日志含「自举」节 = 探路者把自己当 follower 记账（学习 Task + 验收标准，验收形态 = 产出物 + 他人认可）
- `data/profile` → quanttide-profile-of-learning-management：**08-31 重构为三目录**——`learners/{昵称}.md`（Jerry/iGuo）+ `schedules/<名>.md`（agent-engineer 智能体工程师训练营 / product-manager 产品经理训练营；YAML frontmatter title/description；Task 条目带"来源：tasks/<slug>.md"）+ `tasks/<slug>.md`（YAML frontmatter + 任务全文 + 达成条件，含 Issue/PR 通道说明与"不会操作可安排协助"条款）。双变体仍在 AGENTS.md（学员验收制/自助主题制），但选择依据已改"学习是否由 **Schedule** 驱动"（不再提"课程验收标准"——Criterion 概念已从本域移除）
- `data/intention/index.md` → 探路者/学习云/follower 三层结构 + **学习云定位修正（用户纠正 AI 的过度推演后拍板）**：学习云不教做事、不管理任务本身（做事归执行云/业务域），只记录做事产生的学习效果 = **能力账本**；与执行云分工 = 同一个"做"两本账（执行云任务账/事的角度——"PR 章程仓库"做没做完；学习云能力账/人的角度——会提 PR、熟悉章程仓库），同源不同记互不替代
- `data/insight/schedule-core-model.md` → **Schedule = 唯一领域模型**（Schedule + Task 两概念，Task 自足零外键零跨域引用——"Task 就是 Task"，验收判定写在自身描述、跨域对齐归应用层；Criterion/Stage/Step/Milestone 三轮砍除）；Journal/Profile 是数据视图非领域模型
- `docs/specification` → **v0.1.0 已发布**（与 insight 平行收敛、结论一致）：两层——`learner/`（Learner × Completion，**学习云 provider 已落码**，Completion.task_id → Task）+ `schedule/`（Schedule + Task 契约，设计完成未落码）；首个真实实例 `schedule-agent-engineer`；skill 建模教训 = 不要把数据视图当实体
- `docs/bylaw` → 章程三轮返工后的终稿 = **运转规则版**（六章十八条白话：路径协商/先做事后记录/判定只看证据/批量提交核验/异议不写理由书/修正 git 留痕/提炼本人确认/脱敏入档/记录不挪用/走完的承认/档案跟人走）；总纲一条 = "每条记录都要有证据，每个判定都要能复核"。**三轮返工的迭代序沉淀在 SKILL.md 主文件「章程/制度类文档写作」节**（范围先行→理解先行→去概念化→总纲一句话）

## 组织模式总纲（profile/index.md，2026-08-29 内核重写定稿）

用户拍板过程：AI 先给"六条特色罗列"→ 用户"内核的理解还是不够透彻" → AI 追到"把依赖'在场'的价值固化为独立于人的记录与结构" → 用户"yes。也就是'民主'和'法治'" → 定稿。**总纲结构**：

- **内核一句话**：把依赖"在场"的价值，固化为独立于人的记录与结构——四个替换：知识（师傅→提炼链）、信用（口碑→记账/可见性）、权力（拍板→两院程序）、执行（指派人→指派职能槽位如"秘书处值班负责人"）
- **两根支柱**：法治 = 记录与规则是权威来源（"从人治向法治转变"）；民主 = 合法性来自参与（两院/代表选举/显性表决/轮值书记/自组队竞争）。互为地基：记录托起参与（选举权按职级=记录的函数，L1+ 有选举权、实训生无）、参与赋权规则（立宪程序）。创始人一票否决 = 受程序约束的"在场"兜底残余（须通报理由、记录在案、不得委托）
- **一台机器三个剖面**：公司=知识提炼（生产）/ 基地=贡献记账（训练）/ 联盟=介绍即验证（生态，把组织间"关系信任"也外化）
- **文化与产品同构**：第二大脑方法论的自我验证——文化是产品的第一个证据；源头是"从心到设计师"（外化对象在换，动作不变）
- **制度风格节（2026-08-29 补）**：记录介质驱动生产/评价/治理三循环飞轮——不设回避（评价是连续记录时隔离收益消失，"自己评自己"从漏洞变课程）/ 制度即课程（第一问是"能长出什么人"非"怎么防坏事"）/ 诚实的未完成（已定框架+待立清单分栏，档案先于现实）/ 槽位先于人头；底层姿态 = **信任加光照**（默认人可信，靠记录可见性约束）+ 熔断位（否决权/派驻校准）；**记录操纵口径（用户拍板）**：任何组织都存在的老问题，量潮已非常擅长处理（连续记录长期比对+公开可见+审计兜底）——档案写能力陈述，不写风险提醒
- **检验判据**：任何新实践只问"是否依赖某个具体的人在'场'？"——依赖记忆/交情/老板记得=违反；写成记录/进系统/可验证=符合

三层分工：根 index.md=模式总纲 / README.md=组织名单 / 各组织档案=事实记录（四件套）。

### 内核提炼的方法论教训（勿重蹈）

- 六条平列特色 ≠ 内核：用户"内核的理解还是不够透彻"。要找所有条目共同回应的根因（组织失败=关键依赖嵌在"人在场"里），一句话说尽，条目做投影
- 用户用已有概念收编提炼（"也就是民主和法治"）→ 立即挂到用户的词汇上重述定稿，用户的词汇是权威封装，不坚持自己的措辞
- 提炼跨层取材：组织文档 + 方法论（第二大脑）+ 创作观（从心到设计师）+ 产品哲学——自我验证关系是穿透点
