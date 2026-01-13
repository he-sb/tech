+++
title = "宿主机安装 Node Exporter"
description = " "
date = "2023-10-19T00:35:23+08:00"
toc = true
categories = ["Linux"]
tags = ["homelab","监控","prometheus","node-exporter"]
slug = "install-node-exporter-on-linux-host"
draft = false
+++

最近遇到了一些场景，需要在宿主机安装 node-exporter 来监控指标：

- 某些指标只能在宿主机采集，容器内采集不方便 / 无法采集
- 某些机器有特定用途，不需要 / 无法安装 Docker

下面记录一下安装过程，系统环境基于 Debian 11 / 12，安装方式来自 node-exporter 的 [官方文档](https://github.com/prometheus/node_exporter/tree/master/examples/systemd)（理论上 CentOS 以及 Ubuntu 系统也是一样的）。

首先从 [release](https://github.com/prometheus/node_exporter/releases) 下载并解压官方编译好的 node-exporter 二进制程序：

```shell
$ curl -OL https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
$ tar -xvf node_exporter-1.6.0.linux-amd64.tar.gz
```

然后创建一个 `node_exporter` 用户，用来执行 node-exporter 程序：

```shell
$ sudo useradd --no-create-home --shell /sbin/nologin node_exporter
```

将解压后的 node-exporter 程序移动到系统程序目录下，并将所属的用户和组信息修改为刚创建的 `node-exporter` ：

```shell
$ sudo mv node_exporter-1.6.0.linux-amd64/node_exporter /usr/local/bin/
$ sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

创建 systemd 配置文件：

```shell
$ sudo bash -c 'cat <<EOF > /etc/systemd/system/node_exporter.service
[Unit]
Description=Node Exporter
Requires=node_exporter.socket

[Service]
User=node_exporter
Group=node_exporter
EnvironmentFile=/etc/sysconfig/node_exporter
ExecStart=/usr/local/bin/node_exporter  --web.systemd-socket $OPTIONS

[Install]
WantedBy=multi-user.target
EOF'
```

创建 systemd 使用的 socket 文件，监听 `9100` 端口：

```shell
$ sudo bash -c 'cat <<EOF > /etc/systemd/system/node_exporter.socket
[Unit]
Description=Node Exporter

[Socket]
ListenStream=9100

[Install]
WantedBy=sockets.target
EOF'
```

同样，这两个 systemd 配置文件的所有权也要变更为 `node-exporter` ：

```shell
$ sudo chown -R node_exporter:node_exporter /etc/systemd/system/node_exporter.service
$ sudo chown -R node_exporter:node_exporter /etc/systemd/system/node_exporter.socket
```

再创建一个配置文件，用于定义 node-exporter 除了监听端口之外的启动参数：

```shell
$ sudo bash -c 'cat <<EOF > /etc/sysconfig/node_exporter
OPTIONS="--collector.textfile.directory /var/lib/node_exporter/textfile_collector"
EOF'
```

比如这里就开启了 text-collector 功能，这个功能是让 node-exporter 从指定位置的文件中采集指标信息，然后附加到自己采集到的其他指标中输出。Prometheus 社区贡献了 [很多脚本](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts) ，可以将许多 node-exporter 无法采集的指标（比如硬盘的 S.M.A.R.T 信息），转化为 Prometheus 支持的格式，然后写入文件中，这些指标就可以通过 node-exporter 的 text-collector 功能被采集，然后暴露给 Prometheus 拉取，进而可以让这些原本不支持 Prometheus 采集指标的程序，可以以一种相对优雅的方式，集成到围绕 Prometheus 构建的监控报警体系中去。

如果你不需要这个功能，那么 `/etc/sysconfig/node_exporter` 这个配置文件和下面将要创建的，用于存放供 text-collector 收集的指标文件的 `/var/lib/node_exporter/textfile_collector` 目录，都可以先不用配置，以后有需求时再配置也可以。

创建 text-collector 监听的指标文件存放路径，所有权同样改成 `node-exporter` ：

```shell
$ sudo mkdir -p /var/lib/node_exporter/textfile_collector
$ sudo chown -R node_exporter:node_exporter /var/lib/node_exporter
```

最后，让 systemd 重载配置，然后启动 node-exporter，同时允许开机自启：

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now node_exporter
```

查看服务运行状态：

```shell
$ sudo systemctl status node_exporter
```

最后记得在防火墙放行 node-exporter 配置的监听端口（俺这里是 `9100`）：

```shell
$ sudo ufw allow 9100/tcp comment "node-exporter"
```

---

*参考链接：*

1. [node_exporter/examples/systemd at master · prometheus/node_exporter](https://github.com/prometheus/node_exporter/tree/master/examples/systemd)
