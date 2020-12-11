+++
title = "Emby 客户端破解"
description = " "
date = "2020-12-10T19:32:32+08:00"
categories = ["奇技淫巧"]
tags = ["emby"]
slug = "cracking-emby-client"
draft = true
+++

## 前言

对于各路资源大佬们来说，[Emby](https://emby.media/) 的大名肯定不陌生了，不了解这是啥的同学可以移步这篇文章看看效果：[EMBY用刮削好的Google Drive资源，直接挂载即可使用(VIP版) - 科学小怪人的实验室](https://blog.vwert.com/CloudStorage/Emby-GoogleDrive.html) 。

原理和过于细节的操作就不涉及了，如果看不懂，那么你可能需要补充一些 web 相关的基础知识。

本文只记录俺的配置过程，环境为：

- 客户端
    - 操作系统：Manjaro KDE x64
    - App 版本：Emby Theater 3.0.12
- 服务端（伪站所在的服务端，非 Emby Server）
    - 操作系统：CentOS 7 x64
    - Web 环境：Nginx 1.16.0

## 开搞

思路其实很简单，建立一个域名为 `mb3admin.com` 的伪站，并配置伪静态，返回客户端验证所需的 json 数据，然后在客户端所在的机器对伪站的自签名证书添加信任即可。

### 建立伪站

俺的 Nginx 伪静态配置：

```conf
location /admin/service/registration/validateDevice {
	    default_type application/json;
	  return 200 '{"cacheExpirationDays": 3650,"message": "Device Valid","resultCode": "GOOD"}';
	}
	location /admin/service/registration/validate {
	    default_type application/json;
	  return 200 '{"featId":"","registered":true,"expDate":"2099-01-01","key":""}';
	}
	location /admin/service/registration/getStatus {
	    default_type application/json;
	  return 200 '{"deviceStatus":"","planType":"Lifetime","subscriptions":{}}';
	}
```

在伪站的配置文件中添加以下内容，防止跨域访问报错：

```conf
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Headers *;
    add_header Access-Control-Allow-Method *;
    add_header Access-Control-Allow-Credentials true;
```

然后给伪站添加上自签名证书，可以使用 OpenSSL 或者国密自己签，也可以到文末附录部分直接使用俺的，除了证书有效期以外都是一样的。

### 客户端系统对伪站的根证书添加信任

俺的桌面系统使用 Manjaro，Arch Linux 以及其他的衍生发行版是一样的操作：

将 `mb3admin.com.crt` 文件移动至 `/etc/ca-certificates/trust-source/anchors/` 目录下，然后执行：

```bash
sudo trust extract-compat
```

### 客户端 HOSTS 修改 / 劫持 DNS

修改系统的 HOSTS，添加以下这行：

```conf
<伪站服务器 IP> mb3admin.com
```

或者高端的自建 DNS 之类的也可以，总之能实现【劫持客户端机器对 `mb3admin.com` 的请求至伪站服务器】就可以了。

## 尾声

好了，以上就是俺破解 Emby 客户端的过程。虽然上了正版车，但因为一直懒得搭建服务端（正所谓“书非借不能读也”嘛哈哈哈），所以折腾了下白嫖，生命在于折腾。

至于本文没有提及的内容，比如：

- Windows / Android / IOS / MacOS / xxOS 客户端怎么破解
- 如何建立伪站
- 如何修改 HOSTS / 劫持本机 DNS
- ……

原理都是差不多的，自己摸索一下就可以了，也可以参考文末【参考链接】中的教程，本文就是基于两位前辈的分享才完成的，在此感谢他们！
## 附录

以下是俺生成好的证书文件，如果你不会 / 懒得自己生成，可以直接复制粘贴保存下来使用。

注意一下有效期是到 2023.02.23，届时俺应该会提前更新好。

密钥（`mb3admin.com.key`）：
<details>
    <summary>点击展开</summary>

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAyE3Smwqo9E6CbAyN8zLOh44/6/YhBSafpDZbt9LfYl5fV2Qd
6m/pFPyXhJwcLuLB1BCEbCDYjEfq+HVcVV28nioUHyQJ6Uw2rViNNEeL3BHkgJJ4
cGzdSG9JPuuvIWc5b2P7NDhishzmhf23ZsuEHmW/3LXnlxtxFK0ZVW81gcxcgMGK
LwdYMo48+DAEmBaJ/SWeH7yEd2n/1i/75V366CkY3Mx7GDKXl7OL+kll8lssKSC/
OC/aZ/kV9ubUPD6gqjhP7qnm6J/UOVFbjNTvcd+ANZbqg1YMIU6bDruWQ9QGgd28
onlGj1JuCuXBDZIFCABkQnRE7J6Jmnj65bRVIwIDAQABAoIBAQCvxIKr5JXQFFik
QfwKTionj68N+2SMZZEvAzhGvbeiVVfqkwMhZuSsst6u0mJ0zizyTzA0xjngF3fc
YPgOPPw3+oq/rNs5qtRMFumJ18Kl8dmG7YDcfttLJcSrfxul/zPKSHz2HQiWbX/o
UnSusqYUGotbPRCla8I2N5OEgdr9MuxOZmjNiNQzcUNb11phv8L20nDfQ0KJ6FEI
Db2ktIy7hY/5z8HF6QD6azm70TernUPLsXAsnuePSV0roPbRcYWZUqGAH2RvPD/T
aG09zOLVYyZaXbnaEKL0gQvhV6W2sJx5785xzyR8Y9F+0SaijK3QBe0ZI5COdcYK
GfNkWQ+RAoGBAP6YsLC1pxKN0v2sRmKZxQjlgev+dBVlDovdQvRTDdcSgDzELfv2
+s2YfGKtb9QcvEMZnWn0rb66Ccca7yrRV7SvJMvJrV+8d1RoO7CeoPtXEWJtQWoM
XRMsovL8A3vlXaor9+UirATchNWGyNGaUyZ4BmB60rP3LUbVCpmb/5MJAoGBAMlo
go/EThI1vHoL8jkgBRZH76g9el8V8Pp426AqizataZj76alhVWMt5a9tbjKnWjV7
1rNvN4lCtmBGAWZOKEc4DA5ptFFwvA+fjCd7V0XvW9o/NLTxspXsUKqoXvrPm8m4
IJc3l32Ji6OEbRvgk7DPu1nMw+JQWDXLtA6K4xXLAoGBALc7r1bOtfq9hc+6cEHA
h7VacIIndOZ8/9YbnXd4WuqiTxbs3meMHn9fso3WYziWocvtIITEa1NsU2Mv+Fep
qOTKdMISWSwg2QUvq133HMcnp8Rd+4lWcpo9Mt2MEPnXXuz9jgEkTgeFqjh/NALe
fd+e3IANhZ2uVLC43VMmme75AoGBAMOjzQ8xtFRj9kznRcbPn6FhBx75eODcQ1RK
CayvJsNZ93UvXm21qmfIsY+SULTLcwj43jk2E1A1iUpDNiDWUrG7c5qcexeQ1lym
slG3sbKxKxv4wY3yKXMQNdtP6dLfz4hGXwIEchbzgLy5afLmVxAs+OPlz3EKcmTv
Flv59VO5AoGAa4FY+IQQ7qjisOCgdZAI4pw56Fn3PoIRDGtsUYmh9xFVvOjmaTQK
VilPn6PmjK+IyLJ9f4POyd/8yY059M44pKWTc0s70Y2qe6zLFTrtoXo+IZwTOuSx
GrfLpE7j4ct1IgpkOlpj6k6JnqPAfvdEp6Sja910FT4a8amV/309y1E=
-----END RSA PRIVATE KEY-----
```

</details>

证书（`mb3admin.com.pem`）：

<details>
    <summary>点击展开</summary>

```
-----BEGIN CERTIFICATE-----
MIIEIjCCAwqgAwIBAgIJAL0OjFdDMHPyMA0GCSqGSIb3DQEBCwUAMGgxCzAJBgNV
BAYTAkNOMRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdIYWlEaWFuMRMwEQYD
VQQKDApHTUNlcnQub3JnMSAwHgYDVQQDDBdHTUNlcnQgUlNBIFJvb3QgQ0EgLSAw
MTAeFw0yMDExMjExMDQ2NDVaFw0yMzAyMjMxMDQ2NDVaMGIxCzAJBgNVBAYTAkpQ
MQ4wDAYDVQQIDAVKYXBhbjEOMAwGA1UEBwwFSmFwYW4xDTALBgNVBAoMBEVtYnkx
DTALBgNVBAsMBEVtYnkxFTATBgNVBAMMDG1iM2FkbWluLmNvbTCCASIwDQYJKoZI
hvcNAQEBBQADggEPADCCAQoCggEBAMhN0psKqPROgmwMjfMyzoeOP+v2IQUmn6Q2
W7fS32JeX1dkHepv6RT8l4ScHC7iwdQQhGwg2IxH6vh1XFVdvJ4qFB8kCelMNq1Y
jTRHi9wR5ICSeHBs3UhvST7rryFnOW9j+zQ4YrIc5oX9t2bLhB5lv9y155cbcRSt
GVVvNYHMXIDBii8HWDKOPPgwBJgWif0lnh+8hHdp/9Yv++Vd+ugpGNzMexgyl5ez
i/pJZfJbLCkgvzgv2mf5Ffbm1Dw+oKo4T+6p5uif1DlRW4zU73HfgDWW6oNWDCFO
mw67lkPUBoHdvKJ5Ro9SbgrlwQ2SBQgAZEJ0ROyeiZp4+uW0VSMCAwEAAaOB1DCB
0TAMBgNVHRMBAf8EAjAAMAsGA1UdDwQEAwIEsDAdBgNVHSUEFjAUBggrBgEFBQcD
AQYIKwYBBQUHAwIwLAYJYIZIAYb4QgENBB8WHUdNQ2VydC5vcmcgU2lnbmVkIENl
cnRpZmljYXRlMB0GA1UdDgQWBBS9Uwqitf5yOxXykkuecQlXXfn1GzAfBgNVHSME
GDAWgBSaJ+ucgJPLDenDPdFqehzisZ846TAnBgNVHREEIDAeggxtYjNhZG1pbi5j
b22CDioubWIzYWRtaW4uY29tMA0GCSqGSIb3DQEBCwUAA4IBAQAEElFv/yf6r5bj
6BECB6JwU8nEUIou1uRjA/AyxG3w2ssvAwGdMaZIfZ1DS0yZ1Aa9BYOmvpLo1AmX
bZvQQlT5McAMiAKXWN8wmsY90ImfjlZsWgzs+nuf6uP1oUzh6o+W33PSNJrFZ3J4
rcHDipAwi39Oi2UpkWcIsE0Hg9eDirjEdJhsPlVBgJPa2+YqizyrGuQwlvGtXs0a
+Db91zxM5Iht/+rpogE7WTdhGu3hDpUgnbwjGvE9dQg6DsdXov+6RrNIKYXoWIe6
YqG2w/ACjUrVf/G3V0hFO+0I+bRYmuiZEdJhquAAZTCQ3CoAhSaIdKG192iX0+7q
fJ0fVH+u
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIDsDCCApigAwIBAgIJAMjrH5w5KmnFMA0GCSqGSIb3DQEBCwUAMGgxCzAJBgNV
BAYTAkNOMRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdIYWlEaWFuMRMwEQYD
VQQKDApHTUNlcnQub3JnMSAwHgYDVQQDDBdHTUNlcnQgUlNBIFJvb3QgQ0EgLSAw
MTAeFw0xOTEwMjQxMjM3NDRaFw0zOTA3MTExMjM3NDRaMGgxCzAJBgNVBAYTAkNO
MRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdIYWlEaWFuMRMwEQYDVQQKDApH
TUNlcnQub3JnMSAwHgYDVQQDDBdHTUNlcnQgUlNBIFJvb3QgQ0EgLSAwMTCCASIw
DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANCpZk/j4CIM2o2IiZHTsQA10LTN
fD/dV//kyn9QXQwpRpcgTLuYassucaDSvkS56+p7jRKMgD9ZnE4QNf3Ay/UEACYG
UH7OubZtigxJpLjS69dHfy3yqt8GSOKsfFu6VZ//QphFGw4NkkCYngOuxhmV7WU0
xNasollGGuzjBmp46/bev8aomkI33OxSXWna3oCn3BSScgkoyWJTNN1+EwCZANO3
FeKUyPMGOhi49QlV4OyUgCfGlFqhAGZAT/PMo8oPwwmyHrlyn+jqin7+qKVF9loc
Nle9YyBi7eZkDbSoAUOg2WFaDDRrPhUnNU+l2TqCP+uCgyxU74Lphj00v00CAwEA
AaNdMFswHQYDVR0OBBYEFJon65yAk8sN6cM90Wp6HOKxnzjpMB8GA1UdIwQYMBaA
FJon65yAk8sN6cM90Wp6HOKxnzjpMAwGA1UdEwQFMAMBAf8wCwYDVR0PBAQDAgEG
MA0GCSqGSIb3DQEBCwUAA4IBAQBcoJlabv5wgUj6tgbb3gUVYHKlQWr2aaPWg1Vs
ru5ExyPcEhyQ2XM5AdnOMjKiTikyPYwk1/K1tJSNN5AmCfdofWr4m074s+Rf/i+h
dBuh2vjZee9L/NV2ZRcxpwp9e561+JBXoHvZ0JHDBGQ0WYsJ+m9fRxCR12oIVWWv
SAjbyetRRO+oTvi3dX2OQUgJhflS4/cxQblYxgL5nMIa+MVamXUNNfwEk3TZh4K/
NgtQY5KraEUU7bCkbbKdX2r+njobTQpbBV8uZ/JwsNghx4gfB+3QrteVfceQ+ip+
CpEU9X3JD9WkxEVFKBa0Q+TllSny07of0cWmRuwZlLUruBJD
-----END CERTIFICATE-----
```

</details>

客户端所在机器需在系统添加信任的根证书文件（`mb3admin.com.crt`）：

<details>
    <summary>点击展开</summary>

```
-----BEGIN CERTIFICATE-----
MIIDsDCCApigAwIBAgIJAMjrH5w5KmnFMA0GCSqGSIb3DQEBCwUAMGgxCzAJBgNV
BAYTAkNOMRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdIYWlEaWFuMRMwEQYD
VQQKDApHTUNlcnQub3JnMSAwHgYDVQQDDBdHTUNlcnQgUlNBIFJvb3QgQ0EgLSAw
MTAeFw0xOTEwMjQxMjM3NDRaFw0zOTA3MTExMjM3NDRaMGgxCzAJBgNVBAYTAkNO
MRAwDgYDVQQIDAdCZWlqaW5nMRAwDgYDVQQHDAdIYWlEaWFuMRMwEQYDVQQKDApH
TUNlcnQub3JnMSAwHgYDVQQDDBdHTUNlcnQgUlNBIFJvb3QgQ0EgLSAwMTCCASIw
DQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANCpZk/j4CIM2o2IiZHTsQA10LTN
fD/dV//kyn9QXQwpRpcgTLuYassucaDSvkS56+p7jRKMgD9ZnE4QNf3Ay/UEACYG
UH7OubZtigxJpLjS69dHfy3yqt8GSOKsfFu6VZ//QphFGw4NkkCYngOuxhmV7WU0
xNasollGGuzjBmp46/bev8aomkI33OxSXWna3oCn3BSScgkoyWJTNN1+EwCZANO3
FeKUyPMGOhi49QlV4OyUgCfGlFqhAGZAT/PMo8oPwwmyHrlyn+jqin7+qKVF9loc
Nle9YyBi7eZkDbSoAUOg2WFaDDRrPhUnNU+l2TqCP+uCgyxU74Lphj00v00CAwEA
AaNdMFswHQYDVR0OBBYEFJon65yAk8sN6cM90Wp6HOKxnzjpMB8GA1UdIwQYMBaA
FJon65yAk8sN6cM90Wp6HOKxnzjpMAwGA1UdEwQFMAMBAf8wCwYDVR0PBAQDAgEG
MA0GCSqGSIb3DQEBCwUAA4IBAQBcoJlabv5wgUj6tgbb3gUVYHKlQWr2aaPWg1Vs
ru5ExyPcEhyQ2XM5AdnOMjKiTikyPYwk1/K1tJSNN5AmCfdofWr4m074s+Rf/i+h
dBuh2vjZee9L/NV2ZRcxpwp9e561+JBXoHvZ0JHDBGQ0WYsJ+m9fRxCR12oIVWWv
SAjbyetRRO+oTvi3dX2OQUgJhflS4/cxQblYxgL5nMIa+MVamXUNNfwEk3TZh4K/
NgtQY5KraEUU7bCkbbKdX2r+njobTQpbBV8uZ/JwsNghx4gfB+3QrteVfceQ+ip+
CpEU9X3JD9WkxEVFKBa0Q+TllSny07of0cWmRuwZlLUruBJD
-----END CERTIFICATE-----
```

</details>

---

*参考链接：*

1. [白嫖一下Emby - TimeBlog时光轴](https://imrbq.cn/exp/emby_hack.html)

2. [Emby Premiere破解思路 - 远方](http://yuanfangblog.xyz/technology/159.html)