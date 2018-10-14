---
author: ivyxjc
date: 2018-10-04
title: Java 11 的新特性(上)
category: Java
tags: [Java]
keywords:
description:
toc: true
---

Java 11中的新特性（上）

<!--more-->

## JEP 181 Nest-Based Acess Control
这一提案是为了增强自Java 1.1即引入的嵌套类设计。

嵌套类主要有两个用处。

第一是因为其只使用于很短的代码块中，在Java8之前，这主要依靠实现一个匿名类来完成。Java8之后，这种用法可以被lambda表达取代。

另一种用法是因为需要访问另一个类的内部。嵌套类具有和成员变量以及成员方法相同的访问权限。

JEP181主要是为了解决JVM级别的权限与源码权限不一致的问题。

### 代码分析

```java
public class JEP181 {

    public class Nest1 {
        private int var1;

        public void doSome() throws Exception {
            final Nest2 nest2 = new Nest2();
            nest2.var2 = 2;
            System.out.println(nest2.var2);
            final Field f2 = Nest2.class.getDeclaredField("var2");
            f2.setInt(nest2, 2);
            System.out.println(nest2.var2);
        }
    }

    public class Nest2 {
        private int var2;
    }

    public static void main(String[] args) throws Exception {
        JEP181 jep181 = new JEP181();
        JEP181.Nest1 nest1 = jep181.new Nest1();
        nest1.doSome();
    }
}

result:
java 11: 2 2
java 8: java.lang.IllegalAccessException
```
这一段代码在Java8中是不正确的。会抛出`java.lang.IllegalAccessException`。主要问题出在`f2.setInt(nest2,2)`，这里由于在`Nest2`中是private的，所以无法直接set值。但是却又可以直接调用`nest2.var2=2`来设置该值，因为嵌套类是可以访问别的嵌套类的私有属性的。Java 11修复了这个令人困惑的现象。

>JEP 181 官方介绍<br />
A field or method R is accessible to a class or interface D if and only if any of the following conditions are true:<br />
...<br />
R is private and is declared in a different class or interface C, and C and D, are nestmates


## JEP 321 HttpClient

`HttpClient`在Java9开始引入，Java10对此有所更新。Java11根据一些反馈对API进行了一些改进，但是大部分都没有变化。该API通过CompletableFutures提供了非阻塞request和response,
关于请求和响应的背压机制以及流控制都由Java11新提供的Flow API来提供。

虽然API没有什么变化，但是实现几乎全部重写了。Java11的全部实现都是异步的（Java9,10的Http/1.1的实现是阻塞的）.


### 同步

```java
static void syncGet()
        throws IOException, InterruptedException, URISyntaxException {
        HttpClient httpClient = HttpClient.newHttpClient();

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("http://www.baidu.com"))
            .timeout(Duration.ofSeconds(20))
            .header("Content-Type", "application/json")
            .build();
        HttpResponse<Path> response =
            httpClient.send(request, HttpResponse.BodyHandlers.ofFile(Paths.get("abc.txt")));
        log.debug("Response status code: " + response.statusCode());
        log.debug("Response headers: " + response.headers());
        log.debug("Response body: " + response.body());
    }
```
关于同步的用法都比较简单，和别的很多http库的设计也比较相像。

### 异步
```java
 static void asyncGet() {
        HttpClient httpClient = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("http://www.baidu.com"))
            .build();

        CompletableFuture<String> strResponse =
            httpClient.sendAsync(request, HttpResponse.BodyHandlers.ofString())
                .thenApply(HttpResponse::body);
        strResponse.whenComplete((resp, t) -> {
            if (t != null) {
                log.debug(t.getMessage());
            } else {
                log.debug(resp);
            }
        }).join();
    }
```

## JEP 323 Local-Variable Syntax for Lambda Parameters 

Java 9，Java 10引入了var用来简化声明变量的过程，Java 11进一步增强了该特性。

```java
list.sort((@NotNull var t1, @NotNull var t2) -> {
            if (t1.equals(t2)) {
                return 0;
            }
            return t1 > t2 ? 1 : -1;
        });
```
在Java 11之前，上述代码未能正确执行。
`(var x, var y)->...`在一般情况下并没有什么用，但是如果需要给lambda表达式变量添加注解的话，那么lambda中可以使用var就有了作用。
