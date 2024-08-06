+++
title = "Git 操作合集"
description = " "
date = "2020-12-26T20:51:07+08:00"
toc = false
categories = ["奇技淫巧"]
tags = ["Git"]
slug = "usage-of-git"
draft = false
+++

[Git](https://git-scm.com/) 是一个分布式版本控制软件，本博客的源代码就是使用它来管理的，每次提交都会生成一个节点，每个节点记录了本次相对于上次的改动内容，十分便于管理笔记和代码。更详细的介绍可以看看 [维基百科的介绍](https://zh.wikipedia.org/wiki/Git) ，本文记录一下使用和学习 Git 过程中的一些基本操作和笔记，文末整理了俺保存的一些学习资源。

## 配置项

首先配置编辑器的换行符为 `LF`，编码为 `UTF-8`，然后配置 Git ：

```bash
# 禁止换行符自动转换
git config --global core.autocrlf false
# 开启换行符检查，避免 CRLF / LF 混用
git config --global core.safecrlf true
# 取消中文路径转义，不然中文路径看起来像乱码一样
git config --global core.quotePath false
# 设置大小写敏感，保持 Windows / UNIX 一致性
# 在目录名大小写修改时，commit 可正常提交
git config --global core.ignorecase false
# 可选：添加 username 和邮箱信息
# 这两项在 git commit 提交时需要
git config --global user.name "he-sb"
git config --global user.email "test@example.com"
```

## 命令相关

· 初始化本地仓库

```bash
git init
```

· 添加远程仓库

```bash
git remote add <remote_name> <url>
```

· 向远程仓库提交

```bash
# 第一次提交时需添加 -u 参数关联本地与远程分支
# 提交到远程的同名分支
git push <remote_name> <branch_name>
# 提交到远程的指定分支
git push <remote_name> <local_branch>:<remote_branch>
# 本地与远程同名分支建立关联后可以直接使用以下命令
# 其中 <remote_name> 也可以省略
git push <remote_name>
```

· 从远程仓库拉取更新

```bash
# 指定远程仓库与分支
git pull <remote_name> <remote_branch>:<local_branch>
# 不指定，使用已与本地建立关联的远程仓库与分支
git pull
```

· 查看当前当前仓库已添加的远程仓库

```bash
git remote
```

· 跳过暂存区提交改动

如果本次操作只有对文件的【修改】，而没有【新增】或【删除】文件，可以直接用 `git commit -am "<commit messages>"` 来提交修改，省略掉 `git add` 操作。

`-a` 选项跳过了暂存区，直接提交了所有修改过的已跟踪的文件。

· 利用 commit 信息关闭 Issues

在提交改动到【默认分支】时，通过在 commit message 中加入 `<keyword> #<issues number>` 来使本次提交关闭相应的 Issues，并在对应 Issues 中关联引用本次提交。

其中 `<keyword>` 包括：`close`，`closes`，`closed`，`fix`，`fixes`，`fixed`，`resolve`，`resolves`，`resolved`，下面是一个例子：

```bash
git commit -m "Commit message example. Close #666"
```

· 使 `git status` 信息更加现代化

```bash
git status -sb
```

## 一些学习资源

- [用玩游戏的方式学习 Git - 少数派](https://sspai.com/post/47694)

- [git - 简明指南 助你入门 git 的简明指南，木有高深内容 ;)](http://rogerdudler.github.io/git-guide/index.zh.html)

---

*参考链接：*

1. [Easy-Hexo/CONTRIBUTING.md - 配置-git](https://github.com/EasyHexo/Easy-Hexo/blob/master/.github/CONTRIBUTING.md#%E9%85%8D%E7%BD%AE-git)

2. [GitHub 第一坑：换行符自动转换 · Issue #22 · cssmagic/blog](https://github.com/cssmagic/blog/issues/22)

3. [推送提交到远程仓库 - GitHub Docs](https://docs.github.com/cn/free-pro-team@latest/github/using-git/pushing-commits-to-a-remote-repository)

4. [用commit信息关闭Issue · GitHub秘籍（中文版） · 看云](http://static.kancloud.cn/thinkphp/github-tips/37883)

5. [git 保存账号密码 - sdfwew - OSCHINA - 中文开源技术交流社区](https://my.oschina.net/jettWang/blog/532278)

6. [git 删除本地分支与远程分支_bitcarmanlee的博客-CSDN博客](https://blog.csdn.net/bitcarmanlee/article/details/83505326)