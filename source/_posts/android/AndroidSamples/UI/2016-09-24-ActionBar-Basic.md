---
author: ivyxjc
date: 2016-09-24
title: ActionBarCompat
category: Android
tags: [android,android_UI]
keywords:
description: 如何添加菜单项, 以及如何在运行时更改菜单项.
---


```xml
values/style

## 布局文件中添加menu

```xml
menu_main.xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:support="http://schemas.android.com/apk/res-auto"
    >
    <item
        android:id="@+id/menu_setting"
        android:icon="@mipmap/ic_action_settings"
        android:title="@string/menu_setting"
        support:showAsAction="never"/>
    <item
        android:id="@+id/menu_refresh"
        android:icon="@mipmap/ic_action_refresh"
        android:title="@string/menu_refresh"
        support:showAsAction="always"/>
</menu>
```

## Activity中添加menu

```xml
values/ids.xml
<resources>
    <item name="menu_location" type="id"/>
</resources>
```

```java
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_main,menu);
    MenuItem location=menu.add(0,R.id.menu_location,0,"Location");
    location.setIcon(R.mipmap.ic_action_location);
    MenuItemCompat.setShowAsAction(location,MenuItemCompat.SHOW_AS_ACTION_IF_ROOM);
    return true;
}

@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getItemId()){
        case R.id.menu_setting:
            return true;
        case R.id.menu_location:
            return true;
        case R.id.menu_refresh:
            return true;
    }
    return super.onOptionsItemSelected(item);
}
```

## 用法

### 运行时更改菜单项

```java
mButtonForbidRefresh.setOnClickListener(new View.OnClickListener() {
           private int i=0;
           @Override
           public void onClick(View view) {
               i++;
               if(i%2==1)
                   mButtonForbidRefresh.setActivated(true);
               else
                   mButtonForbidRefresh.setActivated(false);
               invalidateOptionsMenu();
           }
       });
```


```java
@Override
    public boolean onPrepareOptionsMenu(Menu menu) {
        menu.clear();
        getMenuInflater().inflate(R.menu.menu_main, menu);
        MenuItem item=menu.findItem(R.id.menu_refresh);
        if(mButtonForbidRefresh.isActivated()){
           item.setEnabled(false);
        }

        MenuItem location = menu.add(0, R.id.menu_location, Menu.CATEGORY_SECONDARY, "Location");
        location.setIcon(R.mipmap.ic_action_location);
        MenuItemCompat.setShowAsAction(location, MenuItemCompat.SHOW_AS_ACTION_IF_ROOM);

        return super.onPrepareOptionsMenu(menu);
    }
```

`menu.findItem(int id)`<br>

`menu.getItem(int index)`

### onPrepareOptionsMenu(Menu menu)和onCreateOptionsMenu(Menu menu)区别
`onCreateOptionsMenu(Menu menu)`只在最初的时候会调用每次点击menu都会
调用一次`onPrepareOptionsMenu(Menu menu)`.使用`invalidateOptionsMenu()`
会直接调用onPrepareOptionsMenu(Menu menu);


### 多个Activity共用相同ActionBar

如果应用包含多个 Activity，且其中某些 Activity 提供相同的选项菜单，则可考虑创建一个仅
实现`onCreateOptionsMenu()` 和 `onOptionsItemSelected()`方法的 Activity。然后，
为每个应共享相同选项菜单的 Activity 扩展此类。 通过这种方式，您可以管理一个用于
处理菜单操作的代码集，且每个子级类均会继承菜单行为。若要将菜单项添加到一个
子级 Activity，请重写该 Activity 中的 `onCreateOptionsMenu()`。
调用 `super.onCreateOptionsMenu(menu)`，以便创建原始菜单项，
然后使用 `menu.add()` 添加新菜单项。此外，您还可以替代各个菜单项的超类行为。
