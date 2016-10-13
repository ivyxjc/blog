---
author: ivyxjc
date: 2016-10-04
title: matplot basic
category: Python
tags: [python,matplotlib]
keywords:
description: matplotlib是一个常见的绘图库. 本文介绍如何使用matplotlib 绘制2d散点图, 3d散点图.
---



## 2d散点图

```python
#设置颜色
colors = ['#0000FF','#3CB371']
#各点大小的list
sizeLab=np.add(sizeLab,20)
# print(sizeLab)

#设置横轴上下限
plt.xlim(0,25)

#设置横轴标号
# plt.xticks(np.linspace(0,15,5,endpoint=True))

#设置记号的标签
plt.xticks([2*np.pi, 4*np.pi, 0, np.pi/2, np.pi],
       [r'$2\pi$', r'$4\pi$', r'$0$', r'$\pi/2$', r'$\pi$'])

#设置横纵坐标label
plt.xlabel("x 轴label")
plt.ylabel("y 轴label")


plt.scatter(datingDataMat[:,1],datingDataMat[:,2],s=sizeLab, c=colors)
plt.show()
```

## 3d散点图

```python
fig=plt.figure()
ax=fig.add_subplot(111,projection='3d')

#设置坐标
ax.set_xlabel("飞行里程数")
ax.set_ylabel("游戏时间百分比")
ax.set_zlabel("冰激凌消耗量")

type1=ax.scatter(dataDidnlike[:,0], dataDidnlike[:,1], dataDidnlike[:,2], c='#3c345b')
type2=ax.scatter(dataSmallDoses[:,0], dataSmallDoses[:,1],dataSmallDoses[:,2],c="#4576f7")
type3=ax.scatter(dataLargeDoses[:,0], dataLargeDoses[:,1], dataLargeDoses[:,2],c="#43ff1a")

#设置图例
ax.legend((type1, type2, type3), (u'不喜欢', u'魅力一般', u'极具魅力'), loc=2)

plt.show()
```
