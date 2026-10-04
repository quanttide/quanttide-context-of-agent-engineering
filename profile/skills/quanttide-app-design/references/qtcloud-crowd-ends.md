# qtcloud-crowd 众包云：端角色确认 + 四端初始化案例

> 2026-08-24 实证。设计 → 文档 → 四端初始化的完整决策链，重点：**端角色两次猜错的纠正过程**。

## 背景

量潮众包 = 自营发销售众包（面向外部渠道与代理，按量潮标准结算）。两个仓库：
- `qtcrowd`（apps/qtcrowd——独立 app）：量潮众包对外站
- `qtcloud-crowd`（apps/qtcloud-crowd）：众包管理云

## 端角色决策链（两次纠正）

### 错误 1：site = 功能介绍页（被纠正）

初始给 `qtcloud-crowd/src/site` 派任务：hero + 人财事三板块（管理方职责说明）。用户纠正：
> "site 是介绍功能的，不应该有数据模型"

**教训**：功能介绍页只做 hero + 功能卡片一句话（任务审核/执行方管理/结算），不展开管理动作/数据字段。kill 已启动的 pi 进程重派。

### 错误 2：qtcrowd studio = 管理端（被纠正）

初始以为两个 studio 都是管理端。用户纠正：
> "qtcrowd studio 是给众包参与人员使用的"

**教训**：同名端在不同产品角色不同——**先问"这个端给谁用"再设计**。用户确认句式："qtcrowd studio 是给众包参与人员使用的"、"现在对了"。

## 最终角色表

| app | 端 | 使用者 | 职责 |
|-----|-----|--------|------|
| qtcrowd | site | 参与人员 | 展示众包信息（任务池——tasks.json 真实数据与 data/profile 同步） |
| qtcrowd | studio | **参与人员** | 参与工作流：看任务 → 认领 → 交付 → 查看结算（本地 my-tasks/my-settlements） |
| qtcloud-crowd | site | 公众 | 功能介绍（hero + 三卡片） |
| qtcloud-crowd | studio | **管理方** | 人财事（任务审核/执行方认证/结算） |
| qtcloud-crowd | cli | 管理方 | 本机操作（tasks/partners/settlements + 模型层校验） |
| qtcloud-crowd | provider | 服务端 | 唯一数据入口（REST API + OSS store） |

## 数据边界

- 参与端本地数据（我的认领/结算——`QTCLOUD_CROWD_STUDIO_DATA` 可覆盖，原子写）**不写管理端数据**
- 两端共用同一 provider（唯一服务端数据入口）；site 静态不依赖；cli 直读同构数据文件
- 真实数据源：qtcrowd AGENTS.md 规定 site 任务清单从父仓库 `data/profile` 同步（sync/validate 脚本保障一致），不自创占位数据

## 文档演进（user-guide 双形态 → 人财事 → 精简）

1. 初版：发单/接单/验收通用指南 → **纠正：众包云是管理方使用的**（不是执行方指南）
2. 管理方版：五块（任务/执行方/结算/信用/市场状态）→ **精简**：砍数据视图（"镜子不是仪表盘"）、合并结算单/台账、信用远期一句
3. 剩三核心动作 → **用户确认"刚好是 人 财 事 三个视角"**（审核任务=事、管理执行方=人、结算=财）
4. 展开三个文档（想象中的样子 + 最精简的样子 + 精简理由）→ 文件名准确化 → 单词风格统一（partners/settlement/review——"像 settlement 统一"）
5. dev-guide 从"最精简的样子"出发：事→人→财 实施顺序（最精简即实施起点）

## 实施（四端并行 pi）

- studio+provider 一个 pi；cli 一个 pi；site 一个 pi（kill 重派过一次）——并行后台
- 每端完成后 Hermes 复跑验证（flutter analyze/test、cargo build/clippy、go vet/test）再提交
- 测试总数 64（Dart 38 + Go 26）+ cli 端到端 + site build/lint 全绿

## 复用要点

- 派 pi 任务前先过一遍端角色表（给谁用/展示什么/操作什么）——两个错误都源于跳过这步
- site 任务描述默认不带数据模型；"展示真实数据"要用户明确确认后才接 provider
- 同名子模块（qtcrowd vs qtcloud-crowd）角色可能完全不同——分别确认
