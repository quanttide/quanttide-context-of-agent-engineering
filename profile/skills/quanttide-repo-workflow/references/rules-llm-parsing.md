# 规则 + LLM 的文本解析模式（2026-09-25 quanttide-founder-toolkit memory engine 实证）

## 用户的设计原则（原话）

「Markdown 不正则解析正文，而是通过规则说明 Markdown 怎么读，可以用大模型。」

即：**正则只管语法结构，语义理解交给「规则说明 + LLM」**。

## 三层管线分工

```
文本文件
  ↓  第一层：语法解析（正则，只管结构——标题/段落/列表/分隔线）
  块序列
  ↓  第二层：结构切分（按标题层级切节，纯结构操作）
  Section 树
  ↓  第三层：语义提取（规则说明 + LLM，按规则填表）
  语义模型（ProfileDoc / InsightDoc / RoadmapDoc / Novel / Chapter）
```

- 第一、二层用正则没问题（是 Markdown 语法，不是语义）
- 第三层**不用正则做语义匹配**（如 `**名称**：详情` 的 pattern 匹配、标题里找关键词分类）——换个写法就崩

## Dart 模块划分（管线层通用化 + 域引擎分离）

```
lib/src/
├── markdown_parser.dart      第一层：语法解析（通用）
├── section_splitter.dart     第二层：结构切分（通用）
├── semantic_extractor.dart   第三层：规则 + LLM（通用）
├── memory_engine.dart        memory 域模型 + 仓储装载
└── fiction_engine.dart       fiction 域模型 + 仓储装载
```

加第三种记忆结构 = 加一个 `xxx_engine.dart` + 几个规则文件，管线层不动。

## memory 与 fiction 同构（三层管线通用的根本原因）

两者都是「用目录命名约定组织的异构 Markdown 文档集合」：

| 共同特点 | memory | fiction |
|---|---|---|
| 结构靠目录命名约定 | `journal/ profile/ insight/ roadmap/` | `{N}_{阶段}/` 前缀 |
| 文件命名承载元数据 | `YYYY-MM-DD.md` 文件名即日期 | `{序号}_{标题}.md` 文件名即编号 |
| 原始→精炼的分层 | journal（原始）→ insight（提炼） | 观察站（原始）→ 灵感→场景→初稿→定稿 |
| 发现逻辑：找特征目录 | 含任一层子目录即为记忆集 | 含 `index.md` 或 `N_` 阶段子目录即为小说 |
| 容忍结构缺失 | 无假说节/无元目标/无一级标题均可解析 | 未编号章节/预留空号/无 index.md 均可处理 |

## fiction 域关键设计

- **Stage 动态发现**：按 `{N}_{名称}` 模式识别，不硬编码阶段名（各小说阶段名不同）
- **Chapter 编号语义**：`0_` 前缀=前言不占编号、未编号=替代草稿、预留空号宁空勿移、同一序号可有多份
- **Observation**：情绪日记 + 社会观察（观察站）
- example `parse_fiction.dart`：输出解析报告含预留空号统计

## 规则文件（YAML）——描述「怎么读」

放 `assets/rules/<type>.yaml`（数据不进 lib/doc）。

```yaml
document_type: profile
title:
  source: "第一个 H1 标题"
  fallback: "文件名去掉 .md"
sections:
  split_by: H2
  subsections: H3
  extract:
    - field: items
      from: "节内列表条目"
      patterns:
        - "**名称**：详情"
        - "名称：详情"
        - "纯文本"
  unknown_content: "保留在 paragraphs 中，不丢弃"
```

## 提取器（可插拔）

- `ParseRule`：加载 YAML，生成自然语言指令供 LLM 理解
- `RuleBasedExtractor`：无 LLM 时的降级方案（纯文本匹配）
- `LlmExtractor`：按规则填表，不自由发挥
- `LlmClient` 抽象接口：调用方式由使用方实现

## 多包仓库的文档分层（2026-09-25 用户两次纠正）

- toolkit 根 `docs/index.md` = **通用逻辑**（换个语言实现还成立的内容）
- 各 package 的 `doc/index.md` = **特定逻辑**（该包的目录结构、类职责表、开发命令）
- 两处互相引用不重复。判断标准：这段内容换个语言实现还成立吗？

## 适用场景

任何「文档有约定俗成的写法，需要提取语义结构」的场景——不只是 Markdown，任何半结构化文本都可以用这个模式替代硬编码正则。
