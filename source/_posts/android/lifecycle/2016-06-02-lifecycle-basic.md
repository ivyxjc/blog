---
date: 2016-06-02
title: Activity的生命周期
category: Android
tags: [android,android_lifecycle]
keywords:
description:
---
## 生命周期
1. `onCreate()``
2. `onStart()`
3. `onResume()`
4. `onPause()`
5. `onPause()`
6. `onDestory()`


### `onCreate()`的作用

1. 实例化组件并将组件放在屏幕上
2. 引用已经实例化的组件
3. 为组件设置监听器以处理用户交互
4. 访问外部模型数据

## 翻转屏幕对生命周期的影响
翻转屏幕会导致`Activity`先会被销毁再重新创建，若不加以处理会导致很多意外的问题。顺序为`onPause()`,`onStop()`,`onDestory()`,`onCreate()`,`onStart()`,`onResume()`
