---
title: RancherOS 初体验
date: 2017-08-17 20:12:50
categories: Docker
tags: 
	- Docker
	- Linux
permalink: rancher-free-style
---

**先来欣赏一张 RancherOS 的系统架构图**

![rancheros](https://github.com/rancher/os/blob/master/docs/rancheros.png?raw=true)

### 序

> 第一次听说 RancherOS 这个东西是在 OSChina 的软件更新上看到的，标题是《基于 Docker 的操作系统 RancherOS》，一听正和我意，集成最新的 Linux4.x 内核，包含最少运行 Docker 所需要的软件，二进制包20M(我去下载的时候是1.0.4版本，大小已经59M 了，不过比起动辄几百 M 的其他 Linux OS，已经很小了)；
    
> 今天偶有兴致，准备拿 RancherOS 作为 Docker 容器的宿主机系统来玩玩儿，同时对安装使用的流程进行了整理和理解。

<!-- more -->

### 下载

官网：https://github.com/rancher/os

当前版本：`v1.0.4 - Docker 17.03.1-ce - Linux 4.9.40`

ISO 镜像下载地址：https://releases.rancher.com/os/latest/rancheros.iso

### 安装

> The RancherOS ISO file can be used to create a fresh RancherOS install on KVM, VMware, VirtualBox, or bare metal servers. 

RancherOS 的 ISO 镜像适用于 KVM，VMware，VirtualBox 或者物理主机。

> You must boot with at least 512MB of memory. If you boot with the ISO, you will automatically be logged in as the rancher user. Only the ISO is set to use autologin by default. If you run from a cloud or install to disk, SSH keys or a password of your choice is expected to be used.

启动至少需要512M 的内存，如果你直接从 ISO 启动（数据全部存储在内存里面，不占用硬盘空间，关机后数据释放，不会保留操作数据），你将会自动登录到`rancher`这个用户，只有从 ISO 启动才会默认自动登录；如果你运行在云主机上或者从硬盘启动，你可以使用 SSH Keys 进行远程连接（从硬盘启动后，rancher 用户在宿主机将无法登录，只能通过 SSH key 进行远程登录）。

---

我这里当然需要安装到硬盘进行使用，数据还是要保留的；

#### 安装到硬盘

> 上面已经讲了从 ISO 启动了 RancherOS，默认登录用户和密码都是`rancher`;

**配置 Configuration**

现在我需要一个在自己的电脑上创建一个叫做`cloud-config.yml`的配置文件，里面的内容如下（这里的配置比较多，我直接贴上了我自己使用的完整配置），其实这个yml 文件就是启动加载文件，每次启动都会去执行里面的配置：

> A cloud-config file can be used to provide configuration when first booting RancherOS.

```yml
#cloud-config
hostname: ros-rmbp
rancher:
  docker:
    tls: true
  network:
    dns:
      nameservers:
      - 8.8.8.8
      - 8.8.4.4
  write_files:
    - container: ntp
      path: /etc/ntp.conf
      permissions: "0644"
      owner: root
      content: |
        server 0.cn.pool.ntp.org iburst
        server 1.cn.pool.ntp.org iburst
        server 2.cn.pool.ntp.org iburst
        server 3.cn.pool.ntp.org iburst

        # Allow only time queries, at a limited rate, sending KoD when in excess.
        # Allow all local queries (IPv4, IPv6)
        restrict default nomodify nopeer noquery limited kod
        restrict 127.0.0.1
        restrict [::1]

ssh_authorized_keys:
  - ssh-rsa AAAAB3Nza......(此处省略256个字符)
```

**上传配置文件**

然后通过 HTTP 下载到 RancherOS 上，我是在本机 Mac 上使用 python http 启动一个静态 Server，然后到 RancherOS 使用 wget 直接 Dowanload 下来，简单暴力；

接下来执行以下 Shell

```
$ sudo mv cloud-config.yml /var/lib/rancher/conf/
$ sudo ros install -c /var/lib/rancher/conf/cloud-config.yml -d /dev/sda

## 会有两次 Y/N 的确认，都输入 Y
## 重启后，就不能直接通过rancher 帐号登录了；
## 只能 通过 SSH Keys 远程登录 ssh rancher@IP
```
**安装必备的 Docker-Compose*

```
## 由于默认没有 curl command，先用 wget 代替
wget https://github.com/docker/compose/releases/download/1.15.0/docker-compose-Linux-x86_64

sudo mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```
**使用**

到目前为止，已经可以愉快的使用 RancherOS 了，试试 Docker

```
$ docker ps 
$ docker images
```

### 下篇开始写RancherOS 进阶


