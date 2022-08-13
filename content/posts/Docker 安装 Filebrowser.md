+++
title = "Docker 安装 Filebrowser"
description = " "
date = "2021-08-12T21:07:53+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["filebrowser","docker"]
slug = "setup-filebrowser-in-docker"
draft = false
+++

**系统环境：** CentOS 7 x64

## 前言

[File Browser](https://filebrowser.org/) 是一个开箱即用的网盘程序，配置过程非常简单（使用 Docker 部署就更简单了），但是功能却很全面，包括了：

- 目录列表；
- 在线预览文本和媒体文件；
- 多用户，支持权限控制和可见范围控制；
- 分享，可以自定义密码和分享链接有效期；
- 甚至包含了一个简易的 Web Shell！

[官网](https://filebrowser.org/) 上的安装说明其实已经足够简明了，俺这里就简单记录下配置过程，补充一点注意事项和进阶配置的说明。

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

- `CUSTOM_PATH`：用于展示的目录，也就是 File Browser 可控制的根目录，可以自定义，如果有多个的话，参见后文的进阶配置部分
- `PATH_OF_DATABASE`：存放数据库文件的目录，这一段可以自定义，不过记得在这个目录下手动创建一个名为 `filebrowser.db` 的新【文件】，否则 Docker 会自动在指定的目录下创建一个名为 `filebrowser.db` 的新【目录】而非新【文件】，但容器内的 File Browser 程序还是将这个新【目录】当作数据库文件来读写，会导致错误
- `-p`：冒号【左边】的 `8080` 代表外部访问服务时使用的端口，可以自己改，冒号【右边】的 `80` 是容器内部使用的端口，不要动它

别忘了给这个脚本文件赋予可执行权限：

```shell
chmod +x Filebrowser-run.sh
```

最后执行这个脚本：

```shell
./Filebrowser-run.sh
```

最后防火墙放行 `8080` 端口（或者你自定义的其他端口），不出意外的话访问服务器 IP:8080 就能看到 File Browser 的登录界面了，默认的帐号密码都是 `admin`，登录之后尽快改掉。

## 进阶配置

### 读取多个目录

在上文运行容器的 Docker 命令中，将 `-v /CUSTOM_PATH:/srv \` 替换为下面的形式：

```shell
    -v /CUSTOM_PATH_1:/srv/CUSTOM_FOLDER_1 \
    -v /CUSTOM_PATH_2:/srv/CUSTOM_FOLDER_2 \
    ...
```

注意其中冒号【右边】的 `CUSTOM_FOLDER_1` 和 `CUSTOM_FOLDER_2` 是文件夹名，不要把冒号左边的整个路径都复制过去，自己随便命名就可以。

### 使用域名访问

俺个人是使用的宝塔面板来设置 Nginx 反代的，比较方便，先去 DNS 服务商那里新建一条域名的解析，地址解析到服务器的 IP，然后在宝塔面板中新建一个网站，域名填写刚才新建了解析的域名，PHP 版本选择纯静态就可以，然后在【网站设置】里，配置好 SSL 证书，一键操作就行了，没什么好说的。最后在【反向代理】设置里，新建一条反向代理的规则：

- 目标域名 `http://127.0.0.1:8080` 发送域名 `$host`

目标域名中的端口号注意改成自己之前运行容器时指定的【外部访问端口】，配置好后就可以直接使用域名来访问了，没啥特殊需要的话此时可以配置防火墙关闭 `8080` 端口了，提高一下安全性。

### 使用自定义配置文件

要使用自定义配置文件，在上文运行 Docker 的命令中，在第一行 `docker run -d \` 之后，最后一行 `filebrowser/filebrowser` 之前的任意位置新增一行：

```shell
    -v /PATH_TO_CUSTOM_CONFIG_FILE/.filebrowser.json:/.filebrowser.json \
```

其中 `PATH_TO_CUSTOM_CONFIG_FILE` 是存放自定义配置文件的路径，`.filebrowser.json` 是自定义配置文件的文件名，配置文件是 `json` 格式的，默认的配置文件内容如下：

```json
{
  "port": 80,
  "baseURL": "",
  "address": "",
  "log": "stdout",
  "database": "/database.db",
  "root": "/srv"
}
```

参数说明如下：

- `port`：默认为 `80`，容器内部暴露出来的端口，也就是上文中端口映射时冒号【右边】的端口，有需要的话可以自行修改，运行容器的命令也需要对应修改下；
- `baseurl`：默认为空，也就是默认路径在网站根目录下，看不懂这句话的同学建议不要修改这个。如果不想单独解析一个域名或子域名，或有其他特殊需求，可以将这一项配置为形如 `"/CUSTOM_PATH"` 的内容，这样访问 File Browser 的路径就变成了 `{[DOMAIN]/[IP:PORT]}/CUSTOM_PATH` ；
- `address`：【容器内】的 File Browser 程序监听的地址，没有特殊需要的话不用动这个；
- `log`：日志输出的位置；
- `database`：默认值为 `"/database.db"`，【容器内】数据库文件的位置，同样的，没有特殊需求的话不用改；
- `"root"`：默认值为 `"/srv"`，【容器内】 File Browser 读写的目录。

---

*参考链接：*

1. [Welcome - File Browser](https://filebrowser.org/)