---
author: ivyxjc
date: 2019-09-10
title: 如何使用VS Code Remote-SSH
category: 效率
tags: [vs-code,remote-ssh]
keywords:
description: 本文主要介绍如何配置VS Code的Remote SSH 
toc: true
---

本文主要介绍如何在Windows环境下配置Remote SSH。主要是按照官方文档[Remote Development using SSH](https://code.visualstudio.com/docs/remote/ssh)来配置并加上一些小trick.


<!-- nomore -->
## 安装

官方教程指明安装需要3步，其中后两步比较简单，就不介绍了。着重介绍如何安装`OpenSSH compatible SSH client`.

1. Install an OpenSSH compatible SSH client if one is not already present.
2. Install Visual Studio Code or Visual Studio Code Insiders.
3. Install the Remote Development extension pack.


## 安装 OpenSSH compatible SSH client

[官方教程](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client)

```powershell
# Make sure you're running as an Administrator
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
Get-Service ssh-agent
```

## SSH config file

ssh config file 格式如下

```
# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host host的昵称 
    HostName #host的IP或者域名
    User #用户名 
    Port #ssh 端口，如果是22可以不写 
    IdentityFile  OpenSSH key 的地址
```

## 如何根据putty private key 生成 OpenSSH key

1. 打开PuTTYgen，点击load，选择putty private key
2. 点击Conversions, 选择Export OpenSSH Key。这里的地址就是上面所说的OpenSSH key的地址。

注意点： 

这里的地址最好选择 `C:\Users\username`下面，否则可能因为文件权限过松导致出现如下错误:

`Permissions for 'file name' are too open. > It is required that your private key files are NOT accessible by others`

## VS Code 配置

1. 打开VS Code 的settings.
2. 搜索Remote.SSH.configFile
3. 将上述SSH config file的文件绝对路径填入框中