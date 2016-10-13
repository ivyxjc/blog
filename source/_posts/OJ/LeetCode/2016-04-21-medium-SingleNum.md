---
author: ivyxjc
date: 2016-04-21
title: LeetCode SingleNum 136,127,260
category: OJ
tags: [quicksort,sort,bit manipulation]
keywords:
description:
---

这三题题意基本一样。
1.136 题：每个数字出现两遍，一个数字例外。找出这个数字。
2.137 题：每个数字出现三遍，一个数字例外。找出这个数字。
3.260 题：每个数字出现两遍，两个数字例外。找出这个数字。

## 使用排序的方法
以136题为例：

这种方法先将给的数组`int[] num`排序，因为数字相同则会在一起，则只需要比较`num[i]`和`nums[i+1]`若不相同，则返回`num[i]`其中`i=i+2`。
特殊情况：
数字出现在最末尾。

```java
public int singleNumber(int[] nums) {
    sort(nums);
    int res=0;
    for(int i=0;i<nums.length-1;){
        if(nums[i]==nums[i+1]){
            i+=2;
        }else{
            res=nums[i];
            i+=2;
            return res;
        }
    }
    return nums[nums.length-1];
}
```

## 使用位操作的方法

### 136题

因为数字会出现两次，所以对所有数字取**异或**会使得最后的数字即所求的数字。


```java
public int singleNumber(int[] nums) {
    int diff=0;
    for(int i:nums){
        diff=diff^i;
    }
    int res=diff;
    return res;
}
```

### 137题

因为每个数字会重复三遍，所以统计32位中每一位上1出现的次数，若次数不为3的整数倍，则说明所求数字该位为1。

```java
public int singleNumber(int[] nums) {
    int [] count=new int[Integer.BYTES*8];
    int res=0;
    for(int i=0;i<count.length;i++){
        int tmp=1<<i;
        for(int j=0;j<nums.length;j++){
            if((nums[j]&tmp)!=0){
                count[i]++;
            }
        }
        if(count[i]%3!=0){
            res=res|tmp;
        }
    }

    return res;
}

```


### 260题

同样是先对所有数字取异或，因为有两个数字只出现了一次，所以最终的结果`diff`是这两个数字的异或值。

因为这两个数字不相同，所以这两个数字的二进制必定有某一位不相同。假设在`Xth`位上，一个数为0，另一个数为1。

那么把所有数字分为两类，一类是在`Xth`为0的，另一类是在`Xth`为1的。

关键是如何找到`Xth`位。方法时`diff ^=-diff`。
该式子会找出最右侧的两个数字不同的位置，并将该为置为1，其余位置为0。其实就是找`diff`中为1的最右边的位置。


```java
public int[] singleNumber(int[] nums) {
    int diff = 0;
    for (int num : nums) {
        diff ^= num;
    }

    diff &= -diff;


    int[] rets = {0, 0};
    
    for (int num : nums)
    {
    //将数字分为两类，一类为某一位为0，另一类为某一位为1
        if ((num & diff) == 0)
        {
            rets[0] ^= num;
        }
        else
        {
            rets[1] ^= num;
        }
    }
    return rets;
}
```


