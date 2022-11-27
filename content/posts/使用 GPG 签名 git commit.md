+++
title = "使用 GPG 签名 Git Commit"
description = " "
date = "2022-11-27T15:16:18+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["gpg","git","github"]
slug = "signaturing-git-commit-with-gpg"
draft = false
+++

## 前言

在 GitHub 上提交代码时，GitHub 会根据【 `git commit` 时的邮箱】和【 `git push` 时使用的用户名和密码】来记录提交信息，其中前者用来标识【谁提交了这次改动】，后者用来对【 `git push` 这个行为】鉴权。

关于后者，GitHub 做了一系列的努力来提高安全性，比如 [从 2021 年 8 月 13 日起，不再支持使用用户名密码来验证 git 操作](https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/) 。通常我们可以认为在【推送代码至远程仓库（也就是 GitHub）】这个环节是没有问题的。

但是对于前者，也就是 `git commit` 的时候，git 默认并没有验证提交人的身份。也就是说，理论上在默认配置下， `git commit` 时任何人都可以伪装成任何用户，因为此时只要配置一下 git 的邮箱就好了，而 git 并没有验证这个 commit 的人是否是对应邮箱的所有者。

为了解决这个问题，可以使用 GPG 私钥来给 git commit 签名（加密），然后把公钥公开。这样一来，想要验证 commit 是否确实是合法的，只需要将签名后的 commit 使用公钥来解密就可以了。关于 GPG 和非对称加密的原理，网上已经有很多高水平的文档了，俺这里就简单一点，只记录一下如何配置 git，在 commit 时自动使用 GPG 签名。

## 生成密钥对

上文提到，使用 GPG 签名时，需要用私钥来签名，其他人需要验证时，使用公钥来验证。

首先生成一对公钥和私钥：
1. 终端输入 `gpg --full-generate-key`
2. 提示选择密钥类型，输入选择后回车确认，或者直接回车，使用默认的 RSA（推荐）
3. 提示输入密钥长度，直接回车可以使用默认值（在俺这里默认长度是 3072），但是 GitHub 要求最低 4096 位，所以此处输入 `4096`
4. 提示输入密钥有效期，直接回车使用默认的永不过期（推荐），或者自定义一下有效期
5. 提示确认上面的选择，如果需要修改，可以直接回车，或者输入 `N` 后回车；没问题直接输入 `y` 后，回车确认
6. 提示输入用户标识，也就是用户名，这里可以任意输入（只要 5 个字符），比如直接使用 GitHub 用户名
7. 提示输入电子邮件地址，注意这里需要和 GitHub 帐号绑定的邮箱地址对应，否则 GitHub 无法认出这是你自己提交的 commit
    - 如果 GitHub 上设置了隐藏邮箱，或者使用了别的邮箱，可以参考 GitHub [官方文档](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key) 来配置
8. 提示输入注释，这里是对用户用户标识的注释，可以备注该密钥的用途等， ~~也可以写自己的个性签名~~ 直接回车可以留空
9. 提示是否确认用户信息（用户名，注释，邮箱地址），有要修改的可以输入提示中对应的字符修改，没问题输入 `o` 回车确认
10. 生成密钥，生成完毕时会提示输入私钥密码
11. 完成后会回显生成密钥对的 ID

## 上传公钥至 GitHub

首先导出公钥，公钥文件默认是存储为二进制文件的，将其以 ASCII 码的形式输出：

```bash
gpg --armor --export <密钥ID>
```

复制输出的公钥信息，打开 GitHub 设置中的 [SSH and GPG keys](https://github.com/settings/keys) ，点击 `New GPG key`，将刚才复制的公钥粘贴进 `Key` 中，`Title` 可以留空，也可以任意填写。

点击 `Add GPG key`，此时可能会要求输入 GitHub 密码或者 2FA 验证码用于验证。

至此，公钥已经添加至 GitHub.

## 配置本地 git 使用私钥签名 commit

很简单，两行命令：

```bash
# 配置 git 使用哪个密钥
git config --global user.signingkey <密钥ID>
# 配置 commit 时使用密钥签名
git config --global commit.gpgsign true
```

如果需要同时签名 tag，那么再加一行：

```bash
git config --global tag.forcesignannotated true
```

经过以上配置，每次本地进行 `git commit` 操作时，git 都会使用配置的私钥对 commit 内容签名， `push` 至 GitHub 时，GitHub 会使用上传的 GPG 公钥来验证签名后的 commit，验证通过后会显示一个 [标识](https://docs.github.com/cn/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits) ，来表明这个 commit 是经过验证的。

---

*参考链接：*

1. [使用 GPG 签名你的 Git Commit | Mogeko`s Blog](https://mogeko.me/posts/zh-cn/065/)
2. [Generating a new GPG key - GitHub Docs](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/generating-a-new-gpg-key)
