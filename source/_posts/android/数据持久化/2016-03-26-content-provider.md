---
author: ivyxjc
date: 2016-03-26
title: 跨程序共享数据
category: Android
tags: [android,android_data]
keywords:
description:
---

可以利用`getContentResolver()`获取`ContentResolver`. `ContentResolver`中提供了一系列方法用于对数据进行CRUD操作.
<!--more-->
## ContentProvider

通过`getContentResolver()`获取`ContentResolver`的实例，`ContentResolver`中提供了一系列方法用于对数据进行CRUD操作.
其中
 1.insert() 用于添加
 2.update() 用于更新
 3.delete() 用于删除
 4.query()  用于查询

### 内容URI及用法

`ContentResolver`中的CRUD操作是不接受表名参数的，而是使用一个`Uri`参数代替。该参数被称为内容`URI`，由两部分组成：authority和path。
内容`URI`标准写法：

```
content://com.example.app.provider/table1
```

需要将内容`URI`解析成`Uri`对象才可以作为参数传入

```java
Uri uri=Uri.parse("content://com.example.app.provider/table1");
```

### 查询操作

```java
Cursor cursor=getContentResolver().query(uri,projection,selection,selectionArgs,orderBy);
```

其它操作与之类似，也和SQLite中的操作相像。
![](/assets/img/posts/content_provider_query.png)

## 自定义ContentProvider

可以`extends ContentProvider`，需要覆盖父类的6个方法.

 1.`public boolean onCreate()`:初始化ContentProvider时调用，完成对数据库的创建和升级，返回true则表示ContentProvider初始化成功，返回false表示失败。**注意：只有存在`ContentResolver`尝试访问该程序中的而数据时，ContentProvider才会初始化**。
 2.`public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder)`：从ContentProvider中查询数据。使用`uri`参数来确定查询哪一张表，`projection`用于确定查询哪些列，`selection`和`selectionArgs`用来约束查询内容，`sordOreder`参数用于对结果进行排序，查询结果放在`Cursor`中返回。
 3.`public Uri insert(Uri uri, ContentValues values)`：向`ContentProvider`中添加一条数据。
 4.`public int delete(Uri uri, String selection, String[] selectionArgs)`：从`ContentProvider`中删除数据。
 5.`public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs)`：更新数据
 6.`public String getType(Uri uri)`:根据传入的内容URI来返回相对应的MIME类型。

一个内容URI对应的MIME字符串由三部分组成：
 1.必须以vnd开头。
 2.如果内容URI以路径结尾，则后街`androi.cursor.dir/`,如果内容URI以id结尾，则后接`android.cursor.item/`。
 3.最后街上`vnd.<authority>.<path>`


## 安全性保证

因为所有的CRUD操作都一定要匹配到相应的内容URI格式才能进行，所以我们可以控制外部程序能够获得的存储内容。
