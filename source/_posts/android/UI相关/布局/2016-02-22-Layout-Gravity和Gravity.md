---
layout: post
title: Layout_Gravity和Gravity
category: Android
tags: [android,android_layout]
keywords:
description:
---

## Android Layout_Gravity和Gravity

简单来说layout_gravity表示子控件在父容器的位置，gravity表示控件内容在控件内的位置。
![如图所示][1]


上面图片的xml代码

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical" >

    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:background="#e3e2ad"
            android:orientation="vertical"
            android:id="@+id/">

        <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:textSize="24sp"
                android:text="gravity=" />
        <!--如何设置为wrap_content 则gravity失效-->
        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:background="#bcf5b1"
                android:gravity="left"
                android:text="left" />

        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:background="#aacaff"
                android:gravity="center_horizontal"
                android:text="center_horizontal" />

        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:background="#bcf5b1"
                android:gravity="right"
                android:text="right" />

        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:background="#aacaff"
                android:gravity="center"
                android:text="center" />

    </LinearLayout>
    <!--match_parent改为wrap_content则下面的layout_gravity失效-->
    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:background="#d6c6cd"
            android:orientation="vertical" >

        <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_gravity="center_horizontal"
                android:textSize="24sp"
                android:text="layout_gravity=" />
        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:layout_gravity="left"
                android:background="#bcf5b1"
                android:text="left" />

        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:layout_gravity="center_horizontal"
                android:background="#aacaff"
                android:text="center_horizontal" />

        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:layout_gravity="right"
                android:background="#bcf5b1"
                android:text="right"
                android:id="@+id/"/>

        <TextView
                android:layout_width="200dp"
                android:layout_height="40dp"
                android:layout_gravity="center"
                android:background="#aacaff"
                android:text="center" />

    </LinearLayout>

</LinearLayout>
```

参考
1.[Gravity and layout_gravity on Android][2]


  [1]: http://i.stack.imgur.com/xKdfI.png
  [2]: http://stackoverflow.com/questions/3482742/gravity-and-layout-gravity-on-android
