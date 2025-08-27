# Kingbase查看数据库隔离级别

## 背景

DBMS: KingbaseES(ver. 12.1)
Driver: Kingbase8 JDBC Driver (ver.V008R006C007B0012, JDBC4.2)

## 解决方案

* 查看当前会话的事务隔离级别

```sql
SELECT current_setting('transaction_isolation');
```

| transaction_isolation |
|-----------------------|
| read committed        |

* 查看所有会话的事务隔离级别

```sql
SELECT pid                                      AS 进程ID,
       usename                                  AS 用户名,
       application_name                         AS 应用名称,
       backend_start                            AS 连接时间, -- 关键列：当前事务的隔离级别
       current_setting('transaction_isolation') AS 事务隔离级别
FROM pg_stat_activity
WHERE backend_type = 'client backend'; -- 仅显示客户端连接
```

| 进程ID  | 用户名      | 应用名称 | 连接时间                | 事务隔离级别         |
|-------|----------|------|---------------------|----------------|
| 12345 | kinguser | psql | 2025-07-23 10:30:00 | read committed |


