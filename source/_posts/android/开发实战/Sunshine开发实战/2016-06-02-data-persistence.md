---
author: ivyxjc
date: 2016-06-02
title: Sunshine开发实战:数据持久化
category: Android
tags: [android,android_data]
keywords:
description: 数据持久化
---

##　Sunshine应用数据存储总体框架
![](http://oezmbgg4j.bkt.clouddn.com/sunshine_data_persistence.png)

## WeatherContract
A contract is an agreement between the data model, storeage, and views, presentation, describing how information is accessed.
事实上就是在一个类中规定好所有相关的数据表中相应的表名,列名等等.

## WeatherDBHelper

创建数据库并进行初始化
