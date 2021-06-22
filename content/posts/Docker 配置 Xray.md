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

## 编辑配置文件

### `Caddyfile`

```shell
vi /root/Caddyfile
```

`Caddy` 的配置文件，包含两部分，走 `/` 的流量反代正常网站，走 `/path` 的流量反代至 `XRay` 监听的端口

```
hd.fkgfw.men {
gzip
tls 0he.sb0@gmail.com
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
vi /root/config/json
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
    - ./xray:/etc/xray
    expose:
    - "443"

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