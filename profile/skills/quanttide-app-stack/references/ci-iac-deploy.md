# 应用部署排障链（GitHub Actions + Terraform + 阿里云 CDN/OSS）

> qthealth 首次部署 health.example.com 的 4 轮 CI 排障实录（2026-08-16）。任何 qtcloud-* 应用首次部署都会遇到同类问题。

## 部署流程（tag 触发）

```
<app>/* tag → Actions：
  terraform init（OSS 远程状态）+ apply（首次建基础设施）
  → build（flutter/npm）→ ossutil cp 上传 OSS 静态桶
  → CDN 刷新 → acme.sh 证书绑定（HTTPS）
```

## 排障链（按出现顺序）

### 1. setup-ossutil 版本：`@v3.0`（v1 不存在）

`manyuanrong/setup-ossutil@v1` → "unable to find version v1"——参考已部署仓库（qtcloud-secret/org）用 `@v3.0`。上传语法：`ossutil cp <src> <dst> -r -f --meta=Cache-Control:public,max-age=31536000`（v3 语法，参数顺序与 v1 不同）。

### 2. CI 环境缺 terraform

`terraform: command not found`（exit 127）——加 `hashicorp/setup-terraform@v3` 步骤。

### 3. terraform apply 卡死（最阴的坑）：`-input=false`

`terraform apply -auto-approve` 遇**无默认值的必填变量**（如 FC 的 `image` 变量，敏感值不写默认）会**交互等待输入**——job 卡 15-20 分钟不动，日志不可见（in progress）。修复：`terraform apply -input=false -auto-approve`（缺变量直接报错）+ 补齐 `-var image=<占位>`。

### 4. 共享 RAM 资源冲突：`-target` 只建所需资源

`EntityAlreadyExists.Policy/Role`（409，AliyunCDNAccessingPrivateOSSRolePolicy/Role）——CDN 私有回源 RAM 角色是**跨应用共享的全局资源**（qtcloud-secret 部署时已创建），新应用复制模板同名创建即冲突。修复：首次部署用 `-target=` 只 apply 本应用所需资源（OSS 桶 + CDN 域名 + config + DNS 记录），跳过共享 RAM 与 FC。`-target` 会自动带上目标的依赖链。

### 5. HTTPS handshake failure：证书未绑定

CDN 域名默认 HTTP only。绑定证书（见下"证书"节）。

### 6. HTTP 404：CDN 源站桶名 ≠ 上传目标桶名

模板复制后 cdn.tf 源站指向 `<app>-studio`（模板默认）而部署上传到 `<app>-site`——桶里文件齐全但源站空桶 → 404。排查：`aliyun oss ls oss://<桶>/`（注意输出可能被 head 截断，先数 Object Number）对比 cdn.tf `sources.content`。修复：`aliyun cdn ModifyCdnDomain --DomainName <域名> --Sources '[{"type":"oss","content":"<正确桶>.oss-cn-hangzhou.aliyuncs.com","port":80,"priority":20}]'`（CLI 直改比重跑 terraform 快），同时修 cdn.tf 保持 IaC 一致。

## 证书（acme.sh + aliyun CLI）

- **泛域名证书 `*.example.com` 匹配单层子域**（health.example.com ✅）但**不匹配两层**（health.cloud.example.com ❌、studio.health.example.com ❌——需单域名证书，acme.sh 签发，90 天续期 + reloadcmd 重绑）
- **两层子域单域名签发**（acme.sh 已配阿里 DNS 凭证，`~/.acme.sh/account.conf` 有 `SAVED_Ali_Key/SAVED_Ali_Secret`——`dns_ali` 自动加 TXT 验证）：
  ```bash
  ~/.acme.sh/acme.sh --issue -d <两层域> --dns dns_ali --server letsencrypt
  # 产物在 ~/.acme.sh/<两层域>_ecc/{fullchain.cer,<域>.key}
  ```
- 绑定：`aliyun cdn SetCdnDomainSSLCertificate --DomainName <域名> --SSLProtocol on --CertType upload --SSLPub "$(cat .../fullchain.cer)" --SSLPri "$(cat .../*.key)"`
- **PEM 路径通配符引号坑**：证书目录名含字面星号（`*.example.com_ecc/`）——`cat ~/.acme.sh/*.example.com_ecc/fullchain.cer` 无引号时 shell 展开错乱，aliyun CLI 报误导性错误 `InvalidCertificate.TooLong`（实际是 PEM 内容空/错）——**用引号保护**（`cat '*.example.com_ecc/fullchain.cer'`）即成功

## 多站部署（同应用多域名，2026-08-16 qthealth site+studio 双站）

一个应用部署多个 Web 站（官网 health.example.com + 客户端 studio.health.example.com）的模式：

1. **IaC 复制资源组**：cdn.tf 复制现有域名块生成新资源组——`alicloud_cdn_domain_new.<新名>`（新域名 + 对应源站桶）+ **独立**的 config 三件套（private_back/root_rewrite/https_force，`domain_name` 引用新资源）+ `alicloud_alidns_record.<新名>`（rr 为完整子域如 `studio.health`）；资源名注意与已有不冲突（.studio 被 site 用 → 新的用 .studio_web）
2. **workflow 补 IaC apply 步骤**：deploy-studio.yml 复制自 secret 时没有 terraform 步骤（假设 IaC 已落地）——新站需补 `hashicorp/setup-terraform@v3` + `terraform apply -input=false -auto-approve -target=<新资源组>`（与首个站相同模式，state key 用 `<app>/studio.tfstate`）
3. **证书**：两层子域（studio.health.…）泛域名不匹配——acme.sh 单域名签发（见上）→ 绑定后 `curl https://<域>` 验证 200
4. **CI 触发**：每个站独立 tag 前缀（`site/*`、`studio/*`）——同 tag 重打需先 `git tag -d` + `git push origin :<tag>` 删除远端再重打
5. **CI 环境变量残留**：模板复制会带旧站 env（如 PROVIDER_BASE_URL/AUTH_BASE_URL 指向 qtcloud-secret 的 FC）——无鉴权应用可清理，不影响部署

## IaC 要点

- 远程状态：`terraform init -backend-config="bucket=<org>-terraform-state" -backend-config="key=<app>/<scope>.tfstate" -backend-config="region=cn-hangzhou"`（CI secrets：ALIYUN_ACCESS_KEY_ID/SECRET → ALICLOUD_ACCESS_KEY/SECRET 环境变量）
- CDN 域名 DNS：cdn.tf 里 `alicloud_alidns_record`（阿里云 DNS，rr 如 "health"）——域名解析生效快
- 复制模板（qtcloud-secret）适配：替换 `qtcloud-secret→<app>`、`secret.cloud.example.com→<域名>`；**移除 JWT/密钥相关**（provider 无鉴权时删 variables + fc.tf 环境变量）；`terraform fmt -recursive` + 残留 grep 校验

## 复制模板适配的隐藏坑（2026-08-21 qtcloud-agent 双站 3 轮排障）

首次部署新应用时从已部署仓库（qthealth 等）复制 terraform/workflows 适配，除"多站部署"已知项外还有 4 个隐藏坑（`Reference to undeclared resource` / `NoSuchBucket` / NXDOMAIN 三连）：

1. **漏复制 platform.tf（data source）**：oss.tf 引用 `data.terraform_remote_state.platform`（共享平台 state 取 resource_group_id），但复制清单常漏 platform.tf → `Error: Reference to undeclared resource`。**platform.tf 必须复制**（内容是共享 data source，无副作用）。同时 **outputs.tf 清理**：复制模板若删了 fc.tf，outputs.tf 里 `alicloud_fcv3_function.this` 引用要删（同样报 undeclared）——outputs 只留实际存在的资源。
2. **site 桶资源缺失**：studio-bucket.tf 只有 studio 桶；site 部署的 `-target` 若不含桶资源（qthealth 的 site 桶是历史遗留手动建的），新应用会 `NoSuchBucket`（ErrorCode=NoSuchBucket, Bucket=<app>-site）。**新建 site-bucket.tf**（bucket=`<app>-site`，website 配置同 studio 桶）+ deploy-site.yml 的 `-target` 加 `alicloud_oss_bucket.site`。
3. **DNS rr 批量替换 bug（最阴）**：`sed -i 's/health.example.com/agent.cloud.example.com/g'` 只替换**完整域名串**——cdn.tf 里 DNS 记录的**独立 rr 值**（`rr = "health"` / `rr = "studio.health"`）保持原样！结果：CDN 域名建对了（terraform 成功），但 DNS 记录仍是旧站的值 → `nslookup` NXDOMAIN、curl HTTP 000。**修**：手动把 cdn.tf 的 `rr` 改成新完整子域（`agent.cloud` / `studio.agent.cloud`）再重打 tag（terraform 幂等更新记录）。批量替换后必须 `grep rr cdn.tf` 人工核对。
4. **单域名证书双发**：两层/三层子域（agent.cloud.…、studio.agent.cloud.…）各需一张 acme.sh 单域名证书——两条 `--issue` 串行即可；CDN 域名建好后统一绑定。

排障顺序特征：terraform 报 undeclared → 补文件；上传报 NoSuchBucket → 补桶；访问 NXDOMAIN → 查 DNS rr。

## 验证

```bash
curl -sI https://<域名>/          # 200 + 无 handshake failure
curl -s <域名>/ | grep "<title>"   # 内容正确
nslookup <域名>                    # DNS 解析（kunlunaq.com = 阿里云 CDN）
```

## 门禁进 CI：两作业形态（2026-09-24 qtdata studio 落地）

家族里 studio 的 CI 是**一个文件两个作业**——分支/PR 只跑门禁，tag 才部署。先例：`domains/quanttide-work/apps/qtcloud-work/.github/workflows/release-studio.yml`；qtdata 沿用文件名 `deploy-studio.yml`（家族注释里点名过 qtdata 用这个名字，**别改名**）。

```yaml
on:
  push:
    branches: ['**']
    tags: ['studio/*']
    paths: ['src/studio/**', '.github/workflows/deploy-studio.yml']
  pull_request:
    paths: ['src/studio/**', '.github/workflows/deploy-studio.yml']

concurrency:                       # 按 ref 分组：分支上的门禁不会把正在跑的发布干掉
  group: deploy-studio-${{ github.ref }}
  cancel-in-progress: true

jobs:
  quality-gates:                   # 分支与 PR 触发，只跑门禁、不部署
    defaults: { run: { working-directory: src/studio } }
    # steps: Checkout → Setup Flutter(cache: true) → Pub get → Format → Analyze → Test
  build-and-deploy:
    needs: quality-gates
    if: startsWith(github.ref, 'refs/tags/studio/')   # 用 if 把部署限在 tag 上
```

门禁三条按平台契约（Dart = `dart format` + `flutter analyze`），再加 `flutter test`：

```yaml
- run: dart format --set-exit-if-changed --output=none .
- run: flutter analyze
- run: flutter test
```

**三处与先例的差异，照抄时注意**：
- qtcloud-work 那份门禁**只有 analyze + test，没有 `dart format`**，而契约明写 dart format 是门禁的一半——qtdata 补齐了；要不要回头给 qtcloud-work 也补，属于跨仓不一致，**问用户别自行动手**
- `paths` 过滤对 tag 推送**同样生效**，所以发布提交必须落在 `src/studio/**`（改版本号/CHANGELOG 正好在那里）；把 workflow 文件自身也列进 paths，改 CI 才会被自己触发验证
- **Flutter 版本要跟本地同版**：`flutter-version` 是硬编码的，「本地从严、与 CI 一致」这条要求版本也对齐——**先用某版把三条门禁在本机跑绿，再把 CI pin 升到那一版**（qtdata 实测 3.44.9 三条全绿后才从 3.44.8 升上去）。升完核对本机 `which flutter` 指向同一版（曾有新旧两套 SDK 并存、`.bashrc` 指旧版的情况，详见 `studio-contract-alignment.md`）

## 验证 CI 真跑了（公开仓免 gh auth）

`gh` 的 token 会过期（报 `could not read Username`），但**公开仓的 Actions 走未认证 API 就读得到**——推完等 1–3 分钟再查：

```bash
# 最近几次运行
curl -s "https://api.github.com/repos/<owner>/<repo>/actions/runs?per_page=3" | python3 -c "
import json,sys
for r in json.load(sys.stdin)['workflow_runs']:
    print(r['name'], r['head_sha'][:7], r['event'], r['status'], r['conclusion'], r['html_url'])"

# 逐作业逐步骤（门禁过没过、部署是不是按设计 skip）
curl -s "https://api.github.com/repos/<owner>/<repo>/actions/runs/<run_id>/jobs" | python3 -c "
import json,sys
for j in json.load(sys.stdin)['jobs']:
    print(j['name'], j['status'], j['conclusion'])
    for s in j.get('steps', []): print('  -', s['name'], s['status'], s['conclusion'])"
```

判读：**部署作业 `skipped` 就是正确结果**（分支推送不该部署）；门禁各步全 `success` 才算过。轮询写法——`for i in $(seq 1 12); do ... sleep 20; done`，`status` 变 `completed` 即停。

**别停在「本地三条绿」**：同一份 workflow 在 GitHub 上真跑一次、读到 run 号与逐步骤结果，才是最硬的证据（qtdata 那次两次 run 都 success，文档里直接记 run id 备查）。
