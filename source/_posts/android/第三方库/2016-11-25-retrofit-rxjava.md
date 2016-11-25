---
author: ivyxjc
date: 2016-11-25
title: 利用Retrofit, RxJava获取网络内容
category: Android
tags: [android,android_lib]
keywords:
description: Retrofit
---


## Retrofit & RxJava
关于如何使用Retrofit和RxJava请阅读参考中的两篇文章.

## Retrofit处理数据

Retrofit是在什么时候处理从网络中获取到的json数据的呢? 我从几个使用了Retrofit的项目源代码寻找了半天, 也没有找到处理json的相关代码. 后来才发现, Retrofit中使用`.addConverterFactory(GsonConverterFactory.create())`其实就是自动添加了json解析, 它会将json数据直接转换为java类(即Pojo).

[convertToPojo](http://pojo.sodhanalibrary.com/)可以根据json的内容自动生成Pojo类.


### 以豆瓣api为例

#### json数据格式
豆瓣正在热映的json数据格式大致如下:

![](http://oezmbgg4j.bkt.clouddn.com/douban_json.png)

```java
DoubanService.java

interface DoubanService {
    @GET("/v2/movie/in_theaters")
    Call<Douban> getList();
}
```

#### pojo类

```java
public class Douban {
    @SerializedName(value = "subjects")
    private List<Subjects> subjects;

    ...Getter and Setter..
}
```

```java

public class Subjects {
    private String id;
    private String title;
    private Rating rating;
}
```

```java
public class Rating {
    private String min;
    private String max;
    private String stars;

    ...Getter and Setter..
}
```

#### 处理并显示数据

```java
public class DoubanRun extends Thread {
    @Override
    public void run() {
        super.run();
        Retrofit retrofit=new Retrofit.Builder()
                .baseUrl("https://api.douban.com")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        DoubanService douban=retrofit.create(DoubanService.class);
        Call<Douban> call=douban.getList();

        try {
            Douban d=call.execute().body();
            for (Subjects l:d.getSubjects()){
                 Log.i("TAGGGG",l.getId()+" "+l.getTitle());
            }
        } catch (IOException e) {
            e.printStackTrace();
            Log.i("TAGGGG","ff");
        }
    }
}
```

结果

```
26630781 我不是潘金莲
25726614 神奇动物在哪里
25793398 海洋奇缘
26365631 冲天火
26596486 名侦探柯南：纯黑的恶梦
3025375 奇异博士
26370431 夏威夷之恋
26673217 我是处女座
26696875 最萌身高差
25983044 比利·林恩的中场战事
22266320 深海浩劫
26876505 怨灵地下室
25921812 驴得水
26598021 航海王之黄
...
```

#### 配合RxJava

`DoubanService.java`改为:

```java
interface DoubanService {
    @GET("/v2/movie/in_theaters")
    Observable<Douban> getList();
}
```

`DoubanRun`改为:

```java
public class DoubanRun extends Thread {

    @Override
    public void run() {
        super.run();

        Retrofit retrofit=new Retrofit.Builder()
                .baseUrl("https://api.douban.com")
                .addConverterFactory(GsonConverterFactory.create())
                //
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())

                .build();

        DoubanService doubanService=retrofit.create(DoubanService.class);

        doubanService.getList()
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Subscriber<Douban>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Douban douban) {
                        for (Subjects l:douban.getSubjects()){
                            Log.i("TAGGGG",l.getId()+" "+l.getTitle());

                        }
                    }
                });
    }
}

```

#### 注意点

1. 出现`Expected BEGIN_ARRAY but was BEGIN_OBJECT at`或者pojo类中对应的变量的类型不对. 比如 如果在`Subjects`类中将rating设为String. 就会有`Expected String but was BEGIN_OBJECT at...`错误.



## 参考文章
1. [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083)<br>[archive.org备份页面](https://web.archive.org/web/20161023043938/http://gank.io/post/560e15be2dca930e00da1083)
2. [RxJava 与 Retrofit 结合的最佳实践](https://gank.io/post/56e80c2c677659311bed9841)<br>[archive.org备份页面](https://web.archive.org/web/20161022232218/http://gank.io/post/56e80c2c677659311bed9841)
