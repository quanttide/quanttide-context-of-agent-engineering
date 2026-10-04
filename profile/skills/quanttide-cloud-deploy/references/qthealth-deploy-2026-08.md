# qthealth 双站上线排障记录（2026-08）

qthealth（量潮健康）site + studio 部署到阿里云的完整排障链。同类部署可参考。

## 排障时间线（6 轮 CI 修复）

1. **setup-ossutil@v1 不存在** → 改 `manyuanrong/setup-ossutil@v3.0`（v1 版本从未发布）
2. **CI 无 terraform** → 加 `hashicorp/setup-terraform@v3` 步骤
3. **terraform apply 卡死 20 分钟** → 原因：`image` 变量无默认值，`-auto-approve` 遇未传变量交互等待输入 → 修：`-input=false` + `-var image=placeholder`
4. **RAM 409 EntityAlreadyExists** → `AliyunCDNAccessingPrivateOSSRole`/Policy 被 qtcloud-secret 部署创建过（账号级共享）→ 修：`-target=` 只建本项目资源（OSS 桶 + CDN 域名 + configs + DNS），跳过 platform.tf/fc.tf
5. **HTTPS handshake failure** → CDN 域名无证书 → 修：acme.sh 泛域名证书 `*.example.com` 绑定（引号保护通配符目录名——`cat '*.example.com_ecc/fullchain.cer'`，否则 shell 通配符展开异常报 InvalidCertificate.TooLong）
6. **全站 404** → CDN 源站指向空桶 <appA>-studio（terraform 建的）但上传目标是 <appA>-site → 修：`aliyun cdn ModifyCdnDomain --Sources '[...<appA>-site...]'` 直接改源站 + 同步 cdn.tf 代码

## studio 二层子域证书

`studio.health.example.com`（两层）不匹配泛域名 `*.example.com`（单层）→ acme.sh 单独签发：
```bash
~/.acme.sh/acme.sh --issue -d studio.health.example.com --dns dns_ali --server letsencrypt
```
绑定后 45s 生效（HTTP 200）。

## 关键命令备忘

```bash
# 查看 workflow 运行
gh run list --workflow=deploy-site.yml
gh run view <run-id> --log-failed | grep -iE "Error|error"

# 重打 tag 触发
git tag -d <tag> && git push origin :<tag> && git tag <tag> && git push origin <tag>

# 桶内容检查（aliyun CLI）
~/.local/bin/aliyun oss ls oss://<bucket>/
```

## 教训总结

- terraform apply 一律 `-input=false`（CI 环境）
- 阿里云 RAM 资源是账号级共享——新项目部署用 -target 隔离
- CDN 源站与上传桶必须一致
- 证书绑定后 curl 验证（handshake failure = 未绑，404 = 源站问题）
