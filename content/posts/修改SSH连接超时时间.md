+++
title = "修改 SSH 连接超时时间"
description = " "
date = "2018-12-06T16:37:35+08:00"
categories = ["linux"]
tags = ["ssh"]
slug = "modify-ssh-timeout"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

SSH连接服务器时，超过一定时间没有输入后就会断开连接，有时会有些不便。下面通过修改服务器参数的方法来解决

```bash
vi /etc/ssh/sshd_config
```

找到以下两项，先删掉行首的注释符 `#` ，再根据自己需求来改这两项的数置：

```bash
ClientAliveInterval 600    #代表超时阈值，单位秒，比如本行代表600秒无输入算超时一次
ClientAliveCountMax 6    #允许的超时次数，本行表示超时6次后断开连接
```

修改好后重启sshd服务即可

```bash
systemctl restart sshd
```