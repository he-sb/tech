+++
title = "Manjaro 系统配置"
description = " "
date = "2020-04-15T20:59:24+08:00"
categories = ["Linux"]
tags = ["Manjaro"]
slug = "configuration-after-installing-manjaro"
draft = true
+++

终于告别了辣鸡 Windows 系统了，从 Win 10 到 LTSC ，再到 Windows Server 2019 ，就没有一个不出 bug 的，还是早日投奔 Linux 的怀抱吧，至少出了 Bug 也可以自己折腾。目前更换了主力系统为 [Manjaro](https://manjaro.org/) ，这是一个基于 [Arch Linux](https://www.archlinux.org/) 的衍生发行版本，继承了 Arch 滚动更新和 AUR 软件包极为丰富的优点，稳定性也十分优秀，适合长期使用。Manjaro 官方镜像按照桌面环境（DE）的不同，分为 XFCE ，KDE ，GNOME 三个版本，个人选择的是 KDE 桌面环境，更为顺手一点。

下面记录一下配置过程，一来方便自己将来万一重装后的恢复，二来方便有需要的朋友们做个参考。

## 0.更换国内镜像源

在终端中执行下面这条命令：

```bash
sudo pacman-mirrors -i -c China -m rank
```

在弹出的对话框中选择一个最快的就可以了，多选几个也并不能加快速度。。

## 1.替换自带的 Vi 为 Vim

系统自带的 Vi 体验简直令人发指，没有方向键（方向键是 h/j/k/l 这四个键来控制的。。）， Backspace 也会变成莫名其妙的符号，直接卸载：

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

## 3.安装 yay 并配置包管理器走代理

3.1 安装 yay

yay 是一个 Go 语言编写的 AUR 助手，有时官方仓库内没有的软件，只能通过 AUR 来安装。详细介绍请移步文末 :)

```bash
sudo pacman -S yay
```

之后安装软件就不需要 `sudo pacman` 了，yay 兼容 pacman 的命令，下文如无特殊说明，均用 `yay` 来代替 `sudo pacman` 。

3.2 让包管理器走代理

首先参考 [简易配置终端代理](/posts/use-proxy-in-terminal) 设置好系统代理，然后

```bash
sudo vim /etc/sudoers
```

添加下面这行：

```conf
Defaults env_keep += "ftp_proxy http_proxy https_proxy"
```

保存即可，现在代理的环境变量会自动传递给 sudo 和 pacman 了，yay 会自动读取 `http_proxy` 和 `https_proxy` 的值，不需要特殊设置（其实 pacman 也能自动读取，但是 pacman 使用时要加上 sudo，而环境变量不会自动传递给 sudo，所以需要特殊设置一下；而且因为 yay 有些命令会调用 `sudo pacman`，所以哪怕只用 yay 也是需要设置这一步的 Orz）。

## 4.安装并配置 zsh

4.1 安装 zsh ：

```bash
yay -S zsh
```

4.2 安装 Oh My Zsh ：

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装完 Oh My Zsh 后会提示替换默认的 shell 为 zsh ，若没有提示的话按照下面的步骤手动替换：

1. 先查看当前系统内已安装的终端：
    ```bash
    chsh -l
    ```
2. 更改当前用户的默认终端：
    ```bash
    chsh -s /bin/zsh
    ```
3. 更改 root 用户的默认终端：
    ```bash
    sudo su
    chsh -s /bin/zsh
    su USERNAME
    ```

且重启系统后若 zsh 有更新，Oh My Zsh 会自动提示。

4.3 更换 Oh My Zsh 主题：

编辑 `~/.zshrc` 这个文件，找到 `ZSH_THEME` 字段，将后面的值修改为 `"ys"` （俺使用 [ys](https://github.com/ohmyzsh/ohmyzsh/wiki/themes#ys) 这个主题），然后 `source ~/.zshrc` 使修改生效。

4.4 安装 Oh My Zsh 插件：

```bash
# zsh-syntax-highlighting（代码高亮）
git clone https://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
# zsh-autosuggestions（自动建议）
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
# zsh-completions（自动补全）
git clone https://github.com/zsh-users/zsh-completions $ZSH_CUSTOM/plugins/zsh-completions
```

在 `~/.zshrc` 文件中找到 `plugins` 字段，默认值为 `(git)`（默认启用了 git 插件）：

```conf
...
plugins=(git)
...
```

括号内添加上刚才安装的三个新插件名字，中间用空格隔开，并在下面增加一行 `autoload -U compinit && compinit`（zsh-completions 插件需要），编辑好后应该类似下面这样：

```conf
...
plugins=(git sudo zsh-syntax-highlighting zsh-autosuggestions zsh-completions extract)
autoload -U compinit && compinit
...
```

其中 sudo 和 extract 是 ohmyzsh 自带的插件，前者的作用是在已经输入好的命令前自动加上 `sudo`，双击 ESC 键即可，非常方便，后者整合了常用的解压文件的命令别名，解压文件时只需 `extract <filename>` 就可以，不再需要记忆不同格式的解压命令。

最后使修改后的配置生效：

```bash
source ~/.zshrc
```

## 5.安装和配置字体

<!-- todo -->

## 6.安装和配置中文输入法

<!-- todo -->

## 7.修改家目录为英文

<!-- todo -->

## 8.个人常用软件列表

<!-- todo -->

## 9.备份当前系统配置，重装后一键恢复

<!-- todo -->

## 附：包管理说明

Arch 系的 Linux 发行版，软件包来源有两个，Community（Arch 官方仓库），AUR（Arch User Repository, Arch 用户仓库）。用户将软件放在 AUR ，Arch 官方则定期挑选 AUR 里的优秀程序到 community，实际表现为 Community 为 AUR 的子集，Community 有的应用 AUR 都有，但 AUR 内有而 Community 没有的那部分软件可能在系统上的运行表现不大稳定。

Manjaro 自带的桌面程序软件中心（pamac-manager）既可以安装 Community 程序也可以安装 AUR 程序，区别是 AUR 程序会显示【构建】而不是【安装】；但在终端中通过自带的 pacman 只能安装 Community 程序，想要安装 AUR 程序则需要安装额外的包管理器，之前很多教程内的 yaourt 已经停止维护了，个人使用 yay （详细介绍请移步官方 Wiki：[AUR helpers (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/AUR_helpers_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) ）。

下面是一些包管理器的常用参数：

1.pacman

```bash
sudo pacman -Syu  # 同步软件包数据库并更新系统
sudo pacman -S <packagename>  # 安装软件包
sudo pacman -R <packagename>  # 删除单个软件包，保留其全部已经安装的依赖关系
sudo pacman -Rs <packagename>  # 删除指定软件包，及其所有未被其他已安装软件包使用的依赖关系
sudo pacman -Rd <packagename>   # 删除软件包，不检查依赖关系
sudo pacman -Ss <keyword>>  # 查找含关键字的软件包
sudo pacman -Qs <keyword>   # 搜索已安装的包
```

2.yay

兼容 pacman 的命令行参数，这部分命令只需简单地将 `sudo pacman` 替换为 `yay` 即可，例如：

```bash
yay -Syu    # 等价于 sudo pacman -Syu
yay -S <packagename> # 等价于 sudo pacman -S <packagename>
```

其他参数类比即可。除此之外还可以：

```bash
yay <keyword>   # 搜索含关键字的软件包，输入序号安装对应的结果项，支持多选和反选
                # 若无需要安装的结果，输入 q 后回车退出
                # 搜索结果包含 Community 和 AUR 仓库内的包
```

---

*参考链接：*

1. [与Manjaro相见恨晚 - 山炮不二](https://xsinger.me/diy/857.html)

2. [Manjaro 个人新装配置 | 禾七博客](https://leay.net/2019/12/18/manjaro/)

3. [Manjaro-KDE配置全攻略 - 知乎](https://zhuanlan.zhihu.com/p/114296129)

4. [manjaro 安装配置总结 | Marsvet's Blog | Where there's a start, there's a finish.](https://www.marsvet.top/2020-08-04/Install-and-configure-manjaro/)

5. [pacman (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/Pacman_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))