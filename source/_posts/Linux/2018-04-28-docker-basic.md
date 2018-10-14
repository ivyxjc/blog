---
layout: post
title: Docker相关命令
category: Web
tags: [docker]
keywords: docker
description:
---

Docker的相关命令
<!--more-->

## 运行docker

### 映射端口
`docker run -p 8080:8080 image_name`

### 映射文件
`docker run -v /logs:/var/log/ -p 5000:80 image_name`

将宿主`/logs`目录加载到container中的`/var/log`

### --add-host
`docker run -it --add-host db-static:1.1.1.1 ubuntu cat /etc/hosts1`

## docker相关命令


### 进入container命令行

`docker exec -i -t container_id bash`

退出 `exit`