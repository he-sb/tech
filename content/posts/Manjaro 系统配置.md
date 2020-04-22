+++
title = "Manjaro 系统配置"
description = " "
date = "2020-04-15T20:59:24+08:00"
categories = ["Linux"]
tags = ["Manjaro"]
slug = "configuration-after-installing-manjaro"
draft = true
+++

终于告别了辣鸡 Windows 系统了，从 Win 10 到 LTSC ，再到 Windows Server 2019 ，就没有一个不出 bug 的，还是早日投奔 Linux 的怀抱吧。目前更换了主力系统为 [Manjaro](https://manjaro.org/) ，这是一个基于 [Arch Linux](https://www.archlinux.org/) 的衍生发行版本，继承了 Arch 滚动更新和 AUR 软件包极为丰富的优点，稳定性也十分优秀，值得长期使用。Manjaro 官方镜像按照桌面环境的不同，分为 XFCE ，KDE ，GNOME 三个版本，个人选择的是 KDE 桌面环境，更为顺手一点。

下面记录一下配置过程，一来方便自己将来万一重装后的恢复，二来方便有需要的朋友们做个参考。

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
sudo pacman -Sy archlinuxcn-keyring
```

最后刷新一下缓存：

```bash
sudo pacman -Syy
```

## 3.配置环境变量

目的是使终端也走代理，这样就不需要添加国内镜像源了：

<!-- mellow 透明代理/ proxychains /环境变量调用 V2Ray -->

## 4.替换默认的 Shell 为 zsh

安装 zsh ：

```bash
sudo pacman -S zsh
```

<!-- todo -->

替换默认的终端：

1. 先查看当前系统内已安装的终端：

    ```bash
    chsh -l
    ```

2. 更改当前用户的默认终端：

    ```bash
    chsh -s full-path-to-shell
    ```

3. 更改 root 用户的默认终端：

    ```bash
    sudo su
    chsh -s full-path-to-shell
    su none-root-user-name
    ```

## 附：包管理说明

Arch 系的 Linux 发行版，软件包来源有两个，Community（Arch 官方仓库），AUR（Arch User Repository, Arch 用户仓库）。用户将软件放在 AUR ，Arch 官方则定期挑选 AUR 里的优秀程序到 community，实际表现为 Community 为 AUR 的子集，Community 有的应用 AUR 都有，但 AUR 内有而 Community 没有的那部分软件可能在系统上的运行表现不大稳定。

Manjaro 自带的桌面程序软件中心（pamac-manager）既可以安装 Community 程序也可以安装 AUR 程序，区别是 AUR 程序会显示【构建】而不是【安装】；但在终端中通过自带的 pacman 只能安装 Community 程序，想要安装 AUR 程序则需要安装额外的包管理器，之前很多教程内的 yaourt 已经停止维护了，个人使用 yay 。

下面是一些包管理器的常用命令。

### pacman

```bash
sudo pacman -Syu  //同步软件库并更新系统到最新状态
```
<!-- todo -->

### yay

---

*参考链接：*

1. [与Manjaro相见恨晚 - 山炮不二](https://xsinger.me/diy/857.html)

2. [Manjaro 个人新装配置 | 禾七博客](https://leay.net/2019/12/18/manjaro/)