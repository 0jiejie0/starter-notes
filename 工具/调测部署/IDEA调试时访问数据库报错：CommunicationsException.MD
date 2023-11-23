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

利用计算器再次执行或是过长时间执行，则出现如下信息：

```log
org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 68886ms.
### The error may exist in file [D:\...\mapper\AMapper.xml]
### The error may involve com.example.mapper.opt.getInfo
### The error occurred while executing a query
### Cause: org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 68886ms.
```

## 分析

第一段信息里说了，距离上次请求数据的时间间隔，超过了服务器设定的“wait_timeout”等待超时时间，由于调试时间过长，导致数据库服务端连接超时断开，造成应用侧连接错误。

第二块信息可能是达到了数据库连接池连接的超时时间，导致连接失效。

这二者间具体原因的区别，有待考究，这里描述也不严谨，仅是根据错误信息推测。

## 解决方案

* 点击继续运行按钮（debug窗口绿色三角），重新发送网络请求，重新调试。

超时报错可遇不可求，下面两种方法没有验证（ai助手给出，其实错误信息里提示了），遇到再说

* 增加服务器配置的客户端超时时长，
* 数据库连接中加入’autoReconnect=true’
