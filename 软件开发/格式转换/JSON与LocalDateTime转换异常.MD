# JSON与LocalDateTime转换异常

    json、LocalDateTime、springboot2、ruoyi-vue

## 背景

在已有代码基础上做接口开发，使用apipost保存信息时，爆出了问题。不明白为什么留到现在没处理，难道别人没出错？（想起来了，这模块没做好，没测过，那还真是没出错）

## 问题描述

bean属性定义如此(其中 LocalDateTime 或为 LocalDate)：

```java
@Excel(name = "出生日期", width = 30, dateFormat = "yyyy-MM-dd HH:mm:ss")
@Schema(description = "出生日期")
private LocalDateTime birthday;
```

使用apipost发送put请求，其中某属性数据写法如此：`"birthday": "2021-11-13 11:46:28"`；

于是在返回到apipost的结果中，就包含了类似这样的错误：
> Invalid JSON input: Cannot deserialize value of type `java.time.LocalDateTime` from String "2020-05-04 00:00": Failed
> to deserialize java.time.LocalDateTime: (java.time.format.DateTimeParseException) Text '2020-05-04 00:00' could not be
> parsed at index 10; nested exception is com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize
> value of type `java.time.LocalDateTime` from String "2020-05-04 00:00": Failed to deserialize java.time.LocalDateTime: (
> java.time.format.DateTimeParseException) Text '2020-05-04 00:00' could not be parsed at index 10

## 原因分析

这个日期时间转换真是叫人头疼，我实在是不想研究这个玩意，太没营养了，难受，真难受。

好在问了问ai，给出了方案：

## 解决方案

在javabean定义的属性上加个注解：

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
```

最终效果如下：

```java
@Excel(name = "出生日期", width = 30, dateFormat = "yyyy-MM-dd HH:mm:ss")
@Schema(description = "出生日期")
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime birthday;
```
