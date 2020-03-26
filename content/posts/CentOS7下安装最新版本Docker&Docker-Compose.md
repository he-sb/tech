+++
title = "CentOS 7 下安装最新版本 Docker & Docker Compose"
description = " "
date = "2020-03-01T19:12:43+08:00"
categories = ["Linux"]
tags = ["centos","docker"]
slug = "install-latest-version-of-docker&compose-in-centos7"
comments = true
draft = false
+++

**系统环境：** CentOS 7 x64

## 安装 `Docker` 

### 卸载旧版本

停止 `Docker` 服务

```shell
systemctl stop docker
```

查看当前 `Docker` 版本

```shell
rpm -qa | grep docker
```

卸载旧的软件包

```shell
yum erase docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine \
docker-ce
```

删除相关配置文件

```shell
find /etc/systemd -name '*docker*' -exec rm -f {} \;
find /lib/systemd -name '*docker*' -exec rm -f {} \;
rm -rf /var/lib/docker   # 删除已有的镜像和容器,非必要
rm -rf /var/run/docker
```

### 安装新版本

安装依赖

```shell
yum install -y yum-utils  device-mapper-persistent-data lvm2
```

添加 `yum` 源

```shell
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

安装最新版本

```shell
yum install -y docker-ce
```

启动 `Docker` 并添加开机自启

```shell
systemctl enable docker
systemctl start docker
```

查看当前运行的 `Docker` 版本

```shell
docker version
```

## 安装 `Docker-Compose` 

先访问 [https://github.com/docker/compose/releases/latest](https://github.com/docker/compose/releases/latest) 查看最新版本 `Docker-Compose` 的版本号，如 `1.25.4` ，然后执行以下命令来安装

```shell
# 下载 1.25.4 版 Docker-Compose 到 /usr/bin 目录下
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

# 赋予可执行权限
chmod +x /usr/bin/docker-compose
```