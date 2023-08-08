---
title: MySQL 事务
date: 2020-05-15 19:32:08
tags: [MySQL]
categories: [数据库, MySQL]
---


# 数据库事务的概念和特性
数据库的事务（`Transaction`）是一种机制、一个操作序列，包含了一组数据库操作命令。事务把所有的命令作为一个整体一起向系统提交或撤销操作请求，即这一组数据库命令要么都执行，要么都不执行，因此事务是一个不可分割的工作逻辑单元。

在数据库系统上执行并发操作时，事务是作为最小的控制单元来使用的，特别适用于多用户同时操作的数据库系统。例如，航空公司的订票系统、银行、保险公司以及证券交易系统等。

事务具有 4 个特性，即原子性（`Atomicity`）、一致性（`Consistency`）、隔离性（`Isolation`）和持久性（`Durability`），这 4 个特性通常简称为`ACID`。
1. 原子性
事务是一个完整的操作。事务的各元素是不可分的。事务中的所有元素必须作为一个整体提交或回滚。如果事务中的任何元素失败，则整个事务将失败。
2. 一致性
当事务完成时，数据必须处于一致状态。也就是说，在事务开始之前，数据库中存储的数据处于一致状态。在正在进行的事务中，数据可能处于不一致的状态，如数据可能有部分被修改。然而，当事务成功完成时，数据必须再次回到已知的一致状态。通过事务对数据所做的修改不能损坏数据，或者说事务不能使数据存储处于不稳定的状态。
3. 隔离性
对数据进行修改的所有并发事务是彼此隔离的，这表明事务必须是独立的，它不应以任何方式依赖于或影响其他事务。修改数据的事务可以在另一个使用相同数据的事务开始之前访问这些数据，或者在另一个使用相同数据的事务结束之后访问这些数据。
另外，当事务修改数据时，如果任何其他进程正在同时使用相同的数据，则直到该事务成功提交之后，对数据的修改才能生效。
4. 持久性
事务的持久性指不管系统是否发生了故障，事务处理的结果都是永久的。
一个事务成功完成之后，它对数据库所作的改变是永久性的，即使系统出现故障也是如此。也就是说，一旦事务被提交，事务对数据所做的任何变动都会被永久地保留在数据库中。

事务的`ACID`原则保证了一个事务或者成功提交，或者失败回滚，二者必居其一。因此，它对事务的修改具有可恢复性。即当事务失败时，它对数据的修改都会恢复到该事务执行前的状态。
# 执行事务的语法和流程
MySQL 提供了多种存储引擎来支持事务。支持事务的存储引擎有 InnoDB 和 BDB，其中，InnoDB 存储引擎事务主要通过 UNDO 日志和 REDO 日志实现，MyISAM 存储引擎不支持事务。

为了维护 MySQL 服务器，经常需要在 MySQL 数据库中进行日志操作：
* UNDO 日志：复制事务执行前的数据，用于在事务发生异常时回滚数据。
* REDO 日志：记录在事务执行中，每条对数据进行更新的操作，当事务提交时，该内容将被刷新到磁盘。

默认设置下，每条 SQL 语句就是一个事务，即执行 SQL 语句后自动提交。为了达到将几个操作做为一个整体的目的，需要使用`BEGIN`或`START TRANSACTION`开启一个事务，或者禁止当前会话的自动提交。

SQL 使用下列语句来管理事务。
1. 开始事务
```
BEGIN;
或
START TRANSACTION;
```
这个语句显式地标记一个事务的起始点。
2. 提交事务
MySQL 使用下面的语句来提交事务：
```
COMMIT;
```
`COMMIT`表示提交事务，即提交事务的所有操作，具体地说，就是将事务中所有对数据库的更新都写到磁盘上的物理数据库中，事务正常结束。
提交事务，意味着将事务开始以来所执行的所有数据都修改成为数据库的永久部分，因此也标志着一个事务的结束。一旦执行了该命令，将不能回滚事务。只有在所有修改都准备好提交给数据库时，才执行这一操作。
3. 回滚（撤销）事务
MySQL 使用以下语句回滚事务：
```
ROLLBACK;
```
`ROLLBACK`表示撤销事务，即在事务运行的过程中发生了某种故障，事务不能继续执行，系统将事务中对数据库的所有已完成的操作全部撤销，回滚到事务开始时的状态。这里的操作指对数据库的更新操作。
当事务执行过程中遇到错误时，使用`ROLLBACK`语句使事务回滚到起点或指定的保持点处。同时，系统将清除自事务起点或到某个保存点所做的所有的数据修改，并且释放由事务控制的资源。因此，这条语句也标志着事务的结束。

`BEGIN`或`START TRANSACTION`语句后面的 SQL 语句对数据库数据的更新操作都将记录在事务日志中，直至遇到`ROLLBACK`语句或`COMMIT`语句。如果事务中某一操作失败且执行了`ROLLBACK`语句，那么在开启事务语句之后所有更新的数据都能回滚到事务开始前的状态。如果事务中的所有操作都全部正确完成，并且使用了`COMMIT`语句向数据库提交更新数据，则此时的数据又处在新的一致状态。
## 实例
下面模拟在张三的账户减少 500 元后，李四的账户还未增加 500 时，有其他会话访问数据表的场景。代码需要在两个窗口中执行，这里我们称为 A 窗口和 B 窗口。

1. 在 A 窗口中开启一个事务，并更新`mybank`数据库中`bank`表的数据：
```
mysql> USE mybank;
Database changed
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
mysql> UPDATE bank SET currentMoney = currentMoney-500 WHERE customerName='张三';
Query OK, 1 row affected (0.05 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
2. 在 B 窗口中查询`bank`数据表中的数据：
```
mysql> SELECT * FROM mybank.bank;
+--------------+--------------+
| customerName | currentMoney |
+--------------+--------------+
| 张三         |      1000.00 |
| 李四         |         1.00 |
+--------------+--------------+
```
从结果可以看出，虽然 A 窗口中的事务已经更改了`bank`表中的数据，但没有立即更新数据，这时其他会话读取到的仍然是更新前的数据。
3. 在 A 窗口中继续执行事务并提交事务：
```
mysql> UPDATE bank SET currentMoney = currentMoney+500
    -> WHERE customerName='李四';
Query OK, 1 row affected (0.05 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> COMMIT;
Query OK, 0 rows affected (0.07 sec)
```
4. 在 B 窗口中再次查询`bank`数据表的数据：
```
mysql> SELECT * FROM mybank.bank;
+--------------+--------------+
| customerName | currentMoney |
+--------------+--------------+
| 张三         |       500.00 |
| 李四         |       501.00 |
+--------------+--------------+
```
在 A 窗口中执行`COMMIT`提交事务后，对数据所做的更新将一起提交，其他会话读取到的是更新后的数据。从结果可以看出张三和李四的总账户余额和转账前保持一致，这样数据从一个一致性状态更新到另一个一致性状态。

当事务在执行中出现问题，也就是不能按正常的流程执行一个完整的事务时，可以使用`ROLLBACK`语句进行回滚，使用数据恢复到初始状态。

在上例中，张三的账户余额已经减少到 500 元，如果再转出 1000 元，将会出现余额为负数，因此需要回滚到原始状态。将张三的账户余额减少 1000 元，并让事务回滚：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
 
mysql> UPDATE bank SET currentMoney = currentMoney-1000 WHERE customerName='张三';
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0
 
mysql> ROLLBACK;
Query OK, 0 rows affected (0.07 sec)
 
mysql> SELECT * FROM mybank.bank;
+--------------+--------------+
| customerName | currentMoney |
+--------------+--------------+
| 张三         |       500.00 |
| 李四         |       501.00 |
+--------------+--------------+
```
从结果可以看出，执行事务回滚后，账户数据恢复到初始状态，即该事务执行之前的状态。

在数据库操作中，为了有效保证并发读取数据的正确性，提出了事务的隔离级别。在上例中，事务的隔离级别为默认隔离级别。在 MySQL 中，事务的默认隔离级别是`REPEATABLE-READ`（可重读）隔离级别，即事务未结束时（未执行`COMMIT`或`ROLLBACK`），其它会话只能读取到未提交数据。
## 注意事项
MySQL 事务是一项非常消耗资源的功能，在使用过程中要注意以下几点。
1. 事务尽可能简短
事务的开启到结束会在数据库管理系统中保留大量资源，以保证事务的原子性、一致性、隔离性和持久性。如果在多用户系统中，较大的事务将会占用系统的大量资源，使得系统不堪重负，会影响软件的运行性能，甚至导致系统崩溃。
2. 事务中访问的数据量尽量最少
当并发执行事务处理时，事务操作的数据量越少，事务之间对相同数据的操作就越少。
3. 查询数据时尽量不要使用事务
对数据进行浏览查询操作并不会更新数据库的数据，因此应尽量不使用事务查询数据，避免占用过量的系统资源。
4. 在事务处理过程中尽量不要出现等待用户输入的操作
在处理事务的过程中，如果需要等待用户输入数据，那么事务会长时间地占用资源，有可能造成系统阻塞。

# 设置事务自动提交（开启和关闭）
MySQL 默认开启事务自动提交模式，即除非显式的开启事务（`BEGIN`或`START TRANSACTION`），否则每条 SQL 语句都会被当做一个单独的事务自动执行。但有些情况下，我们需要关闭事务自动提交来保证数据的一致性。

在 MySQL 中，可以通过`SHOW VARIABLES`语句查看当前事务自动提交模式：
```
mysql> SHOW VARIABLES LIKE 'autocommit';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
```
结果显示，`autocommit`的值是`ON`，表示系统开启自动提交模式。

在 MySQL 中，可以使用`SET autocommit`语句设置事务的自动提交模式，语法格式如下：
```
SET autocommit = 0|1|ON|OFF;
```
对取值的说明：
* 值为 0 和值为`OFF`：关闭事务自动提交。如果关闭自动提交，用户将会一直处于某个事务中，只有提交或回滚后才会结束当前事务，重新开始一个新事务。
* 值为 1 和值为`ON`：开启事务自动提交。如果开启自动提交，则每执行一条 SQL 语句，事务都会提交一次。

## 示例
下面我们关闭事务自动提交，模拟银行转账。

使用`SET autocommit`语句关闭事务自动提交，且张三转给李四 500 元：
```sql
mysql> SET autocommit = 0;
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM mybank.bank;
+--------------+--------------+
| customerName | currentMoney |
+--------------+--------------+
| 张三         |      1000.00 |
| 李四         |         1.00 |
+--------------+--------------+

mysql> UPDATE bank SET currentMoney = currentMoney-500 WHERE customerName='张三' ;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> UPDATE bank SET currentMoney = currentMoney+500 WHERE customerName='李四';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
这时重新打开一个 cmd 窗口，查看`bank`数据表中张三和李四的余额：
```sql
mysql> SELECT * FROM mybank.bank;
+--------------+--------------+
| customerName | currentMoney |
+--------------+--------------+
| 张三         |      1000.00 |
| 李四         |         1.00 |
+--------------+--------------+
```
结果显示，张三和李四的余额是事务执行前的数据。

下面在之前的窗口中使用`COMMIT`语句提交事务，并查询`bank`数据表的数据：
```sql
mysql> COMMIT;
Query OK, 0 rows affected (0.07 sec)
mysql> SELECT * FROM mybank.bank;
+--------------+--------------+
| customerName | currentMoney |
+--------------+--------------+
| 张三         |       500.00 |
| 李四         |       501.00 |
+--------------+--------------+
```
结果显示，`bank`数据表的数据更新成功。

在本例中，关闭自动提交后，该位置会作为一个事务起点，直到执行`COMMIT`语句和`ROLLBACK`语句后，该事务才结束。结束之后，这就是下一个事务的起点。

关闭自动提交功能后，只用当执行`COMMIT`命令后，MySQL 才将数据表中的资料提交到数据库中。如果执行`ROLLBACK`命令，数据将会被回滚。如果不提交事务，而终止 MySQL 会话，数据库将会自动执行回滚操作。

使用`BEGIN`或`START TRANSACTION`开启一个事务之后，自动提交将保持禁用状态，直到使用`COMMIT`或`ROLLBACK`结束事务。之后，自动提交模式会恢复到之前的状态，即如果`BEGIN`前`autocommit = 1`，则完成本次事务后`autocommit`还是 1。如果`BEGIN`前`autocommit = 0`，则完成本次事务后`autocommit`还是 0。
# 事务隔离级别
其中事务的隔离性就是指当多个事务同时运行时，各事务之间相互隔离，不可互相干扰。如果事务没有隔离性，就容易出现脏读、不可重复读和幻读等情况。

为了保证并发时操作数据的正确性，数据库都会有事务隔离级别的概念。

1. 脏读
脏读是指一个事务正在访问数据，并且对数据进行了修改，但是这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
2. 不可重复读
不可重复读是指在一个事务内，多次读取同一个数据。
在这个事务还没有结束时，另外一个事务也访问了该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
3. 幻读
幻读是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。

为了解决以上这些问题，标准 SQL 定义了 4 类事务隔离级别，用来指定事务中的哪些数据改变是可见的，哪些数据改变是不可见的。

MySQL 包括的事务隔离级别如下：
* 读未提交（`READ UNCOMITTED`）
* 读提交（`READ COMMITTED`）
* 可重复读（`REPEATABLE READ`）
* 串行化（`SERIALIZABLE`）

MySQL 事务隔离级别可能产生的问题如下表所示：

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| :--: | :--: | :--: | :--: |
| READ UNCOMITTED | √ | √ | √ |
| READ COMMITTED | × | √ | √ |
| REPEATABLE READ | × | × | √ |
| SERIALIZABLE | × | × | × |

MySQL 的事务的隔离级别由低到高分别为`READ UNCOMITTED、READ COMMITTED、REPEATABLE READ、SERIALIZABLE`。低级别的隔离级别可以支持更高的并发处理，同时占用的系统资源更少。
## 1. 读未提交（READ UNCOMITTED，RU）
顾名思义，读未提交就是可以读到未提交的内容。

如果一个事务读取到了另一个未提交事务修改过的数据，那么这种隔离级别就称之为读未提交。

在该隔离级别下，所有事务都可以看到其它未提交事务的执行结果。因为它的性能与其他隔离级别相比没有高多少，所以一般情况下，该隔离级别在实际应用中很少使用。

下例展示了在读未提交隔离级别中产生的脏读现象。

1. 先在`test`数据库中创建`testnum`数据表，并插入数据。
```sql
mysql> CREATE TABLE testnum(num INT(4));
Query OK, 0 rows affected (0.57 sec)
mysql> INSERT INTO test.testnum (num) VALUES(1),(2),(3),(4),(5);
Query OK, 5 rows affected (0.09 sec)
 ```
2. 在 A 窗口中修改事务隔离级别，因为 A 窗口和 B 窗口的事务隔离级别需要保持一致，所以我们使用`SET GLOBAL TRANSACTION`修改全局变量。
```sql
mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
Query OK, 0 rows affected (0.04 sec)
flush privileges;
Query OK, 0 rows affected (0.04 sec)
 ```
查询事务隔离级别，SQL 语句和运行结果如下：
```
mysql> show variables like '%tx_isolation%'\G
*************************** 1. row ***************************
Variable_name: tx_isolation
        Value: READ-UNCOMMITTED
1 row in set, 1 warning (0.00 sec)
```
结果显示，现在 MySQL 的事务隔离级别为`READ-UNCOMMITTED`。
3. 在 A 窗口中开启一个事务，并查询`testnum`数据表：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
mysql> SELECT * FROM testnum;
+------+
| num  |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.00 sec)
```
4. 打开 B 窗口，查看当前 MySQL 的事务隔离级别：
```
mysql> show variables like '%tx_isolation%'\G
*************************** 1. row ***************************
Variable_name: tx_isolation
        Value: READ-UNCOMMITTED
1 row in set, 1 warning (0.00 sec)
```
确定事务隔离级别是`READ-UNCOMMITTED`后，开启一个事务，并使用`UPDATE`语句更新`testnum`数据表：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
mysql> UPDATE test.testnum SET num=num*2 WHERE num=2;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
5. 现在返回 A 窗口，再次查询`testnum`数据表：
```
mysql> SELECT * FROM testnum;
+------+
| num  |
+------+
|    1 |
|    4 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.02 sec)
```
由结果可以看出，A 窗口中的事务读取到了更新后的数据。
6. 下面在 B 窗口中回滚事务，SQL 语句和运行结果如下：
```
mysql> ROLLBACK;
Query OK, 0 rows affected (0.09 sec)
```
7. 在 A 窗口中查询`testnum`数据表：
```
mysql> SELECT * FROM testnum;
+------+
| num  |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.00 sec)
```
当 MySQL 的事务隔离级别为`READ UNCOMITTED`时，首先分别在 A 窗口和 B 窗口中开启事务，在 B 窗口中的事务更新但未提交之前， A 窗口中的事务就已经读取到了更新后的数据。但由于 B 窗口中的事务回滚了，所以 A 事务出现了脏读现象。

使用读提交隔离级别可以解决实例中产生的脏读问题。
## 2. 读提交（READ COMMITTED，RC）
顾名思义，读提交就是只能读到已经提交了的内容。

如果一个事务只能读取到另一个已提交事务修改过的数据，并且其它事务每对该数据进行一次修改并提交后，该事务都能查询得到最新值，那么这种隔离级别就称之为读提交。

该隔离级别满足了隔离的简单定义：一个事务从开始到提交前所做的任何改变都是不可见的，事务只能读取到已经提交的事务所做的改变。

这是大多数数据库系统的默认事务隔离级别（例如 Oracle、SQL Server），但不是 MySQL 默认的。

下例演示了在读提交隔离级别中产生的不可重复读问题。
1. 使用`SET`语句将 MySQL 事务隔离级别修改为`READ COMMITTED`，并查看。
```
mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
Query OK, 0 rows affected (0.00 sec)
mysql> show variables like '%tx_isolation%'\G
*************************** 1. row ***************************
Variable_name: tx_isolation
        Value: READ-COMMITTED
1 row in set, 1 warning (0.00 sec)
```
2. 确定当前事务隔离级别为`READ COMMITTED`后，开启一个事务：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
 ```
3. 在 B 窗口中开启事务，并使用`UPDATE`语句更新`testnum`数据表：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql>  UPDATE test.testnum SET num=num*2 WHERE num=2;
Query OK, 1 row affected (0.07 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
4. 在 A 窗口中查询`testnum`数据表：
```
mysql> SELECT * from test.testnum;
+------+
| num  |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.00 sec)
```
5. 提交 B 窗口中的事务：
```
mysql> COMMIT;
Query OK, 0 rows affected (0.07 sec)
```
6. 在 A 窗口中查询`testnum`数据表：
```
mysql> SELECT * from test.testnum;
+------+
| num  |
+------+
|    1 |
|    4 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.00 sec)
```
当 MySQL 的事务隔离级别为`READ COMMITTED`时，首先分别在 A 窗口和 B 窗口中开启事务，在 B 窗口中的事务更新并提交后，A 窗口中的事务读取到了更新后的数据。在该过程中，A 窗口中的事务必须要等待 B 窗口中的事务提交后才能读取到更新后的数据，这样就解决了脏读问题。而处于 A 窗口中的事务出现了不同的查询结果，即不可重复读现象。

使用可重复读隔离级别可以解决实例中产生的不可重复读问题。
## 3. 可重复读（REPEATABLE READ，RR）
顾名思义，可重复读是专门针对不可重复读这种情况而制定的隔离级别，可以有效的避免不可重复读。

在一些场景中，一个事务只能读取到另一个已提交事务修改过的数据，但是第一次读过某条记录后，即使其它事务修改了该记录的值并且提交，之后该事务再读该条记录时，读到的仍是第一次读到的值，而不是每次都读到不同的数据。那么这种隔离级别就称之为可重复读。

可重复读是 MySQL 的默认事务隔离级别，它能确保同一事务的多个实例在并发读取数据时，会看到同样的数据行。在该隔离级别下，如果有事务正在读取数据，就不允许有其它事务进行修改操作，这样就解决了可重复读问题。

下例演示了在可重复读隔离级别中产生的幻读问题。

1. 在`test`数据库中创建`testuser`数据表：
```
mysql> CREATE TABLE testuser(
    -> id INT (4) PRIMARY KEY,
    -> name VARCHAR(20));
Query OK, 0 rows affected (0.29 sec)
```
2. 使用`SET`语句修改事务隔离级别：
```
mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
Query OK, 0 rows affected (0.00 sec)
```
3. 在 A 窗口中开启事务，并查询`testuser`数据表：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM test.testuser where id=1;
Empty set (0.04 sec)
```
4. 在 B 窗口中开启一个事务，并向`testuser`表中插入一条数据：
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
mysql>  INSERT INTO test.testuser VALUES(1,'zhangsan');
Query OK, 1 row affected (0.04 sec)
mysql> COMMIT;
Query OK, 0 rows affected (0.06 sec)
```
5. 现在返回 A 窗口，向`testnum`数据表中插入数据：
```
mysql> INSERT INTO test.testuser VALUES(1,'lisi');
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
mysql>  SELECT * FROM test.testuser where id=1;
Empty set (0.00 sec)
```

使用串行化隔离级别可以解决实例中产生的幻读问题。
## 4. 串行化（SERIALIZABLE）
如果一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来。那么这种隔离级别就称之为串行化。

`SERIALIZABLE`是最高的事务隔离级别，主要通过强制事务排序来解决幻读问题。简单来说，就是在每个读取的数据行上加上共享锁实现，这样就避免了脏读、不可重复读和幻读等问题。但是该事务隔离级别执行效率低下，且性能开销也最大，所以一般情况下不推荐使用。
# 查看和修改事务隔离级别
## 查看事务隔离级别
在 MySQL 中，可以通过`show variables like '%tx_isolation%'`或`select @@tx_isolation;`语句来查看当前事务隔离级别。
```
mysql> show variables like '%tx_isolation%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
1 row in set, 1 warning (0.17 sec）
mysql> select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set, 1 warning (0.00 sec)
```
结果显示，目前 MySQL 的事务隔离级别是`REPEATABLE-READ`。

另外，还可以使用下列语句分别查询全局和会话的事务隔离级别：
```
SELECT @@global.tx_isolation;
SELECT @@session.tx_isolation;
```
> 提示：在 MySQL 8.0.3 中，`tx_isolation`变量被`transaction_isolation`变量替换了。在 MySQL 8.0.3 版本中查询事务隔离级别，只要把上述查询语句中的`tx_isolation`变量替换成`transaction_isolation`变量即可。
## 修改事务隔离级别
MySQL 提供了`SET TRANSACTION`语句，该语句可以改变单个会话或全局的事务隔离级别。语法格式如下：
```
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
```
其中，`SESSION`和`GLOBAL`关键字用来指定修改的事务隔离级别的范围：
* `SESSION`：表示修改的事务隔离级别将应用于当前`session`（当前 cmd 窗口）内的所有事务；
* `GLOBAL`：表示修改的事务隔离级别将应用于所有`session`（全局）中的所有事务，且当前已经存在的`session`不受影响；
* 如果省略`SESSION`和`GLOBAL`，表示修改的事务隔离级别将应用于当前`session`内的下一个还未开始的事务。

任何用户都能改变会话的事务隔离级别，但是只有拥有`SUPER`权限的用户才能改变全局的事务隔离级别。

如果使用普通用户修改全局事务隔离级别，就会提示需要超级权限才能执行此操作的错误信息，SQL 语句和运行结果如下：
```
C:\Users\leovo>mysql -utestuser -p
Enter password: ******
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 41
Server version: 5.7.29-log MySQL Community Server (GPL)
 
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
ERROR 1227 (42000): Access denied; you need (at least one of) the SUPER privilege(s) for this operation
mysql> SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
Query OK, 0 rows affected (0.00 sec)
```

使用`SET TRANSACTION`语句分别修改`session`和全局的事务隔离级别 SQL 语句和运行结果如下：
```
mysql>  select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| SERIALIZABLE           |
+------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
Query OK, 0 rows affected (0.00 sec)

mysql>  select @@global.tx_isolation;
+-----------------------+
| @@global.tx_isolation |
+-----------------------+
| REPEATABLE-READ       |
+-----------------------+
1 row in set, 1 warning (0.00 sec)
```
还可以使用`set tx_isolation`命令直接修改当前`session`的事务隔离级别，SQL 语句和运行结果如下：
```sql
mysql> set tx_isolation='READ-COMMITTED';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select @@session.tx_isolation;
+------------------------+
| @@session.tx_isolation |
+------------------------+
| READ-COMMITTED         |
+------------------------+
1 row in set, 1 warning (0.00 sec)
```