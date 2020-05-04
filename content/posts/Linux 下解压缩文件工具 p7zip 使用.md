+++
title = "Linux 下解压缩文件工具 p7zip 使用"
description = " "
date = "2019-07-02T12:37:35+08:00"
categories = ["奇技淫巧"]
tags = ["p7zip","解压"]
slug = "usage-of-p7zip"
comments = true
draft = false
+++

**系统环境：** CentOS 7 x64

## 安装 p7zip

默认 CentOS 7 没有安装 p7zip ，默认源里面也没有这个包，需要先安装 epel 源：

```bash
yum -y install epel-release
```

然后安装 p7zip ：

```bash
yum install p7zip p7zip-plugins
```

## 新建压缩包

使用 `a` 命令。

p7zip 的命令参数不带 `-` ，比如新建一个 7z 格式的压缩包：

```bash
7z a files.7z file1.txt file2.txt file3.txt
```

新建一个带密码的压缩包：

```bash
# 假设密码为 password
7z a files.7z file1.txt file2.txt -ppassword
# 还可以带上标志 -mhe = on 来隐藏压缩包内的结构
7z a files.7z file1.txt file2.txt -ppassword -mhe = on
```

## 查看压缩包中的内容

使用 `l` 命令：

```bash
7z l files.7z
```

## 解压压缩包

使用 `e` 或 `x` 命令：

```bash
# 该命令会无视压缩包内文件结构，将所有文件解压至当前目录下
7z e files.7z
```

或

```bash
# 该命令会保留压缩包内目录结构
7z x files.7z
```

推荐使用 `x` 命令来解压。

如果要指定输出路径，可以使用`-o`参数，后跟绝对路径（路径和 `-o` 参数之间无空格）：

```bash
# 将压缩包解压至 /root/test/ 目录下
7z x files.7z -o/root/test/
```

解压带密码的压缩包，可以使用`-p`参数（密码和 `-p` 参数之间无空格）：

```bash
# 假设密码为 password
7z x files.7z -ppassword
```

## 7z，7za 与 7zr 的区别

* 7z 使用插件处理格式文件；
* 7za 是独立可执行的，可以不需要其它任何插件的处理较少的格式；
* 7zr 是独立可执行的，可以不需要其它任何插件的处理 7z 格式的文件。

---

*参考链接：*

1. [p7zip (简体中文) - ArchWiki](https://wiki.archlinux.org/index.php/P7zip_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))