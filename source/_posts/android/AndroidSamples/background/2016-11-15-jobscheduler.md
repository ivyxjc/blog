---
author: ivyxjc
date: 2016-11-15
title: JobScheduler
category: Android
tags: [android]
keywords:
description: 如何利用JobScheduler API执行预定的操作
---

## 任务写在JobService中
```java
public class TestJobService extends JobService {
    private static final String TAG = "SyncService";
    @Override
    public void onCreate() {
        super.onCreate();
        Log.i(TAG, "Service created");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.i(TAG, "Service destroyed");
    }

    @Override
    public boolean onStartJob(JobParameters params) {
        // fake work
        Log.i(TAG, "on start job: " + params.getJobId());
        return true;
    }

    @Override
    public boolean onStopJob(JobParameters params) {
        Log.i(TAG, "on stop job: " + params.getJobId());
        return true;
    }

    public void scheduleJob(JobInfo info){
        Log.i(TAG, "schedule job ");
        JobScheduler js=(JobScheduler)getApplication().getSystemService(Context.JOB_SCHEDULER_SERVICE);
        js.schedule(info);
    }
}
```

## 设置调度的相关条件

```java

...
mServiceComponent = new ComponentName(this, TestJobService.class);
...

public void scheduleJob(View v){
        JobInfo info=new JobInfo.Builder(sJobId,mServiceComponent)
            //延迟
            .setMinimumLatency(2000)
            //最长延迟
            .setOverrideDeadline(5000)
            //所需网络类型  本例中 为需要无线网络
            .setRequiredNetworkType(JobInfo.NETWORK_TYPE_UNMETERED)
            .setRequiresDeviceIdle(true)
            .setRequiresCharging(true)
            .build();
        JobSchedulerjobScheduler=(JobScheduler)getApplication().getSystemService(Context.JOB_SCHEDULER_SERVICE);
        jobScheduler.schedule(info);

    }
```




## 如何获取正在运行的所有程序的名称


```java
public String getRunningProcessNames(){
        ActivityManager am = (ActivityManager)this.getSystemService(ACTIVITY_SERVICE);

        List<ActivityManager.RunningAppProcessInfo> l = am.getRunningAppProcesses();
        PackageManager pm = this.getPackageManager();
        StringBuilder sb=new StringBuilder();
        for(ActivityManager.RunningAppProcessInfo i:l){
            try{
                CharSequence c = pm.getApplicationLabel(pm.getApplicationInfo(i.processName, PackageManager.GET_META_DATA));
                sb.append(c.toString()+'\n');
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return sb.toString();
    }
```

## 相关博客

[在Android 5.0中使用JobScheduler](http://blog.csdn.net/bboyfeiyu/article/details/44809395)
