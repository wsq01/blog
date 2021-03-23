---
title: MySQL DDL语法
date: 2020-04-19 17:12:01
tags: [MySQL]
categories: [MySQL]
---

# 创建数据库
创建数据库，语法如下所示：
```sql
CREATE DATABASE dbname
```
创建数据库`test1`，命令执行如下：
```sql
CREATE DATABASE test1;
```
如果需要知道系统中都存在哪些数据库，可以用以下命令来查看：
```sql
SHOW DATABASES;
```
在查看了系统中已有的数据库后，可以用如下命令选择要操作的数据库：
```sql
USE dbname
```
选择数据库`test1`：
```sql
USE test1;
```
然后再用以下命令来查看`test1`数据库中创建的所有数据表：
```sql
SHOW TABLES;
```
# 删除数据库
```sql
DROP DATABASE dbname;
```
数据库删除后，下面的所有表数据都会全部删除，所以删除前一定要仔细检查并做好相应备份。
# 创建表
```sql
CREATE TABLE tablename (
	column_name_1 column_type_1 constraints,
	column_name_2 column_type_2 constraints,
	……
	column_name_n column_type_n constraints
)
```
因为 MySQL 的表名是以目录的形式存在于磁盘上，所以表名的字符可以用任何目录名允许的字符。`column_name`是列的名字，`column_type`是列的数据类型，`contraints`是这个列的约束条件。

`CREATE TABLE`语句也可能会包括其他关键字或选项，但至少要包括表的名字和列的细节。
```sql
CREATE TABLE customers(
  cust_id      int       NOT NULL AUTO_INCREMENT,
  cust_name    char(50)  NOT NULL,
  cust_address char(50)  NOT NULL,
  cust_city    char(50)  NOT NULL,
  cust_state   char(5)   NOT NULL,
  cust_zip     char(10)  NOT NULL,
  cust_country char(50)  NOT NULL,
  cust_contact char(50)  NOT NULL,
  cust_email   char(255) NOT NULL,
  PRIMARY KEY (cust_id)
) ENGINE = InnoDB;
```
从上面的例子中可以看到，表名紧跟在`CREATE TABLE`关键字后面。实际的表定义（所有列）括在圆括号之中。各列之间用逗号分隔。这个表由9列组成。每列的定义以列名（它在表中必须是唯一的）开始，后跟列的数据类型。表的主键可以在创建表时用`PRIMARY KEY`关键字指定。这里，列`cust_id`指定作为主键列。整条语句由右圆括号后的分号结束。

在创建新表时，指定的表名必须不存在，否则将出错。如果要防止意外覆盖已有的表，SQL要求首先手工删除该表，然后再重建它，而不是简单地用创建表语句覆盖它。如果你仅想在一个表不存在时创建它，应该在表名后给出`IF NOT EXISTS`。这样做不检查已有表的模式是否与你打算创建的表模式相匹配。它只是查看表名是否存在，并且仅在表名不存在时创建它。
## 使用NULL值
`NULL`值就是没有值或缺值。允许`NULL`值的列也允许在插入行时不给出该列的值。不允许`NULL`值的列不接受该列没有值的行，换句话说，在插入或更新行时，该列必须有值。

每个表列或者是`NULL`列，或者是`NOT NULL`列，这种状态在创建时由表的定义规定。
```sql
CREATE TABLE orders(
  order_num  int      NOT NULL AUTO_INCREMENT,
  order_date datetime NOT NULL,
  cust_id    int      NOT NULL,
  PRIMARY KEY(order_num)
) ENGINE=InnoDB
```
`orders`包含 3 个列，分别是订单号、订单日期和客户 ID。所有 3 个列都需要，因此每个列的定义都含有关键字`NOT NULL`。这将会阻止插入没有值的列。如果试图插入没有值的列，将返回错误，且插入失败。
## 主键
主键值必须唯一。即，表中的每个行必须具有唯一的主键值。如果主键使用单个列，则它的值必须唯一。如果使用多个列，则这些列的组合值必须唯一。

为创建由多个列组成的主键，应该以逗号分隔的列表给出各列名。
```sql
CREATE TABLE orderitems(
  order_num  int      NOT NULL,
  order_item int      NOT NULL,
  prod_id    char(10) NOT NULL,
  quantity   int      NOT NULL,
  item_price decimal(8, 2) NOT NULL,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB
```
`orderitems`表包含`orders`表中每个订单的细节。每个订单有多项物品，但每个订单任何时候都只有 1 个第一项物品，1 个第二项物品，如此等等。因此，订单号（`order_num`列）和订单物品（`order_item`列）的组合是唯一的，从而适合作为主键。
## 使用AUTO_INCREMENT
`AUTO_INCREMENT`告诉 MySQL，本列每当增加一行时自动增量。每次执行一个`INSERT`操作时，MySQL 自动对该列增量，给该列赋予下一个可用的值。
```
cust_id int NOT NULL AUTO_INCREMENT
```
每个表只允许一个`AUTO_INCREMENT`列，而且它必须被索引（如，通过使它成为主键）。

如果一个列被指定为`AUTO_INCREMENT`，则它需要使用特殊的值吗？你可以简单地在`INSERT`语句中指定一个值，只要它是唯一的（至今尚未使用过）即可，该值将被用来替代自动生成的值。后续的增量将开始使用该手工插入的值。

表创建完毕后，如果需要查看一下表的定义，可以使用如下命令：
```sql
DESC tablename
```
## 指定默认值
如果在插入行时没有给出值，MySQL 允许指定此时使用的默认值。默认值用`CREATE TABLE`语句的列定义中的`DEFAULT`关键字指定。
```sql
CREATE TABLE orderitems(
  order_num  int      NOT NULL,
  order_item int      NOT NULL,
  prod_id    char(10) NOT NULL,
  quantity   int      NOT NULL DEFAULT 1,
  item_price decimal(8, 2) NOT NULL,
  PRIMARY KEY (order_num, order_item)
) ENGINE=InnoDB
```
## 引擎类型
与其他 DBMS 一样，MySQL 有一个具体管理和处理数据的内部引擎。在你使用`CREATE TABLE`语句时，该引擎具体创建表，而在你使用`SELECT`语句或进行其他数据库处理时，该引擎在内部处理你的请求。多数时候，此引擎都隐藏在 DBMS 内，不需要过多关注它。

但 MySQL 与其他 DBMS 不一样，它具有多种引擎。它打包多个引擎，这些引擎都隐藏在 MySQL 服务器内，全都能执行`CREATE TABLE和SELECT`等命令。

为什么要发行多种引擎呢？因为它们具有各自不同的功能和特性，为不同的任务选择正确的引擎能获得良好的功能和灵活性。

当然，你完全可以忽略这些数据库引擎。如果省略`ENGINE=`语句，则使用默认引擎（很可能是`MyISAM`），多数 SQL 语句都会默认使用它。但并不是所有语句都默认使用它，这就是为什么`ENGINE=`语句很重要的原因。

以下是几个需要知道的引擎：
* InnoDB 是一个可靠的事务处理引擎，它不支持全文本搜索；
* MEMORY 在功能等同于 MyISAM，但由于数据存储在内存（不是磁盘）中，速度很快（特别适合于临时表）；
* MyISAM 是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。

混用引擎类型有一个大缺陷。外键（用于强制实施引用完整性）不能跨引擎，即使用一个引擎的表不能引用具有使用不同引擎的表的外键。
# 修改表
### 修改表类型
```sql
ALTER TABLE tablename MODIFY [COLUMN] column_definition [FIRST | AFTER col_name]
```
```sql
ALTER TABLE emp MODIFY ename VARCHAR(20);
```
### 增加表字段
```sql
ALTER TABLE tablename ADD [COLUMN] column_definition [FIRST | AFTER col_name]
```
```sql
ALTER TABLE emp ADD COLUMN age INT(3);
```
### 删除表字段
```sql
ALTER TABLE tablename DROP [COLUMN] col_name
```
```sql
ALTER TABLE emp DROP COLUMN age;
```
### 字段改名
```sql
ALTER TABLE tablename CHANGE [COLUMN] old_col_name column_definition [FIRST|AFTER col_name]
```
将`age`改名为`age1`，同时修改字段类型为`int(4)`：
```sql
ALTER TABLE emp CHANGE age age1 INT(4);
```
`change`和`modify`都可以修改表的定义，不同的是`change`后面需要写两次列名，不方便。但是`change`的优点是可以修改列名称，`modify`则不能。
### 修改字段排列顺序
字段增加和修改语法（`ADD/CNAHGE/MODIFY`）中，都有一个可选项`first|after column_name`，这个选项可以用来修改字段在表中的位置，默认`ADD`增加的新字段是加在表的最后位置，而`CHANGE/MODIFY`默认都不会改变字段的位置。

将新增的字段`birth date`加在`ename`之后：
```sql
ALTER TABLE emp ADD birth date AFTER ename;
```
修改字段`age`，将它放在最前面：
```sql
ALTER TABLE emp MODIFY age INT(3) FIRST;
```
### 表改名
```sql
ALTER TABLE tablename RENAME [TO] new_tablename
```
将表`emp`改名为`emp1`。
```sql
ALTER TABLE emp RENAME emp1;
```
# 删除表
```sql
DROP TABLE tablename;
```
