---
layout: post
title: Linux系统和进程相关的问题和解决方法
category: Linux
tags: [linux]
keywords:
description:
---

## 如何查看正在运行的进程

`ps -A`:显示所有的进程
`ps -a`:显示终端中包括其它用户的所有进程
`ps -x`:显示无控制终端的进程

## 如何关闭正在运行的进程

`kill -9 xxx`:xxx是进程的PID
