---
author: ivyxjc
date: 2018-12-30
title: 为什么行星轨道是椭圆 
category: Math
tags: [math,elipse]
keywords:
description: 
---

只根据基本的数学只是，微积分关于极限的思想，万有引力定律和角动量守恒定律来说明为什么行星轨道是椭圆。

本文主要根据**3Blue1Brown**的一个视频[(B站)【官方双语】费曼失传的演讲](https://www.bilibili.com/video/av28012188)来写出的。


##  一种构造椭圆的特殊方法

### 构造方法
1. 首先构建一个圆A, 并在圆内取另外一点C
2. 从点C引线段到圆A上（即CD）
3. 取CD的垂线，其中中垂线和线段AD的焦点为E，
4. 重复2，3 的操作，点E的轨迹即是一个椭圆

如下图： 

<!-- ![一种构造椭圆的方法](https://5gbg9g.ch.files.1drv.com/y4mcF8m_X7tik4PW--_G0A8YX_-cKOFgxzlkYZLVVSXtqZfoNJTf2mYJ8xS_2o0ViPlg6T-s1Zp2o_fWpYfcrrS6dpqRTgD4g7a9f4H2ZoKumtBGg8GPVQhv4YCCOsGT9V3HHzW1wsDRM5CxwJyqM5pb5FWKq6ARBFZt5lxZmlQPjsy5rHOxSOJcL1YuYkFB1LoSseHVugXhZBUZ-XLFfGr2w/elipse_1.gif) -->

### 证明

**椭圆**是平面上到两个固定点的距离之和为常数的点之轨迹。

![](https://ch3301files.storage.live.com/y4mfqesmWk9tcvBJm1cjOBUc9J1vSi1OF7mllcwoMBPO2KCVy9TifP--R9GNtwcvrSR0IL9uFVgkx4p1JLRhN5VYOqogS9p7VyBqLyFHgDmDHu55GhPG33pyLce63JoInaYWU1W5bN_SdY9x63b5zju_i2l4we5vRhHhK9Cvj2Ri8Sie4IrPfjWOaokpjlb7isFBwv__KLJwJ-PzVULUIM3eA/elipse_prove.png?psid=1&width=891&height=778)


因为FG是CD的中垂线，所以ED=EC，因此AE+ED=AE+EC。所以点E到点A和点C的长度和是固定的，因此点E的轨迹是一个椭圆。


如果我们在线FG上另取一点X, 那么XD+XA>AD，意味着点X在椭圆外，这意味着FG与椭圆AC有且只有一个交点（E）。即FG是椭圆AC的切线。


## 开普勒第二定律


**开普勒第二定律**也被称为等面积定律，意即在相等时间内，太阳和运动着行星连线所扫过的面积是相同的。