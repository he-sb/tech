+++
title = "Linux 下对 ext4 分区在线无损扩容"
description = " "
date = "2023-10-30T22:49:51+08:00"
toc = true
categories = ["Linux"]
tags = ["硬盘","linux","ext4"]
slug = "online-lossless-expansion-of-ext4-partition"
draft = false
+++

PVE 上的用来做监控的一台虚拟机，因为安装系统时经验不足，预分配的硬盘容量很快就快用满了。这台虚拟机装的比较早，当时为了图省事，图快，直接挂载了 ISO 镜像手动引导安装，而没有用 Cloud-Init 来引导，这就导致它作为一个私有云环境下的虚拟机，硬盘容量却不能自动扩展，非常地难受。。这篇博客主要记录一下手动在线扩容 ext4 分区的过程，填一下因为装系统时图省事，没有研究最佳实践而遗留下来的坑。

## 0. 安装必需的工具

```shell
sudo apt update && sudo apt intall -y parted resize2fs
```

- `parted` 是硬盘分区工具，这里用来查看磁盘分区情况，按需删除不需要的分区，以及扩展分区容量
- `resize2fs` 用来扩展文件系统容量，以使操作系统可以识别并使用扩展后的硬盘分区
- 俺这台机器系统是 Debian 11，如果是 CentOS 7 的话应该用 `yum install` 来安装

## 1. 开始扩容

查看当前磁盘的分区情况：

```shell
# 进入 parted 工具
$ sudo parted
GNU Parted 3.4
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
# 查看磁盘信息
(parted) print /dev/sda
```

俺这里硬盘容量是 20 GB，扩容前是 10 GB，本文开始前在 PVE 后台给该虚拟机对应的硬盘增加了 10 GB，因此目前的分区使用情况是：
- `/dev/sda1` - 9712 MB
  - 根分区，文件系统是 ext4
- `/dev/sda2` - 1 KB
  - 扩展分区
- `/dev/sda5` - 1022 MB
  - swap 分区

其中需要扩容的是 `/dev/sda1` ，但是因为其后有非空闲的分区（扩展分区和 swap 分区），不能直接扩展容量，所以下面分三步走：
1. 删除扩展分区和 swap 分区
2. 给根分区扩容
3. 将 swap 功能添加回来

如果你只有 `sda1` 一个分区，说明没有扩展分区和单独的 swap 分区，那么可以直接开始扩容，跳过其他部分。

### 删除扩展分区和 swap 分区

**如果磁盘上待扩容的分区后方没有其他分区，可以跳过本节，直接开始扩容。**

首先关闭 swap：

```shell
$ sudo swapoff -a
```

编辑 `/etc/fstab` 和 `/etc/initramfs-tools/conf.d/resume` 这两个文件，删除 swap 所在的行，然后更新引导配置：

```shell
$ sudo update-initramfs -u
$ sudo update-grub
```

此时可以删除分区了：

```shell
# 进入 parted 工具
$ sudo parted /dev/sda
# 查看分区编号
(parted) print
# 删除 swap 分区
(parted) rm 5
# 再次查看分区
(parted) print
# 删除扩展分区
(parted) rm 2
# 保存更改并退出
(parted) quit
```

### 给根分区扩容

```shell
# 进入 parted 工具
$ sudo parted /dev/sda
# 查看磁盘信息
(parted) print
# 此时应该只有一个分区了，直接扩展这个分区
(parted)resizepart 1
Warning: Partition /dev/sda1 is being used. Are you sure you want to continue?
# 直接输入 yes 确认
Yes/No? yes
# 输入新的结束点
# 这里输入的数值，就是上方输出中 Disk: 后方的数值
End?  [9713MB]? 21.5GB
# 扩展完成之后退出 parted
(parted) quit
```

此时分区容量已经扩展完成了，但是文件系统还未识别扩展的容量，所以扩展的容量还没法使用。下面扩展一下文件系统 ：

```shell
$ sudo resize2fs /dev/sda1
```

此时不出意外的话应该扩容完成了，可以使用 `df -h` 来查看容量。


### 将 swap 功能添加回来

**如果不需要 swap，这部分可以直接跳过。**

首先创建一个 swap 文件（俺这里创建一个 1 GB 大小的）：

```shell
$ sudo fallocate -l 1G /swapfile
```

修改文件权限：

```shell
$ sudo chmod 600 /swapfile
```

激活 swap 文件：

```shell
$ sudo mkswap /swapfile
$ sudo swapon /swapfile
```

使用 `sudo swapon -s` 或 `free -m` 命令，查看 swap 功能是否已激活。

开机自动挂载 swap：

```shell
$ sudo bash -c 'echo "/swapfile swap swap defaults 0 0" >> /etc/fstab'
```

---

*参考链接：*
1. [linux下无损扩容分区方法 - 知乎](https://zhuanlan.zhihu.com/p/510655363)
2. [Debian/Ubuntu下如何安全的删除默认swap分区，并移除swap | GHL's Notes](https://www.ghl.name/archives/debian-ubuntu-delete-swap-partition-safely.html)
3. [Debian / Ubuntu 手工添加 Swap 分区 - 烧饼博客](https://u.sb/debian-swap/)
