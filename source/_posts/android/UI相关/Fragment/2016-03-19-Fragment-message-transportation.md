---
author: ivyxjc
date: 2016-03-19
title: Fragment和Activity通信
category: Android
tags: [android,android_UI,android_fragment]
keywords:
description:
---

## Fragment和Activity通信

为了方便碎片和活动之间进行通信， FragmentManager 提供了一个类似于 findViewById()
的方法，专门用于从布局文件中获取碎片的实例，代码如下所示：

```java
RightFragment rightFragment = (RightFragment) getFragmentManager()
.findFragmentById(R.id.right_fragment);
```

这样即可调用Fragment中的方法
在Fragment中都可以用getActivity()方法来得到和当前Fragment相关联的Activity实例。

```java
MainActivity activity=(MainActivity)getActivity();
```
