---
author: ivyxjc
date: 2016-04-26
title: 283 将0移到数组末尾
category: OJ
tags: [array,twoPointer]
keywords:
description: 将数组中的0移动数组尾部, 其它数字顺序不变. 必须就地完成.
---

将数组中的0移动数组尾部，其它数字顺序不变。
必须就地完成

## 解法I（用时34ms）

就是遍历数组，当一个数为0时，将该0和后面每一个数字都替换

用zeroFinalIndex作为一个flag,该数后面的数都为0，初始值为nums.length。每有一个数为0，zeroFinalIndex--

注意点：当有连续的0时，两个0替换，替换完成时nums[i]==0，此时i不应该+1.

```java
public void moveZeroes(int[] nums) {
    int zeroFinalIndex=nums.length;
    for(int i=0;i<nums.length;){
        if(nums[i]==0&&i<zeroFinalIndex){
            int j=i+1;
            if(j<zeroFinalIndex&&nums[j]==0){
                i--;
            }
            while(j<zeroFinalIndex){
                exch(nums,j-1,j);
                j++;
            }
            zeroFinalIndex--;
        }
        i++;

    }
}
```



## 解法II（用时1ms）

即将不为0的数字和第一个0交换，关键在于找到第一个0的位置 `zeroIndex`。

首先将`zeroIndex`设为0，即使该数不为0，也只是自己和自己交换，不影响结果。

当遇到一个数为0，此时`zeroIndex`等于该数的秩（即第一个0的秩序），等到`numsIndex`增加到一个不为0的数，和它交换。

`zeroIndex`再+1仍是第一个0所在的位置。
因为若只有一个0，则`nusm[zeroIndex]`和`nums[numsIndex]`交换后，`zeroIndex`所在的数仍为0。

若有连续多个0，第一0交换后，`zeroIndex++`后仍为交换后第一个0所在的位置。

```java
public void moveZeroesII(int [] nums){
    for(int zeroIndex=0,numIndex=0;numIndex<nums.length;numIndex++){
        if(nums[numIndex]!=0){
            exch(nums,zeroIndex++,numIndex);
        }
    }

}
```
