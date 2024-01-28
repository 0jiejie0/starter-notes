# JDBC 提交失败 Can't call commit when autocommit=true

    SpringBoot、MyBatis、MySQL

本文可能是一个解决方案，不保熟，不保证一定有效，仅仅是提供一个思路。

## 背景

业务系统的同步服务在一次异常断电重启后，不能正常启动，报错，本文是其中出现的问题之一。

## 问题描述

错误信息大概如下：

```log
Caused by: java.sql.SQLException: Can't call commit when autocommit=true
    at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:930)
    at com.mysql.jdbc.ConnectionImpl.commit(ConnectionImpl.java:1602)
    at org.hibernate.transaction.JDBCTransaction.commitAndResetAutoCommit(JDBCTransaction.java:170)
    at org.hibernate.transaction.JDBCTransaction.commit(JDBCTransaction.java:146)
    ... 6 more
```

## 原因分析

* 启用自动提交后，即 autoCommit = true 的情况下，再次调用commit()、rollback()时，往往会报这个异常。

## 解决方案

* 使用connection.setAutoCommit(false) （[参考CSDN博客](https://blog.csdn.net/qq_15371841/article/details/76584979)，未测试）
* 在JDBC的URL里加上 relaxAutoCommit = true 参数 （[参考评论-grem](https://www.oomake.com/question/1297201)）
