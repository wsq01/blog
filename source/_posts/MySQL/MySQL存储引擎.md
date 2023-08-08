---
title: MySQL存储引擎
date: 2020-04-09 14:33:26
tags: [MySQL]
categories: [数据库, MySQL]
---


数据库存储引擎是数据库底层软件组件，数据库管理系统使用数据引擎进行创建、查询、更新和删除数据操作。简而言之，存储引擎就是指表的类型。数据库的存储引擎决定了表在计算机中的存储方式。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能，使用不同的存储引擎还可以获得特定的功能。

现在许多数据库管理系统都支持多种不同的存储引擎。MySQL 的核心就是存储引擎。

MySQL 提供了多个不同的存储引擎，包括处理事务安全表的引擎和处理非事务安全表的引擎。在 MySQL 中，不需要在整个服务器中使用同一种存储引擎，针对具体的要求，可以对每一个表使用不同的存储引擎。

MySQL 5.7 支持的存储引擎有 InnoDB、MyISAM、Memory、Merge、Archive、CSV、BLACKHOLE 等。可以使用`SHOW ENGINES;`语句查看系统所支持的引擎类型。

{% asset_img 1.png 存储引擎 %}

`Support`列的值表示某种引擎是否能使用，`YES`表示可以使用，`NO`表示不能使用，`DEFAULT`表示该引擎为当前默认的存储引擎。

| 存储引擎	  | 描述 |
| :--: | :--: |
| ARCHIVE     | 用于数据存档的引擎，数据被插入后就不能在修改了，且不支持索引。 |
| CSV         | 在存储数据时，会以逗号作为数据项之间的分隔符。 |
| BLACKHOLE	  | 会丢弃写操作，该操作会返回空内容。 |
| FEDERATED	  | 将数据存储在远程数据库中，用来访问远程表的存储引擎。 |
| InnoDB      | 具备外键支持功能的事务处理引擎 |
| MEMORY      | 置于内存的表 |
| MERGE       | 用来管理由多个 MyISAM 表构成的表集合 |
| MyISAM      | 主要的非事务处理存储引擎 |

# InnoDB存储引擎
InnoDB 是 MySQL 中第一个提供外键约束的存储引擎，而且它对事务的处理能力是其它存储引擎无法与之相比的。

MySQL 5.5 版本以后，默认存储引擎由 MyISAM 修改为 InnoDB。一般情况下，除非有特别的原因需要使用其它存储引擎，否则应该优先考虑 InnoDB 引擎。
## InnoDB优势
InnoDB 之所以如此受宠，主要在于其功能方面的较多优势。
#### 1. 支持事务安装
InnoDB 最重要的一点就是支持事务。InnoDB 还实现了 SQL92 标准所定义的 4 个隔离级别（`READ UNCOMMITTED，READ COMMITTED，REPEATABLE READ`和`SERIALIZABLE`）。
#### 2. 灾难恢复性好
InnoDB 通过`commit、rollback、crash-recovery`来保障数据的安全。

具体来说，`crash-recovery`就是指如果服务器因为硬件或软件的问题而崩溃，不管当时数据是怎样的状态，在重启 MySQL 后，InnoDB 都会自动恢复到发生崩溃之前的状态，并回到用户离开的地方。
#### 3. 使用行级锁
InnoDB 改变了 MyISAM 的锁机制，实现了行锁。虽然 InnoDB 的行锁机制是通过索引来完成的，但毕竟在数据库中 99% 的 SQL 语句都要使用索引来检索数据。行锁定机制也为 InnoDB 在承受高并发压力的环境下增强了不小的竞争力。

在 SQL 查询中可以自由地将 InnoDB 类型的表与其他类型的表混合起来，甚至在同一个查询中也可以混合。
#### 4. 实现了缓冲处理
InnoDB 提供了专门的缓存池，实现了缓冲管理，不仅能缓冲索引也能缓冲数据，常用的数据可以直接从内存中处理，比从磁盘获取数据处理速度要快。相比之下，MyISAM 只是缓存了索引。

InnoDB 的表和索引在一个逻辑表空间中，表空间可以包含数个文件（或原始磁盘分区）。这与 MyISAM 表不同，比如在 MyISAM 表中每个表被保存在分离的文件中。InnoDB 表可以是任何尺寸，即使在文件尺寸被限制为 2GB 的操作系统上。
#### 5. 支持外键
InnoDB 支持外键约束，检查外键、插入、更新和删除，以确保数据的完整性。在存储表中数据时每张表的存储都按主键顺序存放，如果没有显式地在定义表时指定主键，InnoDB 会为每一行生成一个 6 字节的 ROWID ，并以此作为主键。

InnoDB 实现外键引用这一重要特性，使在数据库端控制部分数据的完整性成为可能。虽然很多数据库系统调优专家都建议不要这样做，但是对于不少用户来说，大部分情况下，在数据库端加外键控制仍然是成本最低的选择。
#### 6. 适合需要大型数据库的网站
InnoDB 被用在众多需要高性能的大型数据库网站上。

InnoDB 是为处理巨大数据量时的最大性能设计，它的 CPU 效率可能是任何其他基于磁盘的关系数据库引擎所不能匹敌的。

除了以上几个亮点之外，InnoDB 常常还有很多其它的功能特色带给使用者惊喜。当然，使用 InnoDB 存储引擎肯定也有缺点。相对于其它存储引擎来说，使用 InnoDB 存储引擎的读写效率稍差，且占用的数据空间相对较大。
## 物理存储
使用 InnoDB 时，MySQL 会在数据目录（`Data`）下创建一个名为`ibdata1`的 10MB 大小的自动扩展数据文件，以及两个名为`ib_logfile0`和`ib_logfile1`的 5MB 大小的日志文件。

InnoDB 存储引擎和 MyISAM 不太一样，虽然也有`.frm`文件来存放表结构定义相关的元数据，但是表数据和索引数据是存放在一起的。至于是每个表单独存放还是所有表存放在一起，用户可以自己设置。

InnoDB 的物理存储结构分为两大部分：
### 1. 数据文件（表数据和索引数据）
数据文件用来存放数据表中的数据和所有的索引数据，包括主键和其他普通索引。

InnoDB 存储的数据采用表空间（`Tablepace`）进行存放设计。表空间是用来存放 MySQL 系统相关信息的一个特殊共享表空间。

InnoDB 的表空间分为以下两种形式：
* 共享表空间，表数据和索引都存放在同一个表空间。默认的表空间文件就是上面所提到的 MySQL 初始化路径下的`ibdata1`文件。
* 独立表空间，每个表的数据和索引被存放在一个单独的`.ibd`文件中。

可以通过以下命令查看 MySQL 是否使用独立表空间：
```sql
mysql> SHOW VARIABLES LIKE 'innodb_file_per_table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set, 1 warning (0.01 sec)
```
`innodb_file_per_table`值为`ON`时表示开启独立表文件，InnoDB 表的数据和索引都会以单独的形式存放；值为`OFF`时，InnoDB 表的数据和索引都存放在一个表空间。可以通过设置该参数的值来决定是否使用独立表空间。
#### 共享表空间
共享表空间的数据文件可以设置为固定大小和可自动扩展大小两种形式。自动扩展形式的文件可以设置文件的最大大小和每次扩展量。在创建自动扩展的数据文件时，建议大家最好加上最大尺寸的属性，一个原因是文件系统本身有一定的大小限制，还有一个原因就是方便自身维护。

当表空间快要用完的时候，我们必须要为其增加数据文件，当然，只有共享表空间有此操作。

共享表空间增加数据文件的操作比较简单，只需要在`innodb_data_file_path`参数后面按照标准格式设置好文件路径和相关属性即可。

`innodb_data_file_path`参数负责定义共享表空间的路径、初始化大小、自动扩展策略。可以使用以下命令查看当前共享表空间文件的路径、大小和自动化策略：
```sql
mysql> SHOW VARIABLES LIKE 'innodb_data_file_path%';
+-----------------------+------------------------+
| Variable_name         | Value                  |
+-----------------------+------------------------+
| innodb_data_file_path | ibdata1:12M:autoextend |
+-----------------------+------------------------+
1 row in set, 1 warning (0.01 sec)
```
用户可以通过`innodb_data_file_path`参数来指定表空间文件：
```
innodb_data_file_path=datafile_spec1[;datafile_spec2]...
```
其中，`datafile_spec1`格式为表空间文件路径:大小:属性，还可以指定多个文件组成一个表空间，同时指定文件的属性，例如：
```
[mysqld]
innodb_data_file_path=/db/ibdata1:2000M;/dr2/db/ibdata2:2000M:autoextend
```
这里将`/db/ibdata1`和`/dr2/db/ibdata2`两个文件用来组成表空间。若这两个文件位于不同的磁盘上，磁盘的负载可能被平均，因此可以提高数据库的整体性能。

指定多个文件时，`autoextend`属性只在最后一个数据文件中指定，表示表空间自动扩展。这里表示文件 ibdata1 的大小为 2000MB，文件`ibdata2`的大小为 2000MB，如果用完了 2000MB，该文件还可以自动增长。

设置完`innodb_data_file_path`参数后，所有基于 InnoDB 存储引擎的表的数据都会记录到该共享表空间中。

不过这里需要注意的是，InnoDB 在创建新数据文件时不会创建目录，如果指定目录不存在，则会报错并无法启动。另外，InnoDB 给共享表空间增加数据文件之后，必须要重启数据库系统才能生效。

这也是大多数人一直不太喜欢使用共享表空间而选用独立表空间的原因之一。
#### 独立表空间
通过设置`innodb_file_per_table`参数，可以将每个基于 InnoDB 存储引擎的表产生一个独立表空间。

独立表空间的命名规则为表名`.ibd`。通过这样的方式，用户不用将所有数据都存放于默认的表空间中。

使用`SET`命令打开/关闭独立表空间：
```sql
mysql> SET GLOBAL innodb_file_per_table=1;
Query OK, 0 rows affected (0.00 sec)
mysql> SHOW VARIABLES LIKE 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set, 1 warning (0.03 sec)

mysql> SET GLOBAL innodb_file_per_table=0;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | OFF   |
+-----------------------+-------+
1 row in set, 1 warning (0.00 sec)
```
需要注意的是，单独的表空间文件只存储该表的数据、索引和缓冲等信息。所以无论是使用共享表空间还是独享表空间来存放表，共享表空间都是必须存在的。
### 2. 日志文件
默认情况下，InnoDB 存储引擎的数据目录下会有两个名为`ib_logfile0`和`ib_logfile1`的文件。也称为 InnoDB 存储引擎的重做日志文件。

重做日志文件对 InnoDB 存储引擎至关重要。InnoDB 可以通过重做日志将数据库宕机时已经完成但还没有来得及将数据写入磁盘的事务恢复，也能将所有部分完成并已经写入磁盘的未完成事务回滚，并且将数据还原，以此来保证数据的完整性。

每个 InnoDB 存储引擎至少有 1 个重做日志文件组，每个文件组下至少有 2 个重做日志文件，如默认的`ib_logfile0`和`ib_logfile1`。

如果你的数据库中有 InnoDB 的表，那么千万别全部删除 InnoDB 的日志文件，这很可能会让你的数据库 Crash，无法启动，或者丢失数据。

数据库不工作或停止响应、进程中断等情况，叫做数据库 Crash。

在 MySQL 启动参数文件设置中，InnoDB 的所有参数基本上都带有前缀`innodb_`，不论是 InnoDB 数据还是和日志相关，或者是其他一些性能，事务等等相关的参数都是一样。

下面是影响重做日志文件的参数：
* `innodb_log_file_size`：指定每个重做日志的大小。
* `innodb_log_files_in_group`：指定日志文件组中重做日志文件的数量，默认为 1。
* `innodb_mirrored_log_groups`：指定日志镜像文件组的数量，默认为 1。
* `innodb_log_group_home_dir`：指定日志文件组所在路径，默认为`./`。

简而言之，MySQL 中所有和 InnoDB 相关的系统变量都以“innodb_”做为前缀。

在 MySQL 中，可以通过`skip-innodb`参数来屏蔽 InnoDB 存储引擎，这样即使我们在安装编译时，安装了 InnoDB 存储引擎，使用者也无法创建 InnoDB 的表。
# MyISAM存储引擎
MyISAM 存储引擎是 MySQL 中常见的存储引擎，曾（MySQL 5.1及之前版本）是 MySQL 的默认存储引擎。

MyISAM 是基于 ISAM 存储引擎发展起来的。实际上那会还没有存储引擎的概念，ISAM 只是一种算法，或者说是数据的处理方式。如同 SQL Server/Oracle 这类产品一样，MySQL 对表对象的管理方式只有一种。随着 MySQL 架构的不断发展和演进，最终才引入插件式存储引擎的概念，ISAM 也进化为 MyISAM 并一直作为 MySQL 数据库的默认存储引擎，直到 MySQL 5.5 版本才被 InnoDB 引擎取代了默认存储引擎的地位。
## 优缺点
MyISAM 有一些已经开发出来很多年的特性，可以满足用户的实际需求。例如全文索引、压缩、空间函数（GIS）等。但 MySQL 官方的重心早就不在 MyISAM 引擎上了，所以近些年来，MyISAM 一直没有很大的改进，也存在着许多的缺陷。
#### 优点
* 占用空间小
* 访问速度快，对事务完整性没有要求或以`SELECT、INSERT`为主的应用基本上都可以使用这个引擎来创建表
* 可以配合锁，实现操作系统下的复制备份
* 支持全文检索（InnoDB 在 MySQL 5.6 版本以后也支持全文检索）
* 数据紧凑存储，因此可获得更小的索引和更快的全表扫描性能。

1. 加锁与并发
MyISAM 对整张表加锁，而不是针对行。读取时会对需要读到的所有表加共享锁，写入时对表加排他锁。但是在表有读取查询的同时，也可以往表中插入新的记录（这被称为并发插入）。
2. 修复
对于 MyISAM 表，MySQL 可以手工或者自动执行检查和修复操作。另外，如果 MySQL 服务器已经关闭，也可以通过`myisamchk`命令行工具进行检查和修复操作。
3. 索引特性
MyISAM 支持以下 3 种类型的索引：
 * B-Tree 索引
 * R-Tree 索引
 * Full-text 索引

MyISAM 上面三种索引类型中，最经常使用的就是 B-Tree 索引了，偶尔会使用到 Full-text，但是 R-Tree 索引一般系统中都是很少用到的。另外 MyISAM 的 B-Tree 索引有一个较大的限制，那就是参与一个索引的所有字段的长度之和不能超过 1000 字节。
#### 缺点
* 不支持事务的完整性和并发性
* 不支持行级锁，使用表级锁，并发性差
* 主机宕机后，MyISAM表易损坏，灾难恢复性不佳
* 数据库崩溃后无法安全恢复
* 只缓存索引，数据的缓存是利用操作系统缓冲区来实现的，可能会引发过多的系统调用，且效率不佳

## 物理存储
MyISAM 存储引擎的表在数据库中被存储成 3 个物理文件，文件名与表名相同。扩展名为`frm、MYD`和`MYI`。其中：
* `frm`为扩展名的文件存储表的结构；
* `MYD`为扩展名的文件存储数据，其是`MYData`的缩写；
* `MYI`为扩展名的文件存储索引，其是 MYIndex 的缩写。不管表有多少索引，都是存放在同一个`.MYI`文件中。

MyISAM 类型的数据文件和索引文件可以放置在不同的目录，平均分布 IO，以此来获得更快的速度。

虽然每一个 MyISAM 的表数据都存放在后缀名为`.MYD`的文件中，但是每个文件的存放格式可能并不完全一样。因为 MyISAM 支持 3 种不同的数据存放格式，即静态型、动态型和压缩型。
#### 静态型
静态型为 MyISAM 存储引擎的默认存储格式，其字段是固定长度，这样每个记录都是固定长度的，这种存储方式存储非常迅速，容易缓存，出现故障容易恢复。缺点是占用的空间比动态表多。静态型的表的数据在存储的时候会按照列的宽度定义去补足空格，但是在应用访问的时候并不会得到这些空格，空格在返回给应用之前就被去掉了。

需要注意的是，如果需要保存的内容后面本来就带有空格，那么在返回结果的时候也会被去掉。这一点开发人员在编写程序的时候需要特别注意，因为静态表是默认的存储格式，开发人员可能并没有意识到这一点，从而丢失了尾部的空格。
#### 动态型
动态型包含变长字段，记录的长度不是固定的。这样存储的优点是占用的空间相对较少，但是频繁的更新删除记录会产生碎片，需要定期执行`OPTIMIZE TABLE`语句或`myisamchk -r`命令来改善性能，并且出现故障的时候恢复相对比较困难。
#### 压缩型
与上面两种格式相比，压缩型的表就显得特殊一些。压缩型的表需要使用`myisampack`工具创建，解压缩则用另外的`myisamchk`命令。压缩表是制度的，不支持添加或修改记录。

压缩表是基于静态或动态格式表的，优点在于占用的磁盘空间非常小，可以减少磁盘 I/O，从而提升查询性能。因为每个记录都是被单独压缩的，所以只有非常小的开支。

理论上，MyISAM 存储引擎的表可以被多个数据库实例同时使用同时操作，但是一般不建议这样做，建议尽量不要在多个`mysqld`之间共享 MyISAM 存储文件。

如果表在创建并导入数据以后，不会再进行修改操作，这样的表或许适合采用 MyISAM 压缩表。
# 不同存储引擎的数据表在文件系统里是如何表示的
MySQL 支持 InnoDB、MyISAM、Memory、Merge、Archive、CSV、BLACKHOLE 几种存储引擎，不同存储引擎的数据表在文件系统中的表示也各不相同。

MySQL 中的每一个数据表在磁盘上至少被表示为一个文件，即存放着该数据表结构定义的`.frm`文件。不同的存储引擎还有其它用来存放数据和索引信息的文件。

从 MySQL 8.0 版本开始，`frm`表结构定义文件被取消，MySQL 把表结构信息都写到了系统表空间。

不同存储引擎的数据表在文件系统中的表现形式是不同的。
## MyISAM
MyISAM 存储引擎的数据表在数据库目录里使用 3 个文件来代表，这些文件的基本名与数据表的名字相同，扩展名则表明了文件的具体用途。这三个文件的扩展名分别是：
* `.frm`：表结构定义文件，存放着该数据表的结构定义。
* `.MYD`：`MY Data`的缩写，数据文件，存放着该数据表中各个行的数据。
* `.MYI`：`MY Index`的缩写，索引文件，存放着该数据表的全部索引信息。

创建存储引擎为 MyISAM 的`tb_myisam`表。
```sql
mysql> SET default_storage_engine=MyISAM;
Query OK, 0 rows affected (0.02 sec)

mysql> CREATE TABLE tb_myisam( id INT );
Query OK, 0 rows affected (0.03 sec)
```
## MERGE
MERGE 存储引擎的数据表其实是一个逻辑结构。它代表着由一组结构完全相同的 MyISAM 数据表所构成的集合。有关的查询命令会把它当作一个大数据表来对待。

MERGE 存储引擎的数据表除了拥有存储表结构定义的`.frm`文件以外，还有一个扩展名为`.mgr`的文件，这个文件里不保存数据，而是数据的来源地。通俗的说，就是一份由多个 MyISAM 数据表的名单构成的 MERGE 数据表。

下面创建存储引擎为 MERGE 的`tb_merge`表。
```sql
mysql> SET default_storage_engine=Merge;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE tb_merge( id INT );
Query OK, 0 rows affected (0.02 sec)
```
## InnoDB
对于 InnoDB 存储引擎的数据表，一个表对应两个文件，一个是`*.frm`，存储表结构信息；一个是`*.ibd`，存储表中数据。

创建存储引擎为 InnoDB 的`tb_innodb`表。
```sql
mysql> SET default_storage_engine=InnoDB;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE tb_innodb( id INT );
Query OK, 0 rows affected (0.10 sec)
```
## Memory
Memory 存储引擎的数据表是创建在内存中的数据表。因为 MySQL 服务器把 Memory 数据表的数据和索引都存放在了内存中而不是硬盘上，所以除了相应的`.frm`文件外，Memory 引擎表在文件系统里没有其它相应的代表文件。

创建存储引擎为 Memory 的`tb_memory`表。
```sql
mysql> SET default_storage_engine=Memory;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE tb_memory( id INT );
Query OK, 0 rows affected (0.03 sec)
```
## Archive
Archive 存储引擎的数据表除了拥有`.frm`表结构定义文件外，还有一个扩展名为`.arz`的数据文件，用来存储历史归档数据。执行优化操作时可能还会出现一个扩展名为`.arn`的文件。

创建存储引擎为 Archive 的`tb_archive`表。
```sql
mysql> SET default_storage_engine=Archive;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE tb_archive( id INT );
Query OK, 0 rows affected (0.04 sec)
```
## CSV
与其它类型的存储引擎相同，CSV 引擎表也会包含一个`.frm`表结构定义文件，此外还会创建一个扩展名为`.CSV`的数据文件。这个文件是 CSV 格式的文本文件，用来保存表中的实际数据。

`.CSV`文件可以直接在 Excel 中打开，或者是使用其它文件编辑工具查看。另外，还有一个同名的元信息文件，文件扩展名为`.CSM`，用来保存表的状态及表中保存的数据量。

由于 CSV 文件可被直接编辑，如果操作得当，可以不通过 SQL 语句直接修改 CSV 文件中的内容。

CSV 存储引擎基于 CSV 格式文件存储数据，由于自身文件格式的原因，所有列必须强制指定`NOT NULL`。

创建存储引擎为 CSV 的`tb_csv`表。
```sql
mysql> SET default_storage_engine=csv;
Query OK, 0 rows affected (0.02 sec)

mysql> CREATE TABLE tb_csv(
    -> id INT NOT NULL,
    -> name CHAR(10) NOT NULL
    -> );
Query OK, 0 rows affected (0.04 sec)
```
## BLACKHOLE
由于在 BLACKHOLE 存储引擎的数据表中写入任何数据都会消失，所以除了`.frm`文件，BLACKHOLE 引擎表没有其他相应的代表文件。

创建存储引擎为 BLACKHOLE 的`tb_blackhole`表。
```sql
mysql> SET default_storage_engine=BLACKHOLE;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE TABLE tb_blackhole( id INT );
Query OK, 0 rows affected (0.03 sec)
```

不同存储引擎的数据表在文件系统中的表示：

{% asset_img 2.png %}
# 查看和修改默认存储引擎
如果需要操作默认存储引擎，首先需要查看默认存储引擎。可以通过执行下面的语句来查看默认的存储引擎：
```sql
SHOW VARIABLES LIKE 'default_storage_engine%';
```
使用下面的语句可以修改数据库临时的默认存储引擎：
```sql
SET default_storage_engine=< 存储引擎名 >
```
```sql
mysql> SET default_storage_engine=MylsAM
mysql> SHOW VARIABLES LIKE 'default_storage_engine%';
+------------------------+--------+
| Variable_name          | Value  |
+------------------------+--------+
| default_storage_engine | MylsAM |
+------------------------+--------+
1 row in set, 1 warning (0.01 sec)
```
此时，可以发现 MySQL 的默认存储引擎已经变成了 MyISAM。但是当再次重启客户端时，默认存储引擎仍然是 InnoDB。
# 如何选择MySQL存储引擎
不同的存储引擎都有各自的特性、优势和使用的场合，正确的选择存储引擎可以提高应用的使用效率。
 
为了能够正确地选择存储引擎，必须掌握各种存储引擎的特性。

MySQL存储引擎特性汇总和对比：

| 特性         | MyISAM  | InnoDB  |  MEMORY |
| :--: | :--: | :--: | :--: |
| 存储限制     | 有      |  支持   |   有 |
| 事务安全     | 不支持  |  支持   |   不支持 |
| 锁机制       | 表锁    |  行锁   |   表锁 |
| B树索引      | 支持    |  支持   |   支持 |
| 哈希索引     | 不支持  |  不支持 |    支持 |
| 全文索引     | 支持    |  不支持 |    不支持 |
| 集群索引     | 不支持  |   支持  |    不支持 |
| 数据缓存     |         |  支持  |    支持 |
| 索引缓存     | 支持    |  支持   |    支持 |
| 数据可压缩   |  支持   |  不支持 |     不支持 |
| 空间使用     |  低     |  高     |    N/A |
| 内存使用     |  低     |  高     |    中等 |
| 批量插入速度 |  高     |  低     |     高 |
| 支持外键     |  不支持 |  支持   |     不支持  |

这 3 个存储引擎的应用场合：
#### 1. MyISAM
在 MySQL 5.1 版本及之前的版本，MyISAM 是默认的存储引擎。
 
MyISAM 存储引擎不支持事务和外键，所以访问速度比较快。如果应用主要以读取和写入为主，只有少量的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择 MyISAM 存储引擎是非常适合的。
#### 2. InnoDB
MySQL 5.5 版本之后默认的事务型引擎修改为 InnoDB。
 
InnoDB 存储引擎在事务上具有优势，即支持具有提交、回滚和崩溃恢复能力的事务安装，所以比 MyISAM 存储引擎占用更多的磁盘空间。

如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询以外，还包括很多的更新、删除操作，那么 InnoDB 存储引擎是比较合适的选择。
 
InnoDB 存储引擎除了可以有效地降低由于删除和更新导致的锁定，还可以确保事务的完整提交和回滚，对于类似计费系统或者财务系统等对数据准确性要求比较高的系统，InnoDB 都是合适的选择。
#### 3. MEMORY
MEMORY 存储引擎将所有数据保存在 RAM 中，所以该存储引擎的数据访问速度快，但是安全上没有保障。
 
MEMORY 对表的大小有限制，太大的表无法缓存在内存中。由于使用 MEMORY 存储引擎没有安全保障，所以要确保数据库异常终止后表中的数据可以恢复。
 
如果应用中涉及数据比较少，且需要进行快速访问，则适合使用 MEMORY 存储引擎。

不同应用的特点是千差万别的，选择适应存储引擎才是最佳方案也不是绝对的，这需要根据实际应用进行测试，从而得到最适合的结果。
# 修改数据表的存储引擎
```sql
ALTER TABLE <表名> ENGINE=<存储引擎名>;
```
`ENGINE`关键字用来指明新的存储引擎。

下面将数据表`student`的存储引擎修改为 MyISAM。
```sql
mysql> SHOW CREATE TABLE student \G
*************************** 1. row ***************************
       Table: student
Create Table: CREATE TABLE `student` (
  `stuId` int(4) DEFAULT NULL,
  `id` int(4) DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  `stuno` int(11) DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `age` int(4) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1
1 row in set (0.01 sec)
```
```sql
ALTER TABLE student ENGINE=MyISAM;
```
```sql
mysql> SHOW CREATE TABLE student \G;
*************************** 1. row ***************************
       Table: student
Create Table: CREATE TABLE `student` (
  `stuId` int(4) DEFAULT NULL,
  `id` int(4) DEFAULT NULL,
  `name` varchar(20) DEFAULT NULL,
  `stuno` int(11) DEFAULT NULL,
  `sex` char(1) DEFAULT NULL,
  `age` int(4) DEFAULT NULL,
  `stuId2` int(4) unsigned DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

以上这种方法适用于修改单个表的存储引擎，如果希望修改默认的存储引擎，就需要修改`my.cnf`配置文件。在`my.cnf`配置文件的`[mysqld]`后面加入以下语句：
```
default-storage-engine=存储引擎名称
```
然后保存就可以了。