---
title: docker - 常用命令
date: 2021-02-01 15:11:10
categories: 
- 运维
tags:
- 部署 docker
---
## docker 命令

### 帮助命令

```bash
docker version # 显示docker版本信息
docker info    # 显示docker的系统信息 包括镜像和容器的数量
docker [option] --help 
```

[文档](https://docs.docker.com/engine/reference/run/)

### 镜像命令

```bash
docker images                      # 查看本地镜像
docker search                      # 搜索镜像
docker pull                        # 搜索镜像
docker rmi -f $IMAGE_ID            # 删除本地镜像
docker rmi -f $(docker image -aq)  # 删除所有镜像
docker save $IMAGE_ID > 文件路径   # 保存镜像为离线文件
docker save -o 文件路径 $IMAGE_ID  # 保存镜像为离线文件
docker load < 文件路径             # 加载文件为docker镜像
docker load -i 文件路径            # 加载文件为docker镜像
docker tag $IMAGE_ID $NEW_IMAGE_NAME:$NEW_IMAGE_TAG  # 修改镜像TAG
docker history $IMAGE_ID           # 显示镜像每层的变更内容
```

### 容器命令

```bash
docker run [可选参数] $IMAGE_ID $CMD  # 运行一个镜像
  # -d  后台方式运行
  # -it 使用交互式运行，进入容器查看内容 docker run -it centos /bin/bash
  #     exit 退出容器并退出
  #     ctrl + p + q 容器退出不停止
  # -p  指定容易端口  -p 8080（主机端口）:8080(容器端口)

docker ps -a                         # 查看所有容器
docker rm -f $CONTAINER_ID           # 删除容器
docker rm -rf $(docker ps -aq)       # 删除所有容器
docker ps -a -q|xrags docker rm      # 删除所有容器

docker stop $CONTAINER_ID            # 停止docker容器
docker start $CONTAINER_ID           # 启动docker容器
docker restart $CONTAINER_ID         # 重启docker容器
docker kill $CONTAINER_ID            # 强制关闭docker容器
docker pause $CONTAINER_ID           # 暂停容器
docker unpause $CONTAINER_ID         # 恢复暂停的容器
docker rename $CONTAINER_ID          # 重新命名docker容器

docker port $CONTAINER_ID            # 查看container的端口映射
docker commit $CONTAINER_ID $NEW_IMAGE_NAME:$NEW_IMAGE_TAG         # 将容器保存为镜像
docker stats                         # 查看容器的资源使用情况
```

## 常用命令

```bash

docker inspect $CONTAINER_ID         # 查看container的容器属性，比如ip等等
docker inspect $IMAGE_ID            # 查看镜像详情
docker logs $CONTAINER_ID            # 查看docker容器运行日志，确保正常运行
  # docker -tf --tail 10 $CONTAINER_ID
docker top $CONTAINER_ID             # 查看容器中正在运行的进程

docker exit -it $CONTAINER_ID        # 进入容器、开启一个新的终端 可以在里面操作
docker attach $CONTAINER_ID          # 启动一个已存在的docker容器

docker cp $CONTAINER_ID /image_path /主机_path # copy
```

## run 相关命令

```bash
# -d，后台运行容器, 并返回容器ID；不指定时, 启动后开始打印日志, Ctrl+C退出命令同时会关闭容器
# -i，以交互模式运行容器, 通常与-t同时使用
# -t，为容器重新分配一个伪输入终端, 通常与-i同时使用
# --name container_name，设置容器名称, 不指定时随机生成
# -h container_hostname，设置容器的主机名, 默认随机生成
# --dns 8.8.8.8，指定容器使用的DNS服务器, 默认和宿主机一致
# -e docker_host=172.17.0.1，设置环境变量
# --cpuset="0-2" or --cpuset="0,1,2"，绑定容器到指定CPU运行
# -m 100M，设置容器使用内存最大值
# --net bridge，指定容器的网络连接类型, 支持bridge/host/none/container四种类型
# --ip 172.18.0.13，为容器指定固定IP（需要使用自定义网络none）
# --expose 8081 --expose 8082，开放一个端口或一组端口，会覆盖镜像设置中开放的端口
# -p [宿主机端口]:[容器内端口]，宿主机到容器的端口映射，可指定宿主机的要监听的IP，默认为0.0.0.0
# -P，注意是大写的, 宿主机随机指定一组可用的端口映射容器expose的所有端口
# -v [宿主机目录路径]:[容器内目录路径]，挂载宿主机的指定目录（或文件）到容器内的指定目录（或文件）
# --add-host [主机名]:[IP]，为容器hosts文件追加host, 默认会在hosts文件最后追加[主机名]:[容器IP]
# --volumes-from [其他容器名]，将其他容器的数据卷添加到此容器
# --link [其他容器名]:[在该容器中的别名]，添加链接到另一个容器，在本容器hosts文件中加入关联容器的记录，效果类似于--add-host
```

## 参考链接

* [docker docs](https://docs.docker.com/get-docker/)
