---
author: ivyxjc
date: 2016-11-27
title: 知乎专栏android app开发中的一些问题及解决方法
category: Android
tags: [android,project]
keywords:
description: 知乎专栏android app开发中的一些问题及解决方法
---


经过一段时间的开发, 知乎专栏app主体上完成, 但还有很多功能没有做出来, 下面是开发过程中的一些问题以及将要完善的功能.

## 如何根据点击自定义创建Fragment

当我们使用tablayout时, 根据点击选择创建fragment, 这时可以使用下面的方法根据传入的内容自定义生成fragment.


```java
private void replaceFragment(Fragment fragment){
       fm.beginTransaction()
               .replace(R.id.fragment_container,fragment)
               .commit();
   }

...
replaceFragment(FragmentTab.newSingleton(R.array.develop,R.array.develop_suffix));

public static FragmentTab newSingleton(int titleId, int suffixId){
       FragmentTab fragment=new FragmentTab();
       Bundle bundle=new Bundle();
       bundle.putInt(Constant.LIST_ACTIVITY_NAV_TITLE,titleId);
       bundle.putInt(Constant.LIST_ACTIVITY_NAV_SUFFIX,suffixId);
       fragment.setArguments(bundle);
       return fragment;
   }
```

## 如何设置夜间模式



## 如何进行数据缓存

关于图片的缓存, Glide库已经完成. 对于其它内容的缓存, 计划使用DiskLRUCache缓存.
