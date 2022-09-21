---
title: MySQL 日志
date: 2020-06-05 18:25:14
tags: [MySQL]
categories: [MySQL]
---


# 日志及分类
日志是数据库的重要组成部分，主要用来记录数据库的运行情况、日常操作和错误信息。

MySQL 中的日志可以分为二进制日志、错误日志、通用查询日志和慢查询日志。分析这些日志，可以帮助我们了解 MySQL 数据库的运行情况、日常操作、错误信息和哪些地方需要进行优化。

MySQL 中 4 种日志文件的作用：
* 二进制日志：该日志文件会以二进制的形式记录数据库的各种操作，但不记录查询语句。
* 错误日志：该日志文件会记录 MySQL 服务器的启动、关闭和运行错误等信息。
* 通用查询日志：该日志记录 MySQL 服务器的启动和关闭信息、客户端的连接信息、更新、查询数据记录的 SQL 语句等。
* 慢查询日志：记录执行事件超过指定时间的操作，通过工具分析慢查询日志可以定位 MySQL 服务器性能瓶颈所在。

为了维护 MySQL 数据库，经常需要在 MySQL 中进行日志操作，包含日志文件的启动、查看、停止和删除等，这些操作都是数据库管理中最基本、最重要的操作。

例如，当用户`root`登录到 MySQL 服务器后，就会在日志文件里记录该用户的登录事件、执行操作等信息。当 MySQL 服务器运行时出错，出错信息就会被记录到日志文件里。

日志操作是数据库维护中最重要的手段之一。如果 MySQL 数据库系统意外停止服务，我们可以通过错误日志查看出现错误的原因。还可以通过二进制日志文件来查看用户分别执行了哪些操作、对数据库文件做了哪些修改。然后，还可以根据二进制日志中的记录来修复数据库。

在 MySQL 所支持的日志文件里，除了二进制日志文件外，其它日志文件都是文本文件。默认情况下，MySQL 只会启动错误日志文件，而其它日志则需要手动启动。

使用日志有优点也有缺点。启动日志后，虽然可以对 MySQL 服务器性能进行维护，但是会降低 MySQL 的执行速度。例如，一个查询操作比较频繁的 MySQL 中，记录通用查询日志和慢查询日志要花费很多的时间。

日志文件还会占用大量的硬盘空间。对于用户量非常大、操作非常频繁的数据库，日志文件需要的存储空间甚至比数据库文件需要的存储空间还要大。因此，是否启动日志，启动什么类型的日志要根据具体的应用来决定。
# 错误日志
错误日志主要记录 MySQL 服务器启动和停止过程中的信息、服务器在运行过程中发生的故障和异常情况等。
## 启动和设置错误日志
在 MySQL 数据库中，默认开启错误日志功能。一般情况下，错误日志存储在 MySQL 数据库的数据文件夹下，通常名称为`hostname.err`。其中，`hostname`表示 MySQL 服务器的主机名。

在 MySQL 配置文件中，错误日志所记录的信息可以通过`log-error`和`log-warnings`来定义，其中，`log-err`定义是否启用错误日志功能和错误日志的存储位置，`log-warnings`定义是否将警告信息也记录到错误日志中。

将`log_error`选项加入到 MySQL 配置文件的 `[mysqld]`组中：
```
[mysqld]
log-error=dir/{filename}
```
其中，`dir`参数指定错误日志的存储路径；`filename`参数指定错误日志的文件名；省略参数时文件名默认为主机名，存放在`Data`目录中。

重启 MySQL 服务后，参数开始生效，可以在指定路径下看到`filename.err`的文件，如果没有指定`filename`，那么错误日志将直接默认为`hostname.err`。

注意：错误日志中记录的并非全是错误信息，例如 MySQL 如何启动 InnoDB 的表空间文件、如何初始化自己的存储引擎等，这些也记录在错误日志文件中。
## 查看错误日志
在 MySQL 中，通过`SHOW`命令可以查看错误日志文件所在的目录及文件名信息。
```sql
mysql> SHOW VARIABLES LIKE 'log_error';
+---------------+----------------------------------------------------------------+
| Variable_name | Value                                                          |
+---------------+----------------------------------------------------------------+
| log_error     | C:\ProgramData\MySQL\MySQL Server 5.7\Data\LAPTOP-UHQ6V8KP.err |
+---------------+----------------------------------------------------------------+
```
错误日志以文本文件的形式存储，直接使用普通文本工具就可以查看。
### 删除错误日志
在 MySQL 中，可以使用`mysqladmin`命令来开启新的错误日志。
```sql
mysqladmin -uroot -p flush-logs
```
执行该命令后，MySQL 服务器首先会自动创建一个新的错误日志，然后将旧的错误日志更名为`filename.err-old`。

MySQL 服务器发生异常时，可以在错误日志中找到发生异常的时间、原因，然后根据这些信息来解决异常。对于很久之前的错误日志，查看的可能性不大，可以直接将这些错误日志删除。
# 二进制日志
二进制日志也可叫作变更日志。主要用于记录数据库的变化情况，即 SQL 语句的 DDL 和 DML 语句，不包含数据记录查询操作。

如果 MySQL 数据库意外停止，可以通过二进制日志文件来查看用户执行了哪些操作，对数据库服务器文件做了哪些修改，然后根据二进制日志文件中的记录来恢复数据库服务器。

默认情况下，二进制日志功能是关闭的。可以通过以下命令查看二进制日志是否开启：
```sql
mysql> SHOW VARIABLES LIKE 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
```
## 启动和设置二进制日志
在 MySQL 中，可以通过在配置文件中添加`log-bin`选项来开启二进制日志：
```
[mysqld]
log-bin=dir/[filename]
```
其中，`dir`参数指定二进制文件的存储路径；`filename`参数指定二进制文件的文件名，其形式为`filename.number`，`number`的形式为 000001、000002 等。

每次重启 MySQL 服务后，都会生成一个新的二进制日志文件，这些日志文件的文件名中`filename`部分不会改变，`number`会不断递增。

如果没有`dir`和`filename`参数，二进制日志将默认存储在数据库的数据目录下，默认的文件名为`hostname-bin.number`，其中`hostname`表示主机名。

下面在`my.ini`文件的`[mysqld]`组中添加以下语句：
```
log-bin
```
重启 MySQL 服务器后，可以在 MySQL 数据库的数据目录下看到`LAPTOP-UHQ6V8KP-bin.000001`这个文件，同时还生成了`LAPTOP-UHQ6V8KP-bin.index`文件。此处，MySQL 服务器的主机名为`LAPTOP-UHQ6V8KP`。

还可以在`my.ini`文件的`[mysqld]`组中进行如下修改。
```
log-bin=C:log\mylog
```
重启 MySQL 服务后，可以在`C:log`文件夹下看到`mylog.000001`文件和`mylog.index`文件。
## 查看二进制日志
### 1. 查看二进制日志文件列表
可以使用如下命令查看 MySQL 中有哪些二进制日志文件：
```sql
mysql> SHOW binary logs;
+----------------------------+-----------+
| Log_name                   | File_size |
+----------------------------+-----------+
| LAPTOP-UHQ6V8KP-bin.000001 |       177 |
| LAPTOP-UHQ6V8KP-bin.000002 |       154 |
+----------------------------+-----------+
```
### 2. 查看当前正在写入的二进制日志文件
可以使用以下命令查看当前 MySQL 中正在写入的二进制日志文件。
```sql
mysql> SHOW master status;
+----------------------------+----------+--------------+------------------+-------------------+
| File                       | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------------------+----------+--------------+------------------+-------------------+
| LAPTOP-UHQ6V8KP-bin.000002 |      154 |              |                  |                   |
+----------------------------+----------+--------------+------------------+-------------------+
```
### 3. 查看二进制日志文件内容
二进制日志使用二进制格式存储，不能直接打开查看。如果需要查看二进制日志，必须使用`mysqlbinlog`命令。
```
mysqlbinlog filename.number
```
`mysqlbinlog`命令只在当前文件夹下查找指定的二进制日志，因此需要在二进制日志所在的目录下运行该命令，否则将会找不到指定的二进制日志文件。

下面使用`mysqlbinlog`命令，来查看`C:\log`目录下的`mylog.000001`文件：
```
C:\Users\11645>cd C:\log
C:\log>mysqlbinlog mylog.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#200527  9:33:37 server id 1  end_log_pos 123 CRC32 0x69738cfd  Start: binlog v 4, server v 5.7.29-log created 200527  9:33:37 at startup
......
```
使用`mysqlbinlog`命令时，可以指定二进制文件的存储路径。这样可以确保`mysqlbinlog`命令可以找到二进制文件。上面例子中的命令可以变为如下形式：
```
mysqlbinlog C:\log\mylog.000001
```
这样，`mysqlbinlog`命令就会到`C:\log`目录下去查找`mylog.000001`文件。如果不指定路径，`mysqlbinlog`命令将在当前目录下查找`mylog.000001`文件。

除了`filename.number`文件，MySQL 还会生成一个名为`filename.index`的文件，这个文件存储着所有二进制日志文件的列表，可以用记事本打开该文件。

小技巧：实际工作中，二进制日志文件与数据库的数据文件不放在同一块硬盘上，这样即使数据文件所在的硬盘被破坏，也可以使用另一块硬盘上的二进制日志来恢复数据库文件。两块硬盘同时坏了的可能性要小得多，这样可以保证数据库中数据的安全。
## 删除二进制日志
二进制日志中记录着大量的信息，如果很长时间不清理二进制日志，将会浪费很多的磁盘空间。
### 1. 删除所有二进制日志
使用`RESET MASTER`语句可以删除的所有二进制日志：
```
RESET MASTER;
```
登录 MySQL 数据库后，可以执行该语句来删除所有二进制日志。删除所有二进制日志后，MySQL 将会重新创建新的二进制日志，新二进制日志的编号从 000001 开始。
### 2. 根据编号删除二进制日志
每个二进制日志文件后面有一个 6 位数的编号，如 000001。使用`PURGE MASTER LOGS TO`语句，可以删除指定二进制日志的编号之前的日志。
```sql
PURGE MASTER LOGS TO 'filename.number';
```
该语句将删除编号小于`filename.number`的所有二进制日志。
### 3. 根据创建时间删除二进制日志
使用`PURGE MASTER LOGS TO`语句，可以删除指定时间之前创建的二进制日志：
```sql
PURGE MASTER LOGS TO 'yyyy-mm-dd hh:MM:ss';
```
## 暂时停止二进制日志
在配置文件中设置了`log_bin`选项之后，MySQL 服务器将会一直开启二进制日志功能。删除该选项后就可以停止二进制日志功能，如果需要再次启动这个功能，需要重新添加`log_bin`选项。由于这样比较麻烦，所以 MySQL 提供了暂时停止二进制日志功能的语句。

如果用户不希望自己执行的某些 SQL 语句记录在二进制日志中，可以在执行这些 SQL 语句之前暂停二进制日志功能。

使用`SET`语句来暂停/开启二进制日志功能：
```sql
SET SQL_LOG_BIN=0/1;
```
0 表示暂停二进制日志功能，1 表示开启二进制功能。

`my.ini`中的`[mysqld]`组下面有几个设置参数是关于二进制日志的：
```
expire_logs_days = 10
max_binlog_size = 1​00M
```
`expire_logs_day`定义了 MySQL 清除过期日志的时间、二进制日志自动删除的天数。默认值为 0，表示“没有自动删除”。当 MySQL 启动或刷新二进制日志时可能删除。

`max_binlog_size`定义了单个文件的大小限制，如果二进制日志写入的内容大小超出给定值，日志就会发生滚动（关闭当前文件，重新打开一个新的日志文件）。不能将该变量设置为大于 1GB 或小于 4096B（字节），其默认值是 1GB。
# 使用二进制日志还原数据库
二进制日志中记录了用户对数据库更改的所有操作，如`INSERT`语句、`UPDATE`语句、`CREATE`语句等。如果数据库因为操作不当或其它原因丢失了数据，可以通过二进制日志来查看在一定时间段内用户的操作，结合数据库备份来还原数据库。

数据库遭到意外损坏时，应该先使用最近的备份文件来还原数据库。另外备份之后，数据库可能进行了一些更新，这时可以使用二进制日志来还原。因为二进制日志中存储了更新数据库的语句，如`UPDATE`语句、`INSERT`语句等。

二进制日志还原数据库的命令如下：
```sql
mysqlbinlog filename.number | mysql -u root -p
```
以上命令可以理解成，先使用`mysqlbinlog`命令来读取`filename.number`中的内容，再使用`mysql`命令将这些内容还原到数据库中。

技巧：二进制日志虽然可以用来还原 MySQL 数据库，但是其占用的磁盘空间也是非常大的。因此，在备份 MySQL 数据库之后，应该删除备份之前的二进制日志。如果备份之后发生异常，造成数据库的数据损失，可以通过备份之后的二进制日志进行还原。

使用`mysqlbinlog`命令进行还原操作时，必须是编号小的先还原。例如，`mylog.000001`必须在`mylog.000002`之前还原。

下面使用二进制日志来还原数据库：
```sql
mysqlbinlog mylog.000001 | mysql -u root -p
mysqlbinlog mylog.000002 | mysql -u root -p
mysqlbinlog mylog.000003 | mysql -u root -p
mysqlbinlog mylog.000004 | mysql -u root -p
```
# 通用查询日志
通用查询日志用来记录用户的所有操作，包括启动和关闭 MySQL 服务、更新语句和查询语句等。

默认情况下，通用查询日志功能是关闭的。可以通过以下命令查看通用查询日志是否开启：
```sql
mysql> SHOW VARIABLES LIKE '%general%';
+------------------+----------------------------------------------------------------+
| Variable_name    | Value                                                          |
+------------------+----------------------------------------------------------------+
| general_log      | OFF                                                            |
| general_log_file | C:\ProgramData\MySQL\MySQL Server 5.7\Data\LAPTOP-UHQ6V8KP.log |
+------------------+----------------------------------------------------------------+
```
从结果可以看出，通用查询日志是关闭的，`general_log_file`变量指定了通用查询日志文件所在的位置。
## 启动和设置通用查询日志
在 MySQL 中，可以通过在 MySQL 配置文件添加`log`选项来开启通用查询日志：
```
[mysqld]
log=dir/filename
```
其中，`dir`参数指定通用查询日志的存储路径；`filename`参数指定日志的文件名。如果不指定存储路径，通用查询日志将默认存储到 MySQL 数据库的数据文件夹下。如果不指定文件名，默认文件名为`hostname.log`，其中`hostname`表示主机名。
## 查看通用查询日志
如果希望了解用户最近的操作，可以查看通用查询日志。通用查询日志以文本文件的形式存储，可以使用普通文本文件查看该类型日志内容。

首先我们查看通用查询日志功能是否是开启状态，然后查询`tb_student`表的记录：
```sql
mysql> SHOW VARIABLES LIKE '%general%';
+------------------+----------------------------------------------------------------+
| Variable_name    | Value                                                          |
+------------------+----------------------------------------------------------------+
| general_log      | ON                                                             |
| general_log_file | C:\ProgramData\MySQL\MySQL Server 5.7\Data\LAPTOP-UHQ6V8KP.log |
+------------------+----------------------------------------------------------------+

mysql> use test;
Database changed
mysql> SELECT * FROM tb_student;
+----+--------+
| id | name   |
+----+--------+
|  1 | Java   |
|  2 | MySQL  |
|  3 | Python |
+----+--------+
```
执行成功后，打开通用查询日志，这里日志名称为`LAPTOP-UHQ6V8KP.log`，下面是通用查询日志中的部分内容。
```
C:\Program Files\MySQL\MySQL Server 5.7\bin\mysqld.exe, Version: 5.7.29-log (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
2020-05-29T06:43:44.382878Z     7 Quit
2020-05-29T06:44:10.001382Z     8 Connect root@localhost on  using SSL/TLS
2020-05-29T06:44:10.007532Z     8 Query select @@version_comment limit 1
2020-05-29T06:44:11.748179Z     8 Query SHOW VARIABLES LIKE '%general%'
2020-05-29T06:44:25.487472Z     8 Query SELECT DATABASE()
2020-05-29T06:44:25.487748Z     8 Init DB test
2020-05-29T06:44:35.390523Z     8 Query SELECT * FROM tb_student
```
可以看出，该日志非常清晰地记录了客户端的所有行为。
## 停止通用查询日志
通用查询日志启动后，可以通过两种方法停止该日志。一种是将 MySQL 配置文件中的相关配置注释掉，然后重启服务器，来停止通用查询日志。
```
[mysqld]
#log=dir\filename
```
上述方法需要重启 MySQL 服务器，这在某些场景，比如有业务量访问的情况下是不允许的，这时可以通过另一种方法来动态地控制通用查询日志的开启和关闭。

设置 MySQL 的环境变量`general_log`为关闭状态可以停止该日志：
```sql
mysql> SET GLOBAL general_log=off;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE '%general_log%' \G
*************************** 1. row ***************************
Variable_name: general_log
        Value: OFF
*************************** 2. row ***************************
Variable_name: general_log_file
        Value: C:\ProgramData\MySQL\MySQL Server 5.7\Data\LAPTOP-UHQ6V8KP.log
2 rows in set, 1 warning (0.01 sec)
```
### 删除通用查询日志
在 MySQL 中，可以使用`mysqladmin`命令来开启新的通用查询日志。新的通用查询日志会直接覆盖旧的查询日志，不需要再手动删除了。
```bash
mysqladmin -uroot -p flush-logs
```
需要注意的是，如果希望备份旧的通用查询日志，必须先将旧的日志文件拷贝出来或者改名。然后，再执行`mysqladmin`命令。

除了上述方法之外，还可以手工删除通用查询日志。删除之后需要重新启动 MySQL 服务。重启之后就会生成新的通用查询日志。如果希望备份旧的日志文件，可以将旧的日志文件改名，然后重启 MySQL 服务。

由于通用查询日志会记录用户的所有操作，如果数据库的使用非常频繁，通用查询日志将会占用非常大的磁盘空间，对系统性能影响较大。一般情况下，数据管理员可以删除很长时间之前的通用查询日志或关闭此日志，以保证 MySQL 服务器上的硬盘空间。
# 慢查询日志
慢查询日志用来记录在 MySQL 中执行时间超过指定时间的查询语句。通过慢查询日志，可以查找出哪些查询语句的执行效率低，以便进行优化。

MySQL 慢查询日志是排查问题的 SQL 语句，以及检查当前 MySQL 性能的一个重要功能。如果不是调优需要，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。

默认情况下，慢查询日志功能是关闭的。可以通过以下命令查看是否开启慢查询日志功能。
```sql
mysql> SHOW VARIABLES LIKE 'slow_query%';
+---------------------+---------------------------------------------------------------------+
| Variable_name       | Value                                                               |
+---------------------+---------------------------------------------------------------------+
| slow_query_log      | OFF                                                                 |
| slow_query_log_file | C:\ProgramData\MySQL\MySQL Server 5.7\Data\LAPTOP-UHQ6V8KP-slow.log |
+---------------------+---------------------------------------------------------------------+

mysql> SHOW VARIABLES LIKE 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```
参数说明：
* `slow_query_log`：慢查询开启状态
* `slow_query_log_file`：慢查询日志存放的位置（一般设置为 MySQL 的数据存放目录）
* `long_query_time`：查询超过多少秒才记录

## 启动和设置慢查询日志
可以通过`log-slow-queries`选项开启慢查询日志。通过`long_query_time`选项来设置时间值，时间以秒为单位。如果查询时间超过了这个时间值，这个查询语句将被记录到慢查询日志。

将`log_slow_queries`选项和`long_query_time`选项加入到配置文件的`[mysqld]`组中。
```
[mysqld]
log-slow-queries=dir\filename
long_query_time=n
```
其中：
* `dir`参数指定慢查询日志的存储路径，如果不指定存储路径，慢查询日志将默认存储到 MySQL 数据库的数据文件夹下。
* `filename`参数指定日志的文件名，生成日志文件的完整名称为`filename-slow.log`。如果不指定文件名，默认文件名为`hostname-slow.log`，`hostname`是 MySQL 服务器的主机名。
* `n`参数是设定的时间值，单位是秒。如果不设置`long_query_time`选项，默认时间为 10 秒。

还可以通过以下命令启动慢查询日志、设置指定时间：
```sql
SET GLOBAL slow_query_log=ON/OFF;
SET GLOBAL long_query_time=n;
```
## 查看慢查询日志
如果你想查看哪些查询语句的执行效率低，可以从慢查询日志中获得信息。和错误日志、查询日志一样，慢查询日志也是以文本文件的形式存储的，可以使用普通的文本文件查看工具来查看。

开启 MySQL 慢查询日志功能，并设置时间：
```sql
mysql> SET GLOBAL slow_query_log=ON;
Query OK, 0 rows affected (0.05 sec)

mysql> SET GLOBAL long_query_time=0.001;
Query OK, 0 rows affected (0.00 sec)
```
查询`tb_student`表中的数据：
```sql
mysql> USE test;
Database changed
mysql> SELECT * FROM tb_student;
+----+--------+
| id | name   |
+----+--------+
|  1 | Java   |
|  2 | MySQL  |
|  3 | Python |
+----+--------+
```
慢查询日志的部分内容如下：
```
# Time: 2020-06-01T01:59:18.368780Z
# User@Host: root[root] @ localhost [::1]  Id:     3
# Query_time: 0.006281  Lock_time: 0.000755 Rows_sent: 2  Rows_examined: 1034
use test;
SET timestamp=1590976758;
SHOW VARIABLES LIKE 'slow_query%';
```
## 删除慢查询日志
慢查询日志的删除方法与通用日志的删除方法是一样的。可以使用`mysqladmin`命令来删除。也可以使用手工方式来删除。
```
mysqladmin -uroot -p flush-logs
```
执行该命令后，命令行会提示输入密码。输入正确密码后，将执行删除操作。新的慢查询日志会直接覆盖旧的查询日志，不需要再手动删除。

也可以手工删除慢查询日志，删除之后需要重新启动 MySQL 服务。

注意：通用查询日志和慢查询日志都是使用这个命令，使用时一定要注意，一旦执行这个命令，通用查询日志和慢查询日志都只存在新的日志文件中。如果需要备份旧的慢查询日志文件，必须先将旧的日志改名，然后重启 MySQL 服务或执行`mysqladmin`命令。
# 设置日志输出方式
MySQL 的查询日志支持写入到文件或写入数据表两种输出形式。启用了普通查询日志或慢查询日志功能后，可以选择让服务器把日志写入到日志文件、`mysql`数据库中的日志表、或者同时写到这两个地方。

可以通过以下命令查看日志输出类型：
```sql
mysql> SHOW VARIABLES LIKE '%log_out%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+
```
结果显示，日志输出类型为`FILE`。

要想在运行时更改日志输出目标，可以在启动服务器时，设置全局系统变量`log_output`的值：
```sql
SET GLOBAL log_output='value';
```
`value`的值可以是：
* `FILE`：表示把日志写入到文件。如果未指定`log_output`的值，默认为`FILE`。
* `TABLE`：表示把日志写入到`mysql`数据库的`slow_log`或`general_log`表中。
* MySQL 可以同时支持 2 种日志存储方式，配置的时候以逗号隔开，即`log_output='FILE,TABLE'`。

需要注意的是，系统变量`log_output`只确定了日志使用什么输出目标，并不会启用日志功能。

相对于写入到文件，日志写入到数据表中要耗费更多的系统资源。因此，对于需要启用查询日志，又需要获得更高的系统性能，建议优先选择将日志写入到文件。

日志表（`slow_log`或`general_log`）中的内容只允许查看，不允许修改，除非服务器自己进行更改。因此，你只能对日志表使用`SELECT`语句，不能使用`INSERT、DELETE`或`UPDATE`语句。不过，可以使用`TRUNCATE TABLE`语句来清空日志表。
