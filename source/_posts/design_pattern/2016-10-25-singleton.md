---
author: ivyxjc
date: 2016-10-25
title: 单例模式
category: OO
tags: [设计模式]
keywords:
description: 设计模式中的单例模式
toc: true
---

Ensure a class has only one instance, and provide a global point of access to it.


## 简介

### 最基本的实例

```java
public class Emperor {
    private static final Emperor emperor=new Emperor();

    private Emperor(){}

    public static Emperor getInstance(){
        return emperor;
    }
}
```

Java Language Specification 中规定了一个类只会被初始化一次.所以该方法是**线程安全**的, 但是其在方法调用前就初始化了, 比较浪费资源.

### 优点

1. 只有一个实例, 节约内存空间, 减少了系统的性能开销, 如果某一个对象的产生需要比较多的资源时, 可以在启动时直接产生一个单例对象, 使其永驻内存.
2. 可以避免对资源的多重占用,
3. 可以在系统设置全局的访问点, 优化和共享资源访问.

### 缺点

1. 单例模式一般没有接口, 扩展很困难, 除了修改代码基本上没有第二种途径可以实现
2. 单例模式不利于测试, 如果单例模式没有完成, 无法进行测试
3. 于单一职责原则相冲突.

### 其它实现方式

#### 懒汉模式

```java
class Singleton {
    private static Singleton instance;

    private Singleton(){}

    public static synchronized Singleton getInstance(){
        if(instance==null){
            instance=new Singleton();
        }
        return instance;
    }
}
```

该实现只会在需要的时候才会进行初始化且是线程安全的, 但是每次调用`getInstance()`都会进行同步, 会浪费资源

#### Double Check Lock

```java
class SingletonDCL{
    private static SingletonDCL sInstance;

    private SingletonDCL(){}

    public static SingletonDCL getInstance(){
        if(sInstance==null){
            synchronized (SingletonDCL.class){
                if(sInstance==null){
                    sInstance=new SingletonDCL();
                }
            }
        }

        return sInstance;
    }
}
```
该实现只会在需要的时候才会进行初始化, 看似线程安全, 但实际并不是.

假设线程A执行到`sInstance=new SingletonDCL()`, 这句代码并不是一个原子操作, 这句代码大致会被分为下面3个步骤来处理:
1. 给SingletonDCL的实例分配内存
2. 调用SingletonDCL的构造函数, 初始化成员字段
3. 将sInstance对象指向分配的内存空间 (此时sInstance就不是null了).

如果这句代码严格按照这个顺序执行,该DCL单例模式便是线程安全的, 但是事实并非如此. 原因是JVM并没有保证上述第2和第3步的执行顺序.
也就是说执行步骤可能是1-3-2, 这种执行步骤就会出问题:


当先执行第3步时, 另一个线程B开始执行`getInstance()`, 由于此时`sInstance`已经不是`null`了, 所以线程B会返回一个还未初始化的`sInstance`, 出现了错误.

JVM1.5之后改善了这个问题, 在`sInstance`前加上`volatile`关键字可以确保线程安全.
即`private static volatile SingletonDCL sInstance;`

#### 静态内部类单例模式

DCL单例模式并不推荐使用,《Java并发编程实践》推荐使用下面这个方法:

```java
class SingletonStatic{
    private SingletonStatic(){}

    public static SingletonStatic getInstance(){
        return SingletonStaticHolder.sInstance;
    }

    private static class SingletonStaticHolder{
        private static final SingletonStatic sInstance=new SingletonStatic();
    }
}
```

第一次加载时, 并不会初始化`sInstance`, 只在第一调用`getInstance()`时初始化, 且该方法是安全的.

#### 枚举单例

```java
enum SingletonEnum{
    INSTACNE;
    public void doSomething(){
        StdOut.println("doSomething...");
    }
}
```

枚举单例模式有以下3个优点:
1. 线程安全, 任何时候都只有一个实例
2. 反序列化时, 都只会有一个实例
3. 可以防止反射攻击


### 选择哪一种实现方式

无论采用哪一种实现方式, 都要确保线程安全, 防止反序列化导致重新生成实例对象等一些问题. 具体选择哪一种实现方式取决于项目本身.

### 关于序列化

除了枚举单例, 为了避免单例对象在被反序列化时重新生成对象, 必须加入以下方法

```java
private Object readResolve() throws ObjectStreamException{
    return sInstance;
}
```

### 单例模式扩展

如果生成对象的数量不受限制, 可以直接使用`new`. 如果只要有一个对象, 使用单例模式即可, 若需要且只需要两个或者三个对象, 则可以按照下面的方法:
