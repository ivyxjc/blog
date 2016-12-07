---
author: ivyxjc
date: 2016-12-7
title: 如何下拉, 上拉刷新数据
category: Android
tags: [android,android_UI,android_refresh]
keywords:
description: 如何下拉, 上拉刷新数据: 本文介绍SwipeRefreshLayout,
toc: true
---

## SwipeRefreshLayout

`SwipeRefreshLayout`是由google官方提出的下拉刷新空间, 在`android.support.v4`兼容库中.

### 使用

#### 布局文件
`SwipeRefreshLayout`基本上可以包裹任何可以滚动的内容(ListView, RecyclerView...,WebView)

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.SwipeRefreshLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/swipe_refresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/news_rv"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </android.support.v7.widget.RecyclerView>
</android.support.v4.widget.SwipeRefreshLayout>
```

#### 刷新操作

```java
mSwipeRefreshLayout=(SwipeRefreshLayout)findViewById(R.id.swipe_refresh);
mSwipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                pageNum+=1;
                updateDatas(service,apiKey,pageNum);
            }
        });

....
public void updateDatas(Api.NewsService service,String apiKey,int pageNum){
        service.getList(apiKey,pageNum)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<News>() {
                    @Override
                    public void onCompleted() {
                      ...
                    }

                    @Override
                    public void onError(Throwable e) {
                      ...
                    }

                    @Override
                    public void onNext(News news) {
                        datas=news.getDatas();
                        notifyDatasetChanged(datas);
                        mSwipeRefreshLayout.setRefreshing(false);
                    }
                });
    }
```

在获取完数据之后, 需要调用`mSwipeRefreshLayout.setRefreshing(false);`, 否则加载的小圆圈将一直在旋转.

#### 自定义

```java
//设置下拉出现的小圆圈是否缩放出现，出现的位置，最大的下拉位置
 mSwipeRefreshLayout.setProgressViewOffset(true,R.attr.actionBarSize+20,200);

//设置下拉圆圈的大小，两个值 LARGE， DEFAULT
mySwipeRefreshLayout.setSize(SwipeRefreshLayout.LARGE);

// 禁用下拉刷新
mySwipeRefreshLayout.setEnabled(false);

// 设定下拉圆圈的背景颜色
mySwipeRefreshLayout.setProgressBackgroundColor(R.color.red);

// 设置下拉圆圈的颜色, 将按照该颜色顺序展示
mySwipeRefreshLayout.setColorSchemeResources(color1,color2);
```

## Ultra Pull to Refresh

###
