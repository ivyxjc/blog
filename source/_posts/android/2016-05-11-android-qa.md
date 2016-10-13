---
author: ivyxjc
date: 2016-05-11
title: android编程中的一些常见问题和解决办法
category: Android
tags: [android,java]
keywords:
description: Android开发中常见的一些细节问题.
---

### 1.如何解决有些函数只能接受`String`而无法接受`Int`

这样就无法使用`R.string...`.

解决方法:<br>
`getResources().getString(R.string.aaa))`
