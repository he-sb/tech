+++
title = "OpenVZ 开启暴力魔改 BBR"
description = " "
date = "2018-12-06T12:37:35+08:00"
categories = ["linux"]
tags = ["openvz","bbr","加速"]
slug = "openvz-bbr"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

买了台OpenVZ的小鸡，用来挂载Google Drive和One Drive网盘做资源站（[使用rclone挂载Google Drive](/posts/mount-google-drive-as-local-disk-with-rclone)），但是中转后下载速度很不稳定，更别提还经常服务器响应超时，很影响心情，于是谷歌了一番发现ovz虚拟化的VPS也可以开暴力魔改的BBR，毫不犹豫地开了再说：

```bash
wget https://github.com/tcp-nanqinlang/lkl-rinetd/releases/download/1.1.0/tcp_nanqinlang-rinetd-centos.sh
bash tcp_nanqinlang-rinetd-centos.sh
```

运行后按照提示输入想要加速的端口号进行开启即可，实测开启后下载挂载好的gd中的文件速度稳了很多，基本上可以保持满速。

---

*参考来源：*

1. [OpenVZ平台魔改BBR一键脚本之Rinetd方式](https://www.moerats.com/archives/504/)