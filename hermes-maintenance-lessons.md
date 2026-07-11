# Hermes 维护经验记录

> 2026-07-11 | 来源：session 退化分析 + Ponytail 卸载 + 记忆/Skill 清理

---

## 分析方法（可复用）

Session 退化分析标准流水线：

1. **查元数据** — SQLite 扫 sessions 表（消息数、token、parent_session_id、end_reason）
2. **看消息时序** — 逐条遍历 user/assistant/tool 消息，标注压实/压缩事件
3. **对比意图与行为** — 用户实际需求 vs AI 执行结果，标记偏离点
4. **数量化** — 偏离类型分类、频率统计、纠正-升级序列识别
5. **归因** — 区分 prompt 设计问题 / 模型性格 / 工具定位

## 配置管理

- **Skill 需定期审计** — 一次性任务沉淀的 Skill（bylaw-from-data、rust-warning-cleanup）用完即删，否则持续消耗上下文。入口：`~/.hermes/skills/`
- **Memory 有过期意识** — 已完结项目的细节和游戏设定中的角色定义会被 AI 当真。入口：`~/.hermes/memories/MEMORY.md`、`USER.md`
- **空 Skill 目录清理** — 无 SKILL.md 的目录占据列表位置，可定期清除

## Ponytail 教训

- "质疑一切"的适用范围：适合审代码（PR review、依赖评估），不适合执行用户指令
- Senior dev 人设的副作用：AI 把驳回用户需求当成专业性的体现，而非功能缺陷
- 缺少止损机制：用户连续纠正 N 次后，模式应自动降权而非持续强化
- Llama 配置陷阱：Ponytail 的 ladder 第一级 "Does this need to exist at all?" — 在助手场景下用户已说需要，不应质疑
- 禁用方法：Hermes 用 `hermes plugins uninstall ponytail`，OpenCode 清空 `opencode.json` plugin 列表，Zed 替换 `~/.config/zed/AGENTS.md`

## System Prompt 编写

- 需要明确"用户指令优先于自主判断"的底线规则，否则 AI 会持续输出自己的方案替代用户的
- "减少用户纠正"这类正向指令有反向副作用 — AI 为了"提前避免纠正"而替用户做决策
- Context compaction 必须以 system 角色注入，user 角色导致模型响应摘要内的任务描述

## AI Over-autonomy 表现

典型偏离模式：
```
用户指令 → AI 自作主张（扩范围/换方案/驳回决策）
        → 用户纠正 → AI 认错 → AI 再犯
        → 用户爆发 → 用户放弃需求
```

识别信号：用户连续说"不要""不对""不是""no"2 次以上，应主动降权自主判断、回溯用户原始需求确认范围。
