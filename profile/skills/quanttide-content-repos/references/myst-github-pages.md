# MyST + GitHub Pages 配方（2026-09-24 gallery 实录）

## 构建与预览

```bash
cd docs/<repo>
npm install -g mystmd                                    # 或 npx -y mystmd@latest（首次会下载）
BASE_URL=/<repo-name> myst build --html                  # 产物在 _build/html
BASE_URL=/<repo-name> myst start                         # 本地预览服务器
```

`BASE_URL` 与仓库名一致（GitHub Pages 项目站挂在 `/<repo>/`）。**构建日志重定向到文件再 grep**，别用管道接 `head`：`| head -12` 会因 SIGPIPE 提前杀掉进程，日志看着正常、产物不全（本轮踩过，白等一次）。

看构建结果：

```bash
grep -E "Built|pages|rror" /tmp/myst-build.log      # 期望：Built N pages
ls _build/html | grep -v '^build$'                   # 目录即 slug：qtdata/ qtclass/ …
```

## CI（deploy.yml）关键两行

```yaml
env:
  BASE_URL: /${{ github.event.repository.name }}
...
site:                      # myst.yml
  template: book-theme
  options:
    folders: true
```

- 缺 `BASE_URL` → 产物里资源与站内链接都是根路径，GitHub Pages 项目站上 **全 404**（页面裸着渲染，等于站是坏的）
- 缺 `options.folders` → 目录内 `index.md` 的 slug 变成 `index-1`、`index-2`…（且随 toc 顺序漂移），拿不到 `/qtdata` 这种干净 URL
- 两者都补齐后：产物目录名 = 业务线目录名，链接 = `/<repo>/qtdata`

## 线上验证（必须做，别停在本地构建绿）

```bash
# 1) CI 真跑结果（公开仓未认证 API 可读）
curl -s "https://api.github.com/repos/<org>/<repo>/actions/runs?per_page=1"   # 看 status/conclusion

# 2) 线上页面
curl -sL "https://<org>.github.io/<repo>/<slug>/" | grep -oE "<title>[^<]*</title>"
curl -s -o /dev/null -w "%{http_code}\n" "https://<org>.github.io/<repo>/<slug>/"   # 期望 200
```

判据三连：CI success、目标 URL 200、正文关键字命中（如新写的「大模型挖掘」字样）。改 slug 之后**旧 URL 应 404、新 URL 才 200**——这是修好的证据。GH Pages 生效有延迟，轮询几轮（每轮 25s 左右）。

## myst.yml 骨架（gallery 现状）

```yaml
version: 1
project:
  title: <站点名>
  description: <同上>
  license: CC-BY-4.0
  toc:
    - file: index.md            # 站点落地页（README.md 不进 toc）
    - title: 量潮数据
      children:
        - file: qtdata/index.md
    - title: 量潮课堂
      children:
        - file: qtclass/index.md
        - file: qtclass/big-data-practice.md
site:
  template: book-theme
  options:
    folders: true
```

## 仓库卫生

- `.gitignore` 要有 `_build/`（gallery 缺，本地构建留残留，手工 `rm -rf _build`）
- 内部链接写成同目录相对路径（`[案例](./big-data-practice.md)`）：构建时自动带上 `BASE_URL` 前缀，线上可点
- 校验结果：`find _build/html -maxdepth 1 -name "*.json"` 与目录名对得上，说明 toc 里每条都被渲染成页
