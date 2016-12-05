---
author: ivyxjc
date: 2016-07-08
title: CriminalIntent程序中ViewPager相关内容
category: Android
tags: [android,android_view_pager]
keywords:
description: FragmenPagerAdapter和FragmenStatePagerAdapter这两者的区别主要在于卸载不再需要的fragment时采取的方法不同。
---


## FragmenPagerAdapter和FragmenStatePagerAdapter

这两者的区别主要在于卸载不再需要的fragment时采取的方法不同。

使用FragmentStatePagerAdapter会销毁掉不需要的fragment。事务提交后，可以将fragment从activity的FragmentManager中彻底移除。该类名中的`state`表明在销毁fragment时，它会将`onSaveInstanceState(Bundle)`方法中的Bundle信息保存下来。用户切换回来时，保存的实例可用于回复生成新的fragment。


FragmentPagerAdapter对于不再需要的fragment，FragmentManager选择调用事务的`detach(fragment)`方法，而非`remove(fragment)`方法，FragmentPagerAdapter只是销毁fragment的视图，但是仍将实例保留在FragmentManager中。FragmentPagerAdapter创建的fragment不会被销毁。

由上可知:
1. FragmentStatePagerAdapter更省内存，所以当有大量的fragment时，推荐使用FragmentStatePagerAdapter.
2. 但是当用户界面只有少量fragment时，推荐FragmentPagerAdapter。
