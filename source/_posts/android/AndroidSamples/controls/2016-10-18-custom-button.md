---
author: ivyxjc
date: 2016-09-24
title: 自定义FloatingActionButtom
category: Android
tags: [android,android_control]
keywords:
description: 如何自定义fab, 以及给其设置监听
---


## 设置监听的最基本方式

```java
public class FloatingActionButton extends FrameLayout implements Checkable {

    public static interface OnCheckedChangeListener {
        void onCheckedChanged(FloatingActionButton fabView, boolean isChecked);
    }

    private OnCheckedChangeListener mOnCheckedChangeListener;

    public void setOnCheckedChangeListener(OnCheckedChangeListener listener) {
        mOnCheckedChangeListener = listener;
    }
```
