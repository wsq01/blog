---
title: SQL入门
date: 2020-02-03 11:02:21
tags: [SQL]
categories: [数据库, SQL]
---

我们可以把 SQL 语言按照功能划分成 4 个部分：
* DDL（`Data Definition Language`），也就是数据定义语言，它用来定义我们的数据库对象，包括数据库、数据表和列。通过使用 DDL，我们可以创建，删除和修改数据库和表结构。
* DML（`Data Manipulation Language`），数据操作语言，我们用它操作和数据库相关的记录，比如增加、删除、修改数据表中的记录。
* DCL（`Data Control Language`），数据控制语言，我们用它来定义访问权限和安全级别。
* DQL（`Data Query Language`），数据查询语言，我们用它查询想要的记录。

# DB、DBS 和 DBMS 的区别是什么
DBMS（`DataBase Management System`），数据库管理系统，实际上它可以对多个数据库进行管理，所以你可以理解为 DBMS = 多个数据库（DB） + 管理程序。

DB（`DataBase`），也就是数据库。数据库是存储数据的集合，你可以把它理解为多个数据表。

DBS（`DataBase System`），数据库系统。它是更大的概念，包括了数据库、数据库管理系统以及数据库管理人员 DBA。

这里需要注意的是，虽然我们有时候把 Oracle、MySQL 等称之为数据库，但确切讲，它们应该是数据库管理系统，即 DBMS。
## 常见DBMS

| 数据库管理系统 | 数据库模式 |
| :--: | :--: |
| Oracle | 关系型 |
| MySQL | 关系型 |
| Microsoft SQL Server | 关系型 |
| MongoDB | 文档型 |
| Elasticsearch | 搜索引擎 |
| Redis | 键值型 |
| HBase | 列存储 |

关系型数据库（RDBMS）就是建立在关系模型基础上的数据库，SQL 就是关系型数据库的查询语言。

相比于 SQL，NoSQL 泛指非关系型数据库，包括了键值型数据库、文档型数据库、搜索引擎和列存储等，除此以外还包括图形数据库。

键值型数据库通过`Key-Value`键值的方式来存储数据，其中`Key`和`Value`可以是简单的对象，也可以是复杂的对象。`Key`作为唯一的标识符，优点是查找速度快，在这方面明显优于关系型数据库，同时缺点也很明显，它无法像关系型数据库一样自由使用条件过滤（比如`WHERE`），如果你不知道去哪里找数据，就要遍历所有的键，这就会消耗大量的计算。键值型数据库典型的使用场景是作为内容缓存。Redis 是最流行的键值型数据库。

文档型数据库用来管理文档，在数据库中文档作为处理信息的基本单位，一个文档就相当于一条记录，MongoDB 是最流行的文档型数据库。

搜索引擎也是数据库检索中的重要应用，常见的全文搜索引擎有 Elasticsearch、Splunk 和 Solr。虽然关系型数据库采用了索引提升检索效率，但是针对全文索引效率却较低。搜索引擎的优势在于采用了全文搜索的技术，核心原理是“倒排索引”。

列式数据库是相对于行式存储的数据库，Oracle、MySQL、SQL Server 等数据库都是采用的行式存储，而列式数据库是将数据按照列存储到数据库中，这样做的好处是可以大量降低系统的 I/O，适合于分布式文件系统，不足在于功能相对有限。

图形数据库，利用了图这种数据结构存储了实体（对象）之间的关系。最典型的例子就是社交网络中人与人的关系，数据模型主要是以节点和边（关系）来实现，特点在于能高效地解决复杂的关系问题。
# SQL是如何执行的
## Oracle 中的 SQL 是如何执行的
SQL 在 Oracle 中的执行过程：

{% asset_img img1.png %}

从上面这张图中可以看出，SQL 语句在 Oracle 中经历了以下的几个步骤。
* 语法检查：检查 SQL 拼写是否正确，如果不正确，Oracle 会报语法错误。
* 语义检查：检查 SQL 中的访问对象是否存在。比如我们在写 SELECT 语句的时候，列名写错了，系统就会提示错误。语法检查和语义检查的作用是保证 SQL 语句没有错误。
* 权限检查：看用户是否具备访问该数据的权限。
* 共享池检查：共享池（`Shared Pool`）是一块内存池，最主要的作用是缓存 SQL 语句和该语句的执行计划。Oracle 通过检查共享池是否存在 SQL 语句的执行计划，来判断进行软解析，还是硬解析。那软解析和硬解析又该怎么理解呢？
在共享池中，Oracle 首先对 SQL 语句进行`Hash`运算，然后根据 Hash 值在库缓存（`Library Cache`）中查找，如果存在 SQL 语句的执行计划，就直接拿来执行，直接进入“执行器”的环节，这就是软解析。
如果没有找到 SQL 语句和执行计划，Oracle 就需要创建解析树进行解析，生成执行计划，进入“优化器”这个步骤，这就是硬解析。
* 优化器：优化器中就是要进行硬解析，也就是决定怎么做，比如创建解析树，生成执行计划。
* 执行器：当有了解析树和执行计划之后，就知道了 SQL 该怎么被执行，这样就可以在执行器中执行语句了。

共享池是 Oracle 中的术语，包括了库缓存，数据字典缓冲区等。库缓存区主要缓存 SQL 语句和执行计划。而数据字典缓冲区存储的是 Oracle 中的对象定义，比如表、视图、索引等对象。当对 SQL 语句进行解析的时候，如果需要相关的数据，会从数据字典缓冲区中提取。

库缓存这一个步骤，决定了 SQL 语句是否需要进行硬解析。为了提升 SQL 的执行效率，我们应该尽量避免硬解析，因为在 SQL 的执行过程中，创建解析树，生成执行计划是很消耗资源的。

你可能会问，如何避免硬解析，尽量使用软解析呢？在 Oracle 中，绑定变量是它的一大特色。绑定变量就是在 SQL 语句中使用变量，通过不同的变量取值来改变 SQL 的执行结果。这样做的好处是能提升软解析的可能性，不足之处在于可能会导致生成的执行计划不够优化，因此是否需要绑定变量还需要视情况而定。

举个例子，我们可以使用下面的查询语句：
```sql
SQL> select * from player where player_id = 10001;
```
你也可以使用绑定变量，如：
```sql
SQL> select * from player where player_id = :player_id;
```
这两个查询语句的效率在 Oracle 中是完全不同的。如果你在查询 player_id = 10001 之后，还会查询 10002、10003 之类的数据，那么每一次查询都会创建一个新的查询解析。而第二种方式使用了绑定变量，那么在第一次查询之后，在共享池中就会存在这类查询的执行计划，也就是软解析。

因此我们可以通过使用绑定变量来减少硬解析，减少 Oracle 的解析工作量。但是这种方式也有缺点，使用动态 SQL 的方式，因为参数不同，会导致 SQL 的执行效率不同，同时 SQL 优化也会比较困难。
## MySQL 中的 SQL 是如何执行的
首先 MySQL 是典型的 C/S 架构，即 Client/Server 架构，服务器端程序使用的 mysqld。整体的 MySQL 流程如下图所示：

{% asset_img img2.png %}

MySQL 由三层组成：
* 连接层：客户端和服务器端建立连接，客户端发送 SQL 至服务器端；
* SQL 层：对 SQL 语句进行查询处理；
* 存储引擎层：与数据库文件打交道，负责数据的存储和读取。

其中 SQL 层与数据库文件的存储方式无关，我们来看下 SQL 层的结构：

{% asset_img img3.jpeg %}

1. 查询缓存：Server 如果在查询缓存中发现了这条 SQL 语句，就会直接将结果返回给客户端；如果没有，就进入到解析器阶段。需要说明的是，因为查询缓存往往效率不高，所以在 MySQL8.0 之后就抛弃了这个功能。
2. 解析器：在解析器中对 SQL 语句进行语法分析、语义分析。
3. 优化器：在优化器中会确定 SQL 语句的执行路径，比如是根据全表检索，还是根据索引来检索等。
4. 执行器：在执行之前需要判断该用户是否具备权限，如果具备权限就执行 SQL 查询并返回结果。在 MySQL8.0 以下的版本，如果设置了查询缓存，这时会将查询结果进行缓存。

SQL 语句在 MySQL 中的流程是：SQL 语句→缓存查询→解析器→优化器→执行器。在一部分中，MySQL 和 Oracle 执行 SQL 的原理是一样的。

与 Oracle 不同的是，MySQL 的存储引擎采用了插件的形式，每个存储引擎都面向一种特定的数据库应用环境。同时开源的 MySQL 还允许开发人员设置自己的存储引擎，下面是一些常见的存储引擎：
* InnoDB 存储引擎：它是 MySQL 5.5 版本之后默认的存储引擎，最大的特点是支持事务、行级锁定、外键约束等。
* MyISAM 存储引擎：在 MySQL 5.5 版本之前是默认的存储引擎，不支持事务，也不支持外键，最大的特点是速度快，占用资源少。
* Memory 存储引擎：使用系统内存作为存储介质，以便得到更快的响应速度。不过如果 mysqld 进程崩溃，则会导致所有的数据丢失，因此我们只有当数据是临时的情况下才使用 Memory 存储引擎。
* NDB 存储引擎：也叫做 NDB Cluster 存储引擎，主要用于 MySQL Cluster 分布式集群环境，类似于 Oracle 的 RAC 集群。
* Archive 存储引擎：它有很好的压缩机制，用于文件归档，在请求写入时会进行压缩，所以也经常用来做仓库。

需要注意的是，数据库的设计在于表的设计，而在 MySQL 中每个表的设计都可以采用不同的存储引擎，我们可以根据实际的数据处理需要来选择存储引擎，这也是 MySQL 的强大之处。
