---
title: MySQL数据表的基本操作
date: 2020-04-12 17:07:12
tags: [MySQL]
categories: [MySQL]
---



数据表是数据库的重要组成部分，每一个数据库都是由若干个数据表组成的。
# 创建数据表
所谓创建数据表，指的是在已经创建的数据库中建立新表。创建数据表的过程是规定数据列的属性的过程，同时也是实施数据完整性（包括实体完整性、引用完整性和域完整性）约束的过程。
## 基本语法
在 MySQL 中，可以使用`CREATE TABLE`语句创建表。其语法格式为：
```
CREATE TABLE <表名> ([表定义选项])[表选项][分区选项];
```
其中，[表定义选项]的格式为：
```
<列名1> <类型1> [,…] <列名n> <类型n>
```
`CREATE TABLE`语句的主要语法及使用说明如下：
* `CREATE TABLE`：用于创建给定名称的表，必须拥有表`CREATE`的权限。
* <表名>：指定要创建表的名称，必须符合标识符命名规则。表名称被指定为`db_name.tbl_name`，以便在特定的数据库中创建表。无论是否有当前数据库，都可以通过这种方式创建。在当前数据库中创建表时，可以省略`db_name`。如果使用加引号的识别名，则应对数据库和表名称分别加引号。例如，`'mydb'.'mytbl'`是合法的，但`'mydb.mytbl'`不合法。
* <表定义选项>：表创建定义，由列名（`col_name`）、列的定义（`column_definition`）以及可能的空值说明、完整性约束或表索引组成。
* 默认的情况是，表被创建到当前的数据库中。若表已存在、没有当前数据库或者数据库不存在，则会出现错误。

> 提示：使用`CREATE TABLE`创建表时，必须指定以下信息：
> * 要创建的表的名称不区分大小写，不能使用SQL语言中的关键字。
> * 数据表中每个列（字段）的名称和数据类型，如果创建多个列，要用逗号隔开。

## 在指定的数据库中创建表
数据表属于数据库，在创建数据表之前，应使用语句`USE<数据库>`指定操作在哪个数据库中进行，如果没有选择数据库，就会抛出`No database selected`的错误。
### 示例
创建员工表`tb_emp1`，结构如下表所示。

| 字段名称 | 数据类型 | 备注 |
| :--: | :--: | :--: |
| id | INT(11) | 员工编号 |
| name | VARCHAR(25) | 员工名称 |
| deptld | INT(11) | 所在部门编号 |
| salary | FLOAT | 工资 |

选择创建表的数据库`test_db`，创建`tb_emp1`数据表。
```
mysql> USE test_db;
Database changed
mysql> CREATE TABLE tb_emp1
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.37 sec)
```
语句执行后，便创建了一个名称为`tb_emp1`的数据表，使用`SHOW TABLES;`语句查看数据表是否创建成功。
```
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp1            |
+--------------------+
1 rows in set (0.00 sec)
```
# 修改数据表
修改数据表的前提是数据库中已经存在该表。修改表指的是修改数据库中已经存在的数据表的结构。

在 MySQL 中可以使用`ALTER TABLE`语句来改变原有表的结构，例如增加或删减列、更改原有列类型、重新命名列或表等。

其语法格式如下：
```
ALTER TABLE <表名> [修改选项]
```
修改选项的语法格式如下：
```
ADD COLUMN <列名> <类型>
CHANGE COLUMN <旧列名> <新列名> <新列类型>
ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT }
MODIFY COLUMN <列名> <类型>
DROP COLUMN <列名>
RENAME TO <新表名>
CHARACTER SET <字符集名>
COLLATE <校对规则名>
```
## 修改表名
MySQL 通过`ALTER TABLE`语句来实现表名的修改，语法规则如下：
```
ALTER TABLE <旧表名> RENAME [TO] <新表名>；
```
其中，`TO`为可选参数，使用与否均不影响结果。
```
mysql> ALTER TABLE student RENAME TO tb_students_info;
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW TABLES;
+------------------+
| Tables_in_test   |
+------------------+
| tb_students_info |
+------------------+
1 row in set (0.00 sec)
```
> 提示：修改表名并不修改表的结构，因此修改名称后的表和修改名称前的表的结构是相同的。可以使用`DESC`命令查看修改后的表结构。

## 修改表字符集
MySQL 通过`ALTER TABLE`语句来实现表字符集的修改，语法规则如下：
```
ALTER TABLE 表名 [DEFAULT] CHARACTER SET <字符集名> [DEFAULT] COLLATE <校对规则名>;
```
其中，`DEFAULT`为可选参数，使用与否均不影响结果。
```
mysql> ALTER TABLE tb_students_info CHARACTER SET gb2312  DEFAULT COLLATE gb2312_chinese_ci;
Query OK, 0 rows affected (0.08 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE tb_students_info \G
*************************** 1. row ***************************
       Table: tb_students_info
Create Table: CREATE TABLE `tb_students_info` (
  `id` int(11) NOT NULL,
  `name` varchar(20) CHARACTER SET utf8 COLLATE utf8_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
```
## 添加字段
MySQL 数据表是由行和列构成的，通常把表的“列”称为字段，把表的“行”称为记录。随着业务的变化，可能需要在已有的表中添加新的字段。

MySQL 允许在开头、中间和结尾处添加字段。
### 在末尾添加字段
一个完整的字段包括字段名、数据类型和约束条件。MySQL 添加字段的语法格式如下：
```
ALTER TABLE <表名> ADD <新字段名><数据类型>[约束条件];
```
对语法格式的说明如下：                                       
* <表名> 为数据表的名字；
* <新字段名> 为所要添加的字段的名字；
* <数据类型> 为所要添加的字段能存储数据的数据类型；
* [约束条件] 是可选的，用来对添加的字段进行约束。

这种语法格式默认在表的最后位置（最后一列的后面）添加新字段。
```
mysql> USE test;
Database changed
mysql> CREATE TABLE student (
    -> id INT(4),
    -> name VARCHAR(20),
    -> sex CHAR(1));
Query OK, 0 rows affected (0.09 sec)                           

mysql> DESC student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| sex   | char(1)     | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
3 rows in set (0.01 sec)
 ```
```
mysql> ALTER TABLE student ADD age INT(4);
Query OK, 0 rows affected (0.16 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| sex   | char(1)     | YES  |     | NULL    |       |
| age   | int(4)      | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)
```
由运行结果可以看到，`student`表已经添加了`age`字段，且该字段在表的最后一个位置，添加字段成功。
### 在开头添加字段
MySQL 默认在表的最后位置添加新字段，如果希望在开头位置（第一列的前面）添加新字段，可以使用`FIRST`关键字，语法格式如下：
```
ALTER TABLE <表名> ADD <新字段名> <数据类型> [约束条件] FIRST;
```
`FIRST`关键字一般放在语句的末尾。
```
mysql> ALTER TABLE student ADD stuId INT(4) FIRST;
Query OK, 0 rows affected (0.14 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| stuId | int(4)      | YES  |     | NULL    |       |
| id    | int(4)      | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| sex   | char(1)     | YES  |     | NULL    |       |
| age   | int(4)      | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```
由运行结果可以看到，`student`表中已经添加了`stuId`字段，且该字段在表中的第一个位置，添加字段成功。
### 在中间位置添加字段
MySQL 除了允许在表的开头位置和末尾位置添加字段外，还允许在中间位置（指定的字段之后）添加字段，此时需要使用`AFTER`关键字，语法格式如下：
```
ALTER TABLE <表名> ADD <新字段名> <数据类型> [约束条件] AFTER <已经存在的字段名>;
```
`AFTER`的作用是将新字段添加到某个已有字段后面。

注意，只能在某个已有字段的后面添加新字段，不能在它的前面添加新字段。
```
mysql> ALTER TABLE student ADD stuno INT(11) AFTER name;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0
 
mysql> DESC student;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| stuId | int(4)      | YES  |     | NULL    |       |
| id    | int(4)      | YES  |     | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| stuno | int(11)     | YES  |     | NULL    |       |
| sex   | char(1)     | YES  |     | NULL    |       |
| age   | int(4)      | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```
由运行结果可以看到，`student`表中已经添加了`stuno`字段，且该字段在`name`字段后面的位置，添加字段成功。
## 修改字段
### 修改字段名称
MySQL 中修改表字段名的语法规则如下：
```
ALTER TABLE <表名> CHANGE <旧字段名> <新字段名> <新数据类型>;
```
新数据类型：指修改后的数据类型，如果不需要修改字段的数据类型，可以将新数据类型设置成与原来一样，但数据类型不能为空。
```
mysql> ALTER TABLE tb_emp1 CHANGE col1 col3 CHAR(30);
Query OK, 0 rows affected (0.76 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col3   | char(30)    | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
5 rows in set (0.01 sec)
```
`CHANGE`也可以只修改数据类型，实现和`MODIFY`同样的效果，方法是将 SQL 语句中的“新字段名”和“旧字段名”设置为相同的名称，只改变“数据类型”。

提示：由于不同类型的数据在机器中的存储方式及长度并不相同，修改数据类型可能会影响数据表中已有的数据记录，因此，当数据表中已经有数据时，不要轻易修改数据类型。
### 修改字段数据类型
修改字段的数据类型就是把字段的数据类型转换成另一种数据类型。在 MySQL 中修改字段数据类型的语法规则如下：
```
ALTER TABLE <表名> MODIFY <字段名> <数据类型>
```
```
mysql> ALTER TABLE tb_emp1 MODIFY name VARCHAR(30);
Query OK, 0 rows affected (0.15 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col1   | int(11)     | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| col2   | int(11)     | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
6 rows in set (0.00 sec)
```
语句执行后，发现表`tb_emp1`中`name`字段的数据类型已经修改成`VARCHAR(30)`，修改成功。
## 删除字段
删除字段是将数据表中的某个字段从表中移除，语法格式如下：
```
ALTER TABLE <表名> DROP <字段名>;
```
```
mysql> ALTER TABLE tb_emp1 DROP col2;
Query OK, 0 rows affected (0.53 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| col1   | int(11)     | YES  |     | NULL    |       |
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(30) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float        | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
5 rows in set (0.00 sec)
```
# 删除数据表
在删除表的同时，表的结构和表中所有的数据都会被删除，因此在删除数据表之前最好先备份，以免造成无法挽回的损失。

使用`DROP TABLE`语句可以删除一个或多个数据表，语法格式如下：
```
DROP TABLE [IF EXISTS] 表名1 [ ,表名2, 表名3 ...]
```
对语法格式的说明如下：
* 表名1, 表名2, 表名3 ...表示要被删除的数据表的名称。`DROP TABLE`可以同时删除多个表，只要将表名依次写在后面，相互之间用逗号隔开即可。
* `IF EXISTS`用于在删除数据表之前判断该表是否存在。如果不加`IF EXISTS`，当数据表不存在时 MySQL 将提示错误，中断 SQL 语句的执行；加上`IF EXISTS`后，当数据表不存在时 SQL 语句可以顺利执行，但是会发出警告。

> 两点注意：
> * 用户必须拥有执行`DROP TABLE`命令的权限，否则数据表不会被删除。
> * 表被删除时，用户在该表上的权限不会自动删除。

## 删除表的实例
```
mysql> USE test_db;
Database changed
mysql> CREATE TABLE tb_emp3
    -> (
    -> id INT(11),
    -> name VARCHAR(25),
    -> deptId INT(11),
    -> salary FLOAT
    -> );
Query OK, 0 rows affected (0.27 sec)
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp2            |
| tb_emp3            |
+--------------------+
2 rows in set (0.00 sec)

mysql> DROP TABLE tb_emp3;
Query OK, 0 rows affected (0.22 sec)
mysql> SHOW TABLES;
+--------------------+
| Tables_in_test_db  |
+--------------------+
| tb_emp2            |
+--------------------+
1 rows in set (0.00 sec)
```
# 删除被其它表关联的主表
数据表之间经常存在外键关联的情况，这时如果直接删除父表，会破坏数据表的完整性，也会删除失败。

删除父表有以下两种方法：
* 先删除与它关联的子表，再删除父表；但是这样会同时删除两个表中的数据。
* 将关联表的外键约束取消，再删除父表；适用于需要保留子表的数据，只删除父表的情况。

下面是上面所说的删除父表的第二种方法。
```
CREATE TABLE tb_emp4
(
id INT(11) PRIMARY KEY,
name VARCHAR(22),
location VARCHAR (50)
);

CREATE TABLE tb_emp5
(
id INT(11) PRIMARY KEY,
name VARCHAR(25),
deptId INT(11),
salary FLOAT,
CONSTRAINT fk_emp4_emp5 FOREIGN KEY (deptId) REFERENCES tb_emp4(id)
);

mysql> SHOW CREATE TABLE tb_emp5\G;
*************************** 1. row ***************************
       Table: tb_emp5
Create Table: CREATE TABLE `tb_emp5` (
  `id` int(11) NOT NULL,
  `name` varchar(25) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_emp4_emp5` (`deptId`),
  CONSTRAINT `fk_emp4_emp5` FOREIGN KEY (`deptId`) REFERENCES `tb_emp4` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```
由运行结果可以看出，`tb_emp5`表为子表，具有名称为`fk_emp4_emp5`的外键约束；`tb_emp4`为父表，其主键`id`被子表`tb_ emp5`所关联。

删除被数据表`tb_emp5`关联的数据表`tb_emp4`，SQL 语句如下：
```
mysql> DROP TABLE tb_emp4;
ERROR 1217 (23000): Cannot delete or update a parent row: a foreign key constraint fails
```
由运行结果可以看出，当主表在存在外键约束时，不能被直接删除。
 
下面解除子表`tb_emp5`的外键约束，SQL语句和运行结果如下：
```
mysql> ALTER TABLE tb_emp5 DROP FOREIGN KEY fk_emp4_emp5;
Query OK, 0 rows affected (0.03 sec)
Records: 0  Duplicates: 0  Warnings: 0
```
语句成功执行后，会取消表`tb_emp4`和表`tb_emp5`之间的关联关系。

解除关联关系后，可以使用`DROP TABLE`语句直接删除父表`tb_emp4`，SQL 语句如下：
```
DROP TABLE tb_emp4;

mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| tb_emp5        |
| temp           |
+----------------+
2 rows in set (0.00 sec)
```
可以发现，数据库列表中已经不存在名称为`tb_emp4`的表，删除成功。
# 查看表结构命令
创建完数据表之后，经常需要查看表结构（表信息）。在 MySQL 中，可以使用`DESCRIBE`和`SHOW CREATE TABLE`命令来查看数据表的结构。
## DESCRIBE：以表格的形式展示表结构
`DESCRIBE/DESC`语句会以表格的形式来展示表的字段信息，包括字段名、字段数据类型、是否为主键、是否有默认值等，语法格式如下：
```sql
DESCRIBE <表名>;
// 或简写成：
DESC <表名>;
```
```
mysql> DESC tb_emp1;
+--------+-------------+------+-----+---------+-------+
| Field  | Type        | Null | Key | Default | Extra |
+--------+-------------+------+-----+---------+-------+
| id     | int(11)     | YES  |     | NULL    |       |
| name   | varchar(25) | YES  |     | NULL    |       |
| deptId | int(11)     | YES  |     | NULL    |       |
| salary | float       | YES  |     | NULL    |       |
+--------+-------------+------+-----+---------+-------+
4 rows in set (0.14 sec)
```
其中，各个字段的含义如下：
* `Null`：表示该列是否可以存储`NULL`值。
* `Key`：表示该列是否已编制索引。`PRI`表示该列是表主键的一部分，`UNI`表示该列是`UNIQUE`索引的一部分，`MUL`表示在列中某个给定值允许出现多次。
* `Default`：表示该列是否有默认值，如果有，值是多少。
* `Extra`：表示可以获取的与给定列有关的附加信息，如`AUTO_INCREMENT`等。

## SHOW CREATE TABLE：以SQL语句的形式展示表结构
`SHOW CREATE TABLE`命令会以 SQL 语句的形式来展示表信息。和`DESCRIBE`相比，`SHOW CREATE TABLE`展示的内容更加丰富，它可以查看表的存储引擎和字符编码；另外，还可以通过`\g`或者`\G`参数来控制展示格式。

`SHOW CREATE TABLE`的语法格式如下：
```
SHOW CREATE TABLE <表名>;
```
在`SHOW CREATE TABLE`语句的结尾处（分号前面）添加`\g`或者`\G`参数可以改变展示形式。
```
mysql> SHOW CREATE TABLE tb_emp1;
+---------+------------------------------------------------+
| Table   | Create Table                                   |
+---------+------------------------------------------------+
| tb_emp1 | CREATE TABLE `tb_emp1` (                       |
| `id` int(11) DEFAULT NULL,                               |
| `name` varchar(25) DEFAULT NULL,                         |
| `salary` float DEFAULT NULL                              |
|) ENGINE=InnoDB DEFAULT CHARSET=gb2312                    |
+---------+------------------------------------------------+
1 row in set (0.01 sec)

mysql> SHOW CREATE TABLE tb_emp1 \g;
+---------+------------------------------------------------+
| Table   | Create Table                                   |
+---------+------------------------------------------------+
| tb_emp1 | CREATE TABLE `tb_emp1` (                       |
| `id` int(11) DEFAULT NULL,                               |
| `name` varchar(25) DEFAULT NULL,                         |
| `salary` float DEFAULT NULL                              |
|) ENGINE=InnoDB DEFAULT CHARSET=gb2312                    |
+---------+------------------------------------------------+
1 row in set (0.00 sec)

mysql> SHOW CREATE TABLE tb_emp1\G
*************************** 1. row ***************************
       Table: tb_emp1
Create Table: CREATE TABLE `tb_emp1` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(25) DEFAULT NULL,
  `deptId` int(11) DEFAULT NULL,
  `salary` float DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.03 sec)
```
# SQL语句对应的文件操作
### 1. 查询数据表
在 MySQL 中无论哪种存储引擎的表都会有一个`.frm`文件来保存数据表的结构定义。所以，执行`SHOW TABLES;`语句相当于列出数据库目录中所有`.frm`文件的基本名，所得到的结果是相同的。

有些数据库系统使用注册表来记录某数据库里的所有数据表，但 MySQL 没有这样做，因为，MySQL 数据目录的层次结构已经把“注册表”隐藏在其中了。
### 2. 创建数据表
创建数据表时，需要执行`CREATE TABLE`语句定义数据表的结构。无论是哪一种存储引擎，MySQL 服务器都将创建一个`.frm`文件来保存数据表的结构定义的内部编码。

MySQL 服务器还会根据指定数据表的具体类型创建出其他必要的文件。例如，它将为一个 MyISAM 数据表创建出一个`.MYD`数据文件和一个`.MYI`索引文件；为一个`MERGE`数据表创建出一个`.mgr`数据/索引文件。

对于 InnoDB 数据表，InnoDB 处理程序将在 InnoDB 表空间里为数据表初始化一些数据和索引信息。
### 3. 更新数据表
当执行`ALTER TABLE`语句时，MySQL 服务器将对相对应数据表的`.frm`文件重新进行编码，来表明数据表的结构性变化，还要对有关的数据文件和索引文件的内容进行相应的修改。

`CREATE INDEX`和`DROP INDEX`等语句也是对相应数据表的`.frm`文件重新进行编码，因为 MySQL 服务器在内部是把它们当作等效的`ALTER TABLE`语句来处理的。改变 InnoDB 数据表的结构会引起 InnoDB 处理程序修改 InnoDB 表空间中数据表的数据，同时也对索引做出相应的修改。
### 4. 删除数据表
`DROP TABLE`语句是通过删除该相应数据表的各种有关文件而实现的。

对于某些数据表类型，可以通过在相应的数据库目录里删除与数据表有关的各个文件的办法来手动删除这个数据表。例如，假设`mydb`是当前数据库，`mytb1`是一个 MyISAM、Archive 或 MERGE 类型的数据表，那么`DROP TABLE mytb1`语句就大致等效于下面这两条命令：
```
cd DATADIR（数据库文件存放路径）
rm -f mydb/mytb1.* 或 del mydb/mytb1.*
```
对于 InnoDB 数据表，因为它的某些组成部分在文件系统里没有实体性的文件代表，所以没有等效的文件系统命令。例如，InnoDB 数据表在文件系统里只有一个相应的`.frm`文件，用文件系统级命令删除这个文件后，该数据表在 InnoDB 表空间中对应的数据和索引将没有任何意义。