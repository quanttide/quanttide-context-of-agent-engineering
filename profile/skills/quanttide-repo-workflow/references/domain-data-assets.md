# 领域第二大脑的 data 资产：命名、布局与分工

2026-10-01 quanttide-algorithm（算法工程）从零建域实证：一天内 context → intention → insight → profile 四个 data 子仓依次补齐。本文记「叫什么名、里面长什么样、谁写什么」。

## 一、仓库命名

- 领域仓：`quanttide-<缩写>`（缩写取中文名的英文短形，如算法工程 → `algorithm`）
- data 子仓：`quanttide-<类型>-of-<英文全称>`，英文全称是把中文「xx工程／xx管理」翻全

**动手前先校对**，不要凭印象拼「of-xxx」：

```sh
grep -A2 'submodule "data/<类型>"' <兄弟域>/.gitmodules
```

已核实的对照（中文域 → 英文全称）：

| 域 | 缩写仓名 | 英文全称 |
|---|---|---|
| 算法工程 | quanttide-algorithm | algorithm-engineering |
| 软件工程 | quanttide-code | software-engineering |
| 知识工程 | quanttide-knowl | knowledge-engineering |
| 学习管理 | quanttide-learn | learning-management |
| 执行管理 | quanttide-execute | execution-management |
| 认知工程 | quanttide-think | cognitive-engineering |
| 元工程 | quanttide-meta | philosophy |

`--description` 用中文，与邻居一致（`量潮算法工程`／`量潮算法工程语境`）。

## 二、各 data 子仓的文件形态（抄兄弟仓，别自创）

| 子仓 | 布局 | 本次实例 |
|---|---|---|
| context | `README.md` + `<主题>/`；实验记录放 `laboratory/README.md` + 同目录可复现脚本 | `context/laboratory/layer_classification.py` |
| intention | `README.md` + `<主题>/index.md`；纲领取，**不引用文件路径**、不写状态与工程细节 | `intention/incubation/index.md` |
| insight | `README.md` + `<slug>.md` | `insight/candidate-selection.md` |
| profile | `README.md` + `<产品>/<项>/index.md`（**两级**，见下） | `profile/quanttide-asset/asset-classifier/index.md` |

套件范围：`README.md` + `CHANGELOG.md`（`[0.1.0] 初始化`）+ `LICENSE`（`cp` 兄弟仓的，全族 CC BY 4.0）。有些兄弟域没有 CHANGELOG，同域内保持一致即可。

## 三、profile 的两级目录惯例

`data/profile/<产品>/<项>/` —— 第一级是产品名，第二级是这个产品下的具体项：

- `quanttide-data/data/profile/qtdata/{ghtorrent, sec-credit-cleaner, uspto-entity-matching, judicial-document}`（项＝项目）
- `quanttide-code/data/profile/qtcloud-asset/{cli, provider, studio}`、`qtcloud-auth/index.md`、`qtclass/site`（项＝模块）

档案架子（对齐 `sec-credit-cleaner/index.md`、`uspto-entity-matching/index.md`）：标题 → `> 脱敏说明`（本档案记什么、隐去什么）→ 定位 → 核心目标 → 经验。

## 四、档案 ≠ 实验记录（单一事实源）

- **实验的过程记录**留 `context/laboratory/`：目的／判据／数据／做法／结果／排除的候选／结论 ＋ 可复现脚本
- **profile 只写档案**：定位／结构／现状／可复用经验，开头一句指明事实源在语境仓的实验室
- 两处**不复制**。用户说「把这实验整理到 profile」时，落点是**档案**而不是把实验室那份搬过去；如果他的意思是搬家，再改

## 五、README 与 index.md 的分工（书籍比方）

用户提的比方：`index.md` 是**书籍的介绍页面**，正文第一篇是**内容**。据此分：

- `README.md`＝**仓库说明**：定位一句话 ＋ 入口指引 ＋ 维护／版本／许可 —— 讲「这个仓库」
- `index.md`＝**内容入口**（站点首页）：章节地图 ＋ 从哪读起 ＋ 本书边界 ＋ 相关文献 —— 讲「这本书」

**判据：句子的主语是「这本书」还是「这件事」。** 「本库／本书」→ `index.md`；域的业务内容 → 正文篇。先例 `default/quanttide-tech` 根目录就是 README ＋ index 并存。

改动清单（一次做齐）：

1. 新建 `index.md`；`README.md` 减回仓库说明，把「项目结构」「关联档案」这类**内容导航**搬去 index
2. **`myst.yml` 的 toc 首项换成 `index.md`**（先例：quanttide-tech 首项就是 index.md，不是 README）——顺手核一遍 toc 有没有漏注册的文件，本次补了 qtconsult 的 `product/studio.md` 与 `project/self.md`
3. 章节地图**只列到「章」**，不重复 toc 的篇目列表（同一份目录两处维护必漂移）
4. 验证：`npx -y mystmd@latest build --html > /tmp/log 2>&1`（**别把输出管道给 head**，SIGPIPE 会截断而退出码仍是 0），确认首页来自 index.md、新增页已生成

## 六、隐私边界（本次踩到的）

- `data/context` 等仓是 **PUBLIC**：汇总类内容要比原始草稿**更收紧**——人名换职务、客户身份模糊化、金额不写。「原草稿里就有」不构成照搬理由
- 私有侧 `quanttide-tech-private`（约 1000 篇）**不按 `<层>/` 组织**（按业务线／职能／人名），所以公开侧按层做的实验**扩不过去**，别指望样本翻倍
