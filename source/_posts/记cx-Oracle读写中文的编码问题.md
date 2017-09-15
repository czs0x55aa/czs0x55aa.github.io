---
title: 记cx_Oracle读写中文的编码问题
date: 2017-08-24 10:26:34
tags: "零碎记录"
categories:
---
由于要读取Oracle的表结构，在前端页面生成脚本中用到了`cx_Oracle`,之前在linux上写好的代码，用了几个月都很正常。最近在mac下却出现了编码问题，读取的中文都变为了问号`?????`.  
一阵谷歌之后找打的答案都是设置环境变量`NLS_LANG`，然而我原先代码里就已经配置了环境变量的，极度蒙圈...  
最后在[cx_Oracle文档](https://cx-oracle.readthedocs.io/en/latest/index.html)中找到了原因.
在Version 6.0 的 Release Notes中有这样一条：
> Delay initialization of the ODPI-C library until the first standalone connection or session pool is created so that manipulation of the environment variable NLS_LANG can be performed after the module has been imported; this also has the added benefit of reducing the number of errors that can take place when the module is imported.  

也就是说在这之前的版本中设置NLS_LANG和import是有先后顺序的差别的。  
赶紧print下`cx_Oracle.__version__`, 发现是6.0b2, 确实是这个问题，因为习惯性把import写在头部的。
```python
os.environ['NLS_LANG'] = 'SIMPLIFIED CHINESE_CHINA.UTF8'
import cx_Oracle
```
调整先后顺序后，中文编码就正常了。  
而之前在linux上运行都正常，原因可能是linux上装的版本反而是新的，由于是使用conda自动安装的，安装时并没有去关注版本号。
