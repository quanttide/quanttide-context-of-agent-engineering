# qtfiction 阅读自适应案例（2026-08-23）

小说发布平台（quanttide-founder/apps/qtfiction，线上 fiction.example.com）阅读页自适应决策链。

## 需求演进（用户连续纠正）

1. 最初提"手机端更好阅读体验"→ 我给了响应式方案（media query 断点 + 排版/进度/字号/主题）
2. 用户问"能否得到专业阅读器"→ 诚实评估：方案只到 60%（缺字体/划线笔记/翻页模式/设置面板/同步）
3. 用户："翻页效果必须" + 记录方案
4. 用户："二三批不要，能正常阅读就可以。怎么手机端自适应"——**砍掉专业阅读器功能**
5. 用户纠正："你的方案是响应式，不是自适应"——**响应式 ≠ 自适应**

## 关键纠正：自适应 = 设备决定交互形态

- **响应式**：同一布局随宽度缩放（media query）——手机也能滚、桌面也能翻
- **自适应**：**两种阅读模式组件**——手机 = 分页翻页（微信读书式沉浸），桌面 = 连续滚动（网页惯例）

```ts
// src/data/device.ts
const isTouch = window.matchMedia('(pointer: coarse)').matches
// 渲染：{isTouch ? <PagedReader /> : <ScrollReader />}
```

## 双模式实现要点

- **PagedReader（手机）**：隐藏 measurer 容器按段落 offsetTop 与可容纳高度分片；`scroll-snap-type: x mandatory` 横向轨道左右滑动翻页；"第 X 页/共 Y 页"；resize/orientationchange 重新分页
- **ScrollReader（桌面）**：连续滚动 + 限宽 45rem 居中
- 闭包陷阱：scroll 监听用 ref 引用最新 flush（否则章节切换后进度写入旧章节）

## 进度缓存（localStorage 零依赖）

- key `qtfiction.progress.v1`，按系列分组：`{ 系列id: { file, page, updatedAt } }`
- 手机 page=页码；桌面 page=滚动百分比（两模式共用结构）
- 写入：翻页/章节切换防抖 300ms + 卸载兜底；读取：Read 页自动恢复 + 系列页"继续阅读"入口
- 封装 getProgress/saveProgress（try/catch 容错——隐私模式静默失败）
- 为什么不用 IndexedDB：进度是轻量状态；多端同步需服务端——砍掉验证

## 范围收敛教训

"能正常阅读就可以" = 只做 排版（17px/行高1.9/首行缩进2em/safe-area）+ 翻页（必须）+ 进度缓存。字体选择/划线笔记/设置面板/同步/社交全砍——专业阅读器功能不是目标，验证先行。
