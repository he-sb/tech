+++
title = "Docker 配置 Xray"
description = " "
date = "2021-06-21T18:06:00+08:00"
categories = ["奇技淫巧"]
tags = ["v2ray","xray","tls","docker"]
slug = "xray-docker"
comments = true
draft = true
+++

**系统环境：** CentOS 7 x64

## 前言

## 准备工作

相比 [Docker 配置 vmwss + WS + TLS + CDN (optional)](Docker%20一条龙配置%20vmess%20+%20WS%20+%20TLS%20+%20CDN%20(optional).md)，本文新增了证书申请步骤，不再使用 Caddy 自动申请。

首先安装 acme.sh：

```shell
wget -O -  https://get.acme.sh | sh
```

使 acme.sh 命令生效：

```shell
. .bashrc
```

开启 acme.sh 自动更新：

```shell
acme.sh --upgrade --auto-upgrade
```

acme.sh --register-account -m 0he.sb0@gmail.com
acme.sh --issue -d hd.fkgfw.men --standalone --keylength ec-256 --force
acme.sh --install-cert -d hd.fkgfw.men --ecc --fullchain-file /root/cert/hd.fkgfw.men.crt --key-file /root/hd.fkgfw.men.key

## 编辑配置文件

### `Caddyfile`

```shell
vi /root/Caddyfile
```

`Caddy` 的配置文件，回落至此的流量默认反代正常网站

```
hd.fkgfw.men {
gzip
tls 0he.sb0@gmail.com
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

XRay 的配置文件，注意把其中 `id` 对应的值替换为自己的

```json
{
    "log": {
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "port": 443,
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "12345678-90ab-cdef-ghij-klmnopq12345",
                        "flow": "xtls-rprx-direct",
                        "level": 0
                    }
                ],
                "decryption": "none",
                "fallbacks": [
                    {
                        "dest": 80,
                        "xver": 1
                    },
                    {
                        "path": "/vlsws", // 必须换成自定义的 PATH
                        "dest": 1234,
                        "xver": 1
                    },
                    {
                        "path": "/vmsws", // 必须换成自定义的 PATH
                        "dest": 3456,
                        "xver": 1
                    }
                ]
            },
            "streamSettings": {
                "network": "tcp",
                "security": "xtls",
                "xtlsSettings": {
                    "alpn": [
                        "http/1.1"
                    ],
                    "certificates": [
                        {
                            "certificateFile": "/path/to/fullchain.crt", // 换成你的证书，绝对路径
                            "keyFile": "/path/to/private.key" // 换成你的私钥，绝对路径
                        }
                    ]
                }
            }
        },
        {
            "port": 1234,
            "listen": "127.0.0.1",
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "12345678-90ab-cdef-ghij-klmnopq12345",
                        "level": 0
                    }
                ],
                "decryption": "none"
            },
            "streamSettings": {
                "network": "ws",
                "security": "none",
                "wsSettings": {
                    "path": "/vlsws"
                }
            }
        },
        {
            "port": 3456,
            "listen": "127.0.0.1",
            "protocol": "vmess",
            "settings": {
                "clients": [
                    {
                        "id": "12345678-90ab-cdef-ghij-klmnopq12345",
                        "level": 0
                    }
                ]
            },
            "streamSettings": {
                "network": "ws",
                "security": "none",
                "wsSettings": {
                    "path": "/vmsws"
                }
            }
        }
    ],
    "outbounds": [
        {
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
  xray:
    container_name: xray
    image: teddysun/xray
    restart: always
    volumes:
    - ./config.json:/etc/xray/config.json
    network_mode: "host"

  caddy:
    container_name: caddy
    image: teddysun/caddy
    restart: always
    volumes:
    - /root/Caddyfile:/etc/caddy/Caddyfile:ro
    network_mode: "host"
```