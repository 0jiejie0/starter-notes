# 数据库表的关联、集合操作

    join、MySQL、Oracle

> 什么left right的老是忘，归根到底还是不熟练，记录下来，以后就不用再搜了。

设表A、表B分别包含员工信息和部门信息。

表A包含员工的ID、姓名和部门ID，
表B包含部门ID和部门名称。

## (Inner) Join 内连接

返回两个表中满足连接条件的记录。如果两个表中没有匹配的记录，则不会返回任何结果。
inner join 和 join 等价。

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         INNER JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

在两个字段同名的情况下，这个sql可以使用using进一步简化：

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         INNER JOIN TableB B USING (DepartmentID);
```

若两表只有一对同名字段，则可以进一步简化(使用自然连接)：

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         NATURAL JOIN TableB B;
```

返回表A中员工的ID和姓名，以及他们所在的部门名称，只返回满足连接条件的记录。

## Left Join 左连接

等效 `left outer join` ,`outer`可省略

返回左表中所有的记录，以及与右表中满足连接条件的记录。如果右表中没有匹配的记录，则返回NULL值。

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         LEFT JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

返回表A中员工的ID和姓名，以及他们所在的部门名称，如果有员工所在的部门在表B中没有对应的部门记录，那么部门名称将显示为NULL值。

## Right Join 右连接

等效 `right outer join` ,`outer`可省略

返回右表中所有的记录，以及与左表中满足连接条件的记录。如果左表中没有匹配的记录，则返回NULL值。

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         RIGHT JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

返回表B中的部门ID和部门名称，以及该部门下的员工信息，如果有部门在表A中没有员工记录，那么员工ID和姓名将显示为NULL值。

## 全外连接

在 Oracle 中可以使用`Full Outer Join`，等效`full join`,`outer`可省略：

```SQL
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         FULL OUTER JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

返回所有员工和所有部门信息，员工无部门其部门显示NULL，部门无员工其员工显示NULL

### 解决MySQL中不支持全外连接的办法：

将左外连接和右外连接中间利用`union`取并集（去重，执行效率低）：

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         LEFT JOIN TableB B ON A.DepartmentID = B.DepartmentID
UNION 
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         RIGHT JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

或可利用`union all`取并集（不去重，执行效率高）：

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         LEFT JOIN TableB B ON A.DepartmentID = B.DepartmentID
UNION ALL
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         RIGHT JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

## 集合操作

MySQL只支持并集，交、差不支持；

Oracle支持并、交、差。

## 三表查询

## 自连接查询

以上是99语法

## 92语法

### 交叉连接

两表中间使用逗号分隔，表示两表笛卡尔积，然后利用where条件筛选。

1. 92语法更简单但使用繁琐，无特性关键词（个人感觉可以理解为一种语法糖）
2. 92语法的 表的连接条件 和 筛选条件 没有分开
3. 99语法提供了更多查询连接类型，使用和表达上更为简洁，但语法体系更为冗杂
