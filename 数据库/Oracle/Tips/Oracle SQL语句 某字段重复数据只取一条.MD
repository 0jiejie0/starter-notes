# Oracle SQL语句 某字段重复数据只取一条

## 背景

环境：甲方某系统用了Oracle，我用的是MySQL5.6.46。
业务场景：开发中模拟Oracle数据库，缺少某视图，遂自行根据部分数据表内容，创建数据。

## 问题描述

数据主要有两项内容，一个是name，一个是code，code理论上唯一，一个code只对应一个name（反之不是，name不要求唯一），但是在数据表中，该code并不唯一，
这就产生了题目当中的问题，怎么利用sql语句，在某字段有重复数据的情况下，只取其中一条数据？

> 有人可能会说，code重复，代码逻辑里去掉不就行了吗，为什么一定要要求源数据唯一？
>
> 我在代码中当然是有这个去重逻辑的，本文的问题并不是在开发中遇到，而是在测试时，修改某条源数据，目标数据并没有随之改变，
> 调试发现是源数据唯一code不唯一，我的代码逻辑中是只取第一次出现的code有效值，第二次及之后出现的code都会忽略。
>
> 所以想要测试数据修改的状况，要么我把重复源数据全部更改，要么就解决本文的中心问题。
> 所以，还是做点有技术含量的东西（虽然本文技术含量并不高，但积少成多嘛，至少积累个熟练度）。

## 思路分析

手写了一条SQL语句，不能用，想起来原来的某段代码中似乎是有去重语句的

```sql
select code, name
from op
where is_receive = 0
group by code
```

但是，这是在MySQL5.6.46中执行的，在Oracle中这个语句是不能用的
（MySQL能这么写的原因是，mysql5.7.5版本之前，没有严格校验这种情况，如果刚好每组id都有相同的name字段，也是能查出来的。
本结论来源[CSDN博客](https://blog.csdn.net/java_zhangshuai/article/details/110094569)，仅作摘录，未验证）

我知道还有一个去重的方法，distinct关键字，然而这个关键字只能用在select出的整条记录进行去重，并不能针对某字段进行去重。

最终只能用group by 来做这件事，select的字段，要么是在group中指定的分组依据，要么使用聚集函数。
所有最终方案呼之欲出，group by指定code，聚集函数取name

## 解决方案

取code很简单，那取name应该用什么聚集函数呢？
答案是：随便，max，min都可以（这两个能取出字段单行值，也可以使用GROUP_CONCAT将字段的多行拼接到一起（MySQL），默认逗号分隔符）

```sql
create view V_ATTR (CODE, NAME) as
select CODE, max(NAME)
from op
where is_receive = 0
group by CODE
```

最后，创建view别忘了起个列lie名（刚刚看成了别名）
