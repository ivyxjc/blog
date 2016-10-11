---
layout: post
title: Sunshine开发实战:测试
category: Android
tags: [android,android_test]
keywords:
description:
---

## 简单的例子

```java
public class TestPractice extends AndroidTestCase {
    //测试前进行的工作
    @Override
    protected void setUp() throws Exception {
        super.setUp();
    }

//所有测试的类必须以test开头
    public void testThatDemonstratesAssertations() throws Throwable{
        int a=5;
        int b=3;
        int c=5;
        int d=10;

        assertEquals("X should be equal",a,c);
        assertTrue("Y should be true",d>a);
        assertFalse("Z should be false",a==b);

    }

    //测试之后进行的工作
    @Override
    protected void tearDown() throws Exception {
        super.tearDown();
    }
}
```
然后点击`run 'Tests in ...`即可.
![](/assets/img/posts/android_test_run.png)
