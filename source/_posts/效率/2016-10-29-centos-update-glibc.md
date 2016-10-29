---
author: ivyxjc
date: 2016-10-19
title: centos 64位更新glibc
category: 效率
tags: [linux]
keywords:
description: 如何在centos中升级glibc
toc: true
---

## 下载新版本glibc

```git
git clone git://sourceware.org/git/glibc.git
cd glibc
git checkout --track -b local_glibc-2.24 origin/release/2.24/master
```

## 更新glibc

```shell
../configure --prefix=/opt/glibc-2.24
```

## 常见问题

###  These critical programs are missing or too old: as ld

调用`ld -v`是2.20..

```shell
vi ../configure
```

在文件中寻找到
```
case $ac_prog_version in
'') ac_prog_version="v. ?.??, bad"; ac_verc_fail=yes;;
2.1[3-9]*|2.2[0-9]*)
```
**|2.2[0-9]\***是之后添加的, 因为该机的版本为2.20..., 意即该正则表达式要将本机的版本包括进去.

### checking installed Linux kernel header files... missing or too old!
