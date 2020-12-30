+++
title = "efb-wechat-docker 绑定过程记录"
description = " "
date = "2020-02-24T12:37:35+08:00"
categories = ["奇技淫巧"]
tags = ["efb","wechat","docker","telegram"]
slug = "usage-of-efbwechat-docker"
comments = true
draft = false
+++
**系统环境：** Centos 7 x64

## 初次使用

### 新建Telegram Bot

找 [@BotFather](https://telegram.me/botfather) 创建个用来对接微信的 Bot，向它发送 `/newbot` 启动向导，期间需设置 Bot 的 `name` 和 `user name` ，注意 `user name` 必须以 `bot` 结尾，创建完毕后 @BotFather 会返回一个密钥（Token），先记录下来备用。

> **注意，此 Bot 名与密钥不要提供给任何人，否则可能导致聊天信息泄露等风险。**

接下来配置刚建好的 Bot：

* 发送 `/setprivacy` 到 @BotFather，选择刚创建好的 Bot，选择 `Disable` 。
* 发送 `/setjoingroups` 到 @BotFather，选择刚创建好的 Bot，选择 `Enable`。
* 发送 `/setcommands` 到 @BotFather，选择刚创建好的 Bot，发送以下内容：
```
link - 将回话绑定到Telegram群组
chat - 生成会话头
recog - 恢复语音消息以进行识别
extra - 获取更多功能
```

### 获取 User ID

Telegram 中每位用户有一个唯一的数字 ID，可通过 Bot 来查询，以下是一些个人目前已知的 Bot：

* [@get_id_bot](https://t.me/get_id_bot) 发送 `/start`
* [@XYMbot](https://t.me/xymbot) 发送 `/whois`
* [@mokubot](https://t.me/mokubot) 发送 `/whoami`
* [@GroupButler_Bot](https://t.me/groupbutler_bot) 发送 `/id`
* [@jackbot](https://t.me/jackbot) 发送 `/me`
* [@userinfobot](https://t.me/userinfobot) 发送任意内容
* [@orzdigbot](https://t.me/orzdigbot) 发送 `/user`

获得自己的 User ID 后记录下来备用。

### 启动 Docker 并登陆

~~Docker使用 [mikubill/efbwechat](https://hub.docker.com/r/mikubill/efbwechat) 这个镜像，尝试了一圈这个是启动最省心的。~~

> 2020/12/30 更换镜像为 [yhndnzj/efb](https://hub.docker.com/r/yhndnzj/efb) ，因为之前的很久不更新了，而且配置复用太麻烦，也不支持发送 GIF。

首先执行这个命令来初始化配置文件：

```bash
curl -fsSL https://github.com/YHNdnzj/efb-docker/raw/master/init.sh | bash
```

上面的脚本会在 `/etc/ehforwarderbot` 这个路径下生成默认的配置文件，修改一下：

```bash
vi /etc/ehforwarderbot/profiles/wechat/blueset.telegram/config.yaml
```

文件内容是这样的：

```bash
token: "TOKEN"
admins: 
- ID
```

把第一行引号中的 `TOKEN` 替换为刚才新建 Bot 时保存的 Token，第三行的 `ID` 替换为刚才保存的 Telegram User ID，然后启动镜像：

```bash
docker run -d -e EFB_PROFILE=wechat --name efb-wechat -v /etc/ehforwarderbot:/etc/ehforwarderbot yhndnzj/efb
```

启动镜像没有报错的话输入下面这条命令

```bash
docker logs -f efbwechat
```

不出意外的话，稍等片刻终端内即会显示二维码，打开手机微信扫描，确认登陆即可。

### Some Tips

1. 若运行Docker的机器在国外而登陆微信的手机在国内，会有微信网页版登陆权限被封禁的风险；
2. 运行Docker期间手机微信不可长期不在线（此处“长期”指超过24小时。。），否则网页版会被踢下线，需重新登陆；
3. 可以在Telegram内新建若干群组并拉之前建好的微信 Bot 进群，并将各个微信群、好友、公众号等 `link` 至相应群组即可实现方便的私聊 / 免打扰等功能，自行探索。

## 老司机使用

此镜像的配置文件在 `/etc/ehforwarderbot` 这个文件夹内，如服务器重装或迁移时，备份这个文件夹并在重新部署容器时挂载上即可。

---

*参考链接：*

1. [EFB How-to: Send and Receive Messages from WeChat on Telegram (zh-CN) — 1A23 Blog](https://blog.1a23.com/2017/01/09/EFB-How-to-Send-and-Receive-Messages-from-WeChat-on-Telegram-zh-CN/#0x030-创建-Telegram-Bot)
2. [Mikubill/efb-wechat-docker: EFB WeChat Slave Docker Ver.](https://github.com/Mikubill/efb-wechat-docker)
3. [为docker文件挂载指定卷的问题 · Issue #7 · Mikubill/efb-wechat-docker](https://github.com/Mikubill/efb-wechat-docker/issues/7)
4. [YHNdnzj/efb-docker: Docker image for ehForwarderBot](https://github.com/YHNdnzj/efb-docker)