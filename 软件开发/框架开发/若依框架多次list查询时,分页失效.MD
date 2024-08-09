# 若依框架多次list查询时,分页失效

    若依、springboot、mybatis

## 背景

做大屏页面列表分页，从别处摘来分页的代码，那块明明没问题，我改完却发现不能分页，始终是第1页的状态。

## 问题描述

从正常分页的代码处摘得分页代码，调用业务层:

```java
controller(){
        startPage();
        List<vo> list=oService.get(ope);
        return getDataTable(list);
        }
```

业务层使用两次查询，第一次使用查询条件获得结果对象的id列表，第二次使用stream流获取结果对象列表并组装返回：

```java
service(){
        oMapper.select(ope).stream() // id列表
        .map(this::getVoById).collect(Collectors.toList()) // 根据id获取对象
        }
```

前端使用分页组件调用接口，发现数据总数total始终与每页大小一样，页数始终是1，这不正常。

## 问题分析

直觉告诉我，应该是id二次查询导致的问题，第二次查出来的list里边没有查询条件中的总数和页数，出来的对象自然就不正常了，查看`getDataTable()`
源码，
发现其中设置了结果的total值，是从参数的list对象中获取的，难道，我要手动把业务层返回的list对象设置一下total值吗？

于是我搜索`springboot 若依项目使用id列表二次查询导致pageutil分页失效的解决办法`，
搜到了[这么一篇博客](https://blog.csdn.net/qq_37844454/article/details/133635642)，
里面说多次查询导致控制层创建的page对象消耗了，需要把开启分页的代码放到业务层查询最终list结果的逻辑前。

## 解决方案

控制层:

```java
controller(){
//        startPage();
        List<vo> list=oService.get(ope);
        return getDataTable(list);
        }
```

业务层：

```java
service(){
        Stream<Long> stream=oMapper.select(ope).stream(); // id列表
        PageUtils.startPage(); // 开启分页
        return stream.map(this::getVoById).collect(Collectors.toList()); // 根据id获取对象
        }
```

这idea的md插件，好像有点bug，格式化代码上有点问题。
