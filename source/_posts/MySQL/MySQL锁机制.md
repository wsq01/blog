---
title: MySQL 锁机制
date: 2020-05-19 21:09:17
tags: [MySQL]
categories: [数据库, MySQL]
---


为了保证数据并发访问时的一致性和有效性，任何一个数据库都存在锁机制。锁机制的优劣直接影响到数据库的并发处理能力和系统性能，所以锁机制也就成为了各种数据库的核心技术之一。

锁机制是为了解决数据库的并发控制问题而产生的。如在同一时刻，客户端对同一个表做更新或查询操作，为了保证数据的一致性，必须对并发操作进行控制。同时，锁机制也为实现 MySQL 的各个隔离级别提供了保证。

可以将锁机制理解为使各种资源在被并发访问时变得有序所设计的一种规则。

如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。

按锁级别分类，可分为共享锁、排他锁和意向锁。也可以按锁粒度分类，可分为行级锁、表级锁和页级锁。

1.共享锁
共享锁的代号是 S，是 Share 的缩写，也可称为读锁。是一种可以查看但无法修改和删除的数据锁。

共享锁的锁粒度是行或者元组（多个行）。一个事务获取了共享锁之后，可以对锁定范围内的数据执行读操作。会阻止其它事务获得相同数据集的排他锁。

2.排他锁
排他锁的代号是 X，是 eXclusive 的缩写，也可称为写锁，是基本的锁类型。

排他锁的粒度与共享锁相同，也是行或者元组。一个事务获取了排他锁之后，可以对锁定范围内的数据执行写操作。允许获得排他锁的事务更新数据，阻止其它事务取得相同数据集的共享锁和排他锁。

如有两个事务 A 和 B，如果事务 A 获取了一个元组的共享锁，事务 B 还可以立即获取这个元组的共享锁，但不能立即获取这个元组的排他锁，必须等到事务 A 释放共享锁之后才可以。

如果事务 A 获取了一个元组的排他锁，事务 B 不能立即获取这个元组的共享锁，也不能立即获取这个元组的排他锁，必须等到 A 释放排他锁之后才可以。
3.意向锁
为了允许行锁和表锁共存，实现多粒度锁机制，InnoDB 还有两种内部使用的意向锁。

意向锁是一种表锁，锁定的粒度是整张表，分为意向共享锁（IS）和意向排他锁（IX）两类。

意向共享锁表示一个事务有意对数据上共享锁或者排他锁。“有意”表示事务想执行操作但还没有真正执行。

锁和锁之间的关系，要么是相容的，要么是互斥的。
锁 a 和锁 b 相容是指：操作同样一组数据时，如果事务 t1 获取了锁 a，另一个事务 t2 还可以获取锁 b；
锁 a 和锁 b 互斥是指：操作同样一组数据时，如果事务 t1 获取了锁 a，另一个事务 t2 在 t1 释放锁 a 之前无法释放锁 b。

锁模式的兼容情况
其中共享锁、排他锁、意向共享锁、意向排他锁相互之间的兼容/互斥关系如下表所示，其中 Y 表示相容，N 表示互斥。

| 参数             | X |	S |	IX	|IS |
| :--: | :--: | :--: | :--: |:--: |
| X（排他锁）      | N |	N |	N	| N |
| S（共享锁）      | N |	Y |	N	| Y |
| IX（意向排他锁） | N | N |	Y	| Y |
| IS（意向共享锁） | N |	Y |	Y	| Y |

如果一个事务请求的锁模式与当前的锁兼容，InnoDB 就将请求的锁授予该事务；反之，如果两者不兼容，该事务就要等待锁释放。

为了尽可能提高数据库的并发量，需每次锁定的数据范围越小越好，越小的锁其耗费的系统资源越多，系统性能下降。为在高并发响应和系统性能两方面进行平衡，这样就产生了“锁粒度”的概念。
# 全局锁
要使用全局锁，则要执行这条命令：
```shell
flush tables with read lock
```
执行后，整个数据库就处于只读状态了，这时其他线程执行以下操作，都会被阻塞：
* 对数据的增删改操作，比如`insert、delete、update`等语句；
* 对表结构的更改操作，比如`alter table、drop table`等语句。

如果要释放全局锁，则要执行这条命令：
```
unlock tables
```
当然，当会话断开了，全局锁会被自动释放。
## 全局锁应用场景
全局锁主要应用于做全库逻辑备份，这样在备份数据库期间，不会因为数据或表结构的更新，而出现备份文件的数据与预期的不一样。

在全库逻辑备份期间，假设不加全局锁的场景，看看会出现什么意外的情况。

如果在全库逻辑备份期间，有用户购买了一件商品，一般购买商品的业务逻辑是会涉及到多张数据库表的更新，比如在用户表更新该用户的余额，然后在商品表更新被购买的商品的库存。

那么，有可能出现这样的顺序：
1. 先备份了用户表的数据；
2. 然后有用户发起了购买商品的操作；
3. 接着再备份商品表的数据。

也就是在备份用户表和商品表之间，有用户购买了商品。

这种情况下，备份的结果是用户表中该用户的余额并没有扣除，反而商品表中该商品的库存被减少了，如果后面用这个备份文件恢复数据库数据的话，用户钱没少，而库存少了，等于用户白嫖了一件商品。

所以，在全库逻辑备份期间，加上全局锁，就不会出现上面这种情况了。
## 加全局锁缺点
加上全局锁，意味着整个数据库都是只读状态。

那么如果数据库里有很多数据，备份就会花费很多的时间，关键是备份期间，业务只能读数据，而不能更新数据，这样会造成业务停滞。

既然备份数据库数据的时候，使用全局锁会影响业务，那有什么其他方式可以避免？

有的，如果数据库的引擎支持的事务支持可重复读的隔离级别，那么在备份数据库之前先开启事务，会先创建`Read View`，然后整个事务执行期间都在用这个`Read View`，而且由于 MVCC 的支持，备份期间业务依然可以对数据进行更新操作。

因为在可重复读的隔离级别下，即使其他事务更新了表的数据，也不会影响备份数据库时的`Read View`，这就是事务四大特性中的隔离性，这样备份期间备份的数据一直是在开启事务时的数据。

备份数据库的工具是`mysqldump`，在使用`mysqldump`时加上`–single-transaction`参数的时候，就会在备份数据库之前先开启事务。这种方法只适用于支持「可重复读隔离级别的事务」的存储引擎。

InnoDB 存储引擎默认的事务隔离级别正是可重复读，因此可以采用这种方式来备份数据库。

但是，对于 MyISAM 这种不支持事务的引擎，在备份数据库时就要使用全局锁的方法。
# 表锁、行锁和页锁
MySQL 按锁的粒度可以细分为行级锁、页级锁和表级锁。

我们可以将锁粒度理解成锁范围。

## 表级锁（table lock）
表级锁为表级别的锁定，会锁定整张表，可以很好的避免死锁，是 MySQL 中最大颗粒度的锁定机制。

一个用户在对表进行写操作（插入、删除、更新等）时，需要先获得写锁，这会阻塞其它用户对该表的所有读写操作。没有写锁时，其它读取的用户才能获得读锁，读锁之间是不相互阻塞的。

表级锁最大的特点就是实现逻辑非常简单，带来的系统负面影响最小。所以获取锁和释放锁的速度很快。当然，锁定颗粒度大带来最大的负面影响就是出现锁定资源争用的概率会很高，致使并发度大打折扣。

不过在某些特定的场景中，表级锁也可以有良好的性能。例如，`READ LOCAL`表级锁支持某些类型的并发写操作。另外，写锁也比读锁有更高的优先级，因此一个写锁请求可能会被插入到读锁队列的前面（写锁可以插入到锁队列中读锁的前面，反之读锁则不能插入到写锁的前面）。

使用表级锁的主要是 MyISAM，MEMORY，CSV 等一些非事务性存储引擎。

尽管存储引擎可以管理自己的锁，MySQL 本身还是会使用各种有效的表级锁来实现不同的目的。例如，服务器会为诸如`ALTER TABLE`之类的语句使用表级锁，而忽略存储引擎的锁机制。

MySQL 里面表级别的锁有这几种：
* 表锁；
* 元数据锁（MDL）;
* 意向锁；
* AUTO-INC 锁；

### 表锁
如果我们想对学生表（`t_student`）加表锁，可以使用下面的命令：
```sql
//表级别的共享锁，也就是读锁；
lock tables t_student read;

//表级别的独占锁，也就是写锁；
lock tables t_stuent write;
```
需要注意的是，表锁除了会限制别的线程的读写外，也会限制本线程接下来的读写操作。

也就是说如果本线程对学生表加了「共享表锁」，那么本线程接下来如果要对学生表执行写操作的语句，是会被阻塞的，当然其他线程对学生表进行写操作时也会被阻塞，直到锁被释放。

要释放表锁，可以使用下面这条命令，会释放当前会话的所有表锁：
```
unlock tables
```
另外，当会话退出后，也会释放所有表锁。

不过尽量避免在使用 InnoDB 引擎的表使用表锁，因为表锁的颗粒度太大，会影响并发性能，InnoDB 牛逼的地方在于实现了颗粒度更细的行级锁。
### 元数据锁（MDL）
我们不需要显示的使用 MDL，因为当我们对数据库表进行操作时，会自动给这个表加上 MDL：
* 对一张表进行 CRUD 操作时，加的是 MDL 读锁；
* 对一张表做结构变更操作的时候，加的是 MDL 写锁；

MDL 是为了保证当用户对表执行 CRUD 操作时，防止其他线程对这个表结构做了变更。

当有线程在执行`select`语句（ 加 MDL 读锁）的期间，如果有其他线程要更改该表的结构（申请 MDL 写锁），那么将会被阻塞，直到执行完`select`语句（释放 MDL 读锁）。

反之，当有线程对表结构进行变更（ 加 MDL 写锁）的期间，如果有其他线程执行了 CRUD 操作（申请 MDL 读锁），那么就会被阻塞，直到表结构变更完成（释放 MDL 写锁）。

MDL 不需要显示调用，那它是在什么时候释放的?

MDL 是在事务提交后才会释放，这意味着事务执行期间，MDL 是一直持有的。

那如果数据库有一个长事务（所谓的长事务，就是开启了事务，但是一直还没提交），那在对表结构做变更操作的时候，可能会发生意想不到的事情，比如下面这个顺序的场景：
1. 首先，线程 A 先启用了事务（但是一直不提交），然后执行一条`select`语句，此时就先对该表加上 MDL 读锁；
2. 然后，线程 B 也执行了同样的`select`语句，此时并不会阻塞，因为「读读」并不冲突；
3. 接着，线程 C 修改了表字段，此时由于线程 A 的事务并没有提交，也就是 MDL 读锁还在占用着，这时线程 C 就无法申请到 MDL 写锁，就会被阻塞，

那么在线程 C 阻塞后，后续有对该表的`select`语句，就都会被阻塞，如果此时有大量该表的`select`语句的请求到来，就会有大量的线程被阻塞住，这时数据库的线程很快就会爆满了。

为什么线程 C 因为申请不到 MDL 写锁，而导致后续的申请读锁的查询操作也会被阻塞？

这是因为申请 MDL 锁的操作会形成一个队列，队列中写锁获取优先级高于读锁，一旦出现 MDL 写锁等待，会阻塞后续该表的所有 CRUD 操作。

所以为了能安全的对表结构进行变更，在对表结构变更前，先要看看数据库中的长事务，是否有事务已经对表加上了 MDL 读锁，如果可以考虑`kill`掉这个长事务，然后再做表结构的变更。
### 意向锁
* 在使用 InnoDB 引擎的表里对某些记录加上「共享锁」之前，需要先在表级别加上一个「意向共享锁」；
* 在使用 InnoDB 引擎的表里对某些纪录加上「独占锁」之前，需要先在表级别加上一个「意向独占锁」；

也就是，当执行插入、更新、删除操作，需要先对表加上「意向独占锁」，然后对该记录加独占锁。

而普通的`select`是不会加行级锁的，普通的`select`语句是利用 MVCC 实现一致性读，是无锁的。

不过，`select`也是可以对记录加共享锁和独占锁的，具体方式如下：
```
//先在表上加上意向共享锁，然后对读取的记录加共享锁
select ... lock in share mode;

//先表上加上意向独占锁，然后对读取的记录加独占锁
select ... for update;
```
意向共享锁和意向独占锁是表级锁，不会和行级的共享锁和独占锁发生冲突，而且意向锁之间也不会发生冲突，只会和共享表锁（`lock tables ... read`）和独占表锁（`lock tables ... write`）发生冲突。

表锁和行锁是满足读读共享、读写互斥、写写互斥的。

如果没有「意向锁」，那么加「独占表锁」时，就需要遍历表里所有记录，查看是否有记录存在独占锁，这样效率会很慢。

那么有了「意向锁」，由于在对记录加独占锁前，先会加上表级别的意向独占锁，那么在加「独占表锁」时，直接查该表是否有意向独占锁，如果有就意味着表里已经有记录被加了独占锁，这样就不用去遍历表里的记录。

所以，意向锁的目的是为了快速判断表里是否有记录被加锁。
### AUTO-INC 锁
表里的主键通常都会设置成自增的，这是通过对主键字段声明`AUTO_INCREMENT`属性实现的。

之后可以在插入数据时，可以不指定主键的值，数据库会自动给主键赋值递增的值，这主要是通过 AUTO-INC 锁实现的。

AUTO-INC 锁是特殊的表锁机制，锁不是再一个事务提交后才释放，而是再执行完插入语句后就会立即释放。

在插入数据时，会加一个表级别的 AUTO-INC 锁，然后为被`AUTO_INCREMENT`修饰的字段赋值递增的值，等插入语句执行完成后，才会把 AUTO-INC 锁释放掉。

那么，一个事务在持有 AUTO-INC 锁的过程中，其他事务的如果要向该表插入语句都会被阻塞，从而保证插入数据时，被`AUTO_INCREMENT`修饰的字段的值是连续递增的。

但是， AUTO-INC 锁再对大量数据进行插入的时候，会影响插入性能，因为另一个事务中的插入会被阻塞。

因此， 在 MySQL 5.1.22 版本开始，InnoDB 存储引擎提供了一种轻量级的锁来实现自增。

一样也是在插入数据的时候，会为被`AUTO_INCREMENT`修饰的字段加上轻量级锁，然后给该字段赋值一个自增的值，就把这个轻量级锁释放了，而不需要等待整个插入语句执行完后才释放锁。

InnoDB 存储引擎提供了个`innodb_autoinc_lock_mode`的系统变量，是用来控制选择用 AUTO-INC 锁，还是轻量级的锁。

* 当`innodb_autoinc_lock_mode = 0`，就采用 AUTO-INC 锁，语句执行结束后才释放锁；
* 当`innodb_autoinc_lock_mode = 2`，就采用轻量级锁，申请自增主键后就释放锁，并不需要等语句执行后才释放。
* 当`innodb_autoinc_lock_mode = 1`：
 * 普通`insert`语句，自增锁在申请之后就马上释放；
 * 类似`insert … select`这样的批量插入数据的语句，自增锁还是要等语句结束后才被释放；

举个例子，考虑下面场景：

| session A | session B |
| :--: | :--: |
| insert into t values(null, 1, 1);<br>insert into t values(null, 2, 2);<br>insert into t values(null, 3, 3);<br>insert into t values(null, 4, 4); | |
| | create table t2 like t; |
| insert into t2 values(null, 5, 5); | insert into t2(c, d) select c,d from t; |

`session A`往表 t 中插入了 4 行数据，然后创建了一个相同结构的表`t2`，然后两个`session`同时执行向表`t2`中插入数据。

如果`innodb_autoinc_lock_mode = 2`，意味着「申请自增主键后就释放锁，不必等插入语句执行完」。那么就可能出现这样的情况：
* `session B`先插入了两个记录，`(1,1,1)、(2,2,2)`；
* 然后，`session A`来申请自增 id 得到`id=3`，插入了`(3,5,5)`；
* 之后，`session B`继续执行，插入两条记录`(4,3,3)、(5,4,4)`。
  
可以看到，`session B`的`insert`语句，生成的`id`不连续。

当「主库」发生了这种情况，`binlog`面对`t2`表的更新只会记录这两个`session`的`insert`语句，如果`binlog_format=statement`，记录的语句就是原始语句。记录的顺序要么先记`session A`的`insert`语句，要么先记`session B`的`insert`语句。

但不论是哪一种，这个`binlog`拿去「从库」执行，这时从库是按「顺序」执行语句的，只有当执行完一条 SQL 语句后，才会执行下一条 SQL。因此，在从库上「不会」发生像主库那样两个`session`「同时」执行向表`t2`中插入数据的场景。所以，在备库上执行了`session B`的`insert`语句，生成的结果里面，id 都是连续的。这时，主从库就发生了数据不一致。

要解决这问题，`binlog`日志格式要设置为`row`，这样在`binlog`里面记录的是主库分配的自增值，到备库执行的时候，主库的自增值是什么，从库的自增值就是什么。

所以，当`innodb_autoinc_lock_mode = 2`时，并且`binlog_format = row`，既能提升并发性，又不会出现数据一致性问题。
## 页级锁（page lock）
页级锁是 MySQL 中比较独特的一种锁定级别，在其他数据库管理软件中并不常见。

页级锁的颗粒度介于行级锁与表级锁之间，所以获取锁定所需要的资源开销，以及所能提供的并发处理能力同样也是介于上面二者之间。另外，页级锁和行级锁一样，会发生死锁。

页级锁主要应用于 BDB 存储引擎。
## 行级锁（row lock）
行级锁的锁定颗粒度在 MySQL 中是最小的，只针对操作的当前行进行加锁，所以行级锁发生锁定资源争用的概率也最小。

行级锁能够给予应用程序尽可能大的并发处理能力，从而提高需要高并发应用系统的整体性能。虽然行级锁在并发处理能力上面有较大的优势，但也因此带来了不少弊端。

由于锁定资源的颗粒度很小，所以每次获取锁和释放锁需要做的事情也就更多，带来的消耗自然也就更大。此外，行级锁也最容易发生死锁。所以说行级锁最大程度地支持并发处理的同时，也带来了最大的锁开销。

行级锁主要应用于 InnoDB 存储引擎。

随着锁定资源颗粒度的减小，锁定相同数据量的数据所需要消耗的内存数量也越来越多，实现算法也会越来越复杂。不过，随着锁定资源颗粒度的减小，应用程序的访问请求遇到锁等待的可能性也会随之降低，系统整体并发度也会随之提升。

MySQL 这 3 种锁的特性可大致归纳如下：

|  	      | 表级锁       | 行级锁      | 页级锁 |
| :--: | :--: | :--: | :--: |
| 开销    | 小           | 大          | 介于表级锁和行级锁之间 |
| 加锁    | 快           | 慢          | 介于表级锁和行级锁之间 |
| 死锁    | 不会出现死锁 |  会出现死锁| 	会出现死锁 |
| 锁粒度  | 大           | 小          | 介于表级锁和行级锁之间 |
| 并发度  | 低           | 高          | 一般 |

从上述特点可见，很难笼统的说哪种锁更好，只能具体应用具体分析。

从锁的角度来说，表级锁适合以查询为主，只有少量按索引条件更新数据的应用，如 Web 应用。而行级锁更适合于有大量按索引条件，同时又有并发查询的应用，如一些在线事务处理（OLTP）系统。
# InnoDB行锁
在 MySQL 中，InnoDB 行锁通过给索引上的索引项加锁来实现，如果没有索引，InnoDB 将通过隐藏的聚簇索引来对记录加锁。

InnoDB 支持 3 种行锁定方式：
* 行锁（`Record Lock`）：直接对索引项加锁。
* 间隙锁（`Gap Lock`）：锁加在索引项之间的间隙，也可以是第一条记录前的“间隙”或最后一条记录后的“间隙”。
* `Next-Key Lock`：行锁与间隙锁组合起来用就叫做`Next-Key Lock`。前两种的组合，对记录及其前面的间隙加锁。

默认情况下，InnoDB 工作在可重复读（默认隔离级别）下，并且以`Next-Key Lock`的方式对数据行进行加锁，这样可以有效防止幻读的发生。

`Next-Key Lock`是行锁与间隙锁的组合，这样，当 InnoDB 扫描索引项的时候，会首先对选中的索引项加上行锁（`Record Lock`），再对索引项两边的间隙（向左扫描扫到第一个比给定参数小的值， 向右扫描扫到第一个比给定参数大的值， 然后以此为界，构建一个区间）加上间隙锁（`Gap Lock`）。如果一个间隙被事务 T1 加了锁，其它事务不能在这个间隙插入记录。
 
要禁止间隙锁的话，可以把隔离级别降为读已提交（`READ COMMITTED`），或者开启参数`innodb_locks_unsafe_for_binlog`。

注意：以上语句描述的情况，与 MySQL 所设置的事务隔离级别有较大的关系。

开启一个事务时，InnoDB 存储引擎会在更新的记录上加行级锁，此时其它事务不可以更新被锁定的记录。下面我们以示例1演示此过程。

下面的语句需要在两个命令行窗口中执行。为了方便理解，我们分别称之为 A 窗口和 B 窗口。

分别在 A 窗口和 B 窗口中查看事务隔离级别，A 窗口和 B 窗口的事务隔离级别需要保持一致。

A 窗口查看隔离级别的 SQL 语句和运行结果如下所示：
```
mysql> SHOW VARIABLES LIKE 'tx_isolation' \G
*************************** 1. row ***************************
Variable_name: tx_isolation
        Value: REPEATABLE-READ
1 row in set, 1 warning (0.03 sec)
B 窗口查看隔离级别 SQL 语句和运行结果如下所示：
mysql> SHOW VARIABLES LIKE 'tx_isolation' \G
*************************** 1. row ***************************
Variable_name: tx_isolation
        Value: REPEATABLE-READ
1 row in set, 1 warning (0.03 sec)
```
结果显示，A 窗口和 B 窗口的事务隔离级别都为`REPEATABLE-READ`。

在 A 窗口中开启一个事务，并修改`tb_student`表，SQL 语句和运行结果如下：
```sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE test.tb_student SET age ='30' WHERE id = 1;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
在 B 窗口中也开启一个事务，并修改`tb_student`表，SQL 语句和运行结果如下：
```sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)
mysql> UPDATE test.tb_student SET age ='30' WHERE id = 1;
```
会发现 UPDATE 语句一直在执行。这时我们在 A 窗口中提交事务。
```sql
mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)
```
这时我们发现 B 窗口中的 UPDATE 语句执行成功。
```sql
mysql> UPDATE test.tb_student SET age ='30' WHERE id = 1;
Query OK, 0 rows affected (1 min 2.78 sec)
Rows matched: 1  Changed: 0  Warnings: 0
```
查询`tb_student`表中的数据，SQL 语句和运行结果如下：
```sql
mysql> SELECT * FROM test.tb_student;
+----+------+------+------+------+
| id | name | age  | sex  | num  |
+----+------+------+------+------+
|  1 | 张三 |   30 | 男   |    4 |
|  2 | 李四 |   12 | 男   |    4 |
|  3 | 王五 |   13 | 女   |    4 |
|  4 | 张四 |   13 | 女   |    4 |
|  5 | 王四 |   15 | 男   |    4 |
|  6 | 赵六 |   12 | 女   |    4 |
+----+------+------+------+------+
6 rows in set (0.00 sec)
```
如以上实例所示，当有不同的事务同时更新同一条记录时，另外一个事务需要等待另一个事务把锁释放，此时查看 MySQL 中 InnoDB 存储引擎的状态如下：
```
mysql> SHOW ENGINE innodb status \G
......
------------
TRANSACTIONS
------------
Trx id counter 19556
Purge done for trx's n:o < 19554 undo n:o < 0 state: running but idle
History list length 12
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 283572223909376, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 19555, ACTIVE 54 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 2 lock struct(s), heap size 1136, 1 row lock(s)
MySQL thread id 14, OS thread handle 4568, query id 886 localhost ::1 root updating
UPDATE test.tb_student SET age ='30' WHERE id = 1
------- TRX HAS BEEN WAITING 54 SEC FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 197 page no 3 n bits 80 index PRIMARY of table `test`.`tb_student` trx id 19555 lock_mode X locks rec but not gap waiting
Record lock, heap no 2 PHYSICAL RECORD: n_fields 7; compact format; info bits 0
0: len 4; hex 80000001; asc     ;;
1: len 6; hex 000000004c62; asc     Lb;;
```
从上面运行结果可以看出，SQL 语句`UPDATE test.tb_student SET age ='30' WHERE id = 1`在等待，```RECORD LOCKS space id 197 page no 3 n bits 80 index PRIMARY of table `test`.`tb_student` trx id 19555 lock_mode X locks rec but not gap```表示锁住的资源，`locks rec but not gap`代表锁住的是一个索引，不是一个范围。

`“MySQL thread id 14, OS thread handle 4568, query id 886 localhost ::1 root updating”`表示第 2 个事务连接的 ID 为 14，当前状态为正在更新，同时正在更新的记录需要等待其它事务将锁释放。当超过事务等待锁允许的最大时间，此时会提示`“ERROR 1205(HY000):Lock wait timeout exceeded; try restarting transaction"`及当前事务执行失败，则自动执行回滚操作。

MySQL 数据库采用 InnoDB 模式，默认参数`innodb_lock_wait_timeout`设置锁等待的时间是 50s，一旦数据库锁超过这个时间就会报错。可通过以下命令查看当前数据库锁等待的时间。
```
mysql> SHOW GLOBAL VARIABLES LIKE 'innodb_lock_wait_timeout';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 120   |
+--------------------------+-------+
1 row in set, 1 warning (0.02 sec)
```
下面演示了 InnoDB 间隙锁的实现机制。

下面在保证 A 窗口和 B 窗口的前提下，将`tb_student`表中的 id 字段设为外键，并开启一个事务，修改`tb_student`表中`id`为 1 的`age`。SQL 语句和运行结果如下：
```sql
mysql> ALTER TABLE test.tb_student ADD unique key idx_id(id);
Query OK, 0 rows affected (0.17 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE test.tb_student SET age ='31' WHERE id = 1;
Query OK, 0 rows affected (0.01 sec)
Rows matched: 1  Changed: 0  Warnings: 0
```
在 B 窗口中开启一个事务，修改 tb_student 表中 id 为 2 的 age，SQL 语句和运行结果如下：
```sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql>  UPDATE test.tb_student SET age ='28'WHERE id=2;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
这时分别提交 A窗口和 B窗口的事务。
```
mysql> COMMIT;
Query OK, 0 rows affected (0.01 sec)
```
查询 tb_student 表的数据，SQL 语句和运行结果如下：
```
mysql> SELECT * FROM test.tb_student;
+----+------+------+------+------+
| id | name | age  | sex  | num  |
+----+------+------+------+------+
|  1 | 张三 |   31 | 男   |    4 |
|  2 | 李四 |   28 | 男   |    4 |
|  3 | 王五 |   13 | 女   |    4 |
|  4 | 张四 |   13 | 女   |    4 |
|  5 | 王四 |   15 | 男   |    4 |
|  6 | 赵六 |   12 | 女   |    4 |
+----+------+------+------+------+
6 rows in set (0.00 sec)
```
在上述示例中，由于 InnoDB 行级锁为间隙锁，只锁定需要的记录，因此 B 窗口中的事务可以更新其它记录，两个事务之间互不影响。
# 锁等待和死锁
使用数据库时，有时会出现死锁。对于实际应用来说，就是出现系统卡顿。

死锁是指两个或两个以上的事务在执行过程中，因争夺资源而造成的一种互相等待的现象。就是所谓的锁资源请求产生了回路现象，即死循环，此时称系统处于死锁状态或系统产生了死锁。常见的报错信息为“Deadlock found when trying to get lock...”。

死锁发生以后，只有部分或完全回滚其中一个事务，才能打破死锁。多数情况下只需要重新执行因死锁回滚的事务即可。下面我们通过一个实例来了解死锁是如何产生的。

```
mysql> SELECT * FROM tb_student;
+----+------+------+------+------+
| id | name | age  | sex  | num  |
+----+------+------+------+------+
|  1 | 张三 |   31 | 男   |    4 |
|  2 | 李四 |   28 | 男   |    4 |
|  3 | 王五 |   13 | 女   |    4 |
|  4 | 张四 |   13 | 女   |    4 |
|  5 | 王四 |   15 | 男   |    4 |
|  6 | 赵六 |   12 | 女   |    4 |
+----+------+------+------+------+
6 rows in set (0.01 sec)

mysql> DESC tb_student;
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | int(4)      | NO   | PRI | NULL    | auto_increment |
| name  | varchar(25) | NO   |     | NULL    |                |
| age   | int(11)     | YES  | MUL | NULL    |                |
| sex   | char(1)     | YES  |     | NULL    |                |
| num   | int(11)     | YES  |     | NULL    |                |
+-------+-------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```
以下操作需要打开两个会话窗口，即下面所提到的 A 窗口和 B 窗口。

在 A 窗口中执行以下命令：
```sql
mysql> BEGIN;
mysql> UPDATE tb_student SET num=5 WHERE age=13;
Query OK, 2 rows affected (0.04 sec)
Rows matched: 2  Changed: 2  Warnings: 0
```
紧接着在 B 窗口中执行以下命令。由于`age`是索引字段，与 A 窗口中更新的是不同行的数据，所以这时不会出现锁等待现象。
```sql
mysql> BEGIN;
mysql> UPDATE tb_student SET num=8 WHERE age=15;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```
然后在 A 窗口中，执行以下命令，这时就会出现锁等待现象了。
```sql
mysql> UPDATE tb_student SET num=10 WHERE age=15;
```
最后在 B 窗口中，执行以下命令，这时会出现相互等待资源的现象，也就是死锁现象。
```sql
mysql> UPDATE tb_student SET num=12 WHERE age=13;
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```
我们可以通过`SHOW ENGINE INNODB STATUS`命令查看死锁的信息，运行结果如下（这里只展示了部分信息）：
```
LATEST DETECTED DEADLOCK
------------------------
2020-08-24 16:22:23 0x3944
*** (1) TRANSACTION:
TRANSACTION 22656, ACTIVE 108 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 5 lock struct(s), heap size 1136, 6 row lock(s), undo log entries 2
MySQL thread id 33, OS thread handle 8808, query id 1689 localhost ::1 root updating
UPDATE tb_student SET num=10 WHERE age=15
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 197 page no 8 n bits 80 index index_age of table `test`.`tb_student` trx id 22656 lock_mode X waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
0: len 4; hex 8000000f; asc     ;;
1: len 4; hex 80000005; asc     ;;
......
```
通过以上日志，我们就能确定造成死锁的事务和 SQL 语句。
## 死锁检测
InnoDB 的并发写操作会触发死锁，同时 InnoDB 也提供了死锁检测机制。通过设置`innodb_deadlock_detect`参数的值来控制是否打开死锁检测。
* `innodb_deadlock_detect = ON`：默认值，打开死锁检测。数据库发生死锁时，系统会自动回滚其中的某一个事务，让其它事务可以继续执行。
* `innodb_deadlock_detect = OFF`：关闭死锁检测。发生死锁时，系统会用锁等待来处理。

锁等待是指在事务过程中产生的锁，其它事务需要等待上一个事务释放锁，才能占用该资源。如果该事务一直不释放，就需要持续等待下去，直到超过了锁等待时间。当超过锁等待允许的最大时间，就会出现死锁，然后当前事务执行失败，自动执行回滚操作。

MySQL 通过`innodb_lock_wait_timeout`参数控制锁等待的时间，单位是秒。
```
mysql> SHOW VARIABLES LIKE '%innodb_lock_wait%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_lock_wait_timeout | 120   |
+--------------------------+-------+
1 row in set, 1 warning (0.02 sec)
```
在实际应用中，我们要尽量防止锁等待现象的发生，下面介绍几种避免死锁的方法：
* 如果不同程序会并发存取多个表，或者涉及多行记录时，尽量约定以相同的顺序访问表，这样可以大大降低死锁的发生。
* 业务中要及时提交或者回滚事务，可减少死锁产生的概率。
* 在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率。
* 对于非常容易产生死锁的业务部分，可以尝试使用升级锁粒度，通过表锁定来减少死锁产生的概率（表级锁不会产生死锁）。

# 锁监控
通常情况下，当出现锁问题时，我们习惯性通过`SHOW FULL PROCESSLIST`和`SHOW ENGINE INNODB STATUS`命令来判断事务中锁问题的情况。其实还有特别重要的三张表，即在`information_schema`数据库下的`innodb_trx、innodb_locks`和`innodb_lock_waits`表。 这三张表可以更方便地来帮助我们监控当前的事务并分析可能存在的锁问题。

下面通过实例来逐一了解一下这三张表。

在 A窗口中，开启一个事务，在查询 tb_student 表字段 age<15 的语句上加一个写锁，SQL 命令如下：
```sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM test.tb_student WHERE age<15 FOE UPDATE;
+----+------+------+------+------+
| id | name | age  | sex  | num  |
+----+------+------+------+------+
|  3 | 王五 |   13 | 女   |   12 |
|  4 | 张四 |   13 | 女   |   12 |
|  6 | 赵六 |   12 | 女   |    4 |
+----+------+------+------+------+
3 rows in set (0.02 sec)
```
在 B 窗口中开启一个事务，在`tb_student`表中插入`age=14`的记录，出现锁等待超时。
```
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO tb_student(name,age) VALUES ('dd',14);
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```
我们通过开始提到的三张表来分析出现的锁等待问题。

查询`innodb_trx`表，SQL 语句和运行结果如下：
```
mysql> SELECT * FROM information_schema.innodb_trx \G
*************************** 1. row ***************************
                    trx_id: 22694
                 trx_state: LOCK WAIT
               trx_started: 2019-08-25 09:17:26
     trx_requested_lock_id: 22694:197:3:1
          trx_wait_started: 2019-08-25 09:17:26
                trx_weight: 2
       trx_mysql_thread_id: 42
                 trx_query: INSERT INTO tb_student(name,age) VALUES ('dd',14)
       trx_operation_state: inserting
         trx_tables_in_use: 1
         trx_tables_locked: 1
          trx_lock_structs: 2
     trx_lock_memory_bytes: 1136
           trx_rows_locked: 1
         trx_rows_modified: 0
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
trx_adaptive_hash_latched: 0
trx_adaptive_hash_timeout: 0
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
*************************** 2. row ***************************
                    trx_id: 22693
                 trx_state: RUNNING
               trx_started: 2019-08-25 09:17:17
     trx_requested_lock_id: NULL
          trx_wait_started: NULL
                trx_weight: 2
       trx_mysql_thread_id: 41
                 trx_query: select * FROM information_schema.innodb_trx
       trx_operation_state: NULL
         trx_tables_in_use: 0
         trx_tables_locked: 1
          trx_lock_structs: 2
     trx_lock_memory_bytes: 1136
           trx_rows_locked: 7
         trx_rows_modified: 0
   trx_concurrency_tickets: 0
       trx_isolation_level: REPEATABLE READ
         trx_unique_checks: 1
    trx_foreign_key_checks: 1
trx_last_foreign_key_error: NULL
trx_adaptive_hash_latched: 0
trx_adaptive_hash_timeout: 0
          trx_is_read_only: 0
trx_autocommit_non_locking: 0
2 rows in set (0.01 sec)
```
以上各列含义说明如下：

| 列名                  | 描述 |
| :--: | :--: |
| trx_id               | 唯一的事务 id 号。本例为 22694 和 22693 |
| trx_state            | 当前事务的状态。本例中 22694 事务号是 lock_wait 锁等待状态 |
| trx_wait_started     | 事务开始等待的时间。本例为 2019-08-25 09:17:26 |
| trx_mysql_thread_id  | 线程 id，与  SHOW FULL PROCESSLIST 相对应。本例为 42 |
| trx_query            | 事务运行的 SQL 语句，本例为 INSERT INTO tb_student(name,age) VALUES ('dd',14)。 |
| trx_operation_state  | 事务运行的状态。本例为 inserting。 |

使用`SHOW FULL PROCESSLIST`语句查看当前线程处理情况，通常用来处理突发事件，返回的结果是实时变化的。
```
mysql> SHOW FULL PROCESSLIST;
+----+------+-----------------+------+---------+------+----------+---------------------------------------------------+
| id | User | Host            | db   | Command | Time | State    | Info                                              |
+----+------+-----------------+------+---------+------+----------+---------------------------------------------------+
| 35 | root | localhost:64579 | test | Sleep   | 1772 |          | NULL                                              |
| 36 | root | localhost:64582 | NULL | Sleep   | 1775 |          | NULL                                              |
| 45 | root | localhost:64933 | test | Query   |    0 | starting | SHOW FULL PROCESSLIST                             |
| 46 | root | localhost:64934 | test | Query   |    8 | update   | INSERT INTO tb_student(name,age) VALUES ('dd',14) |
+----+------+-----------------+------+---------+------+----------+---------------------------------------------------+
4 rows in set (0.00 sec)
```
以上各列含义说明如下：

| 列名     | 描述 |
| :--: | :--: |
| id      | 一个标识，kill 有问题的线程时使用 |
| user    | 显示当前用户，如果不是 root，这个命令就只显示你权限范围内的 SQL 语句 |
| host    | 显示这个语句是从哪个 ip 的哪个端口上发出的，可以用来追踪出问题语句的用户 |
| db      | 显示这个进程目前连接的是哪个数据库 |
| command	| 显示当前连接的执行命令，一般就是休眠（sleep），查询（query），连接（connect） |
| time    | 这个状态持续的时间，单位是秒 |
| state   | 显示使用当前连接的 SQL 语句的状态 |
| info    | 显示这个 SQL 语句，因为长度有限，所以长的 SQL 语句就会显示不全，是判断问题语句的重要依据 |

下面通过`innodb_lock_waits`和`innodb_locks`两张表来判断持有锁和锁等待的对象。本例中 22696 是锁等待的对象，22695 是持有锁的对象。

`innodb_lock_waits`表包含每个被阻止 InnoDB 事务的一个或多个行，指示它已请求的锁以及阻止该请求的任何锁。
```
mysql> SELECT * FROM information_schema.innodb_lock_waits \G
*************************** 1. row ***************************
requesting_trx_id: 22696
requested_lock_id: 22696:197:3:1
  blocking_trx_id: 22695
blocking_lock_id: 22695:197:3:1
1 row in set, 1 warning (0.00 sec)
```
以上各列含义说明如下：

| 列名               | 描述 |
| :--: | :--: |
| requesting_trx_id	| 请求（阻止）事务的 id |
| requested_lock_id	| 事务正在等待的锁的id |
| blocking_trx_id   | 阻止事务的 id |
| blocking_lock_id	| 阻止另一个事务继续进行的事务所持有的锁的 id |
| innodb_locks      | 表提供有关 InnoDB 事务已请求但尚未获取的每个锁的信息，以及事务持有的阻止另一个事务的锁。 |

```
mysql> SELECT * FROM information_schema.innodb_locks \G
*************************** 1. row ***************************
    lock_id: 22696:197:3:1
lock_trx_id: 22696
  lock_mode: X
  lock_type: RECORD
lock_table: `test`.`tb_student`
lock_index: PRIMARY
lock_space: 197
  lock_page: 3
   lock_pec: 1
  lock_data: supremum pseudo-record
*************************** 2. row ***************************
    lock_id: 22695:197:3:1
lock_trx_id: 22695
  lock_mode: X
  lock_type: RECORD
lock_table: `test`.`tb_student`
lock_index: PRIMARY
lock_space: 197
  lock_page: 3
   lock_pec: 1
  lock_data: supremum pseudo-record
2 rows in set, 1 warning (0.00 sec)
```
以上各列含义说明如下：

| 列名	      | 描述 |
| :--: | :--: |
| lock_id	    | 一个唯一的锁 id 号，内部为 InnoDB |
| lock_trx_id	| 持有锁的交易的 id |
| lock_mode	  | 如何请求锁定。允许锁定模式描述符 S，X， IS，IX， GAP，AUTO_INC 和 UNKNOWN。锁定模式描述符可以组合使用以识别特定的锁定模式。 |
| lock_type	  | 锁的类型 |
| lock_table	| 已锁定或包含锁定记录的表的名称 |
| lock_index	| 索引的名称，如果 lock_type 是 RECORD，否则 NULL |
| lock_space	| 锁定记录的表空间 id，如果 lock_type 是 RECORD，否则 NULL |
| lock_page	  | 锁定记录的页码，如果 lock_type 是 RECORD，否则 NULL。 |
| lock_pec	  | 页面内锁定记录的堆号，如果 lock_type 是 RECORD，否则 NULL。 |
| lock_data	  | 与锁相关的数据。 |

如果`lock_type`是`RECORD`，是锁定的记录的主键值，否则`NULL`。此列包含锁定行中主键列的值，格式为有效的 SQL 字符串。如果没有主键，`lock_data`则是唯一的 InnoDB 内部行 id 号。如果对键值或范围高于索引中的最大值的间隙锁定，则`lock_data`报告`supremum pseudo-record`。当包含锁定记录的页面不在缓冲池中时（如果在保持锁定时将其分页到磁盘），InnoDB 不从磁盘获取页面，以避免不必要的磁盘操作。相反，`lock_data`设置为`NULL`。