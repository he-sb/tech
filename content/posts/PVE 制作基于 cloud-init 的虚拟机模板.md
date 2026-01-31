+++
title = "PVE 制作基于 Cloud Init 的虚拟机模板"
description = " "
date = "2023-08-05T01:56:08+08:00"
toc = true
categories = ["奇技淫巧"]
tags = ["pve","cloud-init","debian"]
slug = "creating-vm-template-for-pve-based-on-cloud-init"
draft = false
+++
**系统环境：** Proxmox VE 7.4 x64

## 00. 准备工作

首先需要在 PVE 创建一个新虚拟机：
- 操作系统
	- 勾选 `不使用任何介质`
- 系统
	- 勾选 `QEMU 代理`
	- 其他均保持默认
- 硬盘
	- 删除硬盘
	- 若已添加，先分离后删除
- CPU
	- 类型修改为 `host`
- 内存
	- 取消勾选 `Bollooning`

创建后，进入【硬件】页面，继续修改：
- 删除 `CD / DVD 驱动器`
- 添加 `串行端口`
	- 序号 `0`
	- 这一步非必须，如果没有通过 xterm.js 访问虚拟机的需求，可以不添加
- 添加 `CloudInit 设备`
	- `总线 / 设备` 选择 `SCSI`
	- 序号 `1`

接下来记住虚拟机 ID（本文以 `108` 为例）， SSH 连接到 PVE 宿主机，下载 cloud-init 版本的系统镜像：

> 本文以 `debian-12-generic-amd64-20230723-1450.qcow2` 镜像为例，要了解 Debian 的不同版本 cloud 镜像的区别，可以参考：
> https://cloud.debian.org/images/cloud/

```shell
curl -OL https://cloud.debian.org/images/cloud/bookworm/20240507-1740/debian-12-generic-amd64-20240507-1740.qcow2
```

校验下载的文件是否正确，完整：

```shell
sha512sum debian-12-generic-amd64-20240507-1740.qcow2
```

查看计算出的 SHA512 值，是否和 [官网公布](https://cloud.debian.org/images/cloud/bookworm/20240507-1740/SHA512SUMS) 的一致，不一致需要重新下载，或者更换镜像源。如果一致，那么将下载到的文件转换为虚拟机磁盘：

```shell
qm importdisk 108 ~/debian-12-generic-amd64-20240507-1740.qcow2 nfs_g4600 --format=qcow2
```

其中，`108` 为上文创建好的虚拟机的 ID， `nfs_g4600` 为 PVE 宿主机用于存储【磁盘镜像】的存储配置（如果你没有修改过 PVE 宿主机的存储配置，那么这里应该是 `local-lvm` ）， `--format=qcow2` 表示导入后的磁盘格式为 `qcow2` ，不加这个参数的话，默认导入后的磁盘格式为 `raw` ，是无法拍摄快照的，使用起来比较不便。

导入完成后，查看虚拟机的【硬件】配置，应该能看到下方多了一行 `未使用的磁盘 0` ，双击此项后勾选 `SSD 仿真` （这一步非必需），`总线 / 设备` 选择 `SCSI` ，点击“添加”按钮，添加为磁盘后，单击新添加的磁盘，点击上方的“磁盘操作”按钮，选择“调整大小”，输入 `1` ，点击“确认”，给磁盘增加 1 G 容量，否则下文配置时会出现磁盘空间不足的情况（默认的硬盘只有 2 G 容量）。

然后进入【选项】修改：
- 使用平板仿真
	- 改为 `否`
- 引导顺序
	- 勾选刚导入的硬盘，并拖至第一行

在启动该虚拟机之前，还有最后一个需要修改的地方，那就是 cloud-init 配置，进入 `Cloud-Init` 配置页面：
- 用户
	- 系统内第一个非 root 用户的名称，可自定义（建议配置）
	- 若未配置
		- 系统启动后不会新增用户，默认使用 root 用户
		- 默认允许 root 用户通过 SSH 登录
- 密码
	- 系统内第一个非 root 用户的密码，可自定义（建议配置）
	- 若【用户】未配置，则此处的密码则为 root 密码
- DNS 域 / DNS 服务器
	- 留空则使用 DHCP，配置了则使用静态 DNS
- SSH 公钥
	- 若未配置，则 SSH 允许通过密码登录
	- 若已配置，则 SSH 禁止通过密码登录，仅允许通过公钥对应的私钥登录
- IP 配置
	- 可分别配置对应网卡的 IPv4 和 IPv6 地址策略
	- 支持静态 IP 或 DHCP

配置完成后，启动虚拟机，准备配置模板系统。

## 01. 配置模板系统

这部分内容强烈建议通过 SSH 连接至虚拟机操作，通过 PVE web 端的【控制台（noVNC）】体验过于生草（比如不支持复制粘贴），严重不推荐。如果因为某些原因，只能通过网页后台来连接，那么建议点击右上角的“控制台”按钮旁的下拉三角，选择【xterm.js】来使用，这里至少可以正常的复制粘贴内容
- 在创建虚拟机时如果没有添加 `串行端口` ，或者 `串行端口` 的序号不为 `0` ，那么是没法使用【xterm.js】的

闲话少说，首先配置 `sudo` 命令免密码，这样避免每次使用 `sudo` 命令时都需要输入前面在 cloud-init 中配置的用户密码：

```shell
sudo tee /etc/sudoers.d/$USER <<< '$USER ALL=(ALL) NOPASSWD: ALL'
```

修改一下 SSH 配置文件，显式地禁止 root 用通过 SSH 登录，顺便关闭普通用户的密码登录，仅允许密钥登录，提升安全性（如果上文在 PVE 中配置 cloud-init 时配置了 SSH 密钥，这里应该默认已经禁止了密码登录：

```shell
sudo vim /etc/ssh/sshd_config
# 删除下面这行前面的注释符号，并确认末尾的值为 no
#PermitRootLogin no
# 检查下面这行，前面不能有注释符号，并确认末尾的值为 no
#PasswordAuthentication no
# 最后重启 sshd 服务，使修改后的配置生效
sudo systemctl restart sshd
```

### 系统时间配置

配置时区：

```shell
sudo timedatectl set-timezone Asia/Hong_Kong
```

默认情况下 Debian 12 应该已包含了 systemd-timesyncd 软件包，并开启了时钟同步，可以通过 `timedatectl status` 这个命令来确认（输出中 `System clock synchronized` 值为 `yes` ）。

### 安装 qemu 代理

不安装的话在 PVE 网页中无法方便的管理虚拟机，比如网页中无法操作重启，无法查看虚拟机的 IP 信息等，建议安装：

```shell
sudo apt update && sudo apt install qemu-guest-agent
```

启用代理：

```shell
sudo systemctl enable --now qemu-guest-agent
```

现在在 PVE 网页后台应该能看到这台虚拟机的 IP 信息了。

### 配置 zsh 和 oh-my-zsh

所以接下来先更新一下系统以及自带软件包，并安装系统基础软件：

```shell
sudo apt update && sudo apt upgrade -y && sudo apt install -y git htop neofetch
```

安装并配置 zsh（通过 oh-my-zsh）：

```shell
# 安装 zsh
sudo apt update && sudo apt install -y zsh
# 安装 oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# 配置主题（ys），并重载配置使其生效
sed -i '/^ZSH_THEME=/c\ZSH_THEME="ys"' ~/.zshrc && source ~/.zshrc
# 安装插件
## zsh-syntax-highlighting（代码高亮）
git clone https://github.com/zsh-users/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
## zsh-autosuggestions（自动建议）
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
## zsh-completions（自动补全）
git clone https://github.com/zsh-users/zsh-completions $ZSH_CUSTOM/plugins/zsh-completions && echo "fpath+=$ZSH_CUSTOM/plugins/zsh-completions/src" >> ~/.zshrc
# 启用插件，并重载配置使其生效
sed -i '/^plugins=/c\plugins=(sudo zsh-autosuggestions zsh-syntax-highlighting)' ~/.zshrc && sed -i '/source $ZSH\/oh-my-zsh.sh/d' ~/.zshrc && echo 'source $ZSH/oh-my-zsh.sh' >> ~/.zshrc && source ~/.zshrc
# 关闭 oh-my-zsh 自动更新，避免阻塞启动
sed -i "/omz:update.*disabled/s/^#\s*//" ~/.zshrc
```

oh-my-zsh 插件注意事项：
1. ~~根据 zsh-syntax-highlighting 插件的 [安装说明](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md#in-your-zshrc) ， `.zshrc` 文件中所有的 `source` 命令都要放在文件的末尾~~ 此为误读，仅在未使用 oh-my-zsh 管理插件时，`source ./zsh-syntax-highlighting/zsh-syntax-highlighting.zsh` 这行需要在 `.zshrc` 文件末尾
2. 根据 zsh-completions 插件的 [安装说明](https://github.com/zsh-users/zsh-completions#oh-my-zsh) ， `.zshrc` 文件中插入的 `fpath+=$ZSH_CUSTOM/plugins/zsh-completions/src` 命令要出现在原有的 `source "$ZSH/oh-my-zsh.sh"` 之前
3. 综上，在上面的命令最后在 `source ~/.zshrc` 之前，先将原本的 `source $ZSH/oh-my-zsh.sh` 移动到了文件末尾
4. 默认关闭了 oh-my-zsh 的自动更新，有需要时可以执行 `omz update` 手动更新

### 配置 UFW 防火墙

安装 UFW 防火墙：

```shell
sudo apt update && sudo apt install ufw
```

此时需要再手动安装一下 linux-headers 软件包，否则后续开启防火墙时，会因为缺少依赖的内核模块导致防火墙无法正常放行端口（这个问题俺在 `debian-12-generic-amd64-20230723-1450.qcow2` 镜像中遇到了，而且如果在创建虚拟机时未扩容硬盘，那么此时会因为硬盘容量不足，安装失败，此时可以扩容后手动重启一下虚拟机，会自动生效）：

```shell
sudo apt install linux-headers-$(uname -r)
```

安装好后，先确认一下防火墙状态，应该是 `inactive` ：

```shell
sudo ufw status
```

由于 UFW 默认的规则为【拒绝全部端口入站，允许全部端口出站】，因此需要手动允许 SSH 端口入站，避免开启防火墙后 SSH 链接被断开：

```shell
sudo ufw allow 22/tcp
```

开启防火墙，注意此时会提示 SSH 连接可能会断开，输入 `y` 确认即可：

```shell
sudo ufw enable
```

UFW 基础操作：

```shell
# 允许端口入站
sudo ufw allow 22
sudo ufw allow 22/tcp
sudo ufw allow 22:29/tcp
# 查看防火墙状态（简略）
sudo ufw status
# 查看防火墙状态（详细）
sudo ufw status verbose
# 查看防火墙状态（在每行前面增加序号）
sudo ufw status verbose
# 删除规则
sudo ufw delete <序号>
```

### 安装 docker （可选）

安装依赖：

```shell
sudo apt update && sudo apt install -y ca-certificates curl gnupg
```

添加 docker 官方源：

```shell
# 导入 docker 官方的 GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
# 配置 docker 官方仓库
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 刷新 apt 缓存
sudo apt update
```

安装 docker 及 compose 插件：

```shell
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

配置 docker 免 sudo（可选）：

```shell
# 新增 docker 用户组
sudo groupadd docker
# 将当前用户加入 docker 用户组
sudo usermod -aG docker $USER
# 使用户组的改动生效
newgrp docker
# 查看 docker 版本，验证改动生效
docker version
```

修改 docker 默认配置：

```shell
# 限制日志文件大小，并开启 ipv6
sudo bash -c 'cat > /etc/docker/daemon.json' << EOF
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "20m",
    "max-file": "3"
  },
  "ipv6": true,
  "fixed-cidr-v6": "fd00:dead:beef::/64",
  "experimental":true,
  "ip6tables":true,
  "default-address-pools": [
    {
      "base": "172.17.0.0/16",
      "size": 24
    },
    {
      "base": "fd00:dead:beef:100::/80",
      "size": 112
    }
  ]
}
EOF
```

重载 docker 配置文件，并重启 docker 服务：

```shell
sudo systemctl reload docker && sudo systemctl restart docker
```

允许 docker 开机自启动：

```shell
sudo systemctl enable docker
```

### 清理善后

执行 `sudo dmesg | grep cfg80211` 看看结果中有没有包含这一行：

```shell
cfg80211: failed to load regulatory.db
```

没有的话可以直接跳到下一段，清理缓存后关机了。如果有，那么需要额外安装 `iw` 软件包：

```shell
sudo apt install -y iw
```

需要安装的软件包全部安装完毕后，可以清理一下 apt 的缓存，毕竟是作为模板使用的虚拟机，硬盘空间尽量精简一些，加快后续克隆新虚拟机时的速度，克隆出来的虚拟机可以自行扩容硬盘后再添加其他软件。

```shell
sudo apt clean
sudo apt autoclean
sudo apt --purge autoremove
```

至此，准备用作模板的虚拟机已配置完成。

在 PVE 管理后台点击“关机”，然后在左侧的虚拟机列表中右键点击这台虚拟机，选择“转换为模板”。这样，一个基于 cloud-init 的虚拟机模板即制作完成了。

模板基于 [debian-12-generic-amd64-20230723-1450](https://cloud.debian.org/images/cloud/bookworm/20230723-1450/debian-12-generic-amd64-20230723-1450.qcow2) ，其中安装好了以下软件包（需要包含哪些软件包可以参考上文的安装过程自行定义，丰俭由人）：
- git
- htop
- neofetch
- qemu-guest-agent
- zsh
- oh-my-zsh
- ufw
- linux-headers-6.1.0-21-amd64
- iw
	- 解决开机时的 `cfg80211: failed to load regulatory.db` 报错
- docker
	- 可选安装
	- 俺个人的选择是：不在模板中预装，只在需要的虚拟机中按照上文的操作，手动安装

## 02. 使用模板

### 克隆新的虚拟机

右键单击模板，选择“克隆”：
- 模式
	- 两种模式的区别
		- `链接克隆` ：类似快照，磁盘上仅存储【克隆后的副本相对于模板发生变化的部分】，克隆后的虚拟机无法使用迁移功能，但优势是占用的磁盘空间较小
		- `完整克隆` ：直接将模板对应的虚拟机磁盘文件全量复制一份，虚拟机中的任何改动，只会存储在自己的虚拟磁盘文件中，可以在 PVE 集群中自由地迁移
	- 如果 PVE 宿主机只有一台，即 PVE 是单节点模式部署的，那么此处可以随意选择
	- 如果 PVE 宿主机不止一台，且组成了集群，有节点迁移和高可用的需求，那么此处最好选择 `完整克隆`
- VM ID
	- 可以自定义，也可以使用 PVE 默认的自增 ID
- 名称
	- 需要自己定义一个，开机后，cloud-init 会使用虚拟机名称作为虚拟机的 hostname

复制完成后，在开机前，最好进入虚拟机的【硬件】页面，自定义一下虚拟机的硬件配置：
- 内存，处理器，网络等
	- 可以随意按需修改
- 硬盘
	- 最好扩容一下，因为按照上文配置的虚拟机模板，硬盘只有 3 GB，一般来说是不够用的

### 开机

得益于模板是基于 Debian 的 cloud image 制作的，机器开机后会自动按照 cloud-init 的配置调整系统，比如生成各种需要随机的 id 和 MAC 地址，配置网络等，一般来说系统开机后就可以直接使用了。
- 开机后，虽然在系统内通过 `hostname` 命令查看主机名，显示确实是虚拟机名称，但在路由器的 DHCP 服务端看来，不知为何依然是和模板相同的 hostname
	- 解决方法倒是很简单，将虚拟机重启，即可对外展示正确的 hostname 了
- 如果开机前忘记扩容硬盘了，那么此时也可以进行扩容
	- 在 PVE 网页后台扩容完成后，重启机器，系统会自动识别并使用扩展后的硬盘容量，可以执行 `df -h` 确认

---

*参考链接：*
1. [Proxmox Virtual Environment 使用指南 - This Cute World](https://thiscute.world/posts/proxmox-virtual-environment-instruction/#%E6%8B%93%E5%B1%95---cloudinit-%E9%AB%98%E7%BA%A7%E9%85%8D%E7%BD%AE)
2. [[ Proxmox 折腾手记 ] PVE创建模板虚拟机 - 哔哩哔哩](https://www.bilibili.com/read/cv19960114/)
3. [佛西博客 - 在Proxmox VE pve里使用cloud-init 构建（centos\ubuntu\debian）cloud images](https://foxi.buduanwang.vip/virtualization/pve/388.html/)
4. [Linux VPS 服务器基础安全设置 - P3TERX ZONE](https://p3terx.com/archives/improve-linux-server-security.html)
5. [打造 Windows 10 下最强终端方案：WSL + Terminus + Oh My Zsh + The Fuck - P3TERX ZONE](https://p3terx.com/archives/the-strongest-terminal-solution-under-windows-10.html)
6. [Debian/Ubuntu 中安装和配置 UFW（简单防火墙） - P3TERX ZONE](https://p3terx.com/archives/installing-and-configuring-ufw-in-debian.html)
7. [cfg80211: failed to load regulatory.db | Proxmox Support Forum](https://forum.proxmox.com/threads/cfg80211-failed-to-load-regulatory-db.137298/#post-658457)
