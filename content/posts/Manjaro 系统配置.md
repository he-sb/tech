+++
title = "Manjaro 系统配置"
description = " "
date = "2020-04-15T20:59:24+08:00"
categories = ["Linux"]
tags = ["Manjaro"]
slug = "configuration-after-installing-manjaro"
draft = true
+++

更换了主力系统为 [Manjaro](https://manjaro.org/) ，记录一下配置过程，一来方便自己将来重装后的恢复，二来方便有需要的朋友们做个参考。

Manjaro 官方镜像按照桌面环境的不同，分为 XFCE ，KDE ，GNOME 三个版本，个人选择的是 KDE 桌面环境，更为顺手一点。

## 1.替换自带的 Vi 为 Vim

系统自带的 Vi 体验简直令人发指，没有方向键， Backspace 也会变成莫名其妙的符号，直接卸载：

```bash
sudo pacman -Rs vi
```

装上完全体的 Vim ：

```bash
sudo pacman -Sy vim
```

## 2.添加 ArchLinuxCN 源

首先编辑 `/etc/pacman.conf` 这个文件，在末尾添加以下内容：

```conf
[archlinuxcn]
SigLevel = Optional TrustedOnly
Server = https://repo.archlinuxcn.org/$arch
```

然后导入 archlinuxcn-keyring ：

```bash
sudo pacman -S archlinuxcn-keyring
```

最后刷新一下缓存：

```bash
sudo pacman -Syy
```

## 3.替换默认的 Shell 为 zsh

安装 zsh ：

```bash
sudo pacman -S zsh
```

<!-- todo -->

## 附：包管理说明

### pacman

### yay

---

*参考链接：*

1. [与Manjaro相见恨晚 - 山炮不二](https://xsinger.me/diy/857.html)

2. [Manjaro 个人新装配置 | 禾七博客](https://leay.net/2019/12/18/manjaro/)