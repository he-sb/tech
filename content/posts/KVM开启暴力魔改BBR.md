+++
title = "KVM开启暴力魔改BBR"
description = " "
date = "2018-12-06T13:37:35+08:00"
categories = ["linux"]
tags = ["kvm","bbr","加速"]
slug = "kvm-bbr"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

再记录一下之前给瓦工kvm小鸡装暴力魔改bbr加速的过程吧。

本文基于搬瓦工的KVM虚拟化VPS，理论上KVM架构的、CentOS 7 x64的鸡都可以用，其他厂家、其他系统不保证可用性，请自行摸索。

## 先检查系统更新

```bash
yum -y update
```

## 更换内核

bbr只支持4.9以上4.13以下的内核，此处更换为4.11.9

```bash
yum install -y wget && wget --no-check-certificate -O C71.sh https://raw.githubusercontent.com/xratzh/CBBR/master/C71.sh && sudo bash C71.sh
```

安装完后需要输入 `y` 确认，之后系统会自动断开putty，重启，重新登陆后会启用新的内核。

## 安装魔改版BBR

```bash
wget --no-check-certificate -O C72.sh https://raw.githubusercontent.com/xratzh/CBBR/master/C72.sh && sudo bash C72.sh
```

结束后显示 `Finish` 表示正常，或执行 `lsmod |grep 'bbr_powered'` ，若返回结果不为空，则加载模块成功。

---

*参考来源：*

1. [CentOS一键魔改bbr加速搬瓦工shadowsocksr教程](https://xratzh.com/2017/10/09/bwg3/)