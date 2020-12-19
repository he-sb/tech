+++
title = "BuyVM 挂载块存储"
description = " "
date = "2020-12-19T20:20:09+08:00"
categories = ["Linux"]
tags = ["挂载"]
slug = "mounting-block-storage-on-buyvm-vps"
draft = false
+++

**系统环境：** CentOS 7 x64

今天给 BuyVM 小鸡添加了块存储块，价格还是非常划算的，俺买的是 256 GB 的，月付 1.25 刀，年付 14.25，考虑到 BuyVM 标配的 G 口带宽，下载点东西方便很多。

下面 ~~水~~ 记录下添加过程。

第一步当然是购买，可以戳这里快捷购买：[https://my.frantech.ca/cart.php?gid=42](https://my.frantech.ca/cart.php?gid=42)（无 aff），也可以点击自己 BuyVM 页面顶端的 `Services` -> `View Available Addons`，然后在左侧列表中点击带有 `Block Storage Slabs` 的项目，就是块存储了。

> 注意：
> 
> 购买块存储的区域一定要和小鸡所在的区域一致，否则没法挂载（比如俺的小鸡是拉斯维加斯的，购买块存储的时候必须要选择拉斯维加斯的块存储）。
> 
> 截止今天，只有【纽约】和【拉斯维加斯】这两个区域可以添加块存储。

购买完毕就可以挂载了。登录 VPS 的控制面板，可以点击这里快速进入：[https://manage.buyvm.net/](https://manage.buyvm.net/) ，或者点击自己 BuyVM 页面顶端的 `Stallion` 按钮进入，登录后点击页面顶部的 `Storage Volumes`，然后应该能看到刚刚购买的存储块了，点击右侧的小齿轮 -> `Attach To Virtual Server`，然后选择需要扩容的小鸡（如果你有不止一个实例的话）。

接下来 SSH 连接到 VPS，查看硬盘是否添加完成：

```bash
fdisk -l
```

输出如下：

```bash
Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00095367

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    39843455    19920704   83  Linux
/dev/vda2        39843456    41940607     1048576   82  Linux swap / Solaris

Disk /dev/sda: 274.9 GB, 274877906944 bytes, 536870912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

可以看到多了一块没有分区的磁盘设备（`/dev/sda`），先创建分区表：

```bash
fdisk /dev/sda
```

此时会进入一个交互式的初始化过程，如下：

```bash
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x477d9fec.

Command (m for help): n  # 此处输入 n，新建分区
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p  # 此处输入 p，新建为主分区
Partition number (1-4, default 1):  # 留空，直接回车
First sector (2048-536870911, default 2048):  # 留空，直接回车
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-536870911, default 536870911):  # 留空，直接回车
Using default value 536870911
Partition 1 of type Linux and of size 256 GiB is set

Command (m for help): w  # 输入 w，保存修改
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

再执行一次 `fdisk -l` 看看分区：

```bash
Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00095367

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    39843455    19920704   83  Linux
/dev/vda2        39843456    41940607     1048576   82  Linux swap / Solaris

Disk /dev/sda: 274.9 GB, 274877906944 bytes, 536870912 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x477d9fec

   Device Boot      Start         End      Blocks   Id  System
/dev/sda1            2048   536870911   268434432   83  Linux
```

可以看到存储块上多了个分区（`/dev/sda1`）。

上面的过程是将整块存储块划分为了一个分区，如果你有特殊需求，比如多个分区之类的，可以自行研究下 `fdisk` 命令，本文就不深入了。

下面对刚建立的分区进行格式化（制作文件系统），俺这里格式化为 ext4：

```bash
mkfs.ext4 /dev/sda1
```

不出意外的话输出应该类似下面这样：

```bash
mke2fs 1.42.9 (28-Dec-2013)
Discarding device blocks: done
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
16777216 inodes, 67108608 blocks
3355430 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2214592512
2048 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

最后创建挂载点并挂载分区：

```bash
mkdir -p /mnt/storage256
mount /dev/sda1 /mnt/storage256
```

至此，访问 `/mnt/storage256` 这个路径，就可以使用这个新添加的存储块了。

最后的最后，为了不用每次重启完都要手动执行挂载命令，设置一下开机自动挂载：

```bash
vi /etc/fstab
```

在最后添加一行：

```bash
/dev/sda1 /mnt/storage256 ext4 defaults 0 0
```

---

_参考链接：_

- [要恰饭的嘛：buyvm测评/块存储挂载-荒岛](https://lala.im/7192.html)