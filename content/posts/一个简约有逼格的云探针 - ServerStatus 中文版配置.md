+++
title = "一个简约有逼格的云探针 - ServerStatus 中文版配置"
description = " "
date = "2020-04-06T17:36:06+08:00"
categories = ["奇技淫巧"]
tags = ["serverstatus","探针","监控","服务器","docker","进程守护"]
slug = "usage-of-serverstatus"
draft = false
+++

**系统环境：** CentOS 7 x64

手头有几台服务器，想要方便的监控各个机器的实时状态，刚好看到有个部署简单，外观也比较有逼格的探针 -- ServerStatus 中文版， Github 项目地址：[https://github.com/cppla/ServerStatus](https://github.com/cppla/ServerStatus) ，搭建好的地址：[https://probe.he-sb.top/](https://probe.he-sb.top)。

下面记录一下搭建过程。

## 1.服务端部署

【服务端】指的是提供监控服务的机器，此机器接收客户端发来的服务器详情数据并处理后提供一个网站来展示探针页面。

### 1.1 Docker 部署

先下载配置文件模板：

```bash
wget https://raw.githubusercontent.com/cppla/ServerStatus/master/autodeploy/config.json
```

下载的 `config.json` 最好改一下名，方便自己辨认，比如俺将其重命名为了 `ServerStatus.json` ，模板内容为：

```json
{"servers":
	[
		{
			"username": "s01",
			"name": "node1",
			"type": "xen",
			"host": "host1",
			"location": "cn",
			"password": "USER_DEFAULT_PASSWORD"
		},
		{
			"username": "s02",
			"name": "node2",
			"type": "vmware",
			"host": "host2",
			"location": "jp",
			"password": "USER_DEFAULT_PASSWORD"
		},
		{
			"disabled": true,
			"username": "s03",
			"name": "node3",
			"type": "Nothing",
			"host": "host3",
			"location": "fr",
			"password": "USER_DEFAULT_PASSWORD"
		},
		{
			"username": "s04",
			"name": "ssss",
			"type": "ssss",
			"host": "ssss",
			"location": "ssss",
			"password": "USER_DEFAULT_PASSWORD"
		}
	]
}
```

其中每个键值对的意义想必不用俺重复了，一看就懂，不懂的对照下面俺修改后的再琢磨琢磨：

```json
{"servers":
	[
		{
			"username": "s01",
			"name": "BuyVM",
			"type": "KVM",
			"host": "host1",
			"location": "U.S",
			"password": "*"
		},
		{
			"username": "s02",
			"name": "主线HD",
			"type": "KVM",
			"host": "host2",
			"location": "U.S",
			"password": "*"
		},
		{
			"username": "s03",
			"name": "备线bwh",
			"type": "KVM",
			"host": "host3",
			"location": "U.S",
			"password": "*"
		}
	]
}
```

其中密码部分的值用 `*` 代替了 :)

然后运行 Docker ：

```bash
docker run -d --restart=always --name=serverstatus -v {$path}/config.json:/ServerStatus/server/config.json -p {$port1}:80 -p {$port2}:35601 cppla/serverstatus
```

参数说明：

1. `{$path}/config.json` 为修改后的配置文件路径；

2. `{$port1}` 为探针页面提供服务的端口，默认 `80` 即可，若该端口被占用（如机器上已有网站等），可自定义一个未使用的端口；

3. `{$port2}` 为服务端与客户端通信使用的端口，默认为 `35601` ，同样，若该端口被占用，可自定义。

下面是俺使用的命令（因为俺搭建服务端的这台机器已经有 Nginx 网站了，因此将 `80` 端口映射到了 `999` ）：

```bash
docker run -d --restart=always --name=serverstatus -v /root/ServerStatus.json:/ServerStatus/server/config.json -p 999:80 -p 35601:35601 cppla/serverstatus
```

最后记得防火墙放行前面自定义的端口（本例中俺打开了 `999` 和 `35601` 端口）。

### 1.2 手动部署

请参考程序作者的说明，鄙人太懒，就不折腾手动部署了：[https://github.com/cppla/ServerStatus#手动安装教程](https://github.com/cppla/ServerStatus#%E6%89%8B%E5%8A%A8%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B) 中的【服务端配置】部分。

## 2.客户端部署

【客户端】指的是被监控的机器，将本机需要记录的数据发送给服务端。

【服务端】机器也可以同时是【客户端】机器（之一）。

### 2.1 快速部署

```bash
wget --no-check-certificate -qO client-linux.py 'https://raw.githubusercontent.com/cppla/ServerStatus/master/clients/client-linux.py' && nohup python client-linux.py SERVER="server_ip" USER="s01" PASSWORD="*" >/dev/null 2>&1 &
```

参数说明：

1. `SERVER` : 将 `server_ip` 替换为搭建了【服务端】的机器的 IP ；

    > 若当前【客户端】同时为【服务端】，即【服务端】也需监控自身，那么此处填 `127.0.0.1` 或 `服务端公网 IP` 均可。

2. `USER` : 将 `s01` 替换为当前【客户端】机器实际对应的【服务端】配置文件中 `username` 的值；

3. `PASSWORD` : 将 `*` 替换为【服务端】配置文件中当前 `username` 对应的密码。

以上命令执行后终端内会返回一个数字，这个数字即是当前客户端程序（其实是一个 Python 脚本）对应的 PID 值。

~~之后记得打开客户端机器防火墙的 `35601` 端口。~~ 客户端端口不用设置，忽略本行。

### 2.2 手动部署

下载客户端脚本：

```bash
wget --no-check-certificate -qO client-linux.py 'https://raw.githubusercontent.com/cppla/ServerStatus/master/clients/client-linux.py'
```

修改其中的参数：

```bash
vi client-linux.py
```

将其中的 `SERVER` , `USERNAME` , `PASSWORD` , `PORT` 分别修改为上文中服务端配置文件内自定义的值（ `port` 为【服务端】定义的与客户端通信端口，默认为 35601 ，若未修改过，此项可以不更改）后保存，然后执行：

```bash
python client-linux.py
```

不出意外的话现在看看服务端页面应该能看到当前客户端的数据了，如果出现意外或想了解更多 ~~姿势~~ 配置请参考 [程序作者的说明](https://github.com/cppla/ServerStatus#%E6%89%8B%E5%8A%A8%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B) 的【客户端配置】部分。

## 3.服务端网页访问

1. 不讲究的话可以直接通过 `[服务端公网 IP]:[服务端网页端口]` 来访问探针页面了（如果端口是 `80` 那么不需要指定端口）；

	> 假设俺的【服务端】公网 IP 为 `123.123.123.123` ，前面指定的服务端网页端口为 `80` ，浏览器访问 `123.123.123.123` ；
	> 
	> 如果端口指定了 `8080` ，那么浏览器访问地址为 `123.123.123.123:8080` 。

2. 如果不需要 HTTPS ，那么只要去配置 DNS ，将域名解析到服务器公网 IP ，将上条的 `[服务端公网 IP]` 替换为域名即可；

3. 如果配置了 HTTPS ，或者不想通过上面两种需要显式地指定端口的方式来访问，或者兼而有之 ~~我全都要~~ ，那么你需要安装一个网站服务器，比如 [Caddy](https://caddyserver.com/) 或 [Nginx](https://www.nginx.com/) 等，并做好相应设置（如反代端口等），此处就不详述了。

## 4.配置客户端进程守护

参考 [使用 Supervisor 实现 Linux 进程守护](/posts/supervisor-in-linux-daemon) 的方法，先新建一个配置文件：

```bash
vi /etc/supervisord.d/ServerStatus.conf
```

写入如下内容：

```ini
[program: ServerStatus]
command=python client-linux.py SERVER="server_ip" USER="s01" PASSWORD="*"  ; 相应参数需要按照上文说明修改
autorestart=true
autostart=true
stderr_logfile=/var/log/ProjectName.err.log
stdout_logfile=/var/log/ProjectName.out.log
user=root
startsecs=1
```

保存后重新载入 Supervisor 即可：

```bash
supervisorctl reload
```