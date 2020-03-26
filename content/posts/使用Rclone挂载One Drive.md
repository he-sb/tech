+++
title = "使用 Rclone 挂载 One Drive"
description = " "
date = "2018-12-06T14:37:35+08:00"
categories = ["奇技淫巧"]
tags = ["rclone","挂载","od"]
slug = "mount-microsoft-one-drive-as-local-disk-with-rclone"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

Rclone挂载One Drive相比Google Drive过程有些许区别，记录一下。

## 安装依赖

## 安装Rclone

以上两部分与挂载GD时相同，可参考[这篇文章](/posts/mount-google-drive-as-local-disk-with-rclone)。

## 新建Rclone配置文件

```bash
rclone config
```

因为之前已经挂载了GD，所以此时输出是这样

```bash
Current remotes:

Name                 Type
====                 ====
he-sb                drive    #此处显示之前挂载的配置

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> n    #输入n新建配置
name> j0ker    #输入配置名称
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
Storage> 18    #本版本Rclone中18代表One Drive，其他版本可能对应数字不同，自行查找
** See help for onedrive backend at: https://rclone.org/onedrive/ **

Microsoft App Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>    #不需设置，留空，直接回车即可
Microsoft App Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>    #同样不需设置，直接回车即可
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n    #不修改高级配置
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n    #不使用自动配置
For this to work, you will need rclone available on a machine that has a web browser available.
Execute the following on your machine:
        rclone authorize "onedrive"
Then paste the result below:
result>
```
在此之前步骤与挂载GD大同小异，到此处开始有很大不同。

**不同之处在于，远程服务器一般是没有图形界面的，但Rclone绑定OD时需要系统内置浏览器才能正常完成授权，变通的办法就是，先在本地的Windows电脑上使用Rclone来获取授权后的token，再将其复制到远程服务器上。**

先到Rclone官网下载Windows版本的Rclone：[https://rclone.org/downloads/](https://rclone.org/downloads/)。

下载后将压缩包解压，然后将 `rclone.exe` 文件解压至 `C:\windows\system32\` ，然后打开命令行，输入 `rclone authorize "onedrive"`，系统默认浏览器会自动打开一个页面进行授权，授权成功后会返回一个token，根据提示将相应需要复制的段落复制出来，后面会用到。

复制出来后即可将本地的命令行关掉，继续配置VPS。

在 `result>` 后面将刚才复制的token粘贴上，然后回车继续配置

```bash
2018/12/06 14:39:48 ERROR : Failed to save new token in config file: section 'j0ker' not found
Choose a number from below, or type in an existing value
 1 / OneDrive Personal or Business
   \ "onedrive"
 2 / Root Sharepoint site
   \ "sharepoint"
 3 / Type in driveID
   \ "driveid"
 4 / Type in SiteID
   \ "siteid"
 5 / Search a Sharepoint site
   \ "search"
Your choice> 1    #因为我这个One Drive账号是商业版，所以选了1，根据实际情况来选
Found 1 drives, please select the one you want to use:
0: OneDrive (business) id=********    #手动打码嘿嘿嘿
Chose drive to use:> 0    #输入0就好了
Found drive 'root' of type 'business', URL: https://**********    #依然是手动打码
Is that okay?
y) Yes
n) No
y/n> y    #输y确认

[j0ker]
type = onedrive
token = ******    #保护隐私，token内容已被我手动打码了嘿嘿嘿
drive_id = b!vDx1HhJobUaNvJPugzWSWtOoZz2uN9hFhCdhE593vj7CcCS_6TKbR5hW7vI4rjY1
drive_type = business

y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y    #配置信息没问题之后输入y确认
Current remotes:

Name                 Type
====                 ====
he-sb                drive
j0ker                onedrive    #可以看到OD已经成功配置了

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q    #退出配置
```

## 挂载为系统磁盘

本部分依然参考挂载GD的那篇即可。

## 添加开机自动挂载

懒得写了。。

---

*参考来源：*

1. [CentOS使用Rclone挂载OneDrive - 小z博客](https://www.xiaoz.me/archives/10397)