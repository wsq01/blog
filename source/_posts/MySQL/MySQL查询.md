---
title: MySQL查询
date: 2020-04-24 17:16:51
tags: [MySQL]
categories: [数据库, MySQL]
---


# SELECT：数据表查询语句
在 MySQL 中，可以使用`SELECT`语句来查询数据。
```sql
SELECT
{* | <字段列名>}
[
  FROM <表 1>, <表 2>…
  [WHERE <表达式>
  [GROUP BY <group by definition>
  [HAVING <expression> [{<operator> <expression>}…]]
  [ORDER BY <order by definition>]
  [LIMIT[<offset>,] <row count>]
]
```
各条子句的含义如下：
* `{*|<字段列名>}`包含星号通配符的字段列表，表示所要查询字段的名称。
* <表 1>，<表 2>…，表 1 和表 2 表示查询数据的来源，可以是单个或多个。
* `WHERE <表达式>`是可选项，如果选择该项，将限定查询数据必须满足该查询条件。
* `GROUP BY<字段>`，该子句告诉 MySQL 如何显示查询出来的数据，并按照指定的字段分组。
* `[ORDER BY<字段>]`，该子句告诉 MySQL 按什么样的顺序显示查询出来的数据，可以进行的排序有升序（`ASC`）和降序（`DESC`），默认情况下是升序。
* `[LIMIT[<offset>，]<row count>]`，该子句告诉 MySQL 每次显示查询出来的数据条数。

## 查询表中所有字段
查询所有字段是指查询表中所有字段的数据。MySQL 提供了以下 2 种方式查询表中的所有字段。
* 使用`*`通配符查询所有字段
* 列出表的所有字段

### 使用 * 查询表的所有字段
`SELECT`可以使用`*`查找表中所有字段的数据：
```sql
SELECT * FROM 表名;
```
使用`*`查询时，只能按照数据表中字段的顺序进行排列，不能改变字段的排列顺序。

> 注意：一般情况下，除非需要使用表中所有的字段数据，否则最好不要使用通配符`*`。虽然使用通配符可以节省输入查询语句的时间，但是获取不需要的列数据通常会降低查询和所使用的应用程序的效率。使用`*`的优势是，当不知道所需列的名称时，可以通过`*`获取它们。

### 列出表的所有字段
`SELECT`关键字后面的字段名为需要查找的字段，因此可以将表中所有字段的名称跟在`SELECT`关键字后面。如果忘记了字段名称，可以使用`DESC`命令查看表的结构。
```sql
SELECT id, name, dept_id, age,sex, height, login_date FROM tb_students_info;
```
这种查询方式比较灵活，如果需要改变字段显示的顺序，只需调整`SELECT`关键字后面的字段列表顺序即可。
## 查询表中指定的字段
查询表中的某一个字段的语法格式为：
```sql
SELECT <列名> FROM <表名>;
```
```sql
mysql> SELECT name FROM tb_students_info;
+--------+
| name   |
+--------+
| Dany   |
| Green  |
| Henry  |
| Jane   |
| Jim    |
+--------+
```
使用`SELECT`声明可以获取多个字段下的数据，只需要在关键字`SELECT`后面指定要查找的字段名称，不同字段名称之间用逗号`,`分隔开，最后一个字段后面不需要加逗号：
```sql
SELECT <字段名1>, <字段名2>, …, <字段名n> FROM <表名>;
```
```sql
mysql> SELECT id, name, height FROM tb_students_info;
+----+--------+--------+
| id | name   | height |
+----+--------+--------+
|  1 | Dany   |    160 |
|  2 | Green  |    158 |
|  3 | Henry  |    185 |
|  4 | Jane   |    162 |
|  5 | Jim    |    175 |
+----+--------+--------+
```
# DISTINCT：过滤重复数据
使用`SELECT`语句执行简单的数据查询时，返回的是所有匹配的记录。如果表中的某些字段没有唯一性约束，那么这些字段就可能存在重复值。为了实现查询不重复的数据，MySQL 提供了`DISTINCT`关键字。

`DISTINCT`关键字的主要作用就是对数据表中一个或多个字段重复的数据进行过滤，只返回其中的一条数据给用户。
```sql
SELECT DISTINCT <字段名> FROM <表名>;
```
其中，“字段名”为需要消除重复记录的字段名称，多个字段时用逗号隔开。

使用`DISTINCT`关键字时需要注意以下几点：
* `DISTINCT`关键字只能在`SELECT`语句中使用。
* 在对一个或多个字段去重时，`DISTINCT`关键字必须在所有字段的最前面。
* 如果`DISTINCT`关键字后有多个字段，则会对多个字段进行组合去重，也就是说，只有多个字段组合起来完全是一样的情况下才会被去重。

```sql
mysql> SELECT * FROM test.student;
+----+----------+------+-------+
| id | name     | age  | stuno |
+----+----------+------+-------+
|  1 | zhangsan |   18 |    23 |
|  2 | lisi     |   19 |    24 |
|  3 | wangwu   |   18 |    25 |
|  4 | zhaoliu  |   18 |    26 |
|  5 | zhangsan |   18 |    27 |
|  6 | wangwu   |   20 |    28 |
+----+----------+------+-------+

mysql> SELECT DISTINCT age FROM student;
+------+
| age  |
+------+
|   18 |
|   19 |
|   20 |
+------+

mysql> SELECT DISTINCT name, age FROM student;
+----------+------+
| name     | age  |
+----------+------+
| zhangsan |   18 |
| lisi     |   19 |
| wangwu   |   18 |
| zhaoliu  |   18 |
| wangwu   |   20 |
+----------+------+

mysql> SELECT DISTINCT * FROM student;
+----+----------+------+-------+
| id | name     | age  | stuno |
+----+----------+------+-------+
|  1 | zhangsan |   18 |    23 |
|  2 | lisi     |   19 |    24 |
|  3 | wangwu   |   18 |    25 |
|  4 | zhaoliu  |   18 |    26 |
|  5 | zhangsan |   18 |    27 |
|  6 | wangwu   |   20 |    28 |
+----+----------+------+-------+
```
因为`DISTINCT`只能返回它的目标字段，而无法返回其它字段，所以在实际情况中，我们经常使用`DISTINCT`关键字来返回不重复字段的条数。

查询`student`表中对`name`和`age`字段去重之后记录的条数：
```sql
mysql> SELECT COUNT(DISTINCT name, age) FROM student;
+--------------------------+
| COUNT(DISTINCT name,age) |
+--------------------------+
|                        5 |
+--------------------------+
1 row in set (0.01 sec)
```
# AS：设置别名
为了查询方便，MySQL 提供了`AS`关键字来为表和字段指定别名。
## 为表指定别名
当表名很长或者执行一些特殊查询的时候，为了方便操作，可以为表指定一个别名，用这个别名代替表原来的名称。
```
<表名> [AS] <别名>
```
`AS`关键字可以省略，省略后需要将表名和别名用空格隔开。

注意：表的别名不能与该数据库的其它表同名。字段的别名不能与该表的其它字段同名。在条件表达式中不能使用字段的别名，否则会出现`ERROR 1054 (42S22): Unknown column`这样的错误提示信息。
```sql
mysql> SELECT stu.name,stu.height FROM tb_students_info AS stu;
+--------+--------+
| name   | height |
+--------+--------+
| Dany   |    160 |
| Green  |    158 |
| Henry  |    185 |
| Jane   |    162 |
| Jim    |    175 |
+--------+--------+
```
## 为字段指定别名
在使用`SELECT`语句查询数据时，MySQL 会显示每个`SELECT`后面指定输出的字段。有时为了显示结果更加直观，我们可以为字段指定一个别名。
```sql
<字段名> [AS] <别名>
```
`AS`关键字可以省略，省略后需要将字段名和别名用空格隔开。
```sql
mysql> SELECT name AS student_name, age AS student_age FROM tb_students_info;
+--------------+-------------+
| student_name | student_age |
+--------------+-------------+
| Dany         |          25 |
| Green        |          23 |
| Henry        |          23 |
| Jane         |          22 |
| Jim          |          24 |
+--------------+-------------+
```
注意：表别名只在执行查询时使用，并不在返回结果中显示。而字段定义别名之后，会返回给客户端显示，显示的字段为字段的别名。
# LIMIT：限制查询结果的条数
`LIMIT`用于指定查询结果从哪条记录开始显示，一共显示多少条记录。

`LIMIT`关键字有 3 种使用方式，即指定初始位置、不指定初始位置以及与`OFFSET`组合使用。
## 指定初始位置
`LIMIT`关键字可以指定查询结果从哪条记录开始显示，显示多少条记录。
```
LIMIT 初始位置, 记录数
```
其中，“初始位置”表示从哪条记录开始显示；“记录数”表示显示记录的条数。第一条记录的位置是 0，第二条记录的位置是 1。后面的记录依次类推。

注意：`LIMIT`后的两个参数必须都是正整数。
```sql
mysql> SELECT * FROM tb_students_info LIMIT 3,5;
+----+-------+---------+------+------+--------+------------+
| id | name  | dept_id | age  | sex  | height | login_date |
+----+-------+---------+------+------+--------+------------+
|  4 | Jane  |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim   |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John  |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily  |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan |       4 |   23 | F    |    170 | 2015-10-01 |
+----+-------+---------+------+------+--------+------------+
```
由结果可以看到，该语句返回的是从第 4 条记录开始的之后的 5 条记录。`LIMIT`关键字后的第一个数字 3 表示从第 4 行开始（记录的位置从 0 开始，第 4 行的位置为 3），第二个数字 5 表示返回的行数。
## 不指定初始位置
`LIMIT`关键字不指定初始位置时，记录从第一条记录开始显示。显示记录的条数由`LIMIT`关键字指定。
```sql
LIMIT 记录数
```
其中，“记录数”表示显示记录的条数。如果“记录数”的值小于查询结果的总数，则会从第一条记录开始，显示指定条数的记录。如果“记录数”的值大于查询结果的总数，则会直接显示查询出来的所有记录。
```sql
mysql> SELECT * FROM tb_students_info LIMIT 4;
+----+-------+---------+------+------+--------+------------+
| id | name  | dept_id | age  | sex  | height | login_date |
+----+-------+---------+------+------+--------+------------+
|  1 | Dany  |       1 |   25 | F    |    160 | 2015-09-10 |
|  2 | Green |       3 |   23 | F    |    158 | 2016-10-22 |
|  3 | Henry |       2 |   23 | M    |    185 | 2015-05-31 |
|  4 | Jane  |       1 |   22 | F    |    162 | 2016-12-20 |
+----+-------+---------+------+------+--------+------------+
```
带一个参数的`LIMIT`指定从查询结果的首行开始，唯一的参数表示返回的行数，即`LIMIT n`与`LIMIT 0, n`返回结果相同。带两个参数的`LIMIT`可返回从任何位置开始指定行数的数据。
## LIMIT和OFFSET组合使用
`LIMIT`可以和`OFFSET`组合使用：
```sql
LIMIT 记录数 OFFSET 初始位置
```
参数和`LIMIT`语法中参数含义相同，“初始位置”指定从哪条记录开始显示；“记录数”表示显示记录的条数。
```sql
mysql> SELECT * FROM tb_students_info LIMIT 5 OFFSET 3;
+----+-------+---------+------+------+--------+------------+
| id | name  | dept_id | age  | sex  | height | login_date |
+----+-------+---------+------+------+--------+------------+
|  4 | Jane  |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim   |       1 |   24 | M    |    175 | 2016-01-15 |
|  6 | John  |       2 |   21 | M    |    172 | 2015-11-11 |
|  7 | Lily  |       6 |   22 | F    |    165 | 2016-02-26 |
|  8 | Susan |       4 |   23 | F    |    170 | 2015-10-01 |
+----+-------+---------+------+------+--------+------------+
```
由结果可以看到，该语句返回的是从第 4 条记录开始的之后的 5 条记录。即`LIMIT 5 OFFSET 3`意思是获取从第 4 条记录开始的后面的 5 条记录，和`LIMIT 3,5`返回的结果相同。
# ORDER BY：对查询结果排序
通过条件查询语句可以查询到符合用户需求的数据，但是查询到的数据一般都是按照数据最初被添加到表中的顺序来显示。为了使查询结果的顺序满足用户的要求，MySQL 提供了`ORDER BY`关键字来对查询结果进行排序。

`ORDER BY`关键字主要用来将查询结果中的数据按照一定的顺序进行排序。
```sql
ORDER BY <字段名> [ASC|DESC]
```
说明：
* 字段名：表示需要排序的字段名称，多个字段时用逗号隔开。
* `ASC|DESC`：`ASC`表示字段按升序排序；`DESC`表示字段按降序排序。其中`ASC`为默认值。

使用`ORDER BY`关键字应该注意以下几个方面：
* `ORDER BY`关键字后可以跟子查询。
* 当排序的字段中存在空值时，`ORDER BY`会将该空值作为最小值来对待。
* `ORDER BY`指定多个字段进行排序时，MySQL 会按照字段的顺序从左到右依次进行排序。

## 单字段排序
```sql
mysql> SELECT * FROM tb_students_info ORDER BY height;
+----+--------+---------+------+------+--------+------------+
| id | name   | dept_id | age  | sex  | height | login_date |
+----+--------+---------+------+------+--------+------------+
|  2 | Green  |       3 |   23 | F    |    158 | 2016-10-22 |
|  1 | Dany   |       1 |   25 | F    |    160 | 2015-09-10 |
|  4 | Jane   |       1 |   22 | F    |    162 | 2016-12-20 |
|  5 | Jim    |       1 |   24 | M    |    175 | 2016-01-15 |
|  3 | Henry  |       2 |   23 | M    |    185 | 2015-05-31 |
+----+--------+---------+------+------+--------+------------+
```
由结果可以看到，MySQL 对查询的`height`字段的数据按数值的大小进行了升序排序。
## 多字段排序
```sql
mysql> SELECT name, height FROM tb_students_info ORDER BY height, name;
+--------+--------+
| name   | height |
+--------+--------+
| Green  |    158 |
| Dany   |    160 |
| Jane   |    162 |
| Lily   |    165 |
| Tom    |    165 |
+--------+--------+
```
注意：在对多个字段进行排序时，排序的第一个字段必须有相同的值，才会对第二个字段进行排序。如果第一个字段数据中所有的值都是唯一的，MySQL 将不再对第二个字段进行排序。

默认情况下，查询数据按字母升序进行排序（A～Z），但数据的排序并不仅限于此，还可以使用`ORDER BY`中的`DESC`对查询结果进行降序排序（Z～A）。
```sql
mysql> SELECT name, height FROM tb_student_info ORDER BY height DESC, name ASC;
+--------+--------+
| name   | height |
+--------+--------+
| Henry  |    185 |
| Thomas |    178 |
| Jim    |    175 |
| John   |    172 |
| Susan  |    170 |
+--------+--------+
```
`DESC`关键字只对前面的列进行降序排列，在这里只对`height`字段进行降序。因此，`height`按降序排序，而`name`仍按升序排序。如果想在多个列上进行降序排序，必须对每个列指定`DESC`关键字。
# WHERE：条件查询数据
如果需要有条件的从数据表中查询数据，可以使用`WHERE`关键字来指定查询条件。
```sql
WHERE 查询条件
```
查询条件可以是：
* 带比较运算符和逻辑运算符的查询条件
* 带`BETWEEN AND`关键字的查询条件
* 带`IS NULL`关键字的查询条件
* 带`IN`关键字的查询条件
* 带`LIKE`关键字的查询条件

## 单一条件的查询语句
单一条件指的是在`WHERE`关键字后只有一个查询条件。
```sql
mysql> SELECT name, height FROM tb_students_info WHERE height=170;
+-------+--------+
| name  | height |
+-------+--------+
| Susan |    170 |
+-------+--------+
```
可以看到，查询结果中记录的`height`字段的值等于 170。如果根据指定的条件进行查询时，数据表中没有符合查询条件的记录，系统会提示`Empty set(0.00sec)`。
## 多条件的查询语句
在`WHERE`关键词后可以有多个查询条件。多个查询条件时用逻辑运算符`AND（&&）、OR（||）`或`XOR`隔开。
* `AND`：记录满足所有查询条件时，才会被查询出来。
* `OR`：记录满足任意一个查询条件时，才会被查询出来。
* `XOR`：记录满足其中一个条件，并且不满足另一个条件时，才会被查询出来。

```sql
mysql> SELECT name, age, height FROM tb_students_info WHERE age>21 AND height>=175;
+--------+------+--------+
| name   | age  | height |
+--------+------+--------+
| Henry  |   23 |    185 |
| Jim    |   24 |    175 |
| Thomas |   22 |    178 |
+--------+------+--------+
```
```sql
mysql> SELECT name, age, height FROM tb_students_info WHERE age>21 OR height>=175;
+--------+------+--------+
| name   | age  | height |
+--------+------+--------+
| Dany   |   25 |    160 |
| Green  |   23 |    158 |
| Henry  |   23 |    185 |
| Jane   |   22 |    162 |
| Jim    |   24 |    175 |
| Lily   |   22 |    165 |
| Susan  |   23 |    170 |
| Thomas |   22 |    178 |
| Tom    |   23 |    165 |
+--------+------+--------+
```
```sql
mysql> SELECT name, age, height FROM tb_students_info WHERE age>21 XOR height>=175;
+-------+------+--------+
| name  | age  | height |
+-------+------+--------+
| Dany  |   25 |    160 |
| Green |   23 |    158 |
| Jane  |   22 |    162 |
| Lily  |   22 |    165 |
| Susan |   23 |    170 |
| Tom   |   23 |    165 |
+-------+------+--------+
```
`OR、AND`和`XOR`可以一起使用，但是在使用时要注意运算符的优先级。
# LIKE：模糊查询
`LIKE`关键字主要用于搜索匹配字段中的指定内容。
```
[NOT] LIKE '字符串'
```
其中：
* `NOT`：可选参数，字段中的内容与指定的字符串不匹配时满足条件。
* 字符串：指定用来匹配的字符串。“字符串”可以是一个很完整的字符串，也可以包含通配符。

`LIKE`关键字支持百分号`%`和下划线`_`通配符。

通配符是一种特殊语句，主要用来模糊查询。当不知道真正字符或者懒得输入完整名称时，可以使用通配符来代替一个或多个真正的字符。
## 带有 % 通配符的查询
`%`代表任何长度的字符串，字符串的长度可以为 0。例如，`a%b`表示以字母`a`开头，以字母`b`结尾的任意长度的字符串。该字符串可以代表`ab、acb、accb、accrb`等字符串。
```sql
mysql> SELECT name FROM tb_students_info WHERE name LIKE 'T%';
+--------+
| name   |
+--------+
| Thomas |
| Tom    |
+--------+
```
注意：匹配的字符串必须加单引号或双引号。

`NOT LIKE`表示字符串不匹配时满足条件。
```sql
mysql> SELECT NAME FROM tb_students_info WHERE NAME NOT LIKE 'T%';
+-------+
| NAME  |
+-------+
| Dany  |
| Green |
| Henry |
| Jane  |
| Jim   |
| John  |
| Lily  |
| Susan |
+-------+
```
## 带有“_”通配符的查询
`_`只能代表单个字符，字符的长度不能为 0。例如，`a_b`可以代表`acb、adb、aub`等字符串。

在`tb_students_info`表中，查找所有以字母`y`结尾，且`y`前面只有 4 个字母的学生姓名：
```sql
mysql> SELECT name FROM tb_students_info WHERE name LIKE '____y';
+-------+
| name  |
+-------+
| Henry |
+-------+
```
## LIKE 区分大小写
默认情况下，`LIKE`关键字匹配字符的时候是不区分大小写的。如果需要区分大小写，可以加入`BINARY`关键字。
```sql
mysql> SELECT name FROM tb_students_info WHERE name LIKE 't%';
+--------+
| name   |
+--------+
| Thomas |
| Tom    |
+--------+

mysql> SELECT name FROM tb_students_info WHERE name LIKE BINARY 't%';
Empty set (0.01 sec)
```
## 使用通配符的注意事项和技巧
下面是使用通配符的一些注意事项：
* 注意大小写。MySQL 默认是不区分大小写的。如果区分大小写，像`Tom`这样的数据就不能被`t%`所匹配到。
* 注意尾部空格，尾部空格会干扰通配符的匹配。例如，`T% `就不能匹配到`Tom`。
* 注意`NULL`。`%`通配符可以到匹配任意字符，但是不能匹配`NULL`。也就是说`%`匹配不到数据表中值为`NULL`的记录。

下面是一些使用通配符要记住的技巧。
* 不要过度使用通配符，如果其它操作符能达到相同的目的，应该使用其它操作符。因为 MySQL 对通配符的处理一般会比其他操作符花费更长的时间。
* 在确定使用通配符后，除非绝对有必要，否则不要把它们用在字符串的开始处。把通配符置于搜索模式的开始处，搜索起来是最慢的。
* 仔细注意通配符的位置。如果放错地方，可能不会返回想要的数据。

如果查询内容中包含通配符，可以使用`\`转义符。例如，在`tb_students_info`表中，将学生姓名`Dany`修改为`Dany%`后，查询以`%`结尾的学生姓名：
```sql
mysql> SELECT NAME FROM test.`tb_students_info` WHERE NAME LIKE '%\%';
+-------+
| NAME  |
+-------+
| Dany% |
+-------+
```
# BETWEEN AND：范围查询
`BETWEEN AND`关键字用来判断字段的数值是否在指定范围内。

`BETWEEN AND`需要两个参数，即范围的起始值和终止值。如果字段值在指定的范围内，则这些记录被返回。如果不在指定范围内，则不会被返回。
```sql
[NOT] BETWEEN 取值1 AND 取值2
```
其中：
* `NOT`：可选参数，表示指定范围之外的值。如果字段值不满足指定范围内的值，则这些记录被返回。
* 取值1：表示范围的起始值。
* 取值2：表示范围的终止值。

```sql
mysql> SELECT name, age FROM tb_students_info WHERE age BETWEEN 20 AND 23;
+--------+------+
| name   | age  |
+--------+------+
| Green  |   23 |
| Henry  |   23 |
| Jane   |   22 |
| John   |   21 |
+--------+------+
```
查询结果中包含学生年龄为 20 和 23 的记录，这就说明，在 MySQL 中，`BETWEEN AND`能匹配指定范围内的所有值，包括起始值和终止值。
```sql
mysql> SELECT name, age FROM tb_students_info WHERE age NOT BETWEEN 20 AND 23;
+------+------+
| name | age  |
+------+------+
| Dany |   25 |
| Jim  |   24 |
+------+------+
```
在表`tb_students_info`中查询注册日期在`2015-10-01`和`2016-05-01`之间的学生信息。SQL 语句和运行结果如下。
```sql
mysql> SELECT name, login_date FROM tb_students_info
    -> WHERE login_date BETWEEN '2015-10-01' AND '2016-05-01';
+-------+------------+
| name  | login_date |
+-------+------------+
| Jim   | 2016-01-15 |
| John  | 2015-11-11 |
| Lily  | 2016-02-26 |
| Susan | 2015-10-01 |
+-------+------------+
```
# IS NULL：空值查询
`IS NULL`关键字用来判断字段的值是否为空值（`NULL`）。空值不同于 0，也不同于空字符串。

如果字段的值是空值，则满足查询条件，该记录将被查询出来。如果字段的值不是空值，则不满足查询条件。
```
IS [NOT] NULL
```
其中，`NOT`是可选参数，表示字段值不是空值时满足条件。
```sql
mysql> SELECT `name`, `login_date` FROM tb_students_info WHERE login_date IS NULL;
+--------+------------+
| NAME   | login_date |
+--------+------------+
| Dany   | NULL       |
| Green  | NULL       |
| Henry  | NULL       |
| Jane   | NULL       |
| Thomas | NULL       |
| Tom    | NULL       |
+--------+------------+
```
`IS NOT NULL`表示查询字段值不为空的记录。
```sql
mysql> SELECT `name`, login_date FROM tb_students_info WHERE login_date IS NOT NULL;
+-------+------------+
| name  | login_date |
+-------+------------+
| Jim   | 2016-01-15 |
| John  | 2015-11-11 |
| Lily  | 2016-02-26 |
| Susan | 2015-10-01 |
+-------+------------+
```
# GROUP BY分组查询
`GROUP BY`关键字可以根据一个或多个字段对查询结果进行分组。
```sql
GROUP BY  <字段名>
```
其中，“字段名”表示需要分组的字段名称，多个字段时用逗号隔开。
## GROUP BY单独使用
单独使用`GROUP BY`关键字时，查询结果会只显示每个分组的第一条记录。
```sql
mysql> SELECT `name`, `sex` FROM tb_students_info GROUP BY sex;
+-------+------+
| name  | sex  |
+-------+------+
| Henry | 女   |
| Dany  | 男   |
+-------+------+
```
结果中只显示了两条记录，这两条记录的`sex`字段的值分别为“女”和“男”。
## GROUP BY 与 GROUP_CONCAT() 
`GROUP BY`关键字可以和`GROUP_CONCAT()`函数一起使用。`GROUP_CONCAT()`函数会把每个分组的字段值都显示出来。
```sql
mysql> SELECT `sex`, GROUP_CONCAT(name) FROM tb_students_info GROUP BY sex;
+------+----------------------------+
| sex  | GROUP_CONCAT(name)         |
+------+----------------------------+
| 女   | Henry,Jim,John,Thomas,Tom  |
| 男   | Dany,Green,Jane,Lily,Susan |
+------+----------------------------+
```
由结果可以看到，查询结果分为两组，`sex`字段值为“女”的是一组，值为“男”的是一组，且每组的学生姓名都显示出来了。
```sql
mysql> SELECT age, sex, GROUP_CONCAT(name) FROM tb_students_info GROUP BY age, sex;
+------+------+--------------------+
| age  | sex  | GROUP_CONCAT(name) |
+------+------+--------------------+
|   21 | 女   | John               |
|   22 | 女   | Thomas             |
|   22 | 男   | Jane,Lily          |
|   23 | 女   | Henry,Tom          |
|   23 | 男   | Green,Susan        |
|   24 | 女   | Jim                |
|   25 | 男   | Dany               |
+------+------+--------------------+
```
上面实例在分组过程中，先按照`age`字段进行分组，当`age`字段值相等时，再把`age`字段值相等的记录按照`sex`字段进行分组。
 
多个字段分组查询时，会先按照第一个字段进行分组。如果第一个字段中有相同的值，MySQL 才会按照第二个字段进行分组。如果第一个字段中的数据都是唯一的，那么 MySQL 将不再对第二个字段进行分组。
## GROUP BY 与聚合函数
在数据统计时，`GROUP BY`关键字经常和聚合函数一起使用。
```sql
mysql> SELECT sex, COUNT(sex) FROM tb_students_info GROUP BY sex;
+------+------------+
| sex  | COUNT(sex) |
+------+------------+
| 女   |          5 |
| 男   |          5 |
+------+------------+
```
## GROUP BY 与 WITH ROLLUP
`WITH POLLUP`关键字用来在所有记录的最后加上一条记录，这条记录是上面所有记录的总和，即统计记录数量。
```sql
mysql> SELECT sex,GROUP_CONCAT(name) FROM tb_students_info GROUP BY sex WITH ROLLUP;
+------+------------------------------------------------------+
| sex  | GROUP_CONCAT(name)                                   |
+------+------------------------------------------------------+
| 女   | Henry,Jim,John,Thomas,Tom                            |
| 男   | Dany,Green,Jane,Lily,Susan                           |
| NULL | Henry,Jim,John,Thomas,Tom,Dany,Green,Jane,Lily,Susan |
+------+------------------------------------------------------+
```
查询结果显示，`GROUP_CONCAT(name)`显示了每个分组的`name`字段值。同时，最后一条记录的`GROUP_CONCAT(name)`字段的值刚好是上面分组`name`字段值的总和。
# HAVING：过滤分组
可以使用`HAVING`关键字对分组后的数据进行过滤。
```sql
HAVING <查询条件>
```
`HAVING`关键字和`WHERE`关键字都可以用来过滤数据，且`HAVING`支持`WHERE`关键字中所有的操作符和语法。

但是`WHERE`和`HAVING`关键字也存在以下几点差异：
* 一般情况下，`WHERE`用于过滤数据行，而`HAVING`用于过滤分组。
* `WHERE`查询条件中不可以使用聚合函数，而`HAVING`查询条件中可以使用聚合函数。
* `WHERE`在数据分组前进行过滤，而`HAVING`在数据分组后进行过滤 。
* `WHERE`针对数据库文件进行过滤，而`HAVING`针对查询结果进行过滤。也就是说，`WHERE`根据数据表中的字段直接进行过滤，而`HAVING`是根据前面已经查询出的字段进行过滤。
* `WHERE`查询条件中不可以使用字段别名，而`HAVING`查询条件中可以使用字段别名。

```sql
mysql> SELECT name, sex, height FROM tb_students_info HAVING height>150;
+--------+------+--------+
| name   | sex  | height |
+--------+------+--------+
| Dany   | 男   |    160 |
| Green  | 男   |    158 |
| Henry  | 女   |    185 |
| Jane   | 男   |    162 |
| Jim    | 女   |    175 |
| John   | 女   |    172 |
| Lily   | 男   |    165 |
| Susan  | 男   |    170 |
| Thomas | 女   |    178 |
| Tom    | 女   |    165 |
+--------+------+--------+

mysql> SELECT name, sex, height FROM tb_students_info WHERE height>150;
+--------+------+--------+
| name   | sex  | height |
+--------+------+--------+
| Dany   | 男   |    160 |
| Green  | 男   |    158 |
| Henry  | 女   |    185 |
| Jane   | 男   |    162 |
| Jim    | 女   |    175 |
| John   | 女   |    172 |
| Lily   | 男   |    165 |
| Susan  | 男   |    170 |
| Thomas | 女   |    178 |
| Tom    | 女   |    165 |
+--------+------+--------+
```
上述实例中，因为在`SELECT`关键字后已经查询出了`height`字段，所以`HAVING`和`WHERE`都可以使用。但是如果`SELECT`关键字后没有查询出`height`字段，MySQL 就会报错。
```sql
mysql> SELECT name, sex FROM tb_students_info WHERE height>150;
+--------+------+
| name   | sex  |
+--------+------+
| Dany   | 男   |
| Green  | 男   |
| Henry  | 女   |
| Jane   | 男   |
| Jim    | 女   |
| John   | 女   |
| Lily   | 男   |
| Susan  | 男   |
| Thomas | 女   |
| Tom    | 女   |
+--------+------+

mysql> SELECT name, sex FROM tb_students_info HAVING height>150;
ERROR 1054 (42S22): Unknown column 'height' in 'having clause'
```
由结果可以看出，如果`SELECT`关键字后没有查询出`HAVING`查询条件中使用的`height`字段，MySQL 会提示错误信息：`having`子句中的列`height`未知”。
```sql
mysql> SELECT GROUP_CONCAT(name), sex, height FROM tb_students_info 
    -> GROUP BY height HAVING AVG(height)>170;
+--------------------+------+--------+
| GROUP_CONCAT(name) | sex  | height |
+--------------------+------+--------+
| John               | 女   |    172 |
| Jim                | 女   |    175 |
| Thomas             | 女   |    178 |
| Henry              | 女   |    185 |
+--------------------+------+--------+

mysql> SELECT GROUP_CONCAT(name), sex, height FROM tb_students_info WHERE AVG(height)>170 GROUP BY height;
ERROR 1111 (HY000): Invalid use of group function
```
由结果可以看出，如果在`WHERE`查询条件中使用聚合函数，MySQL 会提示错误信息：无效使用组函数。
