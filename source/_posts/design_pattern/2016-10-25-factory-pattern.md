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
  public abstract <T extends Product> T createProduct(Class<T> c){
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
