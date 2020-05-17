+++
title = "Docker 容器和镜像的管理"
description = " "
date = "2020-04-16T23:28:45+08:00"
categories = ["笔记"]
tags = ["docker"]
slug = "manage-containers-and-images-in-docker"
draft = false
+++

## 删除容器实例

```bash
docker rm <容器 ID 或容器名称>
```

* 其中 <容器 ID 或容器名称> 可以通过命令 `docker ps -a` 来查看；

* 如果容器还在运行，可以先 `docker stop <容器 ID 或名称>` 停止容器或使用 `docker rm -f <容器 ID 或名称>` 来强制删除。

## 删除镜像

```bash
docker rmi <容器 ID 或容器名称>
```

* 其中 <容器 ID 或容器名称> 可以通过命令 `docker images` 来查看；

* 如果镜像已经产生了容器实例（已经 `docker run` 过），需先删除容器实例才能删除镜像，或添加 `-f` 参数强制删除。

## 更新容器

```bash
docker stop <容器 ID 或容器名称>  # 停止容器
docker rm <容器 ID 或容器名称>  # 删除旧镜像
docker pull <容器 ID 或容器名称> # 拉取新镜像
docker run <容器 ID 或容器名称>  # 启动新容器
```

---

*参考链接：*

1. [docker常规操作——删除容器实例、删除镜像_运维_Michel Liu-CSDN博客](https://blog.csdn.net/Michel4Liu/article/details/80890661)