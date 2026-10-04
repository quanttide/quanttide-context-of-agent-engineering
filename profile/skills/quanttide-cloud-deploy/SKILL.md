---
name: quanttide-cloud-deploy
description: "量潮产品部署阿里云（OSS/CDN/FC/DNS）与 CI 部署流水线排障"
---

# 量潮云部署（CI + IaC + 证书 + 上线）

## 触发条件

- 部署量潮产品（site/studio/provider）到阿里云：OSS 静态托管 + CDN + DNS
- 配置/排障 .github/workflows/deploy-*.yml 部署流水线
- 签发/绑定 HTTPS 证书、处理 CDN 回源问题

## 部署架构（标准链路）

```
<product>/* tag → Actions → terraform apply（创建 OSS/CDN/DNS）
  → build（npm/flutter/go）→ ossutil cp 上传 → CDN 刷新
证书：acme.sh 签发 → aliyun cdn SetCdnDomainSSLCertificate 绑定
```

## 工作流：排障先查历史方案（用户纠正实证）

**"这个问题其他仓库反复解决过"**（2026-08 qtfiction 部署）——部署遇阻时**不要从零猜/问用户**，先查已上线仓库的部署配置：

```bash
# 历史方案在成熟仓库的 IaC/CI 里（qtcloud-agent/qthealth/qtfounder）
grep -rn "alicloud_cdn_domain_config\|l2_oss_key\|private_oss_auth" <成熟仓库>/manifests/terraform/
```

实证：qtfiction 桶公共读被"阻止公共访问"拦截 → 我先试了 set-acl/bucket-policy（都失败）才想起 qtcloud-agent cdn.tf 的 `l2_oss_key private_oss_auth=on` 私有回源方案（第 8 节）——**先查历史，一轮解决**。查证顺序：terraform 配置 → 部署 workflow → references/ 排障记录。

## 关键模式与坑（必须遵守）

### 1. terraform apply 防卡死

```bash
terraform apply -input=false -auto-approve -var ... 
```
- **必须加 `-input=false`**：缺必填变量时 `-auto-approve` 会交互等待输入→CI 卡死 20 分钟（真实事故）
- 所有无默认值的 variable 必须显式传 `-var`（如 `image=placeholder`）

### 2. 共享 RAM 资源冲突 → -target

`AliyunCDNAccessingPrivateOSSRole` 等 RAM 角色/策略是**账号级共享**（被其他产品部署创建过）→ 再次创建报 `EntityAlreadyExists` 409。
- 处理：apply 加 `-target=` 只建本项目资源（OSS 桶/CDN 域名/config/DNS），跳过 RAM 与 FC
- 新加 CDN 域名时同样用 -target 只建该域名资源组

### 3. ossutil 部署

- action 用 `manyuanrong/setup-ossutil@v3.0`（**v1 不存在**，会直接失败）
- 上传语法（v3）：`ossutil cp <src> oss://bucket/ -r -f --meta=Cache-Control:...`
- **源站必须与上传桶一致**：CDN 源站指向空桶 = 全站 404（qthealth 真实事故：cdn.tf 源站 <appA>-studio 但上传 <appA>-site）

### 4. 证书（acme.sh + 阿里云 CDN）

- 泛域名证书 `*.example.com` **只匹配单层子域**（health.example.com ✅ / studio.health.example.com ❌）——两层子域须 acme.sh 签发单域名
- 签发：`acme.sh --issue -d <域名> --dns dns_ali --server letsencrypt`（需 acme.sh account.conf 有 Ali_Key/Ali_Secret）
- 绑定：`aliyun cdn SetCdnDomainSSLCertificate --DomainName <域名> --SSLProtocol on --CertType upload --SSLPub "$PUB" --SSLPri "$PRI"`
- **坑**：acme.sh 目录名含字面星号（`*.example.com_ecc`）——cat 时必须引号保护，否则 PUB 为空报 `InvalidCertificate.TooLong`
- 绑定后等 30-60s 生效；验证 `curl https://<域名>`（handshake failure = 证书未绑定）

### 5. CI 流水线必备步骤

- `hashicorp/setup-terraform@v3`（CI 环境无 terraform）
- 新仓库需确认 org secrets（ALIYUN_ACCESS_KEY_ID/SECRET）继承可用

### 6. 部署后验证

```bash
curl -sI https://<域名>          # HTTP 200
grep -o "<title>[^<]*</title>"   # 内容正确
```

### 7. 从模板复制适配新产品（qtcloud-agent 双站 2026-08-21 实证，3 个坑）

复制成熟仓库的 IaC/CI 做新产品时，批量替换域名/桶名之外还有 3 个**静默坑**：

- **DNS `rr` 独立值漏改 → NXDOMAIN**：批量替换只命中完整域名串（`health.example.com`→`agent.cloud.example.com`），但 cdn.tf 里 DNS 记录的 `rr = "health"` / `rr = "studio.health"` 是**独立值不会被替换** → DNS 记录指向旧域名子域，新域名 NXDOMAIN。修复：替换后**单独检查 alidns_record 的 rr**（`rr = "agent.cloud"` / `"studio.agent.cloud"`），重新打 tag 触发 apply 更新记录
- **漏 platform.tf（data source）→ `Reference to undeclared resource`**：复制模板若只挑部分文件，漏掉 `platform.tf`（`data "terraform_remote_state" "platform"`），oss.tf 引用的 `data.terraform_remote_state.platform.outputs.resource_group_id` 直接报 undeclared；同理 outputs.tf 引用被删 fc.tf 的资源（`alicloud_fcv3_function.this`）也报错——**复制后 grep 一遍全部 `data.` 与资源引用**，确认引用源都在
- **site 桶资源缺失 → NoSuchBucket**：deploy-site.yml 的 `-target` 只含 studio 桶时，上传报 `NoSuchBucket`（<appB>-site 桶从未被创建）——确认每个上传桶都有对应 `alicloud_oss_bucket` 资源且 **-target 列表包含它**（qthealth 当时桶是手动/历史存在，新产品必须显式补 site-bucket.tf）

### 8. 新桶"阻止公共访问"→ 私有回源（2026-08 qtfiction 实证）

2024 后新建 OSS 桶默认开启"阻止公共访问"——`set-acl public-read` / bucket-policy 公共读都报 `Put public bucket acl/policy is not allowed`。**不要尝试公共读**——标准方案：

- 桶保持 `private`（ACL 不动）
- CDN 配三个函数（`aliyun cdn BatchSetCdnDomainConfig --DomainNames <域名> --Functions '<JSON>'`）：
  - `l2_oss_key`：`private_oss_auth=on`（OSS 私有回源鉴权）
  - `back_to_origin_url_rewrite`：`source_url=^/$ target_url=/index.html flag=break`（根改写）
  - `https_force`：`enable=on`（强制 HTTPS）
- 私有回源生效后浏览器经 CDN 正常读私有桶（qtcloud-agent 的 terraform `alicloud_cdn_domain_config` 同款配置，CLI 等价物就是 BatchSetCdnDomainConfig）

### 9. 证书绑定 InvalidSSLPub 的另外两个变体

- **`tr -d '\n'` 删换行 → InvalidSSLPub**：PEM 必须保留换行，直接 `$(cat fullchain.cer)` 传参
- **glob 匹配多个目录 → 拼接多张证书 → InvalidSSLPub**：`~/.acme.sh/*.example.com_ecc/` 会匹配 agent.cloud/health.cloud 等全部 `*.example.com_ecc` 目录（`*` 是通配符）——必须单引号字面化 `~/.acme.sh/'*.example.com_ecc'/fullchain.cer`

### 10. SPA 深链 200（无扩展名 key）

OSS 静态托管深链默认 404（error_document 只回 404.html 状态码不变）。让深链返回 200：`dist/index.html` 上传为**无扩展名 key**（`--meta Content-Type:text/html`）。路由集合须**动态列举**（如 `for s in $(ls -d data/series/*/)` 生成 `/series/<id>`、`/read/<id>` key）——只传一级 key 时更深路径仍 404。

### 11. workflow 在 tag 之后提交 → 删 tag 重打触发

部署 workflow 文件若晚于发布 tag 提交（首次部署常见），tag push 不会触发部署。修复：`git tag -d <tag>` + `git push origin :refs/tags/<tag>` + 重打 + `git push origin <tag>`。另：CDN 首次添加域名 `configuring` 需 5-30 分钟（轮询 `DescribeCdnDomainDetail` 的 DomainStatus=online 再触发部署）；CNAME 也在该接口的 Cname 字段。

### 12. provider + site 双流水线并发 → NoSuchBucket 竞态（2026-08 qtcloud-crowd 实证）

新产品同时建 provider（Terraform apply 建桶）和 site（deploy-site 上传）时：**site run 触发早于 Terraform 建桶完成 → 上传报 `NoSuchBucket`**（不是 -target 漏桶，是时序竞态——Terraform apply 在 provider run 里跑，site run 独立并发）。修复：先确认桶存在（`aliyun oss ls oss://` 看新桶）再**删 tag 重打触发 site**。经验：首次部署时 provider 的 Terraform apply 先完成（它会建 site 桶），site run 失败后重试即可——不是配置问题。

### 13. qtcloud-devops release publish 的 CHANGELOG 规则（2026-08 qtcloud-crowd/qtcrowd 实证）- `publish` 子命令**不支持 `--changelog` 参数**（报 unexpected argument）——版本条目检查自动按 scope 找文件
- 需要**根 CHANGELOG.md 索引** + **各 scope CHANGELOG 有对应版本条目**（`## [x.y.z]` 行）：site scope 读 `src/site/CHANGELOG.md`（不是根文件！qtcrowd 曾把条目写根 CHANGELOG 导致"未找到版本记录"）
- audit 输出里未发布 scope（cli/studio/root 显示"最新版本: (无)"）是**正常提示不是失败**——tag 已创建就算成功（`git tag -l` + `gh release list` 确认）
- 根 CHANGELOG 若用 sed 批量替换标题会改乱条目归属（provider 标题变 site）——写文件时逐条核对 scope 标题

### 14. 手动删 OSS 资源后必须 terraform state rm（2026-08 qtcloud-crowd 实证）

用户图纸变更要求删除已建桶（如后台公开桶）——代码改 terraform 移除资源**不等于真实桶删除**（未 apply）：
- 手动删桶：`aliyun oss rm oss://<bucket> -r -f` 清对象 + `echo "y" | aliyun oss rm oss://<bucket> --bucket` 删桶本身（`--bucket` 参数交互式确认，需管道）
- **state 同步**：本地 `terraform init -backend-config="bucket=<org>-terraform-state" -backend-config="key=<product>/terraform.tfstate" ...`（需导出 ALICLOUD_ACCESS_KEY_ID/SECRET 环境变量——terraform 不读 ~/.aliyun/config.json；AK 从 `python3 -c "import json;d=json.load(open('~/.aliyun/config.json'));print(d['profiles'][0]['access_key_id'])"` 取）→ `terraform state list | grep <资源>` 确认 → `terraform state rm <资源地址>`——否则下次 apply 尝试删不存在的桶报错
- 删桶前先 `aliyun oss ls oss://<bucket>` 确认对象数（0 才能删桶）

### 16. API 网关配置（OpenAPI 签名调用，2026-08 qtcloud-crowd 实证）

FC 函数接入系统级 API 网关（`api.example.com/<product>/...`）时：**aliyun CLI 没有传统 API 网关（apigateway）产品命令**（只有云原生 apig——网关是传统 alicloudapi.com 子域时不可用），且 PyPI 无 apigateway 产品 SDK（alibabacloud_tea_openapi 有 py3.12 兼容 bug）。可靠路径：**Python 手写 RPC 签名**（hmac-sha1 + 参数排序，见 `references/api-gateway-config.md` 的脚本模式）。

流程：`DescribeApiGroups` 找组（qtcloud 组 `34c138c4bec1405d942a57d9bb5ede37`）→ `DescribeApis` 列 API → `DescribeApi` 拿现成 API 作模板（ServiceConfig/RequestConfig）→ `CreateApi`（改 ApiName/RequestPath/ServicePath/ServiceAddress=新 FC URL）→ `DeployApi`（StageName=RELEASE）→ `curl https://api.example.com/<product>/api/tasks` 验证。惯例：terraform `outputs.tf` 记录 apigateway_domain + apigateway_apis 清单（网关本身手动配，IaC 只记录）。路径参数 API（`{id}`）需带 RequestParameters/ServiceParameters（从带参模板复制）。

### 17. 发布前必查 tag/发布历史 + 发布授权纪律（2026-08 alpha.2 重复发布 + 0.2.0 两次越权教训）

**发布前必须查**：`git tag -l "provider/*"`（本地）+ `gh release list --repo <repo>`（远端）——确认版本号未被使用。qtcloud-crowd 曾对已发布过的 `provider/v0.1.0-alpha.2`（18:33 发布）在 19:18 重复发布——qtcloud-devops 输出"标签已创建"但远端 tag 未被覆盖（指向旧提交），却创建了重复 GitHub Release（需 `gh release delete` 清理）。另有：**版本 tag 指向的 release 提交可能不含其后新代码**——发布后检查 `git log -1 <tag>` 确认内容，新代码（如后续清理）须升版本号（alpha.3）而非复用。

**发布授权纪律（两次实证，qtcloud-course 0.2.0）**：**"发版本"（qtcloud-devops release publish——创建 tag + GitHub Release）是独立动作，必须用户明确说"发版本/发布"才执行**。用户说"你配置你部署"、"部署"、"使用 Terraform 配置和部署"都**不等于**授权发版本——部署（terraform apply/CI run）与发版本是两件事。**版本号也归用户决策**（用户会主动问"provider 目前版本到哪里了"来评估——发版本前把版本建议给用户拍板，如"建议 provider/v0.2.0（存储层 minor）——要发吗？"）。

**版本号语义（用户纠正实证）**：判断 major/minor/patch 看**对外 API 是否变更**——内部实现变更（如存储层 SQLite→OSS 替换）但 REST 接口不变 = **patch 级**（不是 minor——曾错判 0.2.0 被纠正"没有变 API"）。用户偏好**预发布**（alpha/beta）先验证再正式版。

**版本记录权威 = git tag（用户实证"以 tag 为准，补 release"）**：
- 发布记录以 `git tag -l` 为准——存在 tag 缺 Release 时**补齐**（`gh release create <tag> --notes "以 tag 为准补齐 Release"`）
- **正式版标记 Latest，预发布不标**（gh 创建顺序会自动把后建的预发布标 Latest——用 `gh release edit <正式版> --latest` 纠正）
- **线上镜像 ≠ 用户发布的版本**（实证：ACR 镜像 v0.1.3 无 tag 无 Release——用户否认"不是我发的"）——查"用户发的最新版本"看 `gh release list` 的 Release 记录，不看镜像/tag 裸记录越权误发后的完整回退：
```bash
gh release delete <tag> --yes --repo <org>/<repo>   # 删 Release
git tag -d <tag> && git push origin :refs/tags/<tag>  # 删本地+远端 tag
# CHANGELOG 已提交的版本条目改回 [Unreleased]（版本未发布不该有已发布条目）
```
已触发的部署 run 删 tag 后停不掉（会完成——幂等无害，后续正式发布复用）。

### 18. FC 无持久化：SQLite/本地文件在容器临时盘（2026-08 qtcloud-course 实证）

FC 3.0 容器只有 disk_size（512MB 临时盘）——**容器重建/实例回收后容器内文件（SQLite DB_PATH、本地 JSON）全部丢失**。部署到 FC 的服务：
- 数据持久化须挂 NAS（FC 挂载点）或数据走 OSS（provider 的 store=oss 模式）
- 依赖本地 DB 的服务（qtcloud-course provider SQLite）**生产内容升级被阻塞**——二选一：NAS 挂载（推荐）或"导入即部署"（每次部署从种子导入）
- 验收 provider 架构时先问"数据在哪持久"——内存/临时盘方案只能用于开发验证

### 19. terraform plan YAML 续行符坑（2026-08 qtcrowd 实证）

GitHub Actions `run:` 块里 `terraform plan ... \` 换行续行（YAML 折叠后）报 **`Error: Too many command line arguments`**（`\` 后行首空格被吞导致参数粘连）——**run 块写单行**（或整块用 `|` 多行但无 `\` 续行符）。qtcrowd deploy-provider.yml 的 4 个 `-target=` 多行续行部署失败 1 次，改单行后 success。

### 20. FC 挂网关后 backend_api 用网关地址（2026-08 qtcrowd 实证）

前台 provider 的 `QTCLOUD_CROWD_BACKEND_API` 指向后台时：若后台 FC 已挂 API 网关，**用网关地址**（`https://api.example.com/<product>`）而非裸 FC URL（`<fn>-<rand>.<region>.fcapp.run`）——统一入口、域名规范。配置前先 `curl https://api.example.com/<product>/health` 或关键 API 验证路由通（404 = 网关未挂/路径错——改回直连 URL 是临时方案，挂好网关后改回）。

### 22. CI 禁止手动触发 + 本地 terraform 网络阻塞的正确路径（2026-08-26 qtcloud-course 实证）

**用户明确纠正：CI 禁止手动触发**——deploy-*.yml **不得加 `workflow_dispatch`**，部署只走 tag 版本流程（`<scope>/*` tag → CI）。曾犯：本地 `terraform init` 因 registry 查询失败（`Failed to query available provider packages`——registry.terraform.io 网络不通）时，擅自加 workflow_dispatch + `gh workflow run` 手动部署——被纠正"谁让你 ci 手动触发的？ci 禁止手动触发"。

本地 init 失败的可靠路径：
- provider 缓存（`~/.terraform.d/plugin-cache`）帮不了本地 init（init 总要查 registry 确认版本，`-plugin-dir` 报"cannot install existing to itself"，复制 `.terraform/providers` + lock 文件仍查询）——**不要在这上面耗时间**
- 正确路径：**等网络恢复走正常 tag 流程**（CI apply 幂等）或用户拍板发版本后由 CI 完成——本地仅可做 terraform fmt/validate（不需 registry）

**纪律总结**：
- CI workflow 保持最小触发面（只 tag）——加任何手动触发入口前先问用户
- 在量潮 CI 里"部署基础设施"与"发版本"是绑定的（tag 触发 apply）——不存在"不发版本只部署"的路径

**build 类 workflow 触发面收窄（2026-08-26 qtcloud-course build-cli 实证）**：`on: release: types: [published]` 会**任何 scope 的 Release**（provider/site/cli）都触发——job 级 `if: startsWith(github.ref, 'refs/tags/cli/')` 只跳过执行（跑出 `startup_failure` 噪音，job 启动即失败）——正确写法是触发面收窄：`on: push: tags: ['cli/**']`（只对应 scope tag 触发，与 deploy-provider 同模式）。多 scope 仓库（provider/site/cli/studio）检查每个 workflow 的 on 条件是否只匹配自己的 tag 前缀。

### 23. 部署验证惯例：桶清单 + FC 函数 + 线上 API 三查

部署/排障后统一验证（qtcloud-crowd/qtcloud-course 实证）：`aliyun oss ls oss:// | grep -i <app>`（桶存在）→ `aliyun fc ListFunctions | grep <app>`（函数在线——FC 3.0 无 service，函数名 = `<project>-<env>`）→ `curl <网关或 FC URL>/<关键API>`（数据通）。FC 函数 HTTP URL 从 `aliyun fc ListTriggers --functionName <fn>` 的 `httpTrigger.urlInternet` 取（`triggerUrl` 字段不存在）。

### 24. gh run 状态排障：列表缓存误导 + cancel 竞态 + runner 波动（2026-08-26 qtcloud-course 实证）

监控/清理部署 run 时的三个坑与可靠做法：

- **`gh run list` 状态缓存误导**——显示 queued 的 run 可能实际 `in_progress`（部署其实在跑）；已被 cancel 的 run 列表仍显示 queued。**查真实状态用 API 直查**：`gh api "repos/<org>/<repo>/actions/runs/<id>" -q '.status + " " + (.conclusion // "")'`——不要信 `gh run list` 的列显状态
- **`gh run cancel` 对 concurrency 排队中 run 竞态**——报 `Cannot cancel a workflow run that is completed` 但状态仍 queued（cancel 请求提交但 GH 状态机未处理）——REST DELETE 也 403（token 权限/排队中限制）——**不要反复尝试**：被取消但卡在队列的 run 通常不阻塞后续 run（concurrency 按 in_progress 判定），等 GH 自行清理；若真阻塞，需用户在控制台手动取消
- **runner 分配失败**——run 排队 15m 后失败，`gh run view` 显示 `The job was not acquired by Runner of type hosted even after multiple attempts`——GitHub Actions 基础设施波动（非代码问题）——修复：`gh run rerun <id> --failed` 重试（或重打 tag）——无需改任何配置。**重试纪律（用户实证\"再失败就不要重试了\"）**：runner 波动重试最多 1-2 次，**用户明确说停止就立即停止**（不循环 rerun）——失败后如实报告状态（run queued / 线上版本未变），把选项给用户（等恢复 / 换路径 / 暂停）
- **run ID 解析**：`gh run list --workflow=<wf>.yml` 行格式的 ID 提取曾多次失败（awk 取到时间戳/in_progress）——可靠模式：`gh run list --workflow=<wf>.yml --limit 1 2>/dev/null | grep -oP 'push\t\K\d+'`（tag 触发行）或 `gh api "repos/<org>/<repo>/actions/runs?workflow=<wf>&per_page=1" -q '.workflow_runs[0].id'`

### 21. Provider OSS 存储实现模式（2026-08-26 qtcloud-course 实证）

FC 上 provider 的数据持久化选 OSS 时（替代 NAS/本地 DB），可复用实现模式（qtcloud-crowd 与 qtcloud-course 两处实证）：

- **存储设计：每表单对象全量原子写**——一个资源表一个 OSS 对象（`programs.json`/`courses.json`……对象内是实体列表）——读时 Get 整个对象解析、写时全量覆盖；**懒加载**（首次读从 OSS 拉取 + 内存缓存 + 写时同步回 OSS + seq 续号——重启恢复）
- **配置约定**：`QTCLOUD_<APP>_STORE=oss|memory`（默认 memory 测试）+ `QTCLOUD_OSS_BUCKET/ENDPOINT/ACCESS_KEY_ID/ACCESS_KEY_SECRET`（QTCLOUD_OSS_* 前缀统一）
- **桶命名惯例**：provider 数据桶 = `<app>-provider`（qtcloud-crowd-provider / qtcrowd-provider / qtcloud-course-provider）——私有桶；terraform 变量给默认值（如 `oss_data_bucket` default）+ CI 用 `TF_VAR_oss_access_key_id/secret` 注入（secrets 同 ALIYUN_ACCESS_KEY_ID）
- **seed 链**：Dockerfile 启动 `seed → seed-catalog → server`（全部 STORE=oss）——数据自动导入桶；`cmd/seed` 幂等（SetID 固定 ID）
- **坑**：OSS AK/SK 会明文落入 tfstate（注释与文档说明即可——与 crowd 现状一致）；store 层改造时保留内存实现（测试/本地），删 SQLite 前确认 handler 共用工厂（`newStores`）消除重复代码
- 测试模式：mock OSS server（校验签名头 + 404 XML——对齐 crowd oss_test）；覆盖重启恢复/ListWhere/SetID 持久化/快照格式（按 ID 排序）

### 15. 证书诊断：curl exit 60 但 openssl verify OK（2026-08 qtcloud-crowd 实证）

`curl https://<域名>` 报 SSL cert verify failed (exit 60)，但 `openssl s_client -connect <域名>:443 -servername <域名> | grep "Verify return code"` 返回 `0 (ok)`——**不是部署问题，是 curl 的 CA bundle**（hermes venv 的 certifi 缺 ZeroSSL 根）。判断链：openssl verify 0 = 服务端证书链完整 → 用 `curl --cacert /etc/ssl/certs/ca-certificates.crt` 对比（系统 CA 有 ZeroSSL）→ 确认 CA 库差异而非站点故障。另：**hostname 不匹配时 openssl verify 仍返回 0**（它只验证链不验证主机名）——python urllib/curl 报 `CERTIFICATE_VERIFY_FAILED: Hostname mismatch` 才是通配符覆盖问题（三层子域，见第 4 节）

```
子模块 commit+push → 领域父仓库更新指针 → 根仓库更新指针
```

## 参考

- `references/api-gateway-config.md` — API 网关配置：RPC 签名脚本模式 + DescribeApi 模板改造 CreateApi 流程（5 步）
- `references/qthealth-deploy-2026-08.md` — qthealth 双站上线完整排障记录（6 轮 CI 修复）
- `references/qtcloud-agent-deploy-2026-08.md` — qtcloud-agent 双站（模板复制适配）4 轮排障：undeclared resource / NoSuchBucket / DNS NXDOMAIN
- `references/qtfiction-deploy-2026-08.md` — qtfiction（仓库根 Vite）部署 4 轮排障：favicon / 阻止公共访问→私有回源 / 证书 InvalidSSLPub 两变体 / SPA 章节级深链 key
