---
author: ivyxjc
date: 2016-12-20
title: Spring装配bean
category: JavaWeb
tags: [javaweb,spring]
keywords:
description: Spring装配bean
toc: true
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

### 开启组件扫描


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

利用JavaConfig来装配bean是比较好的方案. 相对于使用XML文件来装配bean, 它更为强大, 类型安全且更便于重构.


### 创建配置类

创建配置类, 为类添加@Configuration的注解.

```java
@Configuration
public class BeanConfig{
    ...
}
```

### 声明bean

利用JavaConfig来声明bean, 需要编写一个创建所需类型的实例的方法, 并在该方法上添加@Bean注解.

```java
@Bean
public ClassName bean1(){
    return new ClassName();
}
```

@Bean的注解会告诉Spring该方法会返回一个ClassName实例, 该对象将被注册为bean, 方法体中包含了最终产生的bean的具体方法.

该bean的ID默认即为方法名, 当然可以使用`@Bean(name="id")`自定义bean的id.


```java
@Bean
public Class1 classBean1(){
    return ...;
}

public ClassName bean1(){
    return new ClassName(classBean1);
}


public ClassName bean2(){
    return new ClassName(classBean1);
}

```

默认情况下, Spring的bean都是单例, 所以当有多个注入方法中都使用到了classBean1方法的时候, 事实上传入bean1和bean2方法中的Class1实例为同一个.

```java
@Bean
public ClassName bean1(Class1 class1){
    return new ....;
}
```

在这里, `bean1`方法请求了一个Class1参数. 当Spring调用bean1方法来创建一个ClassName bean的时候, 会自动地装配一个Class1到该方法中. 这种方法是比较好的, 使用这种方法, 它不会要求Class1也在同一个配置类中声明. Spring可以自动扫描多个JavaConfig类, 甚至XML文件来实现这种装配.

## XML装配bean

XML在Spring早期是描述配置的主要书写方式, 但是现在更因为依赖于自动化配置和依赖Java的配置. 一个很主要的原因即在于, 使用XML文件来配置时, 大量的class属性是以字符串形式来保存的, 并不能在编译期间接受类型检查. 不过可以利用IDE的自动感知功能来确保XML配置中类型的正确.


最基本的XML配置文件如下:

```xml


```

### 声明一个简单的bean

可以按照如下方式声明bean:

```xml
<bean class="ClassName" />
```

可以向这个bean添加id:

```xml
<bean id="id" class="ClassName" />
```

### 构造器注入初始化bean

两种方案来构建构造器注入:
1. `<construct-arg>`元素
2. `c-`命名空间

在大部分情况下, 这两种方式基本上是相同的, 但是对于部分情况, 只有`<construct-arg>`能够做到(例如注入集合).


#### 构造器注入bean引用

```xml
<bean id="id" class="ClassName"
    <construct-arg ref="bean1"/>
</bean>
```

创建一个ClassName实例,其中将一个ID为bean1的bean引用传递到ClassName的构造器中.

使用c-命名空间:<br>

```xml
<bean id="id" class="ClassName"
    <c:arg-ref="bean1"/>
</bean>

<bean id="id" class="ClassName"
    <c:_0-ref="bean1"/>
</bean>

<bean id="id" class="ClassName"
    <c:_-ref="bean1"/>
</bean>
```

1. 其中c:表示c-命名空间前缀, arg是构造器中相应的参数名称, -ref表示注入的是bean引用. bean1为要注入的bean的ID.
2. 也可以使用参数在参数列表中的位置来注入bean, _0, _1 ...来表示对应的参数
3. 如果只有一个参数, 可以只使用_


#### 构造器注入字面量

```java
public ClassName(String title, String artist){
    ...
}
```

```xml
<bean id="id" class="ClassName"
    <construct-arg value="some strings"/>
    <construct-arg value="some strings"/>
</bean>
```

使用c-命名空间:<br>

```xml
<bean id="id" class="ClassName"
    <c:_title="some strings"/>
    <c:_artist="some strings"/>
</bean>

<bean id="id" class="ClassName"
    <c:_0="some strings"/>
    <c:_1="some strings"/>
</bean>
```

与装配bean引用的区别就在与删除了最后的-ref. 如果只有一个参数, 同样可以使用 _ . 


#### 构造器注入集合

```java
public ClassName(String title, String artist, List<String> tracks){
    ....
}
```

```xml
<bean id="id" class="ClassName"
    <construct-arg value="some strings"/>
    <construct-arg value="some strings"/>
    <construct-arg>
        <list>
            <value>some strings</value>
            <value>some strings</value>
            <value>some strings</value>
        </list>
    </construct-arg>
</bean>
```

如果bean引用的list的话, 将value改为ref即可. 如果需要使用集合, 将list改为set即可.

### 属性注入初始化bean


```java
@Autowired
public void setSomeArg(Class1 arg1){
    ...
}

```

```xml
<bean id="id" ClassName="ClassName">
    <property name="someArg" ref="bean1" />
</bean> 
```

`<property>`元素为属性的Setter方法提供的功能与`<construct-arg>`元素为构造器所提供的功能是一样的, 用法也类似, 同样可以使用p-命名空间来代替.

```
<bean id="id" ClassName="ClassName">
    <p:someArg-ref="bean1" />
</bean> 
```