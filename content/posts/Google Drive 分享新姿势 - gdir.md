+++
title = "Google Drive 分享新姿势 - gdir"
description = " "
date = "2020-05-21T23:05:00+08:00"
categories = ["奇技淫巧"]
tags = ["gd","gdir","gclone"]
slug = "usage-of-gdir"
draft = false
+++

**系统环境：** CentOS 7 x64

拥有了 [无限容量的 Google Drive](https://blog.vwert.com/CloudStorage/GoogleDriveUnlimited.html) 之后，再加上对 [搬山之术](/posts/usage-of-gclone/) 的灵活运用，想必各位的 GD 内已经收藏了不少存货，那么下一步你可能就需要想个办法便捷地将资源分享给朋友们了。毕竟访问 GD 需要稳定的梯子，不是每个人都有这样的条件 / 愿意折腾。如果能够方便地搭建一个网站，让基友们通过浏览器来便捷地访问你精心收藏的资源，想必对于各位增进感情是十分有帮助的 :)

办法当然是有的，而且有几种不同的思路：

- 在 VPS 挂载，再通过 [h5ai](https://larsjung.de/h5ai/) 之类的 FTP 程序来搭建 Web 服务；
- 使用 GoIndex 这样的目录列表程序，借助 Cloudflare 的 Workers 免费服务，无需服务器，即可搭建一个用于分享 GD 资源的在线网盘了；
- ……

本来俺个人使用的是第二种方案，但 GoIndex 作者已经删库，恰好最近又发现了一个功能类似但更加灵活好用的项目 - gdir，项目地址：

> [https://github.com/workerindex/gdir](https://github.com/workerindex/gdir)

似乎项目作者太忙了没顾上写文档，好在程序本身配置很简单，经过一番摸索后俺已经成功部署了起来，下面分享一下过程，可以看作是 gdir 的一个简单教程。

正文之前先介绍下本项目相比 GoIndex（以及相关魔改项目）的优势：

- 多用户权限控制，可以方便地控制每个用户的能看到的目录范围；
- 防盗链，之前用 GoIndex 时基友把某部动漫的直链放进了迅雷里，然后俺的域名就被 CF 给 ban 了……  
    而使用本项目，每次登陆后生成的直链地址后面都有一串随机字符串，重新登陆后这串乱码的值会变，可以防止迅雷吸血；
- 突破 GD 外链下载的单日限额；
- 限制权限的搜索功能；
- 方便使用团队盘；
- 配置文件加密后保存在 Gist 的私有记录中。

## 1.准备工作

> 号外：
> 
> > 更新：
> > 
> > 项目更新了，去除了对于 Node.js 和 NPM 的依赖，因此以下手动教程中的【Node.js with NPM】这一节可以略过；作者的一键脚本也进行了相应的更新，以下操作不受影响。
> > 
> gdir 项目作者（不是俺哈）写了个一键脚本来方便大家安装本节的【Git】， ~~【Node.js with NPM】~~ 和【Golang toolchain】这 ~~三~~ 两部分，适用于 CentOS，Debian，Ubuntu 系统，只需执行下面这一条命令即可：
>
> ```
> eval "$(curl -Lso- https://gist.githubusercontent.com/rixtox/05d5ecd0b067f055cddda436efc9035f/raw/gdir-env-setup.sh)"
> ```
> 
> 安装成功后可以直接跳到【[授权文件](#授权文件)】部分继续阅读，如果一键脚本安装不成功，再尝试以下的手动方式安装。

### Git

只需一行命令：

```bash
yum install -y git
```

### Node.js with NPM

先安装依赖：

```bash
yum install gcc-c++ make
```

执行官方脚本：

```bash
curl -sL https://rpm.nodesource.com/setup_14.x | bash -
```

安装 Node.js：

```bash
yum install -y nodejs
```

### Golang toolchain

**注意：如果你的系统不是 CentOS 7 x64，那么下面的方法可能不适用，请参考 Golang 官网的指导来安装：[https://golang.org/doc/install](https://golang.org/doc/install) 。**

先安装 epel 源：

```bash
yum install -y epel-release
yum clean all && yum makecache
```

安装 Golang：

```bash
yum install -y golang
```

配置环境变量：

```bash
vi /etc/profile
```

在最后加上以下内容：

```bash
# GOPATH
export GOPATH=/root/go
# GOPATH bin
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

使环境变量生效：

```bash
source /etc/profile
```

### 授权文件

本项目可以使用 Service Account（以下简称 SA）或个人账号的授权文件来生成目录列表。

要使用本项目，SA 不是必需的，但俺推荐配置好 SA 后使用本项目。不知道 SA 是什么以及怎样生成的请参考 [gclone 搬山之术](/posts/usage-of-gclone/) 这篇文章中的【1.借助 AutoRclone 批量生成 SA】和【2.为 SA 添加权限】这两个部分来配置，完成后记住保存 SA 授权文件的路径（本文默认是在 `/root/AutoRclone/accounts/`）。

如果你要使用个人账号的授权文件，那么请自行搜索如何获取。

### Cloudflare 的 Global API Key

打开 [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)，点击下方 `Global API Key` 右边的 `View` 即可查看，保存下来备用。

### GitHub 的 Personal access token

打开 [https://github.com/settings/tokens](https://github.com/settings/tokens)，点击右上角的 `Generate new token`，`Note` 随便填写一个，不过最好能方便自己认出用途，下面的权限只需要勾选 `gist` 就够了，选多了反而会导致后面配置 gdir 时报错，而且不安全，最后点击下方绿色的 `Generate token`，保存好生成的 Token 代码，后面会用到。注意这串代码只会在生成后显示一次，如果忘记了就要删掉重新生成。

## 2.配置 gdir

### 快速开始

首先 clone 项目仓库并进入，后文中的配置命令都需要在项目仓库路径下（本文中为 `~/gdir/`）执行：

```bash
git clone https://github.com/workerindex/gdir.git && cd gdir
```

开始配置：

```bash
go run ./tools/setup
```

配置 gdir 本身其实十分简单，全程响应式配置，下面简单记录一下流程：

```bash

                                  _ _
                          __ _ __| (_)_ _
                         / _` / _` | | '_|
                         \__, \__,_|_|_|
                         |___/'

Your Cloudflare login Email: example@gmail.com

Please visit https://dash.cloudflare.com/profile/api-tokens and get
your Global API Key: *************************************  
# 此处粘贴前面保存的【Cloudflare 的 Global API Key】

Your available Cloudflare accounts:
    (1) example@gmail.com\'s Account [*************************************]
Choose an account: 1
Your available Cloudflare Workers:
    (1) **
    (2) **
Choose one or enter 0 to create a new Worker: 0
# 此处输入 0 来新建一个 Worker
Naming rule:
    start with a letter
    end with a letter or digit
    include only letters, digits, underscore, and hyphen
    be 63 characters or less
Please enter a name for your new Worker: gdir
# 输入 Worker 的名字，命名规则见上，本文以“gdir”为例
Please visit https://github.com/settings/tokens and generate a new
token with "gist" scope: ****************************************
# 此处粘贴前面保存的【GitHub 的 Personal access token】

Specify how you want to configure your Accounts Gist:
    (1) Create a new Gist                   (default)
    (2) Enter an existing Gist URL / ID
Please enter your choice: 1
# 如何配置存放【SA 信息】的 Gist，默认为新建一个
Specify how you want to configure your Users Gist:
    (1) Create a new Gist                   (default)
    (2) Enter an existing Gist URL / ID
Please enter your choice: 1
# 如何配置存放【用户信息】的 Gist，默认为新建一个
Specify how you want to configure your Static Gist:
    (1) Create a new Gist                   (default)
    (2) Enter an existing Gist URL / ID
Please enter your choice: 1
# 如何配置存放【静态文件】的 Gist，默认为新建一个
Specify how you want to configure your gdir secret key:
    (1) Generate secure random value           (default)
    (2) Enter your own secret key      (not recommended)
Please enter your choice: 1
# 如何配置 gdir 密钥，默认为生成一个安全的随机值
Please enter account candidates rotations interval (default 60): 60
# 切换 SA 的时间间隔，默认为 60，单位为秒
Please enter account candidates size (default 10): 10
# 切换 SA 的范围，默认为 10 个
Please follow https://github.com/xyou365/AutoRclone to generate
Accounts JSON directory: /root/AutoRclone/accounts/
# SA 授权文件的路径
Encrypting account 1: 00cf37077e264b8071554ada0b74588c46db9825.json
Encrypting account 2: 02a1e033857fb036176b76b5dc985e9e4a0eaf49.json
...
Encrypting account 100: ff6b534ae37217ba92515f049f216b55c4e4bcc4.json
Add an admin user...
Please enter your admin user name: he-sb
# 管理员用户名
Please enter your admin user password:
# 管理员密码
...
Initializing Git repo in accounts...
Specify which protocol you want to configure your accounts Git repo:
    (1) HTTPS                                  (default)
    (2) SSH            (needs SSH key setup with GitHub)
Please enter your choice: 1
# 选择【配置存放 SA 信息的 Gist】时使用的协议，默认为 HTTPS
Initialized empty Git repository in /root/gdir/accounts/.git/
Deploying accounts to Gist...
[master (root-commit) b80ec00]
 100 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 1
 ...
 create mode 100644 99
Username for 'https://gist.github.com': he-sb
# 输入 GitHub 用户名
Password for 'https://he-sb@gist.github.com':
# 输入 GitHub 登陆密码
Counting objects: 102, done.
Compressing objects: 100% (102/102), done.
Writing objects: 100% (102/102), 239.71 KiB | 0 bytes/s, done.
Total 102 (delta 0), reused 0 (delta 0)
To https://gist.github.com/********************************.git
 + 2c42004...b80ec00 master -> master (forced update)
Branch master set up to track remote branch master from origin.
Initializing Git repo in users...
Specify which protocol you want to configure your users Git repo:
    (1) HTTPS                                  (default)
    (2) SSH            (needs SSH key setup with GitHub)
Please enter your choice: 1
# 选择【配置存放用户信息的 Gist】时使用的协议，默认为 HTTPS
Initialized empty Git repository in /root/gdir/users/.git/
Deploying users to Gist...
[master (root-commit) 6de81cc]
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 4c5d01882c8deb3f36ae40474ffb08ad1c4e004f1bf0d5efb4a157933cebfbb1
Username for 'https://gist.github.com': he-sb
Password for 'https://he-sb@gist.github.com':
# 输入 GitHub 用户名与密码
Counting objects: 3, done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 317 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://gist.github.com/********************************.git
 + 04444a2...6de81cc master -> master (forced update)
Branch master set up to track remote branch master from origin.
Initializing Git repo in static...
Specify which protocol you want to configure your static Git repo:
    (1) HTTPS                                  (default)
    (2) SSH            (needs SSH key setup with GitHub)
Please enter your choice: 1
# 选择【配置存放静态文件的 Gist】时使用的协议，默认为 HTTPS
Initialized empty Git repository in /root/gdir/static/.git/
Deploying static to Gist...
[master (root-commit) a92d00b]
 5 files changed, 688 insertions(+)
 create mode 100644 app.js
 create mode 100644 app.js.map
 create mode 100644 favicon.ico
 create mode 100644 index.html
 create mode 100644 styles.css
Username for 'https://gist.github.com': he-sb
Password for 'https://he-sb@gist.github.com':
# 输入 GitHub 用户名与密码
Counting objects: 7, done.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 15.10 KiB | 0 bytes/s, done.
Total 7 (delta 0), reused 0 (delta 0)
To https://gist.github.com/********************************.git
 + 5918211...a92d00b master -> master (forced update)
Branch master set up to track remote branch master from origin.
Deploying Cloudflare Worker gdir...
Now you can go to https://dash.cloudflare.com/*************************************/workers/view/gdir to checkout your Worker!
# 访问这个网址可以查看 Worker
# 点击 Deployed to gdir.**.workers.dev 旁边的开关就可以部署了
Check here to create custom routes with your own domain names:
https://developers.cloudflare.com/workers/about/routes/
# 访问这个地址来查看如何配置自定义域名
```

至此，gdir 已经可以使用了。

### 指定显示哪些盘 / 文件夹

刚配置好的管理员用户是可以显示所有有权限的团队盘的，会有一些不便，如果只想显示其中某一个或某几个盘，可以使用白名单模式，既可以修改现有用户的权限也可以新增用户来设置权限：

```bash
go run ./tools/adduser
```

一样是响应式配置，本文以修改之前创建好的管理员用户的权限为例：

```bash
Loading existing config from config.json
Username: he-sb
# 输入想要修改的用户的用户名
Password: ******
# 密码信息是明文显示的，此处俺手动打码了 :)
Is it correct? (Y/n) y
# 确认以上信息有无问题
The user currently has global access to all drives.
# 该用户当前可以查看全部团队盘
Please specify what do you want to do with it:
    (1) Confirm                         (default)
    (2) Convert to white-list access control list
    (3) Convert to black-list access control list
Please enter your choice: 2
# 转换为白名单模式，只能查看指定的盘
(Use comma to separate between drive IDs.)
Enter white-list access control list of drives: *******************,*******************,*******************
# 输入想显示的团队盘 ID，有多个的话用英文逗号隔开分隔，
# 不知如何查看请参考【gclone 搬山之术】这篇文章
The user currently has following drives in its white-list access list:
    (1) *******************
    (2) *******************
    (3) *******************
Please specify what do you want to do with it:
    (1) Confirm                         (default)
    (2) Append drives to the list
    (3) Remove drives from the list
    (4) Replace with a new list of drives
    (5) Convert to black-list access control
    (6) Disable access control on the user
Please enter your choice: 1
# 确认下想展示的盘，在此处可以新增盘，移除已有的盘，
# 使用新的白名单列表来替换当前的，将列表转换为黑名单，
# 或关闭该用户的访问权限
Saving user to users/4c5d01882c8deb3f36ae40474ffb08ad1c4e004f1bf0d5efb4a157933cebfbb1 ... Deploying users to Gist...
[master f287a63]
 1 file changed, 1 insertion(+), 2 deletions(-)
Username for 'https://gist.github.com': he-sb
Password for 'https://he-sb@gist.github.com':
# 输入 GitHub 用户密码来更新 Gist
```

### 修改 / 添加用户

继续使用这条命令：

```bash
go run ./tools/adduser
```

- 修改用户时输入已有用户的用户名；
- 新增用户时输入新的用户名和想要设置的密码，注意新添加的用户默认是和管理员一样的权限，可查看所有团队盘，要修改的话参考上一节来操作即可。

## 3.注意事项

1. gdir 项目作者建议使用普通用户而非 root 用户来配置 gdir，且项目 Issues 中有位用户提到使用 root 用户来配置，产生了很多报错，最后切换为普通用户配置就正常了（[Failed to run setup · Issue #1 · workerindex/gdir](https://github.com/workerindex/gdir/issues/1)）。但俺经过本文的方法配置，全程使用 root 用户也未见任何异常，推测是 Node.js 或 Golang 安装方法与本文不同导致的权限问题。如果你也使用 CentOS 7 x64，那么按照本文方法来配置是不会有问题的，否则请尝试切换为普通用户再行配置；

2. VPS 或本地机器只是用来生成 CF Worker 的配置文件用，配置好后部署和数据传输都是走 CF；

3. 接上条，如果确定以后不用再修改配置的话，本地的项目文件夹可以删掉；

    但是不建议这样做，项目文件夹下存放有配置文件，Gist 中存放的只是加密后的内容，如果删了本地的项目文件夹，日后想要对用户进行增删改操作就很麻烦（你可能需要手动删掉旧的 Worker 和 Gist 项目，然后 clone gdir 项目重新配置）；

4. 想到了再写，遇到问题欢迎在文章下方留言或去项目仓库给作者提 Issues，这样可以帮到更多的人。

---

*参考链接：*

1. [Linux 使用Yum安装Go和配置环境](https://liangbogopher.github.io/2018/04/05/linux-install-go)