+++
title = "aria2 加入开机启动"
description = " "
date = "2018-08-17T22:02:25+08:00"
categories = ["Linux"]
tags = ["aria2","fedora"]
slug = "aria2-auto-start-when-system-boot"
comments = true
draft = false
+++
**系统环境：** Fedora27 Desktop Workstation x64

折腾半天弄好了aria2的开机启动，记录一下过程。

添加服务

```bash
sudo vi /etc/systemd/system/aria2c.service
```

写入如下内容

```bash
[Unit]
Description=aria2
After=network.target

[Service]
ExecStart=/usr/bin/aria2c --conf-path=/home/j0ker/.aria2/aria2.conf
ExecStop=/bin/kill $MAINPID
RestartSec=always

[Install]
WantedBy=multi-user.target
```

启动，添加自启，查看状态，重启

```bash
sudo systemctl start aria2c
sudo systemctl enable aria2c
sudo systemctl status aria2c
sudo systemctl restart aria2c
```