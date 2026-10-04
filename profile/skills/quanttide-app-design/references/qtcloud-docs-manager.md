# qtcloud-docs 文档管理器设计决策链（2026-08-23）

## 背景

qtcloud-docs（量潮文档工程云）studio 从骨架（services 分层：document_store/document_api/auth_service）按新分层重构。用户旅程：文档列表 → 点开 = 阅读器 → 编辑模式 = 编辑器 → 保存生效。

## 决策链

### 1. 分层收敛（对齐 qtcloud-agent 三件套）

- 旧骨架用 `services/` 分层 → 按 `models + repositories + states + views + screens` 重设计
- 仓储 = DDD Repository（接口 + Local/Api 实现），测试注入内存实现
- states = Cubit 驱动模式切换（reading 默认 / editing）

### 2. Book 级缺失 → DocumentSet 层（用户纠正 × 2）

- 初版设计只有扁平 Document → 用户："只能管理单篇文档，不能管理一个 book level 的"
- 加 Book 层 → 用户："Book 这个名字和 document 不好对齐"（两套词根）
- 备选（文档工程真实术语）：Docset（Dash 术语）/ Publication / Part（篇）→ 用户拍板 **DocumentSet**（同词根 doc-、更少歧义）
- 最终：`DocumentSet（书级：id/title/type/status/chapters）` + `Document（单篇：+ docsetId/order）`——对齐 MyST book 结构

### 3. 两页面合并为单页分栏（用户："为什么需要两个"）

- 初版：DocumentSetScreen（TOC 页）+ DocumentScreen（章节页）两个路由
- 用户质疑 → 合并：**DocumentSetScreen 单页分栏**——"一本书 = 目录 + 正文是一体的"（Notion/语雀/Obsidian 模式）
- 桌面：左栏 TocView + 右栏内容（ReaderView/EditorView 按模式切换）；手机：TOC 折叠为抽屉
- DocumentCubit 并入 DocumentSetCubit（selectChapter/enterEdit/updateContent/save 一个 Cubit）

### 4. 最终结构

```
screens/
├── document_set_list_screen.dart   # 文档集列表页
└── document_set_screen.dart        # 单页分栏（TOC + 内容）
views/  toc_view / reader_view / editor_view
states/ document_set_list_cubit + document_set_cubit
repositories/ document_repository（loadDocumentSets/loadDocumentSet/loadDocument/saveDocument）
```

## 可复用教训

1. 容器/子级命名**同词根对齐**（DocumentSet+Document）——别跨词根（Book+Document）
2. 备选命名先查**领域真实术语**（文档工程：Docset/Publication/Part）再定，选最少歧义的
3. 两级导航（目录页+内容页）合并为**单页分栏**——目录+正文是领域真实一体结构，拆页是"页面导航思维"
4. 页面合并伴随 Cubit 合并（状态归一到单页 Cubit）
