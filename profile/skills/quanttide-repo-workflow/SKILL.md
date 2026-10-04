---
name: quanttide-repo-workflow
description: 使用当在 QuantTide 仓库体系查看/更新子模块、journal、intention 文档或发布版本。
---

# QuantTide 仓库工作流

在用户的 QuantTide 第二大脑知识体系（git 子模块 monorepo）中查看、更新、提交文档。

> 本 SKILL.md 较长，仓库结构与高频参照的完整版在 `references/repo-map.md`（含 data/profile 口径分型、组织名单、quanttide-org data 层现状）；本文件正文保留工作流与陷阱。quanttide-founder 实验室的创作谈引擎实验（2026-09-05 两轮定型：规则集版被否 → 用户给出文体定义后重写为文章体）的完整记录在 `references/creation-talk-engine.md`。**仓库快照 STATUS.md 的体检**（触发语「查看 <仓库> 的 STATUS.md，如果没有的话创建一个，过时就更新」：格式范本、采集命令、差距分析的尺子判据、README 同步）在 `references/repo-status-maintenance.md`

## 触发条件
- 用户要求查看/更新 quanttide-* 仓库（quanttide-founder、quanttide-tech 等）
- 子模块 pull/commit/push、journal 更新、intention 文档编辑、文档结构重组

## 仓库地图
- 根仓库 `~/repos/quanttide/`（父仓库只追踪子模块指针）
- 三级路径：`domains/{name}` 领域知识 / `default/{name}` 默认实现 / `assets/{name}` 资产
- `default/quanttide-tech` 内含：`data/journal`（工作日志，按业务实体分类）、`data/intention`（战略意图）、`docs/*`（文档子模块）、`apps/*`（应用）- 常用位置：`data/journal/qtadmin/`、`data/intention/qtadmin|qtcloud|qtclass|qtdata`
- `domains/quanttide-org`：`data/intention`（组织云建设意图 + `training-base.md` 实训基地公司级全景）+ `data/profile`（**组织档案**——一个文件夹是一个"组织"，非产品档案；名单：qttech 量潮科技 / qthold 量潮控股（拟设）/ qtalliance 量潮创新联盟 / qtfounder 量潮创始人 / qtinstitute 量潮研究院（拟设）/ qtacademy 量潮实训基地（重定义中：课堂+众包+招聘枢纽，内包子公司模型+传统职能部门编组）；README 即名单，见 9f；根 index.md = 组织模式总纲（内核"外化在场"+民主法治两支柱+制度风格节）；qttech/qtalliance/qtacademy 已有 structure + institution + culture 四件套，qtacademy 的 institution/structure 为**目录**）；`apps/qtorg`（公开展示站，见下文专节）
- `references/repo-map.md` 之外的两个高频参照（本 SKILL.md 内联速览）：① **data/profile 口径分型**（2026-08-29）：产品档案（quanttide-product / quanttide-human，一个文件夹一个产品，index/requirement/evaluation）vs 组织档案（quanttide-org/profile，一个文件夹一个组织，README 承载名单）vs 交互设计档案（quanttide-design，记理想不记现状）——动手前先确认目标 profile 属于哪型；② **组织名单**（org/profile README）：qttech/qthold/qtalliance/qtfounder/qtinstitute/qtacademy
- 详见 `references/repo-map.md`
- delib 议事领域（契约 v0.0.7、七节点/四实体/两类模式、决议可见性、开发分期）速览：`references/quanttide-delib-architecture.md`
- **lark-cli 读邮箱申请 + Bitable 问卷（2026-09-06）**：课堂/招聘双邮箱 +triage、wiki space→node→base 三连、record 数组列校准（值顺序≠field 顺序，jq 中文键报错，落盘用 python）：`references/lark-mail-bitable.md`（**第七节为申请处理操作链**：已交问卷直接进群/未交先填再进群、双邮箱对账以发件邮箱为键、进群 API 走不通时用 im chats link 取 share_link 发邀请邮件、record-batch-create 的 json 形状无 "fields" 包裹层且文件须 cwd 相对路径、问卷自填不入档需对账建记录。**2026-09-07 升级**：流程已从 examples 脚本升级为 src 包 `qtclass-private/src/qtclass_private/`（`qtclass-ops` 命令，config/lark/applicant/cli 四模块），模板单一事实源在 `data/default/profile/`（invite-mail.html + group-qrcode.png，二维码 7 天过期）；applink 对外部邮箱用户无效必须附二维码；未交问卷时必须随邮件发问卷链接（`--survey-url`，不给则脚本拒发）；发送后 sleep 15s 再查 send_status（4=送达/3=失败）；**用户手动补充的任何遗漏物（二维码/链接）都是模板缺口信号——当场固化进模板与脚本**）

## 资产聚合仓库模式（assets/ 下的容器仓库）

`assets/quanttide-journal`、`assets/quanttide-intention` 是**聚合容器**：内部用子模块按业务实体组织，仓库本身只放导航文档 + `.gitmodules`。

- **结构**（用户明确纠正过）：子仓库放在容器**内部的 `assets/` 文件夹**下，公司实体叫 `default/company`，其他按领域名命名（data/pay/human）：
  ```
  quanttide-intention/
  ├── assets/
  │   ├── default/company → quanttide-intention-of-business-entity
  │   ├── data/           → quanttide-intention-of-data-engineering
  │   ├── pay/            → quanttide-intention-of-payment-engineering
  │   └── human/          → quanttide-intention-of-human-resources
  └── README.md / docs/ / AGENTS.md / CHANGELOG.md ...
  ```
- **新建流程**：① `gh repo create quanttide/<name> --public` → ② 本地 `git init -b main` + 骨架（README/CHANGELOG/CONTRIBUTING/AGENTS/LICENSE/ROADMAP/docs）→ ③ 提交推送 → ④ `git submodule add <url> assets/<path>` 挂子仓库 → ⑤ 父仓库 `git submodule add` 注册容器 → 指针逐级推送
- **移动子模块路径**：`git mv <path> assets/<path>` 会**自动更新 .gitmodules**（git 识别 100% rename，gitlink 无损）；之后更新 README/docs 引用
- 参考模板：`assets/quanttide-platform`（独立资产仓库，apps/packages/manifests/docs 结构 + Apache 2.0 LICENSE）

## 双轴架构（领域轴 × 平台轴，同一应用仓库双挂载）

正交分解（根仓库 AGENTS.md）：**平台能力轴**（How it runs）与**领域知识轴**（What it expresses）分离。一个云应用（qtcloud-econ、qtcloud-delib 等）**同一仓库**同时挂载两侧：

```
qtcloud-econ（GitHub 单仓库）
  ├── domains/quanttide-econ/apps/qtcloud-econ      # 领域轴（业务表达）
  └── assets/quanttide-platform/apps/qtcloud-econ   # 平台轴（能力承载）
```

- 两侧各自 `git submodule add <同一 url>`，gitlink 可不同步前进（各自 update 指针）
- 应用内容（src/、docs/）在应用仓库内开发，两侧挂载点只追踪指针
- 用户说"在领域轴操作"= 在领域侧挂载点的应用仓库内开发（内容提交到 qtcloud-econ 本体，两侧指针都受益）
- 命名：应用仓库 `qtcloud-<domain>`，领域仓库 `quanttide-<domain>`（如 quanttide-econ）
- 领域仓库结构：`apps/<app>`（挂应用）/ `data/{journal,profile,brochure,context}` / `docs/essay` / `examples/default`

## Flutter 客户端项目（qtcloud-*/src/studio）

初始化、验证、CI/IaC 发布通道见 `references/flutter-studio-init.md`（手工骨架 + `flutter create .` 补全、analyze/test/build 三绿、SDK 断点续传坑、组件/页面测试分层、deploy-studio.yml CI + Terraform IaC——**仅本地运行脚本不算 IaC**）。

**界面数据（seed JSON）由模拟数据换成真实案例**（用户「使用这个最新案例替换 qtdata 当前的模拟数据」）见 `references/studio-seed-data-swap.md`：先枚举测试的内容耦合点（`grep -rn "seed" lib test` + grep 旧案例字面量，顺带确认 **lib 里无硬编码案例内容**）；**让交付物数／阶段数／项数这类结构数值保持不变**，把改动压到文案层（断言条数不变 = 行为没被削弱的可比信号）；未知商务数字填示例值并在 `pricingNote`／`_comment` 两处注明非实际报价 + 落一条带可跑判据的 TODO；**分支推送只跑门禁，线上要发 `studio/*` tag 才更新**（报告里必须点明，否则用户以为推完就换了）。

CI/IaC 部署完整模式（四端验证、terraform -input=false/-target 绕过共享资源、ossutil v3、acme.sh 单域名证书、CDN 源站一致性）见 `references/ci-iac-deployment.md`。

## qtcloud 应用双端结构（src/studio Flutter + src/cli Rust CLI）

### 双 app 角色区分（qtcrowd 参与端 vs qtcloud-crowd 管理端，2026-08-26 实证）

`qtcloud-<domain>` 应用仓库内两端并存，**共享同一份种子数据**（单一事实源，在 `src/studio/assets/data/`）：

- **初始化/设计新端前先确认使用者角色（2026-08-26 qtcrowd 三次纠正实证）**：同一 app 的 studio 可能是**管理方端**（qtcloud-crowd studio=审核/认证/结算）也可能是**参与方端**（qtcrowd studio=认领/交付/看结算）——取决于这个 app 是给谁用的。曾用"通用管理后台三件套"模式猜 qtcrowd studio=管理端，被用户三次纠正（site 是展示众包信息非功能介绍、studio 是参与人员端）。**接手某端前先问/确认"这个端给谁用、解决什么"**——不要用市场常见模式（管理后台/介绍页）替代真实角色；site 与 studio 角色独立判断（site 可以是介绍页也可以是信息展示页，studio 可以是管理端也可以是参与端）

### OSS 共享数据层架构（前后台分离 + 前台自有存储，2026-08-26 qtcrowd/qtcloud-crowd 终稿）

**场景**：一个业务两个 app（参与方端 + 管理方端）共用一份任务数据时的架构——**终稿（用户图纸）**：后台不建任何公开桶（公开层不在后台图纸上），**前台 provider 是前台唯一数据入口**：

```
管理方（qtcloud-crowd provider，私有桶 data：审核/认证/结算）——只响应前台调用
    ▲ 上架：qtcrowd-provider 调后台 API（GET /api/tasks?status=published——后台加 status 过滤）
qtcrowd-provider（前台唯一服务端，自己的桶 qtcrowd-provider）
    ├── 上架：拉后台 published 任务 → 写自己桶（public/tasks/{id}.json 黄页快照；周期同步 5m + 手动触发）
    ├── 数据 API：GET /api/tasks 从自己桶读——site/studio 读+写全经它（**不直读 OSS/CDN**）
    └── 写操作：claim/deliver 转发后台（4xx/5xx 透传、不可达 502）
site/studio → 只认 qtcrowd-provider（纯客户端）
```

- **依赖方向硬约束（用户明示）**：**前台可以依赖后台，反之不行**——后台零投递、不建公开桶、不知道前台存在；上架/写回都是前台主动调后台
- **演进决策链（用户逐步纠正，勿回退）**：推式（后台调前台 API=后台依赖前台 ✗）→ 拉式（方向对但丢"推"语义 ✗）→ Outbox/事件总线（两全但重 ✗）→ OSS 共享桶（后台建公开桶+前台直读 CDN——**被否**："后台不应该直接往 site 里提交，而是应该给前台 provider 的对象存储提交"、"4 不保留，不在我的设计图纸中"）→ **终稿：前台自有存储**（qtcrowd-provider 调后台 API 上架 + 数据存自己的桶 + 经 API 给 site/studio；"只删除不新建"——误建的 qtcloud-crowd-public/qtcrowd-public 桶全删，后台 terraform 移除公开桶资源 + `terraform state rm` 同步）
- **"注册"语义**：后台写自己的数据、前台 provider 上架（拉取+落自己桶）= 前台侧注册/发现——桶归属前台（qtcrowd-provider 桶新建），site/studio 不再直读任何公开桶
- **多租户扩展预留（设计预留，代码不做）**：路径/目录/API 结构不写死单租户——公开层 `public/{tenant}/tasks/`、私有层 `data/{tenant}/crowd.json`、API `/api/{tenant}/...`——新租户市场 = 新前缀 + 新前台实例（前台 provider 带 `QTCLOUD_CROWD_TENANT` 租户上下文）
- **实施坑**：qtcrowd-provider 的 `QTCLOUD_CROWD_BACKEND_API` 必须指向后台 FC URL（`ListTriggers` 的 urlInternet——`https://<fn>-<rand>.cn-hangzhou.fcapp.run`），不是 api.example.com 网关（无此路由 404 → 上架同步拉不到任务、数据 API 返回空）；后台 /health 404 属正常（无该路由）但 `/api/tasks?status=published` 必须有
- 数据流细节：Task 状态机 `pending→reviewing→published→accepted→done`（published=审核通过可接）；后台冗余的 publish 投递代码（internal/publish/ + store PutPublic/DeletePublic）清理掉——published 状态保留（上架查询用）
- `src/studio/`：Flutter 客户端（见上节）
- `src/cli/`：Rust CLI（如 qtcloud-econ-cli，命令名 `qtcloud-econ`，`mechanism list/show` 子命令）
- **坑**：CLI 的 `default_path()` 必须写仓库根相对 `src/studio/assets/data/<file>.json`（写成 `assets/...` 会 No such file）
- 完整初始化配方（Cargo.toml lib+bin、clap 子命令模式、共享路径、cargo test 多段输出）：`references/qtcloud-rust-cli-init.md`

## qtorg 公开展示站（React/Vite SPA，非 Flutter studio）

`domains/quanttide-org/apps/qtorg/src/site`——量潮组织中心官网（org.example.com），**React + Vite SPA**。

- **架构（v0.2.0 改组，2026-08-29）**：**人物是一等实体、数据驱动渲染**——`src/data/people.ts` 是人物/组织单一事实源（`Person = { id: 拼音slug, name, primary主身份, titles: [{org, title, desc?}], order排序权重 }`，手工镜像 profile 各组织 title.md）；页面从数据渲染。路由：`/orgs/<orgId>`（组织页）+ `/people`（人物总览，卡片墙+组织徽章）+ `/people/:personId`（人物详情=职务按组织分组）；旧路由 `/qttech` 等用 `<Navigate>` 重定向。一级导航 = 首页/联盟/公司/实训基地/**人物**（用户拍板"人物"——档案语汇、覆盖跨组织身份；"人员"太 HR、"成员"暗示单组织从属）
- **内容同步模式**：① **人事/职务变更**（title.md 增删人员）→ 只改 `people.ts`，组织页管理/顾问/理事会/秘书处区块自动渲染（qttech 按 title 含"顾问"过滤分两块、qtalliance 按"理事"/"秘书长"过滤）；② **组织结构/部门/职级变更** → 组织页 OrgTree/DepartmentCard/RankTable 仍是各页内联数据，手动同步对应 TSX；治理架构卡（两院/三机构）与制度区块不随人物变更动
- **验证**：改完 `npm run build`（tsc -b && vite build）+ `npm run lint`；本地 SPA 深链验证用 `vite preview`（**必须 terminal background=true**——前台 `&` 会被工具拒）后 curl 各路由；发布后线上验证两步——`curl -s https://org.example.com/` 取新 asset hash（应与本地构建一致），再 curl 该 JS `grep -c "新内容关键词"`
- **发布**：qtorg 仓库直接打 `site/vX.Y.Z` tag 推送即触发 deploy-site.yml（Terraform apply + npm build + ossutil 上传 OSS qtorg-site + 刷 CDN）——版本条目在 **scope 文件 `src/site/CHANGELOG.md`**（根 CHANGELOG 只做索引），`package.json` version 同步 bump。**发布语义实证（v0.1.3/v0.2.0 两例）**：用户说"更新 qtorg 的 site"/改组方案确认"开始" = 编辑+tag+部署一体授权（tag 是部署机制不是独立发布动作），无需单独请示版本号——与 qtcloud-devops publish 的授权纪律不同；功能级改组用 minor（0.1→0.2），内容增补用 patch。**已部署 tag 不可重打（2026-08-29 被拦截教训）**：v0.2.0 上线后又有改动 → 追加 v0.2.1 新 tag，绝不 `git tag -d` + 删远端 tag 重打同一版本号（覆盖线上版本历史=破坏性操作）
- **站点与档案解耦（2026-08-29 用户拍板，覆盖此前"镜像/对齐"模式；同日再次确认）**：用户明示"站点删除所有引用。不需要引用"——**站点是独立公开展示，内容自主维护；档案是组织内部事实源，两者互不引用、互不依赖**。已删除：people.ts 注释中的"title.md 手工镜像"字样、docs/{company,alliance,training-base}.md 的"数据来源"节（`data/profile/...` 路径）。**后续禁令**：① 不在站点代码/docs 里引用档案仓库路径或"镜像/同步自档案"表述；② 不再发起"档案↔站点对齐/双向核对"工作流（旧模式作废）；③ 用户报人事/职务变更时直接改 people.ts，无需（也不要）声称与档案同步。改动归属判断：站点内容变更 = 站点自主决策；档案结构变更 = 档案自主决策，互不触发对方修改。**"组织档案和网站对齐"语境已过时**——用户随后对 profile 做 orgs/+people/ 分组重构并明示"暂时不动网站"，档案与站点自此各走各的，不要在档案变更后主动提议同步站点
- **TSX 相对路径坑（v0.2.0 实证）**：`src/pages/<org>/index.tsx`（pages 一级子目录）import 要 **`../../components`**（上两级到 src/）；`src/pages/People.tsx`（pages 直下）用 `../components`——层级混用后批量替换极易搞混，诊断用 `npx tsc --noEmit --traceResolution` 看"Resolving module X from Y"实际解析路径，比猜快
- **幻影 TS 错误坑**：修复 import 后 TS2307/TS7006 仍反复报 = **tsbuildinfo 缓存陈旧**——`rm -f node_modules/.tmp/*.tsbuildinfo && npx tsc -b --force`；先清缓存再排查代码，别对着幽灵错误改代码
- SPA 深链用"列表 key 模式"（上传无扩展名 `/org` key），不依赖 CDN 回写规则
- 完整参考（数据结构/路由表/页面组件/验证发布命令）：`references/qtorg-site.md`

## qtclass-private 域仓（2026-09-07~10 新建，课堂第二大脑工程仓）

`quanttide-tech-private/data/qtclass-private`（独立私有仓）——从"纯文档分析仓"演进为"工程+运营仓"。结构：`ROADMAP.md`（核心判断：**流程破碎是依赖人的根因**——一个学员旅程横跨十个系统靠肉身缝合；演进=筑河床，P0 图纸→P1 最小连续流程→P2 拆缝合点→P3 制度化）+ `docs/handbook/enrollment-state-machine.md`（**流程唯一定义处**：11 状态/14 迁移/双通道合流/字段映射/硬约束；第八节=演进方法四零件——状态/迁移/触发器/副作用，取消问卷/取消邮件两情景已预演）+ `data/default/profile/`（**运营资产单一事实源**：invite-mail.html/group-qrcode.png/SURVEY_URL，src 引用不复制）+ `src/qtclass_private/`（qtclass-ops CLI + gui/ 双原型）+ `data/profile/iGuo/default.md`（材料暂存区）。**关键教训**：① 整理流程产物先入 default.md 备用（见领地纪律）；② 模板缺口信号——用户手动补充的遗漏物（二维码/链接）当场固化进模板与脚本；③ AGENTS.md 放仓库根不放 src/ 下；④ 用户说"GUI 分两个，一个用户一个管理员"= 对应 ROADMAP 目标形态两端的原型（user=工作台学员侧/admin=课程云学习云管理侧），GUI 是信息架构验证器，工作台 Web 化后废弃（落地结构：`src/qtclass_private/gui/{__init__,user,admin}.py` + pyproject 入口 `qtclass-user`/`qtclass-admin`，与 `qtclass-ops` 共用 applicant 业务流；user 原型提交只建档+给指引，**不自动发邀请邮件**——进群邀请仍由 admin 确认后发；问卷链接落 config.py 的 `SURVEY_URL`；`lark.py` 补 `list_records` 供学员查进度）；⑤ **文档方向性重写**（"重新梳理文档按这个方向写"）= ROADMAP 核心判断/README 定位/AGENTS 纪律全部按新根因重写，非补丁追加。⑥ **方向偏离复盘（GUI 一课的通用教训）**：用户要求"src 升级为 CLI+GUI 两种模式"时我直接做了——但那个 GUI 优化的是"运营者逐条处理申请"，而这个环节在 ROADMAP 目标形态里大部分消失（数据自动流转、报名入口前置）。偏离三层原因：**模式类比移植**（创始人实验室 lab/lab-gui 的成熟模式被顺手搬来，未从本域目标形态推导）、**痛点解法错层级**（处理端效率 vs 入口端存在方式）、**ROADMAP 缺过渡期建设规则**（标的"过渡形态"没写过渡期允许投入什么）。修正动作 = ROADMAP 补**过渡期投入分界表**（业务流/数据契约/资产 = 可平移到工作台 ✅ 照常维护；入口壳 CLI/GUI = 一次性 ❌ **冻结：不新增功能，只修运行错误**）+ 退出条件（工作台稳定运行两周无人工兜底即过渡期结束，GUI 废弃）。**通用动作：用户要求加某种形态（GUI/CLI/脚本/站点）时，先对照该仓 ROADMAP 判断它落在可平移层还是入口形态层——落在后者要先把方向问题提出来再动手。**

## 工作流程
1. **先读目标仓库的 AGENTS.md** —— 每个仓库的 AGENTS.md 是权威规范（skill 索引、提交规范、特殊提醒），内容各不相同
2. **子模块操作前**：`git checkout main && git pull`（AGENTS.md 明示；journal/intention 都在 detached HEAD 状态被打开过）
3. **分离头检查**（本环境常见，勿恐慌）：
   - `git merge-base --is-ancestor <detached> origin/main` 判断是否在主线历史
   - `git diff --numstat origin/main <detached>` 统计：**0 新增 = 纯子集 = 无本地独有内容，可安全丢弃**
   - **提交已落在分离头上**（push 报"不在分支上"）：`git checkout main && git merge --ff-only <sha>`（该提交是 main 直接后代时），再 push
   - **假成功变体（2026-09-29 实证，比报错更隐蔽）**：分离头上 commit 后 `git push origin main` 报 **"Everything up-to-date"**——main 分支自己没动（推的是那个没变的 main），远端**静默落后一个提交**。`git branch -vv` 会显示 `*（头指针自 <sha> 分离）` 而 `main` 停在旧 sha。修法：`git branch -f main <新sha> && git checkout main && git push origin main`。**推完必须核实远端**：`git log --oneline origin/main -1` 或 `git ls-remote origin main`——"Everything up-to-date" 不是成功信号，它就是失败陈述。本仓群子模块常落分离头，这条每轮提交后都该跑一遍
   - **分离头提交的通用抢救路径**（2026-08-22 实证三次：qtcloud-think/roadmap/journal）：`git checkout main && git pull origin main && git cherry-pick <detached-sha> && git push`——当 detached 提交不在 main 直接后代（或 main 已前进）时 cherry-pick 比 merge --ff-only 可靠；cherry-pick 成功后 `git status -sb` 确认无"领先"残留
4. **修改子模块内容** → 子模块内提交并推送（Conventional Commits）
5. **回父仓库更新指针**：`git add <submodule> && git commit -m "chore: update <name> submodule" && git push`
5a. **推送顺序纪律——子模块推不上去时父指针绝不能先落地（2026-10-01 context README 坏窗口事故）**
   - **正确顺序**：子模块提交 → 推子模块 → **核实远端**（`git log --oneline origin/main -1`）→ 才回父仓库 `git add` + 提交 + 推。父指针记录的 sha 必须已在子模块远端存在
   - **事故形态**：子模块 `push` 被拒（非快进）却接着推了父指针 → 父仓库指向一个**远端不存在的提交**，修正前一直是坏窗口。**子模块推送失败 = 立即停下并如实报告，不推父指针**
   - **非快进（远端在我 pull 之后又前进一格，另一 agent 也在推）**：`git fetch origin` → `git rebase origin/main` → `git push`。**rebase 会改 sha，先前提交的父指针随即作废**——必须回父仓库再提一次指针（`git add <path>` 会显示 `旧sha..新sha`），不能以为指针对过了就不管
   - **收尾核实（三级一致，四个值两两相等）**：子模块 `git rev-parse --short HEAD` == 子模块 `git rev-parse --short origin/main` == 父仓库 `git ls-tree HEAD <path>` 取到的 sha == 根仓库 `git ls-tree HEAD <父仓路径>` 取到的 sha
5b. **子模块退役（取消挂载）实证（2026-09-29 quanttide-project-toolkit）**——用户只说「<包名> 退役」时，动作 = 取消挂载 + 三处文档同步：
   - **先查依赖**：`grep -rn "<包名>" --include="pubspec.yaml" --include="pyproject.toml" --include="*.yaml" apps/ packages/`。消费方若走**发布源**（qtconsult 的 `quanttide_project: ^0.2.0` 来自 pub.dev）而非 path 依赖，退役不破坏构建——先确认这条再动手，并在汇报里点明
   - **执行**：`git submodule deinit -f <path>` → `git rm -f <path>` → `rm -rf .git/modules/<path>`（`deinit` 不加 `-f` 会因目录有内容拒绝）
   - **核实**（三条都跑）：`git ls-files -s | awk '$1=="160000"' | wc -l`（虚链接计数应有减）、`grep -c <name> .gitmodules`（0）、`ls <父目录>/`
   - **三处文档同步**：README 记入「已退役」清单（该仓先例 = 「已取消挂载，独立维护」一组：qtmedia/qtcrowd/qtrecurit；退役走独立那段并写明「本地目录已清空」）；STATUS 的子模块计数（`apps 6 + data 11 + docs 6 + examples 1 + packages N` 要对得上）+ 快照日期；CHANGELOG `[Unreleased] / ### 移除`
   - **先翻 CHANGELOG/STATUS 再动手**：本仓出现过「当天加入、当天退役」（0.7.6 新增 project-toolkit，同日退役）——先看一眼能避免误判
   - **远端归档是独立动作**：退役挂载 ≠ 归档仓库（`gh repo view <repo> --json isArchived`），要单独问用户，别自己顺手做
   - **README 表漂移顺带修**：README 子模块表可能漏挂载项、STATUS 计数可能是旧的——用 `git ls-files -s | awk '$1=="160000"' | wc -l` 对数（实证：README/STATUS 都写 25、实际 26，漏了当时刚挂上的 `packages/quanttide-project-toolkit`）
6. **新增文档时**：检查并更新该仓库 myst.yml 的 toc 注册（曾发现 `qtadmin/asset.md`、`qtcloud-support.md` 漏注册）
7. **提交即推送**：默认推送，除非用户明确说"只提交不推"。**"not push!"变体（2026-09-01 实证）**：用户中途喊"not push!"时，先如实对账——若部分提交已被推送（默认推送纪律跑过了头），坦白已推送状态并给撤回选项（force push 回退 vs 保留），不假装没推；随后严格改为"提交不推、推送等明示"，并确认是仅本批还是今后所有仓库的新默认（用户确认过的默认要记）。反向案例：用户喊"not push!"后紧接着指出**另一仓库**"没有推送！补上！"——含义是该推的没推，不是全局禁推；推送纪律以仓库/批次为单位听当次指令，不是一刀切
8. **同步资产仓库时顺带查兄弟仓库（用户"还有日志呢"纠正，2026-08-15）**：用户说"X 未更新"（如 fiction）时，同步 X 后也要检查同族资产（memory/journal 等）——用户预期是全库检查，不是单仓库。检查 journal（如 memory/journal/default/YYYY-MM-DD.md）看作者是否留了创作/决策日志（常含下一步方向，如 08-14 日志的"情绪 Agent=思考云原型"）；有远端更新先 fetch 对比再 pull
9. **用 journal/roadmap/intention 更新产品档案（profile）**：产品档案信息源除 `data/journal/<product>/YYYY-MM-DD.md`（创始人日志常含产品思路/需求/决策，如 qtcloud-agent 的"智能体不是从产生对话开始，而是从分析对话开始"）外，还有 roadmap（实训基地公司级全景已迁至 `domains/quanttide-org/data/intention/training-base.md`——roadmap 仓库只保留 qtclass/qtrecurit 领域侧）与 intention 层——建档案前先 grep 这三层找已有表述，按五维度（定位/核心目标/阶段方法/数据驱动/需求与AI）组织，不凭空写。流程：① 读信息源 → ② 提炼产品定位/目标/阶段方法 → ③ 创建/更新 `data/profile/<product>/index.md`（产品档案）+ `requirement.md`（用户故事地图，见用户故事地图标准）+ 可选 `evaluation.md`（评估档案，见 9e）→ ④ 子模块提交推送 → ⑤ 父仓库指针。**档案可先于应用存在（2026-08-29 实证 qtacademy）**：profile 的 AGENTS.md 约定"目录名与 apps/ 子模块名一致"，但产品化尚在路线图阶段的产品（如实训基地 qtacademy）可先建档案目录（预留架构声明），汇报时说明边界即可，不必等应用仓库立项
9b. **dsh 工作区对话 + 视觉模型分析网页 UI**（dsh headless 在领域根目录跑 → 结果沉淀 data/context；截图+deepseek-v4-flash-vision-exp 分析 UI/UX；交互类归 quanttide-design context）：完整命令与坑见 `references/dsh-vision-analysis.md`（含 dsh 升级、视觉模型确认、临时切 settings.yaml 必须恢复）
9c. **执行档案（quanttide-execute data/profile）——业务×职能 + 脱敏公开（2026-08-22）**：从私有工作报告（qtadmin-execute assets/工作报告/简报Brief，按部门/周）提取事项 → 按**业务文件夹（qtdata/qtclass/qtcloud）× 职能文件（business/operation/product.md）**组织公开档案；每业务 `index.md` 展开（定位一句话/重点/当前状态/职能文件索引）；根 `index.md` 记**整体方向**（把人的能力系统性转译成系统的能力——治理法治化/人才系统化/产品工厂化/事实源制度化/经验资产化五环节）+ 各业务重点。**脱敏公开原则**：产品/技术事实可公开（平台本身开源——版本/上线/阻塞直接写），**人事（候选人人数/个体/成员分工）/客户名与合作状态/治理决策（权力归属）必须脱敏或留在私有仓库**（如"某客户老师"→"某客户项目"、"决策权给某人"→"执行/决策/提案/监督分层"）。私有侧（qtadmin-private/docs/profile）保留完整人事/客户/治理维度；公开侧只留产品事实。**档案职责：profile 记稳定记录不记任务清单**（tasks/ 曾建后被用户否——动态任务归 roadmap/journal，完成即归档）
9d. **journal 主题文件夹合并**（2026-08-22 execution→default）：同日期文件**内容拼接**（`echo -e "\n---\n## 执行\n" >> default/<date>.md && cat execution/<date>.md >> ...`，用 `## 执行` 小节分隔保留时间流），不同日期直接 `git mv`；合并后 `git rm -r` 空文件夹。**注意子模块检出常落分离头**——提交前先 checkout main，提交后立即 push 验证"领先"清零。**执行领域 journal 按业务分文件夹（2026-08-23）**：`data/journal/{qtclass,qtcloud,qtdata}/YYYY-MM-DD.md`（业务 × 日期）+ README 说明结构——与 profile 档案同构（同业务文件夹），日志记事实、档案记状态，成为执行云双视图（journal/profile + AI 提炼管道）的产品原型；当日日志日期用**今天**（曾写 08-22 被用户纠正"今天 23"）
9e. **跨仓库同步评估档案到 profile（2026-08-29 实证）**：用户说"把 <repo> 的 evaluation 文档同步给 <product> 的 profile"= **纯复制**（源文件保留不动，非迁移，不能也不需要 git mv）。源常是实验室产物（实证：quanttide-human examples/default/BUSINESS_EVALUATION.md，招聘原型业务评估 8 维度+三阶段清单），目标 `quanttide-product/data/profile/<product>/evaluation.md`（目录不存在直接 mkdir）。**evaluation.md 是 profile 的标准档案类型（2026-08-29 定型）**——profile AGENTS.md 目录结构已列 `evaluation.md # 评估档案：业务目标验证（可选）` + 独立定义段（按业务目标分解评估维度→每维度评估问题+验证方法→三阶段检查清单+结论判定收口；验证先行，可与需求档案同步建立）。同步首个新档案时若目标仓库 AGENTS.md 未列该类型，顺带补约定 + CHANGELOG Unreleased 条目（一次提交完成，实证 19244f1）。用户引用仓库用裸名（"quanttide-human"），实际路径在 domains/ 下——先 find 三级路径（domains/default/assets）解析再操作，勿按裸名直接拼路径。提交链三层：profile 子模块 → quanttide-product → 根仓库（常规分层提交）。**父仓库可能带上次会话遗留的子模块指针未提交更新（git status 显示 M <submodule> 但子模块内干净）**——提交自己的指针变更会一并带入，先 git diff 确认指向远端已存在的提交即可顺路提交。**但脏指针成片时不顺路带走**（2026-09-24 实证：根仓同时挂着 10 个 unrelated 域的脏指针——那是别轮没收尾的活）：`git add <自己那一个路径>` 只暂存自己的，提交前 `git diff --cached --name-only` 自查暂存区**只有自己的路径**，其余逐个报给用户等点名。把无关指针扫进文档提交 = 把别人的变更混进同一提交，违原子提交
9a. **交互设计档案（quanttide-design data/profile）——记理想不记现状（用户 3 次纠正沉淀，2026-08-21）**：档案回答"这个产品的交互应该是什么样"，**不是**"现在长什么样"。结构 = 产品一级文件夹 + 仅 `index.md`（如 `qtcloud-product/index.md`）。内容 = 理想的交互设计体验（按用户感受维度组织：打开时/地图时/操作时/决策时/结束时 的理想状态）+ 该产品的体验原则。方向演进史（教训）：先写组件状态矩阵+用户流程（被否"patterns 太抽象，不是标准的交互设计信息"）→ 改标准交互设计文档 components.md/flows.md（仍被否"这个方向不是我想要的"）→ 最终 = 理想体验。写交互设计档案前先确认用户要"理想体验"还是"现状记录"，不要默认采集现状；同源分析（context 的 ui-ux-analysis）是素材，不是档案本体。**设计不发明转译层——直接用原始概念（用户纠正 2026-08-22："为什么不直接用原始概念"）：qtcloud-think studio 导航直接采用 4D 认知过程本身（感知→理解→预测→决策），不转译成"收集/澄清/结构"**
9f. **组织档案体系（quanttide-org data/profile，2026-08-29 用户定型；同日两轮重构至终稿「组织+人物」两组）**：与产品档案（9/9c）不同类。**终稿结构（用户拍板"分为组织和人物两组"选彻底方案，全部 git mv 迁移）**：`orgs/<org-name>/`（六个组织目录 qttech/qthold/qtalliance/qtfounder/qtinstitute/qtacademy）+ `people/<person-slug>/index.md`（**一个文件夹一个人物**——用户中途纠正"people/<person-slug>/index.md 这样"，从平铺文件改为目录式，与组织侧结构同构；人物一档 = 姓名 + 职务表[组织×头衔] + 角色定位 + 关联，目录名拼音 slug 对应站点 /people/<id>）。人物是**一等实体**——跨组织身份（创始人=公司CEO+联盟理事长）在人物档一处维护；**职务单一事实源仍在各组织 title.md**（people/index.md 明示"引用不复制，职务变动两处同步"）。配套文档同轮完成：根 README.md 改写为两组结构说明（orgs/ 表 + "人物名单见 people/index.md"）、根 index.md 档案节改双指针、profile AGENTS.md 重写为两组规范、CHANGELOG 补齐全部历史（原只有 init 一行）。注意 profile AGENTS.md 的信息披露规则（公开=架构框架/治理机制/职级框架，不公开=具体人员岗位/薪资/流程细节）与 people/ 存在张力——人物档目前只写职务不写岗位细节，扩写前先核对此规则：qthold/qtfounder 的"创始人→法人演进"来自 `default/quanttide-tech/data/profile/brand/founder.md`（品牌档案），qtalliance 来自 intention/qtalliance，qttech 来自 roadmap index 战略定位，**组织结构来自章程层** `assets/quanttide-bylaw/default/company`（秘书章程=秘书处三层架构、议事机构章程=创始人-上议院-下议院+书记处/执委会/技委会、职级章程=五序列、公司代表章程）——建组织档案前先 grep 品牌/意图/roadmap/章程四层找已有表述。**structure.md 组织结构档案 + index/structure 分离（2026-08-29 三实例定型）**：需要描述部门设置与架构时建 `<org>/structure.md`（README 目录结构已列，当前有 qtacademy、qttech、qtalliance）；profile 根 `index.md` = **组织模式内核总纲**（2026-08-29 拍板重写：内核 = "把依赖'在场'的价值固化为独立于人的记录与结构"；两支柱 = **法治**（记录与规则即权威）+ **民主**（合法性来自参与、记录托起参与——选举资格是记录的函数），创始人一票否决 = 受程序约束的"在场"兜底；三个剖面 = 公司知识提炼链 / 基地贡献记账 / 联盟介绍即验证；检验判据 = 这件事是否依赖某个具体的人在"场"；文化与产品同构=自我验证）——根 index=模式总纲、README=名单、组织档案=事实记录，三层分工；**index.md 记组织是什么**（定位/核心目标/组织形态一句话 + 指针"详见 structure.md"），**structure.md 记组织怎么架构**（逐机构展开）。实例形态：qttech = 治理层（股东代表大会/上议院 + 公司代表大会/下议院两院制，创始人一票否决兜底——源自 bylaw 议事机构章程）+ 运营层（秘书处/事业部/共享中心）+ 矩阵式；qtalliance = 联盟理事会（决策）/联盟代表大会（议事）/联盟秘书处（执行），决策·议事·执行分离；qtacademy = **内包子公司模型 + 传统职能部门编组（终稿，用户三轮修正拍板 2026-08-29）**——实训基地秘书处由公司派驻代表组成（母公司管理接口：治理/考核/转正评审），学员按**传统职能部门**编进部门：**技术部/产品部/市场部/职能部**（公司结构的简化版）。**编组方案演进链（勿回退）**：用户问"复用还是预备部门"→ AI 提议"动态任务编组不设部门"被否（"我们需要把人编进部门里，这是必须的"）→ AI 提议"按公司业务线对接面设数据部/课堂部"被否（"按照传统的职能部门编组，比如技术、产品、市场、职能"）→ 终稿 = 内包子公司 + 传统四部门 + 承接任务时跨部门抽人组队、任务完回部门（部门管训练与归属，项目管交付）；**公司组织结构复杂，实训基地做简化版**（无两院/无职级序列/无共享中心，职能直接用公司的，只留交付最小集）；关系链 = 事业部发包→基地部门承接内包→学员真实任务中训练筛选→转正进公司编制（招聘是出口，众包任务池是另一任务来源）。**institution.md 与 culture.md 档案类型（2026-08-29 三组织同建定型）**：org 档案四件套 = index（组织是什么）/ structure（怎么架构）/ **institution（依什么规则）**/ **culture（什么气质）**。institution.md = 制度地图：制度体系对照表 + 核心约定 + 与组织结构的关系；**章程未立项的组织（qtalliance/qtacademy）如实写"已定框架 + 待立制度清单"**——比编造完整制度诚实，待立清单即未来立项议程；有章程原文的组织（qttech）逐条对应 bylaw 出处。culture.md = 文化气质：底色 → 工作气质 → 精神传统；**子组织文化显式写与母文化的传承关系**（联盟 = 公司文化的组织间版本，基地 = 训练版本——同一价值在组织间/训练场的表达变体）。**民主参与是 culture 的必备维度（2026-08-29 补齐）**：culture.md 要写民主的体感——qttech 有"参与的日常"（下议院周会/真选举/否决公开），qtacademy 有"参与是挣来的"（状态阶梯/自治预演/评审即参与）；**缺席推断禁令见下方文化一致性讨论实证**。**文化一致性讨论实证（2026-08-29，勿重蹈）**：用户问"文化的一致性体现在哪里"→ AI 给四层面分析被评"内核的理解还是不够透彻" → AI 提炼"外化在场"内核获认可，用户补"也就是民主和法治" → AI 用"文档没写=文化缺失"论证公司民主文化薄/基地民主被讲丢，被纠正"**不对。这些只是没有体现在现有资料里**"。**教训：① 从"缺席"推断"不存在"违反查证纪律——资料没写 ≠ 现实没有，制度/实践层已有的东西（下议院周会、显性表决、自组队竞争）不因 culture.md 未记录而不存在；分析组织状态前先核对制度层（institution/bylaw）与实践层（roadmap/journal）事实，"文化文档落后于现实"≠"现实缺机制"；② 文化档案补记录时只写实然（制度与实践已有的），不先于制度编造（qtalliance 留白待章程）；③ 用户拍板的民主实践增量（如基地考核评审权让渡给活跃成员）= 实然新增，institution 记框架+待立细则、culture 记体感条目，直接落档不再论证**。**考核评审民主让渡 + 不设回避（2026-08-29 用户拍板）**：基地一部分考核评审权让渡给活跃成员（参与自己和他人的考核评审），**不设回避**——自我管理是量潮核心文化，公开评价自己、为自己的判断上账即训练（每个人都活在自己生成的记录里）；institution 记框架 + 待立细则（活跃度标准/评审范围/评审权重与派驻兜底关系），culture 记「评审即参与」「评审自己，自我管理」条目。**记录操纵口径（用户拍板）**：刷记录/互评合谋是任何组织都存在的老问题，量潮已非常擅长处理——连续记录的长期比对+公开可见+审计兜底；档案写**能力陈述**，不写风险提醒。**根 index.md 总纲现含「制度风格」节**（记录介质驱动生产/评价/治理三循环飞轮：不设回避/制度即课程/诚实未完成/槽位先于人头；信任加光照+熔断位）。**title.md 职务文档**：qttech/structure/title.md（管理团队+顾问团队）、qtalliance/title.md（理事会+秘书处）——人事名单变更（如新增顾问"某某 资深技术顾问"）直接改 title.md，三层提交；**变更后通常紧跟"更新 qtorg 的 site"指令**（公开站 people.ts 数据镜像，见上文 qtorg 节——v0.2.0 后人事变更只改 `src/data/people.ts` 一处，组织页自动渲染；结构/职级变更仍手动同步组织页 TSX）。qtacademy 档案结构已演进：institution 与 structure 均为**目录**（institution/{index,rank,department}.md、structure/{index,department}.md），非单文件——patch 前先 ls 看实际结构。**用户给结构骨架关键词（机构名列表）时**：名称本身即权威来源，从章程/意图四层找制度原文充实各机构职权；找不到原文的按量潮治理惯例组织框架，显性标注"归纳/占位"并提请制度缺口（如共享中心暂无章程原文）。bylaw 的 default/company 子模块常未初始化（`-` 前缀），先 `git submodule update --init` 再读**注意**：profile 的 AGENTS.md 目录结构模板写的仍是产品档案口径（index/requirement/evaluation），组织档案是 README 层面的新约定——两者并存，产品化后才转产品档案口径
10. **用户故事地图标准**（profile 仓库 `.agents/skills/product-requirement/SKILL.md` 是权威，三段论：目标/流程/验收标准）：流程五步=画像（一行"用户画像：需要…的[角色]"）→ 活动（用户旅程阶段+描述段，不是系统模块）→ 任务（一句话概括，不展开）→ 故事（"用户要[动作]，[目的/价值]"展开细节，一条故事=一个可独立交付的用户价值）→ 评审。**任务名必须是动作（动词+名词），不名词化（用户纠正 2026-08-22："任务都应该是动作"——"清晰度判断"改"判断清晰度"、"本体挂接"改"挂接本体"）**。**不标 P0/P1 优先级**（地图刻画需求是什么，版本规划在迭代阶段做）。验收必须满足：每活动有描述段、故事全"用户要…"句式、活动按旅程时间顺序、**主线故事清晰（读完能理解产品解决了什么问题）**、符合目标用户视角。摘要见 `references/user-story-map.md`

9g. **learn 教学 journal（quanttide-learn data/journal，2026-08-30 实证）**：仓库常为空壳（仅一行 README）——动手前先 pull。**结构已重构（2026-08-31 会话外演进，勿按旧路径找）**：journal = 顶层 `{学习者}/YYYY-MM-DD.md`（Jerry/iGuo 平铺，qtclass/qttech 分组已废）；profile = `learners/{昵称}.md` + `schedules/<名>.md`（YAML frontmatter title/description；Task 条目引用 `tasks/<slug>.md`）+ `tasks/<slug>.md`（YAML frontmatter + 任务全文 + 达成条件）——不再按组织分组；specification 已发 v0.1.0（Learner/Completion JSON 契约已由学习云 provider 落码、Schedule/Task 契约设计完成，`schedule-agent-engineer` 智能体工程师训练营为首个真实 Schedule 实例）。整理格式 = 按议题分组（不逐条罗列），保留关键原话；**头部不注来源**（渠道/群名/时间段/参与者均不披露——来源信息本身属敏感面）。**日期口径**：按对话实际发生日期（消息 create_time），非会话日期。**脱敏纪律（首篇日志当天触发两轮 git-filter-repo，代价教训）**：IM 导出的 sender 是真名，**写入前必须替换为代号**（用户指令里的文件夹名即代号）；**第一轮就要清全**：真名/@提及名/群邀请链接（applink/link_token）/个人邮箱/来源渠道痕迹（平台名、工具名、群名、时间段）——普通 commit 删行清不掉历史，删整行内容也要 filter-repo（先 `git show <旧sha>:<文件>` 抽查）；**真名↔代号映射不写入任何仓库文件**，只在会话上下文。**双变体档案**：变体 A=学员档案（验收制，按 Schedule 记 Task 完成状态 + 学习状态 + 教学动作）；变体 B=自助学习者档案（主题制，学习主题**按管理领域归类**——领域/主题/状态三列表：组织管理/资产管理/机密管理/学习管理；无教学动作区）。**顺序纪律（用户拍板）**：先维护日志再维护档案——档案素材必须先经 journal 沉淀并获用户确认；档案不引用日志路径、不写敏感来源。**iGuo 档案教训**：从产出物（gallery 验收、教学对话）反推的内容被否（"记录的信息和我的实际学习不相关，但我也不知道我在学什么"）——创始人的学习证据藏在决策记录（对 AI 方案的否决与纠偏）里，不从产出物反推。**意图/洞察层沉淀**：体系本质 = 探路者（无 schedule，考古学习）→ 学习云（提炼 schedule）→ follower（跟 schedule + 验收），跟随者走完成为新探路者（intention/index.md）；**学习云定位 = 能力账本**（不教做事、不管理任务本身——做事归执行云/业务域，学习云只记录做事产生的学习效果；与执行云分工 = 同一个"做"两本账：执行云任务账/事的角度，学习云能力账/人的角度，例："PR 章程仓库"任务执行云记完成状态、学习云记"会提 PR/熟悉章程仓库"，intention/index.md）；Schedule 核心模型见 insight/schedule-core-model.md（领域模型只有 Schedule+Task，Task 自足零外键，Journal/Profile 是数据视图；Criterion/Lesson 归课程域——**多轮收敛实证：Criterion 并入→Task 自足不引用→下级概念砍到只剩 Task，每轮收敛都来自用户的单句纠正"Criterion 已经不在学习管理领域了""只要 Task""Task 什么都不引用"**）。**自举模式（2026-08-30 用户指令"进行自举"实证）**：探路者把自己当 follower 记账——从当天决策记录与产出物回溯列举自己的学习 Task + 验收标准写进当日日志；验收标准的形态 = 产出物 + 他人认可，不是答卷；元观察（"给自己列验收标准比给学员难——探路者的标准在做事之后回溯定义"）本身入日志。**bylaw 章程写作四轮返工教训（2026-08-31，写章程/制度类文档的通用迭代顺序）**：① 第一稿按角色/记录体系写 20 条 + 现实依据表 → 否"不对。只局限在 specification 规定的范围内，说明质量控制问题"（范围越界：角色体系/意图不归章程，那些归 intention/profile AGENTS）；② 第二稿纯数据质量控制（准入/回退/抽查）→ 否"太技术了。要围绕人类真实的学习管理活动。**先说说你的理解**"——**被否后先给理解版拿确认再动笔**（理解获"对了"才写）；③ 第三稿"社会契约/信任原则"概念化表述 → 否"如何向着'质量控制'的方向改进，**减少虚头巴脑的概念**"。④ **终稿判据再收紧（2026-09-05 第四稿）**：第三稿虽是人的语言仍有概念残留（"社会契约"定位、"异议权/参与确认权/查看权"权利话语、"范式构建者"身份标签、与 specification 的哲学分工论述）——第四稿把概念词全部转成动作（"异议不需要写理由书"→直接提出、"修正留痕"→git 提交写明改了什么为什么、走完承认→"写入档案，对接实训通道"、档案跟人走）。**终稿原则：每条记录都要有证据，每个判定都要能复核**；每条 = 能执行、能违反、能检查的动作（六章十八条：路径协商/先做事后记录/判定只看证据/异议不写理由书/脱敏入档/记录不挪用/档案跟人走）。**章程写作的完整迭代序（通用）**：范围先行（只管 specification 模型的运行质量）→ 理解先行（被否后先交理解版拿"对了"再动笔）→ 去概念化 → 第四稿再砍概念词（权利话语/身份标签/哲学分工全是虚头巴脑的变体——判据：每条都能执行、能违反、能检查，零需要解释的概念）

9h. **客户支持域（quanttide-support，2026-09-01 建成）与领域 profile 边界纪律**：域结构 = 1 主仓 + 20 子仓（apps/qtcloud-support、data/{journal,profile,insight,...}-of-customer-support、docs/{bylaw,handbook,tutorial,...}），apps/qtsupport 官网独立仓库平级挂载（见发布节）。**journal 五要素案例格式（用户定型）**：每案例 = 客户→问题→解答→状态→沉淀，一天一文件按业务线分目录（如 `data/journal/2026-09-01.md`）；"沉淀"字段是 profile 条目的种子，"状态：已解决"是蒸馏触发旗标——journal 格式直接决定 distill 管线能否自动化。**profile 组织（多轮返工后定型）**：按业务线分目录（qtclass/）+ 一条解决方案一个文档 + README 索引；**分类用元数据不用目录**——曾按活动类型拆 qttech/qtrecurit 子目录被两轮纠正后合并（"分了以后乱"——真实问题同时踩中多类型，deadline 既是课堂也是考核），类型（FAQ/操作指引/制度/背景）作索引列。**业务内主题分组目录**（earn/ 挣券、spend/ 花券）按用户指令建；**命名=客户视角业务名词**：learn/ 文档名三轮重命名（directions→schedules、growth→careers、vouchers→prices）——文件名从"整理者的分类概念"换成"客户在找什么"（客户找课表/职业通道/价格表，不找"方向/成长/代金券"这些内部词汇）；原则已写进 learn/README.md，新增文档前先读。**领域 profile 边界纪律（pay 域 AGENTS.md 定型，source_type 被否实证）**：档案只记本领域一等事实——支付档案只记"资金的规则"（多少钱/怎么算/什么条件触发），不记资金背后的业务语义（"这张券因为什么劳动而发行"归业务系统）；判据 = 这条事实是"资金的规则"还是"某项劳动/成果/身份的事实"；同一事件（初审通过）业务域记成果、支付域只记"触发发券 500 券的计价条件"。**为别的领域发明字段=越界**（曾给 qtcloud-pay 提 source_type issue 被否"这个字段多余，不应该出现"——batch_no 幂等即可，渠道区分由业务方掌握）；**同结构跨域复用时条目命名以商定版本为准，视角可变形状不可变**（pay 的 earn×4/spend×2 曾被我擅自改名 voucher-issuance/consultation-pricing，被纠正"pay的profile的结构又和商量好的不一样！！"）。**同批素材多域落点**（用户纠正过三处）：journal 记事实、insight 记推论/建模信号、profile 记规则档案——用户说"X 改为记录到 insight"时是把内容从 journal 挪到 insight，不是删除重写；来源记录（如职级定价表）先落 journal，用户可能随后直接手工修订 journal（先 pull 再编辑，发现用户已改就以用户版本为基底追加）。**增长管理 profile（quanttide-growth）**：增长策略类问题（如 AARRR 怎么搭）动手前先查 `domains/quanttide-growth/data/profile`——框架已建：qtclass/qtdata/qtcloud 各 AARRR 五阶段 + index 总纲（qtclass/growth-model 判断并入 index：代金券=激活留存工具非收入，现金才进 Revenue，核销率是北极星；qtcloud 按独立业务标准——开源生态获客/意图资产闭环激活/三层收费/社区推荐）。**写增长判断进既有文档，禁自创新文件**（growth-model.md 曾被建又被删——"profile/AGENTS.md 明文：在现有文件基础上修改，profile 是持续维护的活文档，新判断写进既有 index/五阶段文档，主题是章节不是文件"）

**项目管理体系（quanttide-project，2026-09-02 章程体系建成）**：`docs/bylaw` 结构 = 主章程 index.md（领域定义/高质量标准五条/文档体系分工/生命周期三阶段架构）+ **stage/ = 生命周期建模的工作空间**（initiation 立项 → delivery 交付 → retrospective 复盘三阶段专项章程持续维护）。**stage 不是草稿暂存区**——曾把 retrospective.md 移出 stage "转正"被否（"没有允许你移动，也不存在'转正'"）；stage 语义是生命周期本身的结构，不存在谁转正谁待审。**文档体系分工条款**（主章程第五条）：章程管"必须怎样"/手册管"具体怎样做"/案例管"为什么"，代谢关系=案例教训经复盘升格为章程条款；立项/交付章程的"依据"条款显式回指复盘案例（案例→章程升格链已实例化）。**案例脱敏进 gallery**：复盘案例（含客户名/人名/金额/私有仓库/数据文件名）转案例集时按 AGENTS.md 脱敏规则处理（见上方"近期历史的轻量重写"节的脱敏口径），重写为教学叙事（概况→关键节点表→复盘发现→改进方向→案例启示），只保留有沉淀价值的内容不保留空节

**战略章程（quanttide-strategy/docs/bylaw，2026-09-03 建成终稿 + 09-07 章节重排）**：五层战略分层（社区"量潮"→联盟"量潮创新联盟"→公司"量潮科技"→业务→职能），**章节按层级依次成章**（第一章分层框架→第二章社区战略→第三章联盟→第四章公司→第五章业务与职能→附则，用户两次纠正后定稿："社区战略、联盟战略、公司战略、业务战略、职能战略等依次排列"+"第二章就是社区战略"——三大基本战略就是社区层战略本身，不另设"共享原则"章）。**自然人→法人升维贯穿两层**：公司基础主线（制度/标准/档案/系统承载经营能力，发展主线=产品制+规模化获客的前提）+ 联盟核心主线（法人合作机制）同一条主线内外两面。**业务战略三条按用户指定顺序（数据/课堂/云）**：数据=自我数字化改造即服务素材（"自己做过的，才卖得出去"）、课堂=阶梯化人才培养体系（入口/筛选场/出口）、云=社区认知基础设施。**章程内容以用户口述定义逐句下达**（"基本战略是标准化、产教融合、开打闭""产教融合的定义是教育用真实业务当教材，产业用学习过程当交付"）——AI 职责是把口述定义展开为条款落档，不发明定义、不增删条目；每条新战略都是独立追加令，章程与官网方案镜像同步。**简化原则（用户"删除次要细节、似是而非逻辑，重新组织整体逻辑"，17→14 条实证）**：删程序性套话（制定依据/解释权/术语定义）、自我解释尾句、问题背景复述、运作描述伪装成机制；逐段简化法=用户每句纠正执行最小改动+条款号顺延 grep 验证

**章程简化原则（用户"删除次要细节、删除似是而非的逻辑，重新组织整体逻辑"实证，17 条→14 条）**：删——程序性套话（制定依据/解释权/术语定义）、自我解释尾句（"这正是X的差别"）、问题背景复述（"针对当前高校教育与企业需求脱节"）、运作描述伪装成机制、争议定义单独成条；留——每条=定义+一句依据，机制类条款保留构成清单。**逐段简化法**：简化不是一轮到位——用户"合并成一条"（开打闭定义+操作）、"第二章就是社区战略"逐句下达，每句执行最小改动+全篇编号顺延 grep 验证。**章程修正的动词纪律**：用户纠正语义（"stage 是建模""第二章就是社区战略"）是概念澄清不是措辞微调——新语义作为第一性事实落进条文。

**官网方案镜像同步**：qtstrategy 站点建设方案落域仓 `.hermes/qtstrategy-site-plan.md`（事实源=章程，"站点是章程的展示面不是第二份战略文档"，pi 执行建站，方案确认前不启动 pi），**章程更新时方案同步改**（业务战略三条入章后方案 Section 5 同步为数据/课堂/云三卡片）。**站点复刻四要素**：① 先找同构先例（qtsupport 的仓库结构/Terraform 五资源/CI deploy-site.yml，全局替换标识）② 事实源单一 ③ 方案落 staging 交 pi（pi 做脚手架/页面/build 验证；gh 建仓挂子模块/DNS 证书/推 tag 部署需人工收尾，方案显式分节）④ 章程章节→站点 section 一一对应。

**执行管理 specification（quanttide-execute/docs/specification，2026-09-03 落档）**：List × Task 两实体（index 总览 + list.md + task.md，对齐 learn spec 惯例）；List=次级法人 + 四清单归属规则（"任何执行都有归属地"）；Task 九字段（id/list_id/title/description/priority/owner_id/reviewer_id/due_at/status），**owner_id + reviewer_id 双必填**（缺一不可交付），录入规则段引任务录入章程。注意：spec 的 CHANGELOG 尚未建立，首次发布需发 v0.1.0（qtcloud-devops release publish）。**进度汇报章程 stage/progress-reporting.md**（2026-09-03）：状态三分（已完成附证据/进行中附位置/未发生明确标注——"未发生是有效信息，'没写'不是"）、缺口即行动（每缺口带动作链：谁提供什么→谁加工成什么→谁复核）、材料依赖显式化（材料请求清单单独成文+状态标注）、同构映射（事实复用第二场景必须显式映射表+复核人确认）；总前提="汇报反映事实，不塑造事实"。**任务录入章程 stage/task-intake.md**（2026-09-03）：人工路径只要求遵循任务书写标准不另设流程；AI 辅助三步（任务识别→完备性检查→结构化生成），核心约束="AI 只结构化文本中存在的信号，不发明文本中不存在的信号"（缺人留空缺 deadline 留空，补齐是人的责任），四不质量标准（不虚构/不合并/不丢失/可追溯），AI 生成是草稿入库以人确认为准

**领域章程文本修正纪律（2026-09-03 多次实证）**：用户对章程文本的短句纠正（"技术人员->软件工程师"、"no。不要自己改框架，覆盖"、"公开边界和信任规则就是'开打闭'的具体操作"）都是**直接执行令**——落实为最小精确改动（替换措辞/合并条款/调整章节），不做范围外的"顺手整理"；合并条款后条款号全文顺延并 grep 验证编号连续无重复

**领地纪律与"判断降级为整理"（quanttide-work 域 2026-09-10 三轮纠正实证，定稿）**：journal 整理分流是**两阶段模型**，整理动作里彻底没有判断：

- **阶段一（整理时）**：口述流水分节润色（保判断性表述、去口语填充词）→ 移出的内容**一律先进材料库 `data/profile/<作者>/materials/<分类>/index.md` 作材料备用**（**2026-09-11 终态**：材料库按分类目录组织，八类 = agent 智能体 / connect 沟通 / infra 基础设施 / work 知识工作 / org 组织管理 / write 写作 / meta 元工程 / course 课程；条目格式「**标签**：内容」平实陈述、一条一意；早期曾用单文件 `profile/<作者>/default.md`——已被材料库目录取代，`default` 只是八类之一）→ 不做归宿判断（判断"这像规格/像规划"是一次新的判断工作，违背"判断降级为整理"）→ 原文**删除**（不是摘要替代！），只留一份分流索引（各内容去了哪的清单带链接）。**整理已流程化**：工作区任务 `data/context/qtcloud-work/tasks/context-to-profile.yaml`（五步 pull→classify→coarsen→move-out→commit，每步带 executor 与 machine/human 判据），报告落 `data/context/qtcloud-work/artifacts/report/context-to-profile.md`——完整格式、报告形状与"用户只说'整理'时先确认哪个域"的判据见 `references/context-to-profile-pipeline.md`
- **阶段二（使用时，未来独立场景）**：某段材料成熟被认领 → 从 default.md 升级进对应的家（roadmap 规格条目/handbook/insight）并从 default.md 删除

**三轮纠正实录（每条都是违规律）**：① 把"规划类"直接写进 roadmap/system-plan.md、只对材料节做摘要 → 用户："我让你整理到 profile/iGuo/default 备用，而不是直接整理到 roadmap"（整理时不做归宿判断）；② 修正后仍留原文在 journal → 用户："journal 没清理，违反了工作规则"（原文删除是硬性的，摘要+链接只用于"整节移出"的指引，不是留在 journal 的理由）；③ 正确形态 = journal 文件全量重写为分流索引 + default.md 全量入库（每条带来源链接回溯 journal 日期）。**惯例文本落点**：work 域 data/journal/README.md"整理与移出的惯例"四则；第 4 则措辞注意——写"没有明确归宿的才进 default.md"会留判断空间（出错根源），正确措辞"默认全部先移到 default.md，不论看起来是否像规格/规划；认领升级是后置动作"。**材料条目升级 roadmap 的模板**：定义/状态机位置/第一版范围/四阶段演进表挂接总路线推导轴（实证 roadmap/qtcloud-work/material.md）。**context 驱动层（同日定稿 + 09-10 用户两次纠正后的终态）**：journal 转型"按时间线索传递上下文"（记录状态变化、指向生效中的 context），驱动规则归 context（data/context 仓库 + `gallery/artifacts/context.md` 规格；手册 2026-09-12 起重组为件/场/程三件，六件工作物规格迁至画廊 artifacts/——见 `references/knowledge-work-artifacts.md`）——工作物家族六件：语境（**驱动型·无时态**）+ 日志（状态线索）+ 档案/意图/洞察/路线图；验收对称判据"**context 中没有一次性事件，journal 中没有驱动规则**"。**「context 无时态」（用户纠正："context没有时态。纠正一下"）**：语境不是对某个时点的陈述，而是当前生效的驱动规则——因此**不落"现在/未来 × 事件/语义"的时态格子**（别给它臆造类型轴，六件里只有它无时态），且**只能保留最新版本**（陈述型文档可按日期累积，规则型文档累积即自相矛盾；旧版删除、变更过程由 journal 时间线记录）。**context ↔ insight 的分界判据 = 时态**：有预测的归 insight（带时态、可累积、到期证伪），无时态的归 context（只留最新）——同一内容的生命路径 = 洞察（当时预测）→ 事实检验 → 稳定后升为 context 规则（旧预测在 insight 侧按回声三处置）。用户"功能和洞察还稍微有点重复，一定要放最新的东西"即此判据的表述。**通用启示**：用户给出"降级 X 为 Y"类原则（判断→整理、评估→核证据）时，该原则**立即约束自己所有相关动作**（包括自己刚执行过的流程），不只是记录备用。

**用户陈述的"是/不是"常是路径或目的说明，勿升级为结构定位判断（2026-09-10 gallery 误读实证）**：用户说"gallery 定义的实际上都是 work 的契约层"，我读成定位判断、写进 journal"gallery 的定位从图集与展示改为契约层"并提议改 README/index——用户纠正"**不是这个意思**"：其意是**建设路径**——案例库建设偏契约层，目的是**把契约层配套的平台特性需求找出来** → 平台写出来上线 → 加强对知识工作过程之中契约的约束 → 提高工作效率与质量。教训：① 用户给出含"是/定义/层"的短句时，先照录原话再问"这是定位判断还是路径说明"，不要自行升级为结构结论并派生改文档方案；② 该路径本身是 work 域方法论（案例库=需求探针，不是产物；与"从代码出发逐渐导航"同源——不预设平台形态，从一条具体约束里让平台需求自己长出来）。

**spec / contract 语汇（2026-09-10 用户给定，落 journal）**：Specification = 「可接受行为的描述」；Contract = 「交互双方义务与保证的、带责任分配的 specification」；**所有 contract 都可看作一种 spec，但不是所有 spec 都是 contract**（spec 上位、contract 下位加义务保证）。写规范/契约类文档前按此判层级，不混用两词。

**惯例文本的落点要按用户指定的文件（勿自作主张换文件）**：用户明示"journal 整理以后移除成为惯例写到 **journal 的 AGENTS**"——当时落进了同仓 README.md（该仓只有 README，无 AGENTS.md）。用户指定 AGENTS 时应新建/使用 AGENTS.md 承载惯例，README 只做入口导航（对齐"文档独立文件、README 只做入口"标准）。

**默认工作目录（2026-09-11 用户指令）**：用户\"切换到仓库 <X> 的 context，把这个仓库的 iGuo 文件夹标记为默认工作目录\"= 两步——① 会话工作目录切到该路径（`cd`，terminal 状态在会话内持久）；② **在仓库内落显式声明**（实证落为 `data/context/README.md` 的「默认工作目录」节：\"**`iGuo/` 是本仓库的默认工作目录**——知识工作的默认起点：新会话、新成员、AI 系统开工时先从这里读取上下文，再按需进入其他资产\"）。\"标记为 X\"类指令的产出是**仓库内的显式声明**，不是只改会话状态（否则下个会话又不知道）；同时写入 memory。定位依据 = 领域第二大脑一般格式中\"**Context 是默认入口**\"（`docs/gallery/workspaces/domain-second-brain.md`：`data/context/ archive/` = 默认入口与备份出口）。

**同名不同义：tech 域的 context 是「其他仓库的草稿箱」（2026-09-29 用户界定）**——`default/quanttide-tech/data/context` 与上两条（work 域 context = 驱动规则、无时态、只留最新）**不是一回事**，勿互相套用。tech 的 context 语义 = **草稿箱**：内容先在这里起草，定稿后归入真正归属的仓库与分层（`data/insight/`、`data/intention/` 或各领域/业务仓库）；按板块组织且与业务板块同名（default / qtadmin / qtclass / qtcloud / qtconsult / qtdata），板块内按分层建子目录（insight / intention / journal / brochure），默认板块 default；**不是事实源**——查权威口径去归属仓库。用户说「把 X 关系记录到 <仓库> 的 AGENTS.md」时，动作 = 在该仓 AGENTS.md 新增一节（本仓已落「语境仓库（data/context）」节）→ 三层提交链；**改 AGENTS.md 前先 `git diff` 确认只有新增**（该文件可能刚被精简过，长版内容在别的路径，别把旧文写回去）。

**context = 整个第二大脑的草稿箱（2026-10-01 用户升格为全局默认）**：用户说「**context 默认作为整个第二大脑的草稿箱**」——不再只是 tech 域那条特殊约定，而是**默认归属**：内容还没定形、或归属判不清时先落现成的语境仓，判清了再移出（与「资产位置的确定度维度：判不清的先放 context」是同一把尺子）。两条配套实操：

- **用户说「先使用 <仓> 的 <文件夹>」= 不建新仓、不挂新子模块，用现成仓的现成文件夹承载**。实证：本该新建 `quanttide-pi/data/profile` 子模块，用户改口「先使用 context 仓库的 profile 文件夹」→ 落到 `quanttide-pi/data/context/profile/qtfounder/`（**该仓是公开仓，脱敏照旧**）。这个改口顺带绕开了当时建不了新仓的阻塞——草稿箱定位的附带好处就是**不依赖新基建**；所以当建仓/建子模块受阻时，先想「能不能落现成的草稿箱」，而不是卡住等凭据。
- **往语境仓放草稿前先读它的 README**：语境仓的 README 常写着「驱动层 / 无时态 / 只保留当前生效的驱动规则」，与「放草稿」自相矛盾（`quanttide-pi` 的 context README 就是）。**发现矛盾就补一节「草稿」**（写明「兼作草稿箱：没定形或归属判不清的内容先进这里，判定后移出；草稿不属于驱动规则，`profile/` 等目录下即为草稿层」），**并在汇报里点明这一节是你加的、可撤**——别默默放进去让仓库自打脸。
- **该仓无 CHANGELOG 就不建**（`quanttide-pi/data/context` 只有 README + LICENSE + 主题目录）：按现状走，「禁自创新文件」同样约束套件文件；有 CHANGELOG 才记 `[Unreleased]`。

**归位的轴向是反的（2026-10-01 写 context README 时从真实文件核出）**：草稿按**板块**归拢、定稿按**分层**归位——

```
草稿：context/{板块}/{分层}/…      例 context/qtconsult/intention/product.md
定稿：data/{分层}/{板块}/…         例 data/intention/qtconsult/product/studio.md
```

板块名与分层名同名，所以极易按对称假设写反。用户说「把 X 归位 / 合并到 <层>」时按这个轴放置；**这条是从真实文件核出来的**——写这类结构说明前先 `find <context>/<板块> -type f` 与归属仓实际结构对照，不要凭对称直觉。

**重写/精简文档时别照抄源文档里的路径（2026-10-01 自身犯错实证）**：精简 CONTRIBUTING 时我把旧文里的 `docs/index.md`、`docs/myst.yml` 原样搬进新文——两者实际都在**仓库根目录**，旧文本身就是错的，我照搬 = 把错误续命（用户标准：文件名/参数一律从代码或真实数据抄，不得凭印象编）。**硬动作：新文里出现的每个路径，动手时 `ls` 一次**，不因为是「从原文搬的」就免检——上面那条「核验引用存在」只查了目的地的 skill 是否存在，没查这些被搬运的源路径，正好漏掉这个坑。

**草稿箱仓的 index.md = 近期草稿的主线汇总（2026-10-01 实证）**：用户说「浏览 context 的更新，总结我最近在思考什么」→ 读全部草稿后给**本质主线 + 支线**（不罗列议题），用户接着说「总结到 context 仓库的 index.md。新建一个」→ 在草稿箱仓新建 `index.md`，结构与口径：

- **主线一句**（可统摄全部现象的生成机制，不是多角度罗列）+ **支线若干**，每条 = 一句本质 + 关键原话引用 + 出处路径
- 结尾附**板块索引表**（板块 → 内容一句话）
- README 补一行入口（`近期草稿的主线汇总见 [index.md](index.md)`）——草稿箱的 README 是定位说明、index 是内容汇总，两者分工
- **该仓是 PUBLIC，写前必须脱敏**：人名换职务/角色（本人原话里的姓名要去掉）、客户身份模糊化、金额不写——汇总比原草稿更该收紧，不要因为「原草稿里就有」就照搬（先 `gh repo view <repo> --json visibility` 确认可见性）

**分析类请求先出判断再落档**：用户问「总结我在思考什么」时**先在会话里给结论**（等用户认可或修正），不要直接写进仓库——落档是用户看到结论后的下一步指令。

**但草稿箱的 index.md 只是驿站，定稿要合并进归属层（同日实证，别把它当终点）**：用户在 context/index.md 建成后紧接着说「**同意。context/index.md 合并到 intention/intro/index.md**」——动作 = ① 按先前给出的更新建议改写归属层文档（此处 intro=「业务版图与转型叙事」，正是思考主线的正式归属）；② **删除草稿箱源文件 + 撤掉它的 README 入口**（草稿箱不保留副本）；③ 归属层 CHANGELOG 记 `[Unreleased] / ### Changed`；④ 两条子模块链各自推送、父指针一次提交。**判据：这段内容回答的是「我们要什么、为什么」→ 归属层（intention）；是「仓库现在长什么样」→ STATUS；是「最近在想什么」→ 草稿箱（临时）**。用户把同主题两文件称作「旧版本/新版本」时，合并方向就是把新版内容并进旧版所在的正式位置。

**跨层合并的改写纪律**：归属层有自己的文档规范（intention AGENTS：纲领式、**不引用文件路径**、不写状态与工程细节），草稿箱的汇总形态（带出处路径、写「近期」）**不能照搬**——搬运时逐条降级：去掉路径出处、去掉时效措辞、把「最近想通的」改写成「是这样的」。

**纲领 vs 轨迹：intention 与 context 是两种文档物种，规范正好相反（2026-10-01 对比实证）**——用户让「对比 context/index.md 和 intention/intro/index.md」时，要答的不是谁抄谁，而是**两套规范不能互相套用**：

| 维度 | intention（纲领） | context（轨迹/草稿） |
|---|---|---|
| 回答 | 业务**是什么**、怎么咬合 | 我**在想什么** |
| 时态 | **长期稳定**（不随时间变） | **近期快照**——草稿归位即作废，需重写 |
| 引用文件路径 | **禁止**（AGENTS 明文「不引用文件路径」） | **必须**（index 就是导航，每处标出处） |
| 内容性质 | 结论本身 | 指向出处 |
| 组织方式 | 并列定义（条目挨着列）+ mermaid 图 | 层级推导（1 主线 + N 支线） |
| 层级位置 | **主题级** `{主题}/index.md` | **仓级** `index.md`（跨板块才放根） |
| 标题 | 内容标题（「量潮业务版图」） | 元标题（「工作语境索引」） |

推论：① **不拿 intention 的规范去管 context/index**（它天然带路径、天然会过期，不是纲领），反之亦然；② 两份都会描述业务结构时**内容会重叠**（纲领里是「定义」、轨迹里是「最近想通的」），防重叠 = 在 context/index 顶部加一行分工限定（「本页只记正在想的，不记结论；结论的正式归属见 intention」）；③ 对比/检查类请求**只出分析不动文件**。

**index.md 的站点语义只在 MyST 站成立**：intention 有 `myst.yml` + `_build/`，`index.md` 是站点落地页，故主题层用 `index.md`；context 是**纯 Markdown 仓（无 myst.yml）**，板块层用 `README.md` 才对。看两仓 entry 文件名不一致时先 `ls <repo>/ | grep myst` 再判断，别急着「统一」——语义不同不是漂移。

**intention 仓 folder 结构（供快速定位）**：主题层 `{主题}/index.md` 为入口，同主题子题平铺（qtdata 八篇最细分）或二级目录（qtconsult `product/`+`project/`，最新形态）；主题**高于**产品线（治理主题与业务线平级）；领域级意图已下沉到 `domains/quanttide-*/data/intention/`，本库只留公司编排级；失去活性归档到 `data/archive/intention/`（不删）。成熟度不均——qtclass/qtdata 丰满，qtconsult 是 1 行空壳。**新增子文档后核对 `myst.yml` 的 toc 注册**（实证：`qtconsult/product/studio.md`、`project/self.md` 存在但未注册，站点里访问不到）。

**work 域文档体系速览**（工作物矩阵/context 与 insight 分界/材料/journal 整理四则/案例库路径/默认工作目录）：`references/knowledge-work-artifacts.md`

**context → profile 整理流水线**（工作区 tasks yaml 五步/八分类材料库/报告形状/用户只说"整理"时先确认哪个域）：`references/context-to-profile-pipeline.md`（含「去向二：context → insight」节）

**「整理」是搬迁不是建设——「其他事情都不要做」纪律（2026-09-12 实证）**：缺配套的域（无工作区、无材料库）其整理去向可能是 **insight** 而非材料库；用户说「整理」时不要顺手建工作区、建分类目录、补 README、补 AGENTS——我为 code 域出的整套结构方案被否：用户回「不对。只把 Code 的 context 移动到 insight 整理。其他事情都不要做。」正确形态 = 迁入 `<域>/data/insight/<slug>.md`（洞察格式组织）+ 语境日期文件留空占位 + 四层提交（insight 仓 → 语境仓 → 领域仓 → 根仓）。

**编程智能体设计立场**（code 域 09-12：瓶颈是对齐不是生成、工作流不是产品核心、探索者-观察者架构、六个待解问题；code context/insight 仓现状；**含首次验证结果**——零成本翻 journal/profile 得\"反馈循环缺失\"可测形式、观察者只有雏形「AI 建议+人确认」）：`references/programming-agent-design.md`。洞察的最小成本验证方法（证据分级/可数化改写/验证记录写法）在 insight-writing 技能的 `references/insight-verification.md`

**Report 实体设计实录（2026-09-03，"执行完的任务进日报/周报"三轮收敛，spec 扩展方向待拍板）**：终态模型 = Task（过程实体，状态可流转）进 done 时**触发生成独立 Report（静态记录实体，不可变）**，与 delib 的 Topic→Resolution 同构（用户点破的参照："参考议事管理制度中的议题 issue 和决议 resolution 的关系"——决议不做成议题的状态列，报告同理；差异：Resolution:Topic=1:1，Report:Task=1:N 聚合）。三轮判据：① 投影方案（Digest 视图不落库）被"为什么不能转换成新实体"一问否——**投影是现在时（随看板漂移），报告需要过去时（定格当时事实）**；人写的复盘判断不属于单个 Task，必须有独立落点；AI 提炼管线需要可寻址输入单元；② WorkLog 命名被否（"worklog 还不够准确"）→ 盘点体系内既有词汇定名 **Report**——`default/quanttide-tech/data/report/` 已有工作日报/周报/月报惯例（命名对齐惯例不造词，Outcome 也被否——它把实体语义拉向单任务结果与 OKR 语境，而该实体是批任务聚合+人写复盘）；③ **节拍不拆实体**——Daily/Weekly/Monthly 是同一 Report 的 period 枚举。配套：Task 补 `completed_at`；看板 done 卡只保留最近 N 天（默认 7 对齐周会）防泳道膨胀；流转 = Task done → 自动汇入当日 Report → 周日聚合周报 → 周会"读系统"→ AI 提炼回 List（北极星闭环）。Report 与 journal 分工：系统侧实时结构化定格 vs 第二大脑深度沉淀，互补不复制。**实体命名方法论（本轮沉淀）**：被否一个名字后，① 先 grep 体系内既有词汇（data/report/ 惯例）② 检查候选词在协作工具语境的错位（Outcome≈OKR）③ 用户拒绝 AI 提名后说"根据我的定义来"= **等用户给定义，不再自行提名**（曾继续推 Report 被打断"停止"）

**学习管理体系骨架演进警示（2026-08-31 实证，涉及 learn/execute 等新域时先重查结构）**：骨架建好后**当天就可能被用户重构**——learn 的 profile 当天从 `qtclass|qttech/` 组织分组重构为 `learners/ + schedules/ + tasks/` 平铺（learner 档案移 learners/、Schedule 抽出成 schedules/ 独立文档、Task 全文沉淀 tasks/ 单文档 YAML frontmatter）；journal 从 `qtclass|qttech/{学习者}/` 重构为顶层 `{学习者}/`。教训：① **接手当天建过的域，先 ls/git log 重查结构再动手**——结构演化和内容更新并行；② **specification 先行于 profile 演进**：Schedule/Task 契约（spec v0.1.0：Learner/Completion 已落码）定型后，profile 骨架按契约重组（schedules/tasks 从档案中独立=领域模型驱动信息架构）；③ 重构后旧路径引用全部失效（AGENTS.md、SKILL 记录的旧结构都要重查），不要基于记忆中的结构 patch。

### 学习管理域（quanttide-learn）补充记录（2026-09-05 会话）

- **journal 首篇学员日志与脱敏**：qtclass/Jerry（飞书教学组 IM 导出整理）、qttech/iGuo（自学日志，按管理领域归类主题）。**脱敏教训**：来源引用块（"来源：飞书XX群（时间段）"）用户随后要求整行删除——来源渠道本身是敏感面，日志头部不写来源；README 的"lark-cli im 导出"字样同删。脱敏要用 git-filter-repo 全历史重写而非普通 commit（普通 commit 删行历史仍在）
- **iGuo 自学日志的主题归类（2026-09-05 用户拍板）**：AI 提炼五主题后用户按领域重归类——1+2 合并为"组织管理"、3="资产管理"、4="机密管理"、5="学习管理"（**用户的领域划分尺子：管理领域归并，不按事件罗列**）；后续同类整理直接按四领域框架归类。**日志先于档案（用户拍板"先维护日志，再维护档案"）**：档案素材必须先落 journal 获确认，再提炼进 profile
- **iGuo 学习任务自举（用户指令"尝试列举 iGuo 的学习任务和验收标准，写到今天的日志"）**：从当日会话证据回溯列举 Task+验收标准（验收标准形态=产出物+他人认可，非答卷），含"自举观察"节（元收获：给自己列验收标准比给学员难——探路者的标准在做事之后回溯定义）
- **AGENTS 档案定义以最新实践扩展**：profile/AGENTS.md 双变体（学员验收制/自助学习者主题制），提炼规则四条（快照非流水/只写已观测事实/待验证 checkbox/不引用日志路径不写敏感来源）
- **bylaw 章程四轮返工**（详见下方"章程/制度类文档写作"节）：控制范围（只管 specification 模型）→ 理解先行（被否后先交理解版拿"对了"）→ 去概念化（"社会契约/异议权/范式构建者"全砍成动作）→ 白话终稿（六章十八条"每条记录都要有证据，每个判定都要能复核"）。**第五轮再收（2026-09-05）：概念词零容忍**——第四稿白话版仍有"我渐渐发现式"的叙事残留被要求"通俗点"再砍；终稿每条都是纯动作句（"先做事后记录""求助不扣分""档案跟人走"），无任何需要解释的词。**章程标题后的定位段一句话即可，"章节定位/与社会契约的哲学分工"类论述段全是虚头巴脑变体**
- **章程/规范类文档的"分析器 vs 人话"判尺（2026-09-05 learn bylaw 五轮实证，通用）**：用户连续否定时不是往更技术走（"数据质量控制16条"被"太技术了"否）也不是往更概念走（"社会契约/信任原则"被"减少虚头巴脑的概念"否）——**往更具体的动作走**。每条自检三问：能执行吗？能违反吗？能检查吗？三问全过才留。"权利话语"（XX权/XX原则）、"哲学定位"（X是Y的契约）、"身份标签"（范式构建者）全是虚头巴脑的变体；白话的检验法 = 读给非技术的人听，不需要解释任何词
- **章程发 CEO 办公室群**：lark-cli im +messages-send --markdown（chat_id 先 chat-search 拿），以 user 身份发送
- **lark-cli 发消息坑**：`+messages-send --markdown` 成功返回 `ok: true` + message_id；chat-search 找群名拿 chat_id 最快

- CLI 已安装：`~/.cargo/bin/qtcloud-devops`（子命令 `release audit|publish|status`）
- **新建独立官网/门户仓库模式（2026-09-01 qtsupport 实证）**：站点类应用（如 support.example.com）是**独立仓库**（如 `qtsupport`），不挂为 qtcloud-<domain> 的 src/site 子模块（曾挂上被用户纠正"qtsupport和qtcloud-support是两个独立的仓库"——撤销子模块挂载，改为在领域仓库 `apps/qtsupport` 平级挂载 + README 关联说明）。仓库结构对齐 qtorg 惯例：`src/site`（React+Vite SPA）+ `manifests/terraform`（桶+CDN+DNS IaC：私有桶、CDN 私有回源 l2_oss_key、SPA 回退改写 back_to_origin_url_rewrite、https_force、泛域名证书）+ `.github/workflows/deploy-site.yml`（`site/*` tag 触发）。可先复制兄弟站点 node_modules 快速验证 `npm run build` 再删。内容风格参照 support.apple.com：大留白 hero、卡片网格、极简 Markdown 渲染内置（无第三方依赖）
- 流程：① 先更新 `CHANGELOG.md`（`## [X.Y.Z] - YYYY-MM-DD` + `### Added`/`### Changed`，仓库文档风格：无 emoji）→ ② `qtcloud-devops release audit -v <版本>` 预检（版本格式/CHANGELOG 含版本/tag 不冲突/工作区干净，7 项全过才发）→ ③ `publish -v <版本> -y` → ④ `gh release view <版本> --repo quanttide/<repo>` 验证 → ⑤ 父仓库更新子模块指针（分层提交流程）
- **版本号格式必须是 `scope/vX.Y.Z`**（应用仓库，如 `site/v0.1.0-alpha.1`、`studio/v0.1.0-beta.1`；qtcloud-devops help 明示 `vX.Y.Z` 或 `scope/vX.Y.Z`）——裸 `0.1.0-beta.1` 被拒"版本号格式无效"，`v0.1.0-beta.1` 也只能部分通过；文档仓库用 `vX.Y.Z`（scope 版本是给多端应用的 tag 前缀，也是部署 workflow 的 `site/**` 触发条件）
  - **多语言包 monorepo 的 scope = 语言包名（2026-09-29 toolkit 实证）**：一个 toolkit 仓内多语言实现（`packages/dart`、后续 `packages/rust`、`packages/go`）时，scope 取语言名——发 dart 包用 `dart/vX.Y.Z`，不是裸 `vX.Y.Z`（用户纠正：「不对，是 dart/v0.1.0-alpha.1，我等会还要写 Rust 和 go 的」）。**代价提醒**：裸 tag 发出去后要还删远端 tag + `gh release delete` 重来一遍，动手前先确认 scope。scope→CHANGELOG 映射 = 包目录（`dart/` 读 `packages/dart/CHANGELOG.md`），根 CHANGELOG 只做索引（指向各包 CHANGELOG），**根 CHANGELOG 里不要放某语言的版本条目**（首轮误放根目录，audit 报「未找到版本记录」，移到包目录才过）
- **audit 查的是仓库根 CHANGELOG.md**——只改子目录 CHANGELOG（如 src/studio/CHANGELOG.md）会报"未找到版本记录"：根 CHANGELOG.md 也要加同版本条目
- **分 scope 仓库的发布（2026-08-26 qtcloud-crowd/qtcrowd 实证）**：`publish` 子命令**不支持 `--changelog` 参数**（报 `unexpected argument`）——qtcloud-devops 自动定位 scope 的 CHANGELOG 文件。根 CHANGELOG.md 只做**索引**（列各 scope 版本条目），**真正的版本条目必须在 scope 文件里**：发布 `site/vX.Y.Z` 时读 `src/site/CHANGELOG.md`（发布 `provider/*` 读 `src/provider/CHANGELOG.md`）。曾改根 CHANGELOG 加 beta.6 条目仍报"未找到版本记录"——因为 site scope 读的是 src/site/CHANGELOG.md。先确认仓库是"根承载条目"还是"scope 文件承载条目"模式（qtcloud-execute 用 `--changelog` 指定=根做索引；qtcloud-crowd/qtcrowd publish 无此参数=scope 文件承载）
- **配置文件一致性检查**：pubspec.yaml / package.json 的 version 必须与发布版本一致（曾 0.1.0-alpha.1 未同步导致 audit 不过）——`sed -i 's|^version: .*|version: <ver>|'` 后重跑 audit
- **publish 前的用户指令核查（2026-09-01 实证）**：发布前重读用户指令的措辞与范围——"清空 CHANGELOG 发 v0.0.1，用 qtcloud-devops release publish"是显式授权（可直接执行含 dry-run 预检）；但 publish 的目标是**用户指定的那个仓库**（specification），同会话其他仓库（journal/insight/profile）保持常规提交推送即可，不要把 publish 纪律套到无关仓库
- 仓库 `.agents/skills/devops-release/SKILL.md` 与 CLI 并行：预检查逻辑一致，发布动作交给 CLI
- **版本号偏好（用户纠正过）**：文档/意图仓库默认发 **patch**（如 v0.2.2），不要自作主张发 minor——助理曾提议 v0.3.0，用户改口"发 patch"。提版本号前先问，或默认 patch
- **发布授权纪律（2026-08-26 两次越权实证——最高优先级）**：**"部署/配置/你配置你部署" 绝不等于"发版本"授权**——qtcloud-devops release publish（打 tag + 建 Release + 触发部署 workflow）是独立的高风险动作，**必须用户明确说"发版本/发布 vX.Y.Z"才执行**。曾两次误解："你配置你部署，不要偷懒"（理解为全流程含发布）→ 发 provider/v0.2.0 → 用户"谁让你发 0.2.0"；删 tag 后又重发一次 → 用户"没让你发！"。正确解读：**"部署"= terraform apply/手动部署动作（建桶/更新 FC），"发版本"= tag + Release 流程**——两者分开，前者可自主（配置后自己跑），后者必须显式授权。用户否认发布后的恢复流程：`gh release delete <tag> --yes --repo <owner>/<repo>` + `git tag -d <tag>` + `git push origin :refs/tags/<tag>` + **CHANGELOG 版本条目回退 [Unreleased]**（已提交的发布条目要回退并推送）；已触发的部署 run 停不掉（等它完成即可，桶/FC 更新无害）
- **site tag 发布流水线（2026-09-02 qtclass 实证，beta.4→6 三连发）**：流程 = 版本号三同步（package.json + scope CHANGELOG + tag）→ commit → 父仓库指针 → `git tag site/vX.Y.Z && git push origin <tag>` → `gh run watch <run-id>` 等 CI → **线上验证两步**：curl 首页取新 asset hash 确认 CDN 已更新，再 curl 该 JS grep 内容关键词（含"应删除"的负向检查——曾发现 beta.5 线上 bundle 仍含残留条目"超额申请额度"，立即追发 beta.6 修复）。**CHANGELOG 迭代纪律**：多版本快速连发时 Unreleased 条目按版本号顺序落位（beta.6→beta.5→beta.4 倒序），曾因 python 替换把 beta.5 块整体顶丢需补齐；多版本记录一个不漏。**披露内容先过滤再上线**：写对外站点内容前先过"这是不是对客户的价格/事实"这一关——内部考核规则（限额定价/负数职级/自然淘汰）未经确认不披露，宁可板块留空（用户对 careers 板块的否决："数据质量差，全部删除"；超额申请额度"暂时不是实训的价格"）

**站点复刻建设方案（2026-09-03 qtstrategy 实证）**：为某域建"中心/门户"站点时，方案四要素——① **先找同构先例**（qtsupport 的仓库结构/Terraform 五资源/CI deploy-site.yml 是站点模板，全局替换标识即可：qtsupport→qtstrategy / support.example.com→strategy.example.com / qtsupport-site→qtstrategy-site / TF state key qtsupport/site.tfstate→qtstrategy/site.tfstate）；② **事实源单一**（站点内容以领域章程为唯一事实源，"站点是章程的展示面不是第二份战略文档"，语言用章程原文关键句禁同义改写）；③ **方案落 staging 文件交 pi 执行**（域仓 `.hermes/<app>-site-plan.md`；pi 能做：复制脚手架/写页面/本地 build+preview 验证；pi 做不了需人工/Hermes 收尾：gh 建仓挂子模块、DNS+证书、推 tag 触发部署、父仓库 AGENTS.md 子模块表更新——方案里显式分节）；④ **章节映射**（章程章节 → 站点 section 一一对应：Hero→战略分层→社区→联盟→公司→业务战略阶梯卡→Footer 来源声明，用户随后追加新战略内容时章程与方案两处同步更新）。方案确认前不启动 pi。

### 网络波动时的发布续传（GitHub 时通时断环境下常见）

- publish/push 超时不代表失败：先查部分状态 `git tag -l` + `gh release view vX.Y.Z --repo quanttide/<repo>`——**tag 未创建则无脏状态**，网络恢复后重跑 publish 幂等安全
- 网络诊断顺序：先测全局（baidu/gitee），再测 GitHub；DNS 正常但 TCP 超时 = 网络环境问题；检查 proxy（`env | grep -i proxy`、`git config --get http.proxy`）。注意管道会让 `$?` 变成末命令退出码，应单独测：`curl -s -o /dev/null -w "HTTP %{http_code} in %{time_total}s"` 再 `echo $?`
- 先本地 commit（push 与网络解耦），`git status -sb` 可见"领先 N"
- 后台自动重试推送：`for i in $(seq 1 30); do timeout 25 git push && exit 0; sleep 20; done`（terminal background=true + notify_on_complete=true），网络恢复即自动推送
- 网络中断间隙**继续做本地工作**（用户偏好：先继续调整，不干等）；网络恢复后 push 与 publish 分开做，避免再超时
- **本机 proxy 配置失效的识别与绕过（2026-09-05 实证，本机高频）**：症状 = git 报 `Failed to connect to github.com port 443 via 127.0.0.1 after 0 ms`（0 ms 连接被拒），而 `curl https://github.com` 直连返回 200。根因 = 全局 `http.proxy/https.proxy`（本机为 `http://127.0.0.1:7897`）指向的本地代理没开。**绕过而不改全局配置**：`git -c http.proxy= -c https.proxy= <pull|push|ls-remote>`；诊断时先 `bash -c 'echo > /dev/tcp/127.0.0.1/7897'` 探端口（连接被拒=代理没开）。**但别把「代理已死」当成既定事实**（2026-09-24 反例：同一台机该端口又有进程监听，走代理 `https://github.com` 1.6s 返回 200，而**直连 github.com 超时**——此时沿用 `-c http.proxy=` 绕过反而挂到超时，白等一轮）。探活顺序：`ss -ltn | grep 7897` 看有没有人监听 → 有监听就用默认（带代理）配置试 `timeout 40 git ls-remote origin HEAD`（最廉价的连通探针）→ 仍报连不上再考虑绕过。另：`api.github.com` 的可达性常与 `github.com` 不同（实测前者 200、后者超时），只读查 CI 结果走 API 不受影响
- **gh token 过期导致 https 推送失败（同日实证）**：`gh auth status` 报 `The token in ~/.config/gh/hosts.yml is invalid`，随后 git 推送报 `could not read Username for 'https://github.com'`（凭据链 `!gh auth git-credential` 失效）。**不要卡在重新登录**——先 `ssh -T git@github.com` 验证 SSH 可用（返回 `Hi <user>!` 即可用），然后 `git remote set-url origin git@github.com:<owner>/<repo>.git` 再推送（实证 quanttide-founder 从此走 SSH）；gh 命令本身仍需用户择时 `gh auth login`

- **不换 remote、也不改全局配置的轻量绕法（2026-10-01 连续用了三次）**：`git push git@github.com:<owner>/<repo>.git main`——显式给 SSH URL 推送，一条命令搞定。比 `git remote set-url` + 事后还原干净（本仓族 `origin` 全是 https，改了 remote 会让后来人看到的配置与实际不符）。**诊断三步，顺序别颠倒**：① `git config --get http.proxy`（看有没有指到死端口）→ ② `ss -ltn | grep 7897`（代理在不在听）→ ③ `git -c http.proxy= -c https.proxy= ls-remote https://github.com/<owner>/<repo>.git`（直连是否可用）。本次判读：全局 `http.proxy=http://127.0.0.1:7897` 而 Clash 进程没跑 → **所有走 https 的 git 操作被拦**（报 `Failed to connect to github.com port 443 via 127.0.0.1 after 0 ms`），但直连 https 完全正常（`curl -o /dev/null -w "%{http_code}" https://api.github.com` 返回 200）——**是配置指向死端口，不是网络墙**，所以优先绕过或显式 SSH URL，不要急着装/启代理。SSH 侧同一时刻是通的（`ssh -T git@github.com` 返回 `Hi <user>!`）

### git 历史脱敏重写（git-filter-repo）

公开仓库历史含敏感信息（客户名/项目细节/员工名/来源渠道）需要重写历史脱敏时（用户明确要求才做）：

1. **先备份**：`git bundle create /tmp/<repo>-backup.bundle --all`
2. **安装**：GitHub raw 不通时用 `UV_DEFAULT_INDEX=https://pypi.org/simple uv tool install git-filter-repo`（清华镜像常 403；filter-repo 会移除 origin，执行后需重加）
3. **写替换规则** `/tmp/replace-rules.txt`：每行 `旧文本==>新文本`（逐字面替换；覆盖大小写/带空格变体，人名、项目名、交付物名、金额、算法名都要列）。**脱敏范围一次问全（2026-08-30 实证）**：人名/代号之外，主动把**来源渠道痕迹**（平台名、工具名、群名、时间段、邀请链接、个人邮箱）一并列规则——用户首轮只说"脱敏"，第二轮补"来源也删除"，两轮 filter-repo 的代价比一次问全高一倍；宁多勿漏，删干净的行比留痕的行安全
4. **执行**：`git-filter-repo --force --replace-text /tmp/replace-rules.txt`（非 fresh clone 必须 `--force`；会解析全部提交重写并 repack）。**坑（2026-08-26）**：非 fresh clone 会交互提示确认（`response = input(msg)`），无 TTY 时直接 `EOFError` 中止——用 `echo "yes" | git-filter-repo ...` 管道喂确认
5. **重写后**：重新 `git remote add origin <url>`，`git push --force origin main`
6. **父仓库指针**：子模块 gitlink 指向旧提交——父仓库 `git add <submodule>` + commit + 逐级推送
7. **验证**：`git log --all --oneline -S "敏感词"` 无残留；**删除整行内容时也要查历史**——普通 commit 删行清不掉旧提交，仍需 filter-repo（2026-08-30 实证：先普通 commit 删来源引用，抽查 `git show <旧sha>:<文件>` 发现历史仍在，再补一轮 filter-repo）；确认无误后删除备份 bundle 与替换规则文件

注意：force push 重写远端历史，只做用户明确要求；操作前必须备份。

### 近期历史的轻量重写（2026-09-02 gallery 实证，单文件近期提交优先用此法而非 filter-repo）

敏感内容只进了最近几个提交时，`git reset --soft <init或干净提交>` 重建单提交 + `git push --force` 比 filter-repo 简单可靠（filter-repo 适合全历史多文件替换）。流程：改文件 → `git add -A && git commit` → `git reset --soft <干净基线>` → 重新 commit（合并为单提交）→ force push。

**GitHub 悬空提交现实（实测确认，已写入 project gallery/AGENTS.md）**：force push 丢弃的提交**仍可通过精确 SHA 直达**（commit 页面/.patch 都可读，悬空对象不自动回收无过期）——重写历史≠远端已清除，彻底清除需联系 GitHub Support 请求 GC；唯一可靠策略是**第一次提交就写对**。

**内容脱敏要过多轮自查（gallery 实证，首轮必漏）**：首轮脱敏人名/机构/金额后，逐段重读还会漏——① **身份特征组合**（"海外高校研究者"三特征组合可反推客户，改"外部学术研究者"）；② **处理路线直白转写**（"本地处理/云查询/替代数据源三条路线"≈ 替代方案原文，懂行即可反推数据源）；③ **内部工具链形态**（模块名"契约生成/蓝图/规格封装"暴露自家 CLI 结构，收敛为"记录了 N 个问题，清单已回流产品侧"）；④ 用户规模数字（"六万余名"也要删）。检验口径：**读完整篇知道"这是一类什么问题"，无法知道"是哪个客户哪个项目哪条数据"**。脱敏规则成文见 project gallery/AGENTS.md（禁入清单/辨识度下限/先脱敏再写作/Git 历史纪律）。

## 领域知识资产的骨架填充纪律（2026-09-02 多域实证，最高优先级）

用户建好领域骨架（领域仓库 1 主 + N 子的资产位、profile 的 index+阶段文档、bylaw 的主章程+stage 三章程）后，**填充工作遵守**：

1. **框架已定，往格子里填**——不要自创新文件/自建目录/自加状态列。实证连串：support profile 按"活动类型"拆 qttech/qtrecurit 目录被两轮纠正合并（"分了以后乱"）；growth profile 建 growth-model.md 新文件被删（"禁止自己发明新文件！！！"）；project bylaw 给 retrospective 标"已发布"并移出 stage 被否（"没有允许你移动，也不存在'转正'"）。正确动作：新判断写进既有 index/阶段文档的对应章节——**主题是章节不是文件**。
2. **先探明骨架语义再动手**——stage/ 这类目录名可能承载用户定义的建模语义（project 的 stage=生命周期建模工作空间，非草稿暂存）；pay 的 profile 结构（earn×4/spend×2）跨域复用时条目命名以商定版本为准。接手仓库先读 AGENTS.md/README/既有文档的实际语义，不按行业惯例猜测。
3. **披露内容先过滤**——写对外站点/文档前先过"这是不是对客户的事实/价格"关：内部考核规则（限额定价/负数职级/自然淘汰）未经用户确认不披露，宁可板块留空。
4. **多域落点先问归属**——同批素材（如代金券定价）可能跨 support/pay/growth 三域；先定档案归属（写错档案被纠"本来该写 pay 的！！！"、"是 qtclass 的。重写"），归属错了整体迁移而非就地补丁。
5. **骨架填充的顺序**：领域意图/章程先行（bylaw index 主章程是其他章程与手册的依据）→ journal 回溯 → profile/手册/案例展开；每步三层提交推送。

## 文档套件精简（CONTRIBUTING/AGENTS/README/STATUS 家族，2026-09-29 quanttide-tech 实证 134→38 行）

用户问「`<文档>` 如何精简」时**先读全文出方案、等 yes 再改**（与「检查类请求=要方案」同纪律）。判据 = **只留本仓库特有的约定**（别人照抄就会错的东西）。三类内容必砍：

| 类 | 症状 | 本次实例 |
|---|---|---|
| **失效的** | 指向不存在的路径；流程与本仓实际架构矛盾 | SKILL 同步流程的目标 `docs/gallery/devops/<skill>/SKILL.md` 根本不存在（gallery 下只有业务板块目录）；「工作流程」写单仓 PR 流，而本仓是子模块架构 + 提交即推送 |
| **重复的** | 内容已在 `.agents/skills/`、外链规范或别处更全 | 构建/部署文件表 ↔ `docs-deploy` skill；子模块操作命令 ↔ `devops-submodule` skill；SKILL 格式 ↔ agentskills.io 外链 |
| **通用的** | 任何项目都有的套话 | 「更新流程」五步（记录问题→分析原因→修复→更新文档→持续改进）；「建分支→开发→推送→PR」四步 |

**归类规则（砍掉的东西各有去处）**：通用流程 → 外部规范（改外链）；操作步骤 → `.agents/skills/`；AI 工作经验 → `AGENTS.md`（本仓惯例「AI 工作经验增加 → 更新 AGENTS.md」）。留在 CONTRIBUTING 的 = 本仓特有约定（指针提交 commit 格式、新增/取消挂载要同步哪些文件、CHANGELOG 子模块版本标注、提交即推送）。

**动手前后两条硬动作**：① **核验文中引用的路径与 skill 真实存在**（`ls`/`find`）——引用不存在即失效内容，这是最该砍的一类；② 改 AGENTS.md 时先 `git diff` 确认只有预期改动（该文件可能刚被别的会话精简过，长版内容在别的路径，别把旧文写回去）。

**AI 经验迁移要二次去重**：搬进 AGENTS.md 前逐条比对——已在某个 skill 里的（部署模板/构建命令区分/本地先跑通）不搬，与根仓库 AGENTS.md 重复的（试错上限 ↔ 收敛监控）不搬，只留本仓特有的。

**汇报形状**：行数变化（134→38）+ 分类清单（砍了什么/为什么/留下的在哪/去哪了）+ 引用核验结果。

**「对比 X 和 Y」的默认维度是内容，不是结构（2026-10-01 实证）**：用户说「查看 A 并对比 B」时，我先给的是**结构/元信息对照**（仓库类型、entry 文件名、文档套件齐缺）——用户随即收紧为「**只对比内容**」。口径：对比类请求默认产出**内容差异**（两份「说了什么」不同）；结构/元信息差异只在用户明确问到时才讲。**用户把同主题两份文档称作「旧版本 / 新版本」时，产出 = 基于新版的更新建议**——逐条「旧表述 → 新版依据 → 建议改成什么」，按「**必改（直接冲突）/ 该补 / 可留**」分级，不是并列对照表；「新版」是驱动方、「旧版」是被更新方。**这类请求的产出同样是方案**——用户说 yes/同意 才动文件（本轮「同意」后紧跟一句明确的合并指令才落笔）。

**文档层级分工见 `references/content-repo-myst.md` 第十节**：README = 仓库说明 / 根 index = 书籍的介绍页面（封二）/ 章内 index = 正文该章第一篇；判别尺子 = **句子主语是「这本书」还是「这件事」**；`myst.yml` 的 toc 首项必须指 `index.md` 而不是 `README.md`。

**STATUS 的专属判据：只装快照，分析一律移出（2026-10-01 实证 19→13 行）**——STATUS 定位是「仓库当前快照」，任何**分析**都不属于它。本仓 STATUS 里那节「qtdata 差距分析」（9 行，占近一半）实为**单域分析**，正式归属是 `data/insight/index.md`（五应用全景锚点）+ `data/insight/qtdata/intention-gap.md`（本域细节）——STATUS 里那三行只是全景里 qtdata 一格的复述。判据：这段内容回答的是「仓库现在长什么样」还是「为什么/差距在哪」？后者一律移出。

**删内容前先 grep 反向引用**：`data/insight/qtdata/intention-gap.md` 的行动建议写着「STATUS 文档已做差距分析」——删 STATUS 那节前先找这类**指向被删内容的句子**，改为自洽表述（→「差距分析已完成（本目录）」）并连同提交，否则留悬空引用。这一步属独立子模块时走完整分层链（insight 仓 → quanttide-tech → 根仓）。

**工具用法沉淀进 library（2026-10-01）**：用户说「在 <仓库> 的 library 写上**经过验证**的 <工具> 的用法」= 收录到 `data/library/tools/<tool>.md`，对齐该目录现有格式（概览表：官网/GitHub/协议/语言/存储 → 核心能力 → 架构模式），核心价值在**「经过验证的用法」一节 + 踩坑记录**（可复制的启动命令、报错原文与成因、必须同时打开的开关），不是官网文档的复述。

## 章程/制度类文档写作（2026-08-31 bylaw 三轮返工沉淀，通用迭代序）

写章程、制度、规范类文档时的**强制迭代序**——跳步必返工：

1. **范围先行**：先确认该章程管辖什么（如 bylaw 只管 specification 模型的运行质量控制），越界内容（角色体系/意图/档案格式）明确说"归 X 文件管"并排除——第一稿把角色/意图全塞进来被否"只局限在 specification 规定的范围内"
2. **理解先行**：用户否掉范围后说"先说说你的理解"= **先交理解版拿确认，再动笔**。理解版用人类语言描述该制度实际调节什么（实证：从"数据质量规章"改述为"人与人围绕成长的权利义务"获"对了"）
3. **去概念化**：确认方向后的成稿 = 概念全部还原为动作。检验法：每条都必须**能执行、能违反、能检查**；出现"XX权/XX原则/XX契约"这类需要解释的名词即回炉（"社会契约/信任原则/身份跃迁/被看见即激励"全被否，换成"异议不需要写理由书""记录不挪用——要用先经本人同意"）
4. **总纲一句话**：制度文档开头只留一句可执行的总原则（终稿："每条记录都要有证据，每个判定都要能复核"），不要哲学论述段

### 方向漂移的早期信号（2026-09-05 创作谈引擎五轮迭代的根因——先于回退，可更早发现）

1. **范本锚定先核对用户语境**：接手"产出 X"类任务，先找**用户自己写的 X 成品**当范本——不要抓仓库里名字相近的文档（实证：把规则文档 fiction-revision.md 当"创作谈"样板写进规格，它是 AI 操作手册不是作者随笔；用户当天晚些时候在 fiction 仓库亲自提交 `创作谈/` 三篇随笔，那才是真范本）
2. **跨域纪律不裸搬**：档案纪律（证据链强制/保守输出/来源标注）对记录体系对，装进创作/文体工具就产出审计报告——每个域有域的灵魂，建一个域的工具用那个域的灵魂
3. **同类纠偏第二次出现 = 停下交原始样本**：曾把连续纠正（"正反规则都收"→"母题/风格两维度"）逐次读成"框架对，内容再补"，实际产出形态从第一版就错了，每次照做都是分析器加深。正确动作：拿一份**未加工的原始产出**问用户"这是你要的形态吗"——早于任何功能迭代
## 教训（写引擎/文体类实验的通用）

1. **产出被否 ≠ 加功能/加模式**——"分析器 vs 创作谈"是文体错误，不是功能缺失；先确认用户要的文体定义，再动提示词
2. **AI 提议的文体定义未经用户确认时，用户说"退回原始状态"指全部回退**——包括自己刚写的未提交版本；不要保留半成品"以后能用"
3. **实验被否时数据产物一并清理**（用户"也清理"）——AGENTS 临时文件规矩（产物进 data 提交）是通用规矩保留，但具体产物随实验命运走
4. **等用户给定义**：被否后用户说"停下来，先明确 X 是什么"= 用户要自己定义，AI 不再提议
5. **范本锚定先核对用户语境（五轮迭代的根因，详见上方复盘）**：接手"产出 X"类任务先找用户自己写的 X 成品当范本，不抓仓库里名字相近的文档；跨域纪律（档案/审计那套）不裸搬进创作类工具；同类纠偏第二次出现 = 停下交原始样本问"这是你要的形态吗"；文体/风格类产物禁用自造标准自评

## 方向否决后的回退纪律（2026-09-05 创作谈引擎实证）

实验/功能被用户否决方向时，回退要**干净且完整**：

1. **工作区未提交的改动**：`git checkout -- <files>` 丢弃，不留"以后能用"的半成品
2. **已入库的实验产物**：随实验一起清理（用户"也清理"= git rm 提交）——AGENTS 的临时文件规矩（产物入库）是通用规矩保留，具体产物随实验命运走
3. **已提交的历史**：远程与本地一致时不重写历史，叠加清理 commit 即可；无未推送 commit 则无需 reset
4. **AI 自行重写的未提交版本也在"丢弃所有"范围内**——用户说"退回到之前的原始状态"指全部，不要保留自己刚写的版本等确认
5. **被否后用户说"先明确 X 是什么"= 用户要自己定义**——AI 停止提议定义/方案，等用户给

## quanttide-founder 实验室（方法论→可执行程序，2026-08-30 学习实证）

`~/repos/quanttide-founder/examples/default`（创始人实验室，**独立于 quanttide 主仓库群**，~/repos 下平级）——把方法论翻译为可执行程序的实验场。**设计模式（"规格—数据—实现"三层）**：

- **docs/ = 规格**：每篇方法论文档是待实现的规格（输入输出+处理规则明确）。**先人工示范、diff 考古成规格**——如 `fiction/fiction-revision.md` 七条精修原则全部从两轮人工精修 diff 归纳（标【两轮】= 两次独立出现的确认模式），反例也记录（"作者保留判断权：曾一度保留最终仍删"）
- **src/ = 实现**：以 docs/ 对应篇目为验收标准，不跳步、不臆造规格；实现文件 doc comment 首行标注规格路径。`revision.rs` 仅 35 行——SYSTEM_PROMPT 占大半（规格内嵌 + 输出要求"逐条列建议 [规则N] 原文→建议+理由、检查衔接、没有命中不强造"），**AI 定位 = 建议者非执行者（终审归作者）**
- **双入口薄壳**：`lab`（CLI/clap 子命令）+ `lab-gui`（eframe/egui）只做参数解析与 IO，领域逻辑统一 `laboratory_core`（lib.rs）；LLM 层复用 `quanttide-agent` crate，多供应商 env 切换（默认 GLM glm-5.3-flash，`LAB_LLM_PROVIDER=llm/mimo`、`GLM_MODEL` 覆盖）
- **坑**：GLM_API_KEY 未配置时 CLI 返回 401（凭据走 env，不在仓库）
- 与 learning 域同构：人工示范（探路者）→ diff 考古成规格（学习云提炼）→ 程序固化（follower 可执行）——探路者范式的代码域版本
- **创作谈引擎（2026-08-30 新立规格+已实现 v1）**：`docs/fiction/creation-talk-engine.md`——用户思路"**规则可变，提炼规则的程序不变**"：通用提炼引擎从各类创作日志提炼创作谈，与评估器（revision）以规则集文件解耦成流水线。**已实现**：`src/creation_talk.rs`（distill，SYSTEM_PROMPT 只装提炼方法不装具体规则）+ `lab creation-talk <日志...>` 子命令（多文件自动标来源分隔）。**规格与文体演进（多次用户纠正后回滚，勿恢复旧实验）**：规格曾砍 107→23 行（证据分档矩阵/双输入参数/产物模板/排期全是写代码时才需要的事）；后追加**正反规则都收**（反复踩的坑提炼为"别这样做"）与**母题/风格两维度**（写什么 vs 怎写，产出分两节）。**但用户 2026-09-05 对产出彻底否决并回滚整个实验方向**：① 评产出"**方向完全偏离了。输出结果是一个分析器而不是一个完整的创作谈**"（分点罗列+"证据：/操作："标签+母题/风格编号+元评论全是分析器文体）；② AI 自行按"随笔体/代笔人"重写规格与 SYSTEM_PROMPT（未提交）→ 用户"**方向全乱了。丢弃所有和创作谈引擎的代码和文档，退回到之前的原始状态**"→ git checkout 丢弃工作区改动；③ 用户"**也清理**" → data/creation-talk/ 四篇实验产物一并 git rm 提交清理（产物也不留）。**两轮终局（2026-09-05 同日，勿按旧记录判"实验已死"）**：规则集版被否后回滚 + 产物清理（e2cc0b3）；随后用户以方向语「**创作日志 → 程序 → 创作谈**」交付文体定义，并先令"写下你对创作谈的理解"（AI 写 `docs/fiction/creation-talk-reflection.md` 复盘）→ **AI 据用户定义重写规格与提示词为文章体**：提示词角色"代笔人"（以作者第一人称写随笔，含禁止清单：禁分点罗列／标签词／元评论／格言总结／替作者宣布未领悟的领悟），规格首节「什么是创作谈」+ 范本 = 作者自己写的三篇（commit 6b1d6a0，文章体版未跑实测）。**纪律：AI 不得发明文体定义；但用户给出方向语后要立刻据其重写，不要停在"等定义"状态**。曾停留过的中间态：引擎停在 v1（正反规则+母题/风格两维度，commit 568f884），创作谈的文体定义当时完全等用户给出、AI 不再提议任何定义或新文体——该中间态已由用户当日的方向语终结**。教训：① 用户否掉实验方向 = **干净回退**（工作区改动丢弃+数据产物一并清理），不保留半成品等"以后能用"；② AI 对被否产物提出的新定义未经用户确认时，用户说"退回原始状态"指**全部回退**，包括自己刚写的未提交版本；③ 实验产物入库后随实验被否也要清理（AGENTS 临时文件规矩保留——它是通用规矩不依附实验）。**自举三实例（2026-09-05，方法仍有效）**：① 规格自审——把引擎自己的规格喂给 revision 评估器，正确判定"规格非叙事前言、无命中不强造"（判定边界清晰）；② 产出当素材——把引擎自己的产物当素材喂回提炼引擎，产出合并版创作谈，正确元认知（识别输入全是产品非原始日志、声明"证据链经转引抵达日志"）；③ 精修闭环——**未跑通**：长文（11KB）调 revision 持续超时（180/300/590s 三次），API 直连通但 lab 程序挂起——**lab 需加 --timeout 参数/流式处理（工程缺陷待修），非模型问题**。教训：长文评估用后台跑或先分块。**临时文件规矩（2026-09-05 用户要求，已写入实验室 AGENTS.md，实验被否后仍保留——通用规矩）**：实验产生的 /tmp 产物必须整理进 `data/{主题}/{日期}-{描述}.md` 提交，不留仓库外孤儿文件；但产物随后随实验被否时用户会要求一并清理。实测产出与演进史见 `references/creation-talk-engine.md`；产品运营评审框架见 `references/founder-lab.md`
- 完整设计笔记（三层架构/实例一二三/产品运营评审框架/learning 同构）：`references/founder-lab.md`

## 仓库快照 STATUS.md 的体检（2026-09-24 qtdata 实证）

用户指令模式：**「查看 <仓库> 的 STATUS.md，如果没有的话创建一个，过时就更新。」**（同一模式常紧随一条「把 README 也按实际结构修」）。完整配方（格式范本/采集命令/报告形状/陷阱）见 `references/repo-status-maintenance.md`，这里只留三条最容易踩的：

1. **先核对「当初量差距的尺子还成立吗」，再刷新数字**——旧 STATUS 的「战略差距分析」可能按已经作废的业务边界写的：qtdata 旧报告按「平台化终局」算出三方平台/定价权一堆 100% 差距，而 2026-09-04 边界确认后平台型需求归 qtcloud、qtdata 只做组合积木——那批差距**是边界外，不是本仓欠账**。直接沿用旧维度 = 把别的业务线的目标算成本仓的欠账。
2. **两类「脱节」分清**：ROADMAP 写的没做 vs **换了做法没回头改**（qtdata Studio：ROADMAP 要新增两个 package，实际把数据/资产做成了详情页 Tab——事做了，路线写法过时）。写「符合度」列时必须指明是哪一类。
3. **数字实取，版本号按 `git tag` > 工程文件 > 组件 CHANGELOG > commit message 核实**——qtdata 有条 commit 写着 `v0.0.2` 但 tag 不存在（旧 STATUS 照抄错了两个月）。同理 `git tag --sort=-creatordate | head` 会静默截断，权威表用 `git tag | sort -V`。
   落地前 `git pull --ff-only` 确认本地=远端；改完 `chmod 664`；未提交时如实说明（能推就按分层链提交推送，网络不通就说清为什么没推）。

- **「AI 反馈」节（2026-09-24 用户主动要求）**：用户说「你加一段 AI 的反馈，然后去看一下源代码，说一说你的看法，把它写进去」= 在 journal 日志末尾追加 `## AI 反馈（读 <模块> 源码后）` 节。结构 = 三条判断（每条附代码出处：文件路径、grep 结果、行号）+ 一句话结论 + 修法方向。标注为「可反驳分析而非事实记录」，提醒用户哪条站不住可改或撤。判断要基于代码证据，不推测。
- **gallery 案例页写作（2026-09-24）**：用户口述业务线描述（「以我的原始信息为主」）→ 结构化落档，不润色不扩展。分类轴由业务自己长，不套统一模板（数据按加工阶段、课堂按招生方式、云按产品形态、咨询按客户类型）。每页回答三件事：这条业务是什么、分哪几类、每类实际怎么做。口径三条：价格不写、客户不点名、未完成如实说状态。
- **设计提案格式**：结论第一句 + 压缩的理由 + 具体清单与依据（谁放哪、凭什么）+ 需要用户拍板的点。分析过程只在用户问「为什么」时展开。

## 核心原则（来自仓库 AGENTS.md，用户高度认同）
- **最小干预**：仅用户明确请求时操作
- **替换 vs 并存**：修改前确认旧内容怎么处理，不默认替换
- **减法优先**：删除无效内容优先级高于新增
- **目录即语义**：不擅自移除空目录/占位文件（预留目录是有效架构声明）
- **禁止越级**：子仓库里不做父仓库的事（如改父仓库 ROADMAP），反之亦然
- **改内容 vs 改名字**：操作前明确目标是内容还是名称
- **结论先行（2026-08-29 决策讨论实证）**：顾问式回答的结构 = 结论第一句 + 压缩的理由 + 需要用户拍板的点。分析过程（为什么不是别的方案）只在用户问"为什么"时展开；用户要的是"部门架构结论是什么"这种可直接拍板的答案，不是论证过程。**追问"内核/本质不够透彻"= 要求再下钻一层，不是格式问题**：第一层分析（多角度罗列）被评"不够透彻"后，要找的是**一句话能统摄全部现象的生成机制**（实证：从"四个一致性层面"下钻到"把依赖在场的价值固化为独立于人的记录与结构"获认可）——先敢给单句内核命题，用户会用一个词/短语确认或修正（"也就是民主和法治"），确认后立即以用户词为准重写总纲。**推测用户意图后等确认，不自行展开（2026-08-31 learn 意图讨论实证）**：用户给一句方向语（"这符合我们的思路。也就是探路者无 schedule 学习……"）时，先把它作为**用户的命题**沉淀到意图层（intention），不要紧接着做延伸推演（曾推测"学习云=任务管理系统，做完事自动变强"被纠正"不准确。学习云负责记录做事过程中产生的学习效果"——延伸推演越过了用户尚未说出的边界）。正确动作：用户命题 → 沉淀 → 停；用户的下一条纠正（如"与执行云的分工：PR 章程仓库例子"）继续沉淀——意图层只记用户说过的话，不记 AI 的推演
- **用户的类比 = 设计意图**：用户用类比交付决策（"参考大厂的内包子公司"）时，类比即权威模型——按类比的全部特征展开（编制隔离/真实任务/独立简化管理/转正收编），不要把类比当修辞重新论证用户已否定的方向

## 用户协作模式
- 用户用中文，指令极简（"更新 journal"可能指 git pull 而非写日记——先确认语义再动手）
- **疑问句 = 征询判断，不是执行令（三次重犯，最高优先级）**：用户说"X 要淘汰吗""你觉得 X 吗""是否要 X"是在**问你的判断**——先给出分析与判断，再问是否执行。**即使肯定句式也可能是问句**："3R 十分需要淘汰"实为"你觉得 3R 是否需要淘汰"——曾因此误删 5 个文档（git rm + 推送），被纠正"我是问你觉得是否需要淘汰"。删除/重写类结构变更动手前，先确认用户是在下指令还是征询判断（句式含"你觉得/是否/需要吗"即征询）。**第三次变体（2026-08-26 roadmap 实证）**：**"查看/检查"类请求（"查看 roadmap 的 X，是否有过时信息需要修改，是否有最新信息需要添加"）+ 用户的概括反馈（"细节太多，概括核心"）全程都是方案阶段**——检查=要分析清单，概括=要概括后的方案——**都不是执行许可**。曾直接重写 training-base.md 并提交推送，被纠正"我还没同意你修改呢"。检查/审查类请求的产出是**方案**（过时点清单+新增点+修改方案），用户拍板后才动文件
- **秘书模式 → 顾问模式（2026-08-26 用户切换，长期生效）**：用户明确"从秘书模式进入顾问模式"——顾问=主动发现问题、给判断+理由、把关风险、不等指令；汇报给"判断"不只给"选项清单"（我先看/先说该做什么+为什么，你来否决）。但执行权仍在用户——"查看/检查/评估"类请求的产出仍是**方案**（分析清单+修改方案），用户拍板后才动文件；顾问模式不等于可以跳过拍板。**顾问讨论的输出格式（2026-08-29 qtacademy 编组讨论实证）**：决策讨论要**结论前置**——第一句就是结论/立场，理由紧随其后压缩呈现，不分层铺陈多角度分析；用户打断"说了一堆，结论是什么"= 格式否决信号，回答时砍到只剩结论+一句话理由；用户对结论说"no"后停止辩护，等用户的模型输入（类比/约束/新框架）——用户给的类比（如"参考大厂内包子公司"）就是设计意图本身，按类比直接展开落地细节，不再回头推销自己被否的方案
- 结构变更（合并/移动/重写/改名）前：给出分析和方案选项，**让用户拍板**大方向（范围、边界、版本号），再执行；用户提出命名想法时如实评估语义契合度，不盲从也不硬顶
- **细节自主判断（用户纠正过："你自己判断不要偷懒"）**：大方向确认后，细节（具体文件名、命名风格、目录位置、文章归位）自主做出最佳判断并直接执行，不要对每个小决策反复 clarify；在汇报中说明选择理由即可
- 执行中若发现更简单的路径（如"远端已覆盖，无需搬运"），说明后直接采用更简单的——符合减法优先
- 完成后汇报：提交 hash、推送结果、遗留建议
- **完整流程优先于零散诊断（用户纠正过：实验方向很偏，要组织完整流程）**：用户要的是端到端流水线（采集→定义→导入→治理→输出→回流），不是散落的审计/诊断实验。质量实验降级为流水线中的校验环节（如 qtadmin-delib 的 src/ 流水线里 reporter 的辅助指标），不能喧宾夺主。设计实验前先问：这属于哪条完整流程的哪一环？没有流水线就先组织流程再谈实验
- **自己动手执行，不给命令让用户跑（用户纠正过：\"你操作啊\"）**：本机缺工具/依赖时（如 Flutter SDK 未装），直接恢复安装并执行（后台续传下载 + notify_on_complete），不要\"你可以自己跑 X\"把操作推回用户。用户说\"本地有 Flutter\"后仍要自己操作验证——除非用户明确说他来做
- **优先标准工具，勿手工造轮子（用户纠正过：\"为什么不跑 Flutter create\"）**：能用标准脚手架/工具（flutter create、gh repo create）生成的结构就用工具生成（完整平台目录/元数据/测试骨架），手工骨架只是过渡。工具补全后 analyze/test/build 三绿才算完成
- **重大转向要跨层同步**：用户提出关键转向（如课堂即创新引擎）时，按 insight（完整洞察文档，如 insight/qtclass/classroom-as-innovation-lab.md）→ intention（对应文档补章节）→ roadmap（如有对应表述）三层同步，形成互相印证的闭环。insight 层位于 quanttide-tech/data/insight（意图-实现差距分析），是意图的下游消费者。**insight 写作准绳（2026-09-03 实证，recruitment-crisis-as-funnel-strategy.md）**：仓 AGENTS.md 定义"insight 是对未来的预测，不是报告"——每篇必须能一句话读出"预测 X，可被 Y 证伪，若真则改变 Z"；按业务线分目录（qtclass/ 等），文件名断论式英文 slug；**从运营公告/宣传文案预测战略升级的模式**（用户给 brochure 免费学员公告问"预测到了什么战略升级"）：从文档的结构性动作（考核并轨=选人管道、额度稀缺=筛选压力、优先级排序=注意力分配制度化、边界声明=信用防腐、公开坦诚处境=内容获客）读升级信号，写成"标题即断论 + 五重结构（表面动作→实际结构含义）+ 迁移 + 边界（公告承诺≠制度）"；证据压缩为一行来源链接，全文留 journal。**insight 主题重组（2026-09-03 用户"切分内容并按主题重新整理"实证）**：长对话式 insight（如 193 行的 mechanism-design.md，含"要我展开这个吗"对话残留）按机制切分为独立短篇（每篇=断言+机制+迁移+边界），原文 mv 为 archive-*.md 保留；**新建业务线 index.md** 按主题分组（定位/机制/衡量/演进四组）并注明切分来源——目录无 index 时建 index 是合法动作（与"禁自创新文件"不冲突：index 是导航不是新主题）。**策略讨论直接落 insight（2026-09-06 课堂门槛实证）**：用户说"记录到 insight"即把当轮战略讨论（如课堂门槛 vs 招聘问卷的分野）写成该仓 insight 篇目——讨论中用户补充的事实（招聘现行规则是发问卷/准入问卷）要并入分析（问卷 vs 任务的形式取舍），而不是只记最初结论；qtclass/index.md 同步加索引行

## 业务域 roadmap 写法（2026-09-24 三轮纠正沉淀）

写某业务线的产品路线（qtdata/product.md 等）时，核心是**业务域的定义与重组**。正确模型（用户原话）：**① 职能分解与客户视角**——多个职能之间有重合（数据处理流程、项目排期），拆开是为了方便不同职能分别去做；但客户要看的是**项目的 5 个阶段、每阶段某一个角色做某一块的事情、按明确顺序进行**——职能分解是内部的事，客户视角是业务域的事。**② 业务域的定义与重组**——把一整套定义为一个「业务域」：从各职能域**提取**相关内容（数据分析/商务/项目）→ **重组成**一个完整子领域（如「量潮数据」）→ **直接用业务域的概念表达**（不暴露「数据处理模块＋商务模块＋项目模块」的职能拼装痕迹）。

**落地形态** = 逐阶段整合点（每阶段写清谁做什么、哪些职能域内容合并成哪个业务概念）＋ 横切整合点表（同一件事多处出现的统一去向：三套阶段分解统一为一套、交付物只在验收阶段出现一次、工作档案/宣传册/教程/手册都按业务域 5 阶段组织）。**roadmap 只写当前版本**——未拍板的后续版本（v0.3/v0.4 之类）不预先列（用户「0.3 和 0.4 暂不」），规划中的版本等用户点名再加。

**三轮返工实录（勿重蹈）**：① AI 写「业务域组合职能域现成方案」被否——「**业务域要有完整的建模思路，而不是简单的拼装职能域**」；② AI 自发明「领域模型/流程模型/价值模型/组织模型」四层建模框架再被否——「**写作方向还是不对**」（把方向语读成修辞，实为设计意图）；③ 用户给出上述两点后方向才对，再补「列具体的整合点」才收口。**通用教训：给「X 域下定义/建模」类任务，被否后先照录用户原话、等用户给定义框架，不要自己发明建模层次**——与 Report 实体命名（用户拒提名后等定义）、章程写作「理解先行」同构。完整实录见 `references/domain-modeling-lesson.md`。

**主题文件夹的落盘规范（2026-09-29 qtconsult 实证）**：领域路线图下新增主题目录（如 `qtconsult/{self,product}/`）时，四件事一起做——① 该主题 `README.md` 的「项目结构」代码块**登记新文件夹**（结构即真实，漏登记=文档漂移，用户对 README 只做入口的要求与登记目录不冲突）；② 文件按 `{领域}/{主题}/{子题}/{文件}.md` 组织（如 `qtconsult/product/studio.md`）；③ roadmap 仓 `CHANGELOG.md` 的 `[Unreleased]` 记 `### Added` 条目；④ 三层提交推送（roadmap 子模块 → quanttide-tech → 根仓库，逐级核实远端 sha）。\n\n**核心设计取舍写成「待决」，不代拍**：用户说「把你的改进建议写成产品路线图」时，把方向里的未定取舍（如「第一列要不要显性：要 ⇒ 意图先行是完整管理循环；不要 ⇒ 事实先行，仍是顾问逻辑换名字」）落成路线图里的**待决**节，并在汇报里点明「这几条我没替你定」——已定事实与待决事项在文件里必须分开呈现，别把用户才算得清的判断悄悄写成既成事实。

**多包仓库的文档分层（2026-09-25 toolkit 实证，用户两次纠正）**：toolkit 根 `docs/index.md` 记**通用逻辑**（库定位、核心设计原则、可插拔架构——任何语言包都适用）；各 package 的 `doc/index.md` 记**特定逻辑**（该包的目录结构、类职责表、开发命令）。两处互相引用不重复。第一次纠正「3 写到 packages/dart/doc/index.md」（包结构属特定逻辑）；第二次纠正「toolkit 的 docs 记录通用逻辑，各个 package 的文档记录特定逻辑」（分工边界）。判断标准：这段内容换个语言实现还成立吗？成立 → 根 docs；不成立 → 包 doc。

**多平台命题勿混（2026-09-25 实证）**：用户给的 roadmap 可能同时含**产品平台**（studio→qtcloud，工程迁移）和**知识管理平台**（quanttide-founder→iGuo，第二大脑结构）两层——它们是独立的两条线。用户说"不是同一个平台，搞混了"= 结构性纠错：不要把产品侧的「业务域建模」和知识管理侧的「公私结构统一」串成「同一件事的两面」。写多层 roadmap 的看法/分析时，先分清每条属于哪个平台，各自独立评价，不强行找关联。

**git 的 insteadOf 重写坑（2026-09-25 实证）**：仓库级 gitconfig 里可能有 `url.https://github.com/.insteadof git@github.com:`——把所有 SSH URL 静默重写回 HTTPS，导致 `git remote set-url origin git@github.com:...` 后 push 仍走 HTTPS（代理没开时全失败）。诊断：`git config --get-regexp "insteadOf"`；修法：`git config --unset url.https://github.com/.insteadof`。**切 SSH 前先查这条**，否则 set-url 看似生效实则无效。另：代理（127.0.0.1:7897）没开时 `ss -ltn | grep 7897` 无监听，此时直接切 SSH（`git remote set-url origin git@github.com:<owner>/<repo>.git`），不要反复重试 HTTPS。

**规则 + LLM 的文本解析模式（2026-09-25 toolkit memory engine 实证）**：用户设计原则「Markdown 不正则解析正文，而是通过规则说明 Markdown 怎么读，可以用大模型」——正则只管语法结构（标题/段落/列表），语义提取交给「YAML 规则说明 + LLM 按规则填表」；规则文件放 Dart 包 `assets/rules/`（数据不进 lib/doc）。完整模式（三层管线/ParseRule/可插拔提取器/降级方案）见 `references/rules-llm-parsing.md`。

**三段分工：结构存知识、向量管联想、规则做裁决（2026-10-01 反思实证）**——设计检索/识别类功能时，**规则若被推到入口做特征筛选，就顶掉了向量的活**。我当时主张「识别情绪写入点用关键词表+LLM 判断，不需要向量」，被用户驳回（「我觉得有必要用 RAG。反思一下原因」）。两条理由：① 情绪/语义常藏在**平淡叙述**里（「这两天又不太顺利」没有状态词、没有「我害怕」），词表永远碰不到；② 「值不值得关注」本身是**与既有内容的语义近邻比较**（共振／重复／全新），这正是向量的职责。正确位置：结构提供**带坐标的单元** → **向量做联想**（近邻）→ **规则在出口裁决**（阈值丢弃、重复上限标记、段间聚线）。判据：这件事是「识别特征」还是「与既有内容建立关系」？后者必须上向量。

**Dart 包分层（2026-09-29 定型，覆盖 09-25 版）**：库（非 Flutter 应用）按 **feature-first + 域内三文件** 分——

```
lib/src/
├── core/            跨域共享（parse/rules/engine/llm）——域依赖 core，core 不依赖域
├── memory/          models.dart（纯数据）/ repository.dart（装载）/ bloc.dart（工作流编排）
└── fiction/         同上三文件
assets/{artifacts,workflows}/*.yaml
```

**通用共享层叫 `core/`，不叫 `app/`**（用户纠正：app/ 是 Flutter 应用壳的目录名，纯工具库没有应用壳）。**加第三种记忆结构 = 加一个域目录 + 三文件**，core 不动。

**四条设计纪律（用户四轮否决沉淀，详见 `references/bloc-package-layering.md`）**：① 不把分析用的词汇表当代码结构——「六类计算」是分析产物，代码里只有三个真动词（scan/judge/merge）；② 小包不配目录：十来个文件拆五个目录、几十行的类一文件一个 = 摆架子，被斥「这什么玩意」；③ 一个目录一个角色，一个文件一类角色；④ 扩展线分离——加名词加 artifact YAML、加任务加 workflow YAML、加动词才动 core。

**memory 与 fiction 同构（2026-09-25）**：两者都是「用目录命名约定组织的异构 Markdown 文档集合」——结构靠目录名、文件名承载元数据、都有原始→精炼分层、发现逻辑=找特征目录、容忍结构缺失。这是三层管线能通用的根本原因，已写入 toolkit `docs/index.md`。

**fiction 仓库结构（2026-09-25 重构后）**：`草稿箱/` → `观察站/`（1_情绪日记/2_社会观察）；创作日志/创作谈/创作设定迁至 `assets/memory` 的 `write/` 记忆集；各小说阶段目录名不同（职场=灵感/场景/初稿/改稿/定稿，校园=素材/提纲/初稿/改稿，重生=灵感/…/成稿）——Stage 按 `{N}_{名称}` 动态发现不硬编码。章节编号语义：`0_` 前缀=前言不占编号、未编号=替代草稿、预留空号宁空勿移、同一序号可有多份（初稿/改稿/定稿）。example 有 `parse_fiction.dart` 演示入口（输出小说/阶段/章节/观察站解析报告含预留空号统计）。

## 陷阱
- **qtorg 公开站纪律**：site tag 已推送后绝不删重打；v0.2.0 后站点与 profile 档案完全解耦（用户拍板，站点不引用档案、不做对齐）；人物是一等实体（`src/data/people.ts` 数据驱动），人事变更只改 people.ts；发布后配套 = CHANGELOG 归版 + `gh release create site/vX.Y.Z`（见 references/qtorg-site.md「发布后配套」）
- **过时判断要核对战略全景，不凭单一新事实判旧表述过时（2026-08-26 training-base 实证）**：改 roadmap 前认为"免费上课送实习"过时（因新机制"以实际产出代替答卷"），删掉后用户纠正——实训基地是**课堂+招聘双路径枢纽**（免费上课送实习=课堂路径；招聘逆向导流=另一路径，侧重后者）——旧表述是战略的另一侧面，不是过时。教训：用户澄清战略关系（"两条路径都打算支持"）后才知旧表述的定位。改战略/roadmap 文档时：先问/核对该表述在战略全景中的位置（是否多路径/多视角之一），再判"过时 vs 并存"；拿不准就保留并标注，不要删
- **roadmap 组织规范（2026-08-29 修订）**：~~跨领域主题放 intro/（公司级层）~~——intro 层已停用，**跨领域主题全景改放归属领域的意图仓库**（如实训基地 → `domains/quanttide-org/data/intention/training-base.md`，实训基地归属组织管理领域）；roadmap 仓库只保留领域侧视角（qtclass/qtrecurit/training-base.md），与意图仓库全景同名同构、交叉引用不复制。**迁移时的完整性**：① 源仓库 git mv/mv 删除 + qtclass/qtrecurit 领域侧文件的引用改指新位置（`公司级全景见 quanttide-org intention 仓库的 training-base.md`）② AGENTS.md + README.md 的结构图/放置规则/引用规则全改（intro 相关表述逐条清理，`grep -rn "intro" 核对`）③ CHANGELOG [Unreleased] 记 Removed+Changed ④ 目标意图仓库 README 补文档索引 ⑤ 三层提交推送。skill 内 roadmap 规范条目（2026-08-26 定型版）同步作废，勿按旧条目把跨领域主题放回 intro/
- **领域边界判断（内容放错领域比缺失更严重）**：更新领域仓库前先确认内容归属。用户明确过：`domains/quanttide-delib` 只承载议事规则（议题怎么走流程），企业治理实际情况（章程/两院制/设计意图/治理手册）归 `domains/quanttide-org`。跨领域迁移：目标仓库先初始化（README/index/CHANGELOG）→ 复制内容 → 源仓库 `git rm` + README/index 同步 → 各自提交推送 → 父仓库指针（不能 git mv 跨仓库）。边界确认后，领域内"过时点"分析自动排除越界信息
- **契约/领域文档迭代的分层同步**：用户连续给设计指令（如 delib 契约 v0.0.3→v0.0.7：七节点生命周期、四独立实体、研讨/提案两类模式）时，每次变更都要**分层同步**：契约文件 → workflow/流程文档 → index 设计规则 → roadmap 主文档 + 组件文档 → README 索引 → CHANGELOG；改完 `grep -rn "旧表述"` 清理残留（如"九种类型/五节点/无辩论区"在多文件残留）。版本号升级走 `spec_version` 字段 + CHANGELOG 记录
- **契约变更后 grep 清理**：`grep -rn "旧概念" --include="*.md" --include="*.yaml" . | grep -v ".git/"`，逐文件 patch 残留引用（本会话"九种类型→两类模式"清理了 6+ 文件）
- **确认行为：档案命名口径冲突时以详档为准并显性汇报**（2026-08-29 qtacademy 实证）：概览 index.md"职能部"与详档 department.md"综合部"冲突（同一次设计迭代中命名变了但概览没跟上）——统一为详档口径是**低风险一致性修复**（用户"你自己判断不要偷懒"授权范围），直接修 + 汇报理由即可，不属于需要拍板的结构变更。区分：改哪个名字是细节判断（自主），但**不能悄悄只改一边**——两处必须同步
- **"对齐"类指令 = 双向核对任务**：用户说"组织档案和网站对齐"，不是单方向改某一边——先出差异清单（两边各有什么、缺什么、冲突什么），有据的保留、冲突的以权威源（详档）统一、无据的标注待定。曾把对齐做成单向"改站点补内容"并在用户追问"你在做什么"后才意识到还应核对档案侧的一致性缺口
- **实验室验证模式**：领域设计可用 `laboratory/` 文件夹做可运行实验（schema.yaml + 示例数据 + 生成脚本），真实跑一遍脚本验证结构可行性（如决议索引生成脚本 `gen_index.py`：逾期推导 + 治理视图输出），脚本用 `uv run --with pyyaml` 避免系统 Python 缺依赖；PyYAML 会把 ISO 日期解析成 date 对象，解析函数需兼容
- **GLM_API_KEY 获取路径（quanttide-founder 实验室实测）**：实验室 CLI 凭据走 env，key 在 `~/.hermes/auth.json` 的 `credential_pool.zai`（**是数组不是 dict**——`d['credential_pool']['zai'][0]['access_token']`；曾因误判结构 AttributeError 卡两次）。实测流程：python3 读出写到 /tmp/glm_key → `GLM_API_KEY=$(cat /tmp/glm_key)` 跑 lab 命令 → 用完删临时文件。**长调用超时**：LLM 提炼一次可达 1-3 分钟，terminal 前台默认 180s 会掐断（产物空文件）——加 `timeout 300` 或分次跑（一次一篇日志），别批量长时间等待；**11KB 级长文调用 lab 会无限挂起**（lab 无超时控制，2026-09-05 实测 180/300/590s 三次全超时而 API 直连秒回）——长文评估必须 background 跑或分块，不要前台重试烧时间。**请新建临时文件前先确认用户在等什么**：曾为"用创作谈自举改进创作谈"连续跑三轮 LLM 长调用（含两次前台超时 300/590s 空烧近 15 分钟），事后用户根本不需要第三轮——多步 LLM 实验每轮跑之前先对齐"这轮的产出拿去做什么"，别自动连跑
- **创始人实验室定位解读（2026-09-05 "看看现在的功能可以做什么"实证）**：解读实验室功能时按四层产品动机组织——①把"作者本人"产品化（卖创作过程而非作品）②把最贵的人工环节自动化（精修=diff 考古成规则让 AI 承担扫描）③规格-数据-实现三层=方法论→产品的可重复流水线 ④最深=给自己造"另一个不缺席的自己"（与法人记忆同构，自己当第一个客户验证法人级 AI Native 运营）。**提示词考古（用户问"这个提示词为什么这样写"）**：逐特征给出处——规则来自真实 diff、输出格式来自"不做判断只摆材料"哲学、防硬造条款来自真实反例、衔接检查来自踩过的坑——结论=它不是教 AI 写作，是让 AI 模仿"你自己怎么改自己的文章"（编辑指纹只能从自己的行为数据来）
- **quanttide-org data 层（2026-08-29 补齐）**：quanttide-org 原只挂 journal/context/intention 三个 data 子模块（无 profile）；2026-08-29 已补 `data/profile`（新建 quanttide-profile-of-organization-management 并 `git submodule add -b main` 挂载，骨架 README/CHANGELOG/AGENTS/LICENSE，AGENTS.md 沿用 quanttide-product/data/profile 约定含 evaluation.md 档案类型）。org 的 data/intention 与 tech 的 intention 体系不同名——org 是领域级独立仓库
- **data/intention 等子模块显示 `-` 前缀（未初始化）时先 `git submodule update --init <path>` 再 checkout main**——org 的 intention 曾从未检出，直接在空目录 ls 会误判为空仓库
- **子模块指针失效（幽灵指针）**：本地 gitlink 指向远端不存在的提交（远端被 force-push 重写过）——症状 `git submodule update` 报 `fatal: 远程错误：upload-pack: not our ref` / "did not contain <sha>"。诊断：`git ls-tree HEAD <submodule>` vs `git ls-remote <url> main` 对比。修复：`git rm --cached <submodule>` → 删除残留目录 → `git submodule add <url> <path>` 重新挂载到远端 main（可带 `-b main`）→ 提交推送 + 父仓库指针逐级更新
- **手动创建子模块的 remote 坑**：`git submodule add` 报"位于一个尚未初始化的分支"（父仓库 gitdir 链接异常）时，改用 clone+删 .git+gitlink 手动方式。**坑：手动创建的子模块其 origin 会指向父仓库 URL**（不是自己的仓库）——必须 `git remote set-url origin <自己的仓库url>` 并 push 验证；`.gitmodules` 的 url 字段也要同步写正确（曾因远端仓库名错误创建错仓库：`brochure-of-product-development` → 正确名 `quanttide-brochure-of-product-development`，需 `gh repo create` 新仓库迁移内容后再改 remote 与 .gitmodules）。另 `gh repo rename` 语法为 `gh repo rename <新名> --repo <owner>/<旧名>`（不接受双参数位置）
- **邮件发送失败按退信根因处理（2026-09-07 实证）**：`send_status` status=3 时收件箱顶部出现 mailer-daemon"邮件退信"（主题带 Re: 前缀，**别误判成对方回复**）——读 body（**json.loads 解 unicode 转义，不要 unicode_escape**，后者毁中文）取根因：`550 Mailbox not found`=收件地址错 → 从原始来信 `head_from.mail_address` 重新提取重发（实证 <申请人邮箱> 下划线曾漏）；重发后 sleep 15s 再查 send_status 确认 4=送达。**uv 默认清华源 403 时**：`UV_DEFAULT_INDEX=https://mirrors.aliyun.com/pypi/simple/ uv run ...`（实证 qtclass-private 首次 uv run 失败，换阿里源即通）；仓库新 src 包的 `.gitignore` 要补 `.venv/`、`.tmp/`（uv run 自动建 .venv，脚本临时文件写 `.tmp/`）：`git submodule update --init` 克隆失败后目录残留，此时在目录内写文件/commit，git 实际在**父仓库上下文**操作（`git remote -v` 显示父仓库 URL；commit 报"位于未检出的子模组"）。修复：先 `cp` 保存需要的文件 → `rm -rf <子模块目录>` → 重新 `git submodule update --init` → 检出后常落分离头（"HEAD（非分支）"）先 `git checkout main` → `cp` 恢复文件 → 提交推送。**检出后提交前先 `git log --oneline` 看原内容**——曾覆盖 roadmap 原 filter.md（index.md rename 而来）内容，用 `git show <旧sha>:<路径>` 恢复后独立成文件再提交
- **全量拉取后大量小写 `m` = 嵌套子模块指针偏离**：`git submodule foreach --recursive 'git checkout main; git pull --ff-only'` 拉取后，父仓库 `git status` 的小写 `m` 多半是**中层仓库内部嵌套子模块**（如 quanttide-tech/data/insight、quanttide-platform/apps/*）拉取后未提交，不是根级 gitlink 变化。递归处理：对每个 `m` 中层仓库 `(cd $m && git status --short | awk '{print $2}' | xargs git add; git commit -m "chore: update nested submodules"; git push)`，最后根仓库统一 `git add` + commit + push。注意全量 foreach 几十个子模块很慢（曾超时 300s），分步/分轮处理
- **领域仓库 data 层模板化创建**：为领域仓库（如 quanttide-econ）补齐 data/ 子模块（journal/profile/brochure/context）时：① 先 grep 参考仓库（如 quanttide-data）`.gitmodules` 拿命名与格式（`branch = main`）② 远程仓库不存在则 `gh repo create quanttide/quanttide-<type>-of-<domain> --public --description "量潮<领域名><类型名>"` 逐个建 ③ 本地 `git init -b main` + README（一句话定位）+ CHANGELOG（Unreleased/Added/Initialize repository）+ AGENTS.md（参考同类仓库：profile 的 AGENTS.md 抄 quanttide-product/data/profile 的目录结构+约定）+ LICENSE ④ 提交推送（本地 init 在 /tmp 种子目录做，push 后删）⑤ `git submodule add -b main <url> data/<type>` 挂载 ⑥ 提交推送 + 父仓库指针（实证 2026-08-29 quanttide-profile-of-organization-management）

- **新增领域第二大脑（最小形态：只建 context）实证（2026-10-01 quanttide-algorithm 算法工程）**——用户说「quanttide 增加领域第二大脑 `quanttide-<x>`（<中文名>）并创建 context 仓库」时，动作固定为五步：
  1. **命名先对齐**：领域仓 = `quanttide-<缩写>`（缩写用中文名的英文短形，如 `algorithm`）；语境仓 = `quanttide-context-of-<英文全称>`（**全称是把中文「xx工程/xx管理」翻全**——算法工程 → `algorithm-engineering`，对齐 code→software-engineering、knowl→knowledge-engineering、learn→learning-management）。动手前先 `grep -A2 'submodule "data/context"' <兄弟域>/.gitmodules` 校对一遍
  2. `gh repo create quanttide/<两个仓名> --public --description "量潮<中文名>"`（`--description` 用中文，与邻居一致）
  3. **种子目录 `/tmp/seed-*` 做骨架**：语境仓 = README（一句话定位 + 说明 laboratory 语义）+ CHANGELOG（`[0.1.0] 初始化`）+ LICENSE（`cp` 邻居的，全族是 CC BY 4.0）+ `laboratory/`（实验记录 `README.md` + 可复现脚本）；领域仓 = README（定位/功能/子模块表/相关链接）+ AGENTS.md + CONTRIBUTING.md（**直接抄一个成熟兄弟域如 quanttide-code 再全文替换域名词**，闭包：五组目录、获取仓库、子模块游离头、提交顺序、`chore: 更新 <子模块路径> 子模块引用` 格式、文档格式章程）+ CHANGELOG
  4. **`git submodule add` 会写 SSH 地址，但兄弟仓库全是 https**——必须 `git config -f .gitmodules submodule.<path>.url https://github.com/...` 改回并单独提交（否则全族 `.gitmodules` 写法不一致）
  5. **根仓库三处 + 一笔提交**：`git submodule add -b main <url> domains/quanttide-<x>` → 根 README 的「领域轴：N 个领域仓库」计数 +1 → `domains/README.md` 清单按字母序插入 `├── quanttide-<x>/   # <中文名>` → 根 CHANGELOG `[Unreleased]/新增`（**照抄同仓上一条同类条目改名字**，写法/粒度立即一致）→ 只 `git add` 这五个路径，`git diff --cached --name-only` 自查（根仓常挂着别的域没收尾的脏指针，不带）
  6. **验证三级指针**：根 HEAD == `git ls-tree HEAD domains/quanttide-<x>`；领域仓 HEAD == 域仓远端；`git ls-tree HEAD data/context` == 语境仓远端

- **同一域后续补 data 子模块 = 同一套动作的简化版（2026-10-01 同日三次实证：context → intention → insight）**：`gh repo create quanttide-<type>-of-<英文全称> --public --description "量潮<中文名><类型名>"` → `/tmp/seed-*` 种子（README 抄同域已建仓库的措辞 + CHANGELOG `[0.1.0] 初始化` + `cp` 兄弟仓同类型的 LICENSE；内容文件命名对齐兄弟仓同类型仓的实际写法——intention 是 `README.md + <主题>/index.md`、insight 是 `README.md + <slug>.md`）→ push → 在域仓 `git submodule add -b main <本域 url> data/<type>` → **改 .gitmodules 为 https** → 域仓 README 子模块表 + CHANGELOG `[Unreleased]/新增` → 根 CHANGELOG 追加一行 → 根只暂存 `CHANGELOG.md` 与该域路径 → **最后 `git submodule update --init --recursive` 再核对**（见下条陷阱）。命名先 `grep -A2 'submodule "data/<type>"' <兄弟域>/.gitmodules` 校对，别凭印象拼「of-xxx」的英文全称。

- **data 资产的命名、布局与分工速查（同日第四、五个实证：profile）**：命名对照表、各 data 子仓的文件形态、**profile 的两级目录 `data/profile/<产品>/<项>/`**（如 `qtdata/ghtorrent`、`qtcloud-asset/{cli,provider,studio}`）、**档案 ≠ 实验记录（实验过程留 context/laboratory，profile 只写定位/结构/现状/经验并指明事实源，单一事实源不复制）**、README 与 index.md 的分工＋`myst.yml` toc 首项改 index、以及「私有侧不按 `<层>/` 组织、按层做的实验扩不过去」：`references/domain-data-assets.md`

- **资产文档六件套 + 内容归属判据（同日第六~八实证，用户定案）**：给一个资产配文档 = `index.md`（**只做入口**）+ `requirement.md` + `specification.md` + `implementation.md` + `evaluation.md` + `feedback.md`（**留空给用户，一个字都不要代笔**）；判「这段内容该进哪个数据仓」用两层判据 = 性质（陈述性 → `data/`、程序性 → `docs/`、**说不清不硬判**）× 确定度（明确就直接归位、不确定先放 `context/`——`context` 是入口不是终点，所以判错不致命）。定义用**用户自己写的原话**（`quanttide-asset/data/profile/entity/founder/index.md` 的「陈述性=可描述可回忆的事实性知识 / 程序性=可执行可操作的方法指南」），不要另起一套。**同一批文件会在一个会话内被连续改名**（`index.md` → `AGENTS.md` → 又改回 `index.md`，随后拆成四份、再加两份）——用户逐条下改名/拆分令时**逐条执行**：每次 `git mv` 保留历史、每次同步该仓 README 的文档清单与 CHANGELOG、每次走完整分层提交链；**不要合并成一次自作主张的"一步到位"**。完整定义、11/6 类名单、`evaluation.md` 的写法陷阱（业务侧标准 vs 模型侧证据）见 `references/asset-doc-suite-and-placement.md`。

**用户说「把 <实验> 整理到 <域> 的 profile」时，落点是档案不是搬家（本次判断，已说明依据）**：profile 的既有格式是档案（定位/结构/质量/复盘，看 `sec-credit-cleaner`、`uspto-entity-matching`），而实验的过程记录归 `context/laboratory/`——所以写一份**摘要式档案 + 开头一句指明事实源**，不倒复制实验记录；同时把「我按档案理解、若你要的是搬家告诉我」写进汇报。**域内新建第二个及以后的目录层级（如 `quanttide-asset/asset-classifier/`）前，先 `find <兄弟域>/data/profile -maxdepth 2 -type d` 摸清两级各代表什么**（第一级=产品名、第二级=该产品下的项；data 域是「业务线/项目」，code 域是「产品/模块」），别按字面猜。

- **核对 hash ≠ 核对内容（2026-10-04 实证翻车）**：`git add` 的 cwd 我用 `D+"/../../.."` 算错，指到了父目录而非目标仓——改动**压根没暂存**，但随后的 commit 照样生成、push 照样成功（推上去的是「只删未补」的中间态）。而我当时只核对了「本地 == 远端」，hash 相等只证明推成功、不证明推对了。**两条纪律**：①每条 git 命令都带显式 `cwd=`（别用相对路径拼接算仓根）；②push 之后除了核对 hash，还要**核对该版远端文件的内容**（`git show FETCH_HEAD:<path> | grep <关键锚点>`，或匿名拉 GitHub API 读回文件），确认内容就是你要的那份。

- **用脚本连环推三级时，每次 push 后必须 assert 本地 == 远端（2026-10-03 实证翻车）**：写了个 python 脚本依次 commit+push 语境仓→域仓→根仓，语境仓那次推送被拒（远端在我上次推之后被用户在网页上改过一次，非快进），但脚本没检查就继续推父指针，于是域仓记录了一个**从没推上远端的子模块提交**——悬空指针，正是「子模块推送失败立即停」那条纪律要防的坏窗口。修法：先 `git fetch` 看远端多出什么（这里是用户自己改的，要保住）→ `git rebase <远端>` 解冲突（`git rebase --continue` 在 dumb terminal 下会卡编辑器，用 `GIT_EDITOR=true`）→ 推成功并**核对本地==远端**后，再**新开一个 fix 提交**把父指针指到真正落地的那版（不 force-push 已推历史）。**教训：脚本里每步 push 后面都要 `assert local == remote`，不相等就立刻停。**

- **远端被用户从网页抢先推送是常态，不是异常（2026-10-04 一天内两次）**：用户在 GitHub 网页上直接改同一个仓（一次改 `index.md`、一次新增 `intention/index.md`），我的 push 因此被拒。所以 `fetch` → 看 `HEAD..FETCH_HEAD` 多的是什么 → `rebase` 是**常规步骤不是防御动作**。两条实操：① **先看远端多的提交是什么、是谁的**（`git log -1 --format='%an | %ad | %s' FETCH_HEAD` + `git show --stat FETCH_HEAD`）——是用户改的就要**原样保住**，别当成噪声；② **rebase 完多核一步：确认对方的提交还在**（`git show FETCH_HEAD:<对方新增的路径>`）——双方各改各的文件时 rebase 会**静默成功**，静默成功也要核，因为解冲突时最容易把对方那半解没。

- **嵌套子模块的验证陷阱（同日实证，静默且会误判成功）**：`git submodule add` 只克隆一层，域仓内的 `data/context` **是空的**。此时 `git -C <域仓>/data/context rev-parse HEAD` **不会报错**——git 沿目录向上找到域仓，静默返回**域仓的** HEAD 和 origin，看起来"有内容、状态干净"，于是三方指针核对全绿而实际未检出。**判据：嵌套仓未初始化时 `git branch --show-current` 为空或给出父仓分支、且 `git log` 显示的是父仓的提交**。修法：在域仓内 `git submodule update --init --recursive`，再复查 `git rev-parse HEAD` 是否等于域仓 `git ls-tree HEAD data/context` 记的 sha。**凡新增带子模块的仓库，最后一步必跑这个 init + 复查。**

- **同一域追加第二个 data 资产（2026-10-01 算法域加 intention 实证）**：域仓起初只挂了 `data/context`，用户说「**同仓库 intention 文件夹增加：<内容>**」= **先建意图仓再挂载**，不是往已有仓里塞文件（域仓是聚合仓，自身不放内容）。命名 `quanttide-intention-of-<英文全称>`；**套件比 context 更简**——先 `ls */data/intention` 横扫一遍兄弟意图仓看实际套件（多数只有 `README.md` + `LICENSE` + 主题文件夹，有的连 CHANGELOG 都没有）：`# 量潮<域>意图` + 一句话定位 + `## 文档` 清单 + `<主题>/index.md`（主题夹用英文 slug，如 `incubation`）+ CHANGELOG（兄弟仓多无；本域已建则保持一致）。挂载后三处同步：域仓 README 子模块表、域仓 CHANGELOG（`[Unreleased]/### 新增`）、根 CHANGELOG（加一条「`<域>` 新增意图子模块 …，收录首个意图：…」——**照抄同仓上一条同类条目改名字**）
- **意图文档只落用户原话，不推演（本次实证，做对了）**：用户给一句意图（「建立一个算法的自动孵化机制：把通过规则引擎直接写不容易处理的问题，变成算法来处理」）时，成文就是**这句话 + 一个例子**，**不补背景动机、不补建设路径、不补判据**——那些归路线图（与「意图层只记用户说过的话，不记 AI 的推演」同纪律）。落完在汇报里点明「没替你推演、要扩得你给话」，并可以提一个**不越界的下一步**（本次提的是「要不要把实验室那个样本写进『参考』节」——参考节是意图仓既有写法，写指向不算推演）。
- **给一批数据找归宿：选已经在处理这类数据的代码仓，不选主题看起来相近的内容仓（2026-10-01 用户纠正）**——用户否掉「苹果个人记忆」仓时只说了一句「**你选的这个是人类维护的，不是写代码用的**」。做法是先看**哪个仓已经有处理同类数据的工具**，而不是先看哪个仓主题最像。实证：要提取 pi agent 的本机运行时数据（`~/.pi/agent/pi-hermes-memory`：`sessions.db` 105 MB，内为 `memories` 251 / `sessions` 930 / `messages` 13167，另有 `MEMORY.md` · `USER.md` · `failures.md` · `skills/`），归宿是 `Guo-Zhang/thera`（苹果 AI 外脑，Rust 代码仓）——它的 `crates/thera-hermes` 已经在做 `hermes list/export/analyze`、读 `~/.hermes/state.db`，加一个 pi 数据源即可；`thera hermes export` 默认输出 `./data` 的 JSONL，正是「提取」现成的路。配套三条：① **二进制库与活跃 WAL 不进 git**，只把导出物入库；② `skills/` 归 `.agents/skills/`（程序性记忆），记忆正文才归 `.md`；③ `.consolidation-locks/` 与成百个 `.recovery-*`/`.retired-*` 快照是垃圾（git 本身就是版本控制），提取时直接排除。**判据一句话：主题像不像不重要，「谁已经在处理它」才决定归宿。**

  **配套的 crate 拆分（2026-10-01 用户定案）**：我提议扩 `thera-hermes` 时被纠正「**thera 可以，不过应该是 pi crate**」——**每个 agent 运行时一个自己的 crate**（新建 `crates/thera-pi`，不动 `thera-hermes`）。两条依据：① 两库 schema 本就不同（hermes 有 `session_model_usage` / `delivery_obligations`，pi 有 `memories` / `session_files`），硬合会互相污染；② **第二个样本还不够当抽象依据**——不急着抽公共层，等第三个运行时出现再抽（与「模块化借家法」「小包不配目录」同纪律）。接线方式（已核实，别重新摸）：根 `Cargo.toml` 的 `members = ["crates/*"]` 自动纳入新 crate，**不用改**；只改两处——`crates/thera-cli/Cargo.toml` 加 `thera-pi = { path = "../thera-pi" }`，`crates/thera-cli/src/lib.rs` 的 `Command` 枚举加 `Pi { #[command(subcommand)] command: thera_pi::PiCommand }` 变体 + `run()` 的 match 加对应分支。新 crate 照 `thera-hermes` 的形状：`PiCommand` + `run()`，`src/session.rs` 用 `const PI_DB: &str = ".pi/agent/pi-hermes-memory/sessions.db";`（hermes 那边是 `.hermes/state.db`），另加 `src/memory.rs` 读 pi 独有的 `memories` 表。委派 pi 执行、Hermes 验收的协议见 `roadmap-stage-execution` §2f。
- **同会话补齐的 profile 档案链（2026-08-29 完整实证）**：组织档案体系当天从零到完整 = 建立 6 组织 index + README 名单 → qttech/structure.md（两院+三块+矩阵）→ qtalliance 三机构 → qtalliance/structure.md 分离 → qtacademy/structure.md → 根 index.md 组织模式总纲 → 三组织 institution.md + culture.md 同批建立。用户会以**极短指令连续追加**（"qttech/structure.md：秘书处、事业部、共享中心，矩阵式架构"、"qtalliance：联盟理事会、联盟代表大会、联盟秘书处三个主要机构"、"总结量潮组织模式的特色，写到 profile 的 index.md"、"再分别给联盟、公司和实训基地分离出来一篇制度文档和文化文档"）——每个指令都是独立执行令（含明确文件路径=直接执行，不需再拍板），执行时：① 从章程/意图/roadmap 层找制度原文充实 ② 无原文处按治理惯例组织并显性标注占位 ③ 每次三层提交推送 ④ README 目录说明随新增档案类型同步更新
- **本地工作区可能停在分离头+旧历史线，而远端 main 已 force-push 重建**（intention 曾整个仓库只剩 1 个初始提交）。checkout main 后必须**重新读取文件内容**再分析，否则会基于过时内容做方案
- **myst.yml toc 缩进**：子项必须缩进在 `children:` 之下——曾出现 `- file:` 与 `- title:` 平级的缩进错误，导致文件不在站点导航中且 YAML 结构损坏；改 toc 时逐行验证缩进
- **内容仓库（MyST 文档站：docs/* 与 data/* 那批）的增改与验证**：组织规范是 `<主题>/index.md` 作入口 + toc 按主题分组；**`README.md`（给 GitHub）与 `index.md`（站点落地页）应是两份**（那是职责分工）；**产品页 URL 变 `/index-N/` 的根因是 `myst.yml` 缺 `site.options.folders: true`，补 index.md 修不了**（2026-09-24 实证更正，见 reference 第二节）；**项目站的 CI 必须显式设 `env: BASE_URL: /${{ github.event.repository.name }}`**——缺它则 CSS/JS 与站内链接全 404（gallery 实测站点一直是裸页，只查"页面 200"抓不到，要 curl 资源地址）；本地先 `npx -y mystmd@latest build --html > /tmp/log 2>&1` 验证（**别把输出管道给 `head`，SIGPIPE 会截断构建而退出码仍是 0**），再推 GH Pages 并用 `curl -sL -w "%{url_effective}"` 核真实 URL。完整配方（**产品页 URL 退化的真正根因 `site.options.folders: true`（补 index.md 修不了，旧结论已更正）**、兄弟仓 index.md 取素材原则、**产品页写作口径与四条内容纪律**〔分类名取材料正名不自己概括、同一案子别拆成两例、公开页不写价格与客户名、内测口径照实写〕、**内容仓文档套件四份的分工（AGENTS/CONTRIBUTING/STATUS/ROADMAP）与案例库定位纪律（案例≠存档、案例是口径不是文章、分类轴由业务自己长）**、`_build/` 未 gitignore、BASE_URL 取证与修法、上线验证命令、**具体案例页（产品目录下的第二层文件）的落法与「用户贴原始需求」时的补写纪律**、同类欠账清单）：`references/content-repo-myst.md`
- 另一个 agent（`~/.pi/agent/sessions`）也在操作这些仓库，远端可能随时前进——编辑前先 pull
- journal 日志内容规范（CONTRIBUTING.md）：记录事实（时间/人/事）、决策（为什么/依据）、结果（完成度/后续），避免空泛评价
- **日志整理的两条口径（2026-09-24 用户指令「先把这份日志整理一下」实证）**——用户会让我回头整理**别人（AI）已写好的日志**，两条都是硬要求：
  1. **精简 AI 写的进度汇报**：写到「决定 + 结果 + 提交号」为止；实现方式细目（配置加载／持久化／优雅关闭的零件清单、启动方式变更、目录级增删明细）不进日志。**同一件事在多处重复时只留一处**（重复的「互文／收敛」注与复盘同一条合并）——判断哪些算「AI 写的汇报」：带提交 hash 与工程细节的段落就是
  2. **创始人的口述观察（如「人类观察」节）按原意整理成书面语**：保留全部判断与事实，去掉口述碎片、口语填充词与重复（「什么都有」「非常诡异」「yes」「就是」这类不保留；用户说「避免口语化」≠ 可以删意思）。多个并列项（如四个页面各自的定位）并成一段或一列，不逐条堆砌。若原文本是同一口述段落，整理后仍留同一节内，**不擅自拆成新节**（要拆先问）
  3. **AI 反馈/判断可以写进 journal**（用户：「加一段 AI 的反馈，然后去看一下源代码，说一说你的看法，把它写进去」）：节标题 `## AI 反馈（读 <模块> 源码后）`；每条判断附代码出处（文件路径、grep 结果）；结尾标注「可反驳分析而非事实记录」。判断基于代码证据，不推测。
- **整理完要给出改动账**：行数／字节变化 + 逐类说明改了什么（哪节压成几条、删了哪处重复、哪节按原意重写），并点明「哪部分我没动、要不要一起动」——用户据此判断要不要追加指令
- **「总结今天的成果到 journal」（2026-09-24）**：当日工作收尾时用户会说这句 = 在当日 journal 文件末尾追加 `## 当日成果总结` 节，按工作流分条（每条一行加粗主题 + 一句展开，含关键数字如门禁 21/21、版本号），不逐任务罗列细节。写完走三层提交推送
- **roadmap 里混进意图内容时往 intention 迁**（2026-09-24 实证）：`data/roadmap/qtdata/platfrom.md`（平台化转型四阶段）实际是「我们要什么、为什么」= 意图层内容，用户令「platform 移动到 intention 的相关文档里」。判据 = 按 quanttide-tech AGENTS.md 的内容流向（「机制细节进 insight，实现路径进 roadmap，『要什么、为什么』进 intention」）；迁移动作 = 合并进 `data/intention/qtdata/product.md` 对应节（已有重叠内容时只补独有段落，不重复）→ roadmap 原文件删除 → roadmap 建 `product.md` 承载产品迭代（「下一步做什么」）→ 三层提交
- **fiction 仓库实验室模式（2026-09-25）**：创作实验产出落 `实验室/` 文件夹，与人类的 `草稿箱/`（原始素材）分开。实验设计文件命名 `README.md`（用户指定，不用描述性文件名），结果直接写实验室文件夹（不建 result/ 子目录）。实验设计格式 = 目的/假设/步骤（可操作编号步）/标准（通过/不通过表格判据）/产出（落哪、通过后怎么固化）。实证：提炼层验证实验（3 条情绪日记→母题卡片+场景素材→作者判断可写性→试写→对照）——结果不通过，记录失败分析（提炼时从情绪直接跳到场景素材、中间「外部观察」未展开、产出变金句不成画面）。**实验不通过也是有效产出**：记录卡点与下一步方向，不隐藏失败。
- release 类操作：仓库 `.agents/skills/devops-release/SKILL.md` 定义预检（版本格式/CHANGELOG/tag/工作区），实际发布用 qtcloud-devops CLI（见上文"发布版本"节）；发布前必须向用户展示检查结果和待执行命令并请求确认，不可直接执行
