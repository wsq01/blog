---
title: MySQL 操作表中的数据
date: 2020-04-25 17:16:51
tags: [MySQL]
categories: [MySQL]
---


# INSERT：插入数据
数据库与表创建成功以后，需要向数据库的表中插入数据。在 MySQL 中可以使用`INSERT`语句向数据库已有的表中插入一行或者多行元组数据。
## 基本语法
`INSERT`语句有两种语法形式，分别是`INSERT…VALUES`语句和`INSERT…SET`语句。
#### INSERT…VALUES语句
```sql
INSERT INTO <表名> [ <列名1> [ , … <列名n>] ]
VALUES (值1) [… , (值n) ];
```
语法说明如下。
* <表名>：指定被操作的表名。
* <列名>：指定需要插入数据的列名。若向表中的所有列插入数据，则全部的列名均可以省略，直接采用`INSERT<表名>VALUES(…)`即可。
* `VALUES`或`VALUE`子句：该子句包含要插入的数据清单。数据清单中数据的顺序要和列的顺序相对应。

#### INSERT…SET语句
语法格式为：
```sql
INSERT INTO <表名>
SET <列名1> = <值1>,
    <列名2> = <值2>,
        …
```
此语句用于直接给表中的某些列指定对应的列值，即要插入的数据的列名在`SET`子句中指定，col_name 为指定的列名，等号后面为指定的数据，而对于未指定的列，列值会指定为该列的默认值。

由 INSERT 语句的两种形式可以看出：
使用 INSERT…VALUES 语句可以向表中插入一行数据，也可以插入多行数据；
使用 INSERT…SET 语句可以指定插入行中每列的值，也可以指定部分列的值；
INSERT…SELECT 语句向表中插入其他表的数据。
采用 INSERT…SET 语句可以向表中插入部分列的值，这种方式更为灵活；
INSERT…VALUES 语句可以一次插入多条数据。

在 MySQL 中，用单条 INSERT 语句处理多个插入要比使用多条 INSERT 语句更快。

当使用单条 INSERT 语句插入多行数据的时候，只需要将每行数据用圆括号括起来即可。
## 向表中的全部字段添加值
在 test_db 数据库中创建一个课程信息表 tb_courses，包含课程编号 course_id、课程名称 course_name、课程学分 course_grade 和课程备注 course_info，输入的 SQL 语句和执行结果如下所示。
```sql
mysql> CREATE TABLE tb_courses
    -> (
    -> course_id INT NOT NULL AUTO_INCREMENT,
    -> course_name CHAR(40) NOT NULL,
    -> course_grade FLOAT NOT NULL,
    -> course_info CHAR(100) NULL,
    -> PRIMARY KEY(course_id)
    -> );
Query OK, 0 rows affected (0.00 sec)
```
向表中所有字段插入值的方法有两种：一种是指定所有字段名；另一种是完全不指定字段名。

在`tb_courses`表中插入一条新记录，`course_id`值为 1，`course_name`值为`Network`，`course_grade`值为 3，`info`值为`Computer Network`。
```sql
mysql> SELECT * FROM tb_courses;
Empty set (0.00 sec)

mysql> INSERT INTO tb_courses
    -> (course_id,course_name,course_grade,course_info)
    -> VALUES(1,'Network',3,'Computer Network');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
+-----------+-------------+--------------+------------------+
1 row in set (0.00 sec)
```
可以看到插入记录成功。在插入数据时，指定了`tb_courses`表的所有字段，因此将为每一个字段插入新的值。

`INSERT`语句后面的列名称顺序可以不是`tb_courses`表定义时的顺序，即插入数据时，不需要按照表定义的顺序插入，只要保证值的顺序与列字段的顺序相同就可以。

在`tb_courses`表中插入一条新记录，`course_id`值为 2，`course_name`值为`Database`，`course_grade`值为 3，`info`值为`MySQL`。
```sql
mysql> INSERT INTO tb_courses
    -> (course_name,course_info,course_id,course_grade)
    -> VALUES('Database','MySQL',2,3);
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
+-----------+-------------+--------------+------------------+
2 rows in set (0.00 sec)
```
使用`INSERT`插入数据时，允许列名称列表`column_list`为空，此时值列表中需要为表的每一个字段指定值，并且值的顺序必须和数据表中字段定义时的顺序相同。

在`tb_courses`表中插入一条新记录，`course_id`值为 3，`course_name`值为`Java`，`course_grade`值为 4，`info`值为`Jave EE`。
```sql
mysql> INSERT INTO tb_courses
    -> VLAUES(3,'Java',4,'Java EE');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
+-----------+-------------+--------------+------------------+
3 rows in set (0.00 sec)
```
`INSERT`语句中没有指定插入列表，只有一个值列表。在这种情况下，值列表为每一个字段列指定插入的值，并且这些值的顺序必须和`tb_courses`表中字段定义的顺序相同。

> 注意：虽然使用`INSERT`插入数据时可以忽略插入数据的列名称，若值不包含列名称，则`VALUES`关键字后面的值不仅要求完整，而且顺序必须和表定义时列的顺序相同。如果表的结构被修改，对列进行增加、删除或者位置改变操作，这些操作将使得用这种方式插入数据时的顺序也同时改变。如果指定列名称，就不会受到表结构改变的影响。

## 向表中指定字段添加值
为表的指定字段插入数据，是在`INSERT`语句中只向部分字段中插入值，而其他字段的值为表定义时的默认值。

在`tb_courses`表中插入一条新记录，`course_name`值为`System`，`course_grade`值为 3，`course_info`值为`Operating System`。
```sql
mysql> INSERT INTO tb_courses
    -> (course_name,course_grade,course_info)
    -> VALUES('System',3,'Operation System');
Query OK, 1 rows affected (0.08 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            3 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```
## 使用 INSERT INTO…FROM 语句复制表数据
`INSERT INTO…SELECT…FROM`语句用于快速地从一个或多个表中取出数据，并将这些数据作为行数据插入另一个表中。

`SELECT`子句返回的是一个查询到的结果集，`INSERT`语句将这个结果集插入指定表中，结果集中的每行数据的字段数、字段的数据类型都必须与被操作的表完全一致。

在数据库`test_db`中创建一个与`tb_courses`表结构相同的数据表`tb_courses_new`。
```sql
mysql> CREATE TABLE tb_courses_new
    -> (
    -> course_id INT NOT NULL AUTO_INCREMENT,
    -> course_name CHAR(40) NOT NULL,
    -> course_grade FLOAT NOT NULL,
    -> course_info CHAR(100) NULL,
    -> PRIMARY KEY(course_id)
    -> );
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM tb_courses_new;
Empty set (0.00 sec)
```
从`tb_courses`表中查询所有的记录，并将其插入`tb_courses_new`表中。
```sql
mysql> INSERT INTO tb_courses_new
    -> (course_id,course_name,course_grade,course_info)
    -> SELECT course_id,course_name,course_grade,course_info
    -> FROM tb_courses;
Query OK, 4 rows affected (0.17 sec)
Records: 4  Duplicates: 0  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            3 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```
# UPDATE：修改数据
在 MySQL 中，可以使用`UPDATE`语句来修改、更新一个或多个表的数据。
## UPDATE 语句的基本语法
使用`UPDATE`语句修改单个表，语法格式为：
```sql
UPDATE <表名> SET 字段 1=值 1 [,字段 2=值 2… ] [WHERE 子句 ]
[ORDER BY 子句] [LIMIT 子句]
```
语法说明如下：
* <表名>：用于指定要更新的表名称。
* `SET`子句：用于指定表中要修改的列名及其列值。其中，每个指定的列值可以是表达式，也可以是该列对应的默认值。如果指定的是默认值，可用关键字`DEFAULT`表示列值。
* `WHERE`子句：可选项。用于限定表中要修改的行。若不指定，则修改表中所有的行。
* `ORDER BY`子句：可选项。用于限定表中的行被修改的次序。
* `LIMIT`子句：可选项。用于限定被修改的行数。

> 注意：修改一行数据的多个列值时，`SET`子句的每个值用逗号分开即可。

## 修改表中的数据
在`tb_courses_new`表中，更新所有行的`course_grade`字段值为 4。
```sql
mysql> UPDATE tb_courses_new
    -> SET course_grade=4;
Query OK, 3 rows affected (0.11 sec)
Rows matched: 4  Changed: 3  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            4 | Computer Network |
|         2 | Database    |            4 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            4 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```
## 根据条件修改表中的数据
在`tb_courses`表中，更新`course_id`值为 2 的记录，将`course_grade`字段值改为 3.5，将`course_name`字段值改为`B`。
```sql
mysql> UPDATE tb_courses_new
    -> SET course_name='DB',course_grade=3.5
    -> WHERE course_id=2;
Query OK, 1 row affected (0.13 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> SELECT * FROM tb_courses_new;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            4 | Computer Network |
|         2 | DB          |          3.5 | MySQL            |
|         3 | Java        |            4 | Java EE          |
|         4 | System      |            4 | Operating System |
+-----------+-------------+--------------+------------------+
4 rows in set (0.00 sec)
```
注意：保证`UPDATE`以`WHERE`子句结束，通过`WHERE`子句指定被更新的记录所需要满足的条件，如果忽略`WHERE`子句，MySQL 将更新表中所有的行。
# DELETE：删除数据
在 MySQL 中，可以使用`DELETE`语句来删除表的一行或者多行数据。
## 删除单个表中的数据
使用 DELETE 语句从单个表中删除数据，语法格式为：
```sql
DELETE FROM <表名> [WHERE 子句] [ORDER BY 子句] [LIMIT 子句]
```
语法说明如下：
* <表名>：指定要删除数据的表名。
* `ORDER BY`子句：可选项。表示删除时，表中各行将按照子句中指定的顺序进行删除。
* `WHERE`子句：可选项。表示为删除操作限定删除条件，若省略该子句，则代表删除该表中的所有行。
* `LIMIT`子句：可选项。用于告知服务器在控制命令被返回到客户端前被删除行的最大值。

> 注意：在不使用`WHERE`条件的时候，将删除所有数据。

## 删除表中的全部数据
删除`tb_courses_new`表中的全部数据。
```sql
mysql> DELETE FROM tb_courses_new;
Query OK, 3 rows affected (0.12 sec)
mysql> SELECT * FROM tb_courses_new;
Empty set (0.00 sec)
```
## 根据条件删除表中的数据
在`tb_courses_new`表中，删除`course_id`为 4 的记录。
```sql
mysql> DELETE FROM tb_courses
    -> WHERE course_id=4;
Query OK, 1 row affected (0.00 sec)
mysql> SELECT * FROM tb_courses;
+-----------+-------------+--------------+------------------+
| course_id | course_name | course_grade | course_info      |
+-----------+-------------+--------------+------------------+
|         1 | Network     |            3 | Computer Network |
|         2 | Database    |            3 | MySQL            |
|         3 | Java        |            4 | Java EE          |
+-----------+-------------+--------------+------------------+
3 rows in set (0.00 sec)
```
# TRUNCATE：清空表记录
MySQL 提供了`DELETE`和`TRUNCATE`关键字来删除表中的数据。`TRUNCATE`关键字用于完全清空一个表。
```sql
TRUNCATE [TABLE] 表名
```
其中，`TABLE`关键字可省略。

新建表`tb_student_course`，插入数据并查询：
```sql
mysql> CREATE TABLE `tb_student_course` (
    -> `id` int(4) NOT NULL AUTO_INCREMENT,
    -> `name` varchar(25) NOT NULL,
    -> PRIMARY KEY (`id`)
    -> );
Query OK, 0 rows affected (0.04 sec)

mysql> INSERT INTO tb_student_course(name) VALUES ('Java'),('MySQL'),('Python');
Query OK, 3 rows affected (0.05 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM tb_student_course;
+----+--------+
| id | name   |
+----+--------+
|  1 | Java   |
|  2 | MySQL  |
|  3 | Python |
+----+--------+
3 rows in set (0.00 sec)
```
使用`TRUNCATE`语句清空`tb_student_course`表中的记录：
```sql
mysql> TRUNCATE TABLE tb_student_course;
Query OK, 0 rows affected (0.04 sec)

mysql> SELECT * FROM tb_student_course;
Empty set (0.00 sec)
```
## TRUNCATE 和 DELETE 的区别
从逻辑上说，`TRUNCATE`语句与`DELETE`语句作用相同，但是在某些情况下，两者在使用上有所区别。
* `DELETE`是 DML 类型的语句；`TRUNCATE`是 DDL 类型的语句。它们都用来清空表中的数据。
* `DELETE`是逐行一条一条删除记录的；`TRUNCATE`则是直接删除原来的表，再重新创建一个一模一样的新表，而不是逐行删除表中的数据，执行数据比`DELETE`快。因此需要删除表中全部的数据行时，尽量使用`TRUNCATE`语句， 可以缩短执行时间。
* `DELETE`删除数据后，配合事件回滚可以找回数据；`TRUNCATE`不支持事务的回滚，数据删除后无法找回。
* `DELETE`删除数据后，系统不会重新设置自增字段的计数器；`TRUNCATE`清空表记录后，系统会重新设置自增字段的计数器。
* `DELETE`的使用范围更广，因为它可以通过`WHERE`子句指定条件来删除部分数据；而`TRUNCATE`不支持`WHERE`子句，只能删除整体。
* `DELETE`会返回删除数据的行数，但是`TRUNCATE`只会返回 0，没有任何意义。

## 总结
当不需要该表时，用`DROP`；当仍要保留该表，但要删除所有记录时，用`TRUNCATE`；当要删除部分记录时，用`DELETE`。