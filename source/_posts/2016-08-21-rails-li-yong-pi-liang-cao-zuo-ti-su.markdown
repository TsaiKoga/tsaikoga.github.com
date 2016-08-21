---
layout: post
title: "Rails 利用批量操作提速"
date: 2016-08-21 14:29
comments: true
categories: [Rails, Mysql]
---

#### [1.1 批量查找数据](#1.1)
##### [1.2 操作代码](#1.2)
#### [2.1 批量插入数据](#2.1)
##### [2.2 INFILE文件插入数据](#2.2)
#### [3.1 批量更改schema数据库](#3.1)


<h2 id="1.1">1.1 批量查找数据</h2>

------------------------------------

由于之前一直在做后台的数据，其中有个任务是关于 财务库存 的页面.

> 这个“财务库存”有产品和日期选项，您可以通过选择产品，查看它在某段时间内的库存价格等情况。

> 将采购入库单数据[PurchaseOrderBill]，销售出库单数据[OutStorageOrder]，异常取货和仓库调整单数据整合在一个页面；
根据日期排序来显示这个页面的内容，以及这些内容的成本价格变化，这里的价格根据这些取出来的数据进行排列后再进行运算得出。

> 项目中有近5w条产品记录，又要将“采购入库单”和“销售出库单”与 产品product进行关联查找，耗时挺大的。

> 这时候公司那边说要提供一次性下载所有产品的财务库存数据，不过不需要显示价格变化情况，直接显示最后一条价格结果就行了。

近5w条的 product 产品关联(joins/includes)的单据进行计算价格，然后显示结果。
如何加快查找速度呢？

答案是进行 **批量** 查找。



<h3 id="1.2">1.2 操作代码</h3>

用代码说明吧，首先当然对于这么庞大的数据量，要进行 **分页** 处理：
``` ru
    per_page = 100
    page = 0
    total_page = (Product.count.to_f / per_page).ceil
```

每一页进行 **批量** 操作：
``` ru
    total_page.times do
      products = Product.limit(per_page).offset(page * per_page)
      bill_items = bill_inventory_valuation_details products        # 符合条件的采购入库单数据，此方法的查找也必须对 products 进行批量查找
      sell_items = sell_inventory_valuation_details products        # 符合条件的销售出库数据
      package_items = package_inventory_valuation_details products  # 异常出库数据
      adjust = adjust_inventory_valuation_details products          # 调整仓库数据
      products.each do |product|
        datas = []
        datas += bill_items[product.id].map{|item| {id: item.id, quantity: item.quantity, price: item.price}}
        # 对查出的数据进行处理，构造最终结果
        ... ...
      end
    end
```

这样原先没有进行批量操作会耗时接近12个小时，使用了批量操作后仅为2个多小时。




<h2 id="2.1">2.1 批量插入数据</h2>

--------------------------

当你要对大量数据进行插入数据库的操作时，一条条插入将是及其愚蠢的行为。如何进行批量插入呢？



可以先看看这篇文章：[Mass inserting data in Rails without killing your performance](https://www.coffeepowered.net/2009/01/23/mass-inserting-data-in-rails-without-killing-your-performance/)

这篇文章提到 Rails 快速插入数据的四种方法：

1. Use transactions

2. Get down and dirty with the raw SQL

3. A single mass insert

4. Use INSERT statements with multiple VALUES lists with ar-extensions OR activerecord-import



第四种方式是AR方式的70倍左右，而公司后台使用的正是 activerecord-import,
做法很简单，先将数据创建 **存于内存**，然后批量 import **插入** 数据库：

``` ru
    (controller.action_methods.to_a - excepts).each do |action|
      need_imports << Permission.new(subject: subject, action: action, description: "#{action.humanize} of #{subject}")
    end
    Permission.import need_imports
```

这样已经很快了，还有更快的吗？




<h3 id="2.2">2.2 INFILE文件插入数据 </h3>

From MySQL Doc:

> When loading a table from a text file, use LOAD DATA INFILE. This is usually 20 times faster than using INSERT statements


SQL语句：

```
(15955.1ms)  LOAD DATA LOCAL INFILE '/Users/tsaikoga/out_storage_orders.txt'
REPLACE INTO TABLE out_storage_orders FIELDS TERMINATED BY ',' (`id`,`warehouse_id`,`destination_id`,`user_id`,`out_storage_order_type`,`out_storage_reason_type`,`status`,`quantity`,`package_id`)
```

插入行数：

``` sh
    wc -l < /Users/tsaikoga/out_storage_orders.txt
    100000
```




<h3 id="3.1">3.1 批量更改schema数据库</h3>

----------------------------

你有没有发现，当你某张表的数据非常非常多的时候，你想要更改其中一个字段的类型，然后执行 migrate ；
执行时间非常长，你厌倦了等待，只好 ctrl+c 撤销了migrate，但是实际上数据库中的记录已经有很大一部分改变了字段的类型。
这时候你想再migrate，会出现这种类型的字段已存在的提示。（当然你可以利用mysql的界面工具或直接到mysql更改字段类型进行拯救）


这里提供一个 Rails 的方法，将数据大批量进行更改。
比方说，以下是一段更改 orders 表的 shipping_first_name shipping_last_name 名称的方法：

    rename_column :orders, "shipping_first_name", "origin_shipping_first_name"
    rename_column :orders, "shipping_last_name", "origin_shipping_last_name"

执行migrate用时非常长，还不如使用如下代码：

``` ru
    class RenameOldOrderAddressRelatedColumn < ActiveRecord::Migration
      def up
        change_table(:orders, :bulk => true) do |t|
          t.rename :order_status_code_id, :state
          t.remove :in_process
          [:first_name,
           :last_name,
           :country,
           :state].each do |column|
            t.rename "shipping_#{column}", "origin_shipping_#{column}"
          end
        end
      end
    end
```

以上代码只生成一句 SQL 语句，数据库迁移时间大大减少。

```
    ALTER TABLE `orders` DROP `in_process`, CHANGE `shipping_first_name` `origin_shipping_first_name` varchar(50) DEFAULT NULL, CHANGE `shipping_last_name` `origin_shipping_last_name` varchar(50) DEFAULT NULL, CHANGE `shipping_country` `origin_shipping_country` varchar(255) DEFAULT NULL, CHANGE `shipping_state` `origin_shipping_state` varchar(255) DEFAULT NULL...
```


相关文档：

http://dev.mysql.com/doc/refman/5.5/en/load-data.html
http://dev.mysql.com/doc/refman/5.5/en/insert-speed.html


总结，对于大数据一定要进行批量处理。
