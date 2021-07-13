---
title: MySQL约束
date: 2020-04-15 18:21:53
tags: [MySQL]
categories: [MySQL]
---


在 MySQL 中，约束是指对表中数据的一种约束，能够帮助数据库管理员更好地管理数据库，并且能够确保数据库中数据的正确性和有效性。

在 MySQL 中，主要支持以下 6 种约束：
1. 主键约束
在设计数据表时，一般情况下，都会要求表中设置一个主键。主键是表的一个特殊字段，该字段能唯一标识该表中的每条信息。
2. 外键约束
外键约束经常和主键约束一起使用，用来确保数据的一致性。
3. 唯一约束
唯一约束与主键约束有一个相似的地方，就是它们都能够确保列的唯一性。与主键约束不同的是，唯一约束在一个表中可以有多个，并且设置唯一约束的列是允许有空值的，虽然只能有一个空值。
4. 检查约束
检查约束是用来检查数据表中，字段值是否有效的一个手段。
5. 非空约束
非空约束用来约束表中的字段不能为空。
6. 默认值约束
默认值约束用来约束当数据表中某个字段不输入值时，自动为其添加一个已经设置好的值。
默认值约束通常用在已经设置了非空约束的列，这样能够防止数据表在录入数据时出现错误。

以上 6 种约束中，一个数据表中只能有一个主键约束，其它约束可以有多个。
# 主键约束
主键分为单字段主键和多字段联合主键。

使用主键应注意以下几点：
* 每个表只能定义一个主键。
* 主键值必须唯一标识表中的每一行，且不能为 NULL，即表中不可能存在有相同主键值的两行数据。这是唯一性原则。
* 一个字段名只能在联合主键字段表中出现一次。
* 联合主键不能包含不必要的多余字段。当把联合主键的某一字段删除后，如果剩下的字段构成的主键仍然满足唯一性原则，那么这个联合主键是不正确的。这是最小化原则。

## 在创建表时设置主键约束
在创建数据表时设置主键约束，既可以为表中的一个字段设置主键，也可以为表中多个字段设置联合主键。但是不论使用哪种方法，在一个表中主键只能有一个。
### 设置单字段主键
在`CREATE TABLE`语句中，通过`PRIMARY KEY`关键字来指定主键。

在定义字段的同时指定主键，语法格式如下：
```
<字段名> <数据类型> PRIMARY KEY [默认值]
```
```
mysql> CREATE TABLE tb_emp3
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_emp3;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```
或者是在定义完所有字段之后指定主键，语法格式如下：
```
[CONSTRAINT <约束名>] PRIMARY KEY [字段名]
```
```
mysql> CREATE TABLE tb_emp4
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> PRIMARY KEY(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_emp4;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```
### 在创建表时设置联合主键
所谓的联合主键，就是这个主键是由一张表中多个字段组成的。

比如，设置学生选课数据表时，如果用学生编号做主键，那么一个学生就只能选择一门课程。如果用课程编号做主键，那么一门课程只能有一个学生来选。显然，这两种情况都是不符合实际情况的。

实际上设计学生选课表，要限定的是一个学生只能选择同一课程一次。因此，学生编号和课程编号可以放在一起共同作为主键，这也就是联合主键了。

主键由多个字段联合组成，语法格式如下：
```
PRIMARY KEY [字段1，字段2，…,字段n]
```
注意：当主键是由多个字段组成时，不能直接在字段名后面声明主键约束。
```
mysql> CREATE TABLE tb_emp5
    -> (
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> PRIMARY KEY(name,deptId)
    -> );
Query OK, 0 rows affected (0.37 sec)
mysql> DESC tb_emp5;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| name   | varchar(25) | NO   | PRI | NULL    |       |
| deptId | int(11)     | NO   | PRI | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
3 rows in set (0.14 sec)
```
## 在修改表时添加主键约束
主键约束不仅可以在创建表的同时创建，也可以在修改表时添加。但是需要注意的是，设置成主键约束的字段中不允许有空值。

在修改数据表时添加主键约束的语法格式如下：
```
ALTER TABLE <数据表名> ADD PRIMARY KEY(<字段名>);
```
```
mysql> DESC tb_emp2;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)

mysql> ALTER TABLE tb_emp2 ADD PRIMARY KEY(id);
Query OK, 0 rows affected (0.94 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp2;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.12 sec)
```
通常情况下，当在修改表时要设置表中某个字段的主键约束时，要确保设置成主键约束的字段中值不能够有重复的，并且要保证是非空的。否则，无法设置主键约束。
## 删除主键约束
当一个表中不需要主键约束时，就需要从表中将其删除。

删除主键约束的语法格式如下所示：
```
ALTER TABLE <数据表名> DROP PRIMARY KEY;
```
```
mysql> ALTER TABLE tb_emp2 DROP PRIMARY KEY;
Query OK, 0 rows affected (0.94 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
由于主键约束在一个表中只能有一个，因此不需要指定主键名就可以删除一个表中的主键约束。
# 主键自增长
在 MySQL 中，当主键定义为自增长后，这个主键的值就不再需要用户输入数据了，而由数据库系统根据定义自动赋值。每增加一条记录，主键会自动以相同的步长进行增长。

通过给字段添加`AUTO_INCREMENT`属性来实现主键自增长。语法格式如下：
```
字段名 数据类型 AUTO_INCREMENT
```
* 默认情况下，`AUTO_INCREMENT`的初始值是 1，每新增一条记录，字段值自动加 1。
* 一个表中只能有一个字段使用`AUTO_INCREMENT`约束，且该字段必须有唯一索引，以避免序号重复（即为主键或主键的一部分）。
* `AUTO_INCREMENT`约束的字段必须具备`NOT NULL`属性。
* `AUTO_INCREMENT`约束的字段只能是整数类型（`TINYINT、SMALLINT、INT、BIGINT`等）。
* AUTO_INCREMENT 约束字段的最大值受该字段的数据类型约束，如果达到上限，`AUTO_INCREMENT`就会失效。

```
mysql> CREATE TABLE tb_student(
    -> id INT(4) PRIMARY KEY AUTO_INCREMENT,
    -> name VARCHAR(25) NOT NULL
    -> );
Query OK, 0 rows affected (0.07 sec)
```
上述语句执行成功后，会创建名为`tb_student`的数据表。其中，`id`为主键，每插入一条新记录，`id`的值就会在前一条记录的基础上自动加 1。`name`为非空字段，该字段的值不能为空值（`NULL`）。

加上`AUTO_INCREMENT`约束条件后，字段中的每个值都是自动增加的。因此，这个字段不可能出现相同的值。通常情况下，`AUTO_INCREMENT`都是作为`id`字段的约束条件，并且将`id`字段作为表的主键。
## 指定自增字段初始值
如果第一条记录设置了该字段的初始值，那么新增加的记录就从这个初始值开始自增。例如，如果表中插入的第一条记录的`id`值设置为 5，那么再插入记录时，`id`值就会从 5 开始往上增加。
```
mysql> CREATE TABLE tb_student2 (
    -> id INT NOT NULL AUTO_INCREMENT,
    -> name VARCHAR(20) NOT NULL,
    -> PRIMARY KEY(ID)
    -> )AUTO_INCREMENT=100;
Query OK, 0 rows affected (0.03 sec)

mysql> INSERT INTO tb_student2 (name)VALUES('Java');
Query OK, 1 row affected (0.07 sec)

mysql> SELECT * FROM tb_student2;
+-----+------+
| id  | name |
+-----+------+
| 100 | Java |
+-----+------+
```
由结果可以看出，`id`值从 100 开始自动增长。
## 自增字段值不连续
```
mysql> CREATE TABLE tb_student3(
    -> id INT PRIMARY KEY AUTO_INCREMENT,
    -> name VARCHAR(20) UNIQUE KEY,
    -> age INT DEFAULT NULL
    -> );
Query OK, 0 rows affected (0.04 sec)
```
向`tb_student3`表中插入数据，SQL 语句如下：
```
INSERT INTO tb_student3 VALUES(1,1,1);
```
此时，表`tb_student3`中已经有了`（1,1,1）`这条记录，这时再执行一条插入数据命令：
```
mysql> INSERT INTO tb_student3 VALUES(null,1,1);
ERROR 1062 (23000): Duplicate entry '1' for key 'name'
```
由于表中已经存在`name=1`的记录，所以报`Duplicate key error`（唯一键冲突）。在这之后，再插入新的数据时，自增`id`就是 3，这样就出现了自增字段值不连续的情况。
# 外键约束
MySQL 外键约束（FOREIGN KEY）是表的一个特殊字段，经常与主键约束一起使用。对于两个具有关联关系的表而言，相关联字段中主键所在的表就是主表（父表），外键所在的表就是从表（子表）。

外键用来建立主表与从表的关联关系，为两个表的数据建立连接，约束两个表中数据的一致性和完整性。比如，一个水果摊，只有苹果、桃子、李子、西瓜等 4 种水果，那么，你来到水果摊要买水果就只能选择苹果、桃子、李子和西瓜，其它的水果都是不能购买的。

主表删除某条记录时，从表中与之对应的记录也必须有相应的改变。一个表可以有一个或多个外键，外键可以为空值，若不为空值，则每一个外键的值必须等于主表中主键的某个值。

定义外键时，需要遵守下列规则：
* 主表必须已经存在于数据库中，或者是当前正在创建的表。如果是后一种情况，则主表与从表是同一个表，这样的表称为自参照表，这种结构称为自参照完整性。
* 必须为主表定义主键。
* 主键不能包含空值，但允许在外键中出现空值。也就是说，只要外键的每个非空值出现在指定的主键中，这个外键的内容就是正确的。
* 在主表的表名后面指定列名或列名的组合。这个列或列的组合必须是主表的主键或候选键。
* 外键中列的数目必须和主表的主键中列的数目相同。
* 外键中列的数据类型必须和主表主键中对应列的数据类型相同。

## 在创建表时设置外键约束
在`CREATE TABLE`语句中，通过`FOREIGN KEY`关键字来指定外键，具体的语法格式如下：
```
[CONSTRAINT <外键名>] FOREIGN KEY 字段名 [，字段名2，…]
REFERENCES <主表名> 主键列1 [，主键列2，…]
```
为了展现表与表之间的外键关系，在`test_db`数据库中创建一个部门表`tb_dept1`，表结构如下表所示。

| 字段名称 | 数据类型 | 备注 |
| :--: | :--: | :--: |
| id | INT(11) | 部门编号 |
| name | VARCHAR(22) | 部门名称 |
| location | VARCHAR(22) | 部门位置 |

```
mysql> CREATE TABLE tb_dept1
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) NOT NULL,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)
```
创建数据表`tb_emp6`，并在表`tb_emp6`上创建外键约束，让它的键`deptId`作为外键关联到表`tb_dept1`的主键`id`，SQL 语句和运行结果如下所示。
```
mysql> CREATE TABLE tb_emp6
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> CONSTRAINT fk_emp_dept1
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)

mysql> DESC tb_emp6;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | NO   | PRI | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  | MUL | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (1.33 sec)
```
以上语句执行成功之后，在表`tb_emp6`上添加了名称为`fk_emp_dept1`的外键约束，外键名称为`deptId`，其依赖于表`tb_dept1`的主键`id`。

> 注意：从表的外键关联的必须是主表的主键，且主键和外键的数据类型必须一致。例如，两者都是`INT`类型，或者都是`CHAR`类型。如果不满足这样的要求，在创建从表时，就会出现`ERROR 1005(HY000): Can't create table`错误。

## 在修改表时添加外键约束
外键约束也可以在修改表时添加，但是添加外键约束的前提是：从表中外键列中的数据必须与主表中主键列中的数据一致或者是没有数据。

在修改数据表时添加外键约束的语法格式如下：
```
ALTER TABLE <数据表名> ADD CONSTRAINT <外键名>
FOREIGN KEY(<列名>) REFERENCES <主表名> (<列名>);
```
修改数据表`tb_emp2`，将字段`deptId`设置为外键，与数据表`tb_dept1`的主键`id`进行关联，SQL 语句和运行结果如下所示。
```
mysql> ALTER TABLE tb_emp2
    -> ADD CONSTRAINT fk_tb_dept1
    -> FOREIGN KEY(deptId)
    -> REFERENCES tb_dept1(id);
Query OK, 0 rows affected (1.38 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_emp2\G
*************************** 1. row ***************************
       Table: tb_emp2
Create Table: CREATE TABLE `tb_emp2` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_tb_dept1` (`deptId`),
  CONSTRAINT `fk_tb_dept1` FOREIGN KEY (`deptId`) REFERENCES `tb_dept1` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.12 sec)
```
注意：在为已经创建好的数据表添加外键约束时，要确保添加外键约束的列的值全部来源于主键列，并且外键列不能为空。
## 删除外键约束
当一个表中不需要外键约束时，就需要从表中将其删除。外键一旦删除，就会解除主表和从表间的关联关系。

删除外键约束的语法格式如下所示：
```
ALTER TABLE <表名> DROP FOREIGN KEY <外键约束名>;
```
```
mysql> ALTER TABLE tb_emp2 DROP FOREIGN KEY fk_tb_dept1;
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_emp2\G
*************************** 1. row ***************************
       Table: tb_emp2
Create Table: CREATE TABLE `tb_emp2` (
  `id` int(11) NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_tb_dept1` (`deptId`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```
可以看到，`tb_emp2`中已经不存在`FOREIGN KEY`，原有的名称为`fk_emp_dept`的外键约束删除成功。
# 唯一约束
MySQL 唯一约束（`Unique Key`）是指所有记录中字段的值不能重复出现。例如，为`id`字段加上唯一性约束后，每条记录的`id`值都是唯一的，不能出现重复的情况。如果其中一条记录的`id`值为‘0001’，那么该表中就不能出现另一条记录的`id`值也为‘0001’。

唯一约束与主键约束相似的是它们都可以确保列的唯一性。不同的是，唯一约束在一个表中可有多个，并且设置唯一约束的列允许有空值，但是只能有一个空值。而主键约束在一个表中只能有一个，且不允许有空值。比如，在用户信息表中，为了避免表中用户名重名，可以把用户名设置为唯一约束。
## 在创建表时设置唯一约束
唯一约束可以在创建表时直接设置，通常设置在除了主键以外的其它列上。

在定义完列之后直接使用`UNIQUE`关键字指定唯一约束，语法格式如下：
```
<字段名> <数据类型> UNIQUE
```
```
mysql> CREATE TABLE tb_dept2
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) UNIQUE,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)

mysql> DESC tb_dept2;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(40) | YES  | UNI | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.08 sec)
```
## 在修改表时添加唯一约束
在修改表时添加唯一约束的语法格式为：
```
ALTER TABLE <数据表名> ADD CONSTRAINT <唯一约束名> UNIQUE(<列名>);
```
```
mysql> ALTER TABLE tb_dept1 ADD CONSTRAINT unique_name UNIQUE(name);
Query OK, 0 rows affected (0.63 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_dept1;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | NO   | UNI | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```
## 删除唯一约束
在 MySQL 中删除唯一约束的语法格式如下：
```
ALTER TABLE <表名> DROP INDEX <唯一约束名>;
```
```
mysql> ALTER TABLE tb_dept1 DROP INDEX unique_name;
Query OK, 0 rows affected (0.20 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_dept1;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | NO   |     | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```
# 检查约束
MySQL 检查约束（`CHECK`）是用来检查数据表中字段值有效性的一种手段，可以通过 CREATE TABLE 或 ALTER TABLE 语句实现。设置检查约束时要根据实际情况进行设置，这样能够减少无效数据的输入。
## 选取设置检查约束的字段
检查约束使用`CHECK`关键字，具体的语法格式如下：
```
CHECK <表达式>
```
其中，“表达式”指的就是 SQL 表达式，用于指定需要检查的限定条件。

若将`CHECK`约束子句置于表中某个列的定义之后，则这种约束也称为基于列的`CHECK`约束。

在更新表数据的时候，系统会检查更新后的数据行是否满足`CHECK`约束中的限定条件。MySQL 可以使用简单的表达式来实现`CHECK`约束，也允许使用复杂的表达式作为限定条件，例如在限定条件中加入子查询。

> 注意：若将 CHECK 约束子句置于所有列的定义以及主键约束和外键定义之后，则这种约束也称为基于表的 CHECK 约束。该约束可以同时对表中多个列设置限定条件。

## 在创建表时设置检查约束
一般情况下，如果系统的表结构已经设计完成，那么在创建表时就可以为字段设置检查约束了。

创建表时设置检查约束的语法格式如下：
```
CHECK(<检查约束>)
```
```
mysql> CREATE TABLE tb_emp7
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT,
    -> CHECK(salary>0 AND salary<100),
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)
```
## 在修改表时添加检查约束
如果一个表创建完成，可以通过修改表的方式为表添加检查约束。

修改表时设置检查约束的语法格式如下：
```
ALTER TABLE tb_emp7 ADD CONSTRAINT <检查约束名> CHECK(<检查约束>)
```
```
mysql> ALTER TABLE tb_emp7
    -> ADD CONSTRAINT check_id
    -> CHECK(id>0);
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
## 删除检查约束
修改表时删除检查约束的语法格式如下：
```
ALTER TABLE <数据表名> DROP CONSTRAINT <检查约束名>;
```
```
mysql> ALTER TABLE tb_emp7 DROP CONSTRAINT check_id;
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
# 默认值
默认值（`Default`）的完整称呼是“默认值约束（`Default Constraint`）”，用来指定某列的默认值。在表中插入一条新记录时，如果没有为某个字段赋值，系统就会自动为这个字段插入默认值。

默认值约束通常用在已经设置了非空约束的列，这样能够防止数据表在录入数据时出现错误。
## 在创建表时设置默认值约束
创建表时可以使用`DEFAULT`关键字设置默认值约束，具体的语法格式如下：
```
<字段名> <数据类型> DEFAULT <默认值>;
```
其中，“默认值”为该字段设置的默认值，如果是字符类型的，要用单引号括起来。
```
mysql> CREATE TABLE tb_dept3
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22),
    -> location VARCHAR(50) DEFAULT 'Beijing'
    -> );
Query OK, 0 rows affected (0.37 sec)

mysql> DESC tb_dept3;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | YES  |     | NULL    |       |
| location | varchar(50) | YES  |     | Beijing |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.06 sec)
```
以上语句执行成功之后，表`tb_dept3`上的字段`location`拥有了一个默认值`Beijing`，新插入的记录如果没有指定部门位置，则默认都为`Beijing`。

注意：在创建表时为列添加默认值，可以一次为多个列添加默认值，需要注意不同列的数据类型。
## 在修改表时添加默认值约束
修改表时添加默认值约束的语法格式如下：
```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <数据类型> DEFAULT <默认值>;
```
```
mysql> ALTER TABLE tb_dept3
    -> CHANGE COLUMN location
    -> location VARCHAR(50) DEFAULT 'Shanghai';
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_dept3;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | YES  |     | NULL     |       |
| location | varchar(50) | YES  |     | Shanghai |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```
## 删除默认值约束
当一个表中的列不需要设置默认值时，就需要从表中将其删除。

修改表时删除默认值约束的语法格式如下：
```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> DEFAULT NULL;
```
```
mysql> ALTER TABLE tb_dept3
    -> CHANGE COLUMN location
    -> location VARCHAR(50) DEFAULT NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_dept3;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | YES  |     | NULL     |       |
| location | varchar(50) | YES  |     | NULL     |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```
# 非空约束
MySQL 非空约束（`NOT NULL`）指字段的值不能为空。对于使用了非空约束的字段，如果用户在添加数据时没有指定值，数据库系统就会报错。可以通过`CREATE TABLE`或`ALTER TABLE`语句实现。在表中某个列的定义后加上关键字`NOT NULL`作为限定词，来约束该列的取值不能为空。

比如，在用户信息表中，如果不添加用户名，那么这条用户信息就是无效的，这时就可以为用户名字段设置非空约束。
## 在创建表时设置非空约束
创建表时可以使用`NOT NULL`关键字设置非空约束，具体的语法格式如下：
```
<字段名> <数据类型> NOT NULL;
```
```
mysql> CREATE TABLE tb_dept4
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) NOT NULL,
    -> location VARCHAR(50)
    -> );
Query OK, 0 rows affected (0.37 sec)

mysql> DESC tb_dept3;
+----------+-------------+------+-----+---------+-------+
| Field    | Type        | Null | Key | Default | Extra |
+----------+-------------+------+-----+---------+-------+
| id       | int(11)     | NO   | PRI | NULL    |       |
| name     | varchar(22) | NO   |     | NULL    |       |
| location | varchar(50) | YES  |     | NULL    |       |
+----------+-------------+------+-----+---------+-------+
3 rows in set (0.06 sec)
```
## 在修改表时添加非空约束
如果在创建表时忘记了为字段设置非空约束，也可以通过修改表进行非空约束的添加。

修改表时设置非空约束的语法格式如下：
```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名>
<字段名> <数据类型> NOT NULL;
```
```
mysql> ALTER TABLE tb_dept4
    -> CHANGE COLUMN location
    -> location VARCHAR(50) NOT NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_dept4;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | NO   |     | NULL     |       |
| location | varchar(50) | NO   |     | NULL     |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```
## 删除非空约束
修改表时删除非空约束的语法规则如下：
```
ALTER TABLE <数据表名>
CHANGE COLUMN <字段名> <字段名> <数据类型> NULL;
```
```
mysql> ALTER TABLE tb_dept4
    -> CHANGE COLUMN location
    -> location VARCHAR(50) NULL;
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_dept4;
+----------+-------------+------+-----+----------+-------+
| Field    | Type        | Null | Key | Default  | Extra |
+----------+-------------+------+-----+----------+-------+
| id       | int(11)     | NO   | PRI | NULL     |       |
| name     | varchar(22) | NO   |     | NULL     |       |
| location | varchar(50) | YES  |     | NULL     |       |
+----------+-------------+------+-----+----------+-------+
3 rows in set (0.00 sec)
```
# 查看表中的约束
在 MySQL 中可以使用`SHOW CREATE TABLE`语句来查看表中的约束。

查看数据表中的约束语法格式如下：
```
SHOW CREATE TABLE <数据表名>;
```
```
mysql> CREATE TABLE tb_emp8
    -> (
    -> id INT(11) PRIMARY KEY,
    -> name VARCHAR(22) UNIQUE,
    -> deptId INT(11) NOT NULL,
    -> salary FLOAT DEFAULT 0,
    -> CHECK(salary>0),
    -> FOREIGN KEY(deptId) REFERENCES tb_dept1(id)
    -> );
Query OK, 0 rows affected (0.37 sec)

mysql> SHOW CREATE TABLE tb_emp8 \G
*************************** 1. row ***************************
       Table: tb_emp8
Create Table: CREATE TABLE `tb_emp8` (
  `id` int(11) NOT NULL,
  `name` varchar(22) DEFAULT NULL,
  `deptId` int(11) NOT NULL,
  `salary` float DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `name` (`name`),
  KEY `deptId` (`deptId`),
  CONSTRAINT `tb_emp8_ibfk_1` FOREIGN KEY (`deptId`) REFERENCES `tb_dept1` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.19 sec)
```