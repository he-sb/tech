+++
title = "Github Pages + Hugo 搭建个人博客"
description = " "
date = "2020-04-24T16:22:30+08:00"
categories = ["奇技淫巧"]
tags = ["github","actions","travis-ci","cf","cdn","hugo"]
slug = "build-personal-blog-with-hugo-and-github-pages"
draft = true
+++

## 前言

[Hugo](https://gohugo.io/)
[GitHub Pages](https://pages.github.com/)

<!-- todo -->

## 准备工作

- 一个 GitHub 帐号
- 挑选一个主题（比如 [MemE](https://github.com/reuixiy/hugo-theme-meme) ）

## 安装 Hugo

<!-- todo -->

## 本地生成博客

在一个你喜欢的路径打开终端，比如 `~`，先新建一个名为 `tech` 的站点：

```bash
hugo new site tech
```

<!-- todo -->

## 部署到 GitHub Pages

首先去 GitHub 新建一个仓库，名字叫 `<username>.github.io`，其中 `<username>` 是你的 GitHub 用户名，其他选项随便看着选就好了，之前私有仓库好像不能开启 Pages，现在也没有这个限制了，看心情选择即可。

创建好后去仓库的设置，一直往下拉，到 `GitHub Pages` 这节：
- `Source`：定义了你的静态网站的文件所在的分支和路径，不清楚如何设置的话保持默认的（`master` / `main` 和 `/(root)`）就好；
- `Theme Chooser`：忽略，这个是用来选择 GitHub Pages 自带的 Jekyll 生成器的主题，本文使用的是 Hugo，不需要这个；
- `Custom domain`：留空的话博客会使用 `<username>.github.io`，如果你有自己的域名，那么可以在这里写上，只写域名就可以了，不要带 `http://` 或者 `https://` 这类协议名，例如本博客的 `tech.he-sb.top`。

<!-- todo -->

## 进阶

- 如果要走 CF 的 CDN ，先在 Github Pages 设置内勾选 `Enforce HTTPS` ，等待证书申请完毕后再回到 CF 的 `DNS` 页签点亮云朵图标，并确保 `SSL` 页签的模式为 `Full`，不然可能会遇到一些奇奇怪怪的证书问题；

- 本文介绍的是将博客部署至 `<username>.github.io` 仓库的方案，实际上 GitHub Pages 可以实现的远比这种方案要灵活，比如：
    - 网站源码和网站文件存放在同一个仓库的不同分支（或文件夹）
    - 同时部署多个网站并分别设置自定义域名
    - 通过 CI 服务实现源码推送后自动构建并部署网站  
    而且这些都是可以同时实现的，感兴趣的话可以自行摸索一下；

- 如果你也想像俺一样通过推送源码来自动构建，可以参考俺的教程：
    - [使用 Travis CI 自动化博客发布](/posts/using-travis-ci-to-automate-publishing-blogs-on-github-pages/)
    - [使用 GitHub Actions 自动化博客发布](/posts/using-github-actions-to-automate-publishing-blogs-on-github-pages/) （推荐这篇）

## 结语

<!-- todo -->

GitHub Issues #17

---

*参考链接：*

1. [使用 Hugo 生成静态博客教程 - 烧饼博客](https://sb.sb/blog/migrate-to-hugo/)

2. [Hugo 主题 MemE 文档 | reuixiy](https://io-oi.me/tech/documentation-of-hugo-theme-meme/)