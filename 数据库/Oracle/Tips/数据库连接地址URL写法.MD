# Oracle JDBC 连接地址URL写法

## 解决方案

三种方式：

* 使用标准的URL格式

在集群中，service name可以包含所有节点，推荐使用本方式

```
jdbc:oracle:thin:@//host:port/service_name
例如: jdbc:oracle:thin:@//localhost:1521/orcl.city.com
```

获取服务名称的方法：

```Bash
sqlplus / as sysdba
select value from v$parameter where name='service_names';
```

* 使用SID连接方式

对于集群来说，每个节点的sid都不一样

```
jdbc:oracle:thin:@hostname:portNumber:SID 
例如：jdbc:oracle:thin:@localhost:1521:orcl
```

获取sid的方式：

```Bash
sqlplus / as sysdba 
select value from v$parameter where name='instance_name';
```

* 使用TNS连接方式

1. 建立tnsnames.ora文件
2. 通过System.setProperty指明这个文件路径
3. 通过下面URL中的@符号指定文件中的要使用到的资源。

```
jdbc:oracle:thin:@TNSName 
例如：jdbc:oracle:thin:@myTNSName
```

节选自[CSDN博客](https://blog.csdn.net/weixin_58094032/article/details/128950000)
