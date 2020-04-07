+++
title = "使用 Supervisor 实现 Linux 进程守护"
description = " "
date = "2020-04-01T22:58:39+08:00"
categories = ["Linux"]
tags = ["进程守护","supervisor"]
slug = "supervisor-in-linux-daemon"
draft = false
+++

**系统环境：** CentOS 7 x64

[Supervisor](http://supervisord.org/) 是 Linux/Unix 下一个使用 Python 开发的进程管理工具，可以很方便的监听、启动、停止、重启一个或多个进程。用 Supervisor 管理的进程，如果意外停止运行，Supervisor 监听到进程死后，会自动将它重新拉起，可以很方便的实现进程守护。

## 安装 Supervisor

```bash
yum install supervisor
```

如果以上命令不起作用，则需要先添加 `epel` 源：

```bash
yum install -y epel-release
````

完成后再执行上条命令安装 Supervisor 。


## 配置 Supervisor

安装完后会生成一个主配置文件 `/etc/supervisord.conf` 和一个自定义配置文件目录 `/etc/supervisord.d` ，有两种方式来配置 Supervisor ：

1. 在主配置文件末尾添加新的配置。

2. 在自定义配置文件目录下为每个需要守护的进程新建对应的配置文件。

个人倾向于使用方法二，比较便于管理。下面以方法二为例，方法一原理是相同的。

首先编辑主配置文件，指定自定义配置文件的后缀：

```bash
vi /etc/supervisord.conf
```

在末尾添加下面内容后保存：

```bash
[include]
files = /etc/supervisord.d/*.conf
```

然后在自定义配置文件目录下新建配置文件：

```bash
vi /etc/supervisord.d/ProjectName.conf
```

在其中按以下格式添加内容，分号后为注释，可以不写：

```ini
[program: ProjectName]  ; 冒号后面是进程名，可自定义，便于识别即可
command=/path/to/programname  ; 需守护的进程命令，命令是绝对路径或相对路径均可
autorestart=true  ; 程序意外退出是否自动重启
autostart=true  ; 是否自动启动
stderr_logfile=/var/log/ProjectName.err.log  ; 错误日志文件
stdout_logfile=/var/log/ProjectName.out.log  ; 输出日志文件
user=root  ; 执行命令的用户身份
startsecs=1  ; 自动重启间隔，单位为秒
```

更多参数详见官方文档，此处就不一一列举了。

## 启动 Supervisor 服务

```bash
supervisord
```

或

```bash
systemctl start supervisord
```

添加开机自启

```bash
systemctl enable supervisord
```

## Supervisor 常用命令

* `supervisorctl stop | start ProjectName` : 停止/启动某个进程
* `supervisorctl restart ProjectName` : 重启某个进程。
* `supervisorctl status` : 查看所有任务状态。
* `supervisorctl reload` : 载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程。
* `supervisorctl update` : 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启。
* 用 `stop` 停止的进程，`reload` 或 `update` 后都不会自动重启。 

---

*参考链接：*

1. [Supervisor 为服务创建守护进程 - alonghub - 博客园](https://www.cnblogs.com/along21/p/10255681.html)

2. [Linux 后台进程管理利器 Supervisor](https://kuanghy.github.io/2016/03/21/supervisor)

3. [Supervisor 官方文档](http://supervisord.org/)