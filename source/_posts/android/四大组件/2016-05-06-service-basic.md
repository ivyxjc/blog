---
layout: post
title: Service
category: Android
tags: [android , android_service]
keywords:
description:
---

## 服务的生命周期
使用startService()<br>
![](/assets/img/posts/servicelifecycle.png)

一旦调用`startService()`相应的服务就会启动, 并回调`onStartCommand()`方法. 如果该服务没有创建过, 那么`onCreate()`会先于`onStartCommand()`方法执行. 服务启动之后会一直保持运行状态,知道`stopService()`或`stopSelf()`方法被调用. 即使度从此调用`startService()`方法, 每个服务只会存在一个实例,所以只需要调用一次`stopService()`或`stopSelf()`方法, 服务就会停止下来.


另一种方法，使用bindService()<br>
![](/assets/img/posts/servicelifecycle2.png)

这会先回调服务中的`onBind()`方法,若该服务之前没有被创建过,那么`onCreate()`方法会先于`onBind()`方法执行.


注意: 若一个服务既`startService()`又`bindService()`, 那么只要`stopService()`和`unbindService()`都被调用后,服务的`onDestroy()`方法才会执行.


只能解绑定一次,不可以多次解绑定.


## 两种方式的区别

Start方式特点:
1.服务与启动源没有任何关系
2.无法得到服务对象

Bind方式特点:
1.通过`IBinder`接口实例,返回一个`ServiceConnnection`对象给启动源.
2.通过`ServiceConnection`对象的相关方法可以得到`Service`对象.
