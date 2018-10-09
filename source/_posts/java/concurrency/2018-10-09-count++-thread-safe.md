---
author: ivyxjc
date: 2018-10-09
title: i++ 的线程安全性 
category: Java
tags: [Java]
keywords:
description:
toc: true
---


众所周知，++操作并不是线程安全的。这篇文章主要讲述其非线程安全的原因以及相关问题。

## 使用volatile修饰仍不是线程安全的原因

`i++`分为以下3步：
1. 从内存中读取到count
2. count+1
3. 将结果写回内存

这3步每一步之间都是可以被中断的，**加volatile只是保证从内存中读取到的count值是最新的值，但是存在在别的线程中的count还未写回主存的可能**。

例如：
1. 线程A读取到count为10，此时线程中断
2. 线程B读取到count也为10，线程B进行++操作，结果为11写回主存，
3. 此时线程A恢复，由于它已经从内存中读到count了，所以它仍会从10开始加，得到11写回主存。
4. 我们可以发现，10++在线程A,B各自进行了一次

## 线程安全的写法

### 加锁

```java
public class ThreadTest implements Runnable {
    int count = 0;

    @Override
    public void run() {
        synchronized (this) {
            for (int i = 0; i < 100000; i++) {
                count++;
            }
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Runnable runnable = new ThreadTest();
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        Future f1, f2;
        f1 = executorService.submit(runnable);
        f2 = executorService.submit(runnable);
        f1.get();
        f2.get();
        System.out.println(((ThreadTest) runnable).count);
        executorService.shutdown();
    }
}
```

### 使用原子类

原子类可以的单一操作都是原子性的。它的实现并不是依赖于加锁而是使用CAS

```java
public class ThreadTest2 implements Runnable {
    AtomicInteger count = new AtomicInteger();

    @Override
    public void run() {
        synchronized (this) {
            for (int i = 0; i < 100000; i++) {
                count.getAndIncrement();
            }
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Runnable runnable = new ThreadTest2();
        ExecutorService executorService = Executors.newFixedThreadPool(3);
        Future f1, f2;
        f1 = executorService.submit(runnable);
        f2 = executorService.submit(runnable);
        f1.get();
        f2.get();
        System.out.println(((ThreadTest2) runnable).count.get());
        executorService.shutdown();
    }
}
```

## 性能问题

加锁当然会一定程度上影响性能，但是正确性优于性能。

使用`java.util.concurrent.atomic`中的原子类在很多情况下都有着优于锁的性能，但是在本例中并不是如此。我认为是因为compare比较错误次数太多，重复次数太多导致的。