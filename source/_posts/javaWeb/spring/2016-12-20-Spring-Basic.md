---
author: ivyxjc
date: 2016-12-20
title: Spring基础
category: JavaWeb
tags: [javaweb,spring]
keywords:
description: Spring装配bean
---

Spring有三种装配机制:
1. 在XML文件进行显示配置
2. 在Java中进行显示配置
3. 隐式的bean发现机制和自动装配


<!--more-->


## 自动化装配bean

### 使得bean可以被发现

```java
public interface CompactDisc {
    void play();
}
```

```java
@Component
public class SgtPeppers implements CompactDisc{

    private String title="title";
    private String artist="artist";

    @Override
    public void play() {
        System.out.println("Playing "+title+" by "+artist);
    }
}
```

在类上使用@Component注解, 这个注解表明该类会作为组件类.

###　开启组件扫描


#### Java开启组件扫描

在类前添加@ComponentScan注解, 即可开启组件扫描, 默认扫描本类所在的包及其子包, 查找所有带@Component注解的类.

```java
@Configuration
@ComponentScan
public class CDPlayerConfig {
}
```

### 扫描其它的包

可以给ComponentScan添加相关的属性来自定义需要扫描的包:

1. ComponentScan("packageName")
2. ComponentScan(basePackage={"package1","package2"})
3. ComponentScan(basePackageClasses={Class1.class,CLass2.class})


方案1将扫描指定包名, 方案2将扫描指定的多个包及其子包, 方案3将扫描这个几个类所在的包及其子包. 在第三种方法中, 我们可以在包中添加一个空的接口, 然后使用该接口来作为扫描的标记.

#### XML文件开启组件扫描


```xml
<context:component-scan base-package="packageName"/>
```



### 添加注解自动装配

```java
@Autowired
public void setCompactDisc(CompactDisc cd){
    this.cd = cd;
}
```

如果没有匹配的bean, 在创建应用上下文时, Spring会抛出一个异常. 可以使用`@Autowired(required=false)`来避免此异常. 此时, Spring会尝试自动装箱, 但是如果没有匹配的bean时, Spring会让该bean处于未装配的状态. 如果此时调用未装配状态的属性时 可以能会抛出NullPointerException.

## Java装配bean


## XML装配bean

XML在Spring早期是描述配置的主要书写方式, 但是现在更因为依赖于自动化配置和依赖Java的配置. 一个很主要的原因即在于, 使用XML文件来配置时, 大量的class属性是以字符串形式来保存的, 并不能在编译期间接受类型检查. 不过可以利用IDE的自动感知功能来确保XML配置中类型的正确.

