---
layout: post
title: 虚拟机环境搭建和相关问题解决
category: 效率
tags: [faq,python,spider,bs4]
keywords:
description:
---

## 安装pip

```
sudo apt-get remove pip
wget .../get-pip.py
python get-pip.py
```


## 安装scrapy
安装scrapy出现`Python.h No such file...`

```
sudo apt-get install python3-dev
```


若出现`failed to build wheel for Cryptography`

```
sudo apt-get install build-essential libssl-dev libffi-dev python-dev
```

## 安装pycharm

进入pycharm文件内

```
./pycharm.sh
```


```
sudo gedit /usr/share/applications/Pycharm.desktop
然后输入
[Desktop Entry]
Type=Application
Name=Pycharm
GenericName=Pycharm3
Comment=Pycharm3:The Python IDE
Exec="/XXX/pycharm-community-3.4.1/bin/pycharm.sh" %f
Icon=/XXX/pycharm-community-3.4.1/bin/pycharm.png
Terminal=pycharm
Categories=Pycharm;
```

然后进入`/usr/share/applications/`双击打开即可。




## VMware和主机共享文件夹




```bash
sudo apt-get install open-vm-tools-dkms
```


```
sudo gksu gedit/etc/fstab
```
