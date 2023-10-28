+++
title = "修改 SSH 连接超时时间"
description = " "
date = "2018-12-06T16:37:35+08:00"
categories = ["Linux"]
tags = ["ssh"]
slug = "modify-ssh-timeout"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

SSH连接服务器时，超过一定时间没有输入后就会断开连接，有时会有些不便。下面通过修改服务器参数的方法来解决

```shell
vi /etc/ssh/sshd_config
```

找到以下两项，先删掉行首的注释符 `#` ，再根据自己需求来改这两项的数置：

```shell
# 开启 TCP 长连接
TCPKeepAlive yes
# 代表超时阈值，单位秒
# 比如这里配置为客户端 120 秒（2 分钟）无响应算超时 1 次
ClientAliveInterval 120
# 允许的超时次数
# 这里配置为超时 360 次（2 x 360 = 720 分钟，即 12 小时）后断开连接
ClientAliveCountMax 360
```

修改好后重启sshd服务即可

```shell
systemctl restart sshd
```

---

*参考链接：*

1. [How to Keep SSH Session Alive](https://linuxiac.com/how-to-keep-ssh-session-alive/)