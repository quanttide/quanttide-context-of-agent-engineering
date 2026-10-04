# CI/IaC 部署模式（quanttide 项目）

## GitHub Actions 流水线模式

### 四端验证（ci-verify.yml）

push/PR 触发，验证全端构建+测试：

| 端 | 工具 | 验证命令 |
|----|------|---------|
| cli (Rust) | dtolnay/rust-toolchain | cargo build + cargo test |
| provider (Go) | actions/setup-go | go build + go vet + go test |
| studio (Flutter) | subosito/flutter-action | flutter pub get + analyze + test |
| site (React) | actions/setup-node | npm ci + npm run build |

### 部署流水线（deploy-*.yml）

tag 触发（`provider/*`、`studio/*`、`site/*`）：

```yaml
on:
  push:
    tags:
      - 'studio/**'  # 或 provider/** / site/**
```

关键 steps：
1. **Setup terraform**：`uses: hashicorp/setup-terraform@v3`（CI 默认无 terraform）
2. **Apply infrastructure**：`terraform init -backend-config=... && terraform apply -input=false -auto-approve -var ... -target ...`
3. **Setup ossutil**：`uses: manyuanrong/setup-ossutil@v3.0`（v1 不存在）
4. **Build + Upload + CDN refresh**

### ossutil 语法（v3）

```bash
# 正确（v3 参数顺序：源在前，目标在后，选项在后）
ossutil cp src/site/dist/ oss://bucket/ -r -f --meta=Cache-Control:public,max-age=31536000

# 错误（v1 参数顺序）
ossutil cp -r -f src/site/dist/ oss://bucket/
```

## Terraform 模式

### 必加标志

```bash
terraform apply -input=false -auto-approve  # -input=false 防交互等待卡死
```

**为什么**：`-auto-approve` 跳过确认但不跳过变量输入——缺必填变量时 terraform 交互等待，CI 永远挂起。

### 必填变量占位

```bash
-var image=placeholder  # FC 镜像等必填变量无默认值时必须传
```

### -target 绕过共享资源冲突

```bash
# 站点部署只需 OSS/CDN/DNS，跳过 RAM（共享角色已存在）和 FC（provider 专属）
terraform apply -input=false -auto-approve \
  -var region=cn-hangzhou \
  -var image=placeholder \
  -target=alicloud_oss_bucket.studio \
  -target=alicloud_cdn_domain_new.studio \
  -target=alicloud_cdn_domain_config.studio_private_back \
  -target=alicloud_cdn_domain_config.studio_root_rewrite \
  -target=alicloud_cdn_domain_config.studio_https_force \
  -target=alicloud_alidns_record.studio
```

**触发条件**：CDN 域名配置不依赖 RAM 资源时可用 -target；若 CDN config 引用了 RAM role/attachment，需先 import 共享资源到 state。

### -target 的 state 后果

- -target 创建的资源进入 state，其余不进
- 下次全量 apply 会尝试创建未进 state 的资源（RAM 等）
- 长期解法：共享 RAM 资源改为 data source 引用（对齐 quanttide-secret 的 platform.tf 模式）

## SSL 证书管理

### 泛域名 vs 单域名

```
*.example.com    → 匹配一层子域：health.example.com ✅
                    不匹配两层子域：health.cloud.example.com ❌
                    不匹配两层子域：studio.health.example.com ❌
```

### acme.sh 签发单域名证书

```bash
# 签发（DNS 验证——需 aliyun CLI 凭证）
~/.acme.sh/acme.sh --issue -d studio.health.example.com --dns dns_ali --server letsencrypt

# 绑定到 CDN（需 aliyun CLI）
PUB=$(cat ~/.acme.sh/studio.health.example.com_ecc/fullchain.cer)
PRI=$(cat ~/.acme.sh/studio.health.example.com_ecc/studio.health.example.com.key)
aliyun cdn SetCdnDomainSSLCertificate \
  --DomainName studio.health.example.com \
  --SSLProtocol on --CertType upload \
  --SSLPub "$PUB" --SSLPri "$PRI"
```

**通配符坑**：目录名 `*.example.com_ecc` 含字面星号，shell 通配符会匹配——cat 命令必须加引号保护。

## CDN 配置

### 源站必须匹配上传目标

```hcl
# cdn.tf 源站
sources {
  content  = "<appA>-site.oss-cn-hangzhou.aliyuncs.com"  # 必须与 deploy 的 OSS_BUCKET 一致
  type     = "oss"
}
```

```yaml
# deploy.yml 上传目标
env:
  OSS_BUCKET: <appA>-site  # 必须与 cdn.tf 源站一致
```

不一致 → CDN 回源空桶 → 404。

### CDN 域名修改

用 aliyun CLI 直接改（比 terraform apply 快且安全）：
```bash
aliyun cdn ModifyCdnDomain --DomainName studio.health.example.com \
  --Sources '[{"type":"oss","content":"<appA>-site.oss-cn-hangzhou.aliyuncs.com","port":80,"priority":20}]'
```

## 常见报错速查

| 报错 | 原因 | 修复 |
|------|------|------|
| `setup-ossutil@v1` not found | 版本号错误 | 改 `@v3.0` |
| `terraform: command not found` | CI 无 terraform | 加 `hashicorp/setup-terraform@v3` |
| terraform 卡住不动 | 缺必填变量，交互等待 | 加 `-input=false` + `-var image=placeholder` |
| `EntityAlreadyExists.Policy` | RAM 角色/策略已存在（共享） | `-target` 跳过或 data source 引用 |
| `InvalidCertificate.TooLong` | 证书通配符展开问题 | shell cat 加引号保护 |
| `SSL alert handshake failure` | HTTPS 证书未绑定 | acme.sh 签发 + 绑定 CDN |
| HTTP 200 但内容不对 | CDN 源站指向空桶 | 检查 cdn.tf sources 与 deploy OSS_BUCKET 一致性 |
