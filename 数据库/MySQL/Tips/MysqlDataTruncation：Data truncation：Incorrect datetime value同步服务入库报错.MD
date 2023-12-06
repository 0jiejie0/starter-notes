## 背景

线上新部署的服务有部分数据不能同步，查看错误日志发现报错。
数据流向：Oracle-》MySQL

## 问题描述

错误信息如下：

```log
ERROR [integration] - <[sync]mysql插入更新出错:com.mysql.jdbc.MysqlDataTruncation: Data truncation: Incorrect datetime value: '0001-01-01 00:00:00' for column 'point' at row 1>
ERROR [integration] - <入库原数据出错>
com.mysql.jdbc.MysqlDataTruncation: Data truncation: Incorrect datetime value: '0001-01-01 00:00:00' for column 'point' at row 1
...
```

## 分析

感觉应该是超MySQL范围了，但是我不太清楚MySQL的datetime值域。
直接求助ai助手，给我的答案是，’0001-01-01 00:00:00’是一个不合法的日期时间值，
而MySQL中，datetime类型的取值范围是’1000-01-01 00:00:00’到’9999-12-31 23:59:59’。

在Oracle中，日期范围是从January 1, 4712 BC到December 31, 9999 AD。"BC"代表“公元前”，而"AD"代表“公元后”。

这，也就能解释的通为什么Oracle中会有这样的数据了。所以需要判断一下合法性，然后再入库，我想写成这个样子：

```java

//		String oriTimeStr = "0001-01-01 00:00:00";
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		try {
			Date oriDate = dateFormat.parse(oriTimeStr);
			Date minDate = dateFormat.parse("1000-01-01 00:00:00");
			Date maxDate = dateFormat.parse("9999-12-31 23:59:59");
			if (oriDate.compareTo(minDate) < 0) {
				return dateFormat.format(minDate);
			} else if (maxDate.compareTo(oriDate) < 0) {
				return dateFormat.format(maxDate);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return oriTimeStr;
```

然而我又想起来，前两天看的split源码，试了一下，写在解决方案里，应该可以降低点占用空间（顺便提升点复杂度，嘿嘿），哎这个逼格瞬间提上去了。

但很不幸，原解决方案上线后，还是有这个问题：

```log
ERROR [integration] - <[sync]更新出错:com.mysql.jdbc.MysqlDataTruncation: Data truncation: Incorrect datetime value: '1000-01-01 00:00:00' for column 'point' at row 1>
ERROR [integration] - <入库原数据出错>
com.mysql.jdbc.MysqlDataTruncation: Data truncation: Incorrect datetime value: '1000-01-01 00:00:00' for column 'point' at row 1
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3964)
	at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3902)
	at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:2526)
	at com.mysql.jdbc.ServerPreparedStatement.serverExecute(ServerPreparedStatement.java:1284)
	at com.mysql.jdbc.ServerPreparedStatement.executeInternal(ServerPreparedStatement.java:785)
	at com.mysql.jdbc.PreparedStatement.executeUpdateInternal(PreparedStatement.java:2073)
	at com.mysql.jdbc.PreparedStatement.executeUpdateInternal(PreparedStatement.java:2009)
	at com.mysql.jdbc.PreparedStatement.executeLargeUpdate(PreparedStatement.java:5098)
	at com.mysql.jdbc.PreparedStatement.executeUpdate(PreparedStatement.java:1994)
	at com.zaxxer.hikari.proxy.PreparedStatementProxy.executeUpdate(PreparedStatementProxy.java:61)
	at com.zaxxer.hikari.proxy.HikariPreparedStatementProxy.executeUpdate(HikariPreparedStatementProxy.java)
	at com.allook.framework.connector.HikariCPManager.insertAndUpdate(HikariCPManager.java:285)
	at com.temp.job.Order.getOracleView(Order.java:204)
	at com.temp.job.Order.execute(Order.java:47)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at com.allook.framework.commons.CustomJob.execute(CustomJob.java:77)
	at org.quartz.core.JobRunShell.run(JobRunShell.java:202)
	at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:573)
```

搜了搜发现确实是没有超过datetime范围，那这是什么原因呢？我在本地数据库的一个时间字段里修改数据提交，发现确实出现了这个问题，而且用1022年都不行。
然而我突然发现这个字段的定义竟然是timestamp，ai助手告诉我，它的范围是：

> ’1970-01-01 00:00:01’到’2038-01-19 03:14:07’，精度为秒。timestamp类型存储的时间戳范围受限于其数据类型的大小，精度为秒。timestamp类型存储的时间戳范围受限于其数据类型的大小，通常为4个字节或8个字节。

看了一下线上，果然，正是timestamp。可是，1022年，数据范围都没有超啊，难道是精度超了?那也不对啊？

仔细看一看，**仔细看一看！** 数据范围确实超了，眼花了，1970看成了1009年。调整把代码中的上下限，重新部署，发现这个1970还是报错放不进去。
之前看过timestamp不太好用，而且和时区有关系，我预感这应该是系统时区的问题，手动修改数据为’1970-01-01 00:00:01’，执行报错，
加八个小时，’1970-01-01 08:00:01’，修改成功。

可是我在本地用’1970-01-01 00:00:01’没问题，找到查MySQL时区的命令：

```sql
SELECT @@global.time_zone, @@session.time_zone;
```

查出来发现服务器和本地一样，都是system用的系统时区，那都是windows系统，为什么到服务器就不行了？
因为我本地MySQL在docker里，这个东西的内核是用linux支撑的，而win把硬件时钟作为本地时间，linux把硬件时钟作为utc时间（本地时间计算后显示）。

MySQL timestamp存储用的是utc时间，查询时会把utc时间转为本地时间，
这中间有两处转换，存储时的转换，使用系统时区设置，win下就会把时间作为本地时间，需要回拨8h来存（所以才会出现区间起始点+8h才能入库，转换要回拨8h），
而在linux中，会直接把时间当作utc时间保存；对于显示时，win下会把数据+8h展示，在linux中会直接展示。

这就导致，看起来输入的是一样，显示也一样，然而底层存储有偏差，就表现为区间端点上的不同反馈。
首尾各收缩一天后，问题解决。

## 解决方案

两种方案：

* 改程序，判断取到的数据是否合法，然后转换一下。为规避时区问题，首尾端点各收缩一天。

```java
//		String oriTimeStr = "0001-01-01 00:00:00";
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		boolean l;
		try {
			Date oriDate = dateFormat.parse(oriTimeStr);
			Date minDate = dateFormat.parse("1970-01-02 00:00:01");
			Date maxDate = dateFormat.parse("2038-01-18 03:14:07");
			if ((l = oriDate.before(minDate)) || !(l = oriDate.before(maxDate))) {
				return dateFormat.format(l ? minDate : maxDate);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return oriTimeStr;
```

* 通知对方改数据，不推荐，无实际意义的数据完全可以自行处理，改成空串可能导致[查询出错](../../Oracle/Tips/ORA-01722：无效数字.MD)。
