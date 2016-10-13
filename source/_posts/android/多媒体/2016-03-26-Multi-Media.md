---
author: ivyxjc
date: 2016-03-26
title: Notification初步
category: Android
tags: [android,android_notification]
keywords:
description:
---

## 通知

```java
MainActivity.java

NotificationManager notificationManager=(NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE);

Notification.Builder builder=new Notification.Builder(this);

//设置点击通知进入NotificationActivity
Intent intent=new Intent(this,NotificationActivity.class);

//第二个参数int requestCode相同，表示是同一个pendingIntent
PendingIntent pi=PendingIntent.getActivity(this,0,intent,PendingIntent.FLAG_CANCEL_CURRENT);

//设置通知栏 布局，内容等...
builder.setSmallIcon(R.mipmap.ic_launcher);
builder.setContentTitle("Notification Title");
builder.setContentText("Notification text");
builder.setContentIntent(pi);

//第一个参数在应用中是唯一的，代表这个通知
notificationManager.notify(11, builder.build());
```

1. FLAG_CANCEL_CURRENT:如果当前系统中已经存在一个相同的PendingIntent对象，那么就将先将已有的PendingIntent取消，然后重新生成一个PendingIntent对象。
2. FLAG_NO_CREATE:如果当前系统中不存在相同的PendingIntent对象，系统将不会创建该PendingIntent对象而是直接返回null。
3. FLAG_ONE_SHOT:该PendingIntent只作用一次。在该PendingIntent对象通过send()方法触发过后，PendingIntent将自动调用cancel()进行销毁，那么如果你再调用send()方法的话，系统将会返回一个SendIntentException。
4. FLAG_UPDATE_CURRENT:如果系统中有一个和你描述的PendingIntent对等的PendingInent，那么系统将使用该PendingIntent对象，但是会使用新的Intent来更新之前PendingIntent中的Intent对象数据，例如更新Intent中的Extras。


```java
NotificationActivity.java

NotificationManager manager=(NotificationManager)getSystemService(NOTIFICATION_SERVICE);

//使通知在通知栏消失
manager.cancel(11);
```

```java
//设置声音
Uri soundUri=Uri.fromFile(new File("/system/media/audio/ringtones/Boxbeat.ogg"));
builder.setSound(soundUri);

//设置震动上
//下标为偶数和0的标识手机静止的时长，奇数标识手机震动的时长
long [] vibrates={0,1000,1000,1000,1000,1000};
builder.setVibrate(vibrates);
```

### 效果  

![](/assets/img/posts/notification.gif)



## 短信

### 接收短信

```java
mReceiveFilter=new IntentFilter();
mReceiveFilter.addAction("android.provider.Telephony.SMS_RECEIVED");
mMessageReceiver=new MessageReceiver();
registerReceiver(mMessageReceiver,mReceiveFilter);

class MessageReceiver extends BroadcastReceiver{

        @Override
        public void onReceive(Context context, Intent intent) {
            Bundle bundle=intent.getExtras();

            Object[] pdus=(Object[])bundle.get("pdus");//提取短信消息

            SmsMessage[] messages=new SmsMessage[pdus.length];

            for(int i=0;i<messages.length;i++){
                messages[i]=SmsMessage.createFromPdu((byte[])pdus[i]);
            }

            String address=messages[0].getOriginatingAddress();

            String fullMessage="";
            for(SmsMessage message:messages){
                fullMessage+=message.getMessageBody();//获取短信内容
            }
            sender.setText(address);
            content.setText(fullMessage);
        }
    }

```

### 拦截短信
系统发出的短信广播是一条有序广播，所以可以先获得该广播，再中止广播传递即可。

1.提高MessageReceiver的优先级，让它能够先于系统短信程序接收到短信广播。
2.调用`abortBroadcast()`方法，中止广播传递。

### 发送短信

```java
mSendButton.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Log.i("AAABBB", "before");
            SmsManager smsManager = SmsManager.getDefault();
            Log.i("AAABBB", "getDefault");
            smsManager.sendTextMessage(mTo.getText().toString(), null, mEditText.toString(), null, null);
        }
    });
```
