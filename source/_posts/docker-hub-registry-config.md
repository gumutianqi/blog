---
title: 替换 Docker-Hub 的镜像源
date: 2016-12-26 14:59:18
categories: Docker
tags: 
 - Docker
 - Linux
permalink: docker-hub-registry-config
---

> 以下文档说明来自于阿里云 Docker 平台帮助文档

#### 方法一：

**创建一台安装有Docker环境的Linux虚拟机，指定机器名称为default，同时配置Docker加速器地址。**

<!-- more -->

```
docker-machine create --engine-registry-mirror=https://xxxx.mirror.aliyuncs.com -d virtualbox default
```

查看机器的环境配置，并配置到本地。然后通过Docker客户端访问Docker服务。

```
docker-machine env default
eval "$(docker-machine env default)"
docker info
```

这里 xxxx 是您的专有加速器地址

#### 方法二：

**登录已创建的 Docker VM**

```
docker-machine ssh default
sudo vi /var/lib/boot2docker/profile
```

在EXTRA_ARGS中添加

```
--registry-mirror=https://xxxx.mirror.aliyuncs.com
```

重启Docker服务即可

```
sudo /etc/init.d/docker restart
```