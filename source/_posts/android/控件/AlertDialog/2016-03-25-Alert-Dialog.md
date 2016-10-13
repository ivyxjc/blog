---
author: ivyxjc
date: 2016-03-25
title: AlertDialog
category: Android
tags: [android,android_control]
keywords:
description:
---


## 在Activity中调用AlertDialog

只能在`Activity`中调用`AlertDialog`，所以要想在`Receiver`中使用`AlerDialog`，可以在`Receiver`中用`starActivity()`启动有`AlertDialog`的`Activity`。


```java
@Override
public void onReceive(final Context context, Intent intent) {
    Intent i=new Intent(context,AlertDialogActivity.class);
    i.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);;
    context.startActivity(i);
}
```

```java
public class AlertDialogActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        AlertDialog.Builder dialogBuilder=new AlertDialog.Builder(this);
        dialogBuilder.setTitle("Warning");

        dialogBuilder.setMessage("you are force to be offline");
        dialogBuilder.setCancelable(false);
        dialogBuilder.setPositiveButton("OK",
                new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        ActivityCollector.finishAll();
                        Intent intent = new Intent(AlertDialogActivity.this,LoginActivity.class);
                        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                        startActivity(intent);//重新启动LoginActivity

                    }
                });
        AlertDialog alertDialog=dialogBuilder.create();

        alertDialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ALERT);
        alertDialog.show();
    }
}
```
