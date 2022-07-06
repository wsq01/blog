---
title: MySQL多表查询
date: 2020-04-26 11:48:12
tags: [MySQL]
categories: [MySQL]
---

在关系型数据库中，表与表之间是有联系的，所以在实际应用中，经常使用多表查询。多表查询就是同时查询两个或两个以上的表。

在 MySQL 中，多表查询主要有交叉连接、内连接和外连接。
# CROSS JOIN：交叉连接
交叉连接（`CROSS JOIN`）一般用来返回连接表的笛卡尔积。

交叉连接的语法格式：
```
SELECT <字段名> FROM <表1> CROSS JOIN <表2> [WHERE子句]
或
SELECT <字段名> FROM <表1>, <表2> [WHERE子句] 
```
语法说明：
* 字段名：需要查询的字段名称。
* <表1><表2>：需要交叉连接的表名。
* `WHERE`子句：用来设置交叉连接的查询条件。

> 注意：多个表交叉连接时，在`FROM`后连续使用`CROSS JOIN`或`,`即可。

当连接的表之间没有关系时，我们会省略掉`WHERE`子句，这时返回结果就是两个表的笛卡尔积，返回结果数量就是两个表的数据行相乘。需要注意的是，如果每个表有 1000 行，那么返回结果的数量就有 1000×1000 = 1000000 行，数据量是非常巨大的。

交叉连接可以查询两个或两个以上的表。

查询学生信息表和科目信息表，并得到一个笛卡尔积。
```sql
mysql> SELECT * FROM tb_students_info;
+----+--------+------+------+--------+-----------+
| id | name   | age  | sex  | height | course_id |
+----+--------+------+------+--------+-----------+
|  1 | Dany   |   25 | 男   |    160 |         1 |
|  2 | Green  |   23 | 男   |    158 |         2 |
|  3 | Henry  |   23 | 女   |    185 |         1 |
|  4 | Jane   |   22 | 男   |    162 |         3 |
|  5 | Jim    |   24 | 女   |    175 |         2 |
|  6 | John   |   21 | 女   |    172 |         4 |
|  7 | Lily   |   22 | 男   |    165 |         4 |
|  8 | Susan  |   23 | 男   |    170 |         5 |
|  9 | Thomas |   22 | 女   |    178 |         5 |
| 10 | Tom    |   23 | 女   |    165 |         5 |
+----+--------+------+------+--------+-----------+
10 rows in set (0.00 sec)

mysql> SELECT * FROM tb_course;
+----+-------------+
| id | course_name |
+----+-------------+
|  1 | Java        |
|  2 | MySQL       |
|  3 | Python      |
|  4 | Go          |
|  5 | C++         |
+----+-------------+
5 rows in set (0.00 sec)

mysql> SELECT * FROM tb_course CROSS JOIN tb_students_info;
+----+-------------+----+--------+------+------+--------+-----------+
| id | course_name | id | name   | age  | sex  | height | course_id |
+----+-------------+----+--------+------+------+--------+-----------+
|  1 | Java        |  1 | Dany   |   25 | 男   |    160 |         1 |
|  2 | MySQL       |  1 | Dany   |   25 | 男   |    160 |         1 |
|  3 | Python      |  1 | Dany   |   25 | 男   |    160 |         1 |
|  4 | Go          |  1 | Dany   |   25 | 男   |    160 |         1 |
|  5 | C++         |  1 | Dany   |   25 | 男   |    160 |         1 |
|  1 | Java        |  2 | Green  |   23 | 男   |    158 |         2 |
|  2 | MySQL       |  2 | Green  |   23 | 男   |    158 |         2 |
|  3 | Python      |  2 | Green  |   23 | 男   |    158 |         2 |
|  4 | Go          |  2 | Green  |   23 | 男   |    158 |         2 |
|  5 | C++         |  2 | Green  |   23 | 男   |    158 |         2 |
|  1 | Java        |  3 | Henry  |   23 | 女   |    185 |         1 |
|  2 | MySQL       |  3 | Henry  |   23 | 女   |    185 |         1 |
|  3 | Python      |  3 | Henry  |   23 | 女   |    185 |         1 |
|  4 | Go          |  3 | Henry  |   23 | 女   |    185 |         1 |
|  5 | C++         |  3 | Henry  |   23 | 女   |    185 |         1 |
|  1 | Java        |  4 | Jane   |   22 | 男   |    162 |         3 |
|  2 | MySQL       |  4 | Jane   |   22 | 男   |    162 |         3 |
|  3 | Python      |  4 | Jane   |   22 | 男   |    162 |         3 |
|  4 | Go          |  4 | Jane   |   22 | 男   |    162 |         3 |
|  5 | C++         |  4 | Jane   |   22 | 男   |    162 |         3 |
|  1 | Java        |  5 | Jim    |   24 | 女   |    175 |         2 |
|  2 | MySQL       |  5 | Jim    |   24 | 女   |    175 |         2 |
|  3 | Python      |  5 | Jim    |   24 | 女   |    175 |         2 |
|  4 | Go          |  5 | Jim    |   24 | 女   |    175 |         2 |
|  5 | C++         |  5 | Jim    |   24 | 女   |    175 |         2 |
|  1 | Java        |  6 | John   |   21 | 女   |    172 |         4 |
|  2 | MySQL       |  6 | John   |   21 | 女   |    172 |         4 |
|  3 | Python      |  6 | John   |   21 | 女   |    172 |         4 |
|  4 | Go          |  6 | John   |   21 | 女   |    172 |         4 |
|  5 | C++         |  6 | John   |   21 | 女   |    172 |         4 |
|  1 | Java        |  7 | Lily   |   22 | 男   |    165 |         4 |
|  2 | MySQL       |  7 | Lily   |   22 | 男   |    165 |         4 |
|  3 | Python      |  7 | Lily   |   22 | 男   |    165 |         4 |
|  4 | Go          |  7 | Lily   |   22 | 男   |    165 |         4 |
|  5 | C++         |  7 | Lily   |   22 | 男   |    165 |         4 |
|  1 | Java        |  8 | Susan  |   23 | 男   |    170 |         5 |
|  2 | MySQL       |  8 | Susan  |   23 | 男   |    170 |         5 |
|  3 | Python      |  8 | Susan  |   23 | 男   |    170 |         5 |
|  4 | Go          |  8 | Susan  |   23 | 男   |    170 |         5 |
|  5 | C++         |  8 | Susan  |   23 | 男   |    170 |         5 |
|  1 | Java        |  9 | Thomas |   22 | 女   |    178 |         5 |
|  2 | MySQL       |  9 | Thomas |   22 | 女   |    178 |         5 |
|  3 | Python      |  9 | Thomas |   22 | 女   |    178 |         5 |
|  4 | Go          |  9 | Thomas |   22 | 女   |    178 |         5 |
|  5 | C++         |  9 | Thomas |   22 | 女   |    178 |         5 |
|  1 | Java        | 10 | Tom    |   23 | 女   |    165 |         5 |
|  2 | MySQL       | 10 | Tom    |   23 | 女   |    165 |         5 |
|  3 | Python      | 10 | Tom    |   23 | 女   |    165 |         5 |
|  4 | Go          | 10 | Tom    |   23 | 女   |    165 |         5 |
|  5 | C++         | 10 | Tom    |   23 | 女   |    165 |         5 |
+----+-------------+----+--------+------+------+--------+-----------+
50 rows in set (0.00 sec)
```
由运行结果可以看出，`tb_course`和`tb_students_info`表交叉连接查询后，返回了 50 条记录。可以想象，当表中的数据较多时，得到的运行结果会非常长，而且得到的运行结果也没太大的意义。所以，通过交叉连接的方式进行多表查询的这种方法并不常用，我们应该尽量避免这种查询。

查询`tb_course`表中的`id`字段和`tb_students_info`表中的`course_id`字段相等的内容：
```sql
mysql> SELECT * FROM tb_course CROSS JOIN tb_students_info 
    -> WHERE tb_students_info.course_id = tb_course.id;
+----+-------------+----+--------+------+------+--------+-----------+
| id | course_name | id | name   | age  | sex  | height | course_id |
+----+-------------+----+--------+------+------+--------+-----------+
|  1 | Java        |  1 | Dany   |   25 | 男   |    160 |         1 |
|  2 | MySQL       |  2 | Green  |   23 | 男   |    158 |         2 |
|  1 | Java        |  3 | Henry  |   23 | 女   |    185 |         1 |
|  3 | Python      |  4 | Jane   |   22 | 男   |    162 |         3 |
|  2 | MySQL       |  5 | Jim    |   24 | 女   |    175 |         2 |
|  4 | Go          |  6 | John   |   21 | 女   |    172 |         4 |
|  4 | Go          |  7 | Lily   |   22 | 男   |    165 |         4 |
|  5 | C++         |  8 | Susan  |   23 | 男   |    170 |         5 |
|  5 | C++         |  9 | Thomas |   22 | 女   |    178 |         5 |
|  5 | C++         | 10 | Tom    |   23 | 女   |    165 |         5 |
+----+-------------+----+--------+------+------+--------+-----------+
10 rows in set (0.01 sec)
```
如果在交叉连接时使用`WHERE`子句，MySQL 会先生成两个表的笛卡尔积，然后再选择满足`WHERE`条件的记录。因此，表的数量较多时，交叉连接会非常非常慢。一般情况下不建议使用交叉连接。

在 MySQL 中，多表查询一般使用内连接和外连接，它们的效率要高于交叉连接。
## 笛卡尔积
笛卡尔积是指两个集合`X`和`Y`的乘积。

例如，有`A`和`B`两个集合，它们的值如下：
```
A = {1,2}
B = {3,4,5}
```
集合`A×B 和 B×A`的结果集分别表示为：
```
A×B={(1,3), (1,4), (1,5), (2,3), (2,4), (2,5) };
B×A={(3,1), (3,2), (4,1), (4,2), (5,1), (5,2) };
```
以上`A×B`和`B×A`的结果就叫做两个集合的笛卡尔积。并且，从以上结果我们可以看出：
* 两个集合相乘，不满足交换率，即`A×B≠B×A`。
* `A`集合和`B`集合的笛卡尔积是`A`集合的元素个数 × `B`集合的元素个数。

多表查询遵循的算法就是以上提到的笛卡尔积，表与表之间的连接可以看成是在做乘法运算。在实际应用中，应避免使用笛卡尔积，因为笛卡尔积中容易存在大量的不合理数据，简单来说就是容易导致查询结果重复、混乱。
# INNER JOIN：内连接
内连接主要通过设置连接条件的方式，来移除查询结果中某些数据行的交叉连接。简单来说，就是利用条件表达式来消除交叉连接的某些数据行。

内连接使用`INNER JOIN`关键字连接两张表，并使用`ON`子句来设置连接条件。如果没有连接条件，`INNER JOIN`和`CROSS JOIN`在语法上是等同的，两者可以互换。

内连接的语法格式如下：
```sql
SELECT <字段名> FROM <表1> INNER JOIN <表2> [ON子句]
```
语法说明如下。
* 字段名：需要查询的字段名称。
* <表1><表2>：需要内连接的表名。
* `INNER JOIN`：内连接中可以省略`INNER`关键字，只用关键字`JOIN`。
* `ON`子句：用来设置内连接的连接条件。

`INNER JOIN`也可以使用`WHERE`子句指定连接条件，但是`INNER JOIN ... ON`语法是官方的标准写法，而且`WHERE`子句在某些时候会影响查询的性能。

多个表内连接时，在`FROM`后连续使用`INNER JOIN`或`JOIN`即可。

在`tb_students_info`表和`tb_course`表之间，使用内连接查询学生姓名和相对应的课程名称，SQL 语句和运行结果如下。
```sql
mysql> SELECT s.name,c.course_name FROM tb_students_info s INNER JOIN tb_course c 
    -> ON s.course_id = c.id;
+--------+-------------+
| name   | course_name |
+--------+-------------+
| Dany   | Java        |
| Green  | MySQL       |
| Henry  | Java        |
| Jane   | Python      |
| Jim    | MySQL       |
| John   | Go          |
| Lily   | Go          |
| Susan  | C++         |
| Thomas | C++         |
| Tom    | C++         |
+--------+-------------+
10 rows in set (0.00 sec)
```
在这里的查询语句中，两个表之间的关系通过`INNER JOIN`指定，连接的条件使用`ON`子句给出。
 
注意：当对多个表进行查询时，要在`SELECT`语句后面指定字段是来源于哪一张表。因此，在多表查询时，`SELECT`语句后面的写法是表名.列名。另外，如果表名非常长的话，也可以给表设置别名，这样就可以直接在`SELECT`语句后面写上表的别名.列名。
# LEFT/RIGHT JOIN：外连接
内连接的查询结果都是符合连接条件的记录，而外连接会先将连接的表分为基表和参考表，再以基表为依据返回满足和不满足条件的记录。

外连接可以分为左外连接和右外连接。
## 左连接
左外连接又称为左连接，使用`LEFT OUTER JOIN`关键字连接两个表，并使用`ON`子句来设置连接条件。

左连接的语法格式如下：
```sql
SELECT <字段名> FROM <表1> LEFT OUTER JOIN <表2> <ON子句>
```
语法说明如下。
* 字段名：需要查询的字段名称。
* <表1><表2>：需要左连接的表名。
* `LEFT OUTER JOIN`：左连接中可以省略`OUTER`关键字，只使用关键字`LEFT JOIN`。
* `ON`子句：用来设置左连接的连接条件，不能省略。

上述语法中，“表1”为基表，“表2”为参考表。左连接查询时，可以查询出“表1”中的所有记录和“表2”中匹配连接条件的记录。如果“表1”的某行在“表2”中没有匹配行，那么在返回结果中，“表2”的字段值均为空值（`NULL`）。

在进行左连接查询之前，我们先查看 tb_course 和 tb_students_info 两张表中的数据。SQL 语句和运行结果如下。
```sql
mysql> SELECT * FROM tb_course;
+----+-------------+
| id | course_name |
+----+-------------+
|  1 | Java        |
|  2 | MySQL       |
|  3 | Python      |
|  4 | Go          |
|  5 | C++         |
|  6 | HTML        |
+----+-------------+
6 rows in set (0.00 sec)

mysql> SELECT * FROM tb_students_info;
+----+--------+------+------+--------+-----------+
| id | name   | age  | sex  | height | course_id |
+----+--------+------+------+--------+-----------+
|  1 | Dany   |   25 | 男   |    160 |         1 |
|  2 | Green  |   23 | 男   |    158 |         2 |
|  3 | Henry  |   23 | 女   |    185 |         1 |
|  4 | Jane   |   22 | 男   |    162 |         3 |
|  5 | Jim    |   24 | 女   |    175 |         2 |
|  6 | John   |   21 | 女   |    172 |         4 |
|  7 | Lily   |   22 | 男   |    165 |         4 |
|  8 | Susan  |   23 | 男   |    170 |         5 |
|  9 | Thomas |   22 | 女   |    178 |         5 |
| 10 | Tom    |   23 | 女   |    165 |         5 |
| 11 | LiMing |   22 | 男   |    180 |         7 |
+----+--------+------+------+--------+-----------+
11 rows in set (0.00 sec)

mysql> SELECT s.name,c.course_name FROM tb_students_info s LEFT OUTER JOIN tb_course c 
    -> ON s.`course_id`=c.`id`;
+--------+-------------+
| name   | course_name |
+--------+-------------+
| Dany   | Java        |
| Henry  | Java        |
| NULL   | Java        |
| Green  | MySQL       |
| Jim    | MySQL       |
| Jane   | Python      |
| John   | Go          |
| Lily   | Go          |
| Susan  | C++         |
| Thomas | C++         |
| Tom    | C++         |
| LiMing | NULL        |
+--------+-------------+
12 rows in set (0.00 sec)
```
可以看到，运行结果显示了 12 条记录，`name`为`LiMing`的学生目前没有课程，因为对应的`tb_course`表中没有该学生的课程信息，所以该条记录只取出了`tb_students_info`表中相应的值，而从`tb_course`表中取出的值为`NULL`。
## 右连接
右外连接又称为右连接，右连接是左连接的反向连接。使用`RIGHT OUTER JOIN`关键字连接两个表，并使用`ON`子句来设置连接条件。

右连接的语法格式如下：
```sql
SELECT <字段名> FROM <表1> RIGHT OUTER JOIN <表2> <ON子句>
```
语法说明如下。
* 字段名：需要查询的字段名称。
* <表1><表2>：需要右连接的表名。
* `RIGHT OUTER JOIN`：右连接中可以省略`OUTER`关键字，只使用关键字`RIGHT JOIN`。
* `ON`子句：用来设置右连接的连接条件，不能省略。

与左连接相反，右连接以“表2”为基表，“表1”为参考表。右连接查询时，可以查询出“表2”中的所有记录和“表1”中匹配连接条件的记录。如果“表2”的某行在“表1”中没有匹配行，那么在返回结果中，“表1”的字段值均为空值（`NULL`）。
```sql
mysql> SELECT s.name,c.course_name FROM tb_students_info s RIGHT OUTER JOIN tb_course c 
    -> ON s.`course_id`=c.`id`;
+--------+-------------+
| name   | course_name |
+--------+-------------+
| Dany   | Java        |
| Green  | MySQL       |
| Henry  | Java        |
| Jane   | Python      |
| Jim    | MySQL       |
| John   | Go          |
| Lily   | Go          |
| Susan  | C++         |
| Thomas | C++         |
| Tom    | C++         |
| NULL   | HTML        |
+--------+-------------+
11 rows in set (0.00 sec)
```
可以看到，结果显示了 11 条记录，名称为`HTML`的课程目前没有学生，因为对应的`tb_students_info`表中并没有该学生的信息，所以该条记录只取出了`tb_course`表中相应的值，而从`tb_students_info`表中取出的值为`NULL`。

多个表左/右连接时，在`ON`子句后连续使用`LEFT/RIGHT OUTER JOIN`或`LEFT/RIGHT JOIN`即可。

使用外连接查询时，一定要分清需要查询的结果，是需要显示左表的全部记录还是右表的全部记录，然后选择相应的左连接和右连接。
# 子查询
子查询是 MySQL 中比较常用的查询方法，通过子查询可以实现多表查询。子查询指将一个查询语句嵌套在另一个查询语句中。子查询可以在`SELECT、UPDATE`和`DELETE`语句中使用，而且可以进行多层嵌套。在实际开发时，子查询经常出现在 WHERE 子句中。

子查询在`WHERE`中的语法格式如下：
```sql
WHERE <表达式> <操作符> (子查询)
```
其中，操作符可以是比较运算符和`IN、NOT IN、EXISTS、NOT EXISTS`等关键字。
1. `IN | NOT IN`
当表达式与子查询返回的结果集中的某个值相等时，返回 TRUE，否则返回`FALSE`；若使用关键字`NOT`，则返回值正好相反。
2. `EXISTS | NOT EXISTS`
用于判断子查询的结果集是否为空，若子查询的结果集不为空，返回 TRUE，否则返回 FALSE；若使用关键字 NOT，则返回的值正好相反。

使用子查询在`tb_students_info`表和`tb_course`表中查询学习 Java 课程的学生姓名，SQL 语句和运行结果如下。
```sql
mysql> SELECT name FROM tb_students_info 
    -> WHERE course_id IN (SELECT id FROM tb_course WHERE course_name = 'Java');
+-------+
| name  |
+-------+
| Dany  |
| Henry |
+-------+
```
上述查询过程也可以分为以下 2 步执行，实现效果是相同的。

1）首先单独执行内查询，查询出 tb_course 表中课程为 Java 的 id，SQL 语句和运行结果如下。
```sql
mysql> SELECT id FROM tb_course WHERE course_name = 'Java';
+----+
| id |
+----+
|  1 |
+----+
```
2）然后执行外层查询，在 tb_students_info 表中查询 course_id 等于 1 的学生姓名。SQL 语句和运行结果如下。
```sql
mysql> SELECT name FROM tb_students_info WHERE course_id IN (1);
+-------+
| name  |
+-------+
| Dany  |
| Henry |
+-------+
```
习惯上，外层的 SELECT 查询称为父查询，圆括号中嵌入的查询称为子查询（子查询必须放在圆括号内）。MySQL 在处理上例的 SELECT 语句时，执行流程为：先执行子查询，再执行父查询。

在 SELECT 语句中使用 NOT IN 关键字，查询没有学习 Java 课程的学生姓名，SQL 语句和运行结果如下。
```sql
mysql> SELECT name FROM tb_students_info 
    -> WHERE course_id NOT IN (SELECT id FROM tb_course WHERE course_name = 'Java');
+--------+
| name   |
+--------+
| Green  |
| Jane   |
| Jim    |
| John   |
| Lily   |
| Susan  |
| Thomas |
| Tom    |
| LiMing |
+--------+
9 rows in set (0.01 sec)
```
可以看出，运行结果与例 1 刚好相反，没有学习 Java 课程的是除了 Dany 和 Henry 之外的学生。

使用=运算符，在 tb_course 表和 tb_students_info 表中查询出所有学习 Python 课程的学生姓名，SQL 语句和运行结果如下。
```sql
mysql> SELECT name FROM tb_students_info
    -> WHERE course_id = (SELECT id FROM tb_course WHERE course_name = 'Python');
+------+
| name |
+------+
| Jane |
+------+
1 row in set (0.00 sec)
```
使用<>运算符，在 tb_course 表和 tb_students_info 表中查询出没有学习 Python 课程的学生姓名，SQL 语句和运行结果如下。
```sql
mysql> SELECT name FROM tb_students_info
    -> WHERE course_id <> (SELECT id FROM tb_course WHERE course_name = 'Python');
+--------+
| name   |
+--------+
| Dany   |
| Green  |
| Henry  |
| Jim    |
| John   |
| Lily   |
| Susan  |
| Thomas |
| Tom    |
| LiMing |
+--------+
10 rows in set (0.00 sec)
```
可以看出，运行结果与例 3 刚好相反，没有学习 Python 课程的是除了 Jane 之外的学生。

查询 tb_course 表中是否存在 id=1 的课程，如果存在，就查询出 tb_students_info 表中的记录，SQL 语句和运行结果如下。
```sql
mysql> SELECT * FROM tb_students_info
    -> WHERE EXISTS(SELECT course_name FROM tb_course WHERE id=1);
+----+--------+------+------+--------+-----------+
| id | name   | age  | sex  | height | course_id |
+----+--------+------+------+--------+-----------+
|  1 | Dany   |   25 | 男   |    160 |         1 |
|  2 | Green  |   23 | 男   |    158 |         2 |
|  3 | Henry  |   23 | 女   |    185 |         1 |
|  4 | Jane   |   22 | 男   |    162 |         3 |
|  5 | Jim    |   24 | 女   |    175 |         2 |
|  6 | John   |   21 | 女   |    172 |         4 |
|  7 | Lily   |   22 | 男   |    165 |         4 |
|  8 | Susan  |   23 | 男   |    170 |         5 |
|  9 | Thomas |   22 | 女   |    178 |         5 |
| 10 | Tom    |   23 | 女   |    165 |         5 |
| 11 | LiMing |   22 | 男   |    180 |         7 |
+----+--------+------+------+--------+-----------+
11 rows in set (0.01 sec)
```
由结果可以看到，tb_course 表中存在 id=1 的记录，因此 EXISTS 表达式返回 TRUE，外层查询语句接收 TRUE 之后对表 tb_students_info 进行查询，返回所有的记录。

EXISTS 关键字可以和其它查询条件一起使用，条件表达式与 EXISTS 关键字之间用 AND 和 OR 连接。

查询 tb_course 表中是否存在 id=1 的课程，如果存在，就查询出 tb_students_info 表中 age 字段大于 24 的记录，SQL 语句和运行结果如下。
```sql
mysql> SELECT * FROM tb_students_info
    -> WHERE age>24 AND EXISTS(SELECT course_name FROM tb_course WHERE id=1);
+----+------+------+------+--------+-----------+
| id | name | age  | sex  | height | course_id |
+----+------+------+------+--------+-----------+
|  1 | Dany |   25 | 男   |    160 |         1 |
+----+------+------+------+--------+-----------+
1 row in set (0.01 sec)
```
结果显示，从 tb_students_info 表中查询出了一条记录，这条记录的 age 字段取值为 25。内层查询语句从 tb_course 表中查询到记录，返回 TRUE。外层查询语句开始进行查询。根据查询条件，从 tb_students_info 表中查询 age 大于 24 的记录。

子查询的功能也可以通过表连接完成，但是子查询会使 SQL 语句更容易阅读和编写。

一般来说，表连接（内连接和外连接等）都可以用子查询替换，但反过来却不一定，有的子查询不能用表连接来替换。子查询比较灵活、方便、形式多样，适合作为查询的筛选条件，而表连接更适合于查看连接表的数据。
# 子查询注意事项
在完成较复杂的数据查询时，经常会使用到子查询，编写子查询语句时，要注意如下事项。
1) 子查询语句可以嵌套在 SQL 语句中任何表达式出现的位置
在 SELECT 语句中，子查询可以被嵌套在 SELECT 语句的列、表和查询条件中，即 SELECT 子句，FROM 子句、WHERE 子句、GROUP BY 子句和 HAVING 子句。

前面已经介绍了 WHERE 子句中嵌套子查询的使用方法，下面是子查询在 SELECT 子句和 FROM 子句中的使用语法。

嵌套在 SELECT 语句的 SELECT 子句中的子查询语法格式如下。
```sql
SELECT (子查询) FROM 表名;
```
提示：子查询结果为单行单列，但不必指定列别名。

嵌套在 SELECT 语句的 FROM 子句中的子查询语法格式如下。
```sql
SELECT * FROM (子查询) AS 表的别名;
```
注意：必须为表指定别名。一般返回多行多列数据记录，可以当作一张临时表。
2) 只出现在子查询中而没有出现在父查询中的表不能包含在输出列中
多层嵌套子查询的最终数据集只包含父查询（即最外层的查询）的 SELECT 子句中出现的字段，而子查询的输出结果通常会作为其外层子查询数据源或用于数据判断匹配。

常见错误如下：
```sql
SELECT * FROM (SELECT * FROM result);
```
这个子查询语句产生语法错误的原因在于主查询语句的 FROM 子句是一个子查询语句，因此应该为子查询结果集指定别名。正确代码如下。
```sql
SELECT * FROM (SELECT * FROM result) AS Temp;
```
# 子查询改写为表连接
子查询如递归函数一样，有时侯能达到事半功倍的效果，但是其执行效率较低。与表连接相比，子查询比较灵活，方便，形式多样，适合作为查询的筛选条件，而表连接更适合查看多表的数据。

一般情况下，子查询会产生笛卡儿积，表连接的效率要高于子查询。因此在编写 SQL 语句时应尽量使用连接查询。

表连接（内连接和外连接等）都可以用子查询替换，但反过来却不一定，有的子查询不能用表连接来替换。下面我们介绍哪些子查询的查询命令可以改写为表连接。

在检查那些倾向于编写成子查询的查询语句时，可以考虑将子查询替换为表连接，看看连接的效率是不是比子查询更好些。同样，如果某条使用子查询的 SELECT 语句需要花费很长时间才能执行完毕，那么可以尝试把它改写为表连接，看看执行效果是否有所改善。

下面讨论具体该如何做。
1. 改写用来查询匹配值的子查询
下面这条示例语句包含一个子查询，它会把 score 表里的考试成绩查询出来：
```sql
SELECT * FROM score
WHERE grade_id IN (SELECT id FROM grade WHERE category = 'Java');
```
在编写以上语句时，可以不使用子查询，而是把它转换为一个简单的连接：
```sql
SELECT score.* FROM score INNER JOIN grade
ON score.grade_id = grade.id WHERE grade.category = 'Java';
```
再来看另一个示例。下面这条查询语句可以把所有女生的考试成绩查询出来：
```sql
SELECT * from score
WHERE student_id IN (SELECT student_id FROM student WHERE sex = 'F') ;
```
这条语句可以转换为以下连接：
```sql
SELECT score.* FROM score INNER JOIN student
ON score.student_id = student.student_id WHERE student.sex = 'F' ;
```
我们可以发现这些子查询语句都遵从这样一种形式：
```sql
SELECT * FROM table1
WHERE column1 IN (SELECT column2a FROM table2 WHERE column2b = value);
```
其中，column1 代表 table1 中的字段，column2a 和 column2b 代表 table2 表中的字段。这类查询都可以被转换为下面这种形式的连接查询：
```sql
SELECT table1. * FROM table1 INNER JOIN table2
ON table1. column1 = table. column2a WHERE table2. column2b = value;
```
在某些场合，子查询和关联查询可能会返回不同的结果。比如，当 table2 包含 column2a 的多个实例时，就会发生这种情况。这种形式的子查询只会为每个 column2a 值生成一个实例，而连接操作会为所有值生成实例，并且其输出会包含重复行。如果想要防止这种重复记录出现，就要在编写连接查询语句时使用 SELECT DISTINCT，而不能使用 SELECT。
2. 改写用来查询非匹配（缺失）值的子查询
另一种常见的子查询语句类型是：把存在于某个表里，但在另一个表里并不存在的那些值查找出来。“哪些值不存在”有关的问题通常都可以用 LEFT JOIN 来解决。

如下语句用来测试哪些学生没有出现在 absence 表里（用于查找全勤学生）：
```sql
SELECT * FROM student
WHERE student_id NOT IN (SELECT student_id FROM absence) ;
```
以上查询语句可以使用 LEFT JOIN 来改写：
```sql
SELECT student.* FROM student LEFT JOIN absence
ON student.student_id = absence.student_id WHERE absence.student_ id IS NULL;
```
通常情况下，如果子查询语句符合如下所示的形式：
```sql
SELECT * FROM table1
WHERE column1 NOT IN ( SELECT column2 FROM table2) ;
```
那么可以把它改写为下面这样的连接查询：
```sql
SELECT table1.* FROM table1 LEFT JOIN table2
ON table1.column1 = table2.column2 WHERE table2.column2 IS NULL;
```
这里需要假设 table2.column2 被定义成了 NOT NULL 的。

与 LEFT JOIN 相比，子查询更加直观。大部分人都可以毫无困难地理解“没被包含在...里面”的含义，因为它不是数据库编程技术带来的新概念。而“左连接”有所不同，很难用自然语言直观地描述出它的含义。