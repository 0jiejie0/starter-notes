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

## 解决方案

两种方案：

* 改程序，判断取到的数据是否合法，然后转换一下。

```java
//		String oriTimeStr = "0001-01-01 00:00:00";
		SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		boolean l;
		try {
			Date oriDate = dateFormat.parse(oriTimeStr);
			Date minDate = dateFormat.parse("1000-01-01 00:00:00");
			Date maxDate = dateFormat.parse("9999-12-31 23:59:59");
			if ((l = oriDate.compareTo(minDate) < 0) || !(l = oriDate.compareTo(maxDate) < 0)) {
				return dateFormat.format(l ? minDate : maxDate);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return oriTimeStr;
```

* 通知对方改数据。
