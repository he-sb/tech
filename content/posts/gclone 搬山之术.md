+++
title = "gclone 搬山之术"
description = " "
date = "2020-05-02T16:03:25+08:00"
categories = ["奇技淫巧"]
tags = ["gclone","rclone","gd"]
slug = "usage-of-gclone"
draft = false
+++

Google Drive 是个好东西，无限容量（普通用户容量为 15 G ，不要问俺无限容量是怎么做到的，懂的都懂）配合上 Rclone 这样的 API 工具，简直是资源收集 / 收藏的神器，不过谷歌对于每个账户的 API 限制是 750 G / 24 H ，对于各位搬山道人来说每天只能搬运 750 G 的资源肯定是不够的。最近了解到一个神器 - gclone（项目地址：[https://github.com/donwa/gclone](https://github.com/donwa/gclone)），可以看作是 Rclone 的升级版，借助它，再搭配一些奇技淫巧，就可以突破每天 750 G 的 API 限制了。

原理是这样的，单日 750 G 的限制是谷歌对每个账号进行的限制，而我们可以借助谷歌开发者平台创建一些项目（Project），每个项目下可以创建很多服务账号（即 Service Accounts ，以下简称 SA），每个 SA 都有自己的 API ，也就是说每个 SA 每天都有 750 G 的限额。通过赋予这些 SA 对于【源盘】（即待转存资源的存放位置，可以是个人盘的共享目录，团队盘或团队盘内的文件夹）和【目的盘】（即资源转存后的存放位置，可以是团队盘或团队盘内的文件夹）的编辑权限，再通过 gclone ，当每个 SA 达到 750 G 限制后自动切换，就可以突破单账号的单日 750 G 限制。

其中，每个谷歌账号可以创建最多 12 个项目，每个项目下可以生成最多 100 个 SA ，即使只使用 1 个项目，也可以实现一天搬运 1 x 100 x 750 G = 75 T ，相信对大部分人来说已经足够了，不够的话多创建几个项目就行（注意每个团队盘最多添加 600 个账号）。

以下是配置过程，本文基于 CentOS 7 x64 服务器。

## 1.借助 AutoRclone 批量生成 SA

批量生成 SA 需要用到另外一个项目 - AutoRclone（项目地址：[https://github.com/xyou365/AutoRclone](https://github.com/xyou365/AutoRclone)。

首先安装 python 3 和 AutoRclone：

```bash
yum install -y python3 python3-pip git
git clone https://github.com/xyou365/AutoRclone
cd AutoRclone
pip3 install -r requirements.txt
```

然后打开这个地址：[https://developers.google.com/drive/api/v3/quickstart/python](https://developers.google.com/drive/api/v3/quickstart/python) ，点击【Step 1: Turn on the Drive API】下面的 ` Enable the Drive API` 按钮，弹出的【Configure your OAuth client】对话框中保持默认的 `Desktop app` 不要动，点击右下角 `CREATE` 按钮，开启成功之后点击 `DOWNLOAD CLIENT CONFIGURATION` 下载生成的 `credentials.json` ，再将下载到本地的 `credentials.json` 上传至服务器的 `AutoRclone` 文件夹下。

此时回到 SSH ，执行 `python3 gen_sa_accounts.py --quick-setup 1` ，复制返回的网址至浏览器打开，登陆拥有团队盘的账号，选择 `允许` ，然后复制返回的授权代码，粘贴至 SSH 终端，再复制新返回的网址至浏览器打开，使用刚才的账号登陆，点击 `启用` ，回到 SSH 终端内按下回车，此时应该开始创建 SA 了，稍等片刻完成后可以看到 `/root/AutoRclone/accounts/` 目录下出现了一大堆 `.json` 后缀的 SA 授权文件。

这些授权文件就是本文搬山之术的核心，其中记录了每个 SA 相应的权限信息。

## 2.为 SA 添加权限

这个部分非常繁琐，主要是因为这一过程俺只能手动完成，如果你有 G-Suite 管理员权限，那么可以参考 [这里](https://panoan.top/index.php/archives/3/#toc_19) 来快速添加，下面是手动方法：

在此处：[https://console.developers.google.com/apis/dashboard](https://console.developers.google.com/apis/dashboard) ，点击左侧的 `凭据` 可以看到创建好的项目及 SA ，自行想办法将 SA 的邮箱地址保存下来。

接下来创建群组。打开 [https://groups.google.com/](https://groups.google.com/) ，点击左上角的 `新建群组` ，填写相关信息后新建一个群组，建立好后点击右上角 `管理群组` ，点击左侧 `直接添加成员` ，将刚才保存的 SA 邮箱粘贴过来，注意每次只能添加 10 个，24 小时内只能添加 100 个，每个群组最多添加 600 个，邮箱地址间使用英文逗号分隔。

全部添加好之后点击右上角 `关于` ，记下【群组电子邮件】的地址，之后要对所有加入了群组的 SA 添加权限时，只需要添加至这个群组邮箱就行了。

## 3.使用 gclone 开始搬山

繁琐的配置过程完成后，安装 gclone ：

```bash
bash <(wget -qO- https://git.io/gclone.sh)
```

查看 gclone 版本：

```bash
gclone --version
```

编辑 Rclone 配置文件：

```bash
vim ~/.config/rclone/rclone.conf
```

在最后加上下面内容：

```conf
[gc]
type = drive
scope = drive
service_account_file = /root/AutoRclone/accounts/1.json 
service_account_file_path = /root/AutoRclone/accounts/
```

其中 `service_account_file_path` 是存放 SA 账号授权文件的文件夹路径，不要漏了最后的 `/` ，上面一行是该路径下任意一个 SA 账号授权文件的路径，注意文件名，例子中 `1.json` 仅为简写，请务必修改为一个 SA 账号文件夹下的真实文件名。

用法：

命令参数参考 Rclone ，都是一样的，直接将 `rclone` 换成 `gclone` 即可无障碍使用；

唯一不一样的地方在于，原版 Rclone 如果跨团队盘或者共享文件夹，需要多个配置盘符用于操作，gclone 支持对目录和文件的 ID 进行操作：

1. 目录 ID ：

    共享目录和团队盘应该带 `--drive-server-side-across-configs`：

    ```bash
    gclone copy gc:{【源盘】ID} gc:{【目的盘】ID}  --drive-server-side-across-configs
    ```

    目录 ID 可以是：普通目录，共享目录，团队盘； 
  
    也支持目录 ID 后跟后续路径：

    ```bash
    gclone copy gc:{【源盘】ID} gc:{【目的盘】ID}/media/  --drive-server-side-across-configs
    ```

2. 文件 ID：

    共享目录和团队盘应该带 `--drive-server-side-across-configs`：

    ```bash
    gclone copy gc:{共享文件的 ID} gc:{【目的盘】ID}  --drive-server-side-across-configs
    ```

    支持目录 ID 后，跟后续路径：

    ```bash
    gclone copy gc:{共享文件的 ID} gc:{【目的盘】ID}/media/  --drive-server-side-across-configs
    ```

## 4.一些注意事项

1. 生成 SA 及创建 Google Groups 使用普通的谷歌账号就可以；

2. 创建好 SA ，并将其加入到群组之后，对于需要搬运的资源，保证【源盘】和【目的盘】都给这个群组的【群组邮箱】权限就能搬了，【源盘】有读取权限就够了，【目的盘】需要编辑权限；

3. 基于以上两点，即使你没有 G-Suite 账号（即没法自己开团队盘），只要加入了别人开给你的团队盘（网上有部分大佬提供这方面的资源，收费免费的都有，请自行寻找），并有加人权限，那么就可以参照本文教程开始搬山了；

4. 如果要更换机器，只需要把【SA 授权文件】（本例中在 `/root/AutoRclone/accounts/` 路径下）及【gclone 配置文件】（本例中路径为 `/root/.config/rclone/rclone.conf`）备份，在新机器上安装好 gclone 并在配置文件中修改一下 SA 授权文件的路径就可以直接使用了；

5. 生成 SA 过程中如果 AutoRclone 遇到什么奇怪的错误，请善用 `python3 gen_sa_accounts.py --help` 。

---

*参考链接：*

1. [AutoRclone配合gclone突破GoogleTeamDrive750G流量限制 - 幽游地](https://www.uud.me/qiwenzalun/autorclone-gclone.html)

2. [gclone/README_zh.md at master · donwa/gclone](https://github.com/donwa/gclone/blob/master/README_zh.md)

3. [可能是最简单的AutoRclone教程：如何突破Google Drive每日750G限制？ - Great Panoan](https://panoan.top/index.php/archives/3/)