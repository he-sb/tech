+++
title = "FFmpeg 操作合集"
description = " "
date = "2026-01-14T01:43:00+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["ffmpeg"]
slug = "usage-of-ffmpeg"
draft = false
+++

[FFmpeg](https://www.ffmpeg.org/) 是一个非常强大的多媒体编解码器，本文记录一下使用和学习过程中的一些基本操作和笔记，文末整理了俺保存的一些学习资源。

### 00. 安装

#### Linux 系统（以 Debian 12 为例）

首先不建议使用系统的包管理器直接安装，比如 `sudo apt install -y ffmpeg`, 这样安装的 FFmpeg 版本一般都很古早，遇到现代一点的编码格式比如 `av1` `vp9` 这种很容易报错（如果你是 ~~高贵的~~ Arch 系发行版用户当俺没说……），建议是手动安装预编译好的最新版本：

```shell
# 下载预编译好的二进制可执行文件
curl -o ffmpeg-latest.tar.xz -fsSL https://github.com/yt-dlp/FFmpeg-Builds/releases/download/autobuild-2026-01-13-14-23/ffmpeg-N-122452-gf897bcd122-linux64-gpl.tar.xz
# 解压到当前目录下的 ffmpeg-latest 文件夹内
mkdir -p ./ffmpeg-latest && tar -xvf ffmpeg-latest.tar.xz -C ./ffmpeg-latest --strip-components=1
# 将可执行文件复制到系统内
sudo mv ./ffmpeg-latest/bin/ffmpeg /usr/local/bin/
sudo mv ./ffmpeg-latest/bin/ffprobe /usr/local/bin/
# 删除下载的文件和解压的文件夹
rm -rf ./ffmpeg-latest ./ffmpeg-latest.tar.xz
# 验证版本
ffmpeg -version
```

不出意外的话应该能看到输出了最新的版本号。如果之前已经通过包管理器安装了 FFmpeg，导致此时输出的版本号还是旧的，直接卸载旧的就行：

```shell
sudo apt purge ffmpeg && sudo apt autoremove
```

后续如果要更新 FFmpeg 版本，将上面下载预编译文件的链接替换为最新的即可。

#### Windows 系统

这部分内容参考 [在线视频下载神器 You-Get 的安装及使用 | HE-SB-技术栈](/posts/setup-and-useing-guide-for-you-get/#2-再安装-ffmpeg) 的【安装 ffmpeg】部分即可，不再赘述。

### 01. 合并多个视频文件

```shell
ffmpeg -f concat -safe 0 -i 'list.txt' -c copy 'full.flv'
```

选项释义：

- `-f concat`: 指定输入格式为 `concat`，即告诉 FFmpeg 程序，后面的 `list.txt` 不是视频文件，而是一个列表文件，需要按照这个文件来连续读取多个视频文件。
- `-safe 0`: 权限设置。默认情况下，为了安全，FFmpeg 不允许读取包含相对路径或特殊字符的文件。设置为 `0` 表示允许读取任意路径的文件。
- `-i 'list.txt'`: 输入文件。这里的输入不是一个视频文件，而是一个包含视频文件路径列表的文本文件。
- `-c copy`: 流拷贝模式。直接将视频/音频流从原文件拷贝到目标文件，不进行重新编码。速度极快（只取决于磁盘 I/O 速度）且质量无损。
- `'full.flv'`: 输出文件。合并后的最终文件名。

其中，`list.txt` 文件的内容格式是每行以 `file ` 开头，后面是文件路径（如果只有文件名，没有路径，那么就只会在当前路径查找，没有就会报错)，内容类似这样：

```shell
file 'test-1.flv'
file 'test-2.flv'
```

这种方式合并视频，速度非常快（因为没有重新编解码），但是有严格的前提，就是 `list.txt` 内的所有待合并的视频文件，编码参数必须一致，包括分辨率、帧率、编码格式和音频参数等。如果不一致，合并后的视频文件会有问题（画面卡顿或无法播放），此时只能去掉 `-c copy` 参数，重新编码。

### 02. 分割视频

```shell
# 将一个长视频从 06:00:00 处分割为两个文件
ffmpeg -i 'full.flv' -t 06:00:00 -c copy -map 0 '1.flv' -ss 06:00:00 -c copy -map 0 '2.flv'
```

选项释义：

- `-i 'full.flv'`: 指定输入源文件。
- `-t 06:00:00`: 限制输出的时长。这里表示截取从 `00:00:00` 到 `06:00:00` 的内容。
- `-c copy`: 流拷贝模式。
- `-map 0`: 强制映射输入文件（索引号为 0）中的所有流（包括所有视频轨、音频轨、字幕轨等）到输出文件。
- `'1.flv'`: 第一个输出文件的文件名。
- `-ss 06:00:00`: 设定第二个文件的起始时间点。这里表示从 `06:00:00` 的位置开始。
- `'2.flv'`: 第二个输出文件的文件名。

注意到其中 `-c copy` 和 `-map 0` 出现了两次，因为这两个参数都是作用于输出文件的，这里有两个输出文件，对每个输出都要指定一次。

FFmpeg 在执行这条命令时，会打开一次 `full.flv`，然后开启两个写进程：

1. 第一个进程从 `full.flv` 开头复制到 `06:00:00` 处停止。
2. 第二个进程从 `06:00:00` 复制到文件结束。

### 03. 转换封装格式

```shell
ffmpeg -i 'test.flv' -c copy 'test.ts'
```

这个命令可以无损地将 `flv` 格式的视频转换为 `ts` 格式，注意只转换了封装格式，并没有重新编码。

---

*参考链接：*

1. [如何使用 FFmpeg 进行视频转码:首页 - FiveYellowMice's Wiki](https://wiki.fiveyellowmice.com/wiki/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8_FFmpeg_%E8%BF%9B%E8%A1%8C%E8%A7%86%E9%A2%91%E8%BD%AC%E7%A0%81:%E9%A6%96%E9%A1%B5)
2. [FFmpeg 视频处理入门教程 - 阮一峰的网络日志](http://www.ruanyifeng.com/blog/2020/01/ffmpeg.html)
3. [FFmpeg常用命令(长期更新) – MikuBlog](https://mikublog.com/tech/2328)
4. [biliup/Dockerfile at master · biliup/biliup](https://github.com/biliup/biliup/blob/master/Dockerfile)