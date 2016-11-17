---
author: ivyxjc
date: 2016-10-25
title: 工厂方法模式
category: OO
tags: [设计模式]
keywords:
description: 设计模式中的工厂方法模式
toc: true
---

## 工厂方法的通用源码

### 抽象产品类

```java
public abstract class Product{
  public void method(){

  }
  public abstract void method2();
}
```
### 具体产品类
```java
public class ConcreteProduct1 extends Product{
  public void method2(){

  }
}

public class ConcreteProduct2 extends Product{
  public void method2(){

  }
}
```

### 抽象工厂类

```java
public abstract class Creator{
  public abstract <T extends Product> T createProduct(Class<T> c);
}
```

### 具体工厂类

```java
public class ConcreteCreator extends Creator{
  public <T extends Product> T createProduct(Class<T> c){
    Product product=null;
    try{
      product=(Product)Class.forName(c.getName()).newInstance();
    }catch(Exception e){
      ....
    }
    return (T)product;
  }
}

```

##  简单工厂模式

简单工厂模式去掉了抽象工厂类, 并且`createProduct()`方法添加了`static`. 该模式调用过程更为简单, 方便理解. 但是扩展较为困难, 不符合开闭原则.

## 多工厂模式

## 利用工厂模式生成单例


```java
class Singleton{
    private Singleton(){}
    public void doSomething(){}
}
public class FactorySingle {
    private static Singleton sSingleton;

    static {
        try{
            Class cl=Class.forName(Singleton.class.getName());
            Constructor constructor=cl.getDeclaredConstructor();
            constructor.setAccessible(true);
            sSingleton=(Singleton)constructor.newInstance();
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public static Singleton getSingleton(){
        return sSingleton;
    }
}
```

## 抽象工厂模式

为创建一组相关或者是相互依赖的对象提供一个接口, 而不需要制定它们的具体类.
