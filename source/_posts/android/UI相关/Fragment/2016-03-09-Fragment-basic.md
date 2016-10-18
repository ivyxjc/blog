---
author: ivyxjc
date: 2016-03-09
title: Fragment初步
category: Android
tags: [android,android_fragment,android_UI]
keywords:
description: 如何在应用中使用Fragment
---

## Fragment概要
![](http://oezmbgg4j.bkt.clouddn.com/fragmentIntro.png)

## **onCreateView()**方法

Fragment第一次绘制它的用户界面时，系统会调用此方法。为了方便绘制Fragment的UI，此方法必须返回一个View，如果不显示UI，返回null。


### 标识Fragment方法

android:id 提供一个唯一的ID 或
android:tag 提供一个唯一的字符串tag

## 静态加载

在**Activity**的layout文件中声明**Fragment**，其中**android:name**属性制定了在layout中实例化的Fragment类。

静态加载Fragment必须要给Fragment一个唯一的标识。

文件目录：<br>
——src<br>
————Fragment_1.java (extends Fragment)<br>
————Fragment_2.java   (extends Activity)<br>
————MainActivity.java	(extends Activity)<br>
<br>
——layout<br>
————fragment_1.xml<br>
————fragment_2.xml<br>
————activity_main.xml<br>

**Fragment1.java**利用**fragment_1.xml**返回一个**view**，**activity.xml**中的**android:name="com.jc.fragmentbasic.FragmentFirst"**，然后**MainActivity**调用`setContentView(R.layout.activity_main);``


### fragment_1和fragment_2.xml和activity_main.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="This is fragment 1"
        android:textColor="#0000FF"
        android:textSize="25sp"
        android:background="#00FF00"/>

</LinearLayout>
```

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="This is fragment 2"
        android:textColor="#FF0000"
        android:background="#00F0F0"
        android:textSize="25sp" />

</LinearLayout>
```

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:baselineAligned="false"
      android:orientation="horizontal">

    <fragment
        android:id="@+id/fragment1"
        android:name="com.jc.fragment_1.Fragment_1"
        android:layout_width="0dip"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/fragment2"
        android:name="com.jc.fragment_1.Fragment_2"
        android:layout_width="0dip"
        android:layout_height="match_parent"
        android:layout_weight="1" />

</LinearLayout>
```

### Fragment_1.java和Fragment_2.java和MainActivity.java

```java
public class Fragment_1 extends Fragment {

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_1,container,false);
    }
}
```

```java
public class Fragment_2 extends Fragment{
    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_2,container,false);
    }
}
```

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

### 加载过程的代码

```java
case R.id.first:{
    Intent intent=new Intent(this,MainActivity.class);
    startActivity(intent);
    break;
}
```

### 效果

![](http://oezmbgg4j.bkt.clouddn.com/fragment_basic_1.png)


## 动态加载

根据用户的交互情况，对Fragment进行添加，移除，替换，以及执行其他动作，提交Activity的每一套变化被称作一个**事务**。

每一事务都是同时执行一套变化，可以在一个事务中设置你所有执行的变化，包括add(),remove(),replace()，然后提交个Activity，必须调用commit()方法。

如果允许使用Back键返回前一Fragment的状态，调用commit()之前可以加入addToBackStack()方法。

1.获取到FragmentManager，在Activity中可以直接通过getFragmentManager得到。

2.调用beginTransaction方法开启一个事务。

3.向容器内加入Fragment，一般使用replace方法实现，需要传入容器的id和Fragment的实例。

4.提交事务，调用commit方法提交。


### 加载过程代码

```java
case R.id.dynamic:{
    FragmentDynamic fragmentDynamic=new FragmentDynamic();
    FragmentManager fragmentManager=getFragmentManager();
    FragmentTransaction beginTransaction=fragmentManager.beginTransaction();
    beginTransaction.add(R.id.frame,fragmentDynamic);
    beginTransaction.commit();
    break;
}
```

## fragment生命周期

关键方法：**onAttach()**,**onCreateView()**

```java

public class FragmentThird extends Fragment {
    @Nullable
    @Override

    /**
     * 每次创建会绘制Fragment的View组件时回调该方法。
     */
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View v=inflater.inflate(R.layout.fragment_third,container,false);
//        Log.i("Main","Fragment third--onCreateView()");
        TextView textView=(TextView)v.findViewById(R.id.textview_third);
        textView.setText("第三个Fragment");
        Log.i("Main","Fragment third--onCreateView()");
        return v;
    }


    /**
     * 当Fragment被添加到Activity时候会回调这个方法，并且只调用一次
     * @param context
     */
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        Log.i("Main","Fragment third--on attach()");
    }

    /**
     * 创建Fragment时会回调，只会调用一次
     */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.i("Main","Fragment third--onCreate()");
    }


    /**
     *当Fragment所在的Activity启动完成后调用
     */
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        Log.i("Main", "Fragment third--onActivityCreated()");
    }

    /**
     * 启动Fragment
     */
    @Override
    public void onStart() {
        super.onStart();
        Log.i("Main", "Fragment third--onStart()");
    }


    /**
     * 回复Fragment时会被回调，调用onStart()方法后面一定会调用
     * onResume方法
     */
    @Override
    public void onResume() {
        super.onResume();
        Log.i("Main", "Fragment third--onResume()");
    }

    /**
     * 暂停Fragment
     */

    @Override
    public void onPause() {
        super.onPause();
        Log.i("Main","Fragment third--onPause()");
    }

    /**
     * 停止Fragment
     */
    @Override
    public void onStop() {
        super.onStop();
        Log.i("Main","Fragment third--onStop()");
    }

    /**
     * 销毁Fragment所包含的View组件时调用
     */

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        Log.i("Main", "Fragment third--onDestroyView()");
    }

    /**
     * 销毁Framgent时会被调用
     */

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.i("Main", "Fragment third--onDestroy()");
    }


    /**
     * 与onAttach()方法对应
     * Fragment从Activity中删除时会回调该方法，并且只会调用一次
     */
    @Override
    public void onDetach() {
        super.onDetach();
        Log.i("Main", "Fragment third--onDetach()");
    }
}
```

```java
Log结果：
启动应用：null

启动Fragment：
I/Main(8163): Fragment third--on attach()
I/Main(8163): Fragment third--onCreate()
I/Main(8163): Fragment third--onCreateView()
I/Main(8163): Fragment third--onActivityCreated()
I/Main(8163): Fragment third--onStart()
I/Main(8163): Fragment third--onResume()

屏幕锁屏：
I/Main(8163): Fragment third--onPause()
I/Main(8163): Fragment third--onStop()


解锁：
I/Main(8163): Fragment third--onStart()
I/Main(8163): Fragment third--onResume()

切换到其它的Fragment:
I/Main(8163): Fragment third--onPause()
I/Main(8163): Fragment third--onStop()
I/Main(8163): Fragment third--onDestroyView()
I/Main(8163): Fragment third--onDestroy()
I/Main(8163): Fragment third--onDetach()

回到桌面:
I/Main(8163): Fragment third--onPause()
I/Main(8163): Fragment third--onStop()


回到Fragment:
I/Main(8163): Fragment third--onStart()
I/Main(8163): Fragment third--onResume()


退出Fragment：
I/Main(8163): Fragment third--onPause()
I/Main(8163): Fragment third--onStop()
I/Main(8163): Fragment third--onDestroyView()
I/Main(8163): Fragment third--onDestroy()
I/Main(8163): Fragment third--onDetach()
```

## Activity和Fragment通信

Activity ---> Fragment:在Activity中创建Bundle数据包，并调用Fragment的setArguments(Bundle bundle)方法。

Fragment ---> Activity：需要在Fragment中定义一个内部回调接口，让包含该Fragment的Activity实现该回调接口。这样Fragment可调用回调方法将数据传递给Activity。

## 相关博客网址
[Android Fragment完全解析，关于碎片你所需知道的一切](http://blog.csdn.net/guolin_blog/article/details/8881711)
