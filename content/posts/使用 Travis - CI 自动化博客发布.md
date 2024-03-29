+++
title = "使用 Travis CI 自动化博客发布"
description = " "
date = "2020-03-27T16:48:47+08:00"
categories = ["奇技淫巧"]
tags = ["travis-ci","github","hugo"]
slug = "using-travis-ci-to-automate-publishing-blogs-on-github-pages"
draft = false
+++

本博客使用 [Hugo](https://gohugo.io/) 生成，部署在 [Github Pages](https://pages.github.com/) 上，新建一篇文章并发布，常规的操作流程是这样的：

```bash
hugo new posts/new-article.md
# 编辑新文章
hugo
cd public
git add .
git commit -m "New article added."
git push
```

这个过程主要有两个问题：

1. 编辑完文章之后的操作，每次都是一样的流程，一次次的手动操作着实蛋疼，应该使用自动化方法来解放双手。

2. git 是一个版本控制工具，只是用来向 Github 上传最终版本的静态文件实在太不优雅。

经过一番学习，发现不少大佬都在用 [Travis CI](https://travis-ci.org/) 这个持续集成（ Continuous Integration ，简称 CI ）服务来使静态博客的发布过程自动化，思路为：

1. 使用 git 来管理源代码，即用于生成静态站点的配置文件及文章的 `markdown` 源文件等（以下统一称为“源文件”）。

2. `git push` 源文件至 Github 后会自动根据配置文件调用 Travis CI 的服务来完成生成静态站点文件及部署至 Github Pages 等后续操作，不需手动干预。

以下记录一下俺的配置过程。

## 1.申请 Access Token

为了使 Travis CI 有权限接触 Github 仓库，需要生成一个 Personal Access Token，在这个地址可以新建一个：[https://github.com/settings/tokens/new/](https://github.com/settings/tokens/new/) ，名字随便填一个，勾选上 `repo` 内所有项目，其他项目均取消勾选，最后点底下的 `Generate token` 生成 Token。

记下 Token 的值，建议找个空白文档先粘贴保存一下。这个值只会在生成时显示一次，离开此页面后就无法再次查看了，要是忘了就只能重新生成一个。

## 2.Travis CI 网站设置

首先使用 Github 账号登陆，勾选上存放博客【源文件】的仓库，点击 `Settings` ，设置 `Environment Variables` ：

* `Name` : 填写 `GITHUB_TOKEN`
* `Value` : 填写刚刚保存的 Token

填写完毕后点击 `Add` 保存。

接下来的操作分两种情况：

1. [源文件与站点文件在同一 repo 的不同分支](#31-源文件与站点文件在同一-repo)。

2. [源文件与站点文件在不同 repo](#32-源文件与站点文件在不同-repo)。

下面分别进行配置。

## 3.编辑 `.travis.yml` 配置文件

### 3.1 源文件与站点文件在同一 repo

本博客就是这种情况，仓库地址：[https://github.com/he-sb/tech/](https://github.com/he-sb/tech/) ，源文件所在分支为 `master` ，站点文件所在分支为 `gh-pages` 。

在源文件仓库的根目录下新建一个 `.travis.yml` 文件，并写入以下内容：
```yaml
os: linux
dist: bionic
language: python  # python 环境启动比 Golang 耗时少，hugo 为编译好的二进制文件，语言环境不影响 hugo 执行

python:
  - "3.7"  # 指定 Python 3.7

git:
  depth: 1  # 只 clone 最后一次 commit ，加快速度
  quiet: true

env:
  global:
    # 设定 Github Pages 环境变量
    - GH_REF: github.com/he-sb/tech

# 分支白名单限制：只有 master 分支的提交才会触发构建
branches:
  only:
    - master

install:  # 安装依赖
  # 安装 hugo （version: v0.66.0）
  - wget -q -O hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.66.0/hugo_extended_0.66.0_Linux-64bit.deb
  - sudo dpkg -i hugo.deb

script:
  - hugo --minify  # 生成网站

deploy:
  provider: pages  # 部署到 Github Pages
  skip_cleanup: true  # 必须为 true ，否则 Travis 会删除在构建期间创建的所有文件（即删除了要上传的文件）
  local_dir: public  # 被推送到 GitHub Pages 的目录，可以指定为当前目录的绝对路径或相对路径
  on:
    branch: master  # 博客源码所在分支
  target_branch: gh-pages  # 将 local-dir 强制推送到哪个分支（自定义，名称不能与源代码分支名相同），默认为 gh-pages
  token: $GITHUB_TOKEN
  strategy: git
  keep_history: true  # 保持 target-branch 分支的提交记录
```

### 3.2 源文件与站点文件在不同 repo

俺的杂谈随想类博客就是这种情况，源文件仓库地址为：[https://github.com/he-sb/blog/](https://github.com/he-sb/blog/) ，站点文件仓库：[https://github.com/he-sb/he-sb.github.io/](https://github.com/he-sb/he-sb.github.io/) ，源文件与站点文件所在分支均为对应仓库的 `master` 分支。

在源文件仓库的根目录下新建 `.travis.yml` 文件，并写入以下内容：

```yaml
os: linux
dist: bionic
language: python  # python 环境启动比 Golang 耗时少，hugo 为编译好的二进制文件，语言环境不影响 hugo 执行

python:
  - "3.7"  # 指定 Python 3.7

git:
  depth: 1  # 只 clone 最后一次 commit ，加快速度
  quiet: true

env:
  global:
    # 设定 Github Pages 环境变量
    - GH_REF: github.com/he-sb/he-sb.github.io.git

# 分支白名单限制：只有 master 分支的提交才会触发构建
branches:
  only:
    - master

install:  # 安装依赖
  # 安装 hugo （version: v0.66.0）
  - wget -q -O hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.66.0/hugo_extended_0.66.0_Linux-64bit.deb
  - sudo dpkg -i hugo.deb

script:
  - hugo --minify  # 生成网站

after_script:  # 部署至 Github Pages
  - cd ./public
  - git init
  - git config user.name "he-sb"
  - git config user.email "0he.sb0@gmail.com"
  - git add .
  - git commit -m "Update Blog By TravisCI With Build $TRAVIS_BUILD_NUMBER"
  - git push --force --quiet "https://$GITHUB_TOKEN@${GH_REF}" master:master
```

配置完毕后，只要有源文件仓库有新的 `push` 操作，Travis CI 即会按照编辑好的配置文件来自动化站点的构建和部署，也就是说，俺只需操心源文件的编辑改动即可，不用再手动部署 Github Pages 了，方便了许多。

## 4.额外技巧

* 如果某次 `git commit` 不想被自动构建（比如新建了一篇文章，但还没写完，处于草稿状态，不想触发自动构建，那么可以在 `commit` 信息中加上 `[ci skip]` ，比如下面这样：

    ```bash
    git commit -m "[ci skip] commit message"
    ```

    详细参数请看 Travis-CI 官方文档： [Customizing the Build # Skipping a Build - Travis CI](https://docs.travis-ci.com/user/customizing-the-build#skipping-a-build)

* 个人使用时以上两种情况均采用 [【3.2 源文件与站点文件在不同 repo】](#32-源文件与站点文件在不同-repo) 的配置，把最后的 `master:master` 冒号后面的分支名改成比如 `gh-pages` 就可以了，实测速度会快很多，仅需 22s ，使用 Travis-CI 的 Deploy 配置需要 30s + ，有点慢。

    缺点是部署 GitHub Pages 的仓库/分支会失去提交历史（因为部署时的 `git push` 使用了 `--force` 参数），不过无伤大雅，反正这个仅是最终发布版本，源码部分的提交历史不受影响。

* 在博客根目录下新建一个 `.gitignore` 文件，里面写上不需要被 git 追踪的文件或文件夹，可以避免上传无用文件浪费带宽和仓库体积，下面是俺个人的 `.gitignore` 文件内容，供大家参考：

    ```conf
    public
    resources
    hugo.exe
    ```

其中 `public` 文件夹是本地生成的网站静态文件，因为使用了 CI 服务进行自动构建，所以不需要同步这个，仅用于在本地调试使用；`resource` 是在本地运行 `hugo server` 命令预览调试时生成的资源文件夹，同理也不需要同步；`hugo.exe` 是本地的 hugo 程序，同样是本地调试用的。

## 5.尾声

好了，现在发布新文章只需要在博客根目录下执行：

```bash
git add .
git commit -m "New article added."
git push
```

三行命令就可以搞定，博客的构建和发布全部是 CI 自动完成的，git 只用来对博客源码进行版本控制，十分安逸。

另外，如果是对已存在的文章或草稿的修改，没有新增或删除文件的话，可以用 `git commit` 的 `-a` 参数来跳过暂存区，只需两行命令就可以了：

```bash
git commit -am "Article updated."
git push
```

又可以省一个步骤 :)

~~该方案目前还存在一点小 bug —— 每次自动构建并发布后网站内所有文章的修改日期均为构建时的最新时间，目前并不知道该如何修改……~~

~~附上在本博客主题下提交的 Issue 供大家参考：[开启了 git info 但 Travis-CI 自动构建的文章修改日期始终为最新 · Issue #107 · reuixiy/hugo-theme-meme](https://github.com/reuixiy/hugo-theme-meme/issues/107) 。~~

bug 解决了，顺便迁移了博客的自动构建平台到 GitHub Actions：[使用 GitHub Actions 自动化博客发布](/posts/using-travis-ci-to-automate-publishing-blogs-on-github-pages/) 。

---

*参考链接：*

1. [使用 Travis CI 自动部署 Hugo 博客 | Mogeko`s Blog](https://mogeko.me/2018/028/)

2. [Hugo 博客使用 Travis CI 部署 - A simple blog](https://rileyng.github.io/post/hugo-travis/)

3. [使用 Travis CI 自动更新 GitHub Pages | 大专栏](https://www.dazhuanlan.com/2019/10/16/5da60e2b04440/#%E5%A6%82%E4%BD%95%E8%B7%B3%E8%BF%87%E8%87%AA%E5%8A%A8%E6%9E%84%E5%BB%BA)
