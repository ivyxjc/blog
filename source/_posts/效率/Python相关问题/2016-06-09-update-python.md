---
author: ivyxjc
date: 2016-06-09
title: 在Centos 6中升级python
category: 效率
tags: [faq]
keywords:
description:
---
Centos默认的python使用的是`python2.6`,我常用的是`python3.5`,所以需要升级python.

安装过程:

1. 下载python3.5.1
`wget https://www.python.org/ftp/python/3.5.1/Python-3.5.1.tgz`
2. 解压并安装
 1. `tar zxvf Python-3.5.1.tgz`
 2. `cd Python-3.5.1`
 3. `./configure --prefix=/usr/local/python3.5`
 4. `make`
 5. `make install`
3. 备份原有python命令执行文件
`mv /usr/bin/python /usr/bin/pythonbak`
4. 创建新的python软连接
   `ln -s /usr/local/python3.5/bin/python3.5 /usr/bin/python`

问题:
1. 2.3步可能出现`no acceptable C compiler found in $PATH`问题,这是因为完整的安装`development tools`.
    解决方法:`yum groupinstall "Development Tools"`

2. pip无法使用,出现`versionconflic`的问题,好像是因为pip版本的问题.进入pip的目录,删除pip.再直接下载上述错误信息中显示的`requirement.parse()`中的pip版本,解压缩,安装即可.
