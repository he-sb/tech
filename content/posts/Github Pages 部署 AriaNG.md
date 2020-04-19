+++
title = "Github Pages 部署 AriaNG"
description = " "
date = "2020-04-03T21:33:59+08:00"
categories = ["奇技淫巧"]
tags = ["aria2","github"]
slug = "deploy-ariang-on-github-pages"
draft = false
+++

~~本文是个简易教程，~~ AriaNG 本质上是个纯静态网站，不需要服务器也可以运行，只要有个地方用来展示就可以了，因此像 [Github Pages](https://pages.github.com/) 这类提供静态站点托管的服务也是可以的，某种程度上比用自己的服务器更为方便（安全？）。

## 1.下载并解压 AriaNG

1. 去 AriaNG 的 [Releases 页面](https://github.com/mayswind/AriaNg/releases/latest) 下载不带 【AllInOne】 后缀的压缩包；
2. 在本地将压缩包解压。

## 2.在 Github 上新建一个仓库

1. 点击右上角的头像 -> 下面的 `Your repositories` ;
2. 右上角 `New` ；
3. `Repository name` 输入一个方便自己辨别的名字，要英文；
    > 本例中俺新建的仓库名为 `aria-ng`
4. `Description` 可以不填，也可以写上自己对于此仓库的描述，中英文均可；
5. 仓库类型保持默认的 `Public` 不用改，点击下面的 `Create repository` 创建仓库。

## 3.上传 AriaNG

将【[1.](#1下载并解压-ariang)】中解压出来的文件上传至刚刚建好的仓库的根目录，可以使用网页上传或 git 方式上传，效果是一样的。

## 4.配置 Github Pages

1. 进入刚才建立的仓库页面，点击仓库右上方的 `Settings` ；
2. 向下翻，找到 `Github Pages` 栏， `Source` 选择 `master branch` ，此时页面会自动刷新一下，再次向下翻到 `Github Pages` ，会发现显示：
    > Your site is published at http://he-sb.top/aria-ng/
    > 
    因为俺的博客也是部署在 Github Pages 上的，之前设置了自定义域名为 `he-sb.top` ，如果你是第一次使用 Github Pages 或者之前没有配置过自定义域名的话，此处显示的域名应该是类似 `http://he-sb.github.io/aria-ng/` 的形式，其中 `he-sb` 是 Github 的用户名， `aria-ng` 是之前配置的仓库名；
3. 此时访问上面显示的域名应该可以正常使用 AriaNG 了。如果没有特殊需求，以下内容可以不必阅读。

## 5.进阶操作（非必选）

* 如需对 AriaNG 页面自定义域名，先去自己的域名服务商那里新建 DNS 记录，新建一条【欲使用的域名】指向【github 用户名.github.io】的 `CNAME` 记录，
    > 假设俺想使用 `aria2.he-sb.top` 这个域名来访问 AriaNG，那么俺需要新建一条值为 `aria2` ，内容为 `he-sb.github.io` ，类型为 `CNAME` 的记录
    > 
    稍等片刻等待 DNS 解析生效后回到 Github ，在仓库内存放 AriaNG 网站的相同路径下添加一个文件名为 `CNAME` 的文件（注意无后缀名），内容为自定义的域名（注意前面不要带 `http://` 或 `https://` 前缀），
    > 如本例中 `CNAME` 文件内容应为 `aria2.he-sb.top`
    > 
    然后在上一条 `Github Pages` 配置的 `Custom domain` 内填入自定义的域名，点击 `Save` 即可；
* 如需配置 HTTPS 访问，先勾选【[4.](#4配置-github-pages)】中第 2 步 `Github Pages` 栏下面的 `Enforce HTTPS` ，稍等片刻等待申请证书，完毕后此处上方的提示会变为：
    > Your site is published at https://he-sb.top/aria-ng/
    > 
    **特别提醒：** 如果 AriaNG 前端配置了 HTTPS ，那么只能连接 RPC 开启了 TLS 的后端，这是 AriaNG 的限制，请参考 [使用 HTTPS 连接 aria2](/posts/use-https-on-aria2/) 中的说明来配置；
* 【[4.](#4配置-github-pages)】中提到
    > 2. 向下翻，找到 `Github Pages` 栏， `Source` 选择 `master branch` ……
    > 
    此处是本文为了简单起见（毕竟是【简易教程】嘛 ~~实际上就是懒~~），选择了新建一个仓库来演示，实际上如果你之前建立过托管于 Github Pages 的网站，此处还可以选：
    * `master branch /docs folder` : 前提是你在【[3.上传 AriaNG](#3上传-ariang)】这一步将解压好的文件上传到了仓库根目录下的 `docs` 文件夹内；
    * `gh-pages branch` : 前提是你在【[3.上传 AriaNG](#3上传-ariang)】这一步将解压好的文件上传到了仓库的 `gh-pages` 分支，若不存在这个分支，则此处不会有这个选项出现。  
    > 
    以上两个选项一般用于为已有仓库开启 Github Pages 的情况，就可以跳过【[2.在 Github 上新建一个仓库](#2在-github-上新建一个仓库)】了（当然前提是此仓库当前未开启 Github Pages，一个仓库只能同时配置一个 Github Pages 网站），且不影响上两条的【配置自定义域名】与【配置 HTTPS】。