---
layout: post
title:	"matplotlib学习笔记"
date:	2019-06-06 13:07:11 +0800
categories:	python
---

> 最近在学习机器学习的过程中接触到了matplotlib，感觉非常有趣，但是这个绘图库的学习成本还是比较高的，知识点很多，很容易遗忘，这里记录一下自己的理解，方便复习

##  matplotlib是什么

matplotlib是python的一个2D绘图库，提供了类似MATLAB的API

##  matplotlib的用法

### figure

figure相当于画布，可以设置背景颜色，尺寸

 ```python
import matplotlib.pyplot as plt
figure = plt.figure(facesize=(100,100),facecolor='blue') # 设置一个画布，尺寸是100*100,颜色是蓝色
plt.show()
 ```

### subplot()

subplot相当于在画布上画一个子图

```python
import numpy as np
import matplotlib.pyplot as plt

x = np.arange(0, 100)
# 作图1
plt.subplot(221) # 子图占据画布的2行2列，这是第一个子图
plt.plot(x, x)
# 作图2
plt.subplot(222) # 子图占据画布的2行2列，这是第二个画布
plt.plot(x, -x)

plt.show()
```

###  subplots()

subplots相当于画一个子图，同时自带画布

```python
figure, ax1 = plt.subplots(1,1,sharex=True) # 用subplots创建一个画布figure和一个坐标系ax1

ax2 = ax1.twinx() # 创建一个坐标系ax2,和ax1公用x坐标

x = np.linspace(0, 4 * np.pi, 100)
y = np.sin(x)

x2 = np.linspace(0, 4 * np.pi, 100)
y2 = np.cos(x2)

plt.plot(x,y)
plt.plot(x2,y2)
figure.set_facecolor('blue')  # 将画布的颜色设置为蓝色
plt.show()
```

###  plot()

plot()相当于绘图命令

```python
t = np.arange(0,5,0.1)
plt.plot(t,t,t,t**2,t,t**3) # 一次绘多条曲线
plt.show()
```

###  show()

show()相当于显示绘制的图像