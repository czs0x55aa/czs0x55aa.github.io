---
title: Python源码剖析笔记——对象初探
date: 2017-02-27 16:39:13
tags: ["Python", "学习笔记"]
categories:
---
# 笔记说明
该笔记是对《Python源码剖析》学习过程的记录，这本书有点老了，源码是2.5版本的。由于阅读的时候比较仓促，并没有与目前2.7的源码做比较，里面的一些细节可能过时了。
该笔记以整体概念的记录和梳理为主，忽略了书中的一些实现细节的介绍。

# 基本概念梳理
## PyObject
Python中所有的东西都是对象，所有的对象都拥有一些相同的内容，这些内容定义在PyObject中
```cc
typedef struct _object{
	PyObject_HEAD
} PyObject;
```
<!-- more -->
PyObject_HEAD是一个宏定义，实际发布的Python中，PyObject的定义是长成下面这样的。
```cc
typedef struct _object{
	int ob_refcnt;
	struct _typeobject *ob_type;
} PyObject;
```
ob_refcnt：引用计数器
ob_type:类型信息,\_typeobject的结构相对复杂，后续会进一步描述。

## 定长对象和变长对象
实际上，上面提到的PyObject是用来描述定长对象的，还有些对象是变长的，例如字符串，list等  
两者的区别在于，定长对象的不同对象占用的内存大小是一样的（整数对象不论大小，占用相同的内存），而变长对象的不同对象占用的内存可能是不一样的（字符串有长有短）。
**Python中使用PyObject表示定长对象，PyVarObject表示变长对象，他们的宏分别是PyObject_HEAD和PyObject_VAR_HEAD**
变长对象PyVarObject的定义如下
```cc
#define PyObject_VAR_HEAD
	PyObject_HEAD
	int ob_size;

typedef struct {
	PyObject_VAR_HEAD
} PyVarObject;
```
与PyObject相比，只是多了一个ob_size，用来指明变长对象中一共容纳了多少个元素。  
可以看出，不论是定长对象还是变长对象，都拥有相同的对象头部。
所以我们只需要用一个PyObject*指针就可以引用任意的一个对象。

## \_typeobject
```cc
typedef struct _typeobject {
	PyObject_VAR_HEAD
	char *tp_name;
	int tp_basicsize, tp_itemsize;

	destructor tp_dealloc;
	printfun tp_print;
	...

	hashfunc tp_hash;
	ternaryfunc tp_call;
	...
} PyTypeObject;
```
\_typeobject中包含了许多信息，主要可以分为四类：
- 类型名，tp_name，主要是Python内部以及调试的时候使用
- 创建该类型对象时分配内存空间大小的信息，即tp_basicsize和tp_itemsize
- 与该类型对象相关联的操作信息
- 类型信息

有趣的是，在PyTypeObjec定义的头部，也有一个PyObject_VAR_HEAD，**这意味着Python中的类型实际也是一个变长对象**。
- 任何东西都是对象，每一个对象都对应了一种类型
- 类型也是一种对象
那么类型对象PyTypeObject中的ob_type又要指向谁呢？
嗯，这时候需要**PyType_Type**出场了
```cc
#define PyObject_HEAD_INIT(type)  \
	_PyObject_EXTRA_INIT	\
	1, type,

PyTypeObject PyType_Type = {
	PyObject_HEAD_INIT(&PyType_Type)
	0,	// ob_size
	"type",	// tp_name
	sizeof(PyHeapTypeObject),	// tp_basicsize
	sizeof(PyMemberDef),	// tp_itemsize
	......
};
```
实际上PyType_Type只是一个特殊的PyTypeObject，它的ob_type是指向自身的。
在Python中，通过 int.__class__ 能看到 <type 'type'>  
这个 <type 'type'> 是所有class的class， 在Python中被称为metaclass。  
在Python中实例化一个整数对象时，这个整数对象的ob_type会指向一个PyInt_Type对象（这也是个PyTypeObject），而这个PyInt_Type中的ob_type则会指向PyType_type。  
PyObject中的ob_refecnt用来计数对象的引用次数，当引用次数为0时则调用类型对象中的tp_dealloc函数，但是PyType_Type是个例外，它永远不会被销毁。

## 可变对象和不可变对象
上面已经提过了定长对象和变长对象，根据对象维护数据的可变性还可以划分出可变对象和不可变对象。  
比如int对象，它是定长的，且不可变的  
又比如str和tuple，它是变长的，但是不可变  
再比如list，它是变长的，又是可变的
（定长对象中，是否有可变的呢？）

## Python对象的分类
Python中的对象大致可以分为5类，这种分类方式不是绝对的
- Fundamental对象：类型对象
- Numeric对象：数值对象
- Sequence对象：容纳其他对象的序列集合对象
- Mapping对象：类似于C++中map的关联对象
- Internal对象：Python虚拟机在运行时内部使用的对象
书中的后续集几章节分别对这几种对象类型进行了阐述。
