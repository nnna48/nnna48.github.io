# Firefly 博客修改、推送与 GitHub Pages 部署教程

本教程适用于当前项目：

- 本地目录：`E:\Blog\nnna48.github.io`
- GitHub 仓库：`https://github.com/nnna48/nnna48.github.io`
- 网站地址：`https://nnna48.github.io`
- 日常开发分支：`main`
- 旧 Hexo 备份分支：`hexo-archive`
- Firefly 作者仓库：`https://github.com/CuteLeaf/Firefly.git`

## 一、远程仓库说明

执行：

```powershell
git remote -v
```

当前项目使用两个远程仓库：

```text
origin    自己的博客仓库，用于推送和部署
upstream  Firefly 作者仓库，用于获取主题更新
```

日常推送只能使用：

```powershell
git push origin main
```

不要向 `upstream` 推送。

## 二、首次部署前必须检查

### 1. 网站地址

检查 `src/config/siteConfig.ts`：

```ts
site_url: "https://nnna48.github.io",
```

检查 `astro.config.mjs`：

```js
base: "/",
```

因为仓库名就是 `nnna48.github.io`，所以网站部署在根路径，不需要添加仓库子路径。

### 2. 修正 GitHub Actions 监听分支

打开：

```text
.github/workflows/deploy.yml
```

当前开发分支是 `main`，工作流必须写成：

```yaml
on:
  push:
    branches: [main]
  workflow_dispatch:
```

如果写成 `branches: [master]`，推送 `main` 不会自动部署，只能手动运行工作流。

### 3. 开启 GitHub Pages

进入仓库：

```text
Settings → Pages → Build and deployment
```

将 Source 设置为：

```text
GitHub Actions
```

不要选择 `Deploy from a branch`。

## 三、每次修改博客的标准流程

### 1. 进入项目

```powershell
cd E:\Blog\nnna48.github.io
```

### 2. 确认当前分支

```powershell
git branch --show-current
```

应当显示：

```text
main
```

如果不是 `main`：

```powershell
git switch main
```

### 3. 修改前同步自己的远程仓库

先检查工作区：

```powershell
git status
```

只有工作区干净时，才执行：

```powershell
git pull --ff-only origin main
```

如果存在未提交修改，先提交这些修改，不要直接拉取。

### 4. 启动本地开发服务器

```powershell
pnpm dev
```

浏览器访问：

```text
http://localhost:4321
```

修改配置、文章或图片后，开发服务器通常会自动刷新。

### 5. 查看修改内容

```powershell
git status
git diff --stat
git diff
```

重点确认：

- 没有误删文件；
- 没有提交 `.env`、令牌或密码；
- 没有提交 `node_modules`；
- 没有提交 `dist`；
- 图片删除和替换符合预期。

### 6. 执行检查

常规配置或文章修改至少运行：

```powershell
pnpm check
```

准备正式发布或修改了组件时，建议再运行：

```powershell
pnpm type-check
pnpm build
```

如果正在使用 `pnpm dev` 预览简单配置修改，可以稍后再执行正式构建。

### 7. 暂存准备提交的文件

推荐先逐项添加：

```powershell
git add src/config/siteConfig.ts
git add src/config/profileConfig.ts
git add src/config/backgroundWallpaper.ts
git add src/config/effectsConfig.ts
git add src/content
git add src/assets
```

如果已经通过 `git status` 确认所有修改都需要提交，也可以使用：

```powershell
git add -A
```

再次确认暂存内容：

```powershell
git status
git diff --cached --stat
git diff --cached
```

### 8. 创建提交

项目建议使用 Conventional Commits：

```powershell
git commit -m "feat: update blog appearance"
```

常用类型：

```text
feat:    新功能、新文章、新页面
fix:     修复问题
docs:    修改文档
style:   只调整样式
chore:   配置、依赖、部署等维护工作
```

示例：

```powershell
git commit -m "feat: customize profile and homepage"
git commit -m "feat: add a new post"
git commit -m "fix: correct wallpaper configuration"
git commit -m "chore: update GitHub Pages workflow"
```

### 9. 推送到 GitHub

```powershell
git push origin main
```

只推送源码，不需要手动上传 `dist`。

## 四、GitHub 如何自动部署

推送 `main` 后，`.github/workflows/deploy.yml` 会自动：

1. 拉取仓库源码；
2. 安装 Node.js 和 pnpm；
3. 执行 `pnpm install`；
4. 执行 `pnpm run build`；
5. 上传 `dist`；
6. 发布到 GitHub Pages。

进入下面的页面查看进度：

```text
https://github.com/nnna48/nnna48.github.io/actions
```

正常情况下会看到：

```text
build   成功
deploy  成功
```

部署完成后访问：

```text
https://nnna48.github.io
```

GitHub Pages 和浏览器缓存可能导致页面延迟一两分钟更新。

## 五、手动触发部署

如果已经推送但工作流没有自动运行：

1. 打开仓库的 `Actions` 页面；
2. 选择 `Deploy to GitHub Pages`；
3. 点击 `Run workflow`；
4. 选择 `main`；
5. 再次点击绿色的 `Run workflow`。

也可以创建一个空提交触发部署：

```powershell
git commit --allow-empty -m "chore: trigger deployment"
git push origin main
```

前提是 `deploy.yml` 已监听 `main`。

## 六、发布新文章

可以使用项目命令：

```powershell
pnpm new-post 文章文件名
```

也可以在下面的目录手动创建 Markdown：

```text
src/content/posts/
```

例如：

```text
src/content/posts/my-first-post.md
```

对应地址：

```text
https://nnna48.github.io/posts/my-first-post/
```

文章至少应包含：

```yaml
---
title: 文章标题
published: 2026-08-25
description: 文章简介
tags:
  - 标签
category: 分类
draft: false
---
```

正文写在 Frontmatter 下方。

## 七、同步 Firefly 作者更新

同步前必须确保当前修改已经提交：

```powershell
git status
```

然后执行：

```powershell
git switch main
git fetch upstream
git merge upstream/master
```

如果没有冲突：

```powershell
pnpm install
pnpm check
pnpm build
git push origin main
```

如果出现 `CONFLICT`：

1. 不要立即推送；
2. 打开冲突文件；
3. 保留自己的站点资料和需要的上游更新；
4. 删除冲突标记；
5. 重新执行检查；
6. 提交合并结果。

处理完成后：

```powershell
git add -A
git commit
git push origin main
```

## 八、撤销错误操作

### 撤销尚未暂存的单个文件

```powershell
git restore 文件路径
```

### 取消暂存但保留文件修改

```powershell
git restore --staged 文件路径
```

### 撤销已经推送的提交

先查看提交：

```powershell
git log --oneline -10
```

再创建一个反向提交：

```powershell
git revert 提交编号
git push origin main
```

不要轻易使用：

```text
git reset --hard
git push --force
```

旧 Hexo 博客保存在 `hexo-archive` 分支，不要删除该分支。

## 九、最简日常命令清单

```powershell
cd E:\Blog\nnna48.github.io
git switch main
git status
pnpm dev

# 完成修改并停止开发服务器后
pnpm check
git status
git add -A
git diff --cached --stat
git commit -m "feat: update blog"
git push origin main
```

随后前往 GitHub Actions 确认部署成功。
