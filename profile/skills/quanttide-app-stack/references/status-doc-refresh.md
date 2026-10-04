# STATUS 文档的刷新（仓库级 + 组件级）

来源：2026-09-24 qtdata 实录（仓库级 `apps/qtdata/STATUS.md` 刷新；`src/studio/` 三件套从零补），以及 2026-09-12 qtcloud-work 三件套。

## 一、契约在哪——尺子的三条出处（先读，不要凭记忆列条款）

| 尺子 | 路径 |
|---|---|
| 契约原型（结构契约 / 依赖契约 / 阶段条款） | `domains/quanttide-code/data/insight/code-agent/contract.md` |
| 平台契约（角色技术栈 / 桶命名 / 域名 / IaC / 门禁 / 可观测） | `domains/quanttide-code/docs/gallery/categories/platform/index.md` |
| 工作流判据 | `domains/quanttide-work/docs/gallery/workflows/code-implement*.yaml` |

找法（路径会变）：`grep -rl "manifests/terraform" --include="*.md" .` 抓平台契约所在仓；`find . -name "code-implement*.yaml"` 抓工作流。

**qtdata 的尺子不是 `docs/dev-guide/stories/`**——那是需求故事（需求来源），不是工程契约。用户问「拿哪份契约当尺子」时，默认答案是上面这三条，与 qtcloud-work/studio 那份 STATUS 同源，横向可比。

## 二、两种 STATUS，别混

| 文件 | 读者 | 形态 |
|---|---|---|
| `<app>/STATUS.md`（仓库级） | 人 + AI | 快照：业务定位 / Scope 状态表（Scope·目录·最新版本·状态）+ 各 Scope 一段 / 战略差距分析 / ROADMAP 进度 / 文档覆盖 / 已知不一致 |
| `<app>/src/<组件>/STATUS.md`（组件级，三件套之一） | AI / 数据 | 条款 × 现状 × ✓/✗/—，每行带可比数字与出处（见 `studio-contract-alignment.md`） |

组件目录下只有 `ROADMAP.md` / `CHANGELOG.md`（README 常是脚手架模板原文）= **缺三件套**，按组件级补 STATUS + TODO + 重写 ROADMAP。同一仓内的兄弟组件（如 quanttide-tech 下 qtclass vs qtdata）格式要统一：头部四行（更新日期 / 仓库 / 最新 commit / 版本记录）+ Scope 状态表 + 各 Scope 一段。

## 三、仓库级刷新流程

1. **先确认快照的新鲜度**：`git pull --ff-only` → `git rev-list --left-right --count origin/main...HEAD` 要 `0 0`。拿旧副本写快照等于白写。
2. **旧文件里的数字一条都不要信**，逐条核：
   - `git log --oneline -10 --date=short --pretty="%h %ad %s"` — 最新 commit 与日期
   - `git tag | sort -V` — **引用的 tag 可能根本不存在**（qtdata 旧 STATUS 写「最新版本 v0.0.2」，仓里只有 `v0.0.1`；提交信息、CHANGELOG、工程文件三处版本号各写各的）
   - 各组件版本从工程文件抄：`Cargo.toml` / `pubspec.yaml` / `package.json` / `pyproject.toml`
   - `find . -maxdepth 2` — **旧文件列的目录可能已被删**（qtdata 的 `docs/{brd,prd,add,dev,ixd,drd,pmd}` 已移除，由 `docs/dev-guide/` 承接）
   - `find . -name CHANGELOG.md` — 各组件版本记录在不在
3. **重读最新 journal**（`data/journal/<业务线>/`）找**新的业务边界定义**——差距分析的尺子可能已经换了。qtdata 9/4 确认「组合积木」后，「平台型需求归 qtcloud」，旧报告里那几项「平台化 100% 差距」直接作废：**拿旧尺子重量一遍是最典型的过时**。刷新时明写一节「尺子已经换了」。
4. 每个组件的进度落到可观测事实（文件数 / 行数 / 有无网络依赖 / CI 跑不跑门禁 / 部署域 / 最后改动日期），不照抄旧表。
5. 末尾「已知不一致」列体检所得：ROADMAP 落后实现、两组件包名互不一致、tag 与版本号不一致、根 CHANGELOG 停更、README 过时。

## 四、组件级体检的取数命令

```bash
find lib -name "*.dart" -exec wc -l {} + | sort -rn | head -8    # 规模 + 最长文件
find test -name "*.dart" | wc -l                                  # 测试文件数
grep -inE "http|dio|shared_pref|sqflite|bloc|go_router|riverpod" pubspec.yaml   # 依赖与选型实况
cat .github/workflows/*.yml                                        # 门禁？域名？CanvasKit？
grep -rn "go_router\|MaterialApp" lib/main.dart                    # 路由实况
```

判定口径（与 qtcloud-work/studio 那份一致，便于横向比）：**单文件 > 250 行 = 越界触发转聚合**；界面那层叫 `widgets/` 而应为 `views/`（借家法要连命名一起借）；`MaterialApp(home:)` 无路由表 = go_router 未对齐；CI 不跑 format / analyze / test = 门禁缺；桶命名 `{产品线}-{用途}`；域名 `{产品}.cloud.example.com`（`{产品}.example.com` 只是迁移期兼容入口，只刷它 = 正式域名未接）；`manifests/terraform/` 缺 = 无 IaC。

## 五、Pitfalls

- **没装工具链时不要写「测试全绿」**：写「<日期> 的记录是全绿，本机未装 Flutter 没复跑」。文档里的数字必须是观测到的，不然是谎报。
- **模板原文 = 没写过**：README 还留着 `flutter.dev/get-started` 链接、`analysis_options.yaml` 注释块未删——可直接判「未定制」，是干净的证据。
- README 与 STATUS 常同时过时，**但只改用户点名的那份**，另一份先报告再问。
- 子模块之间的相对链接在 GitHub 上点不通：引用契约出处用仓库内路径纯文本，别做成 markdown 链接。
- 写完后如实报三件：改了哪几个文件、哪些结论未复跑、是否提交（分层链 `<app>` → 领域/主体父仓指针 → 根指针）。网络不通时不要假称已推送——先探代理端口（`ss -ltn | grep <port>`）与直连两条路（`curl` github.com 与 api.github.com 各试一次，两者可达性常不同）再下结论。

## 六、交付顺序（实测好用）

仓库级 STATUS 先出（用户点名的第一份）→ 报告过时点与体检发现 → 用户点头后再补组件级三件套与 README。一口气全改会淹掉用户真正要的那一份；qtdata 这轮就是「STATUS → yes → README + 格式统一 → studio 三件套」逐轮确认的。

**把选择题问出去、用户回「yes」= 让我自己定，不是让我再问一遍**：qtdata 这轮问了「拿哪份契约当尺子？你指一个」，回的是「yes」——正确动作是按默认（契约原型 + 平台契约）直接做完，并在汇报里说明选它的理由，而不是追问第二次。
