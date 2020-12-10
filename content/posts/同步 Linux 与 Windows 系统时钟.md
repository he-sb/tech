+++
title = "同步 Linux 与 Windows 系统时钟"
description = " "
date = "2020-12-10T14:44:48+08:00"
categories = ["Linux"]
tags = ["时钟"]
slug = "synchronize-clock-between-linux-and-windows"
draft = false
+++

使用过 Linux 系统的同学应该都遇到过与 Windows 系统的系统时钟不同步的问题。如果 Linux 系统没有开启自动对时，你会发现 Linux 系统时钟总是比 Windows 快 8 小时。

原因是这样的，在关机后之所以时钟信息不会丢失，是因为时钟信息会保存在主板上，由主板上的电池供电。但这个时钟功能比较简单，只能保存年月日时分秒这些数值信息，不能保存时间标准、时区、夏令时这类复杂信息。时间表示有两个标准：

*  UTC：Coordinated Universal Time，协调世界时，是一种与时区无关的全球时间标准；

* localtime：本地时间，依赖于当前时区。

因为对主板中保存的硬件时间理解不同，Linux 系统默认使用 UTC，从主板中读取硬件时钟（也称为实时时钟，Real Time Clock，简称 RTC）后，加上系统中存储的时区信息后成为系统时钟（System Clock）供系统内核和程序使用；而 Windows 系统默认使用的是 localtime，会直接使用从主板读取到的 RTC。

解决办法也简单，有两个思路：

1. 让 Windows 使用 UTC（推荐）；

2. 让 Linux 使用 RTC。

## 让 Windows 使用 UTC

虽然 Windows 默认使用 localtime，但 Windows 是可以处理 UTC 的，只不过需要修改注册表，使用管理员权限运行 CMD 或 Powershell，执行下面这条命令：

```shell
reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_QWORD /f
```

上面是 64 位系统的命令，如果是 32 位系统，把里面的 `REG_QWORD` 改成 `REG_DWORD` 就可以了，执行完毕后重启电脑生效。

如果你之前使用了夏令时和自动对时，也是可以照常使用的，不需要关闭。网上很多教程十分“贴心”地提醒修改后不可以开启时间同步，以免设置失效，俺亲测是不影响的，依然可以正常使用时间同步。

## 让 Linux 使用 RTC

以下以俺使用的 Manjaro KDE x64 为例，其他发行版设置方法大同小异，自己摸索下就好。

首先检查当前时区是否正确：

```bash
timedatectl status | grep "Time zone"
```

如果显示的时区不是 Asian/Shanghai，那么执行这条命令切换时区：

```bash
timedatectl set-timezone Asia/Shanghai
```

然后执行：

```bash
timedatectl set-local-rtc true
```

现在 Linux 就会像 Windows 一样将 RTC 当作 localtime 来对待了，缺点是无法使用夏令时，以及小概率会在启动时遇到某些不可预料的错误（详情请参阅 [这里](https://wiki.archlinux.org/index.php/System_time_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%97%B6%E9%97%B4%E6%A0%87%E5%87%86)）。

如果需要改回 UTC，将上一条命令结尾的 `true` 改为 `false` 就可以了。

---

*参考链接：*

1. [同步 Linux 双系统的时间 | Mogeko`s Blog](https://mogeko.me/2019/062/)

2. [System time (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/System_time_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))