+++
title = "在线视频下载神器 you-get 的安装及使用"
description = " "
date = "2018-12-03T12:37:35+08:00"
categories = ["windows"]
tags = ["you-get","下载"]
slug = "setup-and-useing-guide-for-you-get"
comments = true
draft = false
+++
**系统环境：** Windows 10 专业版 1803 x64

## 本文写作目的

`you-get` 是一个命令行下的小工具，可以便捷地下载各个视频网站的在线视频。网络上介绍 `you-get` 用法的文章很多，但是就本人踩坑过程中的体验，阻碍小白用户的最大障碍其实并不在于 `you-get` 本身，而是 `python3` 和 `ffmpeg` 这两个必不可少的依赖，其安装过程对于小白用户不太友好，遂有此文。

## 先安装python3.7环境

官网下载windows x64用的安装包即可，

下载地址：[https://www.python.org/ftp/python/3.7.1/python-3.7.1-amd64.exe](https://www.python.org/ftp/python/3.7.1/python-3.7.1-amd64.exe)（以当前最新版本3.7.1为例）。

下载完成后双击运行，最后记得勾选**Add Python3.7 To Path**（将安装python3.7的路径加入系统变量）。

最后cmd内输入`python`，返回类似以下内容即代表安装正常

```bash
Python 3.7.1 (v3.7.1:260ec2c36a, Oct 20 2018, 14:57:15) [MSC v.1915 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
```

## 再安装ffmpeg

ffmpeg（[官网链接](https://www.ffmpeg.org/)）是一个命令行下的视频格式转换等的工具，功能非常强大，感兴趣的可以自行搜索其用法，此处安装的目的是you-get下载分了块的视频后需要调用ffmpeg来合并分块。

下载地址：[https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-4.1-win64-static.zip](https://ffmpeg.zeranoe.com/builds/win64/static/ffmpeg-4.1-win64-static.zip)（本次用到的版本，可以根据自己情况来修改）

下载后解压至任意文件夹，之后把解压后的文件夹内 `/bin` 文件夹的完整路径加入系统环境变量内（本例中添加方法：开始->搜索“高级系统设置”并打开->点击”环境变量“->点击上方”用户变量“或下方”系统变量“内的 `Path` 行，再点击对应文本下方的”编辑“->点击右侧”新建“->将解压后的文件夹内`/bin`文件夹的完整路径粘贴至此处并保存即可，本例中路径为 `C:\Tools\ffmpeg-20181118-8f875a9-win64-static\bin` 。）

完成后在cmd中输入`ffmpeg`，返回类似以下内容说明安装正常

```bash
ffmpeg version N-92486-g8f875a90c4 Copyright (c) 2000-2018 the FFmpeg developers
  built with gcc 8.2.1 (GCC) 20181017
  configuration: --enable-gpl --enable-version3 --enable-sdl2 --enable-fontconfig --enable-gnutls --enable-iconv --enable-libass --enable-libbluray --enable-libfreetype --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-libopus --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libtheora --enable-libtwolame --enable-libvpx --enable-libwavpack --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxml2 --enable-libzimg --enable-lzma --enable-zlib --enable-gmp --enable-libvidstab --enable-libvorbis --enable-libvo-amrwbenc --enable-libmysofa --enable-libspeex --enable-libxvid --enable-libaom --enable-libmfx --enable-amf --enable-ffnvcodec --enable-cuvid --enable-d3d11va --enable-nvenc --enable-nvdec --enable-dxva2 --enable-avisynth
  libavutil      56. 23.101 / 56. 23.101
  libavcodec     58. 39.100 / 58. 39.100
  libavformat    58. 22.100 / 58. 22.100
  libavdevice    58.  6.100 / 58.  6.100
  libavfilter     7. 44.100 /  7. 44.100
  libswscale      5.  4.100 /  5.  4.100
  libswresample   3.  4.100 /  3.  4.100
  libpostproc    55.  4.100 / 55.  4.100
Hyper fast Audio and Video encoder
usage: ffmpeg [options] [[infile options] -i infile]... {[outfile options] outfile}...

Use -h to get full help or, even better, run 'man ffmpeg'
```

## 安装you-get

打开cmd，输入

```bash
pip3 install you-get
```
至于 `you-get` 本身的使用，在其官方github中已有详尽介绍，本文不再赘述，可访问其中文介绍页面：[https://github.com/soimort/you-get/wiki/中文说明](https://github.com/soimort/you-get/wiki/中文说明)