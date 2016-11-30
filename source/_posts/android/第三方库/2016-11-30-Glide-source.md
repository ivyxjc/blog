---
author: ivyxjc
date: 2016-11-30
title: Glide源码分析 一
tags: [android,project]
keywords:
description: Glide源码分析
---


## 从Glide入口开始 


Glide最简单的用法如下所示

```java
Glide.with(context);
    .load(url)
    //硬盘存储策略
    .diskCacheStrategy(DiskCacheStrategy.ALL)
    //是否启用内存缓存 默认为true
    .skipMemoryCache(false)
    .centerCrop()
    .into(imageView);
```

### with(context)

```java
public static RequestManager with(Context context) {
    RequestManagerRetriever retriever = RequestManagerRetriever.get();
    return retriever.get(context);
}
```

###  RequestManagerRetriever get(context)

依据传入的context的类型, 进行相应的操作. context的类型主要分为`FragmentActivity`, `Activity`和`ContextWrapper`, 当context是`ContextWrapper`时, 调用`get(Context context)`查看其封装的是哪一种Activity.

```java
RequestManagerRetriever.java

public RequestManager get(Context context) {
    if (context == null) {
        throw new IllegalArgumentException("You cannot start a load on a null Context");
    } else if (Util.isOnMainThread() && !(context instanceof Application)) {
        if (context instanceof FragmentActivity) {
            return get((FragmentActivity) context);
        } else if (context instanceof Activity) {
            return get((Activity) context);
        } else if (context instanceof ContextWrapper) {
            return get(((ContextWrapper) context).getBaseContext());
        }
    }

    return getApplicationManager(context);
}
```
### RequestManagerRetriever get(Activity) get(FragmentActivity)

```java
RequestManagerRetriever.java

public RequestManager get(FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {
        return get(activity.getApplicationContext());
    } else {
        assertNotDestroyed(activity);
        FragmentManager fm = activity.getSupportFragmentManager();
        return supportFragmentGet(activity, fm);
    }
}
```

```java
@TargetApi(Build.VERSION_CODES.HONEYCOMB)
public RequestManager get(Activity activity) {
    if (Util.isOnBackgroundThread() || Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
        return get(activity.getApplicationContext());
    } else {
        //判断activity是否已经被销毁, 如果销毁抛出异常
        assertNotDestroyed(activity);
        //获取对应FragmenManager
        android.app.FragmentManager fm = activity.getFragmentManager();
        return fragmentGet(activity, fm);
    }
}
```

这两个方法差别不大, 主要是因为`FragmentActivity`来自于support v4库中, 和标准库Activity获取FragmentManager的方法以及返回的FragmentManager类型不太相同.


###  RequestManagerRetriever fragmentGet(Context, FramentManager)


```java
RequestManagerRetriever.java

@TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR1)
RequestManagerFragment getRequestManagerFragment(final android.app.FragmentManager fm) {
    RequestManagerFragment current = (RequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
        current = pendingRequestManagerFragments.get(fm);
        if (current == null) {
            current = new RequestManagerFragment();
            pendingRequestManagerFragments.put(fm, current);
            fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
            handler.obtainMessage(ID_REMOVE_FRAGMENT_MANAGER, fm).sendToTarget();
        }
    }
    return current;
}

@TargetApi(Build.VERSION_CODES.HONEYCOMB)
RequestManager fragmentGet(Context context, android.app.FragmentManager fm) {
    RequestManagerFragment current = getRequestManagerFragment(fm);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
        requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
        current.setRequestManager(requestManager);
    }
    return requestManager;
}
```

这两段代码的主要意义在于生成一个`RequestManagerFragment`并没有内容的但是绑定在制定的context上. 其实目的就是将该Fragment的生命周期绑定在该context上, 并在相应的方法调用时执行相应的操作.


```java
RequestManagerFragment.java

ActivityFragmentLifecycle lifecycle;

@Override
public void onStart() {
    super.onStart();
    lifecycle.onStart();
}

@Override
public void onStop() {
    super.onStop();
    lifecycle.onStop();
}

@Override
public void onDestroy() {
    super.onDestroy();
    lifecycle.onDestroy();
}

```


### RequestManager


```java
RequestManager implements LifecycleListener
//ActivityFragmentLifecycle 也实现了 LifecycleListener
@Override
public void onStart() {
    // onStart might not be called because this object may be created after the fragment/activity's onStart method.
    resumeRequests();
}

/**
    * Lifecycle callback that unregisters for connectivity events (if the android.permission.ACCESS_NETWORK_STATE
    * permission is present) and pauses in progress loads.
    */
@Override
public void onStop() {
    pauseRequests();
}

/**
    * Lifecycle callback that cancels all in progress requests and clears and recycles resources for all completed
    * requests.
    */
@Override
public void onDestroy() {
    requestTracker.clearRequests();
}
```

```java
class ActivityFragmentLifecycle implements Lifecycle {
    private final Set<LifecycleListener> lifecycleListeners =
            Collections.newSetFromMap(new WeakHashMap<LifecycleListener, Boolean>());
    private boolean isStarted;
    private boolean isDestroyed;

    /**
     * Adds the given listener to the list of listeners to be notified on each lifecycle event.
     *
     * <p>
     *     The latest lifecycle event will be called on the given listener synchronously in this method. If the
     *     activity or fragment is stopped, {@link LifecycleListener#onStop()}} will be called, and same for onStart and
     *     onDestroy.
     * </p>
     *
     * <p>
     *     Note - {@link com.bumptech.glide.manager.LifecycleListener}s that are added more than once will have their
     *     lifecycle methods called more than once. It is the caller's responsibility to avoid adding listeners
     *     multiple times.
     * </p>
     */
    @Override
    public void addListener(LifecycleListener listener) {
        lifecycleListeners.add(listener);

        if (isDestroyed) {
            listener.onDestroy();
        } else if (isStarted) {
            listener.onStart();
        } else {
            listener.onStop();
        }
    }

    void onStart() {
        isStarted = true;
        for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
            lifecycleListener.onStart();
        }
    }

    void onStop() {
        isStarted = false;
        for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
            lifecycleListener.onStop();
        }
    }

    void onDestroy() {
        isDestroyed = true;
        for (LifecycleListener lifecycleListener : Util.getSnapshot(lifecycleListeners)) {
            lifecycleListener.onDestroy();
        }
    }
}
```

上述的代码总体的目的就是将RequestManager的生命周期和RequestManagerFragment绑定起来, 进而和Activity的生命周期绑定起来.

## RequestManager

下面来看一下RequestManager的功能.

### 构造函数

```java
RequestManager.java

public RequestManager(Context context, Lifecycle lifecycle, RequestManagerTreeNode treeNode) {
    this(context, lifecycle, treeNode, new RequestTracker(), new ConnectivityMonitorFactory());
}

RequestManager(Context context, final Lifecycle lifecycle, RequestManagerTreeNode treeNode,
        RequestTracker requestTracker, ConnectivityMonitorFactory factory) {
    this.context = context.getApplicationContext();
    this.lifecycle = lifecycle;
    this.treeNode = treeNode;
    this.requestTracker = requestTracker;
    this.glide = Glide.get(context);
    this.optionsApplier = new OptionsApplier();

    ConnectivityMonitor connectivityMonitor = factory.build(context,
            new RequestManagerConnectivityListener(requestTracker));

    // If we're the application level request manager, we may be created on a background thread. In that case we
    // cannot risk synchronously pausing or resuming requests, so we hack around the issue by delaying adding
    // ourselves as a lifecycle listener by posting to the main thread. This should be entirely safe.
    if (Util.isOnBackgroundThread()) {
        new Handler(Looper.getMainLooper()).post(new Runnable() {
            @Override
            public void run() {
                lifecycle.addListener(RequestManager.this);
            }
        });
    } else {
        lifecycle.addListener(this);
    }
    lifecycle.addListener(connectivityMonitor);
}
```

#### Glide.get(context)

该方法就是利用单例模式获取一个Glide实例.

```java
Glide.java

private static volatile Glide glide;

public static Glide get(Context context) {
    if (glide == null) {
        synchronized (Glide.class) {
            if (glide == null) {
                Context applicationContext = context.getApplicationContext();
                List<GlideModule> modules = new ManifestParser(applicationContext).parse();

                GlideBuilder builder = new GlideBuilder(applicationContext);
                for (GlideModule module : modules) {
                    module.applyOptions(applicationContext, builder);
                }
                glide = builder.createGlide();
                for (GlideModule module : modules) {
                    module.registerComponents(applicationContext, glide);
                }
            }
        }
    }

    return glide;
}
```

#### GlideBuilder

利用GlideBuilder中的createGlide()方法生成了Glide, 在这个过程中完成了以下几个任务:

1. 初始化线程池
2. 初始化bitmap池
3. 初始化内存缓存类
4. 初始化内部磁盘缓存类
5. 初始化引擎类
6. 设置默认的解码格式

```java
GlideBuilder.java

Glide createGlide() {
    if (sourceService == null) {
        //初始化线程池
        final int cores = Math.max(1, Runtime.getRuntime().availableProcessors());
        sourceService = new FifoPriorityThreadPoolExecutor(cores);
    }
    if (diskCacheService == null) {
        diskCacheService = new FifoPriorityThreadPoolExecutor(1);
    }

    //初始化bitmap池
    MemorySizeCalculator calculator = new MemorySizeCalculator(context);
    if (bitmapPool == null) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
            int size = calculator.getBitmapPoolSize();
            bitmapPool = new LruBitmapPool(size);
        } else {
            bitmapPool = new BitmapPoolAdapter();
        }
    }

    if (memoryCache == null) {
        memoryCache = new LruResourceCache(calculator.getMemoryCacheSize());
    }

    if (diskCacheFactory == null) {
        diskCacheFactory = new InternalCacheDiskCacheFactory(context);
    }

    if (engine == null) {
        engine = new Engine(memoryCache, diskCacheFactory, diskCacheService, sourceService);
    }

    if (decodeFormat == null) {
        decodeFormat = DecodeFormat.DEFAULT;
    }

    return new Glide(engine, memoryCache, bitmapPool, context, decodeFormat);
}
```

#### ManifestParser

```java
ManifestParser.java


private static final String GLIDE_MODULE_VALUE = "GlideModule";

public List<GlideModule> parse() {
    List<GlideModule> modules = new ArrayList<GlideModule>();
    try {
        //根据PackageName获取metadata信息
        ApplicationInfo appInfo = context.getPackageManager().getApplicationInfo(
                context.getPackageName(), PackageManager.GET_META_DATA);
        //若有metadata信息
        if (appInfo.metaData != null) {
            //遍历metadata
            for (String key : appInfo.metaData.keySet()) {
                //将key和GLIDE_MODULE_VALUE相等的全部加入modules 之中
                if (GLIDE_MODULE_VALUE.equals(appInfo.metaData.get(key))) {
                    modules.add(parseModule(key));
                }
            }
        }
    } catch (PackageManager.NameNotFoundException e) {
        throw new RuntimeException("Unable to find metadata to parse GlideModules", e);
    }

    return modules;
}

//通过反射获取GlideModule实例
private static GlideModule parseModule(String className) {
    Class<?> clazz;
    try {
        clazz = Class.forName(className);
    } catch (ClassNotFoundException e) {
        throw new IllegalArgumentException("Unable to find GlideModule implementation", e);
    }

    Object module;
    try {
        module = clazz.newInstance();
    } catch (InstantiationException e) {
        throw new RuntimeException("Unable to instantiate GlideModule implementation for " + clazz, e);
    } catch (IllegalAccessException e) {
        throw new RuntimeException("Unable to instantiate GlideModule implementation for " + clazz, e);
    }

    if (!(module instanceof GlideModule)) {
        throw new RuntimeException("Expected instanceof GlideModule, but found: " + module);
    }
    return (GlideModule) module;
}
```

通过反射的方式获取在Manifest.xml中自定义的GlideModule对象, 获得之后遍历ArrayList<GlideModule>, 对每一个GlideModuled调用相应的applyOptions()和registerComponents()方法.


