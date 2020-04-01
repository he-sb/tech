+++
title = "修改 SSH 默认端口"
description = " "
date = "2018-12-05T12:37:35+08:00"
categories = ["Linux"]
tags = ["ssh","端口"]
slug = "change-default-ssh-ports"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

远程连接VPS的默认端口一般是22，不修改的话会有安全隐患，比如有人恶意扫描端口之类的，非常消耗服务器资源。可以手动修改为其他端口号，步骤如下

```bash
vi /etc/ssh/sshd_config
```

找到 `Port 22` 这一行，把最前面注释用的 `#` 号去掉，再在下面添加一行（**代表想要修改的端口号）

```bash
Port **
```

保存退出，再重启一下sshd服务

```bash
systemctl restart sshd
```

正常情况下应该可以用新端口号来SSH连接VPS了。

此时新端口号和默认的22都可以SSH连接VPS，测试新端口没问题之后再重复以上步骤，删掉 `Port 22` 这一行即可。