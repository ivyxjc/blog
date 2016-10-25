---
author: ivyxjc
date: 2016-10-24
title: 面向对象六大原则(下)
category: OO
tags: [设计模式]
keywords:
description: 设计模式中的六大原则：接口隔离原则, 迪米特法则, 开闭原则
toc: true
---

## 接口隔离原则
Interface Segregation Principles(ISP)

1. 客户端不应该依赖它不需要的接口
2. 类似的依赖关系应该建立在最小的接口上

接口应该细化, 不要使用过于臃肿的接口. 客户端需要什么接口就提供什么接口, 将不需要的接口剔除掉.
不要将太多的方法放在同一个接口之中.

但是接口设计也要有度, 不可过度设计, 这个度往往根据经验和常识判断.

## 迪米特法则

Lswo of Demeter(LOD) , 最少知识f原则(Least Knowledge Principle))

一个对象应该对其它对象有最少的了解, 另一个解释是只与直接的朋友通信.

```
朋友类: 出现在成员变量, 方法的输入输出参数中的类称为成员朋友类, 而出现在方法体内部的类不属于朋友类
```

## 开闭原则

Open Close Principle(OCP)

一个软件实体如类, 模块和函数等应该对扩展开放, 对修改关闭.

### 实例
```java
interface IBook{

    public String getName();

    public int getPrice();

    public String getAuthor();
}

class NovelBook implements IBook{
    private String name;
    private int price;
    private String author;

    public NovelBook(String name, int price, String author){
        this.name=name;
        this.price=price;
        this.author=author;
    }

    @Override
    public String getName() {
        return name;
    }

    @Override
    public int getPrice() {
        return price;
    }

    @Override
    public String getAuthor() {
        return author;
    }
}
```

如果将来要搞打折, 一般可能会用以下两个方法来解决:

1. 在接口中添加`getOffPrice()`方法. 但是这需要对每一个实现`IBook`接口的实现类都添加该方法, 工作繁琐. 且接口应该是稳定且可靠的, 不应该经常发生变化.
2. 修改实现类, 直接在`getPrice()`中实现打折处理, 但是如果仍需要知道原价是多少, 就会出问题.
3. 通过扩展实现

添加一个子类

```java
class OffNovelBook extends NovelBook{

    public OffNovelBook(String name,int price, String author){
        super(name,price,author);
    }

    @Override
    public int getPrice(){
        int selfPrice=super.getPrice();
        int offPrice=selfPrice*90/100;
        return offPrice;
    }
}
```

![](http://oezmbgg4j.bkt.clouddn.com/ocp_.jpg)
