# qtfiction 部署排障记录（2026-08-23，site/v0.1.0-alpha.1 → alpha.2）

参考：qtfounder deploy-site.yml 模式（site/* tag → Actions → OSS 桶 → CDN → 刷新）。
qtfiction 差异：Vite 项目在**仓库根**（非 src/site），路由 /series /read。

## 链路

```
qtcloud-devops release publish site/v0.1.0-alpha.1（audit 7/7：版本/根 CHANGELOG/pubspec/工作区/标签）
→ 建 OSS 桶 qtfiction-site + 静态托管（index.html/404.html）
→ aliyun cdn AddCdnDomain（Sources JSON 数组格式）
→ DNS CNAME（alidns AddDomainRecord：RR=fiction, CNAME=fiction.example.com.w.kunlunaq.com）
→ SetCdnDomainSSLCertificate（泛域名 *.example.com）
→ BatchSetCdnDomainConfig（私有回源/根改写/强制 HTTPS）
→ 删 tag 重打触发 deploy-site.yml
```

## 4 轮排障

1. **favicon 文件名**：workflow 写 `dist/favicon.svg` 但 public/ 是 `vite.svg` → `Error: stat dist/favicon.svg`。修：改 `dist/vite.svg`（上传名可仍为 favicon.svg）。
2. **桶公共读被"阻止公共访问"拦截**：新桶默认开启——`set-acl public-read` 报 `Put public bucket acl is not allowed`，bucket-policy 报 `Put public bucket policy is not allowed`。**不折腾公共读**——历史方案（qtcloud-agent cdn.tf）：
   ```
   aliyun cdn BatchSetCdnDomainConfig --DomainNames fiction.example.com --Functions '[
     {"functionArgs":[{"argName":"private_oss_auth","argValue":"on"}],"functionName":"l2_oss_key"},
     {"functionArgs":[{"argName":"source_url","argValue":"^/$"},{"argName":"target_url","argValue":"/index.html"},{"argName":"flag","argValue":"break"}],"functionName":"back_to_origin_url_rewrite"},
     {"functionArgs":[{"argName":"enable","argValue":"on"}],"functionName":"https_force"}
   ]'
   ```
   桶保持 private，浏览器经 CDN 正常读。
3. **证书绑定 InvalidSSLPub 两变体**：
   - `tr -d '\n'` 删换行 → InvalidSSLPub——PEM 必须保留换行（`$(cat fullchain.cer)` 原样）。
   - glob `~/.acme.sh/*.example.com_ecc/` 匹配全部 `*.example.com_ecc` 后缀目录（agent.cloud/health.cloud…）→ 拼接多张证书 → InvalidSSLPub——单引号字面化 `'*.example.com_ecc'`。
   - 成功标志：`DescribeCdnDomainDetail` 的 `ServerCertificateStatus: on`。
4. **SPA 深链 404（XML NoSuchKey）**：OSS 静态托管对未上传 key 返回 NoSuchKey XML（非 SPA 壳）——私有回源模式下 error_document 不兜底。修：workflow 动态列举全部章节生成无扩展名 key：
   ```yaml
   for s in $(ls -d data/series/*/ | xargs -n1 basename); do
     ossutil cp dist/index.html oss://BUCKET/series/$s -f --meta "Content-Type:text/html" ...
     ossutil cp dist/index.html oss://BUCKET/read/$s -f --meta ...
     for f in $(ls data/series/$s/*.md | xargs -n1 basename); do
       ossutil cp dist/index.html "oss://BUCKET/read/$s/${f%.md}" -f --meta "Content-Type:text/html" ...
     done
   done
   ```
   注意章节 file 名无 .md（series.ts 解析时去扩展名）——key 必须对应 React Router 的实际路径段。

## 验证

```
curl -sI https://fiction.example.com/                  # 200
curl 首页 → grep <title>量潮小说</title>
curl /series/<id> /read/<id>/<file>（URL 编码）→ 200（SPA 壳）
curl http:// → -L → https（https_force 生效）
```

## 教训

- 域名首次添加 CDN `configuring` 5-30 分钟——后台轮询 `DescribeCdnDomainDetail` DomainStatus=online 再触发部署
- workflow 晚于 tag 提交 → 删 tag 重打触发（git tag -d + push :refs/tags/ + 重打 + push）
- **排障先查历史仓库 IaC**（qtcloud-agent cdn.tf 的私有回源）——一轮解决（见 SKILL.md 工作流节）
