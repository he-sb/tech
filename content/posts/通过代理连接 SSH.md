+++
title = "通过代理连接 SSH"
description = " "
date = "2020-05-01T00:38:33+08:00"
categories = ["奇技淫巧"]
tags = ["ssh","加速"]
slug = "ssh-connecting-through-proxy"
draft = false
+++

有时虽然 VPS 没被墙，但通过 SSH 连接进行操作时却十分卡顿，或者 VPS 不幸被墙，只能通过代理来访问。虽然 Putty 等 SSH 客户端可以很方便地配置代理，但如果你像俺一样使用操作系统的终端来通过 OpenSSH 连接，那么需要一番设置才能顺利使 SSH 经过代理。下面分别记录一下 Windows 和 Linux 下的配置过程。

## Windows 系统

使用 connect ，Windows 默认没有自带这个程序，需要去 [https://bitbucket.org/gotoh/connect/downloads/](https://bitbucket.org/gotoh/connect/downloads/) 这里下载 exe 文件并放至本机目录（比如俺这里放在了 `C:\Windows\` 这个目录）。

然后编辑 `C:\Users\Administrator\.ssh\config` 这个文件，其中 `Administrator` 部分改为你电脑当前的用户名，如果这个文件不存在，那就新建一个，在其中写入：

```conf
Host *
    ProxyCommand C:\Windows\connect.exe -S 127.0.0.1:1080 %h %p
```

参数说明：

* 第一行 `Host *` 代表下面配置的代理的作用范围是所有 SSH 连接，如果只需要对个别连接配置代理，那么把型号换成对应连接的别名或者 IP 地址；

* 下面的 `C:\Windows\connect.exe` 是 connect 程序的绝对路径；

* `-S` 是指定使用 socks 5 协议，也可以使用 `-H` 来使用 HTTP 协议来代理，不过后面的 `127.0.0.1:1080` 也要改为你的 HTTP 代理的地址和端口；

* `%h` 和 `%p` 用于替换真正要连接的服务器主机名（host）和端口（port）。

如果只是临时需要走一下代理，那么可以不修改配置文件，而是在原本的 SSH 命令后面加上参数 `-o "ProxyCommand path-to-connect.exe -S 127.0.0.1:1080 %h %p` ，例如上面的例子，可以在连接的时候使用类似下面的命令，是一样的效果：

```bash
ssh user@server -o "ProxyCommand C:\Windows\connect.exe -S 127.0.0.1:1080 %h %p"
```

## Linux 系统

使用 nc ，即 netcat ，这是一个强大的网络工具，不过许多发行版默认没有安装，需要手动安装，CentOS 可以使用 `yum install nc` 来安装，Manjaro 使用 `sudo pacman -S netcat` 来安装。

用法和上文中 Windows 下使用 connect 类似，只是 nc 的参数和 connect 略有不同，下面举个栗子：

```bash
ssh user@server -o "ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p"
```

部分参数说明：

* `-X 5` 指定代理协议为 socks 5 ，如果是 socks 4 则写为 `-X 4` ，如果是 HTTPS 代理则为 `-X connect` ；

    如果不写 `-X` 参数，默认使用 socks 5 ，即相当于 `-X 5` ；

* `-x` 指定代理的主机地址及端口；

其他配置比如修改 SSH 配置文件，除了路径不同（Linux 一般在 `~/.ssh/config`），内容的格式是一样的，参照上文即可。



*参考链接：*

1. [通过 Socks5 代理进行 SSH 连接 - PRIN BLOG](https://printempw.github.io/ssh-over-proxy/)

2. [Windows 下配置 SSH 走 Shadowsocks 代理 – 兴趣使然的博客](https://one-piece.blue/pc-software/ssh-via-shadowsocks/)

3. [ssh怎样通过代理登录远程主机 | Bruce's Blog](https://www.xiebruce.top/650.html)