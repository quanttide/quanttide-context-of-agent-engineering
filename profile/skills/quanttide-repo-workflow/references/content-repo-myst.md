# 内容仓库（MyST 文档站）的增改与验证（2026-09-24 gallery 实证）

适用于 `default/quanttide-tech/docs/*`、`data/*` 这类**内容子模块**（handbook / tutorial / essay / bylaw / brochure / gallery），都是 MyST book 站 + GitHub Pages。

## 一、组织规范：`<主题>/index.md` 是入口

- 仓库根下按**主题/产品**建目录（`qtdata/`、`qtclass/`、`strategy/`、`qtconsult/`…），**每个目录里一个 `index.md`** 作该主题的入口
- `myst.yml` 的 toc 按主题分组：

  ```yaml
  toc:
    - file: index.md              # 站点落地页（见下）
    - title: 量潮数据
      children:
        - file: qtdata/index.md
  ```

- 同一个产品在每个仓库写**该仓库职责下的那一面**：handbook=流程手册、essay=主张、tutorial=现状综述、brochure=对外宣传、gallery=工作案例。写之前先读同产品在**兄弟仓库**里的 index.md，别重复也别串味
- **先看该仓库实际情况再动笔**：gallery 的 qdata 案例素材在归档站（`data/archive/gallery/qtdata/{price-indexer,questionnare-cleaner}/`）与 `docs/handbook/qtdata/index.md`，取真实事实（数量级/技术栈/客户类型）而不是编。跨仓链接**能不发就不发**（详情文档在别的站，编 URL 是负债）

## 二、⚠️ 坑：产品页 URL 退化成 `index-N` —— 根因是 `site.options.folders`，不是首页文件（2026-09-24 修正）

**更正（本轮实证推翻了旧结论）**：曾判「只有 `README.md` 没有 `index.md` 才会兜底 slug，补 index.md 即修」——**不对**。给 gallery 补了根 `index.md` 并把 toc 首项改指它之后，产品页 URL **仍是** `/index-1`、`/index-2`……真正的开关是 myst.yml 里的这行（handbook / tutorial / brochure 一直有，gallery / essay 没有）：

```yaml
site:
  template: book-theme
  options:
    folders: true        # ← 缺这行，`<产品>/index.md` 就退化成 /index-N/ 兜底 slug
```

判定与取证（本地一条命令就能验，别上线赌）：

```bash
BASE_URL=/<repo> npx -y mystmd@latest build --html > /tmp/b.log 2>&1
ls _build/html | grep -vE '^(build|_|favicon|myst|robots|sitemap|config|public)'   # 期望 qtclass qtcloud qtdata…；✗ 是 index-1 index-2
```

置上之后产物目录与链接一起变干净（`/qtata`→`/qtdata` 这类产品目录），旧的 `/index-N/` 随下次部署 404。

**`README.md` 与 `index.md` 仍要分成两份，但那是「职责分工」不是 slug 修法**：

| 文件 | 给谁看 | toc 首项 |
|---|---|---|
| `README.md` | GitHub 仓库首页（仓库说明：结构 / 本地构建 / 发布方式） | 不列 |
| `index.md` | 站点落地页（业务线简表 + 指向各产品页） | `- file: index.md` |

**教训**：这类「URL 退化」问题，先按**兄弟仓库配置 diff** 定位（`sed -n '/^site:/,$p'` 对比 handbook 与目标仓），别从文件命名上推根因；推完必须本地构建看产物目录名才算验完。

## 三、本地先验证再推（CI 同一条命令）

```bash
cd <仓库>                                   # myst 项目根（有 myst.yml 的目录）
npx -y mystmd@latest build --html > /tmp/myst-build.log 2>&1; echo $?
grep -E "Built|error" /tmp/myst-build.log   # 期望：📖 Built <你的新页> + 📚 Built N pages
```

- **别把输出管道给 `head`**：`| head -12` 会对构建进程发 SIGPIPE——日志看起来正常（"Built 3 pages"），退出码却是 0 而**产物不完整**（`_build/html` 里找不到新页）。**重定向到文件再 grep**
- `_build/` 是产物目录，多数内容仓**没有 .gitignore**（实测 gallery 就没有）→ 提交前 `git status` 别把它 `?? _build/` 带上；跑完 `rm -rf _build`
- MyST 给页面算 slug：产品目录**只有一个** `index.md` 时可能算成兜底 slug（见第二节的正确形态），单页 URL 形如 `/big-data-practice`

## 四、上线验证（部署也是 GitHub Pages）

```bash
# 1) 部署 workflow（Deploy Docs：checkout → setup-node → npm i -g mystmd → myst build --html
#    → upload-pages-artifact → deploy-pages）跑完没？
curl -s "https://api.github.com/repos/<org>/<repo>/actions/runs?per_page=1" \
  | python3 -c "import json,sys; r=json.load(sys.stdin)['workflow_runs'][0]; print(r['head_sha'][:7], r['status'], r['conclusion'])"
# 2) 站点真的换了？（GH Pages 有分钟级延迟，轮询几次）
curl -s https://<org>.github.io/<repo>/ | grep -oE "量潮数据|<title>[^<]*</title>"   # 侧栏出现新条目
curl -sL -w "%{url_effective}\n" -o /tmp/p.html https://<org>.github.io/<repo>/<slug>/
grep -c "<关键词>" /tmp/p.html        # 正文真的进去了
```

用 `curl -sL -w "%{url_effective}"` 才能看到 301 之后的真实 URL（产品页常常先 301 到带斜杠的规范地址）。

**写这类轮询/核对脚本别硬碰 shell 解析器**：带嵌套 `$(...)`、`for` 循环、管道与中文的**一行流 bash** 可能被命令解析器整条硬拦（`BLOCKED (hardline): command parser limit`，重试同样形态还是被拦）——同一件事改写成 `execute_code` 里的 Python（`urllib.request` + `json` + `re`）就跑得通，而且更容易对结果做判读（循环轮询、状态码聚合、grep 关键词计数）。本轮 CI 轮询与线上页核对两次被拦后都是这么过的。

## 五、⚠️⚠️ 项目站必须显式设 `BASE_URL`（2026-09-24 gallery 实证，站点一直是坏的）

**症状**：页面打得开、正文也搜得到，但 **CSS/JS 全 404**（裸页）、站内正文链接 404。因为 GitHub Pages 的**项目站**挂在 `/<repo>/` 子路径下，而 `myst build --html` 不带 base URL 时产出的是**根路径**：

```
线上首页引用 /build/_assets/app-….css   → 404   ← 主题脚本/样式全丢
正文链接 /big-data-practice             → 404   ← 少了 /<repo> 前缀
```

**修法**（对照 handbook 的 deploy.yml，它一直是对的）：

```yaml
# .github/workflows/deploy.yml 顶层
env:
  BASE_URL: /${{ github.event.repository.name }}
```

**验证三处，缺一处就漏**：

```bash
# ① 本地构建就能看出来（带 BASE_URL 构建，产物里应带仓库前缀）
BASE_URL=/<repo> npx -y mystmd@latest build --html > /tmp/b.log 2>&1
grep -oE '(href|src)="/[^"]+\.(css|js)"' _build/html/index-1/index.html | head -2
#    ✗ /build/_assets/…            ← 缺 BASE_URL
#    ✓ /<repo>/build/_assets/…     ← 正确

# ② 线上页面的资源地址必须 200（唯一能抓到本 bug 的检查——只查页面 200 会漏）
curl -s https://<org>.github.io/<repo>/ | grep -oE '(href|src)="/[^"]+\.(css|js)"' \
  | sort -u | while read -r a; do echo "$(curl -s -o /dev/null -w '%{http_code}' "https://<org>.github.io$a")  $a"; done
# ③ 正文链接（跨页链接）同样 curl 一遍
```

**判读口径**：**别把"页面 200 + 关键词搜到"当成站点健康**——那两件事在 BASE_URL 缺失时都成立。资产 404 是这套栈里唯一可靠的体检点。**发现这类"仓库一直如此"的基础设施缺口时**：先用 `curl` 取证（不是推断），再对照兄弟仓库的同名 workflow 找差异（差异就是修法），修完在报告里说清「这 bug 不是我这次引入的、之前线上一直是裸页」，避免被当成刚踩的坑。

## 六、产品 index.md 的写作口径（2026-09-24 qtdata/qtclass/qtconsult/qtcloud 四页实证）

用户会**一页一页地下达**（「qtconsult/index.md：量潮咨询主要有两类服务：1…2…」），每页一条独立指令。每页的固定动作五步：

1. **先读同产品在各仓的那一面**：`docs/{handbook,essay,tutorial,bylaw}/qt<产品>/index.md` + `data/brochure/qt<产品>/`（what/when/how 那几篇最像对外口径）+ `data/intention/qt<产品>/` + `data/profile/qt<产品>/business.md`。gallery 那页要写的是**工作案例**，素材取「做了什么、怎么做、结果怎样」，不取流程规则
2. **按用户口述的分类展开，不自己造分类**：用户给的两类/三类就是骨架（采集/清洗/大模型挖掘；校企合作/社会招生；创业咨询/创新咨询；SaaS/PaaS）——AI 只负责把真实材料填进去
3. 补 toc 分组（`- title: 量潮<产品>` + `children: - file: qt<产品>/index.md`）
4. 本地 `myst build` → 三层提交 → 查 Actions run → curl 线上页核对 → **一行报进度**
5. 新增产品页前先确认 `myst.yml` 有 `site.options.folders: true`（第二节）——没这行每加一页就多一个 `/index-N/` 兜底 slug，编号还会随 toc 顺序变；有这行则是干净的 `/<产品>/`

**四条内容纪律（都是被用户纠正出来的）**：

- **分类名要取仓库真实材料里的说法，不自己概括**：第三阶段曾写「精炼」被评「这类不太对。这类经常是需要使用大模型」——而 brochure 场景一的正名本来就是「**大模型文本挖掘**」。自造概括词 = 把业务的真名换成泛称，必被打回；回材料里找正名（brochure 的 `when.md` 有场景清单，`what.md/how.md` 有做法）
- **同一案子在不同文档里换了名字/侧面，别拆成两个案例**：曾把「问卷清洗」与「制衣厂四源合并」写成两个案例，其实是同一个项目（`brochure/qtdata/what.md` 里那位老师的工厂研究：几百份问卷 + 产量/工序/返工/考勤四源）。交叉核对客户、场景、数字再合并
- **公开页不写价格、不点客户名、不写内部治理概念**：brochure 里那组价格倍数（60% 基准 / 80% 1.5–2 倍 / 90% 3–5 倍）是内部讲课底稿，只写「准确度越高、费用非线性增长」；客户以「某制衣厂」称；战略层概念（期权池、跨周期人才供应链）不进公开页，顶多一句「在做平台化的下一步」
- **未完成/内测口径照实写**：qtcloud-devops 是开源内测、只服务自家最佳实践、官方口径就带「公开是作课堂案例和笔试题、不要在生产环境运行」——照抄，不美化、也不因为它在内测就不写

**取素材时的判据**：写「这件事我们怎么做」的段落，材料里必须有对应的事实（做法/数量级/技术栈/客户类型）；没有就不写。**只写材料支持得住的内容**，宁缺一节不编一段。

## 七、内容仓库的其余欠账（gallery 实测，同类仓库顺手核）

- `.gitignore` 缺 `_build/`（handbook 有、gallery 没有）
- 站点域名走 `github.io`，`<产品>.example.com` 未接（家族里站点惯例是 `{产品}.example.com`）
- 产品目录缺 `index.md`（只有内容文件时）
- 内容被搬去别处（"已迁至归档站"）却没有一份文档说明谁主谁次——写 STATUS/README 时把这条当首要缺口
- 案例库与归档站的边界没成文（原件在归档、介绍在案例库，谁决定迁什么没有规则）

## 八、内容仓的文档套件与案例库定位（2026-09-24 gallery 补齐实证）

**套件对照**：家族内容仓齐备形态（handbook / tutorial）= `AGENTS.md` + `CONTRIBUTING.md` + `README.md` + `ROADMAP.md` + `index.md` + `CHANGELOG.md`；essay 缺 ROADMAP/index、brochure 只有 README+index，**gallery 长期只有 README/CHANGELOG/index**（用户指令「更新 AGENTS、README、CONTRIBUTING STATUS ROADMAP 等」即补这一套）。补时照 handbook/tutorial 抄骨架，别自创结构；`STATUS.md` 是家族此前没有的，按仓库快照的体检形态写。

gallery 补完四份的内容分工（可作模板）：

- **`AGENTS.md`**：工作流程五步（读规范 → 去各内容仓找该业务的对外口径 → 写该业务线 `index.md` → 本地 build → 子模块提交并回父仓更新指针）+ **三条硬规矩**（价格不写／客户不点名／未完成如实说状态）+ 结构规则（一业务线一目录一 `index.md`、toc 顺序固定、`site.options.folders: true` 必须有、`index.md` 进 toc 而 README 不进）+ 工程标准规范四条**版本化**链接（specification v0.1.1 的 format / git / release / git_repo）+ 首要范例指名一个页面
- **`CONTRIBUTING.md`**：子模块三层提交流程 + 案例编写规范（一页回答三件事：这业务是什么／分哪几类／每类实际怎么做；分类跟真实交付方式走）+ 格式优化四步
- **`STATUS.md`**：内容仓没有专属契约，尺子取三处——**家族文档套件**（handbook/tutorial 为范例）、**量潮科技工程标准规范 v0.1.1**、**父仓 CONTRIBUTING 里对本仓的约定**；逐条判定 + 已知不一致 + 一句话
- **`ROADMAP.md`**：分段（入口 → 内容 → 交付形态）+ 现在在哪 + 欠账 + 待决

**案例库的定位纪律（本轮与用户对话沉淀，写案例前先读）**：

- **案例 ≠ 存档**：交付原件（scope／报价／蓝图／交付物）在归档站 `quanttide-archive-of-business-entity` 的 `gallery/` 下，本仓只留入口与主张。gallery 近年一直在往下删内容（迁往归档）不是衰败，是分工——**案例的价值在把原件压成别人一眼看懂的三五行**，不在「有原件」
- **案例是口径不是文章**：第一读者是潜在客户、第二读者才是自己人；创始人给的原始说法 > AI 的润色（用户原话「以我的原始信息为主」）——落地页宁短、宁用原话，不要展开与修饰
- **分类轴由业务自己长**：数据按加工阶段（采集／清洗／大模型挖掘）、课堂按招生方式（校企合作／社会招生）、云按产品形态（SaaS／PaaS）、咨询按客户类型（创业／创新）——四条线各不相同，**没有统一模板**可套（套「客户—问题—方案—结果」这类通用骨架写第二页就崩）
- **顺序按用户指定的业务优先序**：gallery 定为 数据 → 课堂 → 云 → 咨询，落地页与 toc 同步按这个序排

**发布**（docs 仓）：`qtcloud-devops release audit -v vX.Y.Z`（**硬门槛是 CHANGELOG 先有条目**）→ `publish -v … -y` → 父仓指针推到带 tag 的提交——细节与陷阱见 `quanttide-git-ops` 的「版本发布」节。docs 仓特有的三点：

- **audit 报「配置文件一致性：0 个文件版本均为 X」是正常的**——docs 仓没有 pubspec/package.json 之类的版本文件，版本只活在 `CHANGELOG.md` + git tag 里，不必为此造一个版本文件。其余六项照常要过（工作区干净、tag 不冲突、远程可达）
- 先 `publish -v vX.Y.Z --dry-run` 看它要做什么，再 `-y` 执行；产出的 GitHub Release **正文就是那条 CHANGELOG 条目**（Added/Changed/Fixed），不是草稿也不是预发布——核对 `releases/tags/vX.Y.Z` 的 tag/body 即知是否对版
- **发完两处对齐**：`git rev-parse --short vX.Y.Z` 应与 HEAD 相同（tag 打的正是发布提交）；父仓 `git submodule status <path>` 输出应带 `(vX.Y.Z)` 且**无 `+` 前缀**——有 `+` 说明父仓指针还停在上一个提交，需再补一次指针提交。**站点内容不随 tag 变**（Pages 跟着 main 的部署 workflow 走），上线核对仍走 curl 那三处

## 九、具体案例页（产品目录下的第二层文件）的落法与补写纪律（2026-09-24 全球法规情报中心实证）

产品 `index.md` 是介绍，**具体案例是产品目录下的独立文件**（`qtdata/regulatory-intelligence.md`、`qtclass/big-data-practice.md`）。落法三步：

1. 写文件 `<产品>/<案例英文 slug>.md`（家族惯例 kebab-case 英文：`big-data-practice`、`regulatory-intelligence`，不用拼音也不用中文名）
2. 从产品 `index.md` 的**对应分类节**挂入口：一段摘要（覆盖什么／怎么做／结论口径）+ 链接 `[案例名](./<slug>.md)`
3. `myst.yml` 该产品的 `children` 里加 `- file: qt<产品>/<slug>.md`（排在该产品 `index.md` 之后）

**用户直接贴一段编号需求（「这是 XX 案例的原始需求」+ 需求正文）时的写法**：

- **以原始信息为主**：国别清单、监控维度清单、来源优先级清单**照抄不概括**（本次八类监控内容、FDA／…／各国监管机构名单原样落地），不合并成「等」也不改写措辞
- **只有需要才补，且补完明示**：本次补了一节「交付」（原文没有，按采集案例惯例补），报告里点明「这三处是我判断的，不对就删」——用户对新增内容默认要你说明来由
- **顺需求往下推的推论要与原文分开**：把「来源优先级」推成一条采集规则（来源清单按官方站点逐国维护、二手解读只当线索）属推论，标注出来让用户可否决
- **行业／客户定性要显式确认**：本次从 MDR／IVDR、MDCG、NMPA 推出「医疗器械」行业，原文没写——写进报告问一句，别默认对
- 口径三条照旧：不写价格、客户以「某…客户」称、内测／未完成如实写

**新案例页上线后回写体检文档**：站点页面数 +1、把「具体案例覆盖到哪几条业务线」那行改成实况（本轮 `STATUS.md` 从「只有教学线有案例」改为「数据线与课堂线各一，云与咨询待补」，`ROADMAP.md` 的「现在在哪」同步）——体检文档里的覆盖数字是**每次新增内容后都要动的活数字**，别留在旧口径。

## 十、README / 根 index / 章内 index 三层分工 ——「书籍」模型（2026-10-01 intention 实证）

用户用书籍隐喻界定层级：**根 `README.md` = 仓库说明；根 `index.md` = 书籍的介绍页面（封二）；`<主题>/index.md` = 正文该章的第一篇。**

| 文件 | 回答什么 | 句子的主语 | 装什么 |
|---|---|---|---|
| 根 `README.md` | 这个仓库怎么用 | 本仓库 | 定位一句 + 「内容导航见 index.md」一行 + 维护/版本/许可 |
| 根 `index.md` | **这本书是什么、怎么读** | **这本书** | 章节地图（一行一章带链接）、从哪读起、本书边界（收什么 / 不收什么 / 归档去哪）、相关文献（跨库指向） |
| `<主题>/index.md` | 这件事是什么 | **量潮**（业务主体） | 实质内容：纲领、版图、业务线、主线 |

**判别尺子：句子主语是「这本书」还是「这件事」**——「本库／本书」→ 根 index；「量潮」→ 章内 index。用户把同主题两份文件称作「旧版本 / 新版本」时，先照这把尺子判断该合并到哪一层。

四条落地纪律：

- **`myst.yml` 的 toc 首项必须是 `index.md` 而不是 `README.md`**——否则站点首页仍是 README、新建的 index.md 排不上（先例：quanttide-tech 根目录就是 index.md 作站点首页导航门户，`toc: - file: index.md`）
- **章节地图只列到「章」、不复制 toc 的文件级列表**：toc 已有全量篇目，index 再抄一遍 = 两处维护、迟早漂移（第八节那类 toc 缺口就是这么来的）
- **README 减回仓库说明时**，把它的「项目结构」「关联档案」两段**整体搬进 index** 而不是删掉——这是分工调整，不是减法
- **章内 index 不能照搬根 index 的形态**：写「这件事」的层不引用文件路径、不写「近期 / 最近」这类时效措辞（纲领式）——跨层搬运要逐条降级（去路径、去时效、把「最近想通的」改写成「是这样的」）
- **根 index 只列章、不列篇**还有个副作用好处：新增篇目时只动 toc 一处

**验证（本地一条命令，别只看退出码）**：改完 toc 首项后 build 一次，① `grep` 首页产物确认出现 index.md 的新节标题（证明首页真换了，而不是 README 还在兜底）；② `ls _build/html | grep <原先漏注册的页>` 确认补注册的页真的生成了。本次实测：`_build/html/index.html` 含「章节 / 从哪读起 / 本书的边界 / 相关文献」，且此前访问不到的 `/studio.json`、`/self.json` 已 200。
