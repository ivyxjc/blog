---
author: ivyxjc
date: 2017-03-01
title: Fibonacci数列及相关问题
category: Algorithm
tags: [Fibonacci,dynamic_programming]
keywords:
description: 与Fibonacci数列相关的问题的解法
mathjax: true
---

## Fibonacci数列及相关的问题的描述

1. 给定正整数n, 求解Fibonacci数列第n项的值;
2. 给定正整数N, 代表台阶, 一次可以跨2个或者1个台阶, 有多少走法
3. 假设成熟的母牛只会生1头小母牛, 并且永远不会死, 第一年农场有1只成熟的母牛,从第二年开始, 母牛开始生小母牛. 每只小母牛3年之后成熟. 给定正整数N, 求出N年后牛的数量.


## 基础解法

### 递归的解法

递归的解法是最基础, 也是最好理解, 但是时间复杂度很高, 为 $O(2^n)$ ,空间复杂度为$O(2^n)$

```java
public static int fibRecursion(int n) {
    if (n == 0) {
        return 0;
    }
    if (n == 1) {
        return 1;
    }
    return fibRecursion(n - 1) + fibRecursion(n - 2);
}
```

### 迭代的解法

迭代的解法即将已经解决的位置处的数存储, 之后直接调用, 不用再计算

时间复杂度为O(n), 空间复杂度为O(n).

```java
public static int fibIteration(int n) {
    if (n == 1) {
        return 1;
    }
    if (n == 0) {
        return 0;
    }
    int[] arr = new int[n + 1];
    arr[0] = 0;
    arr[1] = 1;
    for (int i = 2; i < n + 1; i++) {
        arr[i] = arr[i - 1] + arr[i - 2];
    }
    return arr[n];
}
```

仅记录前两个值, 空间复杂度为O(1)

```java
public static BigInteger fibIteration(int n) {
        if (n == 1) {
            return new BigInteger("1");
        }
        if (n == 0) {
            return new BigInteger("1");
        }


        BigInteger res = new BigInteger("0");
        BigInteger pre_1 = BigInteger.valueOf(1);
        BigInteger pre_2 = BigInteger.valueOf(0);

        for (int i = 2; i < n + 1; i++) {
            res = pre_1.add(pre_2);
            pre_2 = pre_1;
            pre_1 = res;
        }
        return res;
    }
```

## 进阶解法


### 推导过程

$$
F(n)=F(n-1)+F(n-2) \\\\
\Rightarrow
\begin{vmatrix}
F(n) & F(n-1) \\\\
\end{vmatrix}
=
\begin{vmatrix}
F(n) & F(n-1) 
\end{vmatrix}
\times
\begin{vmatrix}
a & b \\\\
c & d 
\end{vmatrix}\\\\
$$

解得:<br>

$$
a=1 \\
b=1 \\
c=1 \\
d=1
$$

即: <br>

$$
\begin{vmatrix}
 F(n) & F(n-1)
\end{vmatrix}
=
\begin{vmatrix}
F(n-1)& F(n-2)
\end{vmatrix}
\times
\begin{vmatrix}
1 & 1 \\\\ 
1 & 0
\end{vmatrix}
\\\\
\Rightarrow
\begin{vmatrix}
 F(n) & F(n-1)
\end{vmatrix}
=
\begin{vmatrix}
 1 & 1
\end{vmatrix}
\times
{\begin{vmatrix}
1 & 1 \\\\ 
1 & 0
\end{vmatrix}}^{n-2}
$$


### 代码

```java

public static BigInteger fibMatrix(int n) {
    if (BigInteger.ZERO.compareTo(BigInteger.valueOf(n)) == 0) {
        return new BigInteger("0");
    }
    if (BigInteger.ONE.compareTo(BigInteger.valueOf(n)) == 0) {
        return new BigInteger("1");
    }
    return matrixPower(new int[][]{{1, 1}, {1, 0}}, n)[0][0];
}

//计算矩阵matrix的p次方
public static BigInteger[][] matrixPower(int[][] matrix, int p) throws IllegalArgumentException {
    if (matrix[0].length != matrix.length) {
        throw new IllegalArgumentException("矩阵输入错误, 无法进行乘法.");
    }
    BigInteger[][] res = new BigInteger[matrix.length][matrix.length];
    for (int i = 0; i < res.length; i++) {
        for (int j = 0; j < res[0].length; j++) {
            res[i][j] = BigInteger.valueOf(0);
        }
    }
    for (int i = 0; i < res.length; i++) {
        res[i][i] = new BigInteger("1");
    }

    BigInteger[][] tmp = new BigInteger[matrix.length][matrix.length];
    for (int i = 0; i < tmp.length; i++) {
        for (int j = 0; j < tmp[0].length; j++) {
            tmp[i][j] = BigInteger.valueOf(matrix[i][j]);
        }
    }

    while (p > 0) {
        int flag = p & 1;
        if (flag == 1) {
            res = multiMatrix(res, tmp);
        }
        tmp = multiMatrix(tmp, tmp);
        p = p >> 1;
    }

    return multiMatrix(new BigInteger[][]{{new BigInteger("1"), new BigInteger("1")}}, res);
}



//矩阵乘法
public static BigInteger[][] multiMatrix(BigInteger[][] m1, BigInteger[][] m2) throws IllegalArgumentException {
    if (m1[0].length != m2.length) {
        throw new IllegalArgumentException("矩阵输入错误, 无法进行乘法.");
    }
    BigInteger[][] res = new BigInteger[m1.length][m2[0].length];
    for (int i = 0; i < m1.length; i++) {
        for (int j = 0; j < m2[0].length; j++) {
            res[i][j] = new BigInteger("0");
        }
    }

    for (int i = 0; i < m1.length; i++) {
        for (int j = 0; j < m2[0].length; j++) {
            for (int k = 0; k < m2.length; k++) {
                res[i][j] = res[i][j].add(m1[i][k].multiply(m2[k][j]));
            }
        }
    }
    return res;
}
```


## 运行时间比较:

当n=45时, 递归的方法就需要6794ms才能完成.

当n=1000000时, 迭代的方法耗时16s左右, 使用矩阵的方法只需耗时1.266s即完成
