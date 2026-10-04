# Skills 状态报告

> 更新日期：2026-10-04
> 落点：`profile/skills/`（quanttide-context-of-agent-engineering）
> 来源：本机 `hermes/skills/quanttide` 备份，入仓前已脱敏
> 入仓提交：`25a910a`（2026-10-04，68 文件原结构复制）

## 判定口径

一份 SKILL 的质量 = 下次遇到同一件事时能否让 agent 少走一步，按三个可测点判：description 能否被找对、正文能否照做、做完有无验收。本报告只报实取数字与已核实的不一致，判断分三档：

- **可用**：SKILL 正文 ≤ 10,000 字符，翻开即可执行；
- **可用但重**：10,001–30,000 字符，可用但命中成本高，需通读；
- **存档倾向**：> 30,000 字符且正文大半是带日期的会话记录，须先提炼才成为规则。

阅读深度声明：逐字通读的只有 `docs-writing` 与 `quanttide-doc-writing` 两份，其余按结构信号（体积、日期戳、有无验收节）判读，未逐字复核。

## 清单

总量：14 份 SKILL，219,573 字符，68 个文件（SKILL + references + 1 个模板），960K；其中 4 份占 75%。字符为脱敏后口径。

| SKILL | 行 | 字符 | 日期戳 | files | 验收节 | 状态 |
|---|--:|--:|--:|--:|--:|---|
| quanttide-doc-writing | 84 | 2,006 | 0 | 1 | 无 | 可用（已通读） |
| docs-writing | 76 | 2,063 | 0 | 2 | 有 | 可用（已通读） |
| real-data-algorithm-training | 55 | 3,594 | 3 | 3 | 无 | 可用 |
| fiction-creation-analysis | 113 | 4,389 | 3 | 1 | 无 | 可用 |
| quanttide-content-repos | 85 | 4,780 | 2 | 3 | 无 | 可用 |
| quanttide-delib-governance | 119 | 5,584 | 6 | 1 | 无 | 可用 |
| quanttide-fiction-analysis | 126 | 6,307 | 13 | 1 | 无 | 可用 |
| quanttide-product-design | 183 | 7,369 | 8 | 4 | 无 | 可用 |
| insight-writing | 173 | 8,963 | 9 | 4 | 无 | 可用 |
| quanttide-app-design | 122 | 9,314 | 12 | 6 | 有 | 可用 |
| quanttide-cloud-deploy | 214 | 14,685 | 5 | 5 | 无 | 可用但重 |
| quanttide-flutter-studio | 458 | 25,850 | 32 | 4 | 无 | 可用但重 |
| quanttide-app-stack | 552 | 37,522 | 43 | 10 | 无 | 存档倾向 |
| quanttide-repo-workflow | 528 | 87,147 | 141 | 23 | 无 | 存档倾向 |

「日期戳」= 正文中 `YYYY-MM-DD` 出现次数，是会话记录密度的代理指标；「files」= 该技能目录下全部文件数。

## 已知不一致（待处理）

- **断链 3 处**：`fiction-creation-analysis/references/creation-talk-engine.md`、`quanttide-fiction-analysis/references/creation-talk-engine.md` 实际位于 `quanttide-repo-workflow/references/`；`quanttide-repo-workflow/references/insight-verification.md` 实际位于 `insight-writing/references/`。
- **触发面重叠**：`docs-writing` 与 `quanttide-doc-writing` 的 description 同义（写技术/产品文档），`fiction-creation-analysis` 与 `quanttide-fiction-analysis` 同理，命中时无法区分该取哪份。
- **description 句子不通**：`quanttide-repo-workflow` 写作「使用当在 QuantTide 仓库体系查看/更新子模块、journal、intention 文档或发布版本。」，且覆盖面最宽，几乎任何仓库任务都会擦到。
- **同一事实四处维护**：`terraform state rm` 的 state 同步段在 `quanttide-repo-workflow`、`quanttide-cloud-deploy`、`quanttide-app-stack` 三份 SKILL 各写一遍；`BlockPublicAccess` 坑在 `quanttide-flutter-studio/SKILL.md` 与 `quanttide-repo-workflow/references/flutter-studio-init.md` 各写一遍，先过时的那份会给出错参数。
- **规则与自身纪律冲突**：`quanttide-repo-workflow` 正文写「context 无时态」「只能保留最新版本」，同文件带 141 个日期戳。
- **脱敏状态**：已按入仓规则清理，校验口径为 11 个人名与 9 个人物 slug、`quanttide.com`、`/home/iguo`、个人与部门邮箱、5 个存储桶名、群 chat-id 全部 0 命中；保留项为部门/机构名、代号文件夹名（iGuo、Jerry）与公共端点。

## 复核命令

数字必须实取，改动 SKILL 后按下列命令重跑并更新本页：

```bash
cd profile/skills
wc -l */SKILL.md && wc -m */SKILL.md | tail -1
grep -c '2026-[0-9][0-9]-[0-9][0-9]' */SKILL.md
grep -c '^## 验收' */SKILL.md

# 断链复核
for f in */SKILL.md; do d=$(dirname "$f"); \
  grep -oE "references/[A-Za-z0-9._-]+\.md" "$f" | sort -u | while read r; do \
    [ -f "$d/$r" ] || echo "缺失: $d/$r"; done; done

# 脱敏复核
grep -rn "quanttide\.com\|/home/iguo\|@qq\.com\|oc_c02f" .
```
