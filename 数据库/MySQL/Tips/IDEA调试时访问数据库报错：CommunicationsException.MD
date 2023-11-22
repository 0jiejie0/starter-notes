# IDEA调试时Mapper访问数据库报错：CommunicationsException: The last packet successfully received

## 背景

很普通的一次调试，之前好像也遇到过，记录一下。
springboot mybatis

## 问题描述

IDEA debug模式启动，网络请求进入某service断点，手动单步一下下执行，在调用另一个service时报错，错误栈定位到其中的一个mapper接口，错误信息如下：

```log
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 574,962 milliseconds ago.  The last packet sent successfully to the server was 574,962 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
### The error may exist in file [D:\...\mapper\AMapper.xml]
### The error may involve com.example.mapper.opt.getInfo-Inline
### The error occurred while setting parameters
### SQL: select 
```

或是出现如下信息：

```log
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 68886ms.
### The error may exist in file [D:\...\mapper\AMapper.xml]
### The error may involve com.example.mapper.opt.getInfo
### The error occurred while executing a query
### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 68886ms.
```

## 分析

推测是由于调试时间过长，导致数据库连接超时断开，造成程序侧连接错误。

第二块信息可能是达到了数据库连接池连接超时时间，导致连接失效。

## 解决方案

* 点击继续运行按钮（debug窗口绿色三角），重新发送网络请求，重新调试。

超时报错可遇不可求，下面两种方法没有验证，遇到再说

* 增加服务器配置的客户端超时时长，
* 数据库连接中加入’autoReconnect=true’
