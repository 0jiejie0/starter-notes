## 概念

TIMESTAMPDIFF() 是 MySQL 中用来计算两个日期或时间之间的差值的函数。该函数返回两个日期/时间之间的差值，可以指定差值的单位（秒、分钟、小时、天等）。

## 语法

```sql
TIMESTAMPDIFF
    (unit, start_date, end_date)
```

参数说明：

* unit：表示差值的单位，可以是以下值之一：MICROSECOND（微秒）、SECOND（秒）、MINUTE（分）、HOUR（小时）、DAY（天）、WEEK（周）、MONTH（月）、QUARTER（季度）或
  YEAR（年）。
* start_date：表示时间段的起始时间。
* end_date：表示时间段的结束时间。

## 用法举例

```sql
SELECT TIMESTAMPDIFF(DAY, '2021-01-01', '2021-01-10')
```

查询结果为9，表示相差九天

[参考文章](https://blog.csdn.net/monicateacat/article/details/131249593)，实测中把这两个时间替换为了某字段，计算相差的分钟数，可行。
