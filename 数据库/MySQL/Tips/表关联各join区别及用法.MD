> 什么left right的老是忘，归根到底还是不熟练，记录下来，以后就不用再搜了。
> 来源：ai助手

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

返回表A中员工的ID和姓名，以及他们所在的部门名称，只返回满足连接条件的记录。

## Left Join 左连接

返回左表中所有的记录，以及与右表中满足连接条件的记录。如果右表中没有匹配的记录，则返回NULL值。

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         LEFT JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

返回表A中员工的ID和姓名，以及他们所在的部门名称，如果有员工所在的部门在表B中没有对应的部门记录，那么部门名称将显示为NULL值。

## Right Join 右连接

返回右表中所有的记录，以及与左表中满足连接条件的记录。如果左表中没有匹配的记录，则返回NULL值。

```sql
SELECT A.ID, A.Name, B.DepartmentName
FROM TableA A
         RIGHT JOIN TableB B ON A.DepartmentID = B.DepartmentID;
```

返回表B中的部门ID和部门名称，以及该部门下的员工信息，如果有部门在表A中没有员工记录，那么员工ID和姓名将显示为NULL值。
