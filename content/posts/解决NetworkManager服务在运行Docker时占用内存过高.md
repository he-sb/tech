+++
title = "解决NetworkManager服务在运行Docker时占用内存过高"
description = " "
date = "2020-02-29T23:39:48+08:00"
categories = ["linux"]
tags = ["networkmanager","docker","内存"]
slug = "fix-high-memory-usage-when-running-docker"
comments = true
draft = false
+++

**系统环境：** CentOS 7 x64

服务器在运行Docker时用 `top` 查看发现 `NetworkManager` 这个服务占用了大量内存，经过一番搜索知道了问题原因，是因为 `NetworkManager` 消耗的内存量随着容器启动/停止的每次迭代而增加，即使在所有容器已被停止和删除之后也不会减少，解决办法有二，下面分别说明。

## 重启 `NetworkManager` 服务

```shell
systemctl restart NetworkManager
```

此方法只适合临时解决问题，不过暂时也够用了，若想避免问题再次发生，请看下文分解~

## 用 `network` 代替 `NetworkManager` 管理网络

```shell
systemctl disable NetworkManager
/sbin/chkconfig network on
kill 'pgrep -o dhclient'
systemctl stop NetworkManager
systemctl start network
```

## 附：`Ubuntu 16.04` 关闭 `NetworkManager` 方法

```shell
systemctl enable networking
systemctl disable NetworkManager
kill 'pgrep -o dhclient'
systemctl stop NetworkManager
systemctl start networking
```

---

*参考链接：*

1. [NetworkManager在运行docker容器时占用大量内存怎么办？_弹性云服务器 ECS_故障排除_Linux操作系统_华为云](https://support.huaweicloud.com/trouble-ecs/ecs_trouble_0335.html)