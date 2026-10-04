# lark-cli 读取飞书邮箱申请与多维表格（Bitable）——招聘/课堂漏斗数据链

2026-09-06 实证：查看 <课堂邮箱> 课堂申请 + <招聘邮箱> 招聘投递 + "量潮招聘工作档案" Bitable（准入问卷原文）。lark-cli 1.0.80 实测。

## 一、邮箱申请列表：mail +triage

```bash
# 列出某邮箱的收件摘要（date/from/subject/message_id 表格）
lark-cli mail +triage --mailbox <课堂邮箱> --max 40 --format table
# 注意：无 --limit flag（用 --max / --page-size）；翻页用返回的 --page-token
# 读单封正文：mail +message --mailbox <addr> --message-id <id>；多封用 +messages --message-ids id1,id2
```

- 同一申请人在两个邮箱出现（<招聘邮箱> 投递 + <课堂邮箱> 转课堂申请）是漏斗分流的正常形态——对照两边邮箱才能还原完整链路
- 申请邮件主题自带动词与结构（"申请量潮课堂学员资格 + 已投递招聘申请"、"产品经理-姓名-学校-可实习6个月"）——table 视图足以初筛，正文按需再读

## 二、找到 Bitable：wiki +space-list → +node-list

```bash
# 1. 列出所有 wiki 空间（量潮体系每个"第二大脑"一个 space）
lark-cli wiki +space-list
# 2. 在目标 space 列根节点（如"量潮招聘第二大脑" space_id=7670134825436695751）
lark-cli wiki +node-list --space-id <id> --format json
# obj_type=bitable 的节点即多维表格，取 obj_token
# 3. 全文搜索 drive/+search 需要 search:docs:read scope（未授权时走 wiki 节点遍历替代）
```

## 三、读 Bitable：base 三连

```bash
# 表清单（id/name/records_count）
lark-cli base +table-list --base-token <obj_token>
# 字段清单（列语义 + 选项）——--base-token 不是 --base
lark-cli base +field-list --base-token <obj_token> --table-id <tbl_xxx> --format json
# 记录列表
lark-cli base +record-list --base-token <obj_token> --table-id <tbl_xxx> --page-size 200 --format json > /tmp/records.json
```

## 四、核心坑：record 值是数组的数组，列语义必须校准

`+record-list` 返回的 `data.data` 是**数组的数组**（不是对象数组），且**值顺序 ≠ field-list 顺序**（实测同一张表顺序错位）。jq 用中文键（`.fields["姓名"]`）直接报 `unexpected token`——**不要用 jq 过滤记录，落盘后用 python 处理**。

校准方法（锚点法）：

1. 先 `+field-list` 拿字段名清单（列语义字典）
2. 打印前几行记录的非空值，用**已知事实锚定列**（如"第一行姓名=某申请人、状态含'暂缓（未问卷）'"）
3. 状态类字段是**单元素数组**（`['暂缓（未问卷）']`），link 字段是 `[{id: recXXX}]`——取值统一过一层 normalize：

```python
def cell(v):
    if isinstance(v, list):
        return '；'.join(str(x.get('text', x.get('name',''))) if isinstance(x, dict) else str(x) for x in v)
    return str(v) if v is not None else ''
```

4. 多人定位不要猜列——全文搜姓名关键字命中行后再按锚定列输出（按姓名关键字只命中 1 行：问卷表姓名写的是全名、邮件署名是昵称，**问卷姓名与邮件署名可能不同**，按邮箱交叉才可靠）

## 五、漏斗分析模板（招聘进度表实测口径）

- 状态枚举：投递 / 暂缓（未问卷）/ 群成员（未入档）/ 群成员（匿名）
- 评估分组枚举：未评估 / 观察 / 留用 / 错配可推 / 不推荐 / 双轨 / 待定
- 关键交叉：**"在群 且 未评估"** = 当前积压池；再分"已发问卷未提交"（`问卷发送主题` 非空 + `问卷提交时间` 空）= 跟进/清理名单
- 双邮箱交叉：<招聘邮箱> 投递者出现在 <课堂邮箱> 申请 = "投递后转课堂"（free.md 公告生效的行为信号），招聘进度表建议加"转课堂"状态

## 六、实证结论（问卷 vs 任务作为筛选工具）

问卷提交表 117 份中大量"卷面优秀、信息量趋零"（模板句），唯一有信息量的常是"投递前对公司的了解"（做过功课可验证）；"投递后转课堂"的候选人跳过问卷直接走课堂通道比问卷通道快数周。支撑 insight：招聘筛能力存量（问卷可）、课堂筛投入意愿（行为样本比自我陈述信度高）——详见 insight/qtclass/classroom-gate-vs-recruitment-survey.md。

## 七、申请处理操作链（2026-09-07 实证：甲/乙双案例）

**先对账再操作**——两位申请人都**不在招聘进度主表**（问卷经链接自填、投递未入档），直接在档案建记录而非假设已存在（按姓名关键字全表无命中确认）。邮箱地址从 `+message` 输出的 `head_from.mail_address` 核实（申请人甲=<申请人邮箱>，原值含下划线易错）。

**处理规则（用户拍板）**：已交问卷 → 直接进群；未交问卷 → 先填问卷再进群；任务在量潮课堂学习中心领取，申请与交付发 <课堂邮箱>；群聊 = "量潮实训基地"（<群 chat-id>，+chat-search 可查）。

**进群动作的可行路径**：`im chat.members create` user 身份对外部群报 232033（无 external chat 管理权）、bot 身份缺 scope——**两条替代**：① `im chats link --chat-id <id> --as user --data '{}'` 取 share_link（applink，7 天有效）→ 发邀请邮件让候选人自行加入；② 邀请邮件从 <课堂邮箱> 发（`mail +send --mailbox <课堂邮箱> --confirm-send`），正文含群链接 + 进群三件事（群昵称格式/学习中心领任务/<课堂邮箱> 申请交付）+ deadline 与自然淘汰规则。

**建记录的 JSON 形状**：`+record-batch-create --json` 是 `{"create_records":[{"姓名":"<申请人姓名>","邮箱":"<申请人邮箱>","投递岗位":"数据工程师","状态":"投递"}]}`——**字段 map 平铺无 "fields" 包裹层**（带包裹层报 800010701 "Cell value does not match any supported shape"，逐字段排查也全 FAIL）；`--json @file` 文件须 cwd 内相对路径（/tmp 绝对路径被拒，cp 到工作目录）。

**流程缺口（对账发现，待补）**：问卷自填提交不自动写入进度主表（交了问卷的人档案里查无此人）——问卷表→进度表缺同步动作，人工或自动化补上前每次都要人肉对账。

## 八、升级为 src 包 + 邮件模板资产（2026-09-07 实证：甲/乙/丙三案）

**流程第二次出现即脚本化，第三次升级为 src 包**（用户拍板"examples 升级为 src"）：

- **src 包形态**：`qtclass-private/src/qtclass_private/`（config.py 常量+模板路径 / lark.py 薄封装 / applicant.py 业务流 / cli.py argparse 入口），pyproject `[project.scripts]` 出 `qtclass-ops` 命令，`uv run qtclass-ops <姓名> <邮箱> <岗位> [--survey-submitted] [--survey-url <URL>]`；`src/AGENTS.md` 固化 examples vs src 判定表（使用频率/失效后果/稳定性/生命周期）
- **模板单一事实源**：`data/default/profile/`（invite-mail.html + group-qrcode.png），src 经 `REPO_ROOT = Path(__file__).resolve().parent.parent.parent` 引用不复制——examples 同目录副本模式已废弃（曾因双份副本导致二维码只更新一份）
- **⚠️ 邮件缺口的用户纠正信号（两次实证）**：applink 只在飞书客户端内有效（QQ/163 用户点不开）→ 模板必须含**群二维码 inline 图片**（`--inline '[{"cid":"group-qrcode","file_path":"group-qrcode.png"}]'`，file_path 相对 cwd）；二维码 7 天过期（来自用户下载目录截图 `~/下载/飞书*.png`，复制入库）。**让申请人"先填问卷"不给链接 = 把找入口成本推给对方**——脚本硬校验：未交问卷且无 `--survey-url` 拒绝发送（问卷链接从 <招聘邮箱> 历史邮件正文提取：`feishu.cn/share/base/form/...`）
- **工程化细节**：lark-cli 的 `--json @f`/`--body-file`/`--inline` 全部要求相对 cwd——封装层 `os.chdir(REPO_ROOT)` + try/finally 恢复；`--body-file`/`--inline` **不用 @ 前缀**（@ 是 base 域 --json 专属，mail 域传 `@./f` 报 `open --body-file <!DOCTYPE...`）；临时文件写 `.tmp/`（gitignore 补 `.tmp/` `.venv/`）；send_status 查询前 sleep 15s
- **uv 源 403**：默认清华镜像 403 时 `UV_DEFAULT_INDEX=https://mirrors.aliyun.com/pypi/simple/ uv run ...`
- **用户补充的遗漏物 = 模板缺口信号**：用户两次纠正"你根本没有发链接和群二维码""又是我补充的"——用户手动补进流程的任何东西（二维码/链接/段落）都要当场固化进模板与脚本，并检查同批其他邮件是否有同样缺口，不等用户指出第三次

## 九、GUI 双原型与流程状态机（2026-09-07 后续演进，终态）

**CLI+GUI 双入口**：`applicant.py` 重构为纯逻辑层（`ApplicantRequest` dataclass 入参 + `log` 回调 + `ProcessResult` 返回，含 next_action 下一步动作提示），CLI（log=print）与 GUI（tkinter，worker 线程 + `after(0)` 回主线程写日志框）共用业务流——硬约束写在业务流层，入口自动共享。GUI 验证法（agent 无桌面授权）：`XAUTHORITY=~/.Xauthority DISPLAY=:1 timeout 8 uv run <entry>`，无 Traceback + 8 秒被杀 = 窗口正常存活。

**GUI 分双原型（用户定调）**：**user GUI = 量潮课堂工作台的原型（学员侧），admin GUI = 量潮课程云和学习云的原型（运营侧）**——`gui/user.py`（qtclass-user）：报名 Tab（姓名/邮箱/学校/意向岗位/意向课程，比申请邮件多"学校+课程"=教育信息种子；问卷链接作为报名两步第一步直接给出）+ 我的进度 Tab（按邮箱查进度表）；**学员自助提交只建档+给指引，不自动发邀请邮件**（自助数据、人工推进点）。`gui/admin.py`（qtclass-admin）：原申请处理单窗口归位运营侧。lark.py 补 `list_records`（整行文本过滤，135 行量级够用）；config.py 加 `SURVEY_URL`（问卷链接同源管理）。**双原型共用同一数据契约（Bitable），不新增字段**（学校并入姓名备注）。

**T1 流程状态机已交付**：`docs/handbook/enrollment-state-machine.md`（V0.2）——11 状态 + 14 迁移（触发事件/执行者/副作用）+ 双通道合流（招聘/课堂，channel_transfer 迁移）+ 字段映射（关键剥离：流程状态与评估结论分层，"状态"只答走到哪、"评估分组"剥离为 graded 的属性）+ 5 硬约束（三案例固化）。**V0.2 增"演进方式"节**：流程 = 四种零件（状态/迁移/触发器/副作用），未来升级 = 零件增删改——取消外部环节（问卷）= 删对账类迁移改系统内触发器；取消沟通渠道（邮件）= 删渠道特有副作用（退信确认/SMTP 复制），触发事件改系统内动作；**判断标准 = 这个迁移的触发，人在不在环上**。演进纪律：先改文档评审再改代码；一次只动一种零件；硬约束随迁移表同次修订。

**ROADMAP 过渡期纪律**（GUI 偏离教训固化）：业务仓 ROADMAP.md 必须含过渡期计划——投入分界表（业务流/契约/资产 ✅ vs 入口壳 CLI/GUI ❌ 冻结）+ T1→T4 步骤（T1 规则状态机化→T2 Bitable 契约收敛→T3 资产配置化→T4 工作台 MVP）+ **退出条件**（工作台稳定两周无人工兜底 → CLI 降兜底/GUI 废弃/邮箱转异常通道）。GUI 原型不违反冻结——原型验证信息架构假设，是 T4 的需求规格来源；GUI 需求提出时先问"这是原型验证还是流程功能"。**功能需求先对照 ROADMAP 再动手，方向不符先给判断再执行**（曾单窗口 GUI 直接实现后被判方向不符——GUI 优化的环节正是目标形态里会被消除的环节）。

## 十、方向级重写：ROADMAP 按"流程破碎→筑河床"重构（2026-09-08~10 终态）

**根因追问链（三轮下沉，最终答案非中途层）**：用户连续追问流程升级的本质——第一轮答"状态机+自动化"（机制层，被"不够本质"否）→ 第二轮答"判断数据化不足"（仍非根因）→ **终答：流程破碎——一个学员旅程横跨十个系统（邮箱/飞书表单/Bitable×2/邮件/applink/飞书群/site/delib），断片之间靠人肉缝合，48 处 lark-cli 调用就是 48 个手工补丁**。自动化收效甚微是因为"给断河修闸门"——断片之间的衔接（真正吃人的部分）没有定义，只存在于创始人习惯里。**方向=筑河床**：学员旅程在单一系统（课堂工作台）内连续发生，外部系统逐段降级（邮箱→异常通道、问卷→内嵌或取消、Bitable→档案镜像、群→社区场所）。journal 侧证词（09-07 用户原话"把人从环节里砍掉""正在研究怎么砍得更精确"、09-08 秘书"发邮件多没效率"）与河床方向互证。

**ROADMAP 重写要点**（用户"重新梳理现有文档按这个方向写，这个对了"= 方向性重写全部相关文档）：① 核心判断节放根因（破碎）而非旧答案（状态机）；② 演进路径 P0 图纸→P1 最小连续流程（报名→问卷→进群→领任务单系统内连续完成，优先级最高的下一步）→P2 拆缝合点→P3 制度化；③ 统一判断标准一句贯穿（"每一段还在靠人缝合的衔接，就是下一个要修的河床缺口"）；④ README/AGENTS 同步按新方向重写定位节（README 从一行字变真导航，AGENTS 开头指向 ROADMAP）。**状态机文档身份改标**：从"独立交付物"改为"河床的图纸/流程唯一定义处"——工作台表单=迁移入口、进度页=状态视图、P2 拆缝合点=触发器从人工换原生事件。

**T1→材料两段式（用户纠正后的正确流程）**：整理 journal 移出的内容**全部先进 profile/iGuo/default.md 材料暂存区备用**（不做归宿判断——"判断降级为整理"），认领升级（如 material.md 进 roadmap）是未来独立动作；journal 原文删除只留分流索引。曾错误地直接把规划类内容升级进 roadmap/system-plan.md，被纠正"我让你整理到 profile/iGuo/default 备用，而不是直接整理到 roadmap"后回退（roadmap 的 system-plan.md 已 git rm）。**journal README 已立"整理与移出的惯例"四则**（主题分节/规格移出原文删除留指引/摘要替代/全部先入 default.md），详见 work 域 journal README。
