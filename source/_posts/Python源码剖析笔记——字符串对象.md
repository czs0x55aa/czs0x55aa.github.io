---
title: Python源码剖析笔记——字符串对象
date: 2017-05-12 20:27:07
tags: ["Python", "学习笔记"]
categories:
---
## PyStringObject
PyStringObject是字符串对象的实现。它是一个边长对象，因为存储的字符串可长可短，但它又是一个不可变对象，即创建后其内容就不能再被改变了。
<!-- more -->
其定义如下：
```cc
typedef struct {
	PyObject_VAR_HEAD
	long ob_shash;
	int ob_sstate;
	char ob_sval[1];
} PyStringObject;
```
从头部的PyObject_VAR_HEAD可以看出它是变长对象，包含了一个ob_size来保存长度的大小。  
ob_shash用于缓存该对象的hash值，这个hash值在dict中有着巨大作用。  
ob_sstate用来标记该对象是否经过intern机制的处理？
ob_sval表面上是一个长度只有1的数组，实际上是作为一个字符指针来用的，指向字符串的实际内容，通过ob_size和结构体自身的大小，可以确定该字符串对象实际占用的空间大小。  

# 字符串的创建
Python中提供了PyString_FromStringAndSize 和 PyString_FromString 函数

# 字符串的intern机制
intern机制的目的是，避免重复创建内容相同的字符串对象。  
试想，一个相同内容的字符被多次创建，而字符串对象又是不可变的，重复的创建相同的字符串对象显然是一种浪费。
所以当创建了一个字符串对象'python'，如果再一次创建'python'这个字符串对象的时候，那么就会直接返回已有的一个引用。  
同时，这一机制的另一个好处是，当比较两个字符串对象时，直接比较两者的地址即可。
- intern机制是如何工作的
Python中有一个函数PyString_InternInPlace函数用于对字符串对象执行intern操作。简单的描述就是内部存在一个dict，每次对字符串对象执行intern的时候会先去dict里面查找，如果已经存在则返回引用。
- intern机制是什么被调用的？
在PyString_FromStringAndSize 和 PyString_FromString 函数中，对于size<=1的字符串执行了intern操作。
那么如果size>2呢，在书上以及网上的很多文章中似乎都忽略掉了这个细节。  
在一篇[外网](http://guilload.com/python-string-interning/)中提到了触发intern的情况，可以是在编译时。在执行intern前，如果字符串包含了非ascii/下划线/数字的会被过滤掉，从而不执行intern。

# 字符缓冲池
与小整数对象的缓冲池类似，Python中还存在一个字符缓冲池。
这个缓冲池是以静态变量的形式存在的，在Python初始化完成之后，该缓冲池中的指针都是空的。
而在创建一个字符对象的时候，则会加入其中，同样受用于intern机制。

# 字符串的连接
在编码过程中，连接字符串是个非常常见的操作。  
如果通过'+'操作符的话会调用string_concat函数，这个函数只连接两个字符串。如果一行连接操作中执行了N次'+'操作,则需要调用N次的string_concat。
官方推荐的方法是利用join操作对list或tuple中的多个字符串进行连接，该方法调用string_join会一次性连接多个字符串对象。
