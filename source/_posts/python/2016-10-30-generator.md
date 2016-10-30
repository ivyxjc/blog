---
author: ivyxjc
date: 2016-10-30
title: python 生成器函数
category: Python
tags: [python,generators]
keywords:
description: python中生成器函数
---




## 生成器函数

生成器函数是一个迭代器, 不过只能遍历其一次, 这是因为生成器并不将数据存储在内存之中, 生成完数据后并不保存下来.

```python

list=(i for i in range(5))

for i in list:
    print(i)

print("---")
for i in list:
    print(i)

out:
0
1
2
3
4
---

```

```python

def createGenerator():
    for i in range(5):
        yield i*i

gen=createGenerator()
for i in gen:
    print(i)

print("---")
for i in gen:
    print(i)
out:

0
1
4
9
16
---
```

另外, 生成器函数并不是再调用的时候就运行的, 而是在遍历的时候才运行, 在上述代码中, 意味着生成器函数是在执行`for i in gen:`时才运行的.

解释器是如何知道其是生成器函数的呢? 当解释器发现`yield`时, 就知道该函数是个生成器函数, 它会设置一个标识来标记这一情况, 调用type也可以发现这一情况.

```python
#生成器标志位在第5位
generator_bit=1<<5
b=bool(createGenerator.__code__.co_flags & generator_bit)
print(b)
print(type(createGenerator))

out:
True
<class 'generator'>
```






















参考文章:

[A Web Crawler With asyncio Coroutines](http://aosabook.org/en/500L/a-web-crawler-with-asyncio-coroutines.html)

[What does the yield keyword do](http://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do)
