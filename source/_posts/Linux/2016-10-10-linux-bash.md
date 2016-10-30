---
author: ivyxjc
date: 2016-10-10
title: Linux 常用命令
category: Linux
tags: [linux]
keywords:
description: 介绍与解压, 文件操作, vim操作, 进程等相关的bash命令.
toc: true
---

## 解压

### 解压.zip文件

```bash
unzip xxx.zip
```

### 解压tar.gz文件


## 文件相关操作

### 空间使用情况

`df -h`: 查看空间使用情况
### 移动


mv dir1 dir2

## 进程相关

### 如何查看正在运行的进程

`ps -A`:显示所有的进程
`ps -a`:显示终端中包括其它用户的所有进程
`ps -x`:显示无控制终端的进程

### 如何关闭正在运行的进程

`kill -9 xxx`:xxx是进程的PID

### 显示端口占用情况

`lsof -i`

## vim相关操作

### 查找

?pattern 光标下方查找
/pattern 光标上方查找


## 定时任务

`crontab -l`:可以查看正在进行的定时任务
`crontab -e`:可以进行修改
`/etc/init.d/cron restart`: 重新该服务
