---
author: ivyxjc
date: 2016-04-26
title: 数根
category: OJ
tags: [math]
keywords:
description:
---

## 简介

数根就是指将一个数的各位数字相加之后得到一个数字，若该数大于等于10则继续该运算所得到的那个数。

例如：
158 --》 1+5+8=14 --》 1+4=5


## O(n) 解法

就是按照树根的定义来做。

```java
public int addDigits(int num) {
   while(num>=10){
       int [] tmp=getDigits(num);
       num=0;
       for(int i=0;i<tmp.length;i++){
           num+=tmp[i];
       }
   }
   return num;
}


public int[] getDigits(int num){
   int tmp=num;
   int [] digits=new int[(int)Math.log(num)];
   int index=0;
   while(tmp!=0){
       digits[index]=tmp%10;
       tmp=tmp/10;
       index+=1;
   }
   return digits;
}
```

## O(1)解法

树根有一些很巧妙的数学规律，使得该题可以在O(1)时间内完成。

例：
$$12345=1*10000+2*1000+3*100+4*10+5*1$$

$$ =1*9999+2*999+3*99+4*9+5*0+(1+2+3+4+5)$$

我们发现一个数的各位数字之和和该数和9同余。

所以树根与原数也应该同余（当一个数为9的倍数时，树根即为9）、

```java
public int addDigits(int num) {
    if(num==0){
        return 0;
    }
    if(num%9==0){
        return 9;
    }else{
        return num%9;
    }

}
```
