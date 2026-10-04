# Flutter Studio 客户端初始化与验证（qtcloud-*/src/studio）

量潮各云应用（qtdata_studio、qtcloud_econ_studio 等）的 Flutter 客户端工作流。

## 初始化流程

1. **手工骨架起步**（可先于 SDK 就绪）：`src/studio/pubspec.yaml`（name 用 `<app>_studio`，参考既有 studio 配置：sdk ^3.11.5、flutter_lints ^6.0.0）+ `lib/main.dart`（应用壳）+ `analysis_options.yaml` + README
2. **用标准工具补全**（用户纠正过"为什么不跑 Flutter create"——必须跑）：
   ```bash
   export PATH="$HOME/flutter/bin:$PATH"
   cd <app>/src/studio
   flutter create . --platforms=web,windows,linux,macos,android,ios --org com.quanttide
   ```
   `flutter create .` 在已有 pubspec.yaml 的目录**保留现有 pubspec/lib/analysis_options**，只补全平台目录（android/ios/web/windows/linux/macos）、.gitignore、.metadata、test/。
3. **修模板测试**：create 生成的 `test/widget_test.dart` 引用模板类 `MyApp`，改为测试实际应用类（如 `EconApp`），否则 analyze 报 `The name 'MyApp' isn't a class`。
4. **注册 asset——目录级**（用户纠正过"种子数据到 assets/data 文件夹"）：种子数据统一放 `assets/data/`，pubspec 注册**整个目录**（`assets: - assets/data/`），后续新种子数据自动打包，无需逐个注册。

## 三绿验证（完成标准）

```bash
flutter analyze        # No issues found
flutter test           # All tests passed
flutter build web      # ✓ Built build/web
```

## 测试要点

- 模型测试：`fromJson` 解析完整对象 + 空列表容错
- widget 测试：`pumpWidget(app)` 后 `pumpAndSettle()`（异步 asset 加载），断言页面标题与真实数据项（如"招聘博弈机制"）
- 异步 asset 加载在 flutter test 中可用（pubspec 注册的 asset 测试环境可读）
- **组件/页面测试分层**（用户要求"增加组件测试"后）：`test/widgets/<widget>_test.dart`（组件渲染/统计/点击）+ `test/screens/<screen>_test.dart`（列表种子加载/详情结构），与模型测试、壳测试组成三层

## 测试 Pitfalls

- **数字断言歧义**：多个统计卡显示相同数字时 `find.text('1')` 会找到多个——用 `findsNWidgets(n)`（如 3 个要素各为 1）或断言唯一数字（如参与者=2）
- **prefer_const_constructors lint**：测试 fixture 构造（`Mechanism(...)`）必须加 `const`（列表元素不加 const，整个构造 const），否则 analyze 报 info 级 issue——analyze 全绿才算完成
- 模板测试类名：create 生成的 widget_test 引用 `MyApp`，自定义 main.dart 后必须改测试（analyze 报 `The name 'MyApp' isn't a class`）

## Pitfalls

- **SDK 下载断点续传**：`~/flutter` 部分下载后，直接跑 `flutter --version` 会自动**从断点续传**（"Resuming transfer from byte position N"），无需重新 clone；首次跑会显示 `Flutter assets will be downloaded from https://storage.flutter-io.cn`（镜像，可信任）
- **平台目录缺失时 flutter run 起不来**：手工骨架只有 lib/ + pubspec 不够，必须 create 补全平台目录
- 版本：Flutter 3.44.9 stable（2026-08）· Dart 3.12.2 · flutter_lints ^6.0.0 兼容
- 网络：`flutter create` 首次可能慢（下载依赖），用 timeout 240+ 前台或后台跑

## CI 与 IaC（发布通道，用户纠正过"iac 严重偷懒了"）

**IaC 必须是 Terraform 声明基础设施，仅 run-studio-linux.sh 本地运行脚本不算 IaC**（用户明确批评）。参考 qtdata（CI）与 qtcloud-data（IaC）模式，发布体系三件套：

### 1. CI：`.github/workflows/deploy-studio.yml`（参考 qtdata，bucket/域名改各自命名）

```yaml
# 触发：push tag studio/*
# 流程：flutter-action(3.44.8) → cd src/studio && flutter pub get && flutter build web
#      → setup-ossutil(aliyun) → ossutil cp build/web/ oss://<app>-studio/ -r -f --meta=Cache-Control:public,max-age=31536000
#      → 入口文件单独 no-cache（manifest.json / index.html / flutter_bootstrap.js——长缓存会让浏览器拿不到新版）
#      → CDN 刷新（aliyun-python-sdk-cdn，RefreshObjectCaches 按 Directory）
# Secrets 需配置：ALIYUN_ACCESS_KEY_ID / ALIYUN_ACCESS_KEY_SECRET
```

### 2. IaC：`manifests/terraform/`（参考 qtcloud-data/manifests/terraform 全套）

```
main.tf                    # alicloud_oss_bucket（acl=public-read + website{index.html}）+ alicloud_cdn_domain_new（回源 OSS）
variables.tf               # region / bucket_name / cdn_domain / cdn_scope（默认 global）
outputs.tf                 # bucket_domain（CDN 回源地址）/ cdn_cname（DNS 配置输出）
terraform.tfvars.example   # 变量示例
bucket_policy_cdn.json     # CDN 回源授权策略（RAM 角色 AliyunCDNRoleForOssPrivateAuth）
README.md                  # 使用 + 前置条件（DNS CNAME 指向 kunlunaq.com、ICP 备案/overseas）+ 踩坑
```

- **关键踩坑**（qtcloud-data 2026-08-08）：新 OSS 桶默认开启【桶级 BlockPublicAccess】→ 需先关闭才能设置 `acl = public-read`；关闭后设 acl + 静态网站托管，根路径 `/` 才返回 index.html
- **命名一致性**：bucket 名、CDN 域名必须与 deploy-studio.yml 的 `oss://` 路径一致（如 `qtcloud-econ-studio` + `econ.example.com`）

### 3. 本地运行辅助（非 IaC）：`scripts/run-studio-linux.sh`

`flutter build linux` → 运行 `build/linux/x64/release/bundle/<app>_studio`（可执行名 = pubspec name，不是 app 名）。

## 领域应用模式（qtcloud-econ 首个模块）

- ROADMAP 放 `src/studio/ROADMAP.md`（第一个目标 = Mechanism 模型 + 展示页面，支撑机制设计模块）
- 领域模型四要素：Mechanism = players（参与者）/ strategies（策略空间）/ rules（规则=结果函数）/ objectives（设计目标）——机制设计理论
- 示例数据来自 journal 真实内容（如招聘博弈：候选人池/自找题/市场化微型创业/三层筛选/招聘给课堂导流），存 `assets/data/mechanisms.json`
- 结构：`lib/models/mechanism.dart` + `lib/screens/mechanism_screen.dart`（列表+详情）+ `lib/widgets/mechanism_card.dart` + 壳侧边栏
- **目录命名用 Flutter 惯例 `widgets/` 而非 `components/`**（用户纠正过"components 改 widgets"；qtdata 老项目用 components 是新项目的反例）。改名用 `git mv lib/components lib/widgets` + 更新 import 引用 + analyze/test 验证
