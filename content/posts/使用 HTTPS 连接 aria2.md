+++
title = "使用 HTTPS 连接 Aria2"
description = " "
date = "2020-04-03T21:28:38+08:00"
categories = ["奇技淫巧"]
tags = ["aria2","https"]
slug = "use-https-on-ariang"
draft = false
+++

aria2 服务端搭好后（[Docker 安装 aria2](/posts/install-aria2-through-docker)），还需要搭配 AriaNG 前端面板来使用，[之前的方法](/posts/setup-offline-download-service-aria2-ariang-filebrowser-on-centos7/#安装ariang)虽然能用，但是配置好后是使用 HTTP 协议来访问的，虽然小破站也没什么人知道，不怎么担心安全问题，但是浏览器地址栏的红色小锁看着让轻度强迫症的俺十分不爽，于是研究了一下如何使用 HTTPS 来访问 AriaNG ，本文应该是目前最全面的 AriaNG 教程了。

## 1.aria2 配置文件中指定证书文件路径

* 如果是参考 [Docker 安装 aria2](/posts/install-aria2-through-docker) 这篇来搭建的 aria2 ，配置文件路径参考 [这一小节](/posts/install-aria2-through-docker/#参数说明) 中的内容；

* 如果是手动安装的 aria2 ，配置文件路径参考 [这一小节](/posts/setup-offline-download-service-aria2-ariang-filebrowser-on-centos7/#%E7%BC%96%E8%BE%91%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6) ；

* 使用一键脚本之类的方法安装的话，配置文件如何修改请参考所使用的脚本的说明。

配置文件中相关参数：

```bash
# 是否启用 RPC 服务的 SSL/TLS 加密,
# 启用加密后 RPC 服务需要使用 https 或者 wss 协议连接
rpc-secure=true
# 在 RPC 服务中启用 SSL/TLS 加密时的证书文件(.pem/.crt)
rpc-certificate=/root/xxx.pem
# 在 RPC 服务中启用 SSL/TLS 加密时的私钥文件(.key)
rpc-private-key=/root/xxx.key
```

注意：
* 通过本方法配置完后， 部署AriaNG 实际上不需要用到自己的服务器，可以使用 [Github Pages](https://pages.github.com/) 等静态站点托管服务，更加方便安全且便于复用，详情请移步 [Github Pages 部署 AriaNG](/posts/deploy-ariang-on-github-pages) 。

* 按照以下说明修改完 aria2 配置文件后，要修改 AriaNG 设置中的 `Aria2 RPC 协议` 参数为 `Https` 。

### 1.1 不使用 CDN

1. 使用 [`acme.sh`](https://github.com/acmesh-official/acme.sh) 或 [`certbot`](https://certbot.eff.org/) 或 [`Caddy`](https://caddyserver.com/) 自动申请及续期 Let's Encrypt 免费证书。

2. 配置文件中指定证书路径。以上三种方法生成的证书文件路径有所不同，请自行查找。

### 1.2 使用 CDN

1. 申请 Cloudflare 15年自签证书：`SSL` 页签 -> 下面的 `Origin Certificates` -> 右侧 `Create Certificate` -> 根据提示来生成证书，最下面的年限选择最长的 `15 years` 就行，因为使用了 CDN 之后此证书只会用来与 CF 的 CDN 服务器进行验证，不必担心安全问题。

2. 点击 `next` 之后此页面下方应该能看到生成好的证书和私钥了，点击右侧的 `Download` 把这两个文件下载下来，将存放这两个文件的路径填入配置文件内。

3. 回到 CF ：

    1. `DNS` 页签，点亮对应域名右侧的云朵图标。

    2. `SSL` 页签：点击 `Overview` ，SSL 模式选择 `Full` 。

4. 现在应该能通过 HTTPS 来访问 AriaNG 了。

## 2.使用 Nginx 反代端口

此方法无需修改 aria2 配置文件，只需修改 Nginx 配置，设置好反向代理即可。

本方法假定系统环境为：**AriaNG 与 aria2 配置在同一台机器上，且 AriaNG 网站使用 Nginx 搭建。**

### 2.1 操作说明

* 若你的环境与本文相同，那么这样操作：

    1. 给网站配置好证书（本方法中对证书来源无要求，只要用来访问 AriaNG 的域名有对应的 SSL 证书即可）。

    2. 在 Nginx 网站配置中添加以下内容：

        ```bash
        location ^~ /jsonrpc {
            proxy_http_version 1.1;
            add_header Front-End-Https on;
            proxy_set_header Connection "";
            proxy_set_header Host $http_host;
            proxy_set_header X-NginX-Proxy true;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:6800/jsonrpc;
            proxy_pass_header X-Transmission-Session-Id;
        }
        ```

    3. 在 AriaNG 设置中更改 `Aria2 RPC 地址` 参数后面的端口号为 `443` 。

    4. 此时 AriaNG 应该可以正常通过 HTTPS 使用了，本文后面的部分不必阅读。

* 若你的环境与本文不同：

    1. 在同一台机器上使用 Nginx 搭建搭建了 AriaNG ，但需通过此站点来访问其他机器的 aria2 ：将上文中添加的 Nginx 配置项，其中的 `127.0.0.1:6800` 字段修改为搭建了 aria2 的   机器的地址和对应端口。

    2. AriaNG 使用静态站点方式来部署：参见上文【[1.aria2 配置文件中指定证书文件路径](#1aria2-配置文件中指定证书文件路径)】部分。

    3. 使用别人搭建好的 AriaNG 来访问自己的 aria2 ：参见上文【[1.aria2 配置文件中指定证书文件路径](#1aria2-配置文件中指定证书文件路径)】部分。

### 2.2 一点小瑕疵

通过反代端口的方法，虽然配置方便，且成功率高，但是仍不完美：

* 必须使用自己的服务器来通过 Nginx 来搭建 AriaNG ，且参照前述方法配置好反代后的 AriaNG 只能访问【一台】机器的 aria2 。

原因是 Nginx 配置中固定了反代地址（即搭建了 aria2 的服务器地址），如果你有多个 aria2 服务需要连接，临时的解决办法：

* 参考上文【[1.aria2 配置文件中指定证书文件路径](#1aria2-配置文件中指定证书文件路径)】部分。

完美的解决办法有待研究。 ~~又可以愉快地咕咕咕了~~

---

*参考链接：*

1. [取巧办法使用HTTPS链接Aria2 - Phenxso | 梦旅奈](https://www.phenxso.com/archives/162.html)