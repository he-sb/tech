+++
title = "使用 Prometheus 抓取硬盘 S.M.A.R.T 信息"
description = " "
date = "2023-10-19T00:38:24+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["homelab","docker","prometheus","硬盘","grafana","监控"]
slug = "prometheus-hdd-smart"
draft = true
+++

## 前言

家里的存储机器上有一块 16 T 的机械硬盘，还没有集成进家中的 PLG (Prometheus + Loki + Grafana) 监控系统中，需要将这块拼图给补全，监控起它的 S.M.A.R.T 信息，并配上相应指标的告警，省得哪天硬盘突然暴毙。因为家中的监控告警体系还在施工中，目前先补齐监控体系中必需的数据源和监控面板，后需再单独撰文记录告警体系的搭建。

## 采集指标

首先参考 [这篇博客](/posts/install-node-exporter-on-linux-host/) ，在宿主机上安装好 node-exporter，并开启 text-collector 功能，创建好指标文件存放的目录，然后再开始下面的环节。

然后安装要用到的软件包，其中：

- 获取硬盘 S.M.A.R.T 信息的脚本需要用到 `smartctl` 这个命令，是由 `smartmontools` 软件包提供的
  - 关于 `smartctl` 命令的使用，可以参考 [Linux 下关于硬盘 S. M. A. R. T 的相关操作 - LUG @ USTC](https://lug.ustc.edu.cn/wiki/linux_digest/smartmontools/) 这篇文章
- 写入指标文件需要用到 `sponge` 命令，包含在 `moreutils` 软件包中

```shell
$ sudo apt update && sudo apt install -y smartmontools moreutils
```

下载 [脚本](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts/blob/master/smartmon.sh) ，并赋予执行权限：

```shell
$ wget https://raw.githubusercontent.com/prometheus-community/node-exporter-textfile-collector-scripts/master/smartmon.sh
$ chmod +x ./smartmon.sh
```

先手动执行脚本输出一次指标：

```shell
$ sudo /home/he-sb/node-exporter/smartmon.sh | sudo -u node-exporter sponge /var/lib/node_exporter/textfile_collector/smartmon.prom
```

注意：

1. 脚本的路径最好用绝对路径，这样后需添加到 crontab 中时，方便直接复制命令，避免遗漏
2. 使用 `sponge` 生成指标文件时，需要切换到 `node-exporter` 用户，直接使用 `sudo` 生成出来的指标文件，所有权是属于 `root` 用户和组的

然后检查一下是否生成了指标文件，以及 node-exporter 是否正确的将指标文件采集并输出了：

```shell
$ cat /var/lib/node_exporter/textfile_collector/smartmon.prom
$ curl localhost:9100/metrics
```

可以看到，node-exporter 输出的指标中，已经附加上了以 `smartmon_` 开头的指标，可以正常被 Prometheus 采集了。

接下来配置 crontab 定时任务，每分钟自动刷写一次文件：

```shell
$ crontab -e
# 新增下面这行
* * * * * sudo /home/he-sb/node-exporter/smartmon.sh | sudo -u node_exporter sponge /var/lib/node_exporter/textfile_collector/smartmon.prom
```

然后就可以去 Grafana 配置可视化面板来展示并监控这些指标了。

## 配置监控面板

下面简单列举一下支持的面板，可以导入后根据需要来调整：

- 13654 - [S.M.A.R.T Dashboard | Grafana Labs](https://grafana.com/grafana/dashboards/13654-s-m-a-r-t-dashboard/)
  - 推荐，直接导入就能用，信息展示简介清晰，该有的都有
- 16514 - [SMART + NVMe status | Grafana Labs](https://grafana.com/grafana/dashboards/16514-smart-nvme-status/)
  - 不太推荐，导入后部分指标无法展示，需要调整查询语句，作者写了篇 [博客](https://www.wirewd.com/hacks/blog/monitoring_a_mixed_fleet_of_flash_hdd_and_nvme_devices_with_node_exporter_and_prometheus) 介绍如何使用这个面板，以及一些告警配置，可以参考一下

关于这些指标怎么用，如何确定该关注哪些指标，如何配置合理的告警条件，可以参考下面的文章：

- [Monitoring a mixed fleet of flash, HDD, and NVMe devices with node_exporter and Prometheus | Wireworld](https://www.wirewd.com/hacks/blog/monitoring_a_mixed_fleet_of_flash_hdd_and_nvme_devices_with_node_exporter_and_prometheus)
  - 上面提到的 16514 这个面板的作者分享的关于他如何监控自家的存储设备的文章
- [What SMART Hard Disk Errors Actually Tell Us](https://www.backblaze.com/blog/what-smart-stats-indicate-hard-drive-failures/)
  - 知名云存储和数据备份备份供应商 [Backblaze](https://www.backblaze.com/) 分享的文档，很有参考价值

## 小尾巴

最后，如果你和俺一样用的是希捷的机械硬盘（HDD），发现监控到的 `Raw Read Error` 和 `Seek Error Rate` 特别高，这是因为希捷的固件中，这两个指标中同时存放了【操作次数】和【错误次数】，原始值共 48 bit（二进制 48 位），转换到 16 进制共 12 位，其中高 4 位存放的是【错误次数】，低 8 位存放【操作次数】。也就是希捷硬盘的 S.M.A.R.T 数据中，这两个值需要转换为 16 进制来看，不够 12 位的在高位补 0，然后只看高 4 位才是对应指标的【错误次数】。

简单来说，只要这两个值没有超过 4294967295 就没有问题（十进制值，对应 16 进制的 FFFFFFFF），详细解释可以参考 [这篇文章](https://www.bilibili.com/read/cv15153028/)。其他厂商的硬盘没有这个问题。

---

*参考链接：*

1. [prometheus-community/node-exporter-textfile-collector-scripts: Scripts for node-exporter's textfile collector](https://github.com/prometheus-community/node-exporter-textfile-collector-scripts)
2. [Node Exporter smartctl 文本插件 — Cloud Atlas 0.1 文档](https://cloud-atlas.readthedocs.io/zh_CN/latest/kubernetes/monitor/prometheus/prometheus_exporters/node_exporter_smartctl_text_plugin.html)
