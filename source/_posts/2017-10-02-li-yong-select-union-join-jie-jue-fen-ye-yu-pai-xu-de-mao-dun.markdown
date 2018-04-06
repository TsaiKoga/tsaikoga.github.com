---
layout: post
title: "利用select union join 解决分页与排序的矛盾"
date: 2017-10-02 18:34
comments: true
categories: [Mysql]
---

刚入职新公司的时候，被数据库的设计吓了一跳；要在一个有缺陷的数据库上解决数据量很大而导致的查询速度慢问题。

首先，先来讲一下这个场景，这是一个APP的交易页面，他要显示的列表信息是衣服的款号，在这款号下面有这个款的几个SKU
（颜色尺码），以及SKU对应的销售量和交易额。

![视图场景](/images/posts/2017-10-02/mysql-view.jpg "视图场景")

然后，数据库中有三个单据表：开单OrderCreate，改单OrderModification，退货单ReturnGoods；
这三张表各有三个子表 开单子表OrderCreateGoodsSku，改单子表OrderModificationGoodsSku，退货子表ReturnGoodsSku；
这些子表都有对应的goods_count和goods_price，分别代表着交易数量和单价；

所以，销售量是（开单+改单-退货）的goods_count，
交易额是（开单goods_count * 开单goods_price + 改单goods_count * 改单goods_price - 退货goods_count * 退货goods_price)
当然，由于还要显示颜色尺码以及款号，我们还需要去join 有这些字段的表才行。

由于全部时间单据实在太多，一次性找出来构造成所需要的形式也很耗时；再者，随着时间推移，单据越来越多，这查询速度就更加不能接受了。

所以应该使用分页解决。但是这里还有一个需求，就是要根据交易额和销售量排序。也就是三张表查出的数据进行计算后的结果集进行排序。

但是这样就无法真正意义上的分页了，所以有没有办法 **在排序后，取出结果集前进行分页** 呢？

CTO建议使用视图。


### 什么是视图

--------------
视图是一个 **虚拟表**，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。
但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在 **引用视图时动态生成**。

视图只是一段逻辑。

### 使用视图解决排序与分页的矛盾

现在我们只需要把三张子表的内容全部 union 成一张虚拟表就行了

``` sql
CREATE ALGORITHM=UNDEFINED DEFINER=`juniu`@`%` SQL SECURITY DEFINER VIEW `juniu_transfer` AS
(select `juniu_order_create_goods_sku`.`goods_id` AS `goods_id`,`juniu_order_create_goods_sku`.`goods_count` AS `goods_count`,`juniu_order_create_goods_sku`.`goods_price` AS `goods_price`,`juniu_order_create_goods_sku`.`goods_sku_id` AS `goods_sku_id`,`juniu_order_create_goods_sku`.`store_id` AS `store_id`,`juniu_order_create_goods_sku`.`customer_id` AS `customer_id`,`juniu_order_create_goods_sku`.`seller_user_id` AS `seller_user_id`,`juniu_order_create_goods_sku`.`operate_user_id` AS `operate_user_id`,`juniu_order_create_goods_sku`.`timestamp` AS `timestamp`,`juniu_order_create_goods_sku`.`transaction_id` AS `transaction_id`,`juniu_order_create_goods_sku`.`dev_flag` AS `dev_flag`,`juniu_order_create_goods_sku`.`deleted_at` AS `deleted_at`
 from `juniu_order_create_goods_sku`)
union all
(select `juniu_order_modification_goods_sku`.`goods_id` AS `goods_id`,`juniu_order_modification_goods_sku`.`goods_count_diff` AS `goods_count_diff`,`juniu_order_modification_goods_sku`.`goods_price` AS `goods_price`,`juniu_order_modification_goods_sku`.`goods_sku_id` AS `goods_sku_id`,`juniu_order_modification_goods_sku`.`store_id` AS `store_id`,`juniu_order_modification_goods_sku`.`customer_id` AS `customer_id`,`juniu_order_modification_goods_sku`.`seller_user_id` AS `seller_user_id`,`juniu_order_modification_goods_sku`.`operate_user_id` AS `operate_user_id`,`juniu_order_modification_goods_sku`.`timestamp` AS `timestamp`,`juniu_order_modification_goods_sku`.`transaction_id` AS `transaction_id`,`juniu_order_modification_goods_sku`.`dev_flag` AS `dev_flag`,`juniu_order_modification_goods_sku`.`deleted_at` AS `deleted_at`
  from `juniu_order_modification_goods_sku`)
union all
(select `juniu_return_goods_sku`.`goods_id` AS `goods_id`,(-(1) * `juniu_return_goods_sku`.`goods_count`) AS `-1*goods_count`,`juniu_return_goods_sku`.`goods_price` AS `goods_price`,`juniu_return_goods_sku`.`goods_sku_id` AS `goods_sku_id`,`juniu_return_goods_sku`.`store_id` AS `store_id`,`juniu_return_goods_sku`.`customer_id` AS `customer_id`,`juniu_return_goods_sku`.`seller_user_id` AS `seller_user_id`,`juniu_return_goods_sku`.`operate_user_id` AS `operate_user_id`,`juniu_return_goods_sku`.`timestamp` AS `timestamp`,`juniu_return_goods_sku`.`transaction_id` AS `transaction_id`,`juniu_return_goods_sku`.`dev_flag` AS `dev_flag`,`juniu_return_goods_sku`.`deleted_at` AS `deleted_at`
  from `juniu_return_goods_sku`);
```

这就是创建的视图，它有了我们想要的goods_count和goods_price，并且将ReturnGoodsSku的goods_count改为了负数，这样我们可以一次性SUM起来，
实现交易额和销量的运算；

然后在取出所有数据之前对结果集进行排序。最后通过top或limit取出来；

你也可以先建立这个视图的Model，然后就可以通过ORM来获取结果集。

最后，说说效果，其实效果还是不能接受，因为三张表的数据 union all 后数据量太庞大，导致加载速度还是非常非常慢，union all时间实在太长了；
mysql 视图又无法使用索引；所以鱼与熊掌不可得兼。

### 使用 select union join as 解决排序与分页的矛盾

那么该怎么办呢，我是可以试图把union all的数据减少，也就是先用 where 进行筛选然后再 union all；
然后对这些已经筛选并且合并的数据进行 join 关联操作（这样可以 **大大减少 join 关联的内容**）；
最后再次将他们 select 出来：

``` sql
SELECT
	g.goods_id,
	SUM(t.saleVolumn) AS saleVolumn
FROM
	juniu_goods AS g
LEFT JOIN (
	(
		SELECT
			c.goods_id AS goods_id,
			c.goods_count AS goods_count,
			c.goods_price AS goods_price,
			sum(
				c.goods_count * c.goods_price
			) AS saleVolumn,
			c.goods_sku_id AS goods_sku_id,
			c.store_id AS store_id,
			c.customer_id AS customer_id,
			c.seller_user_id AS seller_user_id,
			c.operate_user_id AS operate_user_id,
			c. TIMESTAMP AS TIMESTAMP,
			c.transaction_id AS transaction_id,
			c.dev_flag AS dev_flag,
			c.deleted_at AS deleted_at,
			'create' AS order_type,
			0 AS return_count
		FROM
			juniu_order_create_goods_sku AS c
		WHERE
			c.store_id = 651
		GROUP BY
			goods_id
	)
	UNION ALL
		(
			SELECT
				m.goods_id AS goods_id,
				m.goods_count_diff AS goods_count_diff,
				m.goods_price AS goods_price,
				sum(
					m.goods_count_diff * m.goods_price
				) AS saleVolumn,
				m.goods_sku_id AS goods_sku_id,
				m.store_id AS store_id,
				m.customer_id AS customer_id,
				m.seller_user_id AS seller_user_id,
				m.operate_user_id AS operate_user_id,
				m. TIMESTAMP AS TIMESTAMP,
				m.transaction_id AS transaction_id,
				m.dev_flag AS dev_flag,
				m.deleted_at AS deleted_at,
				'modification' AS order_type,
				0 AS return_count
			FROM
				juniu_order_modification_goods_sku AS m
			WHERE
				m.store_id = 651
			GROUP BY
				goods_id
		)
	UNION ALL
		(
			SELECT
				r.goods_id AS goods_id,
				(- 1 * r.goods_count) AS goods_count,
				r.goods_price AS goods_price,
				sum(- 1 * r.goods_price) AS saleVolumn,
				r.goods_sku_id AS goods_sku_id,
				r.store_id AS store_id,
				r.customer_id AS customer_id,
				r.seller_user_id AS seller_user_id,
				r.operate_user_id AS operate_user_id,
				r. TIMESTAMP AS TIMESTAMP,
				r.transaction_id AS transaction_id,
				r.dev_flag AS dev_flag,
				r.deleted_at AS deleted_at,
				'return' AS order_type,
				r.goods_count AS return_count
			FROM
				juniu_return_goods_sku AS r
			WHERE
				r.store_id = 651
			GROUP BY
				goods_id
		)
) AS t ON g.goods_id = t.goods_id
WHERE
	g.store_id = 651
GROUP BY
	g.goods_id
ORDER BY
	saleVolumn DESC
LIMIT 0,
 20
```
这个效果拔群！
