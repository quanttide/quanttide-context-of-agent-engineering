# context → profile 整理流水线（2026-09-11 定型）

领域仓的语境（`data/context`）每天积累口述捕获，靠工作区任务把这些条目粗加工进个人档案的材料库。这是"判断降级为整理"的工程化落地。

## 结构

```
<领域仓>/
├── data/context/
│   ├── iGuo/<YYYY-MM-DD>.md            # 每日捕获（整理后留空占位，文件不删）
│   └── qtcloud-work/                   # 工作区
│       ├── tasks/*.yaml                # 任务定义（context-to-profile 等）
│       └── artifacts/
│           ├── report/<task>.md        # 任务报告（草稿区，人接收后进 journal/report）
│           └── journal/<task>.md
└── data/profile/
    └── iGuo/
        ├── workflows/<name>.yaml       # 工作流的个人侧副本
        ├── materials/<分类>/index.md    # 材料库（八分类）
        └── artifacts/
```

- 材料分类（八类）：**agent** 智能体 / **connect** 沟通 / **infra** 基础设施 / **work** 知识工作 / **org** 组织管理 / **write** 写作 / **meta** 元工程 / **course** 课程
- 跑法（定义里的原话式）：`kg --data <领域仓>/data/context/qtcloud-work task context-to-profile --next`——工作区与工作流目录随任务记着

## 五步（tasks/context-to-profile.yaml）

| 步 | 做什么 | 判据（rule=机器 / agent=智能体 / human=人） |
|---|---|---|
| **pull** | `git -C data/context pull --ff-only`，把新到的日期文件逐条读一遍，清单写进报告「条目清单」节 | rule：能拉到最新；rule：报告含 `## 条目清单` |
| **classify** | 逐条认归属到八分类，把「条目 → 分类」写进报告同节；**拿不准的单列出来** | agent：每条都有着落（进分类或进"拿不准"）；human：拿不准的由创始人裁决 |
| **coarsen** | 每条口语流水收成「**标签**：内容」的一节，写进 `materials/<分类>/index.md`；分类目录不在就新建（内含 index.md） | rule：材料库有改动（`git -C data/profile status --porcelain -- iGuo/materials`）；agent：写法平实、**没有"不是…而是…"式表达** |
| **move-out** | 已进材料的条目从语境日期文件删掉，**日期文件留空、目录保留**；没处理的条目留着 | rule：语境仓有改动 |
| **commit** | 分层提交推送——材料在**档案仓**提交、迁出在**语境仓**提交，再回工作区更新 `data/profile` 与 `data/context` 两个指针 | rule：两个指针已记录（`git status --porcelain --ignore-submodules=dirty -- data/profile data/context` 为空）；human：创始人点头 |

## 报告形状（artifacts/report/context-to-profile.md）

实际跑出来的报告（09-11 实例，24 条）含五节，照此写：

1. **条目清单**——来源日期文件 + 拉取状态 + 逐条编号摘要（一条一行，不展开）
2. **条目 → 分类**表——`# | 分类 | 材料目录 | 依据`；依据写"为什么归这类"的一句话
3. **拿不准**表——`# | 条目 | 候选分类 | 说明` + **创始人裁决记录**（写明日期与结论，裁决后一并粗加工）
4. **粗加工进材料**——按分类列条目名，标注各写进哪个 index.md、共几条
5. **迁出语境**——说明条目已全进材料、日期文件已清空留占位、`git status --porcelain` 已验证

## 已跑实例（判断依据）

- **09-11（24 条）**：work 14 / meta 6 / agent 3 / write 1；三条跨类（AI 造概念·人机分工·工作历史）交创始人裁决后归 write / work / work
- **09-12**：work 域 context 与 code 域 context 各有新到，尚未整理

## 用户只说"整理"时的判断（2026-09-12 实证）

"整理"是操作名，不是域限定词——先查两处：

1. **哪个域的 context 有新到且未整理**（各域 `data/context/iGuo/*.md` 内容非空）
2. **该域是否有配套**：有 `context/<工作区>/tasks/context-to-profile.yaml` + `profile/<作者>/materials/` 才能直接跑

实证：work 域机制齐备（任务定义 + 八分类材料库 + 09-11 已跑一轮）；code 域**缺配套**（无工作区、`profile` 按项目分目录 qtcloud-execute/qtclass/… 而非 `iGuo/materials`）——先如实报差异，再问去向。

**但\"缺配套就先出结构方案\"被用户否掉了（2026-09-12 实证）**：我为 code 域出了一整套方案（建工作区 `qtcloud-code/{tasks,artifacts}` + 建 `profile/iGuo/materials/{agent,alignment,paradigm}` + 补 context README），用户回**\"不对。只把 Code 的 context 移动到 insight 整理。其他事情都不要做。\"** 教训：① **没有配套的域，整理去向可能是 insight 而不是材料库**——不要默认把 work 域那套机制复制过去；② 用户已给出范围/去向时，整理就是**一次搬迁**，不是结构建设——\"其他事情都不要做\"是硬约束，方案里每多一项（建工作区、建分类、补 README）都是越界；③ 判断顺序：先问\"整理到哪\"，再动手，不要先造结构再等拍板。

## 去向二：context → insight（2026-09-12 code 域实证）

整理的去向由用户指定；语境内容本身是判断性/预测性时，可以直接迁入**洞察仓**而不是材料库：

- **写成洞察篇目**：按洞察格式组织（预测句 / 机制 / 边界 / 待解问题），标题用用户原话的判断句；路径 `<领域仓>/data/insight/<slug>.md`（实证 `programming-agent.md`）
- **原文照录式整理**：只做结构化与压缩，不加自己的判断、不改用户定性（连\"不是…而是…\"句式也在洞察里保留原话）
- **迁出语境**：日期文件清空为 0 字节（留占位、目录保留），commit message 写\"迁出语境：<日期> 条目移入洞察仓（<文件>），日期文件留空占位\"
- **提交链四层**：洞察仓 → 语境仓 → 领域仓（`git add data/insight data/context`）→ 根仓（`git add domains/<域>`），每层推送后核对 `git log origin/main --oneline -1`
- 不需要建 AGENTS.md、不需要补 README、不需要建工作区——除非用户另说

## 与其它纪律的接口

- **整理时不做归宿判断**：材料先进材料库，升级进对应的家（roadmap/handbook/insight/specification）是**未来某次整理或使用场景**里的独立判断
- **原文删除而非摘要替代**：日期文件只留分流索引/留空占位，spec 类内容不留副本（单一事实源）
- **材料条目升级模板**：定义 / 状态机位置 / 第一版范围 / 阶段演进表挂接总路线推导轴（实证 `roadmap/qtcloud-work/material.md`）
