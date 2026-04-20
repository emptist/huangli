# 静态网站部署详解

本文档详细介绍多种静态网站部署方式，包括首次部署、账号注册、后续更新等完整流程。

---

## 目录

- [Netlify](#netlify) - 推荐，域名简洁
- [Vercel](#vercel) - 流行，速度快
- [Surge](#surge) - 最简单，一行命令
- [GitHub Pages](#github-pages) - 免费，Git 集成
- [Cloudflare Pages](#cloudflare-pages) - 全球 CDN

---

## Netlify

### 安装

```bash
pnpm add -g netlify-cli
```

### 首次部署

1. 进入项目目录：
```bash
cd /Users/jk/gits/hub/tools_ai/huangli
```

2. 运行部署命令：
```bash
netlify deploy --prod
```

3. 首次使用会提示登录：
   - 输入 `y` 确认
   - 浏览器自动打开授权页面
   - 点击授权后回到终端

4. 选择创建新站点：
   - `What would you like to do?` → 选择 `Create & configure a new site`

5. 选择团队：
   - 选择你的用户名（个人账号）

6. 输入站点名称：
   - 输入 `huangli` 或其他你喜欢的名字
   - 这会决定你的域名：`huangli.netlify.app`

7. 确认部署目录：
   - `Publish directory` → 输入 `.` 或直接回车

8. 等待部署完成，终端会显示：
```
✔ Deploy is live!
URL:        https://huangli.netlify.app
```

### 后续更新

每次修改代码后，重新运行：
```bash
netlify deploy --prod
```

### 连接 Git 仓库（自动部署）

1. 打开 https://app.netlify.com
2. 点击 `Add new site` → `Import an existing project`
3. 选择 `GitHub`，授权后选择 `emptist/huangli` 仓库
4. Build command 留空，Publish directory 填 `/`
5. 点击 `Deploy site`

之后每次 `git push` 会自动触发部署！

### 自定义域名

1. 打开站点设置 → `Domain management`
2. 点击 `Add custom domain`
3. 输入你的域名，按提示配置 DNS

---

## Vercel

### 安装

```bash
pnpm add -g vercel
```

### 首次部署

1. 进入项目目录：
```bash
cd /Users/jk/gits/hub/tools_ai/huangli
```

2. 运行部署命令：
```bash
vercel
```

3. 首次使用会提示登录：
   - 输入 `y` 确认
   - 浏览器自动打开授权页面
   - 选择 GitHub 登录或邮箱注册

4. 按提示回答问题：
   - `Set up and deploy?` → `Y`
   - `Which scope?` → 选择你的用户名
   - `Link to existing project?` → `N`
   - `Project name?` → 输入 `huangli`
   - `In which directory?` → 直接回车（当前目录）
   - `Want to modify settings?` → `N`

5. 等待部署完成，终端会显示：
```
✔ Deployed to production
https://huangli.vercel.app
```

### 后续更新

```bash
vercel --prod
```

### 自动部署

Vercel 默认会连接 Git 仓库，每次 `git push` 自动部署。

### 自定义域名

1. 打开 https://vercel.com/dashboard
2. 进入项目 → `Settings` → `Domains`
3. 添加你的域名

---

## Surge

### 安装

```bash
pnpm add -g surge
```

### 首次部署

1. 进入项目目录：
```bash
cd /Users/jk/gits/hub/tools_ai/huangli
```

2. 运行部署命令：
```bash
surge .
```

3. 首次使用会提示注册：
   - 输入邮箱
   - 输入密码
   - 确认注册

4. 确认部署信息：
   - `project path` → 直接回车确认当前目录
   - `domain` → 输入你想要的域名，如 `huangli.surge.sh`

5. 等待上传完成，终端会显示：
```
Success! Project is published and running at huangli.surge.sh
```

### 后续更新

```bash
surge .
```

或指定域名：
```bash
surge . huangli.surge.sh
```

### 注意事项

- Surge 免费版不支持自定义域名
- 免费版有带宽限制
- 域名是 `xxx.surge.sh` 格式

---

## GitHub Pages

### 方式一：通过网页设置

1. 打开仓库设置页面：
   https://github.com/emptist/huangli/settings/pages

2. 在 `Source` 下选择：
   - `Deploy from a branch`
   - Branch 选择 `master`
   - 目录选择 `/ (root)`
   - 点击 `Save`

3. 等待几分钟，访问：
   https://emptist.github.io/huangli/

### 方式二：通过 GitHub Actions（推荐）

1. 创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [master]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v4
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - uses: actions/deploy-pages@v4
```

2. 在仓库设置中启用 GitHub Actions 作为 Pages 源

3. 每次 `git push` 自动部署

### 自定义域名

1. 在仓库根目录创建 `CNAME` 文件，内容为你的域名
2. 在域名 DNS 设置中添加 CNAME 指向 `emptist.github.io`

---

## Cloudflare Pages

### 方式一：Git 连接（推荐）

1. 打开 https://dash.cloudflare.com
2. 左侧点击 `Workers & Pages`
3. 点击 `Create` → `Pages` → `Connect to Git`
4. 选择 GitHub，授权后选择 `emptist/huangli` 仓库
5. 配置：
   - Framework preset: `None`
   - Build command: 留空
   - Build output directory: `/`
6. 点击 `Save and Deploy`

等待部署完成，访问 `huangli.pages.dev`

### 方式二：Wrangler CLI

1. 安装 Wrangler：
```bash
pnpm add -g wrangler
```

2. 登录：
```bash
wrangler login
```

3. 部署：
```bash
cd /Users/jk/gits/hub/tools_ai/huangli
wrangler pages deploy . --project-name huangli
```

### 后续更新

Git 连接方式：`git push` 自动部署

CLI 方式：
```bash
wrangler pages deploy . --project-name huangli
```

---

## 对比总结

| 平台 | 域名格式 | 自定义域名 | 自动部署 | 推荐指数 |
|------|----------|------------|----------|----------|
| Netlify | `xxx.netlify.app` | ✅ 免费 | ✅ Git 连接 | ⭐⭐⭐⭐⭐ |
| Vercel | `xxx.vercel.app` | ✅ 免费 | ✅ 默认开启 | ⭐⭐⭐⭐⭐ |
| Surge | `xxx.surge.sh` | 💰 付费 | ❌ | ⭐⭐⭐ |
| GitHub Pages | `user.github.io/repo` | ✅ 免费 | ✅ Git/Actions | ⭐⭐⭐⭐ |
| Cloudflare | `xxx.pages.dev` | ✅ 免费 | ✅ Git 连接 | ⭐⭐⭐⭐ |

---

## 常见问题

### Q: 部署后页面空白？

检查浏览器控制台是否有 JS 错误。常见原因：
- 文件路径问题（使用相对路径）
- CDN 资源加载失败

### Q: 如何查看部署日志？

- Netlify: 站点 → Deploys → 点击具体部署
- Vercel: 项目 → Deployments
- GitHub Pages: Actions 标签页
- Cloudflare: Workers & Pages → 项目 → Deployments

### Q: 如何回滚到之前版本？

- Netlify: Deploys → 找到之前的部署 → `Publish deploy`
- Vercel: Deployments → 找到之前的部署 → `Promote to Production`
- GitHub Pages: Actions → 找到之前的成功运行 → `Re-run`

### Q: 多个平台可以同时部署吗？

可以！同一个项目可以部署到多个平台，每个平台有独立域名。

---

## 快速命令参考

```bash
# Netlify
netlify deploy --prod

# Vercel
vercel --prod

# Surge
surge .

# Cloudflare
wrangler pages deploy . --project-name huangli

# Git 推送（触发自动部署）
git add . && git commit -m "update" && git push
```
