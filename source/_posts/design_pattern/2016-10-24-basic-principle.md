---
author: ivyxjc
date: 2016-10-24
title: 面向对象六大原则(上)
category: OO
tags: [linux]
keywords:
description: 设计模式中的六大原则：单一职责原则, 里氏替换原则, 依赖倒置原则
toc: true
---

## 单一职责原则

单一职责原则(Single Responsibility Principle,SRP)简而言之就是对于一个类或者接口, 引起其改变的应该只能有一个原因. 比如要将负责属性和行为的类分开.


## 里氏替换原则

定义：所有引用基类的地方必须能透明地使用其子类的对象. 只要父类出现的地方, 子类就可以出现, 而且替换为子类不会产生任何错误或者一场. 但是反过来不一定可行.




1. 子类中可以增加自己特有的方法。
2. 当子类的方法重载父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。
3. 当子类的方法实现父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格。

### 前置条件

当子类的方法**重载**父类的方法时，方法的前置条件（即方法的形参）要比父类方法的输入参数更宽松。

```java
class Father {

    public Collection doSomething(HashMap map){
        StdOut.println("父类被执行...");
        return map.values();
    }
}

class Son extends Father{

    public Collection doSomething(Map map) {
        StdOut.println("子类被执行");
        return map.values();
    }
}
```

子类方法和父类方法, 方法名相同, 返回类型相同, 但是参数不同, 所以不是Override, 而是Overload. 在这种情况下, 如果传入`HashMap`, 子类的`doSomething()`不会被执行.
这是**正确**的, 因为子类并没有重写父类方法, 而是重载父类方法, 所以


```java
public static void main(String[] args){
    Father f=new Father();
    HashMap map=new HashMap();
    f.doSomething(map);
}

父类被执行
```

```java
public static void main(String[] args){
    //父类出现的地方都可以被子类代替, 且不会改变逻辑
    Son f=new Son();
    HashMap map=new HashMap();
    f.doSomething(map);
}

父类被执行
```

```java
public static void main(String[] args){
   //子类出现的地方, 父类不一定可以代替
    Son f=new Son();
    Map map=new HashMap();
    f.doSomething(map);
}

子类被执行
```


如果父类的前置条件(形参) 范围宽于子类则不正确.

```java
class Father {

    public Collection doSomething(Map map){
        StdOut.println("父类被执行...");
        return map.values();
    }
}

class Son extends Father{

    public Collection doSomething(HashMap map) {
        StdOut.println("子类被执行");
        return map.values();
    }
}
```


```java
public static void main(String[] args){
    Father f=new Father();
    HashMap map=new HashMap();
    f.doSomething(map);
}

父类被执行
```

```java
public static void main(String[] args){
    //父类出现的地方都可以用子类代替
    Son f=new Son();
    HashMap map=new HashMap();
    f.doSomething(map);
}

子类被执行
```

可以注意到, 此时子类方法被执行了, 而子类并没有重写父类的相应的方法, 而是重载了父类的方法.

```java
public static void main(String[] args){
    Son f=new Son();
    Map map=new HashMap();
    f.doSomething(map);
}

父类被执行
```

### 后置条件

当子类的方法实现或覆写父类的抽象方法时，方法的后置条件（即方法的返回值）要比父类更严格或相同。


### 注意点

1. 在类中调用其他类时务必使用其父类或者接口, 如果不能使用父类或者接口, 则说明类的设计已经违背了LSP原则.
2. 如果子类不能完整地实现父类的方法, 或者父类的一些方法在子类中完全和父类业务逻辑不同, 则建议不使用继承关系, 而是用依赖, 聚集, 组合等关系代替.


## 依赖倒置原则

Dependence Inversion Principle

1. 高层模块不应该依赖低层模块
2. 抽象不应该依赖细节
3. 细节应依赖抽象

|||
|--|--|
|高层模块|原子逻辑再组装就是高层模块|
|低层模块|每一个逻辑的实现都由院子逻辑组成,不分割的原子逻辑就是低层模块|

对于Java来说.

1. 模块间的依赖通过抽象产生,实现类之间不发生直接依赖关系, 其依赖关系是通过接口或者抽象类产生的
2. 接口或这抽象类不依赖于实现类
3. 实现类依赖接口或抽象类
