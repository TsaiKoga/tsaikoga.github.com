---
layout: post
title: "Laravel 的数据库事务源码分析"
date: 2019-05-09 17:54
comments: true
categories: [PHP, Mysql]
---

在批发档口记账 app 中，由于用户数量越来越多，有几个老客户了反馈单据丢失的问题；

**打个比方：**

我们开单会去创建开单表 ```order_create```，开单表会生成详情 ```order_create_goods_sku```，然后生成欠货表 ```order_owe_goods_sku```；

然后接着是发货请求，会生成发货主表 ```order_delivery``` 和 发货详情 ```order_delivery_goods_sku```；发货会生成库存变动记录；

<br>

这时候已经有了开单和开单详情，证明已经开单请求成功；

但是库存变动记录有，发货单却找不到；这时候我去查找了日志，发现也有发货请求；
然而通过库存变动记录找不到相应的发货单了，这是怎么回事，难道是被人撤销了吗？

<br>

全局查找了整个项目，没有强删除的接口，难道是代码有问题？


那么我开始定位代码，因为事务包裹着整个"发货的创建流程" 和 "库存变动记录的生成"；
所以要么就全部回滚，那么不太可能是回滚问题？

<br>

# 跟踪数据库操作：

------------------


那是不是有人从数据库删除了？

我写了个触发器，只要有人强删记录就会记录到一张跟踪表：

1.首先创建跟踪表：
``` sql
CREATE TABLE `juniu_track_delivery` (
  `track_delivery_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '订单ID',
  `order_delivery_id` int(11) NOT NULL DEFAULT 0 COMMENT  '发货单ID',
  `order_id` int(11) NOT NULL DEFAULT 0 COMMENT '订单ID',
  `deliver_timestamp` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '发货时间',
  `deleted_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '被删除时间',
  `deleted_by` varchar(50) DEFAULT '' COMMENT '删除的用户ip',
  PRIMARY KEY (`track_delivery_id`)
  ) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8;
```
<br>

2.建立触发器 ```track_delete_delivery``` 跟踪：
``` sql
DELIMITER //
CREATE TRIGGER track_delete_delivery
AFTER DELETE
   ON juniu_order_delivery FOR EACH ROW
BEGIN
   DECLARE vUser varchar(50);
   -- Find username of person performing the DELETE into table
   SELECT USER() INTO vUser;
   -- Insert record into audit table
   INSERT INTO juniu_track_delivery
   ( order_delivery_id,
     order_id,
     deliver_timestamp,
     deleted_at,
     deleted_by)
   VALUES
   ( OLD.order_delivery_id,
     OLD.order_id,
     OLD.deliver_timestamp,
     SYSDATE(),
     vUser );
END; //
DELIMITER ;
```
观察一段时间，仍有数据丢失，但是并没有跟踪到，初步判断不是人为删除；

<br>

# Mysql 事务特性：

------------------

Mysql 事务具有四大特性：A（原子性）C（一致性）I（隔离性）D（持久性）；

事务有五个级别：

> 1. TRANSACTION_NONE  不使用事务。
>
> 2. TRANSACTION_READ_UNCOMMITTED  未提交读，允许脏读。
>
> 3. TRANSACTION_READ_COMMITTED  提交读，防止脏读，最常用的隔离级别,并且是大多数数据库的默认隔离级别
>
> 4. TRANSACTION_REPEATABLE_READ  重复读，可以防止脏读和不可重复读，
>
> 5. TRANSACTION_SERIALIZABLE  序列化，可以防止脏读，不可重复读取和幻读，（事务串行化）会降低数据库的效率


1、**脏读：** 事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据

2、**不可重复读：** 事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。

3、**幻读：** 事务 A 修改了表中所有数据，但是事务 B 插入了一条数据，当事务 A 查询数据发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。

**小结：** 不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表

<br>

最后我们来查看一下 Mysql 数据库事务配置，看是不是事务配置错误：

``` sql
select @@tx_isolation;

select @@session.tx_isolation;

select @@global.tx_isolation;
```

显示结果都是 ```READ-COMMITTED```

<br>

# 查看事务源码：

------------------

重新回到回滚代码，既然都被包裹起来，但是又回滚不成功，我开始查阅 Laravel API，因为我怕会是框架有问题，

找到相应的 laravel 版本 5.1: https://laravel.com/api/5.1/

寻找 ```transaction```，显示有

> Illuminate\Database\ConnectionInterface::transaction
>
> Illuminate\Database\Connection::transaction

接口定义的是规则，所以我看数据库的连接类 ```Illuminate\Database\Connection::transaction```


这里有几个我们要看的方法：
``` php
mixed transaction(Closure $callback)

void beginTransaction()

void commit()

void rollBack()
```
点击右边行号，可以跳转到 5.1 源代码文件：

先查看 ```transaction``` 方法，此处参数是一个闭包，我们项目中使用的不是这种写法：
```php
    public function transaction(Closure $callback)
    {
        $this->beginTransaction();
        // We'll simply execute the given callback within a try / catch block
        // and if we catch any exception we can rollback the transaction
        // so that none of the changes are persisted to the database.
        try {
            $result = $callback($this);
            $this->commit();
        }
        // If we catch an exception, we will roll back so nothing gets messed
        // up in the database. Then we'll re-throw the exception so it can
        // be handled how the developer sees fit for their applications.
        catch (Exception $e) {
            $this->rollBack();
            throw $e;
        } catch (Throwable $e) {
            $this->rollBack();
            throw $e;
        }
        return $result;
    }
```
整个事务用 ```try catch``` 包裹住，如果失败，直接抛出异常，并且回滚，```transactions``` 自然减为 0；



然后我看看我们项目中所使用的方法，通过```DB::beginTransaction();```
开始，然后中途出错，直接写 ```DB::rollback()``` ，
然后 ```return``` 返回，最后成功提交 ```DB::commit()```。

先看看 ```beginTransaction()``` 的源码:
``` php
    public function beginTransaction()
    {
        if ($this->transactions == 0) {
            try {
                $this->pdo->beginTransaction();
            } catch (Exception $e) {
                if ($this->causedByLostConnection($e)) {
                    $this->reconnect();
                    $this->pdo->beginTransaction();
                } else {
                    throw $e;
                }
            }
        } elseif ($this->transactions >= 1 && $this->queryGrammar->supportsSavepoints()) {
            $this->pdo->exec(
                $this->queryGrammar->compileSavepoint('trans'.($this->transactions + 1))
            );
        }
        $this->transactions++;
        $this->fireConnectionEvent('beganTransaction');
    }

```

可以看到，这里的 ```transactions``` 属性记录多少层事务，通过 ```try catch``` 包裹一个事务开始，如果失败，重新尝试连接，并将```transaction + 1```；

<br>
再看 ```rollback()```
``` php
    public function rollBack()
    {
        if ($this->transactions == 1) {
            $this->pdo->rollBack();
        } elseif ($this->transactions > 1 && $this->queryGrammar->supportsSavepoints()) {
            $this->pdo->exec(
                $this->queryGrammar->compileSavepointRollBack('trans'.$this->transactions)
            );
        }
        $this->transactions = max(0, $this->transactions - 1);
        $this->fireConnectionEvent('rollingBack');
    }
```
这里 **只有在 transactions 为 1 的时候，才去 rollback 整个事务**；

<br>
再来看看 ```commit()```

``` php
    public function commit()
    {
        if ($this->transactions == 1) {
            $this->pdo->commit();
        }
        $this->transactions--;
        $this->fireConnectionEvent('committed');
    }
```
也是只有当 ```transactions == 1``` 的时候才会 ```commit``` 整个事务；

不像 ```transaction``` 闭包那样，有整个 ```try catch``` 包裹，这里每一个步骤都要自己控制，

项目中事务的写法经常是：
``` php
DB::beginTransaction();

$res = OrderDelivery::insert([...]);
if (!$res) {
    DB::rollback();
    return $this->fail(-1, '创建失败');
}
$delivery_skus = json_decode($color_size_matrix, true);
OrderDeliveryGoodsSku::insert($delivery_skus);
...
...
DB::commit();

```

那么就有问题了，如果某个步骤出错，但是因为没有抛出异常，因为查看 API 像 ```insert``` 失败会返回 ```false```；

需要人工判断返回值去 ```rollback```，如果没有判断，也没有 ```rollback```，这样 ```transactions``` 就没有减到 1；
这就有可能跑到 ```commit``` 那里去给 ```transactions - 1``` 了；

# 结论：

------------------

所以较好的方法还是，

1. ```transaction``` 带闭包参数；
2. ```beginTransaction + try catch```

用 ```beginTransaction + try catch```，只要有问题，直接到 ```catch``` 那做一次回滚即可，不用担心哪里忘了 ```rollback```；

示例：
``` php
DB::beginTrasaction();
try {
    OrderModificationGoodsSku::insertGetId();
    OrderOweGoodsSku::where()->update();
    DB::commit();
} catch (\Throwable $e) {
    // For php 7
    DB::rollback();
    //throw $e;
    Log::error($e->getMessage());
    return $this->fail(-1, "创建失败");
} catch(\Exception $e) {
    // For php5
    DB::rollback();
    //throw $e;
    Log::error($e->getMessage());
    return $this->fail(-1, "创建失败");
}

```
