---
title: Python源码剖析笔记——整数对象
date: 2017-02-27 17:22:23
tags: ["Python", "学习笔记"]
categories:
---
上一节[Python源码剖析笔记——对象初探](http://czs0x55aa.github.io/2017/02/27/Python%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90%E7%AC%94%E8%AE%B0%E2%80%94%E2%80%94%E5%AF%B9%E8%B1%A1%E5%88%9D%E6%8E%A2/)
## 结构
```cc
typedef struct {
	PyObject_HEAD
	long ob_ival;
} PyIntObject
```
PyObject_HEAD里面包含了int类型的引用计数器ob_refcnt和_typeobject类型的用于存储类信息的结构体指针ob_type。  
对于整形对象而言，这里的ob_type指向的是一个PyInt_Type对象。  
ob_ival则用于存储实际的整数值。
<!-- more -->
## PyInt_Type
PyInt_Type中包含了关于PyIntObject对象的诸多信息，以及PyIntObject对象所支持的操作。

## PyIntObject对象的创建
Python提供了3中途径：
```cc
PyObject *PyInt_FromLong(long ival)
PyObject *PyInt_FromString(char *s, char **pend, int base)
PyObject *PyInt_FromUnicode(Py_UNICODE *s, int length, int base)
```
实际上，PyInt_FromString和PyInt_FromUnicode都是将其转化为长整形后再调用PyInt_FromLong的。
另外，在Python中的整数对象分为大整数和小整数，当创建对象对象时根据数值的大小区别处理。

# 小整数对象与大整数对象
## 小整数对象
- 为什么小整数和大整数对象要分开对待呢？
这是因为在程序中小整数会被频繁的使用，为了提高性能，将小整数放在一个统一的块上进行管理。  
可以试想一下在for循环的时候，常常会用到小整数。
- 小整数和大整数的分界线在哪？
在源码中有如下的定义:
```cc
#ifndef NSMALLPOSINTS
	#define NSMALLPOSINTS		257
#endif
#ifndef NSMALLNEGINTS
	#define NSMALLNEGINTS		-5
#endif
#if NSMALLNEGINTS + NSMALLPOSTINTS > 0
	static PyIntObject *small_ints[NSMALLNEGINTS + NSMALLPOSTINTS]
#endif
```
可以看到小整数的默认定义范围是[-5, 257)。
小整数的这块空间是在Python初始化的时候创建的，并且不会被销毁，直到Python程序结束。

## 大整数对象
除了小整数外的其他整数，Python运行环境提供了一块内存空间，这些空间供大整数轮流使用。  
其底层使用了一个PyIntBlock的结构：
```cc
struct _intblock {
	struct _intblock *next;
	PyIntObject objects[N_INTOBJECTS];
};
```
next指针用构建PyIntBlock的单链表。  
objects数组则用于存储PyIntObject。  
- 这些整数对象是如何进行管理的呢？
在Python中，通过一个block_list指针来维护多个PyIntBlock。  
另外还有一个free_list指针来维护空闲的空间，即串联了objects数组，构成一个单链表。
- 对象的添加
当创建一个新对象的时候，从free_list中申请走一个单位的空间，free_list指向下一个空闲单位。
- 对象的删除
当删除一个对象的时候，这个单位的空间再次由free_list接管。

# 存在的问题
大整数的对象池由block_list维护，当一个Block用完后会向系统申请新的Block，但是申请后的Block即使其objects数组中都是空的（申请又销毁过），也不会对其释放Block。  
也就是说Pythony一旦运行后，PyIntBlock对象的数量只会增加不会减少，这有点类似于内存泄漏。
