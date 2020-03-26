+++
title = "CentOS 7 下 iptables 配置"
description = " "
date = "2018-11-23T12:37:35+08:00"
categories = ["linux"]
tags = ["iptables","防火墙","centos"]
slug = "centos7-iptables-configuration"
comments = true
draft = false
+++
**系统环境：** CentOS 7 x64

闲来无事，想着给网站服务器上装上某个服务作为备用梯子，一时懒得手动编辑配置，用了233大佬的一键脚本（来源：[233blog](https://233yes.com/post/1/)）没想到给自己挖了个大坑。。

为什么呢，倒不是说这个脚本不好使，对我来说坑爹的地方在于，这个脚本把CentOS 7自带的防火墙 `firewalld` 换成了 `iptables` ！之前配置的规则都得重新写，而且最蛋疼的地方在于，之前为了安全，我还把ssh连接的默认端口22给改了，然后直接没办法ssh连接到服务器了。。

不过还好DigitalOcean还是比较人性化的，提供了网页端的控制台连接，虽然延迟非常感人，不过好歹能用了不是，先把ssh端口改回22再说。。

终于能用putty连了回去，下一步就是重新配置一下iptables。

  首先看下现在的防火墙规则是啥样：

```bash
iptables -L -n
```

 然后结果如下：

```bash
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            state NEW udp dpt:443
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:443
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            state NEW udp dpt:80
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:80
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

可以看到除了网站用的80，443端口以外只开了ssh默认的22端口，难怪ssh连不上了，先加上自定义的ssh端口（假设修改为666）：

```bash
iptables -I INPUT 9 -m state --state NEW -p tcp --dport 666 -j ACCEPT
```

其中 `-I` 参数代表插入到具体某一行，原本该行的内容会向后移动一个顺位（个人强迫症，想对齐hhh）。不需要指定位置的话使用 `-A` 参数直接插到对应规则链最后一行就OK了。

然后删掉原本的22端口放行规则：

```bash
iptables -D 8
```

好了看下修改后的状态：

```bash
iptables -L -n
```

输出如下

```bash
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            state NEW udp dpt:443
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:443
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            state NEW udp dpt:80
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:80
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:666
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

完美！接下来还要保存到配置文件中，不然修改仅本次有效，一旦iptables服务或服务器系统重启后还会变回修改前的状态。

保存并重启iptables服务：

```bash
service iptables save && service iptables restart
```