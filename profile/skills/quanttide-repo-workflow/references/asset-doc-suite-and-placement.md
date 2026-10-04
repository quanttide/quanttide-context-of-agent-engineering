# 内容归属与资产文档套件

用户在 2026-10-01 定案的两条约定，落档在 `quanttide-algorithm/data/profile/quanttide-asset/asset-classifier/`。判断「这段内容该放哪」和「一个资产要配哪些文档」时先读这份。

## 一、内容归属：两层判据

定义取自用户自己的话（`domains/quanttide-asset/data/profile/entity/founder/index.md`），不是新造的：

- **陈述性记忆**：可描述、可回忆的事实性知识。记的是「是什么、发生过什么」。
- **程序性记忆**：可执行、可操作的方法指南。记的是「怎么做」。
- **说不清**：两条都不像，或者还没成形。**不硬判**。

判别句：**这段话换个时间还成立吗**——事实会随时间变，方法不会。

| 性质 | 确定度 | 位置 |
|---|---|---|
| 陈述性 | 明确 | `data/` |
| 程序性 | 明确 | `docs/` |
| 说不清 | 低 | `context/` |

陈述性归 `data/`（11 类）：`journal` 日志、`insight` 洞察、`intention` 意图、`roadmap` 路线图、`context` 语境、`profile` 档案、`report` 报告、`brochure` 宣传册、`library` 参考、`history` 历史、`archive` 归档。

程序性归 `docs/`（6 类）：`bylaw` 章程、`specification` 标准、`handbook` 工作手册、`tutorial` 教程、`essay` 札记、`gallery` 案例集。

（两个名单已对过 `quanttide-tech` 实际挂载的子模块，正好 11 和 6。）

**`context` 是入口，不是终点。** 判不清的先放这里，等它自己长成事实或方法再归位——所以判错不致命，代价被限制在「搬一次」。这条是整个逻辑的安全阀，别省。

## 二、资产文档套件：六件

一个资产（产品/算法/工具）在 `profile/` 下按 `<产品>/<项>/` 建目录（对齐 `qtcloud-asset/{cli,provider,studio}` 的写法），套件是：

| 文件 | 回答什么 |
|---|---|
| `index.md` | 这是干什么的 + 文档导航。**只做入口**，一句话结论放开头 |
| `requirement.md` | 要解决什么问题、要满足什么、不满足会怎样 |
| `specification.md` | 判据与规则本身（是非标准、映射表、名单） |
| `implementation.md` | 用什么办法落地；有多条路径就分「规则版 / 算法版」 |
| `evaluation.md` | 效果是否符合业务 |
| `feedback.md` | 使用者反馈——**留空给用户填，一个字都不要代笔** |

拆分依据对齐他们既有的三段口径：`docs/brd`（为什么）→ `docs/prd`（怎么做产品）→ `docs/add`（技术架构）。

**`evaluation.md` 的写法是最容易做错的一件**：先写业务侧验收标准，再写现有证据，最后老实交代证据到哪一层。实例里第一版任务把成败标准定成了「分类准确率」，那是**从标签倒推任务留下的胎记**——评估文档的第一件事就是把标准挪回业务侧（本域即「做出来之后人不再手工做它」），然后明说「目前无法判断是否符合业务：证据只到模型层」。

**改动方式**：改名/移动用 `git mv`（保留历史），别删了重建；每次改动同步 README 的文档清单 + 本仓 CHANGELOG `[Unreleased]`，再逐层更新父仓指针。
