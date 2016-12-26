---
author: ivyxjc
date: 2016-12-25
title: Spring高级装配
category: JavaWeb
tags: [javaweb,spring]
keywords:
description: Spring高级装配
toc: true
---

## 环境与配置

由于在项目开发时和项目配置时, 可能会有一些代码的不同, 如何根据不同的环境来进行配置. Spring有bean profile的功能. 

### 在JavaConfig中配置profile

在Java配置中, 使用@Profile注解来确定这个bean属于哪一个profile.

```java
public class Config{
    @Bean
    @Profile("dev")
    public ClassName function1(){
        ...
    }

    @Bean
    @Profile("prod")
    public ClassName function2(){
        ...
    }
}
```

### 在XML中配置profile


### 激活profile

Spring依据两个独立的属性来判断激活哪一个profile. 这两个独立的属性便是spring.profiles.active和spring.profiles.default. 如果设置了active参数, 则根据该值来判断激活哪一个profile. 若没有, 则根据default值来判断, 若两个值都未设置, 则只会加载那些没有定义在profile中的bean.

## 条件化bean

Spring4之后引入了@Conditional注解, 给定的计算结果为true, 则创建该bean, 否则该bean会被忽略.


```java
@Bean
@Conditional(MagicExistsCondition.class)
public .....{
    ...
}

```

```java
public class MagicExistsCondition implements Condition{
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata){
        ...
    }
}
```

其中ConditionContext是一个接口, 主要有以下几个作用:

1. 借助getRegistry()返回的BeanDefinitionRegistry检查bean定义
2. 借助getBeanFactory()返回的ConfigurableListableBeanFactory检查bean是否存在, 探查bean的属性
3. 借助getEnvironment()返回的Environment检查环境变量是否存在以及它的值是什么
4. 读取并探查getResourceLoader()返回的ResourceLoader所加载的资源
5. 借助getClassLoader()返回的ClassLoader加载并检查类是否存在

AnnotatedTypeMetadata可以检查带有@Bean注解的方法上还有什么注解

可以借助isAnnotated(String str) 来判断带有@Bean注解的方法是否还有特定的注解 

## 处理自动装配的歧义性

当有多个bean都满足某一装配要求时, 就会出现问题. 解决方法有:

1. 标示首选的bean
2. 使用限定符


###  标示首选的bean

@Primary可以来将一个bean标识为首选bean. @Primary既可以与@Component组合用在组件扫描的bean上, 也可以与@Bean组合用在JavaConfig文件中.


```java
@Componen
@Primary
public class ClassName implements InterfaceName{
    ....
}
```

```java
@Bean
@Primary
public  ClassName getBean1(...){
    ....
}
```

当然, 也可以使用XML文件来标识首选bean

```xml
<bean id="id" class="ClassName"
        primary="true" />
```

不可以将多个可能产生歧义的bean设为首选.

### 使用限定符

标示首选bean只可以解决较为简单的bean歧义问题, 如果问题较为复杂, 则无法解决. 

使用@Qualifier注解是使用限定符的主要方式. 它可以与@Autowired和@Inject协同使用.

```java
@Autowired
@Qualifier("bean_id")
public void setArg1(InterfaceName i){
    ...
}
```

@Qualifier注解声明了想要注入的bean的ID.  


@Qualifier也可以使用自定义的限定符而非bean的id, 方法如下:

```java
@Component
@Qualifier("qualifier_id")
public class ClassName implements InterfaceName{
    ...
}
```

### 使用自定义限定符

当一个限定符仍然不可以解决歧义问题, 就需要使用自定义限定符号(Java8之前版本不支持同一个条目重复出现多个相同类型的注解). 

注: Java8允许出现重复注解, 只要注解本身在定义的时候带有@Repeatable即可. 

```java
@Target({ElementType.CONSTRUCTOR,ElementType.FIELD,
        ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Cold{

}

@Target({ElementType.CONSTRUCTOR,ElementType.FIELD,
        ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Warm{

}
```

使用:

```java
@Component
@Cold
@Warm
public class ClassName implements InterfaceName{
    ...
}
```

## Bean的作用域
