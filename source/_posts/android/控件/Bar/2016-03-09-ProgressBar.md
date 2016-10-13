---
author: ivyxjc
date: 2016-03-09
title: ProgressBar和ProgressDialog
category: Android
tags: [android,android_bar]
keywords:
description: 各种类型进度条的samples, 包括ProgressBar, ProgressDialog 以及如何自定义ProgressBar.
---

## 分类

可以精确显示进度(可以显示刻度或者百分比)

不可以精确显示进度()

```java
//显示两种进度条
setProgressBarVisibility(true);
setProgressBarIndeterminateVisibility(false);
```

还可分类为圆圈型，直线型。

## ProgressBar属性

```xml
android:max="100"
android:progress="50"----第一进度
android:secondaryProgress="80"---第二进度
android:indeterminate="true"---设置是否精确显示，true显示不精确显示进度，false表示精确显示进度
```

## 实例

```java
first=progressBar.getProgress();//获取第一进度条的进度
second=progressBar.getSecondaryProgress();//获取第二进度条的进度
max=progressBar.getMax();//获取进度条的最大进度
int firstPercent=(int)(first/(float)max*100);
int seconPercent=(int)(second/(float)max*100);
text.setText("FirstProgress"+firstPercent+"%  SecondProgress"+seconPercent+"%");

addButton.setOnClickListener(this);
reduceButton.setOnClickListener(this);
resetButton.setOnClickListener(this);

public void onClick(View v) {
    switch (v.getId()){
        case R.id.add:{
            first+=5;
            second+=5;
            break;
        }
        case R.id.reduce:{
            first-=5;
            second-=5;
            break;
        }
        case R.id.reset:{
            first=0;
            second=10;
            break;
        }
        default:{

        }
    }
    //也可使用incrementProgress
    progressBar.setProgress(first);
    progressBar.setSecondaryProgress(second);
    int firstPercent=(int)(first/(float)max*100);
    int seconPercent=(int)(second/(float)max*100);
    text.setText("FirstProgress" + firstPercent + "%  SecondProgress" + seconPercent+"%");
}
```

![](http://oezmbgg4j.bkt.clouddn.com/progressbar.gif)

## ProgressDialog

```xml
<Button
    android:id="@+id/showDialog"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Dialog"/>
```

```java
case R.id.showDialog:{
    //新建ProgressDialog的对象
    prodialog=new ProgressDialog(MainActivity.this);
    //设置显示风格
    prodialog.setProgress(ProgressDialog.STYLE_HORIZONTAL);
    //设置标题
    prodialog.setTitle("IVYXJC");
    //设置对话框文本
    prodialog.setMessage("Welcome to ProgressDialog");
    //设置图标
    prodialog.setIcon(android.R.drawable.ic_dialog_alert);
    //设定最大进度
    prodialog.setMax(100);
    //设定初始化已经增长到的进度
    prodialog.incrementProgressBy(50);
    //进度条是明确显示进度的
    prodialog.setIndeterminate(false);

    //设定一个确定按钮
    prodialog.setButton(DialogInterface.BUTTON_POSITIVE,"确定",new DialogInterface.OnClickListener()
    {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            Toast.makeText(MainActivity.this,"欢迎",Toast.LENGTH_SHORT).show();
        }
    });

    prodialog.show();
    //


    break;
}
```

![](http://oezmbgg4j.bkt.clouddn.com/progressdialog.gif)

## 自定义ProgressBar的样式

ProgressBar的样式是可以自定义的，可参考默认的样式更改。
