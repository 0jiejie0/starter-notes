# MyBatis插入数据，返回新增数据的ID

    mybatis、springboot2、MySQL、xml

其实有一些低代码平台生成的代码里本身就有这部分内容，不过不一定实时都能有好用的工具可以用，这是先前的一个疑问点，特此记录以备它用。

## 自增主键，插入数据，获取自增主键值

mybatis为我们提供了一个方法，能够插入数据时获取自动生成的值，并且把取的值赋值给实体类的某一属性。要求主键必须自增。

```xml
<insert id = "insert" useGeneratedKeys = "true" keyProperty = "id">
    insert ...
</insert>
```

* useGeneratedKeys = true　　//是否返回自增主键值
* keyProperty = “xxx”　　//将值赋给哪个属性，这个属性是方法参数中的,通过插入对象entity.getId()可以得到

## 主键非自增，获取主键值

selectKey after示例：查询最后一次添加的主键：

```xml
<insert id = "insertEmp">
    <selectKey resultType = "integer" order = "AFTER" keyProperty = "eid" >
        select last_insert_id()    //查询最后一次添加的主键,mysql函数
    </selectKey>
    insert into dept(id,deptname) values(#{id},#{deptname})
</insert>
```

selectKey before示例：假设Id不是自增长，我们希望在insert之前获取mysql的UUID添加到数据表作为主键Id：

```xml
<insert id = "insertDept">
    <selectKey resultType = "string" order = "BEFORE" keyProperty = "id">
        select uuid() as id
    </selectKey>
    insert into dept(id,name) values(#{id},#{name})
</insert>

```

selectKey属性说明:

* resultType：返回类型，必填
* order：两个值，必填：1、BEFORE 在添加之前查询　2、AFTER 在添加之后查询　　//这两个都是全大写
* keyProperty：将取值赋值给方法参数，实体类作参数，会赋值给实体对象的这个属性，必填
* keyColumn：对应表的列名

## 插入数据时，返回多个值

赋值规则：keyProperty和keColumn的列数相对应：1对1，2对2 …

```xml
<insert id="xxx">
    <selectKey resultType="com.entity.Dept" order="BEFORE" keyProperty="id,name" keyColumn="cid,cname">
        select cid,cname from Category limit 1;
    </selectKey>
    insert into dept(id,name) values (#{id},#{name})
</insert>
<!-- 从Category中查询出cid，cname 赋值在方法参数中，然后添加到dept表 -->
```

感觉这里，好像是 1查多值填进入参对象，2对象值再次入库？没有验证能不能行，用到了再来验证修正吧。

本文参考[CSDN文章](https://blog.csdn.net/web13618542420/article/details/126112961)
