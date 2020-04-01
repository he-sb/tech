+++
title = "使用 Rclone 挂载 Google Drive"
description = " "
date = "2018-12-04T12:37:35+08:00"
categories = ["奇技淫巧"]
tags = ["rclone","挂载","gd"]
slug = "mount-google-drive-as-local-disk-with-rclone"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

搞了个无限容量的Google Drive，使用Rclone挂载到小鸡上进行扩容。

## 安装依赖

```bash
yum install -y fuse fuse-devel
```

## 安装Rclone

```bash
curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64
sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone
sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb
```

## 新建Rclone配置文件

```bash
rclone config
```

因为刚刚安装，还没开始配置，所以输出会是这样

```bash
2018/12/04 15:14:03 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n    #此处输入n表示新建配置
name> he-sb    #此处输入配置文件名，自行设置，后面挂载时会用到此名字
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / A stackable unification remote, which can appear to merge the contents of several remotes
   \ "union"
 2 / Alias for a existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers (AWS, Ceph, Dreamhost, IBM COS, Minio)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Dropbox
   \ "dropbox"
 9 / Encrypt/Decrypt a remote
   \ "crypt"
10 / FTP Connection
   \ "ftp"
11 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
12 / Google Drive
   \ "drive"
13 / Hubic
   \ "hubic"
14 / JottaCloud
   \ "jottacloud"
15 / Local Disk
   \ "local"
16 / Mega
   \ "mega"
17 / Microsoft Azure Blob Storage
   \ "azureblob"
18 / Microsoft OneDrive
   \ "onedrive"
19 / OpenDrive
   \ "opendrive"
20 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
21 / Pcloud
   \ "pcloud"
22 / QingCloud Object Storage
   \ "qingstor"
23 / SSH/SFTP Connection
   \ "sftp"
24 / Webdav
   \ "webdav"
25 / Yandex Disk
   \ "yandex"
26 / http Connection
   \ "http"
Storage> 12    #此处输入12，表示将要挂载的网盘类型是Google Drive，其他版本Rclone可能对应数字不同，自行查找
** See help for drive backend at: https://rclone.org/drive/ **

Google Application Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>    #此处留空即可，直接回车
Google Application Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>    #仍然留空，直接回车
Scope that rclone should use when requesting access from drive.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1    #此处输入1，表示使Rclone具有对网盘的完全访问权限
ID of the root folder
Leave blank normally.
Fill in to access "Computers" folders. (see docs).
Enter a string value. Press Enter for the default ("").
root_folder_id>    #此处留空即可，直接回车
Service Account Credentials JSON file path
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file>    #仍然留空，直接回车
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n    #输入n，表示不修改高级配置
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine or Y didn't work
y) Yes
n) No
y/n> n    #输入n，表示不使用自动配置
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=************    #复制此处的链接至浏览器打开，然后登陆想要用来挂载的Google账号，将返回的确认代码复制
Log in and authorize rclone for access
Enter verification code> **********************************    #此处粘贴刚才复制的确认代码
Configure this as a team drive?
y) Yes
n) No
y/n> y    #根据自己是否需要配置为团队盘来确定
Fetching team drive list...
No team drives found in your account--------------------
[he-sb]
type = drive
scope = drive
token = {"access_token":"***","token_type":"Bearer","refresh_token":"***","expiry":"2018-12-04T16:15:47.397176466+08:00"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y    #检查一下配置信息，没毛病的话输入y回车即可
Current remotes:

Name                 Type
====                 ====
he-sb                drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q    #至此配置文件已配置完成，输入q退出配置
```

## 挂载为系统磁盘

先新建本地文件夹，用于映射想要挂载的网盘（此处路径为/root/he-sb/）

```bash
mkdir -p /root/he-sb
```

挂载磁盘（其中 `he-sb` 处为之前新建配置时起的配置文件名，冒号后面为想要挂载的网盘目录，留空表示挂载根目录，`/root/he-sb` 即刚刚新建的用于映射的本地文件夹）

```bash
rclone mount he-sb: /root/he-sb --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --vfs-cache-mode writes --attr-timeout 0s --default-permissions --buffer-size 32M --dir-cache-time 12h --vfs-read-chunk-size 64M --vfs-read-chunk-size-limit 1G &
```

挂载完成后查看一下磁盘情况

```bash
df -h
```

如返回类似如下信息说明挂载成功

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        25G  6.8G   19G  28% /
devtmpfs        474M     0  474M   0% /dev
tmpfs           496M   28K  496M   1% /dev/shm
tmpfs           496M   57M  440M  12% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
tmpfs           100M     0  100M   0% /run/user/0
he-sb:          1.0P     0  1.0P   0% /root/he-sb    #无限容量在Linux下会显示为1PB
```

卸载磁盘使用以下命令

```bash
fusermount -qzu /root/he-sb
```

## 添加开机自动挂载（可选）

编辑脚本

```bash
vi rcloned
```

粘贴以下内容并保存

```bash
#!/bin/bash

PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
NAME_BIN="rclone"
###BEGIN INIT INFO
#Provides:          rclone
#Required-Start:    $all
#Required-Stop:     $all
#Default-Start:     2 3 4 5
#Default-Stop:      0 1 6
#Short-Description: Start rclone at boot time
#Description:       Enable rclone by daemon.
###END INIT INFO

NAME="he-sb" #Rclone 配置名
REMOTE='' #要挂载的网盘中的文件夹，留空即挂载根目录
LOCAL='/root/he-sb' #本地映射的挂载文件夹路径

Green_font_prefix="\033[32m" && Red_font_prefix="\033[31m" && Green_background_prefix="\033[42;37m" && Red_background_prefix="\033[41;37m" && Font_color_suffix="\033[0m"
Info="${Green_font_prefix}[信息]${Font_color_suffix}"
Error="${Red_font_prefix}[错误]${Font_color_suffix}"
RETVAL=0

check_running(){
    PID="$(ps -C $NAME_BIN -o pid= |head -n1 |grep -o '[0-9]\{1,\}')"
    if [[ ! -z ${PID} ]]; then
    return 0
    else
    return 1
    fi
}
do_start(){
    check_running
    if [[ $? -eq 0 ]]; then
        echo -e "${Info} $NAME_BIN (PID ${PID}) 正在运行..." && exit 0
    else
        fusermount -zuq $LOCAL >/dev/null 2>&1
        mkdir -p $LOCAL
        sudo /usr/bin/rclone mount $NAME:$REMOTE $LOCAL --copy-links --no-gzip-encoding --no-check-certificate --allow-other --allow-non-empty --vfs-cache-mode writes --attr-timeout 0s --default-permissions --buffer-size 32M --dir-cache-time 12h --vfs-read-chunk-size 64M --vfs-read-chunk-size-limit 1G & >/dev/null 2>&1 &
        sleep 2s
        check_running
        if [[ $? -eq 0 ]]; then
            echo -e "${Info} $NAME_BIN 启动成功 !"
        else
            echo -e "${Error} $NAME_BIN 启动失败 !"
        fi
    fi
}
do_stop(){
    check_running
    if [[ $? -eq 0 ]]; then
        kill -9 ${PID}
        RETVAL=$?
        if [[ $RETVAL -eq 0 ]]; then
            echo -e "${Info} $NAME_BIN 停止成功 !"
        else
            echo -e "${Error} $NAME_BIN 停止失败 !"
        fi
    else
        echo -e "${Info} $NAME_BIN 未运行"
        RETVAL=1
    fi
    fusermount -zuq $LOCAL >/dev/null 2>&1
}
do_status(){
    check_running
    if [[ $? -eq 0 ]]; then
        echo -e "${Info} $NAME_BIN (PID $(echo ${PID})) 正在运行..."
    else
        echo -e "${Info} $NAME_BIN 未运行 !"
        RETVAL=1
    fi
}
do_restart(){
    do_stop
    do_start
}
case "$1" in
    start|stop|restart|status)
    do_$1
    ;;
    *)
    echo "使用方法: $0 { start | stop | restart | status }"
    RETVAL=1
    ;;
esac
exit $RETVAL
```

设置自启动

```bash
mv rcloned /etc/init.d/rcloned
chmod +x /etc/init.d/rcloned
vi /etc/rc.d/rc.local
```

在末尾加入

```bash
bash /etc/init.d/rcloned start
```

最后修改一下权限

```bash
chmod +x /etc/rc.d/rc.local
```

至此，全部配置完成。

---

*参考来源：*

1. [在Debian/Ubuntu上使用rclone挂载Google Drive网盘 - Rat's Blog](https://www.moerats.com/archives/481/)
2. [Rclone 挂载 Google Drive实现个人离线播放系统 | 万世残天](http://ctnmb.com/466.html)
3. [Rclone官网安装说明](https://rclone.org/install/)