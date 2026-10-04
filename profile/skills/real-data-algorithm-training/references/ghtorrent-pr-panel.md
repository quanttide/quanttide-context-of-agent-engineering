# GHTorrent 日度面板（量潮数据交付物）

已实测跑通的一份真实算法训练数据。用它设计题目时不必重新摸数据，直接引本文的数字。

## 位置

`domains/quanttide-data/data/profile/qtdata/ghtorrent/delivery/daily_pr_authored_merged.csv`（22.9 MB）

同目录还带交付时的**数据处理逻辑说明书**（`代码逻辑说明书2.md`）、采集脚本（`pr_authored_merged_counter.py`）；上一级有 `blueprint.md`（原始需求：基于数万级样本产出每日活动面板）与 `audit-instructions.md`（脱敏审计指引）。数据从哪里来、怎么核对，都有据可查——这一点本身就值得讲（数据集不是凭空掉下来的）。

## 画像（实测）

- **524,485 行 / 34,761 个作者 / 2011-01-02 → 2021-03-06**
- 单位 = `(author_id, open_date)`，**只含当天开了 ≥1 个 PR 的行**
- 列：`author_id, author_login, open_date, authored_pr_opened_count, merged_pr_count, self/org/external/unknown_pr_merged_count, merge_lag_mean`
- 缺失：仅 `merge_lag_mean` 缺 30.2%（那天没合并），其余全 0
- `authored_pr_opened_count` 重尾：值为 1 的占 348,491 行
- 业务背景：客户是高校经管科研工作者，交付物是「GitHub 开发者行为日度面板」

## 四个候选题的实测基线

逻辑回归，特征 = 当天 opened 数 + 星期/月/年 + 该作者的历史合并率（`shift()` 防泄漏）；训练 2011–2019，测试 2020–2021。

| 标签（当天是否有…被合并） | 测试正例 | 全猜多数类基线 | 模型准确率 | 测试 AUC |
|---|---|---|---|---|
| `self_pr_merged_count > 0` | 48.3% | 0.517 | **0.717** | 0.777 |
| `org_pr_merged_count > 0` | 16.0% | 0.840 | 0.856 | 0.852 |
| `external_pr_merged_count > 0` | 3.8% | 0.962 | 0.960 | 0.769 |
| `merged_pr_count > 0`（合计） | 66.3% | 0.663 | 0.683 | 0.678 |

合计档的**训练集** AUC 0.778，测试 0.678——这 10 个点的落差是真实的**时间漂移**，可直接当素材。

## 三档难度（就是教学过程）

1. **自有仓库**（48.3%，近平衡）——第一课。基线 0.517 / 模型 0.717，+20 个点肉眼可见，学员拿得到正反馈
2. **组织仓库**（16.0%）——开始不平衡，逼着看 AUC 与精确/召回
3. **外部仓库**（3.8%）——**准确率彻底失效**（0.960 < 0.962 基线），讲「别只看准确率」的活教材，真数据算出来的，不是编的例子

顺便一句教学话术：自有仓库的 PR 合不合并几乎全由自己决定，组织/外部仓库要看别人——同一份数据里天然含着「谁说了算」这个业务问题。

## 四个教学点

- **时间切分**：随机切会虚高；真实的训练/测试落差（0.778 → 0.678）就是时间漂移
- **特征泄漏**：`merged_*` 四个字段是标签本体；作者的「历史」合并率必须 `shift()` 后算
- **类别不平衡**：外部仓库档 3.8%
- **先立基线**：每档都报「全猜多数类」

## 最小闭环

```bash
cd <上面的 delivery 路径>
uv run --quiet --with pandas --with scikit-learn python3 - <<'EOF'
import pandas as pd
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import roc_auc_score, accuracy_score

df = pd.read_csv("daily_pr_authored_merged.csv", parse_dates=["open_date"]).sort_values("open_date")
df["y"] = (df.self_pr_merged_count > 0).astype(int)      # 第一档：自有仓库
df["weekday"] = df.open_date.dt.weekday
df["month"]   = df.open_date.dt.month
df["year"]    = df.open_date.dt.year
g = df.groupby("author_id")["y"]
df["hist"] = g.transform(lambda s: s.shift().expanding().mean()).fillna(0)
feats = ["authored_pr_opened_count", "weekday", "month", "year", "hist"]

tr = df[df.open_date <= "2019-12-31"]; te = df[df.open_date >= "2020-01-01"]
m = LogisticRegression(max_iter=1000).fit(tr[feats], tr.y)
s = m.predict_proba(te[feats])[:, 1]
print("基线 %.3f | 模型 %.3f | AUC %.3f" % (
    max(te.y.mean(), 1 - te.y.mean()),
    accuracy_score(te.y, m.predict(te[feats])),
    roc_auc_score(te.y, s)))
EOF
```

## 待办 / 边界

- 落成课（`quanttide-course/data/profile/`，按课程规范的 course/lesson/scene/step 层级）还是考核题，取决于用途——**先问用户**，口径不同
- 进公开教材前先确认 profile 仓的可见性；数据集本身源自公开 GHTorrent、作者字段是公开 GitHub login，但对外材料的脱敏口径仍按仓库规矩走
- 同族的其他候选（样本量都不够当第一课，所以这份仍是最优选）：客服问题分类（`quanttide-support/data/journal`，4 篇日志十来例）、提交类型分类（各仓 git log，但 chore 1259 : fix 7 严重不平衡）、课程异常归类（`quanttide-course` 课时 JSON，场景/异常仅 18 对）
