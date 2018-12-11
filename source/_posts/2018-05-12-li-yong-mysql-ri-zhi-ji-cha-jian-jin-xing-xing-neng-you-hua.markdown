---
layout: post
title: "利用 Mysql 日志及插件进行性能优化"
date: 2018-05-12 17:37
comments: true
categories: [Mysql]
---

#### [1. 数据库优化的目的](#1)

#### [2. SQL及索引优化](#2)

##### [一、慢查询日志设置](#2.1)

##### [二、慢查询日志的分析工具](#2.2)
###### [2.1 如何通过慢查询日志发现有问题的SQL](#2.2.1)
###### [2.2 如何分析SQL查询](#2.2.2)

##### [三、建立索引](#2.2)
###### [3.1 如何选择合适的列建立索引：](#2.3.1)
###### [3.2 索引的维护和优化—重复及冗余索引](#2.3.2)
###### [3.3 删除不用索引](#2.3.3)

##### [四、系统配置优化](#2.4)

#### [3. 硬件要求](#3)



<h1 id='1'>数据库优化的目的：</h1>

---------------------------
#### 避免出现页面访问错误
* 由于数据库连接timeout产生页面5xx错误
* 由于慢查询造成页面无法加载
* 由于阻塞（加锁）造成数据无法提交
<br/>

#### 增加数据库稳定性
* 很多数据库查询都是由低效的查询引起的
<br/>

#### 优化用户体验
* 流畅页面访问速度
* 良好的网站功能体验
<br/>

![mysql性能优化金字塔](/images/posts/2018-05-12/mysql性能优化金字塔.gif "mysql性能优化金字塔")

<br/>
<br/>


<h1 id='2'>SQL及索引优化</h1>
---------------------------

### 如何发现有问题的SQL？

<br/>

<h2 id='2.1'>一、慢查询日志设置</h2>

----------------------------

使用 Mysql 慢查日志对有效率问题的 SQL 进行监控

``` sql
# 首先进入 mysql 的cli
show variables like "slow_query_log"
```

![mysql query log](/images/posts/2018-05-12/mysql_query_log.gif "mysql query log")

``` sql
set global slow_query_log_file='/home/mysql/sql_log/mysql_slow.log' # 设置慢查询日志存储路径
set global log_queries_now_using_indexes=on;                        # 将没有索引的慢查询加入到慢查询日志中
set global long_query_time=1                                        # 将大于1s的设置到慢查询日志中
Set global slow_query_log=on;                                       # 开启慢查询日志记录
```

### 慢查询日志所包含的内容

> 执行 SQL 的主机信息
>
> User@Host:root[root] @ localhost []
>
>
> SQL的执行信息
>
> Query_time: 0.000024 Lock_time: 0.0000 Rows_sent: 0  Rows_examined: 0
>
>
> SQL执行时间
>
> SET timestamp=1402389328
>
>
> SQL的内容
>
> select CONCAT(’storage engine:’,  @@storage_engine) as INFO;

<br/>
<br/>


<h2 id='2.2'>二、慢查询日志的分析工具：</h2>

----------------------------

#### mysqldumpslow

**mysqldumpslow** [OPTS…] [LOGS…]    官方分析工具
返回count为查询次数，time为查询时间 ，rows为查询返回行数

![mysql dump slow](/images/posts/2018-05-12/mysqldumpslow.gif)

<br/>

#### pt-query-digest

输出到文件：

``` sql
pt-query-digest slow-log > slow_log.report
```


输出到数据库表：

``` sql
pt-query-digest slow.log -review h=127.0.0.1, D=test, p=root, P=3306, u=root, t=query_review --create-reviewtable --review-history t=hostname_slow
```

![pt-query-digest](/images/posts/2018-05-12/pt-query-digest.gif "pt-query-digest")
![pt-query-digest](/images/posts/2018-05-12/pt-query-digest2.gif "pt-query-digest")

<br/>
<br/>

<h2 id='2.2.1'>如何通过慢查询日志发现有问题的SQL</h2>

--------------------------

#### 1.查询次数多且每次查询占用时间长的SQL

通常为 pt-query-digest 分析的前几个查询

#### 2.IO大的SQL

注意 pt-query-digest 分析中的 Rows examine 项【扫描行数越多，IO越大】

#### 3.未命中索引的SQL

注意 pt-query-digest 分析中Rows examines 和 Rows Send 的对比 【如果扫描行数 远大于 发送行数，要么没有索引，要么索引无效，导致类似于全表扫描】

![pt-query-digest](/images/posts/2018-05-12/pt-query-digest3.gif "pt-query-digest")

<br/>
<br/>


<h2 id='2.2.2'>如何分析SQL查询：</h2>

-----------------------

使用 explain 查询 SQL 的执行计划

![explain](/images/posts/2018-05-12/explain.gif "explain")

table：                显示这一行数据属于哪张表

type：                 重要的列，显示连接使用了何种类型。从最好到最差的链接类型为const、eq_reg、ref、range、index 和 ALL

possible_keys：        显示可能应用这张表中的索引。如果为空，没有可能的索引。

Key：                  实际使用的索引。如果为NULL，则没有使用索引；

key_len：              使用索引的长度。在不损失精确性的情况下，长度越短越好；

ref：                  显示索引的哪一列被使用了，如果可能的话，是一个常数

rows：                 MYSQL认为必须检查的用来返回请求数据的行数


<br/>

### Max优化：

对max字段使用索引，优化了explain的type；


### 子查询的优化：

通常情况下，把子查询优化成join，要注意是否一对多的关系，注意重复数据


### Limit优化：

limit常用于分页操作，时常会伴随order by 从句使用，因此会使用 filesorts 这样会造成大量IO问题；

#### 优化步骤1：
使用主键，或者索引列来进行Order by 操作；

#### 优化步骤2：
记录上次返回主键，在下次查询使用主键过滤；【如果主键不是顺序增长的，有空缺，则返回的数量每页可能变少】，避免了数据量大时扫描过多的记录


<br/>
<br/>

<h2 id="2.3">三、建立索引</h2>

------------------------


<h3 id="2.3.1"> 3.1 如何选择合适的列建立索引：</h3>

------------------------

<br/>
1.在where从句，group by从句，order by从句，on从句出现的列；


2.索引字段越小越好；


3.离散度越大的列放在联合索引前面；

<br/>

``` sql
select * from payment where staff_id = 2 and customer_id = 584;
```
<br/>

是 index(staff_id, customer_id) 好？还是 index(customer_id, staff_id) 好？
<br/>
由于 customer_id 的离散度更大，所以应该使用 index(customer_id, staff_id)
那么如何知道离散程度：
可以通过一下查询：


``` sql
select count(distinct customer_id), count(distinct staff_id) from payment;
```


![联合索引离散度](/images/posts/2018-05-12/联合索引离散度.gif)

<br/>

得到staff_id比较集中，离散程度比较好，所以选择 customer_id 来作为联合索引前面的字段；

<br/>


<h3 id="2.3.2"> 索引的维护和优化—重复及冗余索引</h3>
重复的索引是指相同的列以相同的顺序建立的同类型的索引，如下表的 unique 索引中包含了 primary key 索引

``` sql
create table test (
  id int not null primary key,
  name varchar(10) not null,
  title varchar(50) not null,
  Unique(name, id)
)engine=innodb;
```

使用pt-duplicate-key-checker工具检查重复及冗余索引

``` sql
pt-duplicate-key-checker -uroot -p -h127.0.0.1
```

<br/>
<br/>


<h3 id="2.3.3"> 删除不用索引：</h3>
 mysql没有记录索引使用情况，但在PerconMysql，MariaDB中有INDEX_STATISTICS表来查看哪些索引未使用，但在mysql中目前只能通过慢查询日志来配合 pt-index-usage 工具来进行索引使用情况分析；

``` sql
pt-index-usage -uroot -p mysql-slow.log
```





<h2 id='2.4'>四、系统配置优化：</h2>

-------------------------
Mysql 配置文件-常用参数说明

``` sql
Innodb_buffer_pool_size
```

配置innodb的缓冲池，如果数据库中只有innodb表，则推荐配置量为总内存的75%；

<br/>

``` sql
innodb_buffer_pool_instances
```

可以控制缓冲池的个数，默认一个缓冲池

<br/>

``` sql
innodb_log_buffer_size
```

Innodb log 缓冲的大小，由于日志最长每秒就会刷新所以一般不用太大；

<br/>


``` sql
innodb_flush_log_at_trx_commit
```

关键参数，对 innodb 影响很大。默认值为1，可以取0，1，2三个值，一般建议设为2，如果安全性要求高则使用默认值1；

<br/>

``` sql
innodb_read_io_threads
Innodb_write_io_threads
```
读写io进程数，默认为4；


<br/>

``` sql
innodb_file_per_table
```

关键参数，控制 Innodb 每个表使用独立的表空间，默认为OFF，也就是所有表会建立在共享表空间中，IO 会形成瓶颈；
建议设置为 ON，这样每个使用独立的共享表空间；

<br/>

``` sql
Innodb_stats_on_metadata
```
决定 mysql 什么情况下会刷新 innodb 表的统计信息；

上述信息可以使用 Per Configuration Wizard 配置


<br/>
<br/>


<h1 id="3">硬件要求：</h1>

---------------------------
如何选择CPU

1. Mysql一些工作只能用单核CPU：   SQL语句执行，Replicate；

2. Mysql对核数并不是越多越好：mysql5.5 不要超过32核；
