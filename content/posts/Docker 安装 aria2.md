+++
title = "Docker 安装 aria2"
description = " "
date = "2020-03-02T19:42:21+08:00"
categories = ["奇技淫巧"]
tags = ["docker","aria2"]
slug = "install-aria2-through-docker"
comments = true
draft = false
+++

**系统环境：** CentOS 7 x64

使用 `Docker` 容器来安装 `aria2` ，相比使用一键脚本或 [手动安装](/posts/setup-offline-download-service-aria2-ariang-filebrowser-on-centos7/#安装aria2) 来说，好处很多

* 下载错误或取消下载后，自动删除未完成的文件，防止磁盘空间占用
* 自动获取 BT tracker
* 配置文件持久化，便于复用和迁移

下面记录一下配置过程。

## 快速开始

使用以下命令拉取并启动容器，注意将下面的 `<SECRET>` 替换为自定义的 RPC 密码

```shell
docker run -d \
    --name aria2-pro \
    --restart unless-stopped \
    --log-opt max-size=1m \
    --network host \
    -e PUID=$UID \
    -e PGID=$GID \
    -e RPC_SECRET="<SECRET>" \
    -e RPC_PORT=6800 \
    -e LISTEN_PORT=6888 \
    -v ~/aria2-config:/config \
    -v ~/downloads:/downloads \
    -e SPECIAL_MODE=rclone \
    p3terx/aria2-pro
```

如果觉得复制一长串命令不太方便，可以把上面的内容保存为一个 shell 脚本：

```shell
vi aria2-pro-run.sh
```

将上面的内容写入，保存，然后给脚本增加执行权限：

```shell
chmod +x aria2-pro-run.sh
```

然后只要执行这个脚本就可以了，脚本文件也比较方便复用：

```shell
./aria2-pro-run.sh
```

配置防火墙开放 `6800` 、`6888`端口：

```shell
firewall-cmd --zone=public --add-port=6800/tcp --permanent
firewall-cmd --zone=public --add-port=6888/tcp --permanent
firewall-cmd --zone=public --add-port=6888/udp --permanent
firewall-cmd --reload
```

此时应该可以通过 `Aria2NG` 来连接使用了，配置文件路径 `/root/aria2-config` ，下载文件路径 `/root/downloads` 。

## 参数说明

* `--aria2-pro` ：本地容器名称，可自定义；
* `--restart` ：容器重启策略，详见 [Docker 官方文档](https://docs.docker.com/engine/reference/commandline/run/#restart-policies---restart) ；
* `--log-pot` ：日志文件大小限制，一般不用改；
* `-e PUID=$UID` , `-e PGID=$GID` ：容器内账户 UID 与 GID 继承自当前用户，详见 [Understanding PUID and PGID](https://docs.linuxserver.io/general/understanding-puid-and-pgid) ；
* `-v ~/aria2-config:/config` ：配置文件目录映射，使配置文件持久化，冒号左边为宿主机路径，可自定义，路径内不要有中文；
* `-v ~/downloads:/downloads` ： 下载目录映射，冒号左边为宿主机路径，可自定义，路径内不要有中文；
* `-e SPECIAL_MODE=rclone` ：开启下载完成后自动上传网盘功能，需将 rclone 配置文件（默认路径 `~/.config/rclone/rclone.conf`）复制至 aria2-pro 配置文件目录，然后修改 aria2-pro 配置文件目录下 `script.conf` 文件内的 `drive-name` 和 `drive-dir` 这两个选项即可。

## 注意事项

1. 远程调用 aria2 时下载目录务必设置为 `/downloads` ，即容器内默认路径，实际下载的文件会出现在上一步中【自定义映射的宿主机路径】下，否则会将文件下载至容器内部，需要手动从容器中 copy 出来，比较麻烦；

2. 想到了再补充。

---

*参考链接：*

1. [Aria2 Pro - 更好用的 Aria2 Docker 容器镜像 - P3TERX ZONE](https://p3terx.com/archives/docker-aria2-pro.html)