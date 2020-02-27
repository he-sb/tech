+++
title = "CentOS 7下安装aria2+AriaNG+Filebrowser实现离线下载"
description = " "
date = "2018-06-15"
categories = ["linux"]
tags = ["aria2","centos","filebrowser","下载"]
slug = "setup-offline-download-service-aria2-ariang-filebrowser-on-centos7"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

## 安装Aria2

### 安装系统依赖和组件

```bash
yum update
yum -y groupinstall "Development Tools"
```

### 获取版本

自动获取

```bash
aria2_new_ver=$(wget --no-check-certificate -qO- https://api.github.com/repos/q3aql/aria2-static-builds/releases | grep -o '"tag_name": ".*"' |head -n 1| sed 's/"//g;s/v//g' | sed 's/tag_name: //g') && echo -e "${aria2_new_ver}"
```

执行上面的自动获取版本步骤后，显示版本号正常的话，下面的【手动获取版本】就不需要执行了。

手动获取版本

首先访问Github的[Releases页面](https://github.com/aria2/aria2/releases)查看版本号，例如 `1.34.0` ，然后我们执行如下代码后即可继续下面的【下载安装】步骤了

```bash
aria2_new_ver="1.34.0"
```

### 下载安装

要下载安装首先要判断你的VPS位数，运行下面的代码

```bash
uname -m
```

如果输出的是 `x86_64` ，则代表你的VPS系统是64位的，如果输出的是 `386`/`i368`/`686`/`i686` 这四个之一，则代表是32位的，根据你的VPS位数来选择下面的下载安装代码（不要选错）

```bash
# 显示 x86_64 的64位系统运行下面这两行命令下载aria2压缩包（不要选错，也不要重复运行32位的下载代码）。
wget -N --no-check-certificate "https://github.com/q3aql/aria2-static-builds/releases/download/v${aria2_new_ver}/aria2-${aria2_new_ver}-linux-gnu-64bit-build1.tar.bz2"
Aria2_Name="aria2-${aria2_new_ver}-linux-gnu-64bit-build1"

# 显示 386/i368/686/i686 这四个之一的32位系统运行下面这两行命令下载aria2压缩包（不要选错，也不要重复运行64位的下载代码）。
wget -N --no-check-certificate "https://github.com/q3aql/aria2-static-builds/releases/download/v${aria2_new_ver}/aria2-${aria2_new_ver}-linux-gnu-32bit-build1.tar.bz2"
Aria2_Name="aria2-${aria2_new_ver}-linux-gnu-32bit-build1"
```

如果下载过程中没有报错，那么接下来解压并开始安装aria2

```bash
# 解压压缩包
tar jxvf "${Aria2_Name}.tar.bz2"

# 为了方便好记，重命名一下解压后的目录
mv "${Aria2_Name}" "aria2"

# 进入解压后的目录
cd "aria2/"

# 运行这个命令才算开始安装Aria2自身
~~make install~~
# 最新的1.35.0此处命令变为
./configure
make
make install
# 赋予一下Aria2的运行权限
chmod +x aria2c
```

## 配置aria2

### 编辑配置文件

```bash
# 我们需要在 当前用户（ROOT）目录新建一个 存放配置文件的文件夹
mkdir "/root/.aria2"

#创建配置文件
vi /root/.aria2/aria2.conf
```

粘贴修改如下字段

```bash
## '#'开头为注释内容, 选项都有相应的注释说明, 根据需要修改 ##
## 被注释的选项填写的是默认值, 建议在需要修改时再取消注释  ##

## 文件保存相关 ##

# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=/root/Downloads
# 启用磁盘缓存, 0为禁用缓存, 需1.16以上版本, 默认:16M
#disk-cache=32M
# 文件预分配方式, 能有效降低磁盘碎片, 默认:prealloc
# 预分配所需时间: none < falloc ? trunc < prealloc
# falloc和trunc则需要文件系统和内核支持
# NTFS建议使用falloc, EXT3/4建议trunc, MAC 下需要注释此项
# file-allocation=none
# 断点续传
continue=true

## 下载连接相关 ##

# 最大同时下载任务数, 运行时可修改, 默认:5
max-concurrent-downloads=10
# 同一服务器连接数, 添加时可指定, 默认:1
max-connection-per-server=5
# 最小文件分片大小, 添加时可指定, 取值范围1M -1024M, 默认:20M
# 假定size=10M, 文件为20MiB 则使用两个来源下载; 文件为15MiB 则使用一个来源下载
min-split-size=10M
# 单个任务最大线程数, 添加时可指定, 默认:5
split=20
# 整体下载速度限制, 运行时可修改, 默认:0
#max-overall-download-limit=0
# 单个任务下载速度限制, 默认:0
#max-download-limit=0
# 整体上传速度限制, 运行时可修改, 默认:0
max-overall-upload-limit=1K
# 单个任务上传速度限制, 默认:0
max-upload-limit=1K
# 禁用IPv6, 默认:false
disable-ipv6=false

## 进度保存相关 ##

# 从会话文件中读取下载任务
input-file=/root/.aria2/aria2.session
# 在Aria2退出时保存`错误/未完成`的下载任务到会话文件
save-session=/root/.aria2/aria2.session
# 定时保存会话, 0为退出时才保存, 需1.16.1以上版本, 默认:0
#save-session-interval=60

## RPC相关设置 ##

# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
rpc-secret="此处自行设定密码"
# 是否启用 RPC 服务的 SSL/TLS 加密,
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
#rpc-secure=true
# 在 RPC 服务中启用 SSL/TLS 加密时的证书文件(.pem/.crt)
#rpc-certificate=/root/xxx.pem
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件(.key)
#rpc-private-key=/root/xxx.key

## BT/PT下载相关 ##

# 当下载的是一个种子(以.torrent结尾)时, 自动开始BT任务, 默认:true
follow-torrent=true
# BT监听端口, 当端口被屏蔽时使用, 默认:6881-6999
listen-port=51413
# 单个种子最大连接数, 默认:55（0表示不限制）
bt-max-peers=0
# 打开DHT功能, PT需要禁用, 默认:true
enable-dht=true
# 打开IPv6 DHT功能, PT需要禁用
enable-dht6=true
# DHT网络监听端口, 默认:6881-6999
#dht-listen-port=6881-6999
# 本地节点查找, PT需要禁用, 默认:false
bt-enable-lpd=true
# 种子交换, PT需要禁用, 默认:true
enable-peer-exchange=true
# 每个种子限速, 对少种的PT很有用, 默认:50K
#bt-request-peer-speed-limit=50K
# 客户端伪装, PT需要
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
# 当种子的分享率达到这个数时, 自动停止做种, 0为一直做种, 默认:1.0
seed-ratio=0.1
# 强制保存会话, 即使任务已经完成, 默认:false
# 较新的版本开启后会在任务完成后依然保留.aria2文件
#force-save=false
# BT校验相关, 默认:true
#bt-hash-check-seed=true
# 继续之前的BT任务时, 无需再次校验, 默认:false
bt-seed-unverified=true
# 保存磁力链接元数据为种子文件(.torrent文件), 默认:false
#bt-save-metadata=true
#最小做种时间
seed-time=0
#dht文件保存位置
dht-file-path=/root/.aria2/dht.dat
```

### 下载aria2的DHT文件

下载BT的话，DHT会很影响速度的，因为aria2默认安装没有DHT文件，然后会在每次下载BT的时候收集DHT信息来新建DHT文件，这会导致一开始使用aria2下载BT速度很慢，下载一个现成的DHT文件能缓解这个情况，当然根据不同资源、不同热度，速度肯定有影响

```bash
wget --no-check-certificate -O "/root/.aria2/dht.dat" "https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/other/Aria2/dht.dat"
```

### 新建一个空文件用于存放下载任务

这样重启aria2也不会丢失任务了

```bash
echo '' > /root/.aria2/aria2.session
```

### 加载配置文件启动

```bash
aria2c --conf-path=/root/.aria2/aria2.conf -D
```

### 将aria2设置为系统服务

```bash
yum install -y psmisc
vi /etc/init.d/aria2c
```

在其中粘贴以下内容

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides: aria2
# Required-Start: $remote_fs $network
# Required-Stop: $remote_fs $network
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Aria2 Downloader
### END INIT INFO
case "$1" in
start)
  echo -n "Starting aria2c"
  /usr/bin/aria2c --conf-path=/root/.aria2/aria2.conf -D
  ;;
stop)
  echo -n "Shutting down aria2c "
  killall aria2c
  ;;
restart)
  killall aria2c
  /usr/bin/aria2c --conf-path=/root/.aria2/aria2.conf -D
  ;;
esac
exit
```

赋予可执行权限

```bash
chmod 7777 /etc/init.d/aria2c
```

控制命令

```bash
#重新加载
systemctl daemon-reload
#启动
systemctl start aria2c
#停止
systemctl stop aria2c
#重启
systemctl restart aria2c
#开启自启动
systemctl enable aria2c
```

### 防火墙放行RPC，BT端口

```bash
firewall-cmd --zone=public --add-port=6800/tcp --permanent
firewall-cmd --zone=public --add-port=6881-6999/tcp --permanent
firewall-cmd --zone=public --add-port=6881-6999/udp --permanent
firewall-cmd --reload
```

### 自动更新bt-tracker脚本（可选）

```bash
vi /root/trackers-list-aria2.sh
```

在其中粘贴

```bash
#!/bin/bash
killall aria2c
list=`wget -qO- https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_all.txt|awk NF|sed ":a;N;s/\n/,/g;ta"`
if [ -z "`grep "bt-tracker" /root/.aria2/aria2.conf`" ]; then
    sed -i '$a bt-tracker='${list} /root/.aria2/aria2.conf
    echo add......
else
    sed -i "s@bt-tracker.*@bt-tracker=$list@g" /root/.aria2/aria2.conf
    echo update......
fi
```

赋予可执行权限

```bash
chmod +x /root/trackers-list-aria2.sh
```

执行脚本

```bash
/root/trackers-list-aria2.sh
```

更新过程会先关闭aria2c进程，更新完成再需要手动开启aria2c

```bash
systemctl start aria2c
```

使用crontab任务计划程序，实现自动更新

```bash
crontab -e
```

粘贴

```bash
#每隔1440分钟（即24小时）更新一次tracker列表
#每隔5分钟启动一次aria2，防止aria2崩溃
*/1440 * * * * /root/trackers-list-aria2.sh
*/5 * * * * /usr/bin/aria2c --conf-path=/root/.aria2/aria2.conf -D
```

至此，aria2安装及配置完成。

## 安装AriaNG

### 先安装一个Nginx

```bash
vi /etc/yum.repos.d/nginx.repo
```

写入

```bash
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```

用yum安装

```bash
yum -y install nginx
```

进入到Nginx的默认站点目录

```bash
cd /usr/share/nginx/html
```

### 下载AriaNG并解压

```bash
wget https://github.com/mayswind/AriaNg/releases/download/0.4.0/aria-ng-0.4.0.zip
unzip aria-ng-0.4.0.zip
```

有一个同名文件直接按y覆盖就行。

启动Nginx并添加至开机启动

```bash
systemctl enable nginx
systemctl start nginx
```

防火墙放行80，443端口

```bash
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --zone=public --add-port=443/tcp --permanent
firewall-cmd --reload
```

此时打开VPS公网IP即可访问AriaNG。

## 安装Filebrowser

下载并解压Filebrowser

```bash
cd /root
wget https://github.com/filebrowser/filebrowser/releases/download/v1.7.0/linux-amd64-filebrowser.tar.gz
tar -zxvf linux-amd64-filebrowser.tar.gz
```

写个服务，让Filebrowser开机启动。

先复制可执行文件到/usr/bin

```bash
cp /root/filebrowser /usr/bin/filebrowser
```

新建一个服务文件

```bash
vi /usr/lib/systemd/system/filebrowser.service
```

写入

```bash
[Unit]
Description=filebrowser

[Service]
User=root
ExecStart=/usr/bin/filebrowser --port 23333
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

启动Filebrowser并设置为开机启动

```bash
systemctl enable filebrowser
systemctl start filebrowser
```

防火墙放行端口23333

```bash
firewall-cmd --zone=public --add-port=23333/tcp --permanent
firewall-cmd --zone=public --add-port=23333/udp --permanent
firewall-cmd --reload
```

至此，打开VPS公网IP+端口23333即可访问Filebrowser，默认管理员账号密码均为admin，默认目录范围为根目录，均可自行修改。

---

*参考来源：*

1. [Aria2离线下载后自动将文件上传到GoogleDrive--荒岛](https://lala.im/2982.html)
2. [BT/种子/磁力链接/HTTP/FTP 离线下载工具 —— Aria2 新 手动安装教程--逗比](https://doub.io/wlzy-37/)
3. [Centos离线下载Aria2 AriaNG bt-tracker自动更新--augustdoit](https://blog.augustdoit.bid/aria2/)