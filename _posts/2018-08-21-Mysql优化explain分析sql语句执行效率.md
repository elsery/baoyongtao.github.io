---
layout: post
title: "Mysql优化explain分析sql语句执行效率"
date: 2017-11-14 
description: "Explain命令在解决数据库性能上是第一推荐使用命令，大部分的性能问题可以通过此命令来简单的解决，Explain可以用来查看SQL语句的执行效 果，可以帮助选择更好的索引和优化查询语句，写出更好的优化语句。"
categories: Nginx
--- 


**Explain命令在解决数据库性能上是第一推荐使用命令，大部分的性能问题可以通过此命令来简单的解决，Explain可以用来查看SQL语句的执行效 果，可以帮助选择更好的索引和优化查询语句，写出更好的优化语句。**
``
Explain语法：explain select … from … [where ...]
``
例如：``explain select * from news;``

输出：
``
+----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
+----+-------------+-------+-------+-------------------+---------+---------+-------+------+-------+
``
下面对各个属性进行了解：

##### 1、id：这是SELECT的查询序列号

##### 2、select_type：select_type就是select的类型，可以有以下几种：

- SIMPLE：简单SELECT(不使用UNION或子查询等)

- PRIMARY：最外面的SELECT

- UNION：UNION中的第二个或后面的SELECT语句

- DEPENDENT UNION：UNION中的第二个或后面的SELECT语句，取决于外面的查询

- UNION RESULT：UNION的结果。

- SUBQUERY：子查询中的第一个SELECT

- DEPENDENT SUBQUERY：子查询中的第一个SELECT，取决于外面的查询

- DERIVED：导出表的SELECT(FROM子句的子查询)


##### 3、table：显示这一行的数据是关于哪张表的

##### 4、type：这列最重要，显示了连接使用了哪种类别,有无使用索引，是使用Explain命令分析性能瓶颈的关键项之一。

结果值从好到坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL

一般来说，得保证查询至少达到range级别，最好能达到ref，否则就可能会出现性能问题。

##### 5、possible_keys：列指出MySQL能使用哪个索引在该表中找到行

##### 6、key：显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL

##### 7、key_len：显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好

##### 8、ref：显示使用哪个列或常数与key一起从表中选择行。

##### 9、rows：显示MySQL认为它执行查询时必须检查的行数。

##### 10、Extra：包含MySQL解决查询的详细信息，也是关键参考项之一。

**Distinct**
一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

**Not exists**
MYSQL 优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，

就不再搜索了

Range checked for each

**Record（index map:#）**
没有找到理想的索引，因此对于从前面表中来的每一 个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

**Using filesort**
看 到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来 排序全部行

**Using index**
列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表 的全部的请求列都是同一个索引的部分的时候

**Using temporary**
看到这个的时候，查询需要优化了。这 里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

**Using where**
使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index， 这就会发生，或者是查询有问题

其他一些Tip：

当type 显示为 “index” 时，并且Extra显示为“Using Index”， 表明使用了覆盖索引。
