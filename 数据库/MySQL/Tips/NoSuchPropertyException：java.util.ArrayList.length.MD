## 背景

开发过程中老系统报错，之前没出现过这个问题。

    mybatis、springboot、idea

## 问题描述

错误信息：

```log
### Error querying database.  Cause: org.apache.ibatis.builder.BuilderException: Error evaluating expression 'aList.length > 1'. Cause: org.apache.ibatis.ognl.NoSuchPropertyException: java.util.ArrayList.length
### Cause: org.apache.ibatis.builder.BuilderException: Error evaluating expression 'aList.length > 1'. Cause: org.apache.ibatis.ognl.NoSuchPropertyException: java.util.ArrayList.length
```

问题代码：

```xml

<if test="aList.length > 1">
    <foreach collection="aList" open="" close="" separator="" item="item" index="index">
        <if test="index > 0">
            inner join (
            select distinct id
            from t_extend
            where status=1
            and code=#{item}
            ) t${index} using(id)
        </if>
    </foreach>
</if>
```

## 分析

经验浅薄的我没有发现问题，以前没报问题这里应该没事才对，都是老代码没动过。
求助ai助手，告知我长度获取应该用size方法，我恍然大悟。

原因算是找到了，aList这个字段在Java代码里原来填的是String数组，我把这段代码类型改成List了，所以才产生了这个问题。

## 解决方案

用 aList.size() 替换 aList.length：

```xml

<if test="aList.size() > 1">
    <foreach collection="aList" open="" close="" separator="" item="item" index="index">
        <if test="index > 0">
            inner join (
            select distinct id
            from t_extend
            where status=1
            and code=#{item}
            ) t${index} using(id)
        </if>
    </foreach>
</if>
```
