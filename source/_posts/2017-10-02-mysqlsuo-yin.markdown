---
layout: post
title: "Mysql索引"
date: 2017-10-02 18:05
comments: true
categories: [Mysql]
---

#### [1.索引的优缺点原因](#1)
#####   [1.1为什么添加索引，查询速度变快](#1.1)
#####   [1.2为什么添加上索引，速度会变慢](#1.2)

#### [2.索引的类别](#2)
#####   [2.1主键索引](#2.1)
#####   [2.2唯一索引](#2.2)
#####   [2.3普通索引](#2.3)
#####   [2.4多列索引](#2.4)
#####   [2.5全文索引](#2.5)

#### [3.加索引情况](#3)
#####   [3.1 关键词](#3.1)
#####   [3.2 小技巧](#3.2)

#### [4.索引生成文件](#4)
#####   [4.1 索引文件](#4.1)

<br/>

之前一直对索引没有很清晰的认识，每次一创建表时，就给外键加索引，只知道索引提高mysql搜索效率；
前阵子查阅了很多资料，有了比较清晰的认识，现在给出如下总结：
<br/>

<h2 id='1'>索引的优缺点原因</h2>

------------------

<h3 id='1.1'>为什么添加索引，查询速度变快</h3>

------------------

计算机查找某一条记录，如果不加索引，会在整个表中一条一条比较，将匹配的记录加入 **结果集**，很多人说这样会很慢，加了索引就快了。

为什么？

计算机先在索引列表中找到记录的 **位置**，既 **rowid**，然后直接去表中的对应位置取出记录，我就不明白了，查找索引列表难道不需要一条一条的匹配？计算机又不会出现说，看索引列表比直接看表中的记录要快，先在索引列表中找到记录对应的 rowid 也是要遍历的？难道不是同样的吗？
<br/>

Mysql 索引主要两种：
<br/>
**1.BTree**
取中间数作为根，比根小的数再取中间数放左边，大数放右边作为右边树叶；
时间复杂度就是log2(n)；
<br/>

**2.Hash**
key-value 保存;
时间复杂度是(n);
缺点：不支持范围查询，只能指定查询；

<br/>
<br/>

<h3 id='1.2'>为什么添加上索引，速度会变慢</h3>

------------------

1. DML 耗时：根据上面的回答，添加索引会 **构建 BTree **，建立 **索引都需要维护**，你  **创建、插入或删除** 记录，都要重新构建部分 BTree；
【所以唯一性太差的字段不适合创建索引；字段频繁变化的字段也不适合创建索引】

2. 消耗磁盘空间；

<br/>
<br/>

<h2 id='2'>索引的类别</h2>

------------------

<h3 id='2.1'>主键索引</h3>

------------------

添加PRIMARY KEY（主键索引）

``` sql
ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )
```

<br/>
<h3 id='2.2'>唯一索引</h3>

------------------

添加UNIQUE(唯一索引)

``` sql
ALTER TABLE `table_name` ADD UNIQUE ( `column` )
```

**唯一索引中 null 不代表重复，可以有多个 null；**

<br/>
<h3 id='2.3'>普通索引</h3>

------------------

添加INDEX(普通索引)

``` sql
ALTER TABLE `table_name` ADD INDEX index_name ( `column`)
```

<br/>
<h3 id='2.4'>多列索引</h3>

------------------

也叫联合索引；
添加多列索引

``` sql
ALTER TABLE `table_name` ADD INDEX index_name ( `column1`,`column2`, `column3` )
```
一般用在 where 几个字段，并且使用频繁的语句；
当使用联合索引后，第一个 column1 将是一级目录，所以不用单独给它做索引，因为他已经是单独的索引了；

第二个 column2 是二级目录索引，所以如果你也可以增加多一个单独索引，因为不可能直接使用二级目录的；


<br/>
<h3 id='2.5'>全文索引</h3>

------------------

添加FULLTEXT(全文索引)


全文索引只能在 **MYISAM** 引擎中使用：

``` sql
ALTER TABLE `table_name` ADD FULLTEXT (`column`)
```

#### 如何使用：
在MySQL中创建全文索引之后，现在就该了解如何使用了。众所周知，在数据库中进行模糊查询是使用LIKE关键字进行查询，例如：
``` sql
SELECT * FROM article WHERE content LIKE '%查询字符串%'
```

那么，我们使用全文索引也是这样用的吗？当然不是，我们必须使用特有的语法才能使用全文索引进行查询。例如，我们想要在article表的title和content列中全文检索指定的查询字符串，可以如下编写SQL语句：

``` sql
SELECT * FROM article WHERE MATCH(title, content) AGAINST('查询字符串')
```

PS: 全文索引并不是所有文本都加索引，而是通过“停止词”；
对一些常用词，没什么特殊含义的词（a, the, at, which, want...）不加索引，这些词叫做停止词；

<br/>


<h3 id='2.6'>查询、修改、删除索引</h3>
------------------------

1.查询
desc 表名;【该方法的缺点，不能显示索引名】
``` sql
show index from 表名;
show keys from 表名;
```
<br/>

2.删除
``` sql
alter table 表名 drop index 索引名;
如果删除主键索引：
alter table 表名 drop primary key;
```
<br/>

3.修改

先删除，再重新创建；

<br/>
<br/>


<h2 id='3'>加索引的情况</h3>

<h3 id='3.1'> 关键词 </h3>
一些关键词代表可能会经常用到索引，具体还是要把 SQL 语句从程序中 debug 出来，
然后在 mysql 中查看用 explain 是否有用到该索引：

1.分组字段：
``` sql
group by
```
很多人觉得不会用到，其实会用到的；
按照 explain 可以看出单独对驱动表使用 group by，mysql 计划是会使用索引的；
```
explain
select *
from juniu_return_goods_sku
group by juniu_return_goods_sku.goods_id
```
这里返回使用了索引，并且不会 用到 using temporary;
所以加了索引，用 explain 验证一下即可；


2.排序字段：
``` sql
order by
```

3.统计字段，【注意运算不会用索引，例如:SUM, YEAR】：
``` sql
select MAX()
```

4.主键外键字段：
``` sql
JOIN()
```

还有要注意无效索引：

1.运算字段：
``` sql
WHERE DATE_FORMATE(created_at, '%Y/%m%/d') = '2018-05-12'
```

2.Like语句：
``` sql
like "%aaa%" // 不会用到索引
like "aaa%" // 会用到索引
```

3.含有NULL值的列【注意：给默认值】


4.NOT IN语句【注意：可以改成 NOT EXISTS】：
``` sql
WHERE id NOT IN (1,2,3);
```

5.OR 语句：
``` sql
WHERE id = 1 OR id = 2;
```


它先对日期值使用 DATE_FORMATE 方法，所以索引无效；

<br/>
<br/>


<h3 id='3.2'> 小技巧 </h3>

1. 在使用 **group by** 分组查询时，默认分组后，还会排序，可能会降低速度；
例如：在一张表中对一个索引字段进行 group by ，explain后发现他不会使用索引，但是 extra 中有 **Using filesort**；
解决：group by 后面加上 **order by null**;

2. 有些情况下，可以使用连接代替子查询。
因为使用 join，Mysql 不需要在 **内存** 中创建 **临时表(不使用索引)**，
除非指定了联接条件时，满足查询条件的记录行数少的表为[驱动表]，未指定联接条件时，行数少的表为[驱动表]，多表联合查询时

``` sql
select * from dept, emp where dept.no = emp.no; // 简单处理方式
select * from dept left join emp on dept.no = emp.no; // 左连接
```


<br/>
<br/>


<h2 id='4'>4. 索引生成文件</h2>

<h3 id='4.1'>索引文件</h3>

可以通过 my.ini 看到索引的目录文件，一般每一张表有三个文件：
- .frm 记录表的结构，【innodb 数据库的数据放在外层目录的 ibdata 文件；】

- .myd 表的数据

- .myi 记录表的索引

<br/>
<br/>
