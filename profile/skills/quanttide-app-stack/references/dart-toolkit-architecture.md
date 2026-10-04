# Dart 工具包架构模式（quanttide-founder-toolkit 实证）

2026-09-27 设计并落地。经多轮用户纠正后定型。

## 两层资产 + 三个引擎动词

```
assets/
├── artifacts/       名词定义（YAML）——「有什么」
│   ├── journal.yaml     字段、来源、提取方式
│   └── ...
└── workflows/       动词定义（YAML）——「做什么」
    ├── classify.yaml    用哪些步骤 + 判断规则
    └── ...
```

引擎契约：workflow 的 steps 只能用三个动词——`scan`（关键词定位，代码）、`judge`（LLM 判断）、`merge`（策略合并，代码）。

**扩展线**：加名词 = 加 artifact YAML；加任务 = 加 workflow YAML；加动词 = 改引擎。三条线互不干扰。

## Bloc 架构分层（Dart/Flutter）

```
lib/src/
├── core/            跨域共享（域依赖 core，core 不依赖域）
│   ├── parse.dart       语法解析 + 结构切分
│   ├── rules.dart       Artifact / Workflow 加载
│   ├── engine.dart      scan / judge / merge
│   └── llm.dart         LlmClient
├── {域}/            按业务域组织（feature-first）
│   ├── models.dart      纯数据，不读文件
│   ├── repository.dart  发现 + 装载（I/O）
│   └── bloc.dart        业务逻辑编排（事件进 → 状态出）
```

**判断标准**：
- 一个文件一类角色，一个目录才有意义时才建目录
- models 不做 I/O，repository 不做语义，bloc 不管 I/O
- 跨域共享叫 `core/` 不叫 `app/`（app 是 Flutter 应用壳的概念，纯工具库没有应用壳）

## 坑

1. **别按分析词汇表建类**——分析里的「六类计算」≠ 代码里的六个类。先分清哪些是同一机制（LLM 判断）、哪些是确定性函数（代码），再定类。
2. **切分轴要一致**——parse/rules/engine 按管线阶段切，memory/fiction 按域切，别混。
3. **import 路径**：`lib/src/{域}/` 引 `lib/src/core/` 用 `../core/`，不是 `../../core/`。
4. **私有成员跨文件访问**：Dart 的 `_` 是库私有不是类私有——同库可访问，跨文件同库也行；但跨库不行，需要时改公开或加 getter。
