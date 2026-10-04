# 库的 Bloc 式分层（2026-09-29 toolkit 实证，用户四轮否决后定型）

给**纯 Dart 库**（非 Flutter 应用）分层的判据。四轮否决的顺序本身就是教学：

| 轮 | AI 方案 | 用户否决 |
|----|---------|----------|
| 1 | 按「六类计算」拆六个类 + `compute_engine.dart` 装一起 | 「模块划分很奇怪 反思并重新设计」 |
| 2 | 收成 5 目录（parse/rules/models/repos/compute），逐个模型一文件 | 「这什么玩意」 |
| 3 | 收成 2 目录（models/ repos/）+ 3 平 файл | 「也还是不对。**遵循 bloc 架构**重新设计」 |
| 4 | Bloc 式，但跨域层叫 `app/` | 「**app 文件夹为什么会需要**」→ 改 `core/` 认可 |

## 三条根因

1. **把分析用的词汇表当成了代码结构**。「六类计算」（分类/评分/聚类/合并/升降级/候选定位）是分析产物，不是代码单元——它们只是**三个真动词**的组合：
   - `scan`（关键词定位，纯代码）
   - `judge`（LLM 按规则判断——分类/评分/聚类/升降级都是它）
   - `merge`（策略合并，纯代码）
   分析里的六个词是**用例**，落在 workflow YAML 里；代码里只有三个动词。**每发明一个概念就给它建一个类 = 把描述物化成抽象。**
2. **小包摆架子**：十来个文件拆五个目录、几十行的数据类一文件一个。判据是**一个目录一个角色，一个文件一类角色**——`models/` 和 `repos/` 值得建目录（是被依赖的两层），parse/rules/compute 各是一个角色，单文件就够。
3. **借错目录名**：`app/` 是从 Flutter 应用仓库借来的（那边有 MaterialApp/路由/引导=应用壳）。纯工具库没有应用壳，跨域共享层在 Bloc 约定里叫 **`core/`**。

## 终态结构

```
lib/src/
├── core/            跨域共享：域依赖 core，core 不依赖域
│   ├── parse.dart       文本 → 结构（语法解析 + 结构切分）
│   ├── rules.dart       Artifact / Workflow 的 YAML 加载
│   ├── engine.dart      scan / judge / merge
│   └── llm.dart         LlmClient 接口
├── memory/          域：models.dart + repository.dart + bloc.dart
└── fiction/         域：models.dart + repository.dart + bloc.dart

assets/
├── artifacts/*.yaml     名词：有什么（字段、从哪提取）
└── workflows/*.yaml     动词：做什么（用哪几个 step、判断规则）
```

域内三文件的分工（Bloc 的 data / logic 分离落到库上）：

| 文件 | 装什么 | 不做什么 |
|------|--------|----------|
| `models.dart` | 域模型，**纯数据** | 不读文件 |
| `repository.dart` | 发现结构、装载 | 不做语义 |
| `bloc.dart` | 工作流编排（事件进 → 跑 scan/judge/merge → 结果出） | 不管 I/O |

**空的 bloc 是信号**：迁移只搬结构没搬逻辑时，bloc 里只剩一个转发方法。真正的 bloc 装着域特有的编排——那不是通用引擎该知道的事（memory 的三问路由/证据分级/合并策略、fiction 的编号轴/阶段流转）。

## 三条扩展线互不干扰

- 加**名词** = 加一个 `assets/artifacts/*.yaml`
- 加**任务** = 加一个 `assets/workflows/*.yaml`
- 加**动词** = 才改 `core/engine.dart`

引擎的执行词汇（scan/judge/merge）是**引擎契约**——只写在 docs 里，不进 assets、不建 step 定义文件。用户原话：「steps 不是太有必要」。

## 定稿前先让用户看方案

四轮里我都是直接开写，被否三次。**正确顺序：先给分层方案（目录树 + 每层职责表 + 依赖方向），用户认可后再动代码。** 用户说「core 比较合适」才是执行令。
