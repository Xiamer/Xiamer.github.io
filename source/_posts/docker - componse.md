
---
title: docker - componse
date: 2021-02-25 15:11:10
categories: 
- 运维
tags:
- 部署 docker
---

## Compose 简介

Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

Compose 使用的三个步骤：

1. 使用 `Dockerfile` 定义应用程序的环境。

2. 使用 `docker-compose.yml` 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。

3. 执行 `docker-compose up` 命令来启动并运行整个应用程序。

**作用：批量容器编排**

dockerfile 让程序在任何地方运行。web 服务、Redis、MySQL、nginx……多个容器
即使有100个服务，只要文件没问题，就可以一键上线

官方`docker-compose.yml` 编写案例:

```bash
version: '2.0'
services:
  web:
    build: .
    ports:
    - "5000:5000"
    volumes:
    - .:/code
    - logvolume01:/var/log
    links:
    - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

Compose：重要概念
• 服务services ：容器、应用（web、Redis、MySQL……）
• 项目project：一组关联的容器。博客（web、mysql、wp）


## Compose 安装

1. 下载(加速:http://get.daocloud.io/)

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

#可能有点慢,用下面这个也行
curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

2. 授权并测试

```bash
chmod +x /usr/local/bin/docker-compose

#测试
docker-compose version

docker-compose version 1.26.2, build eefe0d31
docker-py version: 4.2.2
CPython version: 3.7.7
OpenSSL version: OpenSSL 1.1.0l  10 Sep 2019
```


## 体验

地址：https://docs.docker.com/compose/gettingstarted/

python应用。 计数器。redis！

1. Create a directory for the project:
```bash
 mkdir composetest
 cd composetest
```

2. Create a file called app.py in your project directory and paste this in: 
> redis 链接的host 写的redis，而不是ip。是因为docker Compose 会将容器放在同一个网络（桥接模式）上。

```bash
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```
3. Create another file called requirements.txt in your project directory and paste this in:
```bash
flask
redis
```

4. Dockerfile 应用打包为镜像

```bash
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
# 官网的用来flask框架，我们这里不用它
# 这告诉Docker
# 从python3.7开始构建镜像
# 将当前目录添加到/code印像中的路径中
# 将工作目录设置为/code
# 安装Python依赖项
# 将容器的默认命令设置为python app.py
```

5. Docker-compose yaml文件（定义整个服务，需要的环境 web、redis） 完整的上线服务！
```bash
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

6. 启动compose 项目 （docker-compose up）

**docker 小节**

1. Docker 镜像， run -> container
2. Dockerfile 构建镜像(服务打包)
3. docker-componse启动项目(编排、多个微服务/环境)
4. Docker 网络！(docker network ls)

## yml 规则

docker-componse.yml 核心
https://docs.docker.com/compose/compose-file/#compose-file-structure-and-examples

```bash
# 3层
version: '' # 1.版本
service: '' # 2.服务
  服务1 web
    # 服务配置
    images
    build
    network
  服务2 redis
    ...
  服务3 redis
    ...
#3. 其他配置 网络/卷 数据规则
volumes: ''
networks: ''
config: ''
```

## NUXT docker componse 例子

![docker](/images/docker/componse-1.png)


1.  在 app(nuxt应用)文件夹下创建 Dockerfile

```shell
FROM node:10.7

ENV APP_ROOT /src

RUN mkdir ${APP_ROOT}
WORKDIR ${APP_ROOT}
ADD . ${APP_ROOT}

RUN npm install
RUN npm run build

ENV HOST 0.0.0.0
```
2. 更目录创建 `docker-compoose.yml`

```shell
services:
  nuxt:
    # 寻找dockerfile
    build: ./app/
    container_name: vuevixens-website
    restart: always
    ports:
      - "3333:3333"
    command:
      "npm run start"

  nginx:
    image: nginx:1.13
    container_name: vuevixens-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx:/etc/nginx/conf.d
    depends_on:
      - nuxt
```

3. nginx default.conf 配置

```shell
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://vuevixens-website:3333;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

4.  docker-compose up --build -d

![docker](/images/docker/componse-2.png)


## 参考链接

* [nuxtjs docker](https://github.com/MattGould1/nuxtjs-docker/blob/master/src/Dockerfile)
* [nuxtjs docker demo](https://dev.to/frontendfoxes/dockerise-your-nuxt-ssr-app-like-a-boss-a-true-vue-vixens-story-4mm6)
* [docker 狂神视频](https://www.bilibili.com/video/BV1og4y1q7M4?p=36)
* [docker 狂神笔记](https://blog.csdn.net/qq_21197507/article/details/115071715)
