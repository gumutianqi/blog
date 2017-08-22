---
title: 配置CENTOS YUM更新源
date: 2017-08-22T09:42:50.000Z
categories:
  - Linux
tags:
  - Linux
  - CentOS
permalink: change-centos-yum-mirrors
---
{% asset_img centos-logo.png%}

### 前言

大家都知道CentOS 有个很方便的软件安装工具yum，但是默认安装完CentOS，系统里使用的是国外的CentOS更新源，这就造成了我们使用默认更新源安装或者更新软件时速度很慢的问题。

为了使用yum工具能快速的安装更新软件，我们需要将默认的yum更新源配置为国内的更新源。yum更新源配置文件位于CentOS目录 `/etc/yum.repos.d/` 下。

<!-- more -->

### 提供几个国内快速的更新源

**教育网资源**

> 上海交大： http://ftp.sjtu.edu.cn/centos/

服务器位于北京，中国教育网网络中心，下载速度高达十M。
北方用户与教育网用户推荐，速度飞快。
需要手动创建 CentOS-Base.repo文件。

> 中国科技大学：http://centos.ustc.edu.cn

服务器位于合肥。 南方用户推荐。 同样的，`CentOS`版本非常丰富，适合长期使用。

**非教育网资源**

> sohu的开源镜像服务器：http://mirrors.sohu.com/

服务器位于山东省联通

> 网易的开源服务器镜像： http://mirrors.163.com/centos

速度也不错，全国用户推荐
总之，大家在使用前可以 ping 一下一上更新源，看哪个快就用哪个。

### 配置源 ( 以 CentOS7 为例 )

> CentOS-Base.repo文件示例，这个文件在这个目录下  /etc/yum.repos.d/

```
[base]
name=CentOS-$releasever - Base
baseurl=http://mirrors.163.com/centos/7/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-6 

[updates]
name=CentOS-$releasever - Updates
baseurl=http://mirrors.163.com/centos/7/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-6

[addons]
name=CentOS-$releasever - Addons
baseurl=http://mirrors.163.com/centos/$releasever/addons/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras
baseurl=http://mirrors.163.com/centos/7/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

[centosplus]
name=CentOS-$releasever - Plus
baseurl=http://mirrors.163.com/centos/7/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
```

---

从以上配置文件可以看出，需要根据各家源情况 有选择的配置 [base]  [updates]  [addons]  [extras]  [centosplus]   这几项。

每一项只要修改`baseurl`和`gpgkey`为相应源地址即可。

以上配置结束之后，要清空yum缓存，并重建yum缓存，执行以下命令：

```
yum clean all && yum clean metadata && yum clean dbcache && yum makecache && yum update
```
