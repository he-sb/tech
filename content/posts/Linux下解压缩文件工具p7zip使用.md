+++
title = "Linux下解压缩文件工具p7zip使用"
description = " "
date = "2019-07-02T12:37:35+08:00"
categories = ["linux"]
tags = ["p7zip","解压"]
slug = "usage-of-p7zip"
comments = true
draft = false
+++
首先安装

```bash
yum install p7zip
```

解压使用

```bash
7za x test.7z
```

或

```bash
7za e test.7z
```

区别在于，`-e` 参数会无视压缩包内文件结构，将所有文件解压至当前目录下，而 `-x` 参数会保留压缩包内目录结构。

想要指定输出路径，可以使用`-o`参数，后跟绝对路径。

解压带密码的压缩包，可以使用`-p`参数，注意后面和密码之间无空格，例如

```bash
7za x test.7z -ppassword
```