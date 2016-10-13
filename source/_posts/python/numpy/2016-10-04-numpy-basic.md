---
author: ivyxjc
date: 2016-10-04
title: numpy user guide
category: Python
tags: [python,numpy]
keywords:
description: numpy是常见的科学计算库. 本文介绍numpy库的一些常见操作,例如numpy.tile(A, reprs), numpy.sum(), axis,  dtye等相关知识.
---

## numpy.tile(A,reprs)
[numpy.tile](http://docs.scipy.org/doc/numpy/reference/generated/numpy.tile.html)

` numpy.tile(A, reps)`

This method will return a `max( d, A.ndim) ` dimension array( `d= len( reprs) `).

if`d> A.ndim`, it will return a d-dimension array by prepending new axes;<br>
etc.

```python
>>>a = np.array([[1,2,3],[1,2,3]])
>>>np.tile(a,(2,3,2))

[[[1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]]

 [[1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]
  [1 2 3 1 2 3]]]
```

if `d< A.ndim`, it will return an A.ndim-dimension array by prepending 1's to it .
For example, for an shape of(1,2,2,3), a reps of (2,3) will be treated as (1,1,2,3)

```python
>>>a = np.array([[[[1,2,3],[1,2,3]]]])
>>>np.tile(a,(2,2))

[[[[1 2 3 1 2 3]
   [1 2 3 1 2 3]
   [1 2 3 1 2 3]
   [1 2 3 1 2 3]]]]

>>>a = np.array([[[[1,2,3],[1,2,3]]]])
>>>np.tile(a,(2,1,3))

[[[[1 2 3 1 2 3 1 2 3]
   [1 2 3 1 2 3 1 2 3]]

  [[1 2 3 1 2 3 1 2 3]
   [1 2 3 1 2 3 1 2 3]]]]
```

## numpy.sum()
[numpy.sum()](http://docs.scipy.org/doc/numpy/reference/generated/numpy.sum.html)

`numpy.sum(a, axis=None, dtype=None, out=None, keepdims=False)`

## axis
This is a litte complex. `abs(axis)` must be less than `a.ndim`.

![](/assets/img/posts/numpy_sum_1.jpg)
![](/assets/img/posts/numpy_sum_2,jpg)

### dtype

```python
>>>b=np.array([0.3,0.4,0.9,1.5,1.9])
>>>print(b.sum(dtype=np.int32))

2
```


### keepdims

If this parameter is assigned to True, the it will return an array as dimension with the size that input array is


```python
>>>b=np.array([[[0.3,0.4,0.9,1.5,1.9]]])
>>>print(b.sum())

5.0


>>>b=np.array([[[0.3,0.4,0.9,1.5,1.9]]])
>>>print(b.sum(keepdims=True))

[[[ 5.]]]
```
