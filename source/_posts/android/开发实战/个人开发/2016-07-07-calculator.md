---
author: ivyxjc
date: 2016-07-07
title: 简易计算器开发
category: Android
tags: [android,android_project]
keywords:
description: 一个简易计算器的开发中遇到的EditText相关问题.
---

## edittext按回车时操作

如果有多行，没一行有多个edittext时，按回车时，它会到下一行的edittext之中，而不是同一行的下一个editext。

在xml文件中添加`imeOptions`。

```xml
<EditText
        android:layout_width="0dp"
        android:layout_weight="1"
        android:layout_height="wrap_content"
        android:inputType="numberDecimal"
        android:id="@+id/editText3"
        android:imeOptions="actionNext"/>
```

但是最后一行按回车回跳到下一行最后一个而不是下一行第一个。



## listfragment自己编写listadapter中button监听问题

listfragment自己编写listadapter中button总是监听最后一个item中的数据。

![](http://oezmbgg4j.bkt.clouddn.com/listview_button_click.gif)

无论点击哪一个button都是计算最后一行。
