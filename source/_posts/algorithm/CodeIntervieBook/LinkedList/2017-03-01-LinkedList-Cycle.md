---
author: ivyxjc
date: 2017-03-04
title: 链表相交以及链表含环的相关问题
category: Algorithm
tags: [linkedlist]
keywords:
description: 如何判断一个链表是否含环, 以及如何判断两个链表是否相交.
mathjax: true
toc: true
---



## 初步: 如何判断一个链表是否含有环

使得两个名为fast, slow的节点等于头节点, fast每次向前两步,  slow每次向前一步, 如果fast.next为null, 或者fast.next.next为null, 则说明没有环. 如果fast和slow相遇, 说明有环.

时间复杂度和环和长度以及链表长度有关, 为线性时间.


### 代码

```java
public boolean hasCycle(ListNode head) {
        if(head==null){
            return false;
        }
        ListNode fast=head;
        ListNode slow=head;
        //0: 无环 ; 1: 有环
        int circleFlag=0;
        while (true){
            if(fast.next==null||fast.next.next==null){
                return false;
            }
            fast=fast.next.next;
            slow=slow.next;
            if(fast==slow){
                slow=head;
                return true;
            }
        }
    }
```

## 进阶: 如果一个链表有环, 找出进入环的第一个节点


使得两个名为fast, slow的节点等于头节点, fast每次向前两步,  slow每次向前一步, 如果fast.next为null, 或者fast.next.next为null, 则说明没有环. 如果fast和slow相遇, 说明有环. 此时将fast节点移动号head节点(判断fast和slow是否相遇, 如果相遇,说明head节点即使入环的第一个节点). 之后fast的节点每次走一步, slow节点每次走一步. 下一次相遇的节点即是入环的第一个节点.

### 证明

某个链表总长度为n+m, 环的长度为m. <br>
其中令

$$n\%m=n-x\cdot m   ,(x=\left \lfloor \tfrac{n}{m} \right \rfloor)$$


1. slow节点到达入环的第一个节点时fast位置:<br>
入环的第$n\%m$节点, 即$n-x\cdot m$.
2. 第一次相遇时, 两个节点所处的位置:<br>
入环的第$m-(n-x\cdot m)$节点, 即$(x+1)\cdot m-n$.
3. 此时fast节点回到原点, 当fast节点再次走到入环的第一个节点时, slow节点的位置:<br>
入环的地$((x+1)\cdot m-n+n)\%m$. 即$((x+1)\cdot m)%m$. 即为入环的第一个节点. 因为从fast节点回到原点再到入环的第一个节点过程中, fast节点一直在环外, slow节点一直在环内. 所以此次相遇为第二次相遇. 即按照之前的走法, 第二次相遇的节点即为入环的第一个节点.

### 代码

```java
public ListNode detectCycle(ListNode head) {
    if(head==null){
        return null;
    }
    ListNode fast=head;
    ListNode slow=head;
    //0: 无环 ; 1: 有环

    int circleFlag=0;
    while (true){
        if(fast.next==null||fast.next.next==null){
            circleFlag=0;
            break;
        }
        fast=fast.next.next;
        slow=slow.next;
        if(fast==slow){
            fast=head;
            circleFlag=1;
            break;
        }
    }
    if(circleFlag==0){
        return null;
    }
    while(true){
        if(fast==slow){
            return fast;
        }
        slow=slow.next;
        fast=fast.next;

    }
}
```

## 进阶: 以及如何判断两个链表是否相交

### 一链表有环, 另一链表无环, 不相交

### 两个链表都没有环

计算两个链表的长度, 较长的链表先向前走`Math.abs(lenA-lenB)`步, 之后两个节点一起前进, 并相互比较, 若有相同的节点, 则说明两个链表在该节点出相交, 否则说明两个链表不相交.

#### 代码


```java
public int listLengthWithoutCycle(ListNode head){
    int res=0;
    while (head!=null){
        head=head.next;
        res+=1;
    }
    return res;
}
public ListNode noLoop(ListNode headA, ListNode headB){
    int lenA= listLengthWithoutCycle(headA);
    int lenB= listLengthWithoutCycle(headB);
    System.out.println(lenA);
    System.out.println(lenB);
    ListNode a=headA;
    ListNode b=headB;
    if(lenA<lenB){
        a=headB;
        b=headA;
    }
    int diff=Math.abs(lenA-lenB);
    for (int i = 0;  i < diff;  i++) {
        a=a.next;
    }
    while(true){
        if(a==null||b==null){
            return null;
        }
        if(a==b){
            return a;
        }
        a=a.next;
        b=b.next;
    }
}
```
### 两个链表都有环

找出两个链表入环的第一个节点, 其中一个节点不断的赋值为该节点的next, 若能够遇到另一个链表的入环的一个节点, 则说明两个链表环内相交, 否则说明两个链表为环外相交或者不相交.

如果两个链表是在环外相交的, 解法和都没有环的解法类似.




#### 代码

```java
public ListNode bothLoop(ListNode headA, ListNode headB,ListNode aIns,ListNode bIns){
    //环内相交
    ListNode aInsCopy=aIns;
    while (true){
        aIns=aIns.next;
        if(aIns==aInsCopy){
            break;
        }
        if(aIns==bIns){
            return aIns;
        }
    }

    //在while循环没有return, 说明并没有在环内相交
    int lenA=listLengthWithCycle(headA);
    int lenB=listLengthWithCycle(headB);
    ListNode a=headA;
    ListNode b=headB;
    if(lenA<lenB){
        a=headB;
        b=headA;
    }
    int diff=Math.abs(lenA-lenB);
    for (int i = 0;  i < diff;  i++) {
        a=a.next;
        if(a==aIns){
            return null;
        }
    }
    while(true){
        if(a==b){
            return a;
        }
        if(a==aIns){
            return null;
        }
        if(b==bIns){
            return null;
        }
        a=a.next;
        b=b.next;
    }
}


public int listLengthWithCycle(ListNode node){
    int res=0;
    ListNode ins=detectCycle(node);
    int flag=-1;
    while (true){
        if(flag==1){
            break;
        }
        if(node==ins){
            flag++;
        }
        res+=1;
    }
    return res;
}
```