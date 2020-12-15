+++
title = "Github Pages + Hugo 搭建个人博客"
description = " "
date = "2020-04-24T16:22:30+08:00"
categories = ["奇技淫巧"]
tags = ["github","actions","travis-ci","cf","cdn","hugo"]
slug = "build-personal-blog-with-hugo-and-github-pages"
draft = false
+++

## 前言

本博客是使用 [Hugo](https//gohugo.io/) 生成的纯静态博客，部署在 [Github Pages](https://pages.github.com/) 上。这里讲解一下搭建过程，希望能帮到有需要的朋友。

准备工作

- 一个 GitHub 帐号；
- 挑选一个主题（比如本博客使用的 [MemE](https://github.com/reuixiy/hugo-theme-meme) ），如果不喜欢本博客的主题，文末的【参考链接】部分放了些俺觉得不错的主题可以看看。

## 安装 Hugo

Hugo 是使用 Go 语言开发的，编译好的二进制文件不需要任何依赖，直接去官方 GitHub 仓库下载符合你操作系统的版本并解压就可以用了，可以参考 [官网的安装指导](https://gohugo.io/getting-started/installing/) 。

> 注意：部分主题要求使用 `extended` 版本的 Hugo，比如本博客使用的 MemE，请仔细阅读主题的文档，不要装错版本了。

Windows 安装：

直接下载合适版本的 zip 压缩包，比如 [hugo_extended_0.79.0_Windows-64bit.zip](https://github.com/gohugoio/hugo/releases/download/v0.79.0/hugo_extended_0.79.0_Windows-64bit.zip) ，然后解压到任意目录，比如 `D:\hugo`，解压后的可执行文件路径就是 `D:\hugo\hugo.exe`，把这个目录添加到系统环境变量（方法是：开始 -> 搜索“高级系统设置”并打开 -> 点击“环境变量” -> 点击上方“用户变量”或下方“系统变量”内的 `Path` 行，再点击对应文本下方的“编辑” -> 点击右侧“新建” -> 将解压后的文件夹完整路径粘贴至此处并保存即可，本例中路径为 `D:\hugo\` ）。

同时推荐 Windows 用户使用开源的 [cmder](https://cmder.net/) / [Git-bash](https://git-scm.com/downloads) 代替系统自带的终端（cmd / PowerShell），它们均集成了 git 等常用命令，非常好用。

Linux 安装：

下载合适版本的 tar 压缩包，比如 [hugo_extended_0.79.0_Linux-64bit.tar.gz](https://github.com/gohugoio/hugo/releases/download/v0.79.0/hugo_extended_0.79.0_Linux-64bit.tar.gz) ，解压后添加执行权限，然后放到 `usr/local/bin/` 目录就可以了；也可以直接用包管理器来安装，缺点是需要特定版本的 Hugo 时可能不太方便：
- Arch 系（比如俺使用的 Manjaro）：`sudo pacman -Sy hugo`
- Debian / Ubuntu：`sudo apt install hugo`
- Fedora：`sudo dnf install hugo`

## 本地生成博客

建立新网站：

```bash
hugo new site blog
```

会在执行命令时终端所在的目录（以 `~` 为例）生成一个名为 `blog` 的文件夹，里面是用于生成静态网站的源代码，本文后续部分提到的【博客目录】均是指这个文件夹（本文中为 `~/blog`）。

> 注意：本文中后续提到的 Hugo 命令都是在【博客目录】内执行的。

进入你喜欢的主题的 GitHub 项目主页，仔细阅读主题的文档（本文以 MemE 主题为例），安装主题，并按照文档说明来修改【博客目录】下的配置文件（`~/blog/config.toml`）。不同主题的安装方式大同小异，不过最终都是安装到【博客目录】下的 `themes` 文件夹内。

看下博客目录的结构：

```bash
blog
├── archetypes
├── config.toml  # 网站配置文件
├── content  # 博客内容存放在这里
├── data
├── layouts
├── public  # Hugo 生成的静态博客
├── static
└── themes
   └── meme  # 博客的主题文件夹
```

主题配置好后，新建一篇文章：

```bash
hugo new posts/hello_world.md
```

Hugo 默认的网站内容目录是【博客目录】内的 `content` 文件夹，因此上面这条命令生成的新文章会出现在 `~/blog/content/posts/hello_world.md`。

接下来运行这个命令：

```bash
hugo server -D
```

不出意外的话，此时浏览器访问 `http://localhost:1313/` 就可以在本机预览网站了，此时改动【博客目录】下的文件，比如修改网站配置（`config.toml`），新建文章，修改文章等，网页会实时刷新，便于你直观地查看改动效果。

此时不妨用编辑器打开新建的 `hello_world.md`，使用 Markdown 语法（不懂的话可以查看这个项目花几分钟学习一下：[younghz/Markdown: Markdown 基本语法。](https://github.com/younghz/Markdown)）写点东西，然后回到网页看看效果。

要停止预览，回到终端内，按组合键 Ctrl + C 终止上条命令就可以了。

如果预览没问题，就可以生成网站了：

```bash
hugo
```

是的，Hugo 生成网站的命令就是这么简洁……

生成的静态网站文件在【博客目录】内的 `public` 文件夹（`~/blog/public/`），后续需要部署到 GitHub Pages 上的就是这个目录下的所有文件。

Hugo 更详细的使用方式及命令参数可以执行 `hugo help` 或前往官网查看。

## 部署到 GitHub Pages

首先去 GitHub 新建一个仓库，名字叫 `<username>.github.io`，其中 `<username>` 是你的 GitHub 用户名，其他选项随便看着选就好了，之前私有仓库好像不能开启 Pages，现在也没有这个限制了，看心情选择即可。

创建好后去仓库的设置，一直往下拉，到 `GitHub Pages` 这节：
- `Source`：定义了你的静态网站的文件所在的分支和路径，不清楚如何设置的话保持默认的（`master` / `main` 和 `/(root)`）就好；
- `Theme Chooser`：忽略，这个是用来选择 GitHub Pages 自带的 Jekyll 生成器的主题，本文使用的是 Hugo，不需要这个；
- `Custom domain`：留空的话博客会使用 `<username>.github.io`，如果你有自己的域名，那么可以在这里写上，只写域名就可以了，不要带 `http://` 或者 `https://` 这类协议名，例如本博客的 `tech.he-sb.top`；
- `Enforce HTTPS`：这个先不勾选，部署完毕查看正常之后再打开。

设置完成之后，在【博客目录】内打开终端：

```bash
# 进入 public 文件夹
cd public
# 初始化 git
git init
# 添加远程仓库
# 也就是刚刚在 GitHub 开启了 Pages 服务的仓库
git remote add origin https://github.com/<username>/<username>.github.io.git
# 将文件夹内所有文件添加到暂存区
git add .
# 提交改动
git commit -m "Initialize"
# 将改动推送到远程仓库
# 并将本地仓库的 master 分支和远程仓库的同名分支建立关联
git push -u origin master
```

就可以推送本地生成好的网站文件到 GitHub 仓库内了。

之后如果新建或更新了文章，需要进入【博客目录】内打开终端，然后执行：

```bash
# 首先生成新的网站文件
hugo
# 然后还是进入 public 目录
cd public
# 添加所有改动到暂存区
git add .
# 提交改动
git commit -m "Blog updated"
# 推送到远程仓库
git push
```

至于 git 的用法，网上教程非常多，俺就不再浪费口水啦。

## 进阶

- 如果要走 CF 的 CDN ，去添加一条指向 `<username>.github.io` 的 CNAME 记录，此时先把云朵图标点灰掉，然后在 Github Pages 设置内勾选 `Enforce HTTPS` ，等待证书申请完毕后再回到 CF 的 `DNS` 页签点亮云朵图标，并确保 `SSL` 页签的模式为 `Full`，不然可能会遇到一些奇奇怪怪的证书问题；

- 本文介绍的是将博客部署至 `<username>.github.io` 仓库的方案，实际上 GitHub Pages 可以实现的远比这种方案要灵活，比如：

    - 网站源码和网站文件存放在同一个仓库的不同分支（或文件夹）
    - 同时部署多个网站并分别设置自定义域名
    - 通过 CI 服务实现源码推送后自动构建并部署网站

    而且这些都是可以同时实现的，感兴趣的话可以自行摸索一下；

- 如果你也想像俺一样通过推送源码来自动构建，可以参考俺的教程：
    - [使用 Travis CI 自动化博客发布](/posts/using-travis-ci-to-automate-publishing-blogs-on-github-pages/)
    - [使用 GitHub Actions 自动化博客发布](/posts/using-github-actions-to-automate-publishing-blogs-on-github-pages/)（推荐这篇）

---

*参考链接：*

1. [使用 Hugo 生成静态博客教程 - 烧饼博客](https://sb.sb/blog/migrate-to-hugo/)

2. [Hugo 主题 MemE 文档 | reuixiy](https://io-oi.me/tech/documentation-of-hugo-theme-meme/)

3. 一些个人觉得不错的 Hugo 主题：
    - [AmazingRise/hugo-theme-diary: Moments piled up. A Hugo theme ported from SumiMakito/hexo-theme-Journal.](https://github.com/AmazingRise/hugo-theme-diary)
    - [dillonzq/LoveIt: ❤️A clean, elegant but advanced blog theme for Hugo 一个简洁、优雅且高效的 Hugo 主题](https://github.com/dillonzq/LoveIt)
    - [MeiK2333/github-style](https://github.com/MeiK2333/github-style)
    - [mrmierzejewski/hugo-theme-console: A minimal, responsive and light theme for Hugo inspired by Linux console.](https://github.com/mrmierzejewski/hugo-theme-console)
    - [MunifTanjim/minimo: Minimo - Minimalist theme for Hugo](https://github.com/MunifTanjim/minimo)
    - [xianmin/hugo-theme-jane: A readable & concise theme for Hugo](https://github.com/xianmin/hugo-theme-jane)
    - [yihui/hugo-xmin: eXtremely Minimal Hugo theme: about 150 lines of code in total, including HTML and CSS (with no dependencies)](https://github.com/yihui/hugo-xmin)