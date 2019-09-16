---
author: ivyxjc
date: 2016-02-18
title: Codeforces Education Round7
category: Algorithm
tags: [格式化输出,onlinejudge,io,线段树,区间查询]
keywords:
description: Codeforces Education Round7几道题目的解答
---

[题目网址](http://codeforces.com/contest/622/)

## A:Infinite Sequence


A题比较简单,主要就利用公式$ \sum_{k=1}^n k=\frac {(1+n)* n}{2} $来确定距离所要求的数字最近的一个1所在的位置。然后就可得到该数值了。<br>
代码如下：

### 代码

```java
public class Round7_A{
    public static void main(String[] args){
        Scanner in=new Scanner(System.in);
        long  a=in.nextLong();
        long a_one=count(a);

        if(a_one*a_one+a_one==2*a){
            System.out.println(a_one);
        }else{
            long tmp=(long)a-(a_one*a_one+a_one)/2;
            System.out.println(tmp);
        }
        return;
    }

    public static long count(long  num) {
        long num_two = 2 * num;
        long i = (long) Math.sqrt(num_two) - 1;
        //System.out.println(i);
        if (i <= 0) {
            i = 0;
        }
        for (; i <= num; i++) {
            //System.out.println(i);
            if (i * i + i == num_two) {
                return i;
            }
            if (i * i + i > num_two) {
                return i - 1;
            }
        }
        return 0;
    }
}
```

## B. The Time
B题也比较简单，主要就注意几个边界条件即可以及格式化输出即可。

```java
//num只有个位时，会自动在十位补0
DecimalFormat df=new DecimalFormat("00");
df.format(num);
```


## C. Not Equal on a Segment

### 题目
题意为查询一个数组的某一区间是否存在指定的$$$x$$$不相同的数，若不存在输出-1，否则输出任意一个不相同的树的秩。<br>

### 解法介绍
1.暴力解法就是遍历整个区间，但时间肯定不够。<br>

2.线段树法：线段树广泛用于区间查询，但是该题仅需输出任意一个不相同的树的位置即可，所以可以不用该法。(其实我还没有特别熟线段树)

3.解法：设输入数据的数组为```array[]```<br>
创建一个数组```diff[]=new int[arrayLength]```使其任意秩```diff[i]```值为满足$$$ a_j \ne a_i (j>i) $$$的第一个j的值。这也是官方答案所给的解法。代码如下：<br>

### 边界条件
该代码需要考虑到的边界条件有：<br>

```
1.数组内容全部相同
2.数组内容出现连续相同的项
3.数组内容全部不相同
4.diff数组的最后一项是否有值(统一将diff数组的最后一项设为数组长度)

```

### 代码

```java
int tmpI=0;
for(int j=tmpI+1;j<arrayLength;){
    if(array[j]==array[tmpI]) {
        j++;
        continue;
    }
    for(;tmpI<j;tmpI++){
        diff[tmpI]=j;
    }
}
//若数组中最后出现连续相同的项，上述代码不能将这些项赋值为arrayLength
//所以需要下面的代码
for(int i=tmpI;i<arrayLength;i++){
    diff[i]=arrayLength;
}

diff[arrayLength-1]=arrayLength;
```

查询时使用的代码:<br>

```java
try{
    for(int i=0;i<queryTimes;i++){
        int left=getInt()-1;
        int right=getInt()-1;
        int x=getInt();
        if(array[left]!=x){
            writer.write((left+1)+"\n");
            continue;
        }
        if(diff[left]>right){
            writer.write((-1)+"\n");
        }else{
            writer.write((diff[left]+1)+"\n");
        }
    }
    writer.flush();
}catch (IOException e){
    e.printStackTrace();
}
```

### IO问题
这一题以及下面的e题还有一个问题就是如果使用```Scanner```和```System.out.println()```，无论如何都不会通过测试，因为io就基本上快超时了。

我使用了[ java_acm快速输入和输出 ][1]中提供的一个输入输出方法，代码如下
#### 解决代码

```java
public class Main {  
    private static Reader reader = null;  
    private static Writer writer = null;  

    public static void main(String[] args) {  
        reader = new InputStreamReader(System.in);  
        writer = new OutputStreamWriter(System.out);  
        try {  
                m = getInt();  
                writer.write("something");  
                writer.flush();  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  

    /**
     * 获取键盘输入的整数
     *  
     * @return 输入的整数
     */  
    public static int getInt() {  
        int read;  
        int res = 0;  
        boolean isNegative = false;// 是不是负数  
        try {  
            while ((read = reader.read()) != -1) {  
                if ((char) read == '-') {  
                    res = 0;  
                    isNegative = true;  
                    break;  
                } else if (isNumber((char) read)) {  
                    res = read - '0';  
                    break;  
                }  
            }  
            while ((read = reader.read()) != -1) {  
                char ch = (char) read;  
                if (isNumber(ch)) {  
                    res = res * 10 + (read - '0');  
                } else {  
                    break;  
                }  
            }  
        } catch (IOException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        if (isNegative == true) {  
            res = -1 * res;  
        }  
        return res;  
    }  

    /**
     * 判断字符是否空白
     *  
     * @param ch
     *            字符
     * @return 判断结果
     */  
    public static boolean isBlank(char ch) {  
        if (ch == '\r' || ch == '\n' || ch == ' ') {  
            return true;  
        }  
        return false;  
    }  

    /**
     * 判断字符是不是数字
     *  
     * @param ch
     *            字符
     * @return 判断结果
     */  
    public static boolean isNumber(char ch) {  
        if (ch <= '9' && ch >= '0') {  
            return true;  
        }  
        return false;  
    }  
}  

```

## E. Ants in Leaves

### 题目
You are given a tree with n vertices and a root in the vertex 1. There is an ant in each leaf of the tree. In one second some ants can simultaneously go to the parent vertex from the vertex they were in. No two ants can be in the same vertex simultaneously except for the root of the tree.<br>

Find the minimal time required for all ants to be in the root of the tree.<br>

题意：n只蚂蚁从树的叶节点向根节点爬，每秒只能爬到它的父节点处，但是除了根节点，其它所有节点都只能同时存在一只蚂蚁。<br>

### 解法

This problem was suggested by Aleksa Plavsic allllekssssa.

Let z be the array of the depths of all leaves in the subtree of the vertex v. Let's sort z.

Statement 1: it's profitable to lift the leaves in order of their appearing in z.

Statement 2: denote ax — the time of appearing the x-th leaf in the vertex v, let's consider the leaves $$$z_i$$$ and $$$z_{i+1}$$$ then  $$$a_{z_i+1} \geq  a_{z_i}+1$$$.

Statement 3: $$$a_{z_i+1}=max(d_{z_i}+1,a_{z_i}+1)$$$, where $$$d_x$$$ is the depth of the x-th leaf in the subtree of the vertex v. The last statement gives us the solution for the problem: we should simply iterate over z from left to right and recalculate the array a by formula from the third statement. All statements can be easily proved and it's recommended to do by yourself to understand better the idea of the solution.

#### 代码实现

##### 建图

```java
int V=getInt();
    Graph G=new Graph(V,V-1);
    for(int i=0;i<V-1;i++){
        G.addEdge(getInt(),getInt());
    }
    G.dfs();
    G.solve();
```

##### dfs()详细代码

```dfs()```的目的是将根节点的每个直接子节点的所有叶子节点的深度存储在一个```ArrayList```中组成一个数组depthArray中。

```java
public void dfs(int v){
//从根节点出发
    if(v==1){
        int rootSonsNum=adj[v].getConnectedvertexs().size();
        depthArray=new ArrayList[rootSonsNum];
        for(int i=0;i<rootSonsNum;i++){
            depthArray[i]=new ArrayList<>();
        }
    }
    marked[v]=true;
    adj[v].setDepth(depth);
    if(adj[v].getDegree()==1)
        depthArray[count].add(adj[v].getDepth());
    depth++;
    for(Integer i:adj[v].getConnectedvertexs()){
        if(!marked[i]){
            dfs(i);
            depth--;
        }
        if(v==1){
            count++;
        }
    }
}

public void dfs(){
    dfs(1);
}
```

##### solve()代码

```solve()```代码的作用就是先将```depthArray```中的每一```个ArrayList```按从小到大排序，再对每一个ArrayList中的数据进行$$$max(d_{z_i}+1,a_{z_i}+1)$$$比较，最后```depthArray```数组中每一个```ArrayList```的resMax比较取最大的值。

```java
public int solve(){
    for(ArrayList<Integer> i:depthArray) {
        Collections.sort(i);
//            i.sort(new Comparator<Integer>() {
//                @Override
//                public int compare(Integer o1, Integer o2) {
//                    if (o1 > o2)
//                        return 1;
//                    else if (o1 == o2)
//                        return 0;
//                    else
//                        return -1;
//                }
//            });

/*
注释掉的代码在有些时候会出现
java.lang.IllegalArgumentException: Comparison method violates its general contract!。可见16057179提交的错误提示
*/

    int res=0;
    for(ArrayList<Integer> i:depthArray){
        //System.out.println(i);
        int tmpres=i.get(0);
        int tmpIndex=1;
        while(tmpIndex<i.size()){
            tmpres=max(tmpres+1,i.get(tmpIndex));
            tmpIndex++;
        }
        res=max(tmpres,res);
    }
    System.out.println(res);
    return res;
}
```



















  [1]: http://blog.csdn.net/a601025382s/article/details/46999711
