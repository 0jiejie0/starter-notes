# 若依框架多次list查询时,分页失效

    若依、springboot、mybatis

## 背景

做大屏页面列表分页，从别处摘来分页的代码，那块明明没问题，我改完却发现不能分页，始终是第1页的状态。

## 问题描述

从正常分页的代码处摘得分页代码，调用业务层:

```java
controller(ope){
        startPage();
        List<vo> list=oService.get(ope);
        return getDataTable(list);
        }
```

业务层使用两次查询，第一次使用查询条件获得结果对象的id列表，第二次使用stream流获取结果对象列表并组装返回：
（解释一下为什么这么写，因为实体类有字典名称等信息需要二次查询补充，业务层已经有方法实现单个对象的信息补充了，我不想再复制重复代码了，于是想查询出来后流处理下列表）

```java
service(ope){
        return oMapper.select(ope).stream() // id列表
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

然而这种改良方式只适合最后的list对象是从数据库中查询得到的情况，我这是使用stream流产生的，不可能支持那种分页了。

突然想到，别的也有查询之后二次处理（拼接补充信息）的分页，查看代码，发现那些地方最终是返回了初次查询得到的list对象，中间有foreach处理逻辑。

那么，可以借鉴，dao层查询的列表可以使用实体对象来保存id，然后想办法使用循环，依次查询，替换其中的实体对象或者赋值某属性，最终返回这个list就行了

## 解决方案

1. dao层调整返回类型，由long列表改为对象列表；

2. 业务层：

```java
service(ope){
        List<OVo> oVos=oMapper.select(ope); // dao类型由id列表改为对象列表，其实质还是id列表
        oVos.replaceAll(oVo->getVoById(oVo.getId())); // 根据id获取对象，替换列表元素
        return oVos;
        }
```

这idea的md插件，好像有点bug，格式化代码上有点问题。
