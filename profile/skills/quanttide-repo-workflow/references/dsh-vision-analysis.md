# dsh 工作区对话 + 视觉模型分析网页 UI

用 dsh（DeepSeek Harness）在量潮领域工作区开对话，分析结果沉淀到领域 data/context。含视觉模型（deepseek-v4-flash-vision-exp）分析线上页面 UI/UX 的完整工作流。

## dsh 升级

```bash
npm view @deepseek-ai/dsh versions          # 查可用版本（npmmirror 源已配置）
npm install -g @deepseek-ai/dsh@0.1.1-rc.1  # 升级
dsh --version                               # 验证
```

## 视觉模型确认（DeepSeek 官方）

```bash
curl -s https://api.deepseek.com/models -H "Authorization: Bearer $DEEPSEEK_API_KEY" | python3 -m json.tool
# deepseek-v4-flash-vision-exp = 视觉模型（2026-08 上线）
```

dsh 支持选择视觉模型（实测 headless 跑通）：`~/.dsh/settings.yaml` 的 `agent-default-model.model`。

## 视觉分析工作流

```bash
# 1. 截图（chrome headless，--virtual-time-budget 等 SPA 渲染）
google-chrome --headless=new --disable-gpu --no-sandbox \
  --window-size=1440,900 --screenshot=/tmp/ui-shot/full.png \
  --virtual-time-budget=15000 "https://<域名>/"

# 2. 临时切换视觉模型
sed -i 's|model: deepseek-v4-flash$|model: deepseek-v4-flash-vision-exp|' ~/.dsh/settings.yaml

# 3. 在领域仓库根目录跑（cwd=工作区），任务里给图片路径
cd ~/repos/quanttide/domains/quanttide-<领域>
timeout 300 dsh --profile headless "分析图片 /tmp/ui-shot/full.png 的 UI 和 UX 设计优缺点"

# 4. 必须恢复默认模型
sed -i 's|model: deepseek-v4-flash-vision-exp$|model: deepseek-v4-flash|' ~/.dsh/settings.yaml
```

## 结果沉淀

- 视觉分析产出：优点/缺点/改进建议（能指出具体文案 bug、层级问题）——直接可作产品迭代输入
- **归属**：交互体验/UI 类分析 → **quanttide-design** 的 `data/context/<app>/ui-ux-analysis-<日期>.md`（用户明确：交互体验=设计领域，不混入 quanttide-product context）
- 结构分析类 → 本领域 `data/context/<主题>.md`，头部标注 `> 来源：dsh headless 会话分析（YYYY-MM-DD）`
- 分层提交：context 子模块 → 领域父仓库 → 根
- context 仓库内同步维护 `.agents/skills/dsh-session-analysis/SKILL.md`（三段论：目标/流程/验收标准）

## 坑

- dsh headless 没有 --model 参数——模型在 settings.yaml 配置层
- 视觉模型临时切换后**必须恢复**（默认 deepseek-v4-flash）
- 截图空白 = SPA 未渲染完，加大 --virtual-time-budget
