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

## 结论

（实验完成后填写）
