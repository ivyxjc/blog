---
author: ivyxjc
date: 2018-03-26
title: class.getResource() class.getClassLoader.getResource()之间的区别
category: Java
tags: [Java]
keywords:
description:
toc: true
---

使用`*.class.getResouce()`和`*.class.getClassLoader.getResource()`是有一定的区别的。

在maven，gradle等构建工具构建的项目，resouces文件夹下的内容默认在classpath下面。

所以可以直接用`this.class.getClassLoader.getResource(...)`来获取对应的文件。

<!--more-->

##  Class与ClassLoader.getResource()之间的区别

使用`*.class.getResouce()`和`*.class.getClassLoader.getResource()`是有一定的区别的。

`*.class.getResouce()`先会`resolveName(name)`然后在委托给`classLoader`来处理。所以区别主要在于`resolveName(name)`。 `resolveName(name)`主要是功能是：若`name`以`/`开头，那么则直接调用`classLoader`来处理。若没有以`/`开头，则会将类全名中的点替换成`/`作为路径再加上name委托给`classLoader`来处理。

所以要获取`resources`文件夹下的内容，`*.class.getResouce(name)`中的`name`要以`/`开头。而`this.class.getClassLoader.getResource(...)`不需要。

```java
public java.net.URL getResource(String name) {
    name = resolveName(name);
    ClassLoader cl = getClassLoader0();
    if (cl==null) {
        // A system class.
        return ClassLoader.getSystemResource(name);
    }
    return cl.getResource(name);
}

private String resolveName(String name) {
        if (name == null) {
            return name;
        }
        if (!name.startsWith("/")) {
            Class<?> c = this;
            while (c.isArray()) {
                c = c.getComponentType();
            }
            String baseName = c.getName();
            int index = baseName.lastIndexOf('.');
            if (index != -1) {
                name = baseName.substring(0, index).replace('.', '/')
                    +"/"+name;
            }
        } else {
            name = name.substring(1);
        }
        return name;
    }
```

