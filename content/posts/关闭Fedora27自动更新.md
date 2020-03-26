+++
title = "关闭 Fedora 27 自动更新"
description = " "
date = "2018-08-30T12:37:35+08:00"
categories = ["linux"]
tags = ["fedora"]
slug = "disable-fedora-27-auto-update"
comments = true
draft = false
+++
**系统环境：** Fedora 27 Desktop Workstation x64

先安装 `dconf-editor`

```bash
sudo dnf install dconf-editor
```

安装完成后打开

```bash
dconf-editor
```

关闭 `/org/gnome/software/allow-updates` 开关即可。