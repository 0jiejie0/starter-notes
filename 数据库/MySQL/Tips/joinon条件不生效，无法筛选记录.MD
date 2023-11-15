# 多表关联查询，join on 条件不生效，无法筛选记录

    MySQL、left join、join on、筛选、多表关联查询

## 背景

* 环境版本：局域网-win10-MySQL5.6
* 业务场景：MySQL多表联查，意图把查询条件按照[关键字执行顺序](./MySQL%20关键字执行顺序.MD)往前提，尽可能降低数据库查询对性能的占用。

## 问题描述

* 现象（简化）

现有两个表doc(key,time)（这是个列转行后的虚表）、main(key,id)（实体表）,程序中执行:

```sql
select d.key,m.id,d.time
from doc d
         left join main m on m.id is not null and d.time >= '2023-11-15'
```

doc表中没有15号以后的数据，但是在运行结果中，不符合这个条件的doc数据仍然存在。手动拼写sql语句执行，发现不合条件语句仍然存在。
然而应与其关联的子表的数据(m.id)均为null

## 原因分析

不难想到，这是因为关联条件on加到了子表上，没有控制到主表。

[//]: # (todo * 深入探究：对于sql中关键字的优先级仍然困惑，嵌套关联、调整语序后，孰先孰后？如何检验？而在这之中，又有关键字的控制范围未知)

## 解决方案

将d.time筛选条件外移一层，用where查询，例如：
```sql
select d.key, m.id, d.time
from doc d
         left join main m on m.id is not null
where d.time >= '2023-11-15'
```
