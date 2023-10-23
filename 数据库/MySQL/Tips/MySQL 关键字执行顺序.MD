## MySQL 关键字执行顺序如下：

```sql
from
on
join
where
group by
select
having
order by
limit
```

## 涉及问题

select 后写的别名，只有在select之后执行的语句中才能用，如 having/order by/limit

参照[此文](https://blog.csdn.net/m0_69057918/article/details/132146466)
