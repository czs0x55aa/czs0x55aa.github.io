---
title: logistic与softmax的梯度计算
date: 2017-09-14 08:56:57
tags: ["机器学习", "机器学习", "学习笔记"]
categories:
---
在多分类和神经网络中softmax的使用非常普遍，由于实践中基本都是直接使用库和框架中封装好的接口，一直没有去思考过softmax的梯度计算。
顺便复习下logistic。

## 1. 逻辑回归的梯度计算
### 1.1 逻辑回归的cost
$$ J(\theta) = \frac{1}{m}\sum\_{i=1}^{m}\left[y^{i}\,log\, h(x^{i})\ +\ (1\ -\ y^{i})\,log\,(1\ -\ h(x^{i}))\right] $$
由于逻辑回归中使用了非线性的sigmoid函数，为了保持目标函数为凸函数，该目标函数与线性回归有所不同。
<!-- more -->
### 1.2 梯度计算
$$
\begin{align}
\frac{\partial}{\partial\theta\_{j}}J(\theta) &= -\frac{1}{m}\sum\_{i=1}^{m}\left[y^{i}\,\frac{1}{\frac{1}{1+e^{-(x\theta+b)}}}\,-\,(1-y^{i})\,\frac{1}{1-\frac{1}{1+e^{-(x\theta+b)}}}\right]\,\frac{-e^{-(x\theta+b)}}{(1+e^{-(x\theta+b)})^{2}}\,(-x) \\\\
&=-\frac{1}{m}\sum\_{i=1}^{m}\left[y\,(1+e^{-(x\theta+b)})\,-\,(1\,-\,y)\,\frac{1+e^{-(x\theta+b)}}{e^{-(x\theta+b)}}\right]\,\frac{1}{1+e^{-(x\theta+b)}}\,\frac{e^{-(x\theta+b)}}{1+e^{-(x\theta+b)}}\,x \\\\
&= -\frac{1}{m}\sum\_{i=1}^{m}\left[y\,\frac{e^{-(x\theta+b)}}{1+e^{-(x\theta+b)}}\,-\,(1-y)\,\frac{1}{1+e^{-(x\theta+b)}}\right]\,x \\\\
&=-\frac{1}{m}\sum\_{i=1}^{m}\left[y\,(1-h(x))\,-\,(1-y)\,h(x)\right]\,x \\\\
&= \frac{1}{m}\sum\_{i=1}^{m}(h(x)\,-\,y)\,x \\\\
\end{align}
$$
偷懒省略了一些上标。  
感觉比较神奇的是，虽然目标函数和线性回归的不同，但是梯度却是一致的。

## 2.softmax
### 2.1 softmax函数
$$ softmax(x\_i) = \frac{exp(x\_i)}{\sum\_{j=1}^{n}\,exp(x\_j)} $$
这是softmax的基本形式，由于exp运算可能出现上溢或下溢的情况，实际应用中一般会采用log softmax。
### 2.2 目标函数
在神经网络中，softmax一般使用交叉熵作为目标函数。
$$ Loss = -\sum\_{i}^{n}\,y\_{i}\,ln\,a\_{i} $$
由于大多数时候都是单类别分类，所有可能的y中，只有目标项为1，其他项则都为0，所以实际上可以忽略累加运算。
### 2.3 梯度计算
令：
$$ z\_j = \sum\_{i=1}^{n}\,x\_i\,w\_{ij} \\\\
a\_i = \frac{exp(x\_i)}{\sum\_{j=1}^{n}\,exp(x\_j)} $$
则有：
$$
\begin{align}
\frac{\partial\,Loss}{\partial\,w\_{j}} &= \frac{\partial\,Loss}{\partial\,a\_i}\,\frac{\partial\,a\_i}{\partial\,z\_i} \\\\
&=-\frac{1}{a\_i}\,\frac{\partial\,a\_i}{\partial\,z\_i}
\end{align}
$$
其中的$\frac{\partial\,a\_i}{\partial\,z\_i}$需要分类讨论。
这是因为softmax函数的分子部分对应了一个节点，这个节点是不是目标项，对其求导是有影响的。  
一种是分子的节点j与目标项对应（这样的描述好像不准确...不知道怎么表述）：
$$
\begin{align}
\frac{\partial\,a\_j}{\partial\,z\_{i}} &= \frac{e^{z\_j}\,\sum\_{k=1}^{n}\,e^{z\_k}\,-\,e^{z\_j}\,e^{z\_j}}{(\sum\_{k=1}^{n}\,e^{z\_k})^2} \\\\
&= \frac{e^{z\_j}}{\sum\_{k=1}^{n}\,e^{z\_k}}\,-\,\frac{e^{z\_j}}{\sum\_{k=1}^{n}\,e^{z\_k}}\,\frac{e^{z\_j}}{\sum\_{k=1}^{n}\,e^{z\_k}} \\\\
&= a\_j\,(1\,-\,a\_j)
\end{align}
$$
另一种是分子节点j不是与目标项对应的，那么求导时分子的前一项为0：
$$
\begin{align}
\frac{\partial\,a\_j}{\partial\,z\_{i}} &= \frac{-\,e^{z\_i}\,e^{z\_j}}{(\sum\_{k=1}^{n}\,e^{z\_k})^2} \\\\
&= -\,\frac{e^{z\_i}}{\sum\_{k=1}^{n}\,e^{z\_k}}\,\frac{e^{z\_j}}{\sum\_{k=1}^{n}\,e^{z\_k}} \\\\
&= -\,a\_i\,a\_j
\end{align}
$$
将两种情况分别带到之前梯度的式子中，分别为$(a\_j-1)$和$a\_i$。
