---
author: ivyxjc
date: 2016-07-06
title: CriminalIntent程序中Fragment，Activity通信
category: Android
tags: [android,android_adapter]
keywords:
description: 使用Intent在Fragment, Activity中进行通信.
---

## 最简单的应用

```java
Intent intent=new Intent(getActivity(), ActivityCrimeBase.class);
startActivity(intent);
```

## 使用extra添加信息

### 附加extra信息

```java
intent.putExtra(FragmentCrime.EXTRA_CRIME_ID,c.getId());
```

### 获取extra信息

```java

FragmentCrime.java

public static final String EXTRA_CRIME_ID="com.jc.criminalIntent2.crime_id";
....
....
UUID crimeId=(UUID)getActivity()
                .getIntent()
                .getSerializableExtra(EXTRA_CRIME_ID);
```

此方法会破坏fragment的封装性，因为这时fragment总是需要由某个具体的activity托管。

为了解决这一问题，可以就爱你个`mCrimeId`存放在fragment的arguments bundle中。


```java

ActivityCrime.java
public class ActivityCrime extends ActivitySetFragmentBase {

    @Override
    protected Fragment createFragment() {
        UUID crimeId=(UUID)getIntent()
                .getSerializableExtra(FragmentCrime.EXTRA_CRIME_ID);

        return FragmentCrime.newInstance(crimeId);
    }
}

FragmentCrime.java

public void onCreate(@Nullable Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     UUID crimeId=(UUID)getArguments()
             .getSerializable(EXTRA_CRIME_ID);
     mCrime=CrimeLab.get(getActivity()).getCrime(crimeId);
 }


 public static FragmentCrime newInstance(UUID crimeId){

     Bundle args=new Bundle();
     args.putSerializable(EXTRA_CRIME_ID,crimeId);
     FragmentCrime fragment=new FragmentCrime();
     fragment.setArguments(args);
     return fragment;
 }

```
