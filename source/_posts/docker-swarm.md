
---
title: docker - swarm (集群)
date: 2021-02-28 15:11:10
categories: 
- 运维
tags:
- 部署 docker
---

## 购买服务器

购买4台服务器 配置在<font>同一个安全组</font>内，这样内网可以互ping，不需要流量。

## Swarm集群搭建

[工作机制](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)

![docker](/images/docker/swarm.jpeg)


```shell

docker swarm init --help
 
ip addr # 获取自己的ip（用内网的不要流量）
 
[root@xiamer ~]# docker swarm init --advertise-addr 172.16.250.97
Swarm initialized: current node (otdyxbk2ffbogdqq1kigysj1d) is now a manager.
To add a worker to this swarm, run the following command:
    docker swarm join --token SWMTKN-1-3vovnwb5pkkno2i3u2a42yrxc1dk51zxvto5hrm4asgn37syfn-0xkrprkuyyhrx7cidg381pdir 172.16.250.97:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

```

初始化结点docker swarm init

docker swarm join 加入一个结点！

>  获取令牌
> docker swarm join-token manager
> docker swarm join-token worker

把其他节点键入到sswarm 中。
可将2、3节点 worker加入，4 manager加入，构成双主双从。

## Raft协议

双主双从：假设一个结点挂了！其他结点是否可以用！
Raft协议：保证大多数结点存活才可以使用，只要>1, 集群至少大于3台！

测试：
双主双从
  若将docker1-manager机器停止。宕机！双主，另外一个结点也不能使用了！
  若将docker3-worker机器停止. 还可用。
3主1从
  若将1个主停止将不可用

十分简单：集群，可用！ 3个主（manager）节点。 > 1台管理结点存活！

Raft协议：保证大多数结点存活，才可以使用，高可用！

## docker service

弹性、扩缩容！集群！

以后告别 docker run！

docker-compose up！启动一个项目。单机！

集群： swarm docker-service

k8s service

容器 => 服务！

容器 => 服务！ => 副本！

redis => 10个副本！（同时开启10个redis容器）

体验：创建服务、动态扩容服务、动态更新服务

> docker run 容器启动！ 不具有扩缩容器
> docker service 服务！ 具有扩缩容器，滚动更新！
  
```shell
# 创建一个服务，当前的副本 随机分布在4个节点的某一个
docker service create -p 8080:80 --name my-nginx nginx

# 创建3个副本，分布在4个节点
docker service update --replicas 3 my-nginx

# 创建10个副本，分布在4个节点，每一个节点可能分布不同个数的副本
docker service update --replicas 10 my-nginx

# 用 scale 和 update --replicas 一样，创建5个副本
docker service scale my-nginx=5
```

移出！docker service rm

只要会搭建集群、会启动服务、动态管理容器就可以了！


## 概念的总结

**swarm**

集群的管理和编号，docker可以初始化一个swarm集群，其他结点可以加入。（管理，工作者）

**Node**

就是一个docker结点，多个结点就组成了一个网络集群（管理、工作者）

**Service**

任务，可以在管理结点或者工作结点来运行。核心，用户访问。

**Task**

容器内的命令、细节任务！

![docker](/images/docker/swarm-2.png)

> service

![docker](/images/docker/swarm-3.jpeg)

命令 -> 管理 -> api -> 调度 -> 工作结点（创建Task容器维护创建！）

>服务副本和全局服务

![docker](/images/docker/swarm-4.jpeg)


## 其他命令

* Docker Stack

```shell
ocker-compose 单机部署项目
docker stack 集群部署
# 单机
docker-compose up -d wordpress.yaml
# 集群
docker stack deploy wordpress.yaml
```

* Docker Secret

安全！配置密码！证书！
```shell
[root@xiamer ~]# docker secret --help
 
Usage:  docker secret COMMAND
 
Manage Docker secrets
 
Commands:
  create      Create a secret from a file or STDIN as content
  inspect     Display detailed information on one or more secrets
  ls          List secrets
  rm          Remove one or more secrets
```

* Docker Config

```shell

配置！
[root@xiamer ~]# docker config --help
 
Usage:  docker config COMMAND
 
Manage Docker configs
 
Commands:
  create      Create a config from a file or STDIN
  inspect     Display detailed information on one or more configs
  ls          List configs

```

## 参考链接

* [docker 狂神视频](https://www.bilibili.com/video/BV1og4y1q7M4?p=36)
* [docker 狂神笔记](https://blog.csdn.net/qq_21197507/article/details/115071715)
