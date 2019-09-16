---
layout: post
title: Docker相关命令
category: Linux
tags: [docker]
keywords: docker
description:
---

Docker的相关命令
<!--more-->


## docker build

### 基本的build

`docker build -t hello:0.0.1 .`

### 指定Dockerfile

`docker build -t hello:0.0.1 -f dockerfilePath .`

## 运行docker

### 映射端口
`-P`会随机选取一个宿主端口来映射容器暴露的端口（`Dockerfile`中被`expose`，或者`docker run`中指定`--expose`）

-p 需要指明宿主端口和容器端口

`docker run -p 宿主端口:容器端口 image_name`

### 映射文件
`docker run -v /logs:/var/log/ -p 5000:80 image_name`

将宿主`/logs`目录加载到container中的`/var/log`

### --add-host
`docker run -it --add-host db-static:1.1.1.1 ubuntu cat /etc/hosts1`

## docker其它相关命令

### 进入container命令行

`docker exec -i -t container_id /bin/bash`

退出 `exit`

## 垃圾清理

### stop  remove container

`docker stop $(docker ps -a -q)`

`docker rm $(docker ps -a -q)`

### 删除images

`dangling`表明该`image`未被打标签且没有被任何容器引用的镜像

`docker rmi $( docker images --filter dangling=true -q)`

#### 删除指定image

`docker rmi imagename:tag`

### 相关博客

1. [Docker网络原则入门：EXPOSE，-p，-P，-link](http://dockone.io/article/455)<br /> [archive.org备份页面](https://web.archive.org/web/20180809011552/http://dockone.io/article/455)
2. [ENTRYPOINT 入口点](https://yeasy.gitbooks.io/docker_practice/image/dockerfile/entrypoint.html) <br />[archive.org备份页面](https://web.archive.org/web/20190708170855/https://yeasy.gitbooks.io/docker_practice/image/dockerfile/entrypoint.html)
3. [Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice)