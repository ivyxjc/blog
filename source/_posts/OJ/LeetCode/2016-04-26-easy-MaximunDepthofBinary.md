---
author: ivyxjc
date: 2016-04-26
title: Maximum Depth of Binary Tree
category: OJ
tags: [recursion]
keywords:
description:
---

## 寻找二叉树最长路径


###　递归的方法

1.节点为`null`时，返回深度为0
2.节点的左右子节点中有一个为`null`时，返回`1+maxDepth(root.left(or root.right))`
3.若节点的左右子节点都不为`null`时，返回两者中大的那一个。

```java
public int maxDepth(TreeNode root) {
        if(root==null){
            return 0;
        }else if(root.left!=null&& root.right!=null){
            return max(1+maxDepth(root.left),1+maxDepth(root.right));
        }else if(root.left==null){
            return 1+maxDepth(root.right);
        }else{
            return 1+maxDepth(root.left);
        }
    }

    public int max(int a,int b){
        return (a>b?a:b);
    }
```

