---
title: Python源码剖析笔记——List对象
date: 2017-05-18 17:30:00
tags: ["Python", "学习笔记"]
categories:
---
## PyListObject对象
PyListObject是一个变长对象，也是可变对象，可以有效的支持对元素的插入、添加、删除等操作。
<!-- more -->
PyListObject的定义
```cc
typedef struct {
	PyObject_VAR_HEAD
	PyObject **ob_item;
	int allocated;
} PyListObject;
```
其中ob_item指向了元素列表的首地址，allocated维护了当前列表可容纳的元素的总数。  
- 在PyObject_VAR_HEAD中有一个ob_size，与allocated有什么不同呢？
在创建list对象时，由于不确定需要使用的大小，为了效率考虑，会申请一大块内存。  
allocated中记录的是申请的内存大小，而ob_size中记录的则是实际使用的大小。

# 列表操作
## 创建对象
要创建一个列表对象，Python中只提供了唯一的一种途径————PyList_New。这个函数接受一个size参数，该参数指明了要申请的内存大小。  
在创建list对象时，会检查一个变量num_free_lists，判断缓冲池中是否有可用的空间，默认缓冲池中最多维护80个PyListObject对象。  

## 设置元素
例如在执行list[3] = 100时，Python内部会调用PyList_SetItem来完成这个动作。
依次执行
- 类型检查
- 索引的有效性检查
- 将指针指向新的元素
- 旧的元素如果不是NULL，将其计数器减1

## 插入元素
Python内部通过调用PyList_Insert来完成元素的插入。  
和设置元素一个显著的不同时，插入的时候元素的数量发生了变化，这有可能会改变ob_item指向的内存。
当插入后的newsize < allocated && newsize > allocated/2的时候，简单的调整ob_size值；其他情况则会调用realloc重新分配空间。
在确定插入位置后，会从该位置开始的元素全部后移，用来插入新元素。
另外，append操作的话，则只用在ob_size+1个上位置插入即可。

## 删除元素
当进行remove时会调用listremove函数，该函数会对整个列表进行遍历，逐个比较元素，当找到匹配项的时候进行删除，并将后面的元素前移。  

# PyListObject对象缓冲池
同样的，list对象也具有一个缓冲池。在Python运行环境初始的时候，list缓冲池中是空的。
当有list对象被销毁时，如果缓冲池未满则加入到缓冲池中。
需要注意的是，缓冲池中只是PyListObject对象，list对象维护的数据元素都已经被释放掉了。
