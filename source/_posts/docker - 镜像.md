
---
title: docker - 镜像
date: 2021-02-07 15:11:10
categories: 
- 运维
tags:
- 部署
---

## 镜像是什么？

##Docker 镜像## 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 不包含 任何动态数据，其内容在构建之后也不会被改变。

### 如何等得到镜像

* 从远程下载
* copy 
* 制作一个 Dockerfile


## Docker镜像加载原理

### UnionsFS(联合文件系统)

UnionFs（联合文件系统）：Union文件系统（UnionFs）是一种分层、轻量级并且高性能的文件系统，他支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（ unite several directories into a single virtual filesystem)。Union文件系统是 Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

### Docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootlloader和kernel,bootfs主要是引导加载kernel,Linux刚启动时会加载bootfs文件系统,在docker镜像的最底层是bootfs,这一层与我们典型的Linux/Unix系统是一样的,包含boot加载器和内核,当boot加载完成之后整个内核就在内存中了,此时内存的使用权已由bootfa转交给内核,此时系统也会卸载bootfs


平时我们安装进虚拟机的CentOS都是好几个G，为什么Docker这里才200M？

对于个精简的OS,rootfs可以很小，只需要包合最基本的命令，工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供rootfs就可以了。由此可见对于不同的Linux发行版， boots基本是一致的， rootfs会有差別，因此不同的发行版可以公用bootfs.

## 分层理解

我们可以去下载一个镜像，注意观察下载的日志输出，可以看到是一层层的在下载

![docker](/images/docker/x6.png)

### 为什么Docker镜像要采用这种分层的结构呢？

**资源共享** 比如有多个镜像都从相同的Base镜像构建而来，那么宿主机只需在磁盘上保留一份base镜像，同时内存中也只需要加载一份base镜像，这样就可以为所有的容器服务了，而且镜像的每一层都可以被共享。

## 举例

所有的镜像都起始于一个基础镜像层,当进行修改或增加新的内容时,就会在当前镜像层之上,创建一个新的镜像层

假如基于Ubuntu Linux 16.64创建一个新的镜像,这就是新镜像的第一层,如果在该镜像中添加python包,就会在该镜像之上创建第二个镜像层; 如果继续添加一个安全补丁,就会创建第三个镜像层。

![docker](/images/docker/x7-1.png)
![docker](/images/docker/x7-2.png)
![docker](/images/docker/x7-3.png)
![docker](/images/docker/x7-4.png)

Docker镜像都是只读的,当容器启动时,一个新的可写层被加载到镜像的顶部!

这一层就是我们通常所说的容器层,容器之下的都叫镜像层

![docker](/images/docker/x7-5.png)


## 参考链接
* [docker docs](https://docs.docker.com/get-docker/)
* [狂神 docker](https://www.bilibili.com/video/BV1og4y1q7M4)
