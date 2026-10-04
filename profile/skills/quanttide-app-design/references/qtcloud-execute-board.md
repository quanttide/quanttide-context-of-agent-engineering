# qtcloud-execute 执行云设计决策链（2026-08-23）

## 背景

从工作报告（qtadmin-execute 简报）→ 档案体系（profile 业务×职能）→ 执行云设计。用户导航式纠正的完整记录。

## 档案结构决策

- profile 按 业务（qtdata/qtclass/qtcloud）× 职能（business/operation/product）组织——**与飞书任务/滴答清单的"清单→分组"结构同构**
- journal 按 业务文件夹 × 日期文件（YYYY-MM-DD.md）——事实流
- 关系：journal 是原料，profile 是提炼（AI 提炼管道 = 执行云核心价值）
- 执行云 = 双视图（日志流 + 档案视图），与思考云同构（感知→理解→判断→行动）
- 整体方向：把人的能力系统性转译成系统的能力（治理法治化/人才系统化/产品工厂化/事实源制度化/经验资产化）

## 事件风暴收敛（用户纠正序列）

1. 初版 4 聚合（JournalEntry/Business/ProfileItem/Distillation）→ 用户："聚合应该只需要 Task 和 TaskList 足够了"
2. 字段审核链：
   - blockReason → "BlockReason 是什么？" → 砍（阻塞 = status 的一种）
   - 补 title（"缺少 title"——标题与描述分开）
   - source → "source 是做什么的？" → 砍（无实际操作价值）
   - content → "content 和 description 哪个更符合设计意图？" → **description**（任务领域惯例 title+description）
   - priority → "紧急、高、中、低四个"
   - status → "未开始、进行中、评审中、已完成四个状态"
3. TaskList：清单=业务实体（一个业务一个清单），分组=Group（职能）——"分组为什么是 Section"→ Group（命名跟随心智，非受现有代码影响）

## 页面设计决策

- **清单切换器数据驱动**：三个业务 Tab 被否（"不应该对清单做静态假设，而是应该有切换器"）——清单列表从仓储加载，支持新增
- **详情用弹窗组件**："task-detail 做个组件就可以，然后清单里弹窗点开"——views/task_detail_dialog.md，不配独立页面/路由
- **二维看板**（用户确认）：列=分组（结构横向，与档案同构）、行=状态（推进纵向，"未开始"行=执行入口）——"结构横着摆，时间竖着走"；行内按优先级排序；"已完成"行默认折叠
- 卡片不重复显示状态（行已表达）——单一信息源

## ⚠️ 最终转折（2026-08-23 下午——推翻上面）

**BoardView 实现后用户指出："只是一个表格，不是真正的看板。反思原因"**

- 我的首次反思（"列=状态才对"）被否："反思方向不对"——问题不在布局选择，在**我没把 BoardView 当看板设计**：设计起点是数据结构投影（BoardProjection 矩阵），数据形状（Map<Group, Map<Status, List>>）定义了产品形态（Table）；doc/views/board_view.md 通篇表格语言（矩阵/单元格/交叉定位），pi 按文档实现表格是必然
- 真正理解用户行列结构：列=分组泳道、行=**列内状态分段**（卡片在列内上下流动=状态推进）——我实现成固定网格（交叉定位）丢了"列内流动"
- 用户自查："似乎需要把状态调整到列更合适"→ 列=状态泳道（Kanban 本质）；分组放列内"意义有限还干扰"→ 分组降为筛选
- **最终模型**：取消 TaskGroup（Group 枚举/TaskList.groups 全删），降级为 `Task.category`（String，业务自定义分类）；Task 定稿 = {id/title/description/status(未开始/进行中/评审中/已完成)/priority(紧急/高/中/低)/category}；BoardProjection 改为 状态列→任务流
- 教训沉淀见 SKILL.md 设计原则 8/9（看板=状态泳道；先交互范式后数据结构）

## 状态层（states/）

- TaskListCubit（清单：loadLists 动态/switchList）+ BoardCubit（看板：loadTasks/updateTask 重投影）
- 弹窗纯组件（不持 Cubit，回调 BoardCubit）——单向数据流
- 目录名 states/（概念）非 blocs/（绑定 flutter_bloc）

## 文档体系最终形态

```
src/studio/doc/
├── models/task.md + task_list.md
├── screens/task_list_screen.md
├── views/task_card.md + task_detail_dialog.md + task_list_switcher.md + board_view.md
└── states/index.md
```

每个组件文档：定位/结构（ASCII 布局）/展示规则/交互/数据/测试/验收。
