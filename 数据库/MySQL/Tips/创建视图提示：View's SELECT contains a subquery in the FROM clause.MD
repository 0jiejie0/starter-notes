# 创建视图提示：View's SELECT contains a subquery in the FROM clause.

    mysql、view、select、subquery

## 背景

* 环境版本：mysql 5.6.46
* 业务场景：甲方要求提供一个查询视图，字段比较多，处理逻辑也比较复杂，什么分组、子查询、排序、多表连接都用上了

## 问题描述

* 现象

创建视图的语句大概是这样（查询语句从网上扒的，以免泄密）：

```sql
create definer = root@`%` view a_view as
SELECT c.scan_man AS scan_man, c.DOC_ID AS DOC_ID, c.db_time arrive_time, d.db_time assign_time
FROM (SELECT DOC_ID AS DOC_ID, DB_TIME AS db_time, SCAN_MAN AS scan_man FROM test WHERE SCAN_TYPE = 13) c
         LEFT JOIN (SELECT DOC_ID AS DOC_ID, DB_TIME AS db_time
                    FROM test
                    WHERE SCAN_TYPE = 10) d
                   ON c.DOC_ID = d.DOC_ID
GROUP BY c.DOC_ID, c.scan_man;
```

不加 create view 的时候，单独执行select语句一切正常，数据也没问题

* 开发及调测工具信息

加上create view语句，在idea的数据库console文件里执行（和select一样的执行方式），下面的services窗口却提示：`View's SELECT
contains a subquery in the FROM clause.`

## 原因分析

* 经验之谈：一看这英文不是说的什么子查询吗，有子查询有什么问题吗，单纯select也没什么问题啊？而且这是说的select字段中的子句，还是说的where还是说的join？不明白指的是哪里；
* 周边见解：我写一半的时候有人和我说，我原来的写法太费事（所有关联操作都放在了where里），建议我直接在select的字段处写，我当时虽然非常怀疑这个性能，但还是照做了；
* 网络搜索：找到[一篇博客](https://blog.csdn.net/marks59848660/article/details/51860465/)
  ，原来说的是where里不能有子查询（就是上边SQL的括号里的部分）。

## 解决方案

1. 逐个从where中删除子查询
2. 在select中的相应字段重写查询关联逻辑
