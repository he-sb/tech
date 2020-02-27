+++
title = "IPv6相关配置"
description = " "
date = "2018-12-09T12:37:35+08:00"
categories = ["windows"]
tags = ["ipv6","dns","hosts"]
slug = "ipv6-related-system-configuration"
comments = true
draft = false
+++
**系统环境：** Windows 10 专业版 1803 x64

## IPv6公共dns

Google

```
2001:4860:4860::8888
2001:4860:4860::8844
```

Cloudflare

```
2606:4700:4700::1111
2606:4700:4700::1001
```

IBM Quad9

```
2620:fe::fe
2620:fe::9
```

OpenDNS

```
2620:0:ccc::2
2620:0:ccd::2
```

中华电信/HiNet

```
2001:b000:168::1
2001:b000:168::2
```

Yandex

```
2a02:6b8:0:1::feed:0ff
2a02:6b8::feed:0ff
```

## 替换hosts文件

路径：`C:\Windows\System32\drivers\etc\hosts`

替换为：[https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts](https://raw.githubusercontent.com/lennylxx/ipv6-hosts/master/hosts)

## 修改系统优先使用IPv6

控制面板->网络与共享中心->更改适配器设置->右击需要修改的网络连接，选择“属性”->分别双击“Internet协议版本4（TCP/IPv4）”与“Internet协议版本6（TCP/IPv4）”->高级->取消勾选“自动跃点”，并将IPv6的该值设置的比IPv4小即可，如IPv4中该值设为20，IPv6中为10。