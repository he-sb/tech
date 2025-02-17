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
help - 显示命令列表.
link - 将远程会话绑定到 Telegram 群组
chat - 生成会话头
recog - 回复语音消息以进行识别
info - 显示当前 Telegram 聊天的信息.
unlink_all - 将所有远程会话从 Telegram 群组解绑.
update_info - 更新群组名称和头像
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

> 2025.01.03 镜像默认没有安装任何插件，有需要的话可以自行进入容器内安装

> 2024.03.20 镜像更换为和小伙伴合作打包的 [j0k3rh/efb-wechat](https://hub.docker.com/r/j0k3rh/efb-wechat/)，下文的教程已经适配了这个镜像

> ~~2020/12/30 更换镜像为 [yhndnzj/efb](https://hub.docker.com/r/yhndnzj/efb) ，因为之前的很久不更新了，而且配置复用太麻烦，也不支持发送 GIF。~~

~~Docker使用 [mikubill/efbwechat](https://hub.docker.com/r/mikubill/efbwechat) 这个镜像，尝试了一圈这个是启动最省心的。~~

首先克隆仓库，仓库中包含了初始化配置文件：

```bash
git clone https://github.com/he-sb/efb-update efb-update
```

修改 `efb-update/data/profiles/default` 路径下的配置文件：

1. 主配置文件 `efb-update/data/profiles/default/config.yaml`
    - ~~`middlewares` 定义了启用的转发通道和中间件~~
        - ~~`catbaron.voice_recog` 语音转文字~~
        - ~~`patch.PatchMiddleware` 手机微信标记已读~~
        - ~~默认启用两个插件，如果不需要某个插件，删除或注释对应的行即可~~
        - ~~如果两个都不需要，可以直接删除或注释 `middlewares` 小节~~

2. Telegram 配置 `efb-update/data/profiles/default/blueset.telegram/config.yaml`
    - `token`
        - Telegram 的 bot token
        - 后方的值替换为刚才新建 Bot 时保存的 Token
    - `admins`
        - Telegram 账号的数字 ID
        - 下方的值替换为刚才保存的 Telegram User ID
    - `request_kwargs`
        - 如果镜像部署在国内机器上（国内 VPS 或者 HomeLab，软路由之类的场景），那么需要在这里配置 http 代理才能调用 Telegram Bot
        - 需要时取消这一行以及下方 `proxy_url` 这行的注释，并修改代理地址
        - 如果 http 代理需要鉴权，那么再取消下方 `username` 和 `password` 行的注释，并将其修改为正确的代理鉴权信息

3. wechat 配置 `efb-update/data/profiles/default/blueset.wechat/config.yaml`
    - 其他可用的配置及含义参考插件仓库： [ehForwarderBot/efb-wechat-slave](https://github.com/ehForwarderBot/efb-wechat-slave?tab=readme-ov-file#%E5%AE%9E%E9%AA%8C%E5%8A%9F%E8%83%BD)

4. ~~插件 `catbaron.voice_recog` 配置 `efb-update/profiles/default/catbaron.voice_recog/config.yaml`~~
    - ~~语音转文字使用的 API 配置~~
    - ~~配置方法参考插件仓库： [catbaron0/efb-voice_recog-middleware](https://github.com/catbaron0/efb-voice_recog-middleware)~~

5. ~~插件 `patch.PatchMiddleware` 配置 `efb-update/profiles/default/patch.PatchMiddleware/config.yaml`~~
    - ~~`auto_mark_as_read` 是否自动在手机微信标记已读~~
    - ~~`remove_emoji_in_title` 是否移除 Telegram 群组名称中的 emoji~~
    - ~~其他可用配置参考插件仓库： [ehForwarderBot/efb-patch-middleware](https://github.com/ehForwarderBot/efb-patch-middleware)~~

然后启动镜像：

```bash
docker compose up -d
```

启动镜像没有报错的话输入下面这条命令

```bash
docker logs -f efb-wechat
```

不出意外的话，稍等片刻终端内即会显示二维码，打开手机微信扫描，确认登陆即可。

### Some Tips

1. 若运行Docker的机器在国外而登陆微信的手机在国内，会有微信网页版登陆权限被封禁的风险；
2. 运行Docker期间手机微信不可长期不在线（此处“长期”指超过24小时。。），否则网页版会被踢下线，需重新登陆；
3. 可以在Telegram内新建若干群组并拉之前建好的微信 Bot 进群，并将各个微信群、好友、公众号等 `link` 至相应群组即可实现方便的私聊 / 免打扰等功能，自行探索。

## 老司机使用

此镜像内的配置文件和所有的微信绑定关系，都在 `efb-update/data/profiles` 这个文件夹内，如服务器重装或迁移时，备份这个文件夹并在重新部署容器时挂载上即可。

---

*参考链接：*

1. [EFB How-to: Send and Receive Messages from WeChat on Telegram (zh-CN) — 1A23 Blog](https://blog.1a23.com/2017/01/09/EFB-How-to-Send-and-Receive-Messages-from-WeChat-on-Telegram-zh-CN/#0x030-创建-Telegram-Bot)
2. [Mikubill/efb-wechat-docker: EFB WeChat Slave Docker Ver.](https://github.com/Mikubill/efb-wechat-docker)
3. [为docker文件挂载指定卷的问题 · Issue #7 · Mikubill/efb-wechat-docker](https://github.com/Mikubill/efb-wechat-docker/issues/7)
4. [YHNdnzj/efb-docker: Docker image for ehForwarderBot](https://github.com/YHNdnzj/efb-docker)