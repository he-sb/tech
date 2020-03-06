+++
title = "Docker安装aria2"
description = " "
date = "2020-03-02T19:42:21+08:00"
categories = ["linux"]
tags = ["docker","aria2"]
slug = "install-aria2-through-docker"
comments = true
draft = false
+++

**系统环境：** CentOS 7 x64

使用 `Docker` 容器来安装 `aria2` ，相比使用一键脚本或[手动安装](/posts/setup-offline-download-service-aria2-ariang-filebrowser-on-centos7/#安装aria2)来说，好处很多

* 下载错误或取消下载后，自动删除未完成的文件，防止磁盘空间占用
* 自动获取 BT tracker
* 配置文件持久化，便于复用和迁移

下面记录一下配置过程。

## 快速开始

使用以下命令拉取并启动容器，注意将下面的 `SECRET` 替换为自定义的 RPC 密码

```shell
docker run -d \
    --name aria2-pro \
    --restart unless-stopped \
    --log-opt max-size=1m \
    -e PUID=0 \
    -e PGID=0 \
    -e RPC_SECRET="SECRET" \
    -p 6800:6800 \
    -p 6888:6888 \
    -p 6888:6888/udp \
    -v /root/aria2-config:/config \
    -v /root/downloads:/downloads \
    -e RCLONE=enable \
    p3terx/aria2-pro
```

配置防火墙开放 `6800` 、`6888`端口

```shell
firewall-cmd --zone=public --add-port=6800/tcp --permanent
firewall-cmd --zone=public --add-port=6888/tcp --permanent
firewall-cmd --zone=public --add-port=6888/udp --permanent
firewall-cmd --reload
```

此时应该可以通过 `Aria2NG` 来连接使用了，配置文件路径 `/root/aria2-config` ，下载文件路径 `/root/downloads` 。

## 参数说明

* `--aria2-pro` ：本地容器名称，可自定义
* `--restart` ：容器重启策略，详见 [Docker 官方文档](https://docs.docker.com/engine/reference/commandline/run/#restart-policies---restart)
* `--log-pot` ：日志文件大小限制，一般不用改
* `-e PUID=0` , `-e PGID=0` ：容器内账户 UID 与 GID ，详见 [Understanding PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid)
* `-p 6800:6800` ：`aria2` 的 RPC 端口映射，冒号左边为宿主机端口，可自定义
* `-p 6888:6888` , `-p 6888:6888/udp` ：前者为 BT 监听端口 (TCP) ，后者为 DHT 监听端口 (UDP) ，冒号左边的宿主机端口可自定义
* `-v /root/aria2-config:/config` ：配置文件目录映射，使配置文件持久化，冒号左边为宿主机路径，可自定义，路径内不要有中文
* `-v /root/downloads:/downloads` ： 下载目录映射，冒号左边为宿主机路径，可自定义，路径内不要有中文
* `-e RCLONE=enable` ：开启下载完成后自动上传网盘功能，需将宿主机目录 `~/.config/rclone/` 下的 `rclone.conf`（rclone配置文件）复制至 aria2-pro 配置文件目录，然后修改 aria2-pro 配置文件目录下 `autoupload.sh` 文件内的【网盘名称】和【目标路径】这两个选项即可

---

*参考链接：*

1. [Aria2 Pro - 更好用的 Aria2 Docker 容器镜像 - P3TERX ZONE](https://p3terx.com/archives/docker-aria2-pro.html)