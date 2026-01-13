+++
title = "curl 操作合集"
description = " "
date = "2026-01-13T22:53:24+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["curl"]
slug = "usage-of-curl"
draft = false
+++

curl 是一个非常实用的命令行程序（是的，俺知道它也是一个函数库，但这和本文无关），并且几乎所有现代的 Linux 发行版都自带。在命令行编程，调试网络，测试 debug 等场景，几乎找不到比 curl 更方便，更万能的工具了，更详细的介绍可以看看 [curl 官网对它的介绍](https://curl.se/docs/faq.html)，本文记录一些个人日常经常使用的命令，并配上选项介绍，文末整理了俺保存的一些学习资源。

### 00. 通过代理发起连接

使用 `-x` 选项来指定代理服务器，参数格式为 `[protocol://]host[:port]` ，不指定代理协议类型时默认为 `http` ，代理协议也可以是 `socks4` `socks5` `socks5h` 等，不指定端口时默认为 `1080`.

下面是几个示例：

```shell
# http 代理
## 下面两个命令是等价的
curl -x 192.168.1.111 https://example.com
curl -x http://192.168.1.111:1080 https://example.com
# socks5 代理（不代理 DNS 请求）
curl -x socks5://192.168.1.111:1080 https://example.com
# socks5 代理（代理 DNS 请求）
curl -x socks5h://192.168.1.111:1080 https://example.com
```

### 01. 下载文件

```shell
curl -O -fsSL https://example.com/test.zip
```

选项释义：

- `-O` / `--remote-name`: 使用远程文件系统的文件名来保存下载的文件
  - 如果需要自定义下载的文件名，使用 `-o <custom_file_name>` 参数
- `-f` / `--fail`: 在服务器返回 HTTP 错误（例如 `404 Not Found`）时静默失败，不会输出错误页面，而是直接以非零的退出码结束
  - 在脚本中非常有用，可以方便地判断下载是否成功
- `-s` / `--silent`: 在下载过程中不显示进度条，或错误信息，只在下载失败时返回一个非零的退出码
  - 同上个选项，脚本中非常实用
- `-S` / `--show-error`: 在使用了 `-s` 选项的情况下，如果出现错误，依然会显示错误信息，方便调试
- `-L` / `--location`: 如果服务器返回重定向（例如 `301` 或 `302` 跳转），自动跟踪重定向到新的 URL

### 02. 测量延迟 / 网站测速

```shell
curl -so /dev/null -w "\ntime_namelookup: %{time_namelookup}\ntime_connect: %{time_connect}\ntime_appconnect: %{time_appconnect}\ntime_pretransfer: %{time_pretransfer}\ntime_redirect: %{time_redirect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" https://www.google.com
```

选项释义：

- `-s` / `--silent`: 同上节，静默模式
- `-o /dev/null`: 丢弃获取到的页面内容
- `-w` / `--write-out`: 请求完成后以指定的格式，输出指定的时间变量

上面的命令执行后，输出类似下面这样：

```shell
time_namelookup: 0.004687
time_connect: 0.005549
time_appconnect: 0.179526
time_pretransfer: 0.180173
time_redirect: 0.000000
time_starttransfer: 0.365058
time_total: 0.365386
```

具体支持哪些变量，以及变量对应的意义是什么，请参见官方文档……

### 03. 附加 HTTP 请求头（Headers）

使用 `-H` / `--header` 选项来附加或覆盖请求头：

```shell
# 附加单个请求头
curl -H "<Header-Name>: <Value>" URL
# 附加多个请求头
curl -H "Authorization: Bearer <YOUR_TOKEN>" \
     -H "Accept: application/json" \
     https://api.example.com/user
# 覆盖默认请求头
## 这个例子会替换 curl 默认的 "User-Agent: curl/7.x.x" 请求头
curl -H "User-Agent: MyBrowser/1.0" https://example.com
# 删除默认请求头
## 这个例子发出的请求中将会没有 "User-Agent" 请求头
curl -H "User-Agent:" https://example.com
```

### 04. 改变 Method

`curl example.com` 命令默认使用 `GET` 方法，如果要发起其他类型的请求（比如 `POST`），需要通过 `-X` 选项来指定：

```shell
# 发起 POST 请求
curl -X POST http://prometheus.he-sb.home/-/reload
```

特别地，如果一个请求既需要使用 `POST` 方法，又要提交请求体信息（实际上大多数 `POST` 请求都是这样），那么可以使用 `-d` 选项添加请求体，此时可以省略 `-X POST` 选项：

```shell
curl -H "Content-Type: application/json" -d '{"name": "test"}' https://api.example.com
```

---

*参考链接：*

1. [curl - Frequently Asked Questions](https://curl.se/docs/faq.html)
2. [curl网站开发指南 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2011/09/curl.html)
3. [curl 的用法指南 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2019/09/curl-reference.html)
4. [Timing With Curl - Susam Pal](https://susam.net/timing-with-curl.html)
5. [curl 常用的一些命令 - 格物致知](https://pyer.dev/post/tips-for-curl-4c8c6242)
6. [使用curl进行网站测速 - 暗无天日](http://blog.lujun9972.win/blog/2021/06/08/%E4%BD%BF%E7%94%A8curl%E8%BF%9B%E8%A1%8C%E7%BD%91%E7%AB%99%E6%B5%8B%E9%80%9F/index.html)