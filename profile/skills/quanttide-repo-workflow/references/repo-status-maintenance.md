# 仓库快照 STATUS.md 的维护（体检）

触发语（用户原话）：**「查看 <仓库> 的 STATUS.md，如果没有的话创建一个，过时就更新。」**
同一模式还出现过配套指令：把仓库 README 一并按实际结构修（用户答 yes）。

## 两类同名文档，别搞混

| 类型 | 落点 | 读者/形态 |
|------|------|----------|
| **仓库快照 STATUS**（本文） | `apps/<app>/STATUS.md`（qtdata、qtclass…） | 协作者/AI 快速了解现状：Scope 状态 + 业务定位 + 差距分析 |
| **契约对齐三件套 STATUS** | `apps/<app>/src/<scope>/{STATUS,TODO,ROADMAP}.md` | 逐条契约 × 现状 × ✓/✗/—，每行带可比数字（详见 quanttide-app-stack 技能的 references/studio-contract-alignment.md） |

本次是前者。用户说「studio 的 status 看看」时先 `ls src/studio/*.md` 确认落点是否存在——qtdata 的 studio **只有** ROADMAP/CHANGELOG/README，无 STATUS/TODO；且 README 还是 `flutter create` 的英文模板原文（未定制）。补三件套前先问「拿哪份契约当尺子」（候选：`docs/dev-guide/stories/*.md` 业务线拆解 vs 领域契约文档），两者口径不同。

## 格式：同一仓库树内的版本要对齐

范本 = 同仓后来更新的兄弟档 `default/quanttide-tech/apps/qtclass/STATUS.md`（2026-08-26）：

```
# <app> 状态报告
> 更新日期：YYYY-MM-DD
> 仓库：quanttide/<repo>
> 最新 commit：<sha> (YYYY-MM-DD)
> 版本记录：`src/<scope>/CHANGELOG.md`（逐个列）
## Scope 状态
| Scope | 最新版本 | 状态 |
### <Scope>（每个 scope 一段大白话：技术栈 + 做到哪 + 部署目标）
```

qtdata 这份保留了仓库专有节，与上述头部/表形对齐：

```
## 业务定位          ← 一句话业务边界 + 来源 journal 日期
## Scope 状态        ← Scope | 目录 | 最新版本 | 状态 + 各 Scope 一段
## 战略差距分析      ← 三方对照 + 尺子说明 + 根因
## ROADMAP 进度      ← 每个 ROADMAP 文件一行，带「与实现的符合度」
## 文档覆盖          ← 目录级状态（含已移除的旧目录，注明移除日期与承接者）
## 已知不一致（待处理） ← 体检结论，逐条可核对
```

旧版（2026-07-20）标题是「更新日期/仓库/最新 commit/最新版本」+「版本历史/组件进度」——**版本全景可折进 Scope 状态表**（tag 信息写进「最新版本」列），不必单列一节。

## 采集命令清单（数字必须实取，不许凭印象）

```bash
git pull --ff-only                                   # 先确认本地=远端（打印"已经是最新的"）
git status -sb; git rev-list --left-right --count origin/main...HEAD   # 0 0 = 一致
git log --oneline -10 --date=short --pretty="%h %ad %s"
git tag | sort -V                                    # ← 权威 tag 表（见陷阱）
find src -maxdepth 2 -type d | sort                  # 组件结构
find <scope> -name '*.dart' | wc -l                  # 文件数
find <scope> -name '*.dart' -exec wc -l {} + | tail -1   # 总行数
for d in lib/screens lib/widgets lib/models; do ... done # 分层文件数/行数
grep -inE "http|dio|shared_pref|provider:|bloc" <pubspec.yaml|package.json>   # 依赖即能力边界
ls -1 .github/workflows/                             # 部署机制（tag 触发什么）
head -12 src/<scope>/CHANGELOG.md                    # 各组件版本条目
find . -name ROADMAP.md -not -path "*/node_modules/*" | sort
find . \( -name STATUS.md -o -name TODO.md \) -not -path "*/node_modules/*"
```

三方对照（差距分析）：`data/journal/<业务线>/*.md`（真实业务）+ `data/intention/<业务线>/*.md`（战略）+ `src/`（实现）。

## 「尺子会变」——更新差距分析前先核对边界（本次最重要的发现）

旧 STATUS 的「战略差距分析」按**平台化终局**量 qtdata，得出三方平台/供给侧整合/定价权一堆「100% 差距」。但 2026-09-04 边界确认：**平台型需求归 qtcloud（自建平台+卖标准品），积木型需求归 qtdata（可拼装、按需组合）**——那批差距**属于边界外，不是本仓欠账**。

规则：拿到旧 STATUS 的差距分析，**先问「当初量差距的尺子现在还成立吗」**，再决定刷新还是重写。直接沿用旧维度会把别的业务线的目标算成本仓的欠账（用户会立刻看出不对）。重写后按新边界列真实差距，并单列一行写「边界外（本仓不背）」。

qtdata 现状的真实差距（可作量法的样板）：实现层只覆盖数据加工最浅一段（Markdown→JSON 的 LLM 转换 + 看板展示），而意图写的是**技术管理体系**——需求拆解 / 过程管控 / 质量兜底三件事都还停在意图层。另有一条值得写进 STATUS 的形态差：**界面跑在数据模型前面**（Studio 5 Tab 观测面已做，数据来自 seed JSON，连 Provider 的网络依赖都还没引）。

## 两类「脱节」要分清

| 形态 | 判据 | qtdata 实证 |
|------|------|------------|
| 没做 | ROADMAP 有、代码里找不到 | CLI 四阶段全未勾选，只有 46 行 `main.rs` 的骨架 |
| **换了做法没回头改** | 事做了，但实现形态和 ROADMAP 的写法不一致 | Studio 的 ROADMAP 写「新增 qtdata-data/qtdata-asset 两个 package + 增加数据/资产页面」，实际没有 `packages/` 目录——数据页/资产页做成了详情页的 Tab |

写「符合度」列时要指明是哪一类，措辞差别很大（「事做了，路线写法过时」≠「没做」）。

## 版本号核实顺序（别信 commit message 和旧文档）

`git tag` > 各组件工程文件（Cargo.toml / pubspec.yaml / package.json / pyproject.toml）> 该组件 CHANGELOG > commit message。

实证坑：qtdata 有个提交信息写着 `v0.0.2 — 新增 CLI 工具`，但**这个 tag 根本不存在**，`src/cli/CHANGELOG.md` 记的是 `[v0.0.1]`，`Cargo.toml` 已是 `0.1.0`——旧 STATUS 直接照抄 commit message 写成「最新版本 v0.0.2」，错了两个月。这类不一致本身就是「已知不一致」里值得记的一条。

## README 同步体检（用户会接着让你修）

README 只做入口，但结构块必须**照着实际目录重写**。qtdata 实证的过时项：列着早已删除的 `docs/{brd,prd,add,dev,ixd,drd,pmd}`、不存在的 `assets/{images,videos}` 与 `src/studio/packages/`，且完全没提 `src/cli`、`src/site`。重写后结构块与 `## 文档索引` 表（指向 STATUS/AGENTS/各 ROADMAP/组件 CHANGELOG）即可，不塞内容。

## 父仓与子仓 STATUS 的关系

子模块自己的 STATUS 才是权威。父仓（如 `default/quanttide-tech/STATUS.md`）若也有同名小节（实证：父仓有「qtdata 差距分析」），检查它是否只有来源表、没有分析内容（像写了一半）——若是，**建议改成一行指向子模块的 STATUS**，而不是两处各写一份（两份必然漂移）。

## 报告形状（给用户的收尾）

1. **过时点清单**：旧文件错在哪（逐条，带证据）
2. **改了哪些文件** + 变更量
3. **最新的 status 摘要**（用户可能直接要「告诉我最新的 status」）——平铺结构：定位一句 / Scope 表 / 真实差距 / 已知不一致
4. **未提交 + 为什么**（见下）；提交走本仓分层链（子模块 → 中层 → 根）

## 陷阱

- **`git tag --sort=-creatordate | head -12` 会静默截断**，看起来像「最新的 tag 就是这些」——实权在 `git tag | sort -V`（本次前者漏掉了根级 `v0.0.2` 的判断依据）
- **判代理是否可用，最终判据是 `git ls-remote origin` 能否返回 sha**，不是端口在不在监听：本机 7897 端口**在监听**、`https://api.github.com` 返回 200，但 `https://github.com` 直连超时；经代理的 `git ls-remote` 正常返回即通（端口在监听 ≠ 代理可用）
- **`git -c http.proxy=` 绕过全局代理**只对「代理没开」有效；代理活着时反而要走默认配置
- 写文件后 `chmod 664`（仓库文档惯例 `-rw-rw-r--`）
- 子模块里改完，父仓 `git status` 显示 **小写 `m` <submodule>** = 子模块内部有改动待提交（大写 `M` 才是 gitlink 指针变化），别混
- **动手前先 `git pull --ff-only`**：本地可能落后，STATUS 会基于旧事实
