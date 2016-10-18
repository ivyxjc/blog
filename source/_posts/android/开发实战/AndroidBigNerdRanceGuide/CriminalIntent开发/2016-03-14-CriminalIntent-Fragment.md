---
author: ivyxjc
date: 2016-03-14
title: CriminalIntent程序中Fragment相关内容
category: Android
tags: [android,android_UI]
keywords:
description: 如何使用Fragment
---

## Activity中托管UI fragment有两种方式：
1. 添加fragment到acitivity中
2. 在activity代码中添加fragment

第一种方法即将fragment添加到acitivity的布局中，这样会使得fragment视图和activity视图绑定。

第二种方法可以在运行时控制fragment，可以决定何时将fragment添加到activity中，也可以移除当前fragment，用其它fragment代替当前fragment...

为了能够灵活地设计UI，所以常用第二种方式来添加fragment。

### 详细步骤

#### 托管UI fragment
1. 需要在activity视图层级结构中为fragment试图安排位置。

#### 创建UI fragment
1. 通过定义布局文件中的组件,组装界面
2. 创建fragment类并设置试图为定义的布局
3. 通过代码的方式,连接布局文件中生成的组件

`Fragment`类要实现两个方法--`onCreate()`和`onCreateView()`方法。`onCreateView()`方法用来生成fragment视图的布局。

```java
@Nullable
@Override
public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
    View v=inflater.inflate(R.layout.fragment_crime,container,false);
    return v;
}
```

`onCreateView()`方法也是生成其它组件的地方。

#### 代码

1. 设置容器

```xml
activity_crime.xml
<FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:id="@+id/fragment_container"/>
```

2. 编写fragment布局文件

```xml
fragment_crime.xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/crime_title_edit_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="@string/crime_title_hint"/>
</LinearLayout>
```

3.使用代码添加fragment

```java
activity_main.java

FragmentManager fm=getSupportFragmentManager();
        Fragment fragment=fm.findFragmentById(R.id.fragment_container);

        if(fragment==null){
            fragment=new FragmentCrime();
            fm.beginTransaction()
                    .add(R.id.fragment_container,fragment)
                    .commit();
        }
```

#### 添加UI fragment到FragmentManager

FragmentManager主要处理：
1. fragment队列
2. fragment事务的回退栈



## Fragment与Activity数据传送

Fragment传送数据到Activity

```java
Intent i=new Intent(getActivity(),CrimeActivity.class);
i.putExtra(CrimeFragment.EXTRA_CRIME_ID,c.getId());
startActivity(i);
```

Activity中接受数据

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    UUID crimeId=(UUID)getActivity().getIntent()
            .getSerializableExtra(EXTRA_CRIME_ID);

    mCrime=CrimeLab.get(getActivity()).getCrime(crimeId);
}
```
