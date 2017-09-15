---
title: python的MySQLdb中使用utf8mb4编码遇到的问题
date: 2016-08-08 14:35:38
tags: "零碎记录"
---
今天在导入微博文本到mysql的时候遇到了emoji字符，导致数据插入出错的情况。
查了一下是因为mysql中的utf8只支持3个字节，emoji和少部分不常用汉字则是4字节的。据说是在mysql5.5开始的版本中支持了utf8mb4编码类型来应对这种4字节的uft8编码。
<!-- more -->
看了下自己用的版本5.7,按照网上常见的教程配置完mysql的默认编码（win下的配置文件是my.ini和linux上不同），不幸的是在python中一直还是报错。
```
_mysql_exceptions.OperationalError: (1366, "Incorrect string value: '\\xF0\\x9F\\x9A\\xA3\\xE6\\x9C...' for column 'content' at row 1")
```
一开始还以为是数据库编码配置的问题，后来手动插入一个emoji字符发现正常，所以应该是代码部分引起的。
猜测是代码连接数据库的部分任然使用utf8而没有使用utf8mb4。由于MySQLdb.connect的参数charset只能使用utf8，不支持utf8mb4。
在google上搜了很久，似乎是python的mysqldb中特有的问题（不确定和win是否有关）,但是没有找到解决办法。
最后试了各种方法，找到了一种暂时的解决方法：
在python代码中连接数据库后，手动执行SET NAMES:
```python
cur.execute("SET NAMES utf8mb4;")
```
这种方式大概相当于手动设置了连接使用的编码方式。
