---
author: ivyxjc
date: 2016-10-30
title: Raspberry安装
category: Python
tags: [python,generators]
keywords:
description: Raspberry第一次安装
toc: true
---


## 更改源

需要更改的地方有两个:
1. `/etc/apt/source.list`
2. `/etc/apt/source.list.d/raspi.list`

将其中的网址换掉即可.

## 如何连接局域网中的小米硬盘

命令行的方式都试了一下, 没找到有效的.

用的方法是远程桌面连接Raspberry, `GO->Network->`找到硬盘, 然后移动到pi文件下面.
