---
author:ivyxjc
date: 2016-04-05
title: Handler与多线程
category: Android
tags: android,android_thread
status:publish
description: Handler是android中一套用来更新UI的机制，也是一套消息处理机制。在非UI线程中更新UI会报错，但是直接在OnCreate()方法中并不会报错，因为此时检查机制还未使用，但是不要这么做。必须在UI线程中更新UI。
---

Handler是android中一套用来更新UI的机制，也是一套消息处理机制。

在非UI线程中更新UI会报错，但是直接在OnCreate()方法中并不会报错，因为此时检查机制还未使用，但是不要这么做。必须在UI线程中更新UI。

```java
mHandler.post(new Runnable() {
    @Override
    public void run() {
        mTextView.setText("update thread");
    }
});
```

## sendMessage()和removeCallbacks()


```java
private Handler mHandler=new Handler(){
    @Override
    public void handleMessage(Message msg) {
        mTextView.setText(""+msg.arg1+msg.obj);
    }
};

```java
new Thread(){
    @Override
    public void run() {
        try{
            Thread.sleep(2000);
//          Message message=new Message();
            Message message=mHandler.obtainMessage();
            Person person=new Person();
            message.arg1=100;
            person.age=10;
            person.name="aaa";
            message.obj=person;
            message.sendToTarget();
        }catch (InterruptedException e){
            e.printStackTrace();
        }

    }
    }.start();
```

```java
mHandler.removeCallbacks(mMyRunnable);
```

### 消息截获

```java
private Handler mHandler=new Handler(new Handler.Callback() {
    @Override
    public boolean handleMessage(Message msg) {
        Toast.makeText(getApplicationContext(),""+1,Toast.LENGTH_SHORT).show();
        //若返回true，则下面的handleMessage不会执行，表示别截获
        return false;
    }
}){
    @Override
    public void handleMessage(Message msg) {
//            mTextView.setText(""+msg.arg1+msg.obj);
        Toast.makeText(getApplicationContext(),""+2,Toast.LENGTH_SHORT).show();
    }

......
};
```

## handler总结

`handler`负责发送消息，`Looper`负责接收Handler发送的消息，并将消息发给`handler`自己。

### 使用Handler更新UI的原因

如果在一个`Activity`中有多个线程去更新UI，若都没有加锁会导致**界面更新混乱**。若都加锁，则会导致**性能下降**。
