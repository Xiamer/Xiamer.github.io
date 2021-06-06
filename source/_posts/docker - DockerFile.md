
---
title: docker - DockerFile
date: 2021-02-10 15:11:10
categories: 
- 运维
tags:
- 部署 docker
---
## DockerFile 介绍

dockerfile 是用来构建docker镜像的文件！命令参数交脚本！

构建步骤：

1. 编写dockerfile文件
2. docker build 构建一个镜像
3. docker run 运行镜像
4. docker push 发布镜像（DockerHub、阿里云镜像仓库）

## DockerFile 构建过程

**基础知识**

1. 每个保留关键字（指令）都必须是大写字母
2. 执行从上到下顺序执行
3. #表示注释
4. 每一个指令都会创建提交一个新的镜像层，并提交！

![docker](/images/docker/f-1.png)

**步骤：开发、部署、运维。。。缺一不可！**

DockerFile：构建文件，定义了一切的步骤，类似于源代码
DockerImages：通过Dockerfile构建生成的镜像，就是我们最终发布运行的产品
Docker容器：容器就是镜像运行起来提供服务


## Dockerfile 指令

![docker](/images/docker/f-2.png)

```bash
FROM                # 基础镜像,一切从这开始构建
MAINTAINER          # 镜像是谁写的,姓名+邮箱
RUN                 # 镜像构建的时候需要运行的命令
ADD                 # 假设构建tomcat镜像,这个tomcat压缩包,必须要添加进去吧. 用ADD添加的文件会自动解压
WORKDIR             # 镜像的工作目录
VOLUME              # 挂载的目录
EXPOSE              # 指定暴露端口		  
CMD                 # 指定这个容器启动的时候要运行的命令,只有最后一个会生效,可被替代
ENTRYPOINT          # 指定这个容器启动的时候要运行的命令,可以追加命令
ONBUILD             # 当构建一个被继承的 dockerfile , 这个时候就会运行ONBUILD的指令,触发指令
COPY                # 类似ADD，将我们的文件拷贝到镜像中
ENV                 # 构建的时候设置环境变量
```

## CMD 和 ENTRYPOINT 的区别

```pre
CMD               # 指定这个容器启动的时候要运行的命令可以追加命令，只有最后一个会生效，可被替代
ENTRYPOINT        # 指定这个容器启动的时候要运行的命令可以追加命令
```

## 实战：创建一个centos

1. 编写自己的dockerfile 文件 `mydockerfile-centos`

```bash
FROM centos
MAINTAINER  xiamer<xiaoguuang_10@qq.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUM yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH

CMD echo "--------end----------"

CMD /bin/bash
```

2. 通过文件构建镜像

```bash
# 命令:  docker build -f dockerfile路径  -t 镜像名:[TAG] .   

docker build -f mydockerfile-centos -t mycentos:0.1 .

#结果:
# Successfully built sd79ka03lde0
# Successfully tagged mycentos:0.1
```

3. 运行

```bash
#运行:
docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
mycentos              0.1                 sd79ka03lde0        6 minutes ago       295MB

docker run -it mycentos:0.1

```

4. 对比

我们增加之后的mycentos 多了 `net-tools` `vim`

5. 我们可以列出本地的镜像变更历史  docker history 镜像id

```bash
root@kylin:~# docker history  mycentos:0.1 
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
sd79ka03lde0        12 minutes ago      /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B                  
d8ad2lkdsad9        12 minutes ago      /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B                  
sa0dsak22kl2        12 minutes ago      /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B                  
asd90232ds02        12 minutes ago      /bin/sh -c #(nop)  EXPOSE 80                    0B                  
ds90dal2okd0        12 minutes ago      /bin/sh -c yum -y install net-tools             22.8MB              
dds090dskk2l        12 minutes ago      /bin/sh -c yum -y install vim                   57.2MB              
ds0w0kd0alx0        12 minutes ago      /bin/sh -c #(nop) WORKDIR /usr/local            0B                  
23uodadnahiq0        12 minutes ago      /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B                  
9dha2dadzzlkq        12 minutes ago      /bin/sh -c #(nop)  MAINTAINER xiamer<xiaoguang_10@q…   0B                  
89ampgadija82m        7 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B                  
<missing>           7 weeks ago         /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B                  
<missing>           7 weeks ago         /bin/sh -c #(nop) ADD file:84700c11fcc969ac0…   215MB               
```

通过docker history 这个命令可以查看其它官方镜像的构建过程

## 发布镜像到阿里云

1. 登录阿里云
2. 找到容器镜像服务
3. 创建命名空间（一个账号只能创建三个命名空间）
4. 创建一个镜像仓库（默认私有）
5. 阿里云给的官方教程

![docker](/images/docker/f-3.png)








## 参考链接

* [docker docs](https://docs.docker.com/get-docker/)
* [docker 狂神笔记](https://www.yuque.com/vipkylin/hv5t54/ixkugt#0I9B9)
