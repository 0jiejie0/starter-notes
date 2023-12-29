# MyBatis映射配置文件中，动态传递参数

有两种方式：占位符 #{} 和 拼接符 ${}

## 区别

1. #{} 为参数占位符 ?，即sql 预编译，${} 为字符串替换，即 sql 拼接
2. #{}：动态解析 -> 预编译 -> 执行，${}：动态解析 -> 编译 -> 执行
3. #{} 的变量替换是在DBMS 中，${} 的变量替换是在 DBMS 外
4. 变量替换后，#{} 对应的变量自动加上单引号 ''，变量替换后，${} 对应的变量不会加上单引号 ''
5. #{} 能防止sql 注入，${} 不能防止sql 注入

## 举例(过程演示)

1. 假设传入参数为 1
2.
    * #{}：select * from t_user where uid=#{uid}
    * ${}：select * from t_user where uid= '${uid}'
3.
    * #{}：select * from t_user where uid= ?
    * ${}：select * from t_user where uid= '1'
4.
    * #{}：select * from t_user where uid= '1'
    * ${}：select * from t_user where uid= '1'

## #{} 和 ${} 的大括号中的值

感觉这里写的不太明白，没看懂

1. 单个参数的情形

* #{} 无MyBatis 默认值，可任意，且与参数名无关
* ${}
    1. 使用 MyBatis 默认值 value，即 ${value}
    2. 使用自定义参数名，前提：在映射器接口方法的参数前加注解@Param("")

2. 多个参数的情形
   1）#{}

<1>使用MyBatis 默认值 arg0、arg1、arg2 … 或 param1、param2、param3 …
<2>使用自定义参数名，前提：在映射器接口方法的参数前加注解@Param("")
2）${}
<1>使用MyBatis 默认值 arg0、arg1、arg2 … 或 param1、param2、param3 …
<2>使用自定义参数名，前提：在映射器接口方法的参数前加注解@Param("")
注：@Param("") 是 @Param(value="") 的简写

## #{} 和 ${} 在使用中的技巧和建议

1. 不论是单个参数，还是多个参数，都建议使用注解@Param("")
2. 能用 #{} 的地方就用 #{}，不用或少用 ${}
3. 表名作参数时，必须用 ${}。如：select * from ${tableName}
4. order by 时，必须用 ${}。如：select * from t_user order by ${columnName}
5. 使用 ${} 时，要注意何时加或不加单引号，即 ${} 和 '${}'

本文参考[csdn文章](https://blog.csdn.net/siwuxie095/article/details/79190856)
