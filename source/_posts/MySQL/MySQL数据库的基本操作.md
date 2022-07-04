---
title: MySQL数据库的基本操作
date: 2020-04-05 16:52:01
tags: [MySQL]
categories: [MySQL]
---

# 查看数据库
数据库可以看作是一个专门存储数据对象的容器，每一个数据库都有唯一的名称，并且数据库的名称都是有实际意义的，这样就可以清晰的看出每个数据库用来存放什么数据。在 MySQL 数据库中存在系统数据库和自定义数据库，系统数据库是在安装 MySQL 后系统自带的数据库，自定义数据库是由用户定义创建的数据库。

在 MySQL 中，可使用`SHOW DATABASES`语句来查看或显示当前用户权限范围以内的数据库。
```sql
SHOW DATABASES [LIKE '数据库名'];
```
语法说明如下：
* `LIKE`从句是可选项，用于匹配指定的数据库名称。`LIKE`从句可以部分匹配，也可以完全匹配。
* 数据库名由单引号`' '`包围。

## 实例1：查看所有数据库
列出当前用户可查看的所有数据库：
```sql
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
+--------------------+
5 row in set (0.22 sec)
```
可以发现，在上面的列表中有 5 个数据库，它们都是安装 MySQL 时系统自动创建的，其各自功能如下：
* `information_schema`：主要存储了系统中的一些数据库对象信息，比如用户表信息、列信息、权限信息、字符集信息和分区信息等。
* `mysql`：MySQL 的核心数据库，主要负责存储数据库用户、用户访问权限等 MySQL 自己需要使用的控制和管理信息。常用的比如在`mysql`数据库的`user`表中修改`root`用户密码。
* `performance_schema`：主要用于收集数据库服务器性能参数。
* `sakila`：MySQL 提供的样例数据库，该数据库共有 16 张表，这些数据表都是比较常见的，在设计数据库时，可以参照这些样例数据表来快速完成所需的数据表。
* `sys`：`sys`数据库主要提供了一些视图，数据都来自于`performation_schema`，主要是让开发者和使用者更方便地查看性能问题。

## 实例2：使用 LIKE 从句
先创建三个数据库，名字分别为`test_db、db_test、db_test_db`。

1. 使用`LIKE`从句，查看与`test_db`完全匹配的数据库：
```sql
mysql> SHOW DATABASES LIKE 'test_db';
+--------------------+
| Database (test_db) |
+--------------------+
| test_db            |
+--------------------+
1 row in set (0.03 sec)
```
2. 使用`LIKE`从句，查看名字中包含`test`的数据库：
```sql
mysql> SHOW DATABASES LIKE '%test%';
+--------------------+
| Database (%test%)  |
+--------------------+
| db_test            |
+--------------------+
| db_test_db         |
+--------------------+
| test_db            |
+--------------------+
3 row in set (0.03 sec)
```
3. 使用`LIKE`从句，查看名字以`db`开头的数据库：
```sql
mysql> SHOW DATABASES LIKE 'db%';
+----------------+
| Database (db%) |
+----------------+
| db_test        |
+----------------+
| db_test_db     |
+----------------+
2 row in set (0.03 sec)
```
4. 使用`LIKE`从句，查看名字以`db`结尾的数据库：
```sql
mysql> SHOW DATABASES LIKE '%db';
+----------------+
| Database (%db) |
+----------------+
| db_test_db     |
+----------------+
| test_db        |
+----------------+
2 row in set (0.03 sec)
```

# 创建数据库
在 MySQL 中，可以使用`CREATE DATABASE`语句创建数据库：
```sql
CREATE DATABASE [IF NOT EXISTS] <数据库名>
[[DEFAULT] CHARACTER SET <字符集名>] 
[[DEFAULT] COLLATE <校对规则名>];
```
`[ ]`中的内容是可选的。语法说明如下：
* <数据库名>：创建数据库的名称。MySQL 的数据存储区将以目录方式表示 MySQL 数据库，因此数据库名称必须符合操作系统的文件夹命名规则，不能以数字开头，尽量要有实际意义。注意在 MySQL 中不区分大小写。
* `IF NOT EXISTS`：在创建数据库之前进行判断，只有该数据库目前尚不存在时才能执行操作。此选项可以用来避免数据库已经存在而重复创建的错误。
* `[DEFAULT] CHARACTER SET`：指定数据库的字符集。指定字符集的目的是为了避免在数据库中存储的数据出现乱码的情况。如果在创建数据库时不指定字符集，那么就使用系统的默认字符集。
* `[DEFAULT] COLLATE`：指定字符集的默认校对规则。

MySQL 的字符集（`CHARACTER`）和校对规则（`COLLATION`）是两个不同的概念。字符集是用来定义 MySQL 存储字符串的方式，校对规则定义了比较字符串的方式。
## 实例1：最简单的创建 MySQL 数据库的语句
```sql
mysql> CREATE DATABASE test_db;
Query OK, 1 row affected (0.12 sec);
```

若再次输入`CREATE DATABASE test_db;`语句，则系统会给出错误提示信息：
```sql
mysql> CREATE DATABASE test_db;
ERROR 1007 (HY000): Can't create database 'test_db'; database exists
```
提示不能创建`test_db`数据库，数据库已存在。MySQL 不允许在同一系统下创建两个相同名称的数据库。

可以加上`IF NOT EXISTS`从句，就可以避免类似错误。
```sql
mysql> CREATE DATABASE IF NOT EXISTS test_db;
Query OK, 1 row affected (0.12 sec)
```
## 实例2：创建 MySQL 数据库时指定字符集和校对规则
```
mysql> CREATE DATABASE IF NOT EXISTS test_db_char
    -> DEFAULT CHARACTER SET utf8
    -> DEFAULT COLLATE utf8_chinese_ci;
Query OK, 1 row affected (0.03 sec)
```
这时，可以使用`SHOW CREATE DATABASE`查看`test_db_char`数据库的定义声明，发现该数据库的指定字符集为`utf8`：
```sql
mysql> SHOW CREATE DATABASE test_db_char;
+--------------+-----------------------------------------------------------------------+
| Database     | Create Database                                                       |
+--------------+-----------------------------------------------------------------------+
| test_db_char | CREATE DATABASE `test_db_char` /*!40100 DEFAULT CHARACTER SET utf8 */ |
+--------------+-----------------------------------------------------------------------+
1 row in set (0.00 sec)
```
# 修改数据库
在 MySQL 数据库中只能对数据库使用的字符集和校对规则进行修改，数据库的这些特性都储存在`db.opt`文件中。

在 MySQL 中，可以使用`ALTER DATABASE`来修改已经被创建或者存在的数据库的相关参数。
```sql
ALTER DATABASE [数据库名] { 
[ DEFAULT ] CHARACTER SET <字符集名> |
[ DEFAULT ] COLLATE <校对规则名>}
```
语法说明如下：
* `ALTER DATABASE`用于更改数据库的全局特性。
* 使用`ALTER DATABASE`需要获得数据库`ALTER`权限。
* 数据库名称可以忽略，此时语句对应于默认数据库。
* `CHARACTER SET`子句用于更改默认的数据库字符集。

```sql
mysql> SHOW CREATE DATABASE test_db;
+----------+-----------------------------------------------------------------+
| Database | Create Database                                                 |
+----------+-----------------------------------------------------------------+
| test_db  | CREATE DATABASE `test_db` /*!40100 DEFAULT CHARACTER SET utf8 */|
+----------+-----------------------------------------------------------------+
1 row in set (0.05 sec)

mysql> ALTER DATABASE test_db
    -> DEFAULT CHARACTER SET gb2312
    -> DEFAULT COLLATE gb2312_chinese_ci;
mysql> SHOW CREATE DATABASE test_db;
+----------+------------------------------------------------------------------+
| Database | ALTER Database                                                   |
+----------+------------------------------------------------------------------+
| test_db  | ALTER DATABASE `test_db` /*!40100 DEFAULT CHARACTER SET gb2312 */|
+----------+------------------------------------------------------------------+
1 row in set (0.00 sec)
```
# 删除数据库
删除数据库是将已经存在的数据库从磁盘空间上清除，清除之后，数据库中的所有数据也将一同被删除。

在 MySQL 中，当需要删除已创建的数据库时，可以使用`DROP DATABASE`语句。
```sql
DROP DATABASE [ IF EXISTS ] <数据库名>
```
语法说明如下：
* <数据库名>：指定要删除的数据库名。
* `IF EXISTS`：用于防止当数据库不存在时发生错误。
* `DROP DATABASE`：删除数据库中的所有表格并同时删除数据库。使用此语句时要非常小心，以免错误删除。如果要使用`DROP DATABASE`，需要获得数据库`DROP `权限。

```sql
mysql> CREATE DATABASE test_db_del;
Query OK, 1 row affected (0.08 sec)
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
| test_db            |
| test_db_char       |
| test_db_del        |
| world              |
+--------------------+
9 rows in set (0.00 sec)

mysql> DROP DATABASE test_db_del;
Query OK, 0 rows affected (0.57 sec)
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sakila             |
| sys                |
| test_db            |
| test_db_char       |
| world              |
+--------------------+
8 rows in set (0.00 sec) 
```
此时数据库`test_db_del`不存在。再次执行相同的命令，直接使用`DROP DATABASE test_db_del`，系统会报错：
```sql
mysql> DROP DATABASE test_db_del;
ERROR 1008 (HY000): Can't drop database 'test_db_del'; database doesn't exist
```
如果使用`IF EXISTS`从句，可以防止系统报此类错误，如下所示：
```sql
mysql> DROP DATABASE IF EXISTS test_db_del;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
使用`DROP DATABASE`命令时要非常谨慎，在执行该命令后，MySQL 不会给出任何提示确认信息。`DROP DATABASE`删除数据库后，数据库中存储的所有数据表和数据也将一同被删除，而且不能恢复。因此最好在删除数据库之前先将数据库进行备份。
# 选择数据库
在 MySQL 中就有很多系统自带的数据库，那么在操作数据库之前就必须要确定是哪一个数据库。

在 MySQL 中，`USE`语句用来完成一个数据库到另一个数据库的跳转。

当用`CREATE DATABASE`语句创建数据库之后，该数据库不会自动成为当前数据库，需要用`USE`来指定当前数据库。
```
USE <数据库名>
```
该语句可以通知 MySQL 把<数据库名>所指示的数据库作为当前数据库。该数据库保持为默认数据库，直到语段的结尾，或者直到遇见一个不同的`USE`语句。 只有使用`USE`语句来指定某个数据库作为当前数据库之后，才能对该数据库及其存储的数据对象执行操作。
```sql
mysql> USE test_db;
Database changed
```
在执行选择数据库语句时，如果出现`Database changed`提示，则表示选择数据库成功。

可以使用`SELECT DATABASE();`查看当前使用的数据库。