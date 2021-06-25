+++
title = "Docker 一条龙配置 vmess + WS + TLS + CDN (optional)"
description = " "
date = "2020-02-28T22:00:00+08:00"
categories = ["奇技淫巧"]
tags = ["v2ray","cdn","ws","tls","cf","docker"]
slug = "vmess-ws-tls-cdn-docker"
comments = true
draft = true
+++

**系统环境：** CentOS 7 x64

## 前言

本方案数据流向如下

```
client 原始数据 -> 封装为 WebSocket -> TLS加密 -> (CDN) -> server 解密后反代至 V2Ray 端口 -> 该干嘛干嘛
```

全过程模拟正常 HTTPS 流量，最大程度避免被墙。

## 准备工作

### 系统时间校准

```shell
yum install -y ntp    # 安装 NTP 服务
systemctl enable ntpd
systemctl start ntpd
timedatectl set-timezone Asia/Shanghai    # 修改时区，非必需
timedatectl set-ntp true    # 启用时间自动同步
ntpq -p    # 同步时间
```

### 安装 BBR 加速

首先下载并运行四合一脚本（[Github地址](https://github.com/chiakge/Linux-NetSpeed/)）

```shell
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
chmod +x tcp.sh
./tcp.sh
```

输入 `2` 安装 `BBRplus版内核` ，安装完后机器会自动重启，重启后再次运行脚本

```shell
./tcp.sh
```

输入 `7` 来 `启用 BBRplus 版加速` ，显示 “开启成功” 即可，不放心的话可再次运行脚本确认开启状态。

### 系统组件更新

```shell
yum update -y
```

### 安装 Docker 和 Docker-Compose

安装依赖

```shell
yum install -y yum-utils  device-mapper-persistent-data lvm2
```

添加 yum 源

```shell
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
```

安装最新版本 Docker 并添加开机自启

```shell
yum install -y docker-ce
systemctl enable docker
systemctl start docker
```

安装 Docker-Compose

```shell
curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

### 域名和 CDN 部分

TLS 要求域名作为前提。

* 域名在 Freenom 申请免费的即可
* CDN 可以使用 CloudFlare 免费套餐

登陆 CloudFlare ，点击 `add site` 添加域名，根据提示修改 Freenom 的域名服务器为 CloudFlare 提供的，返回 CF 这边等待 DNS 解析生效，生效后添加一条 A 记录

```
@ 服务器IP
```

此时点击 DNS 记录旁边的云朵图标使其保持灰色，即不通过 CDN ，稍后配置成功后再开启，或者暂不开启，登服务器 IP 被墙了再开启 CDN ~~强行~~复活。

## 编辑配置文件

### `Caddyfile`

```shell
vi /root/Caddyfile
```

`Caddy` 的配置文件，包含两部分，走 `/` 的流量反代正常网站，走 `/path` 的流量反代至 `V2Ray` 监听的端口

```
example.com {
gzip
tls example@example.com
proxy /path v2ray:44333 {
websocket
header_upstream -Origin
}
proxy / https://v2ex.com
# write log to stdout for docker
log stdout
errors stdout
}
```

### `config.json`

```shell
vi /root/config.json
```

V2Ray 的配置文件，注意把其中 `id` 对应的值替换为自己的

```json
{
    "inbounds": [
    {
        "listen": "0.0.0.0",
        "streamSettings": {
        "network": "ws",
        "wsSettings": {
            "path": "/path"
        },
        "security": "auto"
        },
        "settings": {
        "clients": [
            {
            "id": "12345678-90ab-cdef-ghij-klmnopq12345",
            "alterId": 4
            }
        ]
        },
        "protocol": "vmess",
        "port": 44333
    }
    ],
    "outbounds": [
    {
        "tag": "direct",
        "settings": {},
        "protocol": "freedom"
    }
    ]
}
```

### `docker-compose.yml`

```shell
vi /root/docker-compose.yml
```

`Docker-Compose` 的配置文件，将上面用到的 `Caddy` 和 `V2Ray` 相应的容器运行起来

```yml
version: '3'

services:
  v2ray:
    container_name: v2ray
    image: teddysun/v2ray
    restart: always
    volumes:
    - ./config.json:/etc/v2ray/config.json
    expose:
    - "44333"

  caddy:
    container_name: caddy
    image: abiosoft/caddy
    restart: always
    volumes:
    - ./Caddyfile:/etc/Caddyfile:ro
    - ./caddyCertificates:/root/.caddy
    environment:
    - ACME_AGREE=true
    ports:
    - "80:80"
    - "443:443"
```

## 启动 Docker

```shell
docker-compose up -d
```

一切正常的话此时 V2Ray 应该可以正常使用了。

## 完善 CDN（可选）

若上一步 V2Ray 可以正常工作，此时就可以回到 CF 开启 CDN 了。

1. `DNS` 页签：将云朵图标点亮
2. `SSL` 页签：模式选择 `Full`

## 客户端配置示例

注意以下并非配置文件，不可直接导入

```
地址(address)：example.com
端口(port)：443
用户ID(id)：12345678-90ab-cdef-ghij-klmnopq12345
额外ID(alterId)：4
加密方式(security)：auto
传输协议(protocol)：ws
路径(path)：/path
底层传输安全：TLS
```

## 故障排除

### 执行 `docker-compose` 提示 `permission deny`

关闭 Selinux 即可：

```shell
#查看SELinux状态（如果 SELinux status 参数为 enabled 即为开启状态）
/usr/sbin/sestatus -v

#临时关闭
setenforce 0

#修改配置文件重启机器禁用（将 SELINUX=enforcing 改为 SELINUX=disabled ）
vim /etc/selinux/config
```

### 防火墙相关

* 查看防火墙状态：`firewall-cmd --state`
* 开启防火墙：`systemctl start firewalld`
* 查看已打开的端口：`firewall-cmd --list-ports`
* 开启 `80` 和 `443` 端口，以及 `SSH` 端口

```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public -add-port=443/tcp --permanent
firewall-cmd --zone=public -add-port=22/tcp --permanent
```

* 重启防火墙：`firewall-cmd --reload`
* 再次确认已开启端口：`firewall-cmd --list-ports`

### Docker 相关

* 查看容器基本信息： `docker ps -a`
* 查看反代是否生效，即访问域名，看能否访问正常网站。如果不生效，执行 `docker logs docker-id` 查看原因
* 反代生效基本就没啥问题了，测试 V2Ray 能否正常工作
* 如果 Docker 有问题，先 stop：`docker stop docker-id` ，再删除容器：`docker rm docker-id` ，最后再次执行 `docker-compose up -d` ，看是否正常

---

*参考链接：*

1. [v2ray(vmess+ss)+ws+tls+cdn](https://www.notion.so/v2ray-vmess-ss-ws-tls-cdn-acc54f01f4c747d694cca1a9172b09f6)

2. [teddysun/v2ray - Docker Hub](https://hub.docker.com/r/teddysun/v2ray)