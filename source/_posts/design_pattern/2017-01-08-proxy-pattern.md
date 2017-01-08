---
author: ivyxjc
date: 2017-01-08
title: 代理模式
category: OO
tags: [设计模式]
keywords:
description: 
toc: true
---


代理模式也叫委托模式, 应用非常广泛. 其定义:<br>
Provide a surrogate or placehoder for another object to control access to it.(为其它对象提供一种代理以控制对这个对象的访问)

<!--more-->


## 基本用法

### 抽象主题类

```java
public interface Subject{
    //定义一个方法

    public void request();
}

```

###真实主题类

```java
public class RealSubject implements Subject{
    @Override
    public void request(){

    }
}
```

### 代理类

```java
public class Proxy implements Subject{
    private Subject subject = null;

    public Proxy(){
        this.subject=new Proxy();
    }

    public Proxy(Subject subject){
        this.subject=subject;
    }

    public Proxy(Object... objects){
        //todo
    }

    public void request(){
        //todo
    }

    public void before(){
        //todo
    }

    public void after(){
        //todo
    }
}
```

一个代理类可以代理多个被代理者, 而在使用时, 该代理类到底代理哪个真实类, 可以由场景类决定.


## 优点

1. 职责清晰
2. 高扩展性
3. 智能化

## 代理分类

设计模式中的代理分为普通代理和强制代理.

###　普通代理

普通代理的要求便是: 客户端只能访问代理角色, 而不能访问真实角色. 可以通过代理角色传递真实角色所需要的参数.

```java

public interface IGamePlayer {

    public void login(String user, String password);
}


public class GamePlayer implements IGamePlayer {
    private String name="";

    public GamePlayer(IGamePlayer gamePlayer,String name)throws Exception{
        if(gamePlayer==null){
            throw new Exception("不能创建真实角色");
        }else{
            this.name=name;
        }
    }

    @Override
    public void login(String user, String password) {
        System.out.println("登录名"+user+"的用户"+this.name+"登录成功!");
    }

}

public class GamePlayerProxy implements IGamePlayer {

    private IGamePlayer mIGamePlayer=null;

    public GamePlayerProxy(String name){
        try{
            mIGamePlayer=new GamePlayer(this,name);
        }catch(Exception e ){
            e.printStackTrace();
        }
    }
    @Override
    public void login(String user, String password) {
        this.mIGamePlayer.login(user,password);
    }
}
```

```java
客户类代码

public static void main(String[] args){
    IGamePlayer proxy=new GamePlayerProxy("张三");
    
    System.out.println("start time");  
    proxy.login("zhangsan","password");
    System.out.println("end time");
}
```

###　强制代理

强制代理要求必须通过真实角色去寻找代理角色, 否则不能访问.

```java

public interface IGamePlayer {
    public void login(String user, String password);

    //每个人都可以找到自己的代理
    public IGamePlayer getProxy();
}

public class GamePlayer implements IGamePlayer {
    private String name="";

    //我的代理是谁
    private IGamePlayer proxy=null;

    public GamePlayer(String name){
        this.name=name;
    }

    @Override
    public void login(String user, String password) {
        if(this.isProxy()){
            System.out.println("登录名"+user+"的用户"+this.name+"登录成功!");
        }else{
            System.out.println("请使用指定的代理访问");
        }
    }

    @Override
    public IGamePlayer getProxy() {
        this.proxy=new GamePlayerProxy(this);
        return this.proxy;
    }

    private boolean isProxy(){
        if(this.proxy==null){
            return false;
        }else{
            return true;
        }
    }

}

public class GamePlayerProxy implements IGamePlayer {
    private IGamePlayer gameplayer=null;

    public GamePlayerProxy(IGamePlayer gamePlayer){
        this.gameplayer=gamePlayer;
    }

    @Override
    public void login(String user, String password) {
        this.gameplayer.login(user,password);
    }

    @Override
    public IGamePlayer getProxy() {
        return this;
    }
}
```

```java
客户类代码

public class Client {
    public static void main(String[] args){
        IGamePlayer gameplayer=new GamePlayer("张三");
        IGamePlayer proxy=gameplayer.getProxy();
        System.out.println("start time");
        proxy.login("zhangsan","password");
        System.out.println("end time");
    }
}
```

## 动态代理


```java

public class GamePlayIH implements InvocationHandler{
    //被代理者
    Class cls=null;

    //被代理的实例
    Object obj=null;

    public GamePlayIH(Object obj){
        this.obj=obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result=method.invoke(this.obj,args);
        if(method.getName().equalsIgnoreCase("login")){
            System.out.println("有人正在用我账号登录");
        }
        return result;
    }
}
```

```java

public class GamePlayer implements IGamePlayer {
    private String name="";

    public GamePlayer(String name){
        this.name=name;
    }

    @Override
    public void login(String user, String password) {
        System.out.println("登录名"+user+"的用户"+this.name+"登录成功!");
    }
}
```

```java
客户类代码
public static void main(String[] args) throws Throwable{
    IGamePlayer player=new GamePlayer("张三");
    InvocationHandler handler=new GamePlayIH(player);

    System.out.println("start time");
    //获取类的ClassLoader
    ClassLoader cl=player.getClass().getClassLoader();
    //动态生产一个代理者
    IGamePlayer proxy=(IGamePlayer) Proxy.newProxyInstance(cl,new Class[]{IGamePlayer.class},handler);
    
    proxy.login("zhangsan","password");
    System.out.println("end time    ");
}
```