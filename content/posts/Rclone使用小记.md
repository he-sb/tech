+++
title = "Rclone 使用小记"
description = " "
date = "2018-12-06T19:37:35+08:00"
categories = ["奇技淫巧"]
tags = ["rclone"]
slug = "usage-of-rclone"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

前几天用 Rclone 挂载好了 Google Drive（[使用Rclone挂载Google Drive](/posts/mount-google-drive-as-local-disk-with-rclone)），使用过程中遇到一点问题，记录一下。

**0.**

刚开始时向挂载的 GD 中上传文件直接使用 `mv /root/Downloads/* /mnt/he-sb` 命令将本地文件移动至挂载后的GD目录，小文件时没问题，但大文件有时会莫名出错，经大佬指点，改为使用 `rclone move source:path dest:path` 命令。

**1.**

使用 `rclone move /root/2mv/*.zip he-sb:OldDriver` 命令时报错：

```bash
Usage:
  rclone move source:path dest:path [flags]

Flags:
      --delete-empty-src-dirs   Delete empty source dirs after move
  -h, --help                    help for move

Use "rclone [command] --help" for more information about a command.
Use "rclone help flags" for to see the global flags.
Use "rclone help backends" for a list of supported services.
Command move needs 2 arguments maximum
```

~~搜索了一番发现 `rclone move` 命令的路径貌似必须是文件夹，不能是文件，于是修改为 `rclone move /root/2mv he-sb:OldDriver` 问题解决。~~

问题在于此命令目的路径也要是文件而非文件夹，因此使用 `rclone move /root/2mv he-sb:OldDriver` 或 `rclone move /root/2mv/*.zip he-sb:OldDriver/*.zip` 均可。

**2.**

后来在移动完一个含子目录的文件夹后发现虽然该文件夹及子文件夹内所有内容都移动完毕了，但剩下空的文件夹都没有删，又搜索了一下发现可以在 `rclone move` 命令后加参数 `--delete-empty-src-dirs` 来在移动完成后将源路径下的空文件夹删掉（其实上一条报错信息内也写了，怪自己蠢没注意……）。

**3.**

`-v` 参数用于输出 Rclone 当前操作的进度。

**4.**

`--transfers [int]` 参数修改线程数，即同时开始传输的文件数，避免占用资源（主要是内存）过多，该参数默认值为 `4` 。

**5.**

`--bwlimit` 参数可以限制同步时占用的带宽，可以用此命令来避免触发 GD 的 API 每天 750G 的流量限制，此时使用 `--bwlimit 8M` 即可。

**6.**

`--disable copy` 参数禁用 server side copy ，同样用于突破单日 750G 的 API 流量限制。

**7.**

`-P`=`--progress` 显示实时传输进度（与 `-v` 的区别在于进度是【实时】更新的）。

**8.**

`--ignore-existing` 忽略已存在的文件。

**9.**

`rclone size [DriveName]:[Folder]` 获取指定路径下文件内容的总大小。

**10.**

`rclone sync source:path dest:path` 命令用来同步数据，同步数据时，可能会删除目的地址的数据，可先使用 `--dry-run` 标志来检查要复制、删除的数据。同步数据出错时，不会删除任何目的地址的数据。此命令同步的始终是 `path` 目录下的数据，而不是 `path` 目录（空目录将不会被同步）。

**11.**

Rclone 配置文件路径为 `/root/.config/rclone/rlcone.conf` ，在 VPS 重装系统或迁移前可以备份下来免去重新配置的麻烦。

**12.**

`--drive-server-side-across-configs` 参数可在团队盘复制中使用 server side copy ，不占用服务器流量和带宽。