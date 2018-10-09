---
author: ivyxjc
date: 2018-10-04
title: Java 11 的新特性(下)
category: Java
tags: [Java]
keywords:
description:
toc: true
---

## JEP 309 Dynamic Class-File Constants 
JVM规范中在常量池中添加一个新的类型 CONSTANT_Dynamic

## JEP 315 Improve Aarch64 Intrinsics 


## JEP 318 Epsilon: A No-Op Garbage Collector
一个新的垃圾收集器，在GC时并不执行任何操作。
主要有一下几的目的：
1. 性能测试。便于在性能测试的时候排除GC的影响
2. 内存压力测试
3. VM接口测试。
4. 短生命周期任务。 短生命周期的任务希望能够非常快速地得到响应，对于no-op GC，可以排除GC对此造成的影响。
5. Last-drop latency improvements 
6. Last-drop throughput improvements. 

## JEP 327 Uincode 10
支持最新的Unicode

## JEP 328 Flight Recorder

## JEP 330 Launch Single-File Source-Code Programs
合并之前的javac以及javap。

```
javac A.java
java A 1 2

Java 11:
java A.java 1 2
```

## JEP 332 Transport Layer Security (TLS) 1.3
实现了RFC 8446中的TLS1.3

## JEP 333 ZGC: A Scalable Low-Latency Garbage Collector (Experimental)

引入ZGC垃圾收集器，主要实现了以下几个目标：
1. 停顿时间不超过10ms
2. 能够处理小(几百兆)的到非常大(几TB)的heap大小
3. 相比于G1垃圾收集器，不超过15%的吞吐量下降
4. 为将来的GC收集器奠定基础


## remove and deprecate
### JEP 320 Remove the Java EE and CORBA Modules
删除Java9已经标记为Deprecate的Java EE以及CORBA模块。主要包括

1. JAX-WS: Java API for XML-Based Web Service
2. JAXB: Java Architecture for XML Binding
3. JAF: JavaBeans Activation Frameword


### JEP 335 Deprecate the Nashorn JavaScript Engine 

### JEP 336 Deprecate the Pack200 Tools and API