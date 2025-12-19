# KingbaseES查看与监控事务隔离级别

    人大金仓、KingbaseES、事务隔离级别、read committed、repeatable read、SQL查询、会话监控、pg_stat_activity

## 背景

* **环境版本**：KingbaseES V12.1 (基于PostgreSQL 12内核，项目使用Oracle模式)
* **技术框架**：JDBC 4.2 (Kingbase8 Driver V008R006C007B0012)
* **业务场景**：
    * 在事务相关的功能开发或数据一致性排查时，需确认当前数据库连接的事务隔离级别。
    * 在性能分析或高并发场景下，需要监控所有应用连接使用的隔离级别，判断配置是否符合预期。
    * 默认隔离级别为“读已提交”，但应用可能通过`SET TRANSACTION ISOLATION LEVEL`或连接参数修改，需验证。

## 问题描述

* **现象**：需要查询KingbaseES数据库中当前会话（或所有会话）的事务隔离级别，但不确定应使用何种系统函数或视图。
* **开发及调测工具信息**：在金仓管理工具、DBeaver或应用程序的日志中，无直接的报错信息，属于信息查询需求。

## 原因分析

* **经验之谈**：KingbaseES与PostgreSQL高度兼容，其事务隔离级别的设置和查询方式也基本一致。
* **周边见解**：隔离级别是影响并发事务下数据可见性和一致性的核心参数，明确其设置是排查“脏读”、“不可重复读”、“幻读”等问题的基础。
* **网络搜索**：KingbaseES官方文档中关于`transaction_isolation`参数的说明，以及`pg_stat_activity`系统视图的字段解释。
* **深入探究**：`transaction_isolation`是一个会话级（session-level）运行时参数。`current_setting()`
  函数用于获取当前会话的参数值。`pg_stat_activity`视图包含了每个后端进程（包括客户端连接）的实时信息，结合函数可查看所有会话的隔离级别。

## 解决方案

**步骤：**

1. **连接数据库**：使用任何SQL客户端（如金仓管理工具、psql、DBeaver）或通过应用程序的JDBC连接执行以下SQL。
2. **查询当前会话的隔离级别**：执行单行查询，直接返回结果。
   ```sql
   -- 最直接的方法，返回当前连接的事务隔离级别
   SELECT current_setting('transaction_isolation');
   ```
   **预期返回示例：**
   ```
    transaction_isolation
   -----------------------
    read committed
   ```
3. **（可选）查询所有活动会话的隔离级别**：此查询有助于监控和分析。
   ```sql
   -- 查看所有客户端连接的进程及其隔离级别
   SELECT pid                                      AS 进程ID,
          usename                                  AS 用户名,
          application_name                         AS 应用名称,
          client_addr                              AS 客户端IP,
          backend_start                            AS 连接开始时间,
          current_setting('transaction_isolation') AS 事务隔离级别
   FROM pg_stat_activity
   WHERE backend_type = 'client backend'  -- 筛选出客户端进程
     AND state IS NOT NULL;               -- 排除空闲进程（可选）
   ```
   **返回结果将显示一个列表，每行代表一个活动连接及其详细信息。**

## 补充说明

* **常见隔离级别值**：在KingbaseES/PostgreSQL中，可能返回的值包括 `read uncommitted`（极少使用）、`read committed`
  （默认）、`repeatable read`、`serializable`。
* **修改隔离级别**：如需修改当前会话的隔离级别，可在事务开始时使用 `SET TRANSACTION ISOLATION LEVEL <level>;`
  命令，例如 `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;`。
* **驱动参数**
  ：也可以在JDBC连接字符串中设置默认隔离级别，例如在URL中添加 `?defaultTransactionIsolation=TRANSACTION_REPEATABLE_READ`。

> 本文经AI调整，如有不妥还请赐教
