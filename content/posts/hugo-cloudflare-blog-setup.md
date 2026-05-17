+++
title = "用 Hugo + Cloudflare Pages 搭建个人博客"
date = 2026-05-17
draft = false
categories = ["技术"]
tags = ["hugo", "cloudflare", "博客搭建"]
+++

一直想搭一个个人博客，要求很简单：轻量、免费、写完就能发布。折腾了一圈，最后选了 Hugo + Cloudflare Pages 的方案。整个过程不到半小时，记录下来分享给有同样需求的朋友。

## 整体架构

```
本地写文章（Markdown）
  ↓ git push
GitHub 仓库（hugo-blog）
  ↓ 自动触发
Cloudflare Pages（构建 + 托管）
  ↓
xxx.pages.dev（上线）
```

每一层只做一件事：Hugo 负责把 Markdown 变成网页，GitHub 存源码，Cloudflare 负责构建和发布。全链路免费，维护成本几乎为零。

## 需要准备的东西

- Git（macOS 自带）
- Hugo（静态站点生成器）
- GitHub 账号
- Cloudflare 账号（免费）
- 一个终端

不需要域名，Cloudflare Pages 会免费分配一个 `xxx.pages.dev` 的地址。

## 第一步：安装 Hugo

macOS 用 Homebrew 一行搞定：

```bash
brew install hugo
```

安装完验证一下：

```bash
hugo version
```

## 第二步：初始化项目

```bash
cd ~/Workspace
hugo new site hugo-blog
cd hugo-blog
git init
```

## 第三步：安装主题

选主题是个见仁见智的事。我用了 PaperMod，简洁、现代、中文支持好，而且不依赖 Dart Sass（有些主题需要额外装 Sass，容易踩坑）。

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

> 踩坑记录：最开始试了 hugo-texify3 主题，构建时报错 `TOCSS-DART: failed to transform`，因为 Hugo extended 版本虽然支持 SCSS，但 Dart Sass 需要单独安装，而 macOS 上装 Dart 又遇到 Command Line Tools 版本问题。换 PaperMod 后一切顺利，它用的是纯 CSS。

## 第四步：配置站点

编辑 `hugo.toml`：

```toml
baseURL = "https://hugo-blog-4ai.pages.dev/"
title = "My Blog"
theme = "PaperMod"
hasCJKLanguage = true

[taxonomies]
tags = "tags"
categories = "categories"

[params]
defaultTheme = "auto"
ShowReadingTime = true
ShowShareButtons = false
ShowPostNavLinks = true
ShowBreadCrumbs = true
ShowCodeCopyButtons = true

[params.homeInfoParams]
Title = "Hi there 👋"
Content = "Welcome to my blog"

[[params.socialIcons]]
name = "github"
url = "https://github.com/wfl36"

[menu]
[[menu.main]]
identifier = "posts"
name = "Posts"
url = "/posts/"
weight = 10
[[menu.main]]
identifier = "tags"
name = "Tags"
url = "/tags/"
weight = 20
[[menu.main]]
identifier = "categories"
name = "Categories"
url = "/categories/"
weight = 30
```

`baseURL` 先填 Cloudflare Pages 分配的地址，后面买了域名再改。

## 第五步：创建 .gitignore

```bash
cat > .gitignore << EOF
public/
.hugo_build.lock
EOF
```

`public/` 是 Hugo 的构建输出目录，不应该提交到仓库——Cloudflare Pages 会自己构建。

## 第六步：写第一篇文章

```bash
hugo new content posts/my-first-post.md
```

编辑生成的文件，注意把 `draft = true` 改成 `draft = false`，否则不会显示：

```markdown
+++
date = '2026-05-17'
draft = false
title = 'My First Post'
categories = ['notes']
tags = ['hello']
+++

Hello World!
```

本地预览一下：

```bash
hugo server --buildDrafts
```

打开 http://localhost:1313 看效果，满意了就继续。

## 第七步：推送到 GitHub

先在 GitHub 上创建一个空仓库 `hugo-blog`（不要勾选 Initialize with README），然后：

```bash
cd ~/Workspace/hugo-blog
git remote add origin git@github.com:wfl36/hugo-blog.git
git add -A
git commit -m "init: Hugo blog with PaperMod theme"
git push -u origin main
```

## 第八步：配置 Cloudflare Pages

1. 打开 https://dash.cloudflare.com → Workers & Pages
2. 创建应用 → Pages → Connect to Git
3. 授权 GitHub，选择 `hugo-blog` 仓库
4. 构建配置：

| 字段 | 值 |
|------|------|
| 构建命令 | hugo |
| 输出目录 | public |
| 环境变量 HUGO_VERSION | 0.161.1 |

5. 保存并部署

> 踩坑记录：如果不设置 `HUGO_VERSION` 环境变量，Cloudflare 会用默认的旧版本 Hugo，可能导致构建失败或页面 522 超时。加上版本号后一切正常。

部署完成后，几分钟后就能通过 `https://hugo-blog-4ai.pages.dev` 访问了。

## 日常发布流程

整个博客搭好后，发新文章只需要三步：

```bash
# 1. 创建文章
hugo new content posts/my-article.md

# 2. 编辑内容，改 draft = false

# 3. 推送
git add -A && git commit -m "add: my-article"
git push
```

推送后 Cloudflare 自动构建，几分钟内上线。

## 其他操作

| 操作 | 方法 |
|------|------|
| 删文章 | 删除 `content/posts/` 下的 md 文件 → git push |
| 改文章 | 编辑 md 文件 → git push |
| 改站点名 | 编辑 `hugo.toml` 的 title → git push |
| 换主题 | 换 theme 字段 → git push |
| 本地预览 | `hugo server --buildDrafts` |

## 后续可以做的事

- **买域名**：在 Cloudflare 买一个喜欢的域名，绑定到 Pages 项目，SSL 和 DNS 自动处理
- **接入思源笔记**：安装 `siyuan-plugin-publisher` 插件，在思源里写完一键发布，不用碰终端
- **加评论系统**：接入 Giscus（基于 GitHub Discussions），免费且无广告
- **加统计**：接入 Cloudflare Web Analytics，免费且隐私友好

## 总结

这套方案的核心优势：

- **免费**：Cloudflare Pages 免费额度完全够个人博客用
- **轻量**：不需要数据库、不需要后台、不需要维护服务器
- **简单**：写 Markdown → git push → 自动上线
- **快速**：Hugo 构建速度极快，Cloudflare CDN 全球加速

个人博客最重要的是让写作和发布尽可能简单，而不是把时间花在搭环境上。这套方案做到了。
