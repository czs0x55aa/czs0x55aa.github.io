---
title: Python在MySQLdb中使用存储过程
date: 2016-08-14 14:40:03
tags: "零碎记录"
---
起因是为了创建dif词典集，文本中不重复词有18万多（讲道理18W也不是很多呀(╯‵□′)╯︵┻━┻…），但是执行过程非常慢，因为每个词在文档中的出现次数都是通过数据库中查询得到的，意味着要进行18W次查询，于是想试试用存储过程能不能提高速度。
结果在调用存储过程的时候遇到了问题…（最近经常遇到些奇奇怪怪的问题…）
<!-- more -->
以下为自己编写的存储过程
```python
CREATE PROCEDURE `get_df2`(IN `word` VARCHAR(30), OUT `total_num` INT)
SELECT COUNT(*) INTO total_num FROM
(SELECT `uid` FROM `personas_status` WHERE `content` like concat('%', word, '%') GROUP BY `uid`) as t
```
奇怪的是在phpmyadmin中需要把begin和end去掉才能顺利执行，网上有说和定界符有关（懒得纠结这个了..）
如果存储过程中没有OUT参数的话，调用callproc后直接fetch出来即可：
```python
cur.callproc('存储过程名称', (参数...,))
res = cur.fetchall()
```
这里个cur对象是通过conn.cursor()得到的。
由于自己的存储过程中有OUT变量，按照上面的方法是取不到数据的，需要手动select @变量名来获取返回值。
但是MySQLdb的callproc却有点奇葩，它自定义了调用存储过程时的变量命名（其实人家备注了在他们的主页文档里，代码注释里也有提到...QAQ）
通过查看函数callproc的源码，有如下内容：
```python
db = self._get_db()
for index, arg in enumerate(args):
  q = "SET @_%s_%d=%s" % (procname, index,
  db.literal(arg))
  if isinstance(q, unicode):
    q = q.encode(db.unicode_literal.charset)
    self._query(q)
    self.nextset()

q = "CALL %s(%s)" % (procname,
    ','.join(['@_%s_%d' % (procname, i)
      for i in range(len(args))]))
```
这段代码中定义了变量名，格式是：@\_存储过程名\_编号
按照这个格式手动select，顺利取到了返回值...(此时的内心是崩溃的，这样的小问题坑了我一个多小时...)
但是最终的执行速度并不理想，和原本的查询速度几乎一样。
找了很久没有找到，不过推测的原因大概是本身存储结构的硬伤，存储过程加快的部分是SQL编译的时间，但是我这里的SQL语句本身很简单，绝大部分的时间是发生在查询的过程中的，所以没起到加速的效果。
最后附上MySQLdb主页文档中的说明：
>callproc(procname, args)
Calls stored procedure procname with the sequence of arguments in args. Returns the original arguments. Stored procedure support only works with MySQL-5.0 and newer.

>Compatibility note: PEP-249 specifies that if there are OUT or INOUT parameters, the modified values are to be returned. This is not consistently possible with MySQL. Stored procedure arguments must be passed as server variables, and can only be returned with a SELECT statement. Since a stored procedure may return zero or more result sets, it is impossible for MySQLdb to determine if there are result sets to fetch before the modified parmeters are accessible.

>The parameters are stored in the server as @_*procname*_*n*, where n is the position of the parameter. I.e., if you cursor.callproc('foo', (a, b, c)), the parameters will be accessible by a SELECT statement as @_foo_0, @_foo_1, and @_foo_2.
