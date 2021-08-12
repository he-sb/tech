+++
title = "Docker 安装 Filebrowser"
description = " "
date = "2021-08-12T21:07:53+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["filebrowser","docker"]
slug = "setup-filebrowser-in-docker"
draft = true
+++

**系统环境：** CentOS 7 x64

## 前言

[Filebrowser](https://filebrowser.org/) 是一个开箱即用的网盘程序，配置过程非常简单（使用 Docker 部署就更简单了），但是功能却很全面，包括了：

- 目录列表；
- 在线预览文本和媒体文件；
- 多用户，支持权限控制和可见范围控制；
- 分享，可以自定义密码和分享链接有效期；
- 甚至包含了一个简易的 Web Shell！

官网上的安装说明其实已经足够简明了，俺这里就简单记录下配置过程，补充一点注意事项和进阶配置的说明。

## 快速上手

Docker 一条命令即可运行，不过为了可读性和方便复用，俺将这条 Docker 命令保存为了 Shell 脚本：

```shell
vi Filebrowser-run.sh
```

写入以下内容并保存：

```shell
docker run -d \
    --name filebrowser \
    -v /CUSTOM_PATH:/srv \
    -v /PATH_OF_DATABASE/filebrowser.db:/database.db \
    --user $(id -u):$(id -g) \
    -p 8080:80 \
    --restart=always \
    filebrowser/filebrowser
```

参数说明：

- `CUSTOM_PATH`：用于展示的目录，也就是 Filebrowser 可控制的根目录，可以自定义，如果有多个的话，参见后文的进阶配置部分
- `PATH_OF_DATABASE`：存放数据库文件的目录，这一段可以自定义，不过记得在这个目录下手动创建一个名为 `filebrowser.db` 的新【文件】，否则 Docker 会自动在指定的目录下创建一个名为 `filebrowser.db` 的新【目录】而非新【文件】，但容器内的 Filebrowser 程序还是将这个新【目录】当作数据库文件来读写，会导致错误

别忘了给这个脚本文件赋予可执行权限：

```shell
chmod +x Filebrowser-run.sh
```

最后执行这个脚本：

```shell
./Filebrowser-run.sh
```

不出意外的话访问服务器 IP:8080 就能看到 Filebrowser 的登录界面了，默认的帐号密码都是 `admin`，登录之后尽快改掉。

## 进阶配置

### 读取多个目录

*to do*

### 使用域名访问

### 使用 Filebrowser 预设的环境变量

### 使用自定义配置文件

---

*参考链接：*

1. [Welcome - File Browser](https://filebrowser.org/)