---
title: docker - 基础
date: 2021-01-31 15:11:10
categories: 
- 运维
tags:
- 部署
---

## 什么是容器？

一句话概括容器：容器就是将软件打包成标准化单元，以用于开发、交付和部署。
 * 容器镜像是轻量的、可执行的独立软件包 ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
 * 容器化软件适用于基于Linux和Windows的应用，在任何环境中都能够始终如一地运行。
 * 容器赋予了软件独立性，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

### docker 为什么出现？

* 一款产品: 开发+上线 两套环境。应用环境 应用配置！
* 开发 ... 运维 问题： 本地可以，版本更新有问题
* 环境配置麻烦（nginx、node、pm2等等）

传统: 开发zip,运维来做.
现在: 开发打包镜像上线。

js - zip - 发布（应用商店）- 张三使用apk - 安装使用

js - node(环境) - 打包带上环境(镜像) - docker 仓库 商店 - 下载发布镜像运行即可。

## 虚拟机与容器化有什么区别？

![docker](/images/docker/x1.png)
![docker](/images/docker/x2.jpeg)

> 容器虚拟化的是操作系统而不是硬件，容器之间是共享同一套操作系统资源的。虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统。因此容器的隔离级别会稍低一些。

虚拟机缺点
* 资源占用多
* 冗余步骤多
* 启动慢


## docker 的组成

![docker](/images/docker/1.jpg)

### 镜像 image

Docker镜像就好比是面向对象过程中的类，可以通过这个类创建实例（容器），通过这个镜像可以创建多个容器。（最终运行在容器中）

### 容器 container 

Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建的.

### 仓库 repository

仓库存放镜像的地方
仓库分为共有仓库和私有仓库

## CentOS 安装 docker

```bash
#!/bin/bash
# remove old version
sudo yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
    docker-engine

# remove all docker data 
sudo rm -rf /var/lib/docker

#  preinstall utils 
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# add repository 
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo # 默认是国外的

sudo yum-config-manager \
  --add-repo \
    http://mirrors.aliyuncs.com/docker-ce/linux/centos/docker-ce.repo # 推荐阿里云 比较快

# make cache
sudo yum makecache fast

# install the latest stable version of docker docker-ce(社区版)
sudo yum install -y docker-ce docker-ce-cli containered.io


# 配置镜像加速器 通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxxhsese.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

# start deamon and enable auto start when power on
sudo systemctl start docker
sudo systemctl enable docker

# add current user 配置当前用户对docker的执行权限
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo systemctl restart docker
```

## docker 使用

```bash
# 查看版本
docker version

# 拉取镜像
# docker pull hello-word , docker run hello-word
docker run hello-word

## 卸载
yum remove -y docker-ce docker-ce-cli containered.io
rm -rf /var/lib/docker
```
## 底层原理

### Docker是怎么工作的?

Docker是一个C/S结构的系统,Docker的守护进程运行在主机上.通过Socket从客户端访问
DockerServer接收到Docker-Client的指令,就会执行这个命令
![docker](/images/docker/x3.png)

### 为什么Docker比Vm快?

1. docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。
2. docker利用的是宿主机的内核,而不需要Guest OS。

> GuestOS： VM（虚拟机）里的的系统（OS）
> HostOS：物理机里的系统（OS）

![docker](/images/docker/x4.png)

所以说,新建一个容器的时候,docker不需要像虚拟机一样重新安装一个操作系统内核,虚拟机是加载Guest OS,分钟级别的,而docker是利用宿主机的操作系统,省略了这个复杂的过程。


## 参考链接
* [docker docs](https://docs.docker.com/get-docker/)
* [狂神 docker](https://www.bilibili.com/video/BV1og4y1q7M4)
