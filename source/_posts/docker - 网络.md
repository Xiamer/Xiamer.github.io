
---
title: docker - 网络
date: 2021-02-20 15:11:10
categories: 
- 运维
tags:
- 部署 docker
---
## 理解Docker0

查看本地网卡： `ip addr`

![docker](/images/docker/net-1.png)

> 问题:docker 是怎样处理网络访问的?

![docker](/images/docker/net-2.png)

```bash
#开启一个tomcat容器,端口随机
root@xiamer:~# docker run -d -P --name tomcat01 tomcat

#查看容器内部ip地址 ip addr  ,发现容器启动的时候会得到一个内部ip地址 eth0@if65 ,这个是docker分配的
root@xiamer:~# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
64: eth0@if65: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever

#思考:Linux能不能ping通容器内部的这个ip?
root@xiamer:~# ping 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.061 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.049 ms

#Linux可以ping 通 docker 容器内部
```

**原理:**
1. <font>我们每安装一个docker 容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡docker0，采用的是桥接模式，使用的技术是  veth-pair  技术</font>，在主机查看网卡信息：多了一对  65  64 网卡

```bash
root@xiamer:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:16:3e:16:25:20 brd ff:ff:ff:ff:ff:ff
    inet 172.31.106.145/20 brd 172.31.111.255 scope global dynamic eth0
       valid_lft 315188155sec preferred_lft 315188155sec
    inet6 fe80::216:3eff:fe16:2520/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:a3:4b:1b:fe brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a3ff:fe4b:1bfe/64 scope link 
       valid_lft forever preferred_lft forever
65: vethcb4a334@if64: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 1e:f9:86:06:14:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::1cf9:86ff:fe06:1402/64 scope link 
       valid_lft forever preferred_lft forever
```

2. 再启动一个容器测试，又多了一对 67-66网卡

```bash
root@xiamer:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:16:3e:16:25:20 brd ff:ff:ff:ff:ff:ff
    inet 172.31.106.145/20 brd 172.31.111.255 scope global dynamic eth0
       valid_lft 315188027sec preferred_lft 315188027sec
    inet6 fe80::216:3eff:fe16:2520/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:a3:4b:1b:fe brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 brd 172.18.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:a3ff:fe4b:1bfe/64 scope link 
       valid_lft forever preferred_lft forever
65: vethcb4a334@if64: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 1e:f9:86:06:14:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::1cf9:86ff:fe06:1402/64 scope link 
       valid_lft forever preferred_lft forever
67: veth781ac81@if66: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether aa:01:41:a1:b6:87 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::a801:41ff:fea1:b687/64 scope link 
       valid_lft forever preferred_lft forever
```

**我们发现这个容器的网卡，都是一对一对的 veth-pair 技术一对的虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连**

正因为有这个特性，因此 让 veth-pair 充当一个桥梁，连接各种虚拟网络设备的OpenStack ，docker容器之间的连接，ovs的连接，都是使用veth-pair技术.

3. 我们来测试下 tomcat01 和 tomcat02 是否可以ping通

```bash
# tomcat01  172.18.0.2
# tomcat02  172.18.0.3

# tomcat01 ping tomcat02
root@xiamer:~# docker exec -it tomcat01 ping 172.18.0.3
PING 172.18.0.3 (172.18.0.3) 56(84) bytes of data.
64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.085 ms
64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.062 ms
64 bytes from 172.18.0.3: icmp_seq=3 ttl=64 time=0.070 ms

# tomcat02 ping tomcat01
root@xiamer:~# docker exec -it tomcat02 ping 172.18.02
PING 172.18.02 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.058 ms
64 bytes from 172.18.0.2: icmp_seq=2 ttl=64 time=0.063 ms
64 bytes from 172.18.0.2: icmp_seq=3 ttl=64 time=0.062 ms

#结论:
容器之间是可以互相ping通的
```

绘制一个网络模型图

![docker](/images/docker/net-3.png)

结论: tomcat01 和 tomcat02 是公用的同一个路由器：docker0.
所有的容器不指定网络的情况下，都是docker0 路由的，docker会给我们的容器分配一个默认的可以ip

> 00000000.00000000.00000000.00000000
> 255.255.0.1/16 域 局域网 -> （255 * 255 - 255.255.0.0 - 255.255.255.255）个
> 255.255.0.1/24 域 局域网 -> （255 - 255.255.0.0）个


### 小节

docker使用的是Linux的桥接，宿主机中是一个docker容器的网桥  docker0

![docker](/images/docker/net-4.png)

Docker 中的所有网络接口都是虚拟的，虚拟的转发效率很高。（内网传递文件）


### --link(不推荐使用)

我们编写了一个微服务，database url=ip: ，我们希望项目不重启，但是数据库ip可以换，那怎样处理这个问题？可以用名字进行访问服务

```bash
root@xiamer:~# docker exec -it tomcat01 ping tomcat02
ping: tomcat02: Name or service not known

#如何解决? 在启动服务的时候加上 --link
root@xiamer:~# docker run -d -P --name tomcat03 --link tomcat02 tomcat
57b83d42d6cd8b09e965f34c7969175ca4227e2591f6b9814709d5e505d504b7

root@xiamer:~# docker exec -it tomcat03 ping tomcat02              #tomcat03成功ping通tomcat02    
PING tomcat02 (172.18.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.18.0.3): icmp_seq=1 ttl=64 time=0.095 ms
64 bytes from tomcat02 (172.18.0.3): icmp_seq=2 ttl=64 time=0.061 ms

#但是tomcat02 ping不通tomcat03
root@xiamer:~# docker exec -it tomcat02 ping tomcat03 
ping: tomcat03: Name or service not known
```

探究: `docker network ls` -> `docker network inspect [网卡id]`  查看网卡的各种信息

```bash
[
    {
        "Name": "bridge",
        "Id": "631d93103872ffd1b9494a2bcdfdd5d52a165d3cd06409b9bcc7c0be1750bcba",
        "Created": "2021-05-31T15:24:09.144381839+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                  # docker0
                    "Subnet": "172.18.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "077e4b60ca4d643197decff60d0ca1a12008016968bbede04323eed2010ddce1": {
                "Name": "tomcat01",
                "EndpointID": "f537e0628474d61887bc51d515fd95b049b333feee1d5c4e30591147caea8ac3",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "7d2e829b52a10a49fd7d4c0d47dc7d72a6467f933d222d3eb66040c23da97222": {
                "Name": "tomcat02",
                "EndpointID": "b95b387fb40569ed43c6bd982c0edd941413de80ca2f1951acc5d32a0fdb985b",
                "MacAddress": "02:42:ac:12:00:05",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "57b83d42d6cd8b09e965f34c7969175ca4227e2591f6b9814709d5e505d504b7": {
                "Name": "tomcat03",
                "EndpointID": "56b6f46cfbbd7300e8cc3a97e12bfbf0b5d4aac63a3603cf74fd9c52d19b9925",
                "MacAddress": "02:42:ac:12:00:06",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

其实这个tomcat03 就是在本地配置了tomcat02 的网络配置(hosts)

```bash
# 查看 tomcat03 中的hosts配置
root@xiamer:~# docker exec -it tomcat03 cat /etc/hosts
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.18.0.3	tomcat02 7d2e829b52a1 #注意看,增加了tomcat02的映射
172.18.0.4	57b83d42d6cd
```

<font>--link  就是我们在hosts配置</font>中增加了一个 172.17.0.3  tomcat02 aaefe7048c1a 
我们现在玩docker,已经不建议使用--link了，不使用docker0，
docker0问题：他不支持容器名连接访问
我们都使用自定义网络，比较方便

## 自定义网络(推荐)

查看所有的docker网络：`docker network ls`

```bash
[root@xiamer ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
631d93103872   bridge    bridge    local
3f1f8fdbcabb   host      host      local
12bd53c050c3   none      null      local
```
### 网络模式

bridge : 桥接模式  docker(默认,自己创建也是 bridge 模式)
none   : 不配置网络 
host    :  和宿主机共享网络
container  : 容器网络连通  (用的少！局限很大)

**测试**
```bash
# 我们直接启动的命令  --net bridge (不加网络配置,就是使用这个默认的) ,而这个就是我们的docker0
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat01 --net bridge tomcat

#docker0 特点: 默认域名不能访问,但--link可以打通

#我们可以自定义一个网络
```

1. 创建一个网络 `docker network create`
```bash
# 创建自定义网络:
# --subnet 192.168.0.0/16    子网范围定义
# --gateway 192.168.0.1      网关
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

#创建成功:
root@xiamer:~# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
631d93103872   bridge    bridge    local
3f1f8fdbcabb   host      host      local
cfbab2fa3513   mynet     bridge    local
12bd53c050c3   none      null      local

#查看一下自己配的网络
root@xiamer:~ # docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "cfbab2fa3513e754c8e37d533ee62a95cc22881bb7a88c112a15d56fdd5a7ca6",
        "Created": "2021-06-10T01:58:24.710395508+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

2. 开启两个tomcat，开启时使用自定义的网络mynet

```bash
[root@xiamer ~]# docker run -d -P --name tomcat-net-01 --net mynet  tomcat
ec2e5b851d116806e1a6075fede8ced56a0e010e57cfeb9948a3b62b1a092fa1
[root@xiamer ~]# docker run -d -P --name tomcat-net-02 --net mynet  tomcat
c997e35c4e16d991f63ea30a9cdb24b80ec142be49086a5eaab49c8b933dfa5d
```
3. 查看一下 mynet的信息 `docker network inspect mynet`

```bash
[
    {
        "Name": "mynet",
        "Id": "cfbab2fa3513e754c8e37d533ee62a95cc22881bb7a88c112a15d56fdd5a7ca6",
        "Created": "2021-06-10T01:58:24.710395508+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "c997e35c4e16d991f63ea30a9cdb24b80ec142be49086a5eaab49c8b933dfa5d": {
                "Name": "tomcat-net-02",
                "EndpointID": "d8e6e1670b675fd9ae460ff7296a5eb94900fb2bcde113ec5cd7443c24673cfc",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "ec2e5b851d116806e1a6075fede8ced56a0e010e57cfeb9948a3b62b1a092fa1": {
                "Name": "tomcat-net-01",
                "EndpointID": "a3a0260a6f614cfff20400115a245ff14f9f5b3d169b206ac67820a196354e67",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

4. 开始ping

```bash
[root@xiamer ~]# docker exec -it tomcat-net-01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.061 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.076 ms
^C
--- 192.168.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.061/0.068/0.076/0.011 ms
[root@xiamer ~]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.046 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.066 ms
^C
--- tomcat-net-02 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.046/0.056/0.066/0.010 ms
```

可以看见，不管是使用ip地址还是使用服务名称，都可以找到对应的主机（192.168.0.3）
我们自定义的网络docker都已经帮我们维护好了对应的关系，推荐我们平时这样使用网络

好处:
MySQL 不同的集群使用不同的网络，保证集群是安全和健康的
Redis 不同的集群使用不同的网络，保证集群是安全和健康的

## 网络连通

![docker](/images/docker/net-5.jpeg)

不同网卡不能直接连接（ping不到）
但是容器和网卡可以连通（ping）

使用 `docker network connect`

```bash
[root@xiamer ~]# docker network connect --help

Usage:  docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network

Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
```

```bash
#测试打通 tomcat01  到 mynet
root@kylin:~# docker  network connect mynet tomcat01

root@kylin:~# docker  network inspect mynet
```

![docker](/images/docker/net-6.png)

可以看到，docker 直接把 tomcat01 的网络加入到了 mynet 中
官方说法：一个容器两个ip，类似于阿里云的公网ip和私网ip

测试: 分别使用 **tomcat01(打通)** 和 **tomcat02(没打通)**  测试

```bash
# 01 打通
[root@xiamer ~]# docker exec  -it tomcat01 ping tomcat-net-01
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.062 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.074 ms
# 02 打不通
[root@xiamer ~]# docker exec  -it tomcat02 ping tomcat-net-01
ping: tomcat-net-01: Name or service not known
[root@xiamer ~]# docker exec  -it tomcat01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.078 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.080 ms
```

结论: 假设要跨网络操作其他容器，就需要使用`docker network connect`连通 ！。。。

## 参考链接

* [docker 狂神视频](https://www.bilibili.com/video/BV1og4y1q7M4?p=36)
* [docker 狂神笔记](https://blog.csdn.net/qq_21197507/article/details/115071715)
* [docker 狂神笔记](https://www.yuque.com/vipkylin/hv5t54/fxgdw8#1s16b)
