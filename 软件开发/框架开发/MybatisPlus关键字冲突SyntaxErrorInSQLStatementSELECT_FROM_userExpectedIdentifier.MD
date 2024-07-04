# JdbcSQLSyntaxErrorException: Syntax error in SQL statement "SELECT  id,name,age,email  FROM [*]user"; expected "identifier"; SQL statement:

    SpringBoot3.2.7、mybatis-plus3.5.7、mybatisplus、user、h2、入门、快速开始、demo、maven、idea

## 背景

近来了解到MybatisPlus对于表的简单查询，竟然不用写sql，连xml文件都不用建，故此学习[MP入门](https://baomidou.com/getting-started/)。

然而万事开头难，项目启动一波三折，这个教程的第一页我就卡了下来，小问题不断，好在多数都是一搜我就知道怎么办了。

然而知道遇到了题目这个问题，我花在上面得好几个小时，才顺利启动项目，特此记录。

## 问题描述

按照背景中提到的教程，我创建了一个springboot3.2.7的项目，导入MP依赖：

```xml

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>
    <version>3.5.7</version>
</dependency>
```

在**资源文件夹`resources`下**新建`db`文件夹，创建`db/schema-h2.sql`（此处有坑）：

```h2
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user`
(
    id    BIGINT      NOT NULL COMMENT '主键ID',
    name  VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age   INT         NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);
```

以及`db/data-h2.sql`。

在`application.yml`中加入：

```yml
spring:
  application:
    name: demoMybatisPlus # 自定义的项目名
  datasource:
    driver-class-name: org.h2.Driver
    username: root
    password: test
  sql:
    init:
      schema-locations: classpath:db/schema-h2.sql
      data-locations: classpath:db/data-h2.sql
```

在 Spring Boot 启动类中添加 @MapperScan 注解，扫描 **Mapper 文件夹**，添加此注解后，各mapper.java文件中就不再需要@Mapper/@Component注解了，

```java
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
```

创建User.java实体类（此处有坑）：

```java

@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

然后，创建测试类开始测试，使用`userMapper.selectList`方法，报错：

```log
org.springframework.jdbc.BadSqlGrammarException: 
### Error querying database.  Cause: org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "SELECT  id,name,age,email  FROM [*]user"; expected "identifier"; SQL statement:
SELECT  id,name,age,email  FROM user [42001-224]
### The error may exist in com/xkm/demoMybatisPlus/mapper/UserMapper.java (best guess)
### The error may involve com.xkm.demoMybatisPlus.mapper.UserMapper.selectList
### The error occurred while executing a query
### SQL: SELECT  id,name,age,email  FROM user
### Cause: org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "SELECT  id,name,age,email  FROM [*]user"; expected "identifier"; SQL statement:
SELECT  id,name,age,email  FROM user [42001-224]
; bad SQL grammar []

	at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.doTranslate(SQLErrorCodeSQLExceptionTranslator.java:246)
	at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:107)
	at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:92)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:439)
	at jdk.proxy2/jdk.proxy2.$Proxy55.selectList(Unknown Source)
	at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:224)
	at com.baomidou.mybatisplus.core.override.MybatisMapperMethod.executeForMany(MybatisMapperMethod.java:164)
	at com.baomidou.mybatisplus.core.override.MybatisMapperMethod.execute(MybatisMapperMethod.java:77)
	at com.baomidou.mybatisplus.core.override.MybatisMapperProxy$PlainMethodInvoker.invoke(MybatisMapperProxy.java:152)
	at com.baomidou.mybatisplus.core.override.MybatisMapperProxy.invoke(MybatisMapperProxy.java:89)
	at jdk.proxy2/jdk.proxy2.$Proxy59.selectList(Unknown Source)
	at com.xkm.demoMybatisPlus.SampleTest.testSelect(SampleTest.java:21)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
	at java.base/java.util.ArrayList.forEach(ArrayList.java:1511)
Caused by: org.h2.jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "SELECT  id,name,age,email  FROM [*]user"; expected "identifier"; SQL statement:
SELECT  id,name,age,email  FROM user [42001-224]
	at org.h2.message.DbException.getJdbcSQLException(DbException.java:514)
	at org.h2.message.DbException.getJdbcSQLException(DbException.java:489)
	at org.h2.message.DbException.getSyntaxError(DbException.java:261)
	at org.h2.command.Parser.readIdentifier(Parser.java:5533)
	at org.h2.command.Parser.readTablePrimary(Parser.java:1739)
	at org.h2.command.Parser.readTableReference(Parser.java:2268)
	at org.h2.command.Parser.parseSelectFromPart(Parser.java:2718)
	at org.h2.command.Parser.parseSelect(Parser.java:2824)
	at org.h2.command.Parser.parseQueryPrimary(Parser.java:2708)
	at org.h2.command.Parser.parseQueryTerm(Parser.java:2564)
	at org.h2.command.Parser.parseQueryExpressionBody(Parser.java:2543)
	at org.h2.command.Parser.parseQueryExpressionBodyAndEndOfQuery(Parser.java:2536)
	at org.h2.command.Parser.parseQueryExpression(Parser.java:2529)
	at org.h2.command.Parser.parseQuery(Parser.java:2498)
	at org.h2.command.Parser.parsePrepared(Parser.java:627)
	at org.h2.command.Parser.parse(Parser.java:592)
	at org.h2.command.Parser.parse(Parser.java:564)
	at org.h2.command.Parser.prepareCommand(Parser.java:483)
	at org.h2.engine.SessionLocal.prepareLocal(SessionLocal.java:639)
	at org.h2.engine.SessionLocal.prepareCommand(SessionLocal.java:559)
	at org.h2.jdbc.JdbcConnection.prepareCommand(JdbcConnection.java:1166)
	at org.h2.jdbc.JdbcPreparedStatement.<init>(JdbcPreparedStatement.java:93)
	at org.h2.jdbc.JdbcConnection.prepareStatement(JdbcConnection.java:316)
	at com.zaxxer.hikari.pool.ProxyConnection.prepareStatement(ProxyConnection.java:327)
	at com.zaxxer.hikari.pool.HikariProxyConnection.prepareStatement(HikariProxyConnection.java)
	at org.apache.ibatis.executor.statement.PreparedStatementHandler.instantiateStatement(PreparedStatementHandler.java:88)
	at org.apache.ibatis.executor.statement.BaseStatementHandler.prepare(BaseStatementHandler.java:90)
	at org.apache.ibatis.executor.statement.RoutingStatementHandler.prepare(RoutingStatementHandler.java:60)
	at org.apache.ibatis.executor.SimpleExecutor.prepareStatement(SimpleExecutor.java:90)
	at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:64)
	at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:336)
	at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:158)
	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:110)
	at org.apache.ibatis.executor.CachingExecutor.query(CachingExecutor.java:90)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:154)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:147)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:142)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:425)
	... 11 more
```

## 分析

这个报错说得很好，期望标识符，就是说缺标识符，可是，对照官方教程，命名不缺标识符啊。

我再吐槽一下，文心一言真垃圾。

找到[stackoverflow上一篇和我问题差不多的帖子](https://stackoverflow.com/questions/71722483/org-h2-jdbc-jdbcsqlsyntaxerrorexception-syntax-error-in-sql-statement-drop-tab/71722686)，
里边说，user是新版H2的关键字，要使用这个词就必须要转义它，帖子说要用双引号"替换单引号'。

但是在我的项目中，我其实需要反引号，因为sql文件中的建表语句就是反引号。

这里说user是**新版**H2的关键字，为啥是新版才有呢？我写着写着想起，MySQL中，这个user也是个关键字啊，根本上来说，这个教程就不应该用user这个名字。

不过一味逃避也不是办法，如果真的有个表不能改名，必须要用关键字，应该怎么解决呢？也就是说，转义user，应该在哪里转义呢？
我想到了搜索过程中看到的[一篇CSDN博客](https://blog.csdn.net/wondersfan/article/details/126631804)，这里也是user表和关键字冲突，
不过文中使用`@Table(name="UserLxy")`貌似是修改了表名映射关系。

我的项目中没有使用`Hibernate/JPA`，但受此启发，我尝试输入@Table相关注解，找到

```java
@TableName("`user`")
```

其中使用反引号包装表名，但是，这个注解加在哪里呢？加到mapper上发现不行；
但转念一想，实体类对应数据库的实体表，那就应该把这个注解加在实体类User上，实测可行。

[开启SQL执行日志](./MybatisPlus在控制台输出sql执行日志.MD)，发现没有这个注解时，底层执行的表名不带任何符号；

如果此处使用反引号包装表名，那么执行SQL中的表名就是带有反引号的，相应的，其建表语句也得是反引号；

如果此处使用双引号（java中另需反斜杠\转义）包装表名，那执行SQL中就会带有双引号，相应的，其建表语句也得是双引号；

如果此处使用单引号包装表名，那执行SQL中就会带有单引号，但是，scheme文件中，**冲突关键字就不能使用单引号**了。

综上，个人倾向于在sql文件和java代码中使用反引号（使用非关键字表名时可能会有问题，待验证），其一不用多余转义符号，其二避免冲突方式简洁。

## 解决方案

1. 如果修改实体类名、修改表名成本不高的话，可以将其改为非关键字；
2. 在尾大不掉、改名成本高的情况下，在实体类上使用 @TableName("\`user\`") ，手动指定实体类和表的映射关系
