---
title: Python源码剖析笔记——Dict对象
date: 2017-10-22 21:32:17
tags: ["Python", "学习笔记"]
---
在Python中提供了关联式容器dict，它类似于map；由于对搜索的效率要求非常高，python中的dict使用哈希表来实现的，在理想情况下搜索的复杂度为O(1)。
<!-- more -->
## 哈希表
通过哈希函数（hash function）将键值映射为一个整数，将这个整数作为索引来访问一段内存空间。
### 哈希冲突
不同的元素，经过散列可能得到相同的结果，这就会发生冲突。尤其当元素逐渐增多时，发生冲突的概率也会变大。
`装载率`表示哈希表中已使用空间和总空间的比值，例如哈希表长度为10，目前已装入6个元素，那么它的装载率就是6/10。
在解决哈希冲突时，常见的使用开链法和开放定址法，在Python中采用的是后者。
并且在删除元素时采用“伪删除”操作。

## PyDictObject
### entry
关联容器中的每个元素是一个entry，其定义如下：
```cc
typedef struct {
    Py_ssize_t me_hash;    /* cached hash code of my_key */
    PyObject *me_key;
    PyObject *me_value;
} PyDictEntry;
```
其中me_hash存储的时me_key的散列值，避免在查询时进行重复的计算。
entry存在三个状态：
- Unused状态：当entry未被使用时，其me_key和me_value都是NULL，每个entry初始后会出于这个状态
- Active状态：当entry中存储了一个(key, value)对时，me_key和me_value都不能为NULL，此时为active状态
- Dummy状态：当存储的(key, value)被删除后，entry从active状态转变为dummy状态，这就是伪删除操作

### 关联容器
关联容器PyDictObject实际上就是一大堆的entry的集合，其结构定义如下：
```cc
#define PyDict_MINSIZE 8
typedef struct _dictobject PyDictObject;
struct _dictobject{
    PyObject_HEAD
    Py_ssize_t ma_fill;    // 元素个数 Active + Dummy
    Py_ssize_t ma_used;    // 元素个数 Active
    Py_ssize_t ma_mask;
    PyDictEntry *ma_table;
    PyDictEntry *(*ma_lookup)(PyDictObject *mp, PyObject *key, long hash);
    PyDictEntry ma_smalltable[PyDict_MINSIZE];
};
```
- - -
未完成
