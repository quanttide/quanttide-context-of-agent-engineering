---
name: quanttide-content-repos
description: 写/改量潮内容仓库（MyST 文档站）时触发：业务线组织、对外口径、发布排障。
---

# 量潮内容仓库（MyST 文档站）的写作与发布

量潮的内容仓库族在 `quanttide-tech` 下挂成子模块，各自是独立 git 仓库（`quanttide-<主题>-of-business-entity`），站点多为 **MyST book-theme → GitHub Pages**：

| 位置 | 内容 | 读者 |
|---|---|---|
| `docs/gallery` | 量潮科技工作案例（**业务案例库**） | 潜在客户、合作方、应聘者 |
| `docs/handbook` | 工作手册（流程、清单、话术模板） | 自己人（操作） |
| `docs/tutorial` | 教程与业务介绍（现状、卖点） | 新人、外部 |
| `docs/essay` | 工作札记与主张（为什么） | 半公开 |
| `docs/bylaw` | 章程 | 内部 |
| `data/brochure` | 宣传册底稿（客户视角 5W1H） | 外部 |
| `data/archive` | 归档站（**交付原件**：scope／blueprint／quotation／delivery） | 留档 |

**同一件事在多个仓各有一版、口径不同**——这是本族最重要的事实。写对外材料时的工作是「跨仓取材 + 压回对外口径」，取材地图与四条业务线的原始口径见 `references/business-lines.md`。

## 组织规范（一业务线/主题一目录）

- 一个业务线（或职能主题）一个目录，**目录内一个 `index.md` 作入口**（`qtdata/index.md`、`qtclass/index.md`…）；具体案例作为同目录下的文件挂在它后面（`qtclass/big-data-practice.md`）
- `myst.yml` 的 toc 按业务线分组（`- title: 量潮数据` + `children: - file: qtdata/index.md`），顺序由用户定（gallery 现为 数据 → 课堂 → 云 → 咨询）
- **`site.options.folders` 必须为 `true`** —— 否则产品页 URL 退化成 `index-1`、`index-2` 兜底 slug（且编号随 toc 顺序变）；置上后才是 `/qtdata`、`/qtclass`（handbook／tutorial 一直有这行，gallery 曾缺）
- **`index.md` 进 toc 当站点落地页，`README.md` 只作仓库说明**（handbook 惯例：README 讲这是什么仓库，index.md 是站点首页）；README 与 index 曾共用一个文件（gallery 只有一行标题的 README 兼两职）
- 文档套件对齐家族：`AGENTS.md`（工作流程／口径规矩／结构规则／工程标准规范链接／首要范例）、`CONTRIBUTING.md`（子模块三层提交、编写与格式规范）、`README.md`、`ROADMAP.md`、`CHANGELOG.md`、`STATUS.md`（**这份是全家族都缺的，按应用仓的三件套体检法写**：规模／逐条对照／一句话）

## 案例／对外材料的三条口径规矩

1. **价格不写进公开页**（报价倍数、客单价、订阅方案）——价格是商务信息，留在 `data/brochure` 与业务档案里
2. **客户不点名**——写「某高校研究者」「某制衣厂」「某高新科技企业」；已公开的口径除外（浙江理工大学计算机系的校企合作已在课程材料中公开）。**项目里的人名也不上**
3. **未完成／内测的如实说状态**——例：量潮 DevOps 云命令行工具写明 v0.4.x 内测、公开它是作为课堂案例和笔试题、不建议生产使用

## 案例怎么写（2026-09-24 被纠正两次）

- **分类跟业务的真实交付方式走，不套统一模板**。四条业务线的切法各不相同：数据按加工阶段、课堂按招生方式、云按产品形态、咨询按客户类型——硬套「客户—问题—方案—结果」或「5W1H」都不成立。**名字不是修辞，是交付方式的指认**：把数据业务第三段写成「精炼」被当场纠正——「这类经常需要使用大模型」，正名是 brochure 里的「大模型文本挖掘」
- **宁少而准**：落地页要的是口径（谁读都不会误解我们在卖什么），不是文采。第一版把每页写满（还自己加「难点在哪」），用户改判为「以我的原始信息为主」——**用创始人给的原话**，别润色
- **一页回答三件事**：这条业务是什么／分哪几类／每一类里我们实际怎么做（给真实做法与数字）。范例：gallery 的 `qtdata/index.md`
- **素材必须是仓库里真有的**（十年 CPP 的 1000+ 关键词、500 万数据点／日；制衣厂四源合并的产值公式 `员工 × 工序 × 产量`；准确度抽检定价机制），文件里的数字、参数一律抄自真实材料

## 发布链与验证（必做，别只信「文件写完了」）

```bash
npm install -g mystmd                                 # 或 npx -y mystmd@latest
BASE_URL=/<repo-name> myst build --html               # 构建到 _build/html
BASE_URL=/<repo-name> myst start                      # 本地预览
```

- **`BASE_URL` 不能省**：省掉时产物里的资源路径与站内链接都是根路径（`/build/_assets/…`、`/qtdata`），本地能看，发到 GitHub Pages 项目站（`/<repo>/`）**全部 404**——gallery 曾长期如此，站其实是坏的（CSS/JS 都加载不到）。CI 里对应 `env: BASE_URL: /${{ github.event.repository.name }}`
- **验证要落到线上**：CI（Deploy Pages）跑绿 → `curl -sL` 真实 URL 看 `200`＋`<title>`＋正文关键字；**旧 URL 会 404、新 URL 才 200**，这正是 slug 修好了的证据。公开仓的 Actions 未认证 API 就读得到运行结果
- 构建产物 `_build/` 要在 `.gitignore` 里（gallery 缺这条，本地构建会留残留）
- 走自有域名（`gallery.example.com` 这类）时改走阿里云 OSS+CDN 那套，见 skill `aliyun-static-site-deploy`

### 版本发布（内容仓也走 release CLI，2026-09-24 gallery v0.1.2 实录）

```bash
qtcloud-devops release status                    # 当前版本 / 未发布提交数 / 变更日志
qtcloud-devops release audit -v v0.1.2           # 预检七项（先跑这个，别直接 publish）
qtcloud-devops release publish -v v0.1.2 -y      # 校验 CHANGELOG → 打 tag → 推送 → 建 GitHub Release
```

- **CHANGELOG 是唯一硬门槛**：audit 七项里只有「CHANGELOG 未找到 `<版本>` 记录」会拦（另六项＝版本号格式／配置文件一致性／工作区干净／标签可用／远程可达／Release 未发布）。docs 仓没有版本文件时该项显示「0 个文件版本均为 x.y.z」，照过。**顺序：按 Keep a Changelog 写 Added／Changed／Fixed 条目并提交推送 → 复跑 audit 到 7/7 → 再 publish**
- **版本号归用户拍板，绝不擅自发**；内容仓固定 patch——纯内容新增 + CI/URL 修复都算 patch（即使产品页 URL 从 `/index-1` 换成 `/qtdata`，站从未对外发过链接，仍按 patch；「无 API 变更＝patch」）
- **发布后把父仓指针更新到带 tag 的那个提交**（否则父仓记的是发布前的提交）：`git submodule status docs/<name>` **无 `+` 前缀＝已对齐**，有 `+` 即落后
- Release 正文自动取自 CHANGELOG 条目，用当前 `gh` 身份创建；`--dry-run` 可先看它打算改哪些文件、在不做任何操作的条件下过一遍

## 子模块提交流程（三层）

在内容子模块内提交推送 → 回 `quanttide-tech` 更新 `docs/<name>` 指针 → 回根仓库 `quanttide` 更新 `default/quanttide-tech` 指针。提交信息用 Conventional Commits（`docs:`／`fix(ci):`／`chore:`）。

## Pitfalls（本轮全部实踩）

- **缺 `BASE_URL`**：整站资源与站内链接 404（页面裸着渲染）——先看线上，别只看本地
- **缺 `site.options.folders`**：产品页 slug 变 `index-1/2/3/4`，还随 toc 顺序漂移，链接发不出去
- **`head -12` 截断构建**：`npx mystmd build --html | head -12` 会因 SIGPIPE 提前杀掉构建，日志看着成功、产物不全——重定向到文件再看
- **同一条业务在五六个仓各有一版**：直接抄某一版会漏对外口径（宣传册的倍数、档案里的价格、随笔里的内部语汇都不能上公开页）
- **tech 的 CONTRIBUTING 里「稳定的 SKILL 同步到 gallery」已过时**（写的是 `cp .agents/skills/<skill>/SKILL.md docs/gallery/devops/<skill>/SKILL.md`，本仓连 `devops/` 都没有）——本仓现定位是业务案例库，改它在 tech 仓
- **归档站与本仓同名同主题**（archive 里有 `gallery/qtdata/...` 全套原件）：原件在归档、介绍在案例库；「把原件压成人能看懂的三五行」才是案例的活，别把原件搬回来

## 支持文件

- `references/business-lines.md` —— 四条业务线的对外口径（用户亲口给的切法）、跨仓取材地图、每条业务线的现有案例与素材出处
- `references/myst-github-pages.md` —— MyST + GitHub Pages 的构建/发布/排障配方与验证命令
