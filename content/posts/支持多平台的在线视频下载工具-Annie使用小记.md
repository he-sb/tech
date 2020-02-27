+++
title = "支持多平台的在线视频下载工具-Annie使用小记"
description = " "
date = "2019-09-18T12:37:35+08:00"
categories = ["windows"]
tags = ["annie","下载"]
slug = "usage-of-annie"
comments = true
draft = false
+++
**0.** 命令格式

```bash
annie [options] url [url...]
```

**1.** 下载播放列表参数 `-p` ，例如

```bash
annie -p https://example
```

**2.** 列举可选清晰度但不下载，使用  `-i` 参数。

**3.** 调用本机aria2来下载，使用 `-aria2` 参数即可，若调用其他机器的aria2，需加上 `-aria2addr ip_address:port -aria2token password` 参数，输出目录默认为aria2任务目录，例如

```bash
annie -aria2 https://example    # 调用本机aria2
annie -aria2 -aria2addr 192.168.1.254:6800 -aria2token passwd    # 通过RPC调用地址为192.168.1.254端口6800，token为passwd的机器上的aria2
```

**4.** 指定输出目录，使用 `-o` 参数，后跟绝对路径，例如

```bash
annie https://example -o /root/Downloads    # 指定输出目录为"/root/Downloads"
```

**5.** 如需使用cookie，则使用 `-c cookie.txt` 或 `-c "name=value; name2=value2"` 来实现，前者需提前将cookie内容导出至txt文件中。导出cookie可使用chrome插件 `EditThisCookie` ，将导出格式设置为 `Semicolon separated name=value pairs` 即可。

---

*参考链接：*

1. [iawia002/annie: 👾 Fast, simple and clean video downloader-github](https://github.com/iawia002/annie)
2. [一款跨平台的快速，简单，干净的视频下载器：Annie，支持Bilibili/Youtube等多个网站 - Rat's Blog](https://www.moerats.com/archives/935/)