+++
title = "使用 GitHub Actions 自动化博客发布"
description = " "
date = "2020-12-12T14:52:32+08:00"
categories = ["奇技淫巧"]
tags = ["actions","github","hugo"]
slug = "using-github-actions-to-automate-publishing-blogs-on-github-pages"
draft = false
+++

本博客之前是 [通过 Travis CI 来自动构建](/posts/using-travis-ci-to-automate-publishing-blogs-on-github-pages) 的，而 [Actions](https://github.com/features/actions) 是 GitHub 自家的 CI 服务，提供的构建环境更好，配置更容易，也和 GitHub 其他服务比如 Pages 结合的更紧密，从结果来看，体验还是非常不错的，下面是俺的配置过程，供参考。

## 修改仓库的 Pages 选项

GitHub Pages 现在支持直接通过 Actions 来自定义部署 Pages，首先打开 https://github.com/he-sb/tech/settings/pages ，找到 `Build and deployment` 部分，将 `Source` 选项选择为 `GitHub Actions`.

> 另一个选项 `Deploy from a branch` 是旧版的默认行为，此时会选择一个分支内的文件作为静态网页的内容直接部署为 Pages 网站，如果你将构建好的静态网站上传为仓库的一个分支，那么可以选择此项。

## 编辑配置文件

GitHub Ac­tions 的配置文件叫做 work­flow 文件（官方中文翻译为“工作流程文件”），存放在代码仓库的 `.github/workflows` 路径下。和 Travis CI 的配置文件一样采用 YAML 格式。文件名任意，后缀名统一为 `.yml`，只要 `.github/workflows` 目录里面有 `.yml` 文件，GitHub 就会按照文件中所指定的触发条件在符合条件时自动运行该文件中的工作流程。

那么先新建一个配置文件，路径为 `.github/workflows/build-on-GitHub-Pages.yml`，此处俺使用的文件名是 `build-on-GitHub-Pages.yml`，这个文件用来定义 GitHub Actions 自动构建时将如何操作。

以下是文件内容，必要的部分已经添加了注释，也可以直接 [访问仓库查看最新版本](https://github.com/he-sb/tech/blob/master/.github/workflows/build-on-GitHub-Pages.yml)：

```yaml
name: build-on-Github-Pages

# 在 master 分支更新时触发构建
on:
  push:
    branches:
      - master

jobs:
  build:
    runs-on: ubuntu-24.04
    env:
      TZ: Asia/Shanghai
      SOURCE_REPO: "he-sb/tech"  # 博客源码仓库
    steps:
      # 配置 git，避免一些莫名其妙的错误
      - name: Git Configuration
        run: |
          git config --global core.quotePath false
          git config --global core.autocrlf false
          git config --global core.safecrlf true
          git config --global core.ignorecase false
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
      # 拉取源码
      - name: Clone Repository
        run:
          git clone --branch=master --quiet https://github.com/$SOURCE_REPO site
      # 安装 hugo
      - name: Setup Hugo
        env:
          HUGO_VERSION: 0.155.1  # hugo 版本号
        run: |
          wget -q -O hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_Linux-amd64.deb && \
          sudo dpkg -i hugo.deb && \
          hugo version
      # 构建
      - name: Build
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run:
          cd site && hugo --gc --minify --cleanDestinationDir
      # 上传构建结果
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: site/public/
  deploy:
    needs: build  # 依赖上一步构建成功
    runs-on: ubuntu-24.04
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

几点说明：

- 【拉取源码】这一步的命令是从源码所在仓库的 `master` 分支拉取的，如果你的情况比较特殊，那么需要修改命令中的 `--branch` 参数；
- 其他必须修改的参数对照注释很容易就能看懂，就不多解释了；
- 修改完成之后，只要把这个配置文件 push 上去就可以自动构建了。

## 结语

本博客之前使用 Travis CI 时有个历史遗留问题，就是每次自动构建完毕后，所有文章的修改时间都会变成【执行自动构建】的时间（详见 [这篇文章](/posts/using-travis-ci-to-automate-publishing-blogs-on-github-pages/) 的【尾声】部分），而这个问题在刚转到 GitHub Actions 时依然存在。

经过仔细对比 Actions 和 Travis CI 的构建日志，俺终于确认了问题是 `git checkout` 操作引起的（详情见 [这个 Issues](https://github.com/reuixiy/hugo-theme-meme/issues/107) 中的相关讨论），但还不确定这是 hugo 的问题还是 MemE 主题的问题。

俺的解决办法是手写 Actions 配置文件，而不是像很多教程中提到的通过组合已有的 actions 来实现，虽然看起来逼格不够，但确实解决了问题，而且也比较方便复用。

---

*参考链接：*

1. [GitHub Actions 入门教程 - P3TERX ZONE](https://p3terx.com/archives/github-actions-started-tutorial.html)

2. [使用 Github Actions 來自動化部署 Hugo 到 Github Pages | Puck's Blog](https://blog.puckwang.com/post/2020/use-github-actions-deploy-hugo/)

3. [Environment variables - GitHub Docs](https://docs.github.com/en/free-pro-team@latest/actions/reference/environment-variables)

4. [Authentication in a workflow - GitHub Docs](https://docs.github.com/en/free-pro-team@latest/actions/reference/authentication-in-a-workflow)

5. [Encrypted secrets - GitHub Docs - #Using encrypted secrets in a workflow](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets#using-encrypted-secrets-in-a-workflow)

6. [Context and expression syntax for GitHub Actions - GitHub Docs - #github context](https://docs.github.com/en/free-pro-team@latest/actions/reference/context-and-expression-syntax-for-github-actions#github-context)

7. [Adding a workflow status badge - GitHub Docs](https://docs.github.com/en/free-pro-team@latest/actions/managing-workflow-runs/adding-a-workflow-status-badge)
