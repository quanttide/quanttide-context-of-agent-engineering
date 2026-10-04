# qtorg 站点参考（v0.2.0 改组后架构，2026-08-29）

`domains/quanttide-org/apps/qtorg/src/site` — org.example.com，React 19 + Vite + react-router-dom 7。

## 数据层（单一事实源）

`src/data/people.ts`：

```ts
type OrgId = 'qtalliance' | 'qttech' | 'qtacademy'
interface Person {
  id: string            // 拼音 slug，如 '<person-slug>'
  name: string
  primary: string       // 一句话主身份
  titles: { org: OrgId; title: string; desc?: string }[]  // desc 仅当档案有职责原文
  order: number         // 排序权重：创始人→合伙人→治理→顾问
  academic?: {          // 学术履历（可选区块，v0.2.1 起）
    position: string    // 如 '西安交通大学经济与金融学院副教授'
    fields: string[]    // 研究领域徽章
    achievements: string[]  // 项目/论文条目
  }
}
```

- 内容自主维护（2026-08-29 用户拍板与档案**解耦**）：站点不再镜像/引用 profile 档案仓库——人物/职务数据直接在 people.ts 维护，docs 无"数据来源"节。档案结构与站点结构各自演进（档案 2026-08-29 晚重组为 orgs/ + people/ 两组，站点不受影响、无需跟改）
- 组织页人物区块从数据过滤渲染：qttech 按 `title.includes('顾问')` 分管理/顾问两块；qtalliance 按 `'理事'`/`'秘书长'` 过滤分理事会/秘书处两块
- **同步指令的双侧范围（2026-08-29/30 实证）**：用户给履历/人事素材并要求"更新官网"时，**双侧同步**：① 站点 people.ts 加 `academic?` 字段 + PersonDetail 条件渲染（person.academic && 条件渲染，position/fields/achievements 三段，academic-fields 复用 org-badge 样式）；② 档案 people/<slug>/index.md 补「学术背景」（机构+研究领域一段）与「学术成果」（列表）节——虽然两者已解耦（互不引用），但**内容素材级变更**用户预期两边都落（"对齐"语义在内容层面仍存活，废止的是路径引用与自动同步工作流）。素材直接来自用户消息，拆三段落档，不外推不补全。**履历含真实姓名/机构时档案侧照实写**（people/ 档案属内部事实源，某学者学术履历实证：机构全称+研究领域+项目论文如实落档），但**站点侧同样照实写**——站点公开的是用户主动提供的公开履历（学术职称/论文是公开信息），与学员代号脱敏（IM 教学日志）不同类，勿混淆两种敏感度

## 路由（main.tsx）

```
/                      Home（组织卡 + 人物精选前5）
/orgs                  → 同 Home（预留总览）
/orgs/qtalliance|qttech|qtacademy   组织页
/people                人物总览（People.tsx）
/people/:personId      人物详情（PersonDetail.tsx）
/qttech 等旧路由        <Navigate to="/orgs/..." replace>
```

## 页面/组件

- `components/PersonCard.tsx` — 人物卡（链接到详情，组织徽章 org-badge）
- `pages/People.tsx` — 总览卡片墙，平铺不分组（跨组织身份是量潮组织事实）
- `pages/PersonDetail.tsx` — 头部（姓名+主身份+组织徽章链）+ 职务按组织分组（title-group）+ **学术履历区块（person.academic 存在才渲染，v0.2.1）** + 其他人物
- 学术履历先例（2026-08-29 某学者）：人物详情可含对外可公开的学术/履历区块——数据加在 people.ts 的 `academic?` 字段，页面条件渲染；用户给一段履历文字时拆成 position/fields/achievements 三段落档，档案侧（people/<slug>/index.md）同步补「学术背景」「学术成果」节。**档案侧新结构（2026-08-30 起）**：profile 重构后人物档在 `data/profile/people/<slug>/index.md`，履历节为「学术背景」（机构+研究领域一段）与「学术成果」（列表）。**双侧同步的触发边界（2026-08-30 某学者实证）**：用户单独给履历素材（"某某，西安交通大学副教授…"）且语境是"更新官网"时只动站点（people.ts academic 字段 + PersonDetail 渲染）+ 发布（tag/release）——档案侧当天稍后由用户单独指令落（"从日志提炼学习状态"等），不要在同一轮替档案侧做主；用户对档案侧的素材取舍有自己的节奏（先 journal 后 profile 的顺序纪律在 learn 域）
- 组织页 `<org>/index.tsx` — OrgTree/DepartmentCard/RankTable 仍为页内联数据（结构/职级变更手动同步）

## 相对路径层级（易错）

- `src/pages/<org>/index.tsx`（子目录）→ `../../components`、`../../data/people`
- `src/pages/Xxx.tsx`（pages 直下）→ `../components`、`../data/people`
- 诊断 import 解析：`npx tsc --noEmit --traceResolution` grep "Resolving module"

## 幻影 TS 错误

修复 import 后 TS2307/TS7006 仍报 = tsbuildinfo 缓存陈旧：
`rm -f node_modules/.tmp/*.tsbuildinfo && npx tsc -b --force`

## 批量改 TSX 的正则替换坑

用 python3 heredoc 正则替换 JSX 大块（如把硬编码 team-grid 换成数据渲染）时，匹配模式若不含闭合边界（`</div>`），会留下孤儿残块（`<h4>...</h4></div>` 无父）→ TS 编译错误 TS1005/TS2657。每次替换后**必须** `npx tsc -b --force` 全量验证再提交；残块修复用精确 patch（含上下文闭合标签）。

## 验证与发布

```bash
npm run lint && npm run build          # 双绿
npx vite preview --port 4173 &         # ⚠ terminal background=true，前台 & 被拒
curl localhost:4173/people/<person-slug>    # SPA 深链 200
git tag site/vX.Y.Z && git push origin main site/vX.Y.Z   # tag 触发 deploy-site.yml
curl -s https://org.example.com/ | grep -o 'index-[^"]*\.js'   # 新 hash
curl -s https://org.example.com/assets/<hash>.js | grep -c "关键词"  # 内容上线
```

版本条目在 `src/site/CHANGELOG.md`（scope 文件承载），package.json version 同步 bump。发布语义：用户"更新 site"/"开始" = 编辑+tag+部署一体授权；功能改组 minor、内容增补 patch。

## 发布后配套（用户指令实证 2026-08-29：v0.2.1 上线后"补 changlog 和 GitHub release"）

1. **CHANGELOG 归版**：scope CHANGELOG（src/site/CHANGELOG.md）把 [Unreleased] 改写为正式版本条目（`## [X.Y.Z] - YYYY-MM-DD`，Unreleased 期间累积的 Added/Changed 分组整理）；如有多版本未归版（如 v0.2.0 与 v0.2.1 连发），一并在同次整理中拆成各自版本节
2. **gh release create**：`gh release create site/vX.Y.Z --title "site vX.Y.Z" --notes "<markdown 摘录对应 CHANGELOG 版本节>"`——notes 从 CHANGELOG 版本节摘录，不另写内容
3. **连发补洞**：若前一版本（如 v0.2.0）漏建 release，同次补建（`gh release create site/v0.2.0 --notes ...`），release list 核对 Latest 标记
4. 归版 commit 三层推送（site → qtorg → quanttide-org → 根）

## 发布纪律：已部署 tag 不可重打

- **已上线 tag 绝不 `git tag -d` + 删远端重打**（2026-08-29 被用户拦截）——覆盖线上版本历史 = 破坏性操作。上线后又有改动 → 追加新 patch tag（v0.2.0 → v0.2.1）
- 曾在未确认下执行删 tag 重打流程被用户"你在做什么"拦截——发布相关任何 git tag 删除/强推操作前必须显式确认
- 拦截时的正确恢复姿势：本地未推的 commit 留着不推，向用户说明已完成什么/哪步走偏/当前状态/两个选项（追加 patch tag vs 放弃 commit），等拍板
- **恢复后的再授权模式**：用户拦截后给出选项（如"追加 v0.2.1 vs 放弃 commit"），用户回复即拍板——后续按拍板结果独立执行（追加 patch tag 推送部署），不要再回问

## 档案↔站点关系（2026-08-29 终态：解耦，用户方向演进两次）

演进链：镜像/对齐（早）→ 用户"组织档案和网站对齐"（做对齐核查）→ **"站点删除所有引用。不需要引用"（终态：完全解耦）**。当前状态：

- 站点内容自主维护（people.ts 即权威），不引用档案路径、不做对齐核对、不声称与档案同步
- 档案侧同日完成「组织+人物」两组重构（orgs/ + people/<slug>/index.md 一人一档 9 人），用户明示"暂时不动网站"——档案变更不触发站点修改，反向亦然
- **若用户重新要求对齐/引用，等用户显式发起新指令再动**，不主动恢复任何同步工作流
