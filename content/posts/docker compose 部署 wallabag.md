+++
title = "Docker Compose 部署 Wallabag"
description = " "
date = "2023-08-09T16:53:49+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["homelab","docker","wallabag","postgresql","traefik"]
slug = "setup-wallabag-with-docker-compose"
draft = true
+++

这是一篇在内网服务器上部署 wallabag 的踩坑过程记录，使用 docker compose 部署，并使用内网其他机器上已有的 PostgreSQL 数据库和 Redis 缓存，同时使用 traefik 作为前置的反向代理。

[wallabag](https://github.com/wallabag/wallabag) 是一个可以自己部署的网页剪藏 / 稍后读应用，可以方便地保存各种网页资源，不仅能保存链接，还可以抓取页面内容（从此不怕 404），同时支持 tag 和全文搜索，确实非常方便。然而网上搜到的 wallabag 部署教程多数都是基于 [官方示例](https://github.com/wallabag/docker#docker-compose) ，使用 MySQL / MariaDB 作为数据库，而且没有配置 Redis 缓存。这应该是为数不多使用 PostgreSQL 作为数据库的教程（用的人少，坑自然多些，不过俺已经替各位趟平了，可以放心阅读）。

至于为什么要用 PostgreSQL 作为数据库，是因为：
1. 内网中还有其他应用在用，不希望每个应用都部署一个自己的数据库，一来不方便备份数据，二来对 HomeLab 场景下有限的计算和存储资源也是一种浪费
2. PostgreSQL 在大多数场景下可以替代 MySQL，在容器时代，大多数现代构建的镜像都支持使用 PostgreSQL 作为数据库，或者至少同时支持 PostgreSQL 和 MySQL 。俺认为长远来看这也是技术上的趋势，详见：
  - [Postgres vs. MySQL: a Complete Comparison in 2023](https://www.bytebase.com/blog/postgres-vs-mysql/)
  - [PostgreSQL: About](https://www.postgresql.org/about/)

言归正传，下面开始踩坑之旅。

## 启动容器

首先新建一个文件夹，用来部署 wallabag：

```shell
mkdir wallabag
cd wallabag
```

<details>
<summary>
docker-compose.yml 文件内容（俺注释掉了官方示例中的 `healthcheck` 部分）
</summary>

```yaml
version: '3'

x-common: &default
  restart: unless-stopped
  networks:
    - traefik
  logging:
    driver: json-file
    options:
      max-size: '10m'
  environment: &default-environment
    TZ: Asia/Shanghai

services:
  wallabag:
    <<: *default
    image: wallabag/wallabag:2.6.2
    container_name: wallabag
    volumes:
      # 将图片挂载出来存储
      - ./data:/var/www/wallabag/web/assets/images
    environment:
      <<: *default-environment
      POSTGRES_USER: homelab_pgsql
      POSTGRES_PASSWORD: homelab_pgsql
      SYMFONY__ENV__DATABASE_DRIVER: pdo_pgsql
      SYMFONY__ENV__DATABASE_HOST: 192.168.1.11
      SYMFONY__ENV__DATABASE_PORT: '5432'
      SYMFONY__ENV__DATABASE_NAME: wallabag
      SYMFONY__ENV__DATABASE_USER: homelab_pgsql
      SYMFONY__ENV__DATABASE_PASSWORD: homelab_pgsql
      SYMFONY__ENV__LOCALE: zh
      SYMFONY__ENV__DOMAIN_NAME: https://wallabag.he-sb.home
      SYMFONY__ENV__SERVER_NAME: wallabag on HE-SB home
      SYMFONY__ENV__REDIS_HOST: 192.168.1.11
      SYMFONY__ENV__REDIS_PORT: '6379'
      # Redis 密码
      # Redis 部署时未配置密码的话删掉就行了
      SYMFONY__ENV__REDIS_PASSWORD: homelab_redis
    labels:
      traefik.enable: 'true'
      traefik.docker.network: traefik
      # http 配置
      traefik.http.routers.wallabag-http.entrypoints: http
      traefik.http.routers.wallabag-http.rule: Host(`wallabag.he-sb.home`)
      traefik.http.routers.wallabag-http.service: noop@internal
      traefik.http.routers.wallabag-http.middlewares: https-redirect@file
      # https 配置
      traefik.http.routers.wallabag.entrypoints: https
      traefik.http.routers.wallabag.tls: 'true'
      traefik.http.routers.wallabag.rule: Host(`wallabag.he-sb.home`)
      traefik.http.services.wallabag.loadbalancer.server.port: '80'
    # healthcheck:
      # test: ["CMD", "wget" ,"--no-verbose", "--tries=1", "--spider", "http://localhost"]
      # interval: 1m
      # timeout: 3s

networks:
  traefik:
    external: true
```
</details>

字段说明：

<details>
<summary>
环境变量部分
</summary>

- `POSTGRES_USER` 和 `POSTGRES_PASSWORD`
  - PostgreSQL 数据库的用户名和密码
- `SYMFONY__ENV__DATABASE_DRIVER`
  - 指定数据库类型为 PostgreSQL
- `SYMFONY__ENV__DATABASE_HOST`
  - 数据库地址
- `SYMFONY__ENV__DATABASE_PORT`
  - 数据库端口
  - 如果 PostgreSQL 部署时没有修改端口，那么默认应该就是 `5432` ，这一行可以直接删掉
- `SYMFONY__ENV__DATABASE_NAME`
  - 数据库名称
- `SYMFONY__ENV__DATABASE_USER` 和 `SYMFONY__ENV__DATABASE_PASSWORD`
  - PostgreSQL 数据库的用户名和密码
  - 和上方环境变量的区别是，上面的是给容器内的初始化脚本用的，而这两个是给容器内的 php 程序用的（俺也不晓得为啥要整两份。。）
- `SYMFONY__ENV__LOCALE`
  - 修改默认语言为中文
  - 原本的默认语言是英语（ `en` ）
- `SYMFONY__ENV__DOMAIN_NAME`
  - 程序反代后的完整 URL
  - 需要带上协议名称（https / http）
- `SYMFONY__ENV__SERVER_NAME`
  - 程序的主机名
  - 可以随便起一个，不是很影响，删掉也没关系
- `SYMFONY__ENV__REDIS_HOST` 和 `SYMFONY__ENV__REDIS_PORT`
  - Redis 的地址和端口
  - 如果 Redis 也部署在同一台宿主机上，这里最好使用宿主机的局域网 IP，而不是 `127.0.0.1`
- `SYMFONY__ENV__REDIS_PASSWORD`
  - Redis 的密码
  - 如果 Redis 部署时未配置密码，直接删掉这一行就好了

</details>

<details>
<summary>
labels 部分
</summary>

`labels` 部分是提供给 traefik 用于服务发现的，如果你用的是 NPM 或者 caddy 之类的反代，可以将容器的 `80` 端口暴露出来，然后直接将域名反代至容器暴露出来的端口，然后删掉 `docker-compose.yml` 文件中的 `networks` 和 `labels` 部分即可。

`docker compose up -d` 将容器启动后，访问 `wallabag.he-sb.home` ，不出意外的话应该会显示 `500` 错误，推测是因为俺直接连接了已有的 PostgreSQL 数据库，而 wallabag 在已有的数据库中新建数据库失败了，所以服务没法正常运行。下面来手动修复一下这个问题。

</details>

## 解决问题

### 创建数据库表结构

```shell
# 进入 wallabag 容器
docker exec -it wallabag sh
# 下面的命令都是在容器内执行的
# 首先进入程序目录
cd /var/www/wallabag
# 执行数据库迁移（用于创建正常的表结构）
php bin/console doctrine:migrations:migrate --no-interaction --env=prod
```

中间会闪过若干个报错信息，先不用在意，能看到命令执行最后，输出了 `finished` 字样即可。此时刷新一下 wallabag 网页，应该可以正常展示登录界面了。

### 创建管理员账户

然而，此时无论输入什么用户名和密码（比如网上教程中都说 wallabag 刚安装好时的默认用户名密码都是 `wallabag` ），都会提示用户名密码错误，没法登录。这是因为容器刚启动时，因为创建新的数据库失败了，导致没有新建成功任何账户，而上文的数据库迁移命令仅能迁移已有的数据库表和模式，对于原本就没有的账户数据，并没有新增，这就导致了此时的数据库内实际上并没有任何用户数据。下面俺通过手动再执行一次安装命令，来解决这个问题。

```shell
# 进入 wallabag 容器
docker exec -it wallabag sh
# 下面的命令都是在容器内执行的
# 首先进入程序目录
cd /var/www/wallabag
# 重新启动安装流程
php bin/console wallabag:install -e prod
```

此时会有一些交互式的操作，俺在必要的地方增加了注释，注意看清提示信息再输入：

<details>
<summary>
点击展开
</summary>

```shell
wallabag installer
==================

Step 1 of 4: Checking system requirements.
------------------------------------------

 ------------------------ -------- ---------------- 
  Checked                  Status   Recommendation  
 ------------------------ -------- ---------------- 
  PDO Driver (pdo_pgsql)   OK!                      
  Database connection      OK!                      
  Database version         OK!                      
  curl_exec                OK!                      
  curl_multi_init          OK!                      
 ------------------------ -------- ---------------- 

 [OK] Success! Your system can run wallabag properly.

Step 2 of 4: Setting up database.
---------------------------------

# 因为已经通过数据库迁移命令创建成功了数据库
# 此处不需要重置数据库，避免影响其他数据
# 直接输入 no 后回车即可
 It appears that your database already exists. Would you like to reset it? (yes/no) [no]:
 > no

# 同上，输入 no 后回车即可
 Seems like your database contains schema. Do you want to reset it? (yes/no) [no]:
 > no

 Clearing the cache...

 Database successfully setup.

Step 3 of 4: Administration setup.
----------------------------------

# 进入这个流程的唯一目的就在这里
# 输入 yes 并回车，创建一个新的管理员账户
 Would you like to create a new admin user (recommended)? (yes/no) [yes]:
 > yes

# 要新建的用户名
# 不输入直接回车，会使用默认值 wallabag
 Username [wallabag]:
 > admin

# 设置密码
# 密码不会回显，而且只能输入一次，没有二次输入校验
# 注意别打错字，否则只能重新创建了
# 不输入直接回车，会使用默认值 wallabag
 Password [wallabag]:
 > 

# 输入用户对应的邮箱
# 不输入直接回车，会使用默认值 wallabag@wallabag.io
 Email [wallabag@wallabag.io]:
 > wallabag@he-sb.home

 Administration successfully setup.

Step 4 of 4: Config setup.
--------------------------

 Config successfully setup.

 [OK] wallabag has been successfully installed.
 [OK] You can now configure your web server, see https://doc.wallabag.org
```

</details>

此时再次刷新 wallabag 网页，使用刚才创建的用户名和密码登录就可以正常使用了。

> 理论上可以不用先执行迁移数据库的命令，直接进入安装流程，然后在提示是否重置数据库和表结构的时候，都选择 `yes` ，来让 wallabag 重新生成数据库和表结构，然后再新增管理员账户，也是一样的效果。但是俺懒得试了，看到这篇的网友们如果愿意尝试一下，可以在评论中分享一下结果，方便后来的网友们

### 给挂载的图片目录增加写入权限

然而，到这里问题还是没有全部解决，如果此时尝试保存一篇带图片的文章，那么图片内容是没法保存的（无论源网页有没有反爬措施），然后再尝试删除这篇文章，又会跳出来一个 500 错误。这是因为在 `docker-compose.yml` 文件中，将容器内的 `/var/www/wallabag/web/assets/images` 目录挂载到了 `data` 这个宿主机文件夹，但是不知为何容器内的应用对这个路径没有写入权限。俺尝试在 `docker-compose.yml` 文件中增加 `user: '1000:1000'` 配置，但重建容器后发现容器压根启动不起来了，查看日志能看到这个报错：

```log
Starting wallabag ...
/entrypoint.sh: line 28: can't create app/config/parameters.yml: Permission denied
```

推测是因为镜像构建过程中使用了其他用户，导致指定的 `1000` 用户并没有权限操作其中的文件，不过问题解决到这里，俺已经麻了，并不想再去翻 wallabag 官方的 dockerfile 。。最终俺选择简单粗暴地解决这个问题——先删掉刚才在 `docker-compose.yml` 文件中增加的 `user: '1000:1000'` 配置，然后在宿主机执行下面这个命令，将宿主机挂载目录的权限调整至 `777` ，也就是允许（任何）其他用户读写：

```shell
sudo chmod -R 777 data
```

最后再执行一下 `docker compose up -d` 重建一下容器就可以了（因为刚才修改了 compose 配置文件）。

## 小尾巴：启用 Redis 作为异步导入的缓存

进入 wallbag 网页界面，登录后，点击右上角的头像，选择【内部设置】，点击【导入】标签，下方的 `启用 Redis 来异步导入数据` 值默认是 `0` ，改为 `1` 后点击“应用”按钮，就可以开启 Redis 作为异步导入的缓存了。

此时再次点击右上角的头像，选择【导入】，应该能看到页面顶部有一行很显眼的提示：

> 导入是异步进行的。一旦导入任务开始，一个外部 worker 就会一次处理一个 job。目前的服务是： **Redis**

---

*参考链接：*
1. [wallabag/wallabag - Docker Image | Docker Hub](https://hub.docker.com/r/wallabag/wallabag)
2. [Cannot upgrade to 2.6.0 / Issue with scheb_two_factor · Issue #6649 · wallabag/wallabag](https://github.com/wallabag/wallabag/issues/6649#issuecomment-1616021842)
3. [Default user not created · Issue #6795 · wallabag/wallabag](https://github.com/wallabag/wallabag/issues/6795#issuecomment-1665284333)
