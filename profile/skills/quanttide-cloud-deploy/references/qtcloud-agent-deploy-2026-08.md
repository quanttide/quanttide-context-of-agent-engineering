# qtcloud-agent 双站部署排障记录（2026-08-21）

部署 agent.cloud.example.com（site）+ studio.agent.cloud.example.com（studio），
复制 qthealth 模板适配，4 轮 CI 排障。

## 背景

- 从 qthealth 复制 manifests/terraform + .github/workflows/deploy-{site,studio}.yml
- 脚本批量替换：`health.example.com→agent.cloud.example.com`、
  `studio.health.example.com→studio.agent.cloud.example.com`、
  `<appA>-site→<appB>-site`、`<appA>-studio→<appB>-studio`、
  state key、project 默认值
- 同时签发两个单域名证书（两层/三层子域，泛域名 *.example.com 不匹配）：
  `acme.sh --issue -d <域名> --dns dns_ali --server letsencrypt`

## 排障序列

### 第 1 轮：undeclared resource（两个部署都失败）

```
Error: Reference to undeclared resource
  on oss.tf line 18: resource_group_id = data.terraform_remote_state.platform.outputs.resource_group_id
  on outputs.tf line 8: value = alicloud_fcv3_function.this.function_name
```

原因：
- 复制时漏了 platform.tf（`data "terraform_remote_state" "platform"` data source）
- 复制模板时删了 fc.tf（provider 未部署），但 outputs.tf 仍引用 fc 资源

修复：复制 platform.tf + 重写 outputs.tf（移除 fc 引用，只留 oss_bucket）。

### 第 2 轮：site NoSuchBucket（studio 成功，site 失败）

```
Error: oss: service returned error: StatusCode=404, ErrorCode=NoSuchBucket,
Bucket=<appB>-site
```

原因：deploy-site.yml 的 -target 只含 `alicloud_oss_bucket.studio`（studio 桶）——
<appB>-site 桶没有任何 terraform 资源定义（qthealth 当时桶已历史存在）。

修复：新建 site-bucket.tf（`alicloud_oss_bucket.site` bucket = <appB>-site，
静态网站托管 index.html）+ deploy-site.yml -target 加 `alicloud_oss_bucket.site`。

### 第 3 轮：DNS NXDOMAIN（两个部署 success 但域名无法解析）

```
server can't find agent.cloud.example.com: NXDOMAIN
```

原因：批量替换 `health.example.com→agent.cloud.example.com` 只命中完整域名串，
但 cdn.tf 的 alidns_record 里 `rr = "health"` / `rr = "studio.health"` 是独立值——
DNS 记录还是旧子域（health.example.com 的 CNAME），新域名无记录。

修复：rr 改为 `"agent.cloud"` / `"studio.agent.cloud"`，重新打 tag 触发
terraform apply 更新记录（apply 幂等，只更新 diff）。

## 验证

```
nslookup agent.cloud.example.com        # CNAME → CDN IP
curl -s https://agent.cloud.example.com/        # HTTP 200，<title>量潮智能体云</title>
curl -s https://studio.agent.cloud.example.com/ # HTTP 200
```

## 教训

1. 批量替换域名时，**DNS rr 值单独 grep 检查**（`grep -B1 -A4 alidns_record`）
2. 复制模板后 grep 全部 `data.` 与资源引用（platform data source 常漏）
3. 每个上传桶必须有 terraform 资源 + 在 -target 列表里（不靠历史存在）
4. 证书可在部署并行签发（不依赖 CDN 域名创建）；绑定用 `SetCdnDomainSSLCertificate`
   需在部署成功（域名存在）后执行
