# 语义变更实验记录

## 变更

为 Ponytail 行为规则新增一条：

> **"Do not repeat dependency names in error messages."**
>
> 理由：错误信息中重复的依赖名浪费 token、增加阅读噪音，与 YAGNI 原则一致。

## 语义表达方式

同一份语义，在各工具中的原生表达：

- **Zed**：直接在 `AGENTS.md` 的 Rules 列表中加一条
- **OpenCode**：在共享模块 `ponytail-instructions.js` 的 `getFallbackInstructions` 函数中加一条
- **Hermes**：尚未集成，暂不修改

## 实验数据

### 步骤 1：记录语义意图

耗时：约 30s
操作：写本文件

### 步骤 2：修改 Zed AGENTS.md

耗时：约 15s
操作：在 Rules 列表末尾新增该规则

修改文件：`data/profile/zed/AGENTS.md`

### 步骤 3：修改 OpenCode ponytail 插件（共享指令模块）

耗时：约 10s
操作：在 `getFallbackInstructions` 的 Rules 段落末尾新增该规则

修改文件：`ponytail/hooks/ponytail-instructions.js`（ponytail 仓库，非本仓库）

### 步骤 4：Hermes

Hermes 尚未集成 Ponytail，跳过。

### 步骤 5：提交并同步

耗时：约 60s
操作：
- profile 子模组提交并推送
- context 子模组提交并推送
- 主仓库提交并推送

## 结论

### 耗时汇总

| 步骤 | 耗时 |
|------|------|
| 记录语义意图 | 30s |
| 修改 Zed AGENTS.md | 15s |
| 修改 ponytail-instructions.js（OpenCode） | 10s |
| 提交同步 | 60s |
| **总计** | **约 2 分钟** |

### 观察

1. **零学习成本** — 不需要理解任何抽象层，直接改对应位置的文件
2. **修改点自然分布** — 两个修改点分别在两个仓库中，但各自在"该在的位置"
3. **Hermes 尚未集成** — 如果要加，也只涉及在 Hermes 中加一份，不涉及现有代码改动
4. **git revert 即可回滚** — 每个修改点独立 commit，回滚不影响其他工具

### 与"统一抽象层"方案对比

如果采用统一格式 + 转换器：
- 需要设计抽象格式
- 需要为每个工具写转换器
- 工具升级后需要更新转换器
- 本次修改工作量预计是 2 分钟的 3-5 倍

**结论**：语义管理（同一份概念在各工具中以原生方式表达）在当前规模下比统一抽象层更高效。工具数量增长到 10+ 时可重新评估。
