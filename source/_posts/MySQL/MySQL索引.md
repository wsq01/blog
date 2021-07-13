


# 索引（Index）是什么
索引是一种特殊的数据库结构，由数据表中的一列或多列组合而成，可以用来快速查询数据表中有某一特定值的记录。本节将详细讲解索引的含义、作用和优缺点。

通过索引，查询数据时不用读完记录的所有信息，而只是查询索引列。否则，数据库系统将读取每条记录的所有信息进行匹配。

可以把索引比作新华字典的音序表。例如，要查“库”字，如果不使用音序，就需要从字典的 400 页中逐页来找。但是，如果提取拼音出来，构成音序表，就只需要从 10 多页的音序表中直接查找。这样就可以大大节省时间。

因此，使用索引可以很大程度上提高数据库的查询速度，还有效的提高了数据库系统的性能。
为什么要使用索引
索引就是根据表中的一列或若干列按照一定顺序建立的列值与记录行之间的对应关系表，实质上是一张描述索引列的列值与原表中记录行之间一 一对应关系的有序表。

索引是 MySQL 中十分重要的数据库对象，是数据库性能调优技术的基础，常用于实现数据的快速检索。

在 MySQL 中，通常有以下两种方式访问数据库表的行数据：
1) 顺序访问
顺序访问是在表中实行全表扫描，从头到尾逐行遍历，直到在无序的行数据中找到符合条件的目标数据。

顺序访问实现比较简单，但是当表中有大量数据的时候，效率非常低下。例如，在几千万条数据中查找少量的数据时，使用顺序访问方式将会遍历所有的数据，花费大量的时间，显然会影响数据库的处理性能。
2) 索引访问
索引访问是通过遍历索引来直接访问表中记录行的方式。

使用这种方式的前提是对表建立一个索引，在列上创建了索引之后，查找数据时可以直接根据该列上的索引找到对应记录行的位置，从而快捷地查找到数据。索引存储了指定列数据值的指针，根据指定的排序顺序对这些指针排序。

例如，在学生基本信息表 tb_students 中，如果基于 student_id 建立了索引，系统就建立了一张索引列到实际记录的映射表。当用户需要查找 student_id 为 12022 的数据的时候，系统先在 student_id 索引上找到该记录，然后通过映射表直接找到数据行，并且返回该行数据。因为扫描索引的速度一般远远大于扫描实际数据行的速度，所以采用索引的方式可以大大提高数据库的工作效率。

简而言之，不使用索引，MySQL 就必须从第一条记录开始读完整个表，直到找出相关的行。表越大，查询数据所花费的时间就越多。如果表中查询的列有一个索引，MySQL 就能快速到达一个位置去搜索数据文件，而不必查看所有数据，这样将会节省很大一部分时间。
索引的优缺点
索引有其明显的优势，也有其不可避免的缺点。
优点
索引的优点如下：
通过创建唯一索引可以保证数据库表中每一行数据的唯一性。
可以给所有的 MySQL 列类型设置索引。
可以大大加快数据的查询速度，这是使用索引最主要的原因。
在实现数据的参考完整性方面可以加速表与表之间的连接。
在使用分组和排序子句进行数据查询时也可以显著减少查询中分组和排序的时间
缺点
增加索引也有许多不利的方面，主要如下：
创建和维护索引组要耗费时间，并且随着数据量的增加所耗费的时间也会增加。
索引需要占磁盘空间，除了数据表占数据空间以外，每一个索引还要占一定的物理空间。如果有大量的索引，索引文件可能比数据文件更快达到最大文件尺寸。
当对表中的数据进行增加、删除和修改的时候，索引也要动态维护，这样就降低了数据的维护速度。

使用索引时，需要综合考虑索引的优点和缺点。

索引可以提高查询速度，但是会影响插入记录的速度。因为，向有索引的表中插入记录时，数据库系统会按照索引进行排序，这样就降低了插入记录的速度，插入大量记录时的速度影响会更加明显。这种情况下，最好的办法是先删除表中的索引，然后插入数据，插入完成后，再创建索引。
# 索引类型
索引的类型和存储引擎有关，每种存储引擎所支持的索引类型不一定完全相同。MySQL 索引可以从存储方式、逻辑角度和实际使用的角度来进行分类。
存储方式区分
根据存储方式的不同，MySQL 中常用的索引在物理上分为  B-树索引和 HASH 索引两类，两种不同类型的索引各有其不同的适用范围。
1) B-树索引
B-树索引又称为 BTREE 索引，目前大部分的索引都是采用 B-树索引来存储的。

B-树索引是一个典型的数据结构，其包含的组件主要有以下几个：
叶子节点：包含的条目直接指向表里的数据行。叶子节点之间彼此相连，一个叶子节点有一个指向下一个叶子节点的指针。
分支节点：包含的条目指向索引里其他的分支节点或者叶子节点。
根节点：一个 B-树索引只有一个根节点，实际上就是位于树的最顶端的分支节点。

基于这种树形数据结构，表中的每一行都会在索引上有一个对应值。因此，在表中进行数据查询时，可以根据索引值一步一步定位到数据所在的行。

B-树索引可以进行全键值、键值范围和键值前缀查询，也可以对查询结果进行 ORDER BY 排序。但 B-树索引必须遵循左边前缀原则，要考虑以下几点约束：
查询必须从索引的最左边的列开始。
查询不能跳过某一索引列，必须按照从左到右的顺序进行匹配。
存储引擎不能使用索引中范围条件右边的列。
2) 哈希索引
哈希（Hash）一般翻译为“散列”，也有直接音译成“哈希”的，就是把任意长度的输入（又叫作预映射，pre-image）通过散列算法变换成固定长度的输出，该输出就是散列值。

哈希索引也称为散列索引或 HASH 索引。MySQL 目前仅有 MEMORY 存储引擎和 HEAP 存储引擎支持这类索引。其中，MEMORY 存储引擎可以支持 B-树索引和 HASH 索引，且将 HASH 当成默认索引。

HASH 索引不是基于树形的数据结构查找数据，而是根据索引列对应的哈希值的方法获取表的记录行。哈希索引的最大特点是访问速度快，但也存在下面的一些缺点：
MySQL 需要读取表中索引列的值来参与散列计算，散列计算是一个比较耗时的操作。也就是说，相对于 B-树索引来说，建立哈希索引会耗费更多的时间。
不能使用 HASH 索引排序。
HASH 索引只支持等值比较，如“=”“IN()”或“<=>”。
HASH 索引不支持键的部分匹配，因为在计算 HASH 值的时候是通过整个索引值来计算的。
逻辑区分
根据索引的具体用途，MySQL 中的索引在逻辑上分为以下 5 类：
1) 普通索引
普通索引是 MySQL 中最基本的索引类型，它没有任何限制，唯一任务就是加快系统对数据的访问速度。

普通索引允许在定义索引的列中插入重复值和空值。

创建普通索引时，通常使用的关键字是 INDEX 或 KEY。
例 1
下面在 tb_student 表中的 id 字段上建立名为 index_id 的索引。
CREATE INDEX index_id ON tb_student(id);

2) 唯一索引
唯一索引与普通索引类似，不同的是创建唯一性索引的目的不是为了提高访问速度，而是为了避免数据出现重复。

唯一索引列的值必须唯一，允许有空值。如果是组合索引，则列值的组合必须唯一。

创建唯一索引通常使用 UNIQUE 关键字。
例 2
下面在 tb_student 表中的 id 字段上建立名为 index_id 的索引，SQL 语句如下：
CREATE UNIQUE INDEX index_id ON tb_student(id);

其中，id 字段可以有唯一性约束，也可以没有。
3) 主键索引
顾名思义，主键索引就是专门为主键字段创建的索引，也属于索引的一种。

主键索引是一种特殊的唯一索引，不允许值重复或者值为空。

创建主键索引通常使用 PRIMARY KEY 关键字。不能使用 CREATE INDEX 语句创建主键索引。
4) 空间索引
空间索引是对空间数据类型的字段建立的索引，使用 SPATIAL 关键字进行扩展。

创建空间索引的列必须将其声明为 NOT NULL，空间索引只能在存储引擎为 MyISAM 的表中创建。

空间索引主要用于地理空间数据类型 GEOMETRY。对于初学者来说，这类索引很少会用到。
例 3
下面在 tb_student 表中的 line 字段上建立名为 index_line 的索引，SQL 语句如下：
CREATE SPATIAL INDEX index_line ON tb_student(line);

其中，tb_student 表的存储引擎必须是 MyISAM，line 字段必须为空间数据类型，而且是非空的。
5) 全文索引
全文索引主要用来查找文本中的关键字，只能在 CHAR、VARCHAR 或 TEXT 类型的列上创建。在 MySQL 中只有 MyISAM 存储引擎支持全文索引。

全文索引允许在索引列中插入重复值和空值。

不过对于大容量的数据表，生成全文索引非常消耗时间和硬盘空间。

创建全文索引使用 FULLTEXT 关键字。
例 4
在 tb_student 表中的 info 字段上建立名为 index_info 的全文索引，SQL 语句如下：
CREATE FULLTEXT INDEX index_info ON tb_student(info);

其中，index_info 的存储引擎必须是 MyISAM，info 字段必须是 CHAR、VARCHAR 和 TEXT 等类型。
实际使用区分
索引在逻辑上分为以上 5 类，但在实际使用中，索引通常被创建成单列索引和组合索引。
1）单列索引
单列索引就是索引只包含原表的一个列。在表中的单个字段上创建索引，单列索引只根据该字段进行索引。

单列索引可以是普通索引，也可以是唯一性索引，还可以是全文索引。只要保证该索引只对应一个字段即可。
例 5
下面在 tb_student 表中的 address 字段上建立名为 index_addr 的单列索引，address 字段的数据类型为 VARCHAR(20)，索引的数据类型为 CHAR(4)。SQL 语句如下：
CREATE INDEX index_addr ON tb_student(address(4));

这样，查询时可以只查询 address 字段的前 4 个字符，而不需要全部查询。
2）多列索引
组合索引也称为复合索引或多列索引。相对于单列索引来说，组合索引是将原表的多个列共同组成一个索引。多列索引是在表的多个字段上创建一个索引。该索引指向创建时对应的多个字段，可以通过这几个字段进行查询。但是，只有查询条件中使用了这些字段中第一个字段时，索引才会被使用。

例如，在表中的 id、name 和 sex 字段上建立一个多列索引，那么，只有查询条件使用了 id 字段时，该索引才会被使用。
例 6
下面在 tb_student 表中的 name 和 address 字段上建立名为 index_na 的索引，SQL 语句如下：
CREATE INDEX index_na ON tb_student(name,address);

该索引创建好了以后，查询条件中必须有 name 字段才能使用索引。
提示：一个表可以有多个单列索引，但这些索引不是组合索引。一个组合索引实质上为表的查询提供了多个索引，以此来加快查询速度。比如，在一个表中创建了一个组合索引(c1，c2，c3)，在实际查询中，系统用来实际加速的索引有三个：单个索引(c1)、双列索引(c1，c2)和多列索引(c1，c2，c3)。
# 创建索引
创建索引是指在某个表的一列或多列上建立一个索引，可以提高对表的访问速度。创建索引对 MySQL 数据库的高效运行来说是很重要的。
基本语法
MySQL 提供了三种创建索引的方法：
1) 使用 CREATE INDEX 语句
可以使用专门用于创建索引的 CREATE INDEX 语句在一个已有的表上创建索引，但该语句不能创建主键。

语法格式：
CREATE <索引名> ON <表名> (<列名> [<长度>] [ ASC | DESC])

语法说明如下：
<索引名>：指定索引名。一个表可以创建多个索引，但每个索引在该表中的名称是唯一的。
<表名>：指定要创建索引的表名。
<列名>：指定要创建索引的列名。通常可以考虑将查询语句中在 JOIN 子句和 WHERE 子句里经常出现的列作为索引列。
<长度>：可选项。指定使用列前的 length 个字符来创建索引。使用列的一部分创建索引有利于减小索引文件的大小，节省索引列所占的空间。在某些情况下，只能对列的前缀进行索引。索引列的长度有一个最大上限 255 个字节（MyISAM 和 InnoDB 表的最大上限为 1000 个字节），如果索引列的长度超过了这个上限，就只能用列的前缀进行索引。另外，BLOB 或 TEXT 类型的列也必须使用前缀索引。
ASC|DESC：可选项。ASC指定索引按照升序来排列，DESC指定索引按照降序来排列，默认为ASC。
2) 使用 CREATE TABLE 语句
索引也可以在创建表（CREATE TABLE）的同时创建。在 CREATE TABLE 语句中添加以下语句。语法格式：
CONSTRAINT PRIMARY KEY [索引类型] (<列名>,…)

在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的主键。

语法格式：
KEY | INDEX [<索引名>] [<索引类型>] (<列名>,…)

在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的索引。

语法格式：
UNIQUE [ INDEX | KEY] [<索引名>] [<索引类型>] (<列名>,…)

在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的唯一性索引。

语法格式：
FOREIGN KEY <索引名> <列名>

在 CREATE TABLE 语句中添加此语句，表示在创建新表的同时创建该表的外键。

在使用 CREATE TABLE 语句定义列选项的时候，可以通过直接在某个列定义后面添加 PRIMARY KEY 的方式创建主键。而当主键是由多个列组成的多列索引时，则不能使用这种方法，只能用在语句的最后加上一个 PRIMARY KRY(<列名>，…) 子句的方式来实现。
3) 使用 ALTER TABLE 语句
CREATE INDEX 语句可以在一个已有的表上创建索引，ALTER TABLE 语句也可以在一个已有的表上创建索引。在使用 ALTER TABLE 语句修改表的同时，可以向已有的表添加索引。具体的做法是在 ALTER TABLE 语句中添加以下语法成分的某一项或几项。

语法格式：
ADD INDEX [<索引名>] [<索引类型>] (<列名>,…)

在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加索引。

语法格式：
ADD PRIMARY KEY [<索引类型>] (<列名>,…)

在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加主键。

语法格式：
ADD UNIQUE [ INDEX | KEY] [<索引名>] [<索引类型>] (<列名>,…)

在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加唯一性索引。

语法格式：
ADD FOREIGN KEY [<索引名>] (<列名>,…)

在 ALTER TABLE 语句中添加此语法成分，表示在修改表的同时为该表添加外键。
创建普通索引
创建普通索引时，通常使用 INDEX 关键字。
例 1
创建一个表 tb_stu_info，在该表的 height 字段创建普通索引。输入的 SQL 语句和执行过程如下所示。
mysql> CREATE TABLE tb_stu_info
    -> (
    -> id INT NOT NULL,
    -> name CHAR(45) DEFAULT NULL,
    -> dept_id INT DEFAULT NULL,
    -> age INT DEFAULT NULL,
    -> height INT DEFAULT NULL,
    -> INDEX(height)
    -> );
Query OK，0 rows affected (0.40 sec)
mysql> SHOW CREATE TABLE tb_stu_info\G
*************************** 1. row ***************************
       Table: tb_stu_info
Create Table: CREATE TABLE `tb_stu_info` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL,
  KEY `height` (`height`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.01 sec)
创建唯一索引
创建唯一索引，通常使用 UNIQUE 参数。
例 2
创建一个表 tb_stu_info2，在该表的 id 字段上使用 UNIQUE 关键字创建唯一索引。输入的 SQL 语句和执行过程如下所示。
mysql> CREATE TABLE tb_stu_info2
    -> (
    -> id INT NOT NULL,
    -> name CHAR(45) DEFAULT NULL,
    -> dept_id INT DEFAULT NULL,
    -> age INT DEFAULT NULL,
    -> height INT DEFAULT NULL,
    -> UNIQUE INDEX(height)
    -> );
Query OK，0 rows affected (0.40 sec)
mysql> SHOW CREATE TABLE tb_stu_info2\G
*************************** 1. row ***************************
       Table: tb_stu_info2
Create Table: CREATE TABLE `tb_stu_info2` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL,
  UNIQUE KEY `height` (`height`)
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)

# 查看索引
索引创建完成后，可以利用 SQL 语句查看已经存在的索引。在 MySQL 中，可以使用 SHOW INDEX 语句查看表中创建的索引。

查看索引的语法格式如下：
SHOW INDEX FROM <表名> [ FROM <数据库名>]

语法说明如下：
<表名>：指定需要查看索引的数据表名。
<数据库名>：指定需要查看索引的数据表所在的数据库，可省略。比如，SHOW INDEX FROM student FROM test; 语句表示查看 test 数据库中 student 数据表的索引。
示例
使用 SHOW INDEX 语句查看《MySQL创建索引》一节中 tb_stu_info2 数据表的索引信息，SQL 语句和运行结果如下所示。
mysql> SHOW INDEX FROM tb_stu_info2\G
*************************** 1. row ***************************
        Table: tb_stu_info2
   Non_unique: 0
     Key_name: height
 Seq_in_index: 1
  Column_name: height
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment:
Index_comment:
1 row in set (0.03 sec)
其中各主要参数说明如下：
参数	说明
Table	表示创建索引的数据表名，这里是 tb_stu_info2 数据表。
Non_unique	表示该索引是否是唯一索引。若不是唯一索引，则该列的值为 1；若是唯一索引，则该列的值为 0。
Key_name	表示索引的名称。
Seq_in_index	表示该列在索引中的位置，如果索引是单列的，则该列的值为 1；如果索引是组合索引，则该列的值为每列在索引定义中的顺序。
Column_name	表示定义索引的列字段。
Collation	表示列以何种顺序存储在索引中。在 MySQL 中，升序显示值“A”（升序），若显示为 NULL，则表示无分类。
Cardinality	索引中唯一值数目的估计值。基数根据被存储为整数的统计数据计数，所以即使对于小型表，该值也没有必要是精确的。基数越大，当进行联合时，MySQL 使用该索引的机会就越大。
Sub_part	表示列中被编入索引的字符的数量。若列只是部分被编入索引，则该列的值为被编入索引的字符的数目；若整列被编入索引，则该列的值为 NULL。
Packed	指示关键字如何被压缩。若没有被压缩，值为 NULL。
Null	用于显示索引列中是否包含 NULL。若列含有 NULL，该列的值为 YES。若没有，则该列的值为 NO。
Index_type	显示索引使用的类型和方法（BTREE、FULLTEXT、HASH、RTREE）。
Comment	显示评注。
# 修改和删除索引
删除索引是指将表中已经存在的索引删除掉。不用的索引建议进行删除，因为它们会降低表的更新速度，影响数据库的性能。对于这样的索引，应该将其删除。

在 MySQL 中修改索引可以通过删除原索引，再根据需要创建一个同名的索引，从而实现修改索引的操作。
基本语法
当不再需要索引时，可以使用 DROP INDEX 语句或 ALTER TABLE 语句来对索引进行删除。
1) 使用 DROP INDEX 语句
语法格式：
DROP INDEX <索引名> ON <表名>

语法说明如下：
<索引名>：要删除的索引名。
<表名>：指定该索引所在的表名。
2) 使用 ALTER TABLE 语句
根据 ALTER TABLE 语句的语法可知，该语句也可以用于删除索引。具体使用方法是将 ALTER TABLE 语句的语法中部分指定为以下子句中的某一项。
DROP PRIMARY KEY：表示删除表中的主键。一个表只有一个主键，主键也是一个索引。
DROP INDEX index_name：表示删除名称为 index_name 的索引。
DROP FOREIGN KEY fk_symbol：表示删除外键。
注意：如果删除的列是索引的组成部分，那么在删除该列时，也会将该列从索引中删除；如果组成索引的所有列都被删除，那么整个索引将被删除。

删除索引
【实例 1】删除表 tb_stu_info 中的索引，输入的 SQL 语句和执行结果如下所示。
mysql> DROP INDEX height
    -> ON tb_stu_info;
Query OK, 0 rows affected (0.27 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW CREATE TABLE tb_stu_info\G
*************************** 1. row ***************************
       Table: tb_stu_info
Create Table: CREATE TABLE `tb_stu_info` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
【实例 2】删除表 tb_stu_info2 中名称为 id 的索引，输入的 SQL 语句和执行结果如下所示。
mysql> ALTER TABLE tb_stu_info2
    -> DROP INDEX height;
Query OK, 0 rows affected (0.13 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW CREATE TABLE tb_stu_info2\G
*************************** 1. row ***************************
       Table: tb_stu_info2
Create Table: CREATE TABLE `tb_stu_info2` (
  `id` int(11) NOT NULL,
  `name` char(45) DEFAULT NULL,
  `dept_id` int(11) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=gb2312
1 row in set (0.00 sec)
# 索引在什么情况下不会被使用
索引可以提高查询的速度，但并不是使用带有索引的字段查询时，索引都会起作用。使用索引有几种特殊情况，在这些情况下，有可能使用带有索引的字段查询时，索引并没有起作用，下面重点介绍这几种特殊情况。
1. 查询语句中使用LIKE关键字
在查询语句中使用 LIKE 关键字进行查询时，如果匹配字符串的第一个字符为“%”，索引不会被使用。如果“%”不是在第一个位置，索引就会被使用。
例 1
为了便于理解，我们先查询 tb_student 表中的数据，SQL 语句和运行结果如下：
mysql> SELECT * FROM tb_student;
+----+------+------+------+
| id | name | age  | sex  |
+----+------+------+------+
|  1 | 张三 |   12 | 男   |
|  2 | 李四 |   12 | 男   |
|  3 | 王五 |   13 | 女   |
|  4 | 张四 |   13 | 女   |
|  5 | 王四 |   15 | 男   |
|  6 | 赵六 |   12 | 女   |
+----+------+------+------+
6 rows in set (0.03 sec)
下面在查询语句中使用 LIKE 关键字，且匹配的字符串中含有“%”符号，使用 EXPLAIN 分析查询情况，SQL 语句和运行结果如下：
mysql>  EXPLAIN SELECT * FROM tb_student WHERE name LIKE '%四'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_student
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 6
     filtered: 16.67
        Extra: Using where
1 row in set, 1 warning (0.01 sec)

mysql> CREATE INDEX index_name ON tb_student(name);
Query OK, 6 rows affected (0.13 sec)

mysql>  EXPLAIN SELECT * FROM tb_student WHERE name LIKE '李%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_student
   partitions: NULL
         type: range
possible_keys: index_name
          key: index_name
      key_len: 77
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
第一个查询语句执行后，rows 参数的值为 6，表示这次查询过程中查询了 6 条记录；第二个查询语句执行后，rows 参数的值为 1，表示这次查询过程只查询 1 条记录。同样是使用 name 字段进行查询，因为第一个查询语句的 LIKE 关键字后的字符串是以“%”开头的，所以第一个查询语句没有使用索引，而第二个查询语句使用了索引 index_name。
2. 查询语句中使用多列索引
多列索引是在表的多个字段上创建一个索引，只有查询条件中使用了这些字段中的第一个字段，索引才会被使用。
例 2
在 name 和 age 两个字段上创建多列索引，并验证多列索引的使用情况，SQL 语句和运行结果如下：
mysql> CREATE INDEX index_name_age ON tb_student(name,age);
Query OK, 6 rows affected (0.11 sec)

mysql> EXPLAIN SELECT * FROM tb_student WHERE name LIKE '李%'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_student
   partitions: NULL
         type: range
possible_keys: index_name_age
          key: index_name_age
      key_len: 77
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.05 sec)

mysql> EXPLAIN SELECT * FROM tb_student WHERE age LIKE '12'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_student
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 6
     filtered: 16.67
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
第一条查询语句的查询条件使用了 name 字段，分析结果显示 rows 参数的值为 1，且查询过程中使用了 index_name_age 索引。第二条查询语句的查询条件使用了 age 字段，结果显示 rows 参数的值为 6，且 key 参数的值为 NULL，这说明第二个查询语句没有使用索引。

因为 name 字段是多列索引的第一个字段，所以只有查询条件中使用了 name 字段才会使 index_name_age 索引起作用。
3. 查询语句中使用OR关键字
查询语句只有 OR 关键字时，如果 OR 前后的两个条件的列都是索引，那么查询中将使用索引。如果 OR 前后有一个条件的列不是索引，那么查询中将不使用索引。
例 3
下面演示 OR 关键字的使用。
mysql> EXPLAIN SELECT * FROM tb_student WHERE name='张三' or sex='男'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_student
   partitions: NULL
         type: ALL
possible_keys: index_name,index_name_age
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 6
     filtered: 30.56
        Extra: Using where
1 row in set, 1 warning (0.06 sec)
mysql> EXPLAIN SELECT * FROM tb_student WHERE name='张三' or id='12'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_student
   partitions: NULL
         type: index_merge
possible_keys: PRIMARY,index_name,index_name_age
          key: index_name,PRIMARY
      key_len: 77,4
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: Using union(index_name,PRIMARY); Using where
1 row in set, 1 warning (0.01 sec)
由于 sex 字段没有索引，所以第一条查询语句没有使用索引；name 字段和 id 字段都有索引，所以第二条查询语句使用了 index_name 和 PRIMARY 索引 。
总结
使用索引查询记录时，一定要注意索引的使用情况。例如，LIKE 关键字配置的字符串不能以“%”开头；使用多列索引时，查询条件必须要使用这个索引的第一个字段；使用 OR 关键字时，OR 关键字连接的所有条件都必须使用索引。
# 索引的设计原则
索引的设计可以遵循一些已有的原则，创建索引的时候应尽量考虑符合这些原则，便于提升索引的使用效率，更高效的使用索引。本节将介绍一些索引的设计原则。
1. 选择唯一性索引
唯一性索引的值是唯一的，可以更快速的通过该索引来确定某条记录。例如，学生表中学号是具有唯一性的字段。为该字段建立唯一性索引可以很快的确定某个学生的信息。如果使用姓名的话，可能存在同名现象，从而降低查询速度。
2. 为经常需要排序、分组和联合操作的字段建立索引
经常需要 ORDER BY、GROUP BY、DISTINCT 和 UNION 等操作的字段，排序操作会浪费很多时间。如果为其建立索引，可以有效地避免排序操作。
3. 为常作为查询条件的字段建立索引
如果某个字段经常用来做查询条件，那么该字段的查询速度会影响整个表的查询速度。因此，为这样的字段建立索引，可以提高整个表的查询速度。
注意：常查询条件的字段不一定是所要选择的列，换句话说，最适合索引的列是出现在 WHERE 子句中的列，或连接子句中指定的列，而不是出现在 SELECT 关键字后的选择列表中的列。

4. 限制索引的数目
索引的数目不是“越多越好”。每个索引都需要占用磁盘空间，索引越多，需要的磁盘空间就越大。在修改表的内容时，索引必须进行更新，有时还可能需要重构。因此，索引越多，更新表的时间就越长。

如果有一个索引很少利用或从不使用，那么会不必要地减缓表的修改速度。此外，MySQL 在生成一个执行计划时，要考虑各个索引，这也要花费时间。创建多余的索引给查询优化带来了更多的工作。索引太多，也可能会使 MySQL 选择不到所要使用的最佳索引。
5. 尽量使用数据量少的索引
如果索引的值很长，那么查询的速度会受到影响。例如，对一个 CHAR(100) 类型的字段进行全文检索需要的时间肯定要比对 CHAR(10) 类型的字段需要的时间要多。
6. 数据量小的表最好不要使用索引
由于数据较小，查询花费的时间可能比遍历索引的时间还要短，索引可能不会产生优化效果。
7. 尽量使用前缀来索引
如果索引字段的值很长，最好使用值的前缀来索引。例如，TEXT 和 BLOG 类型的字段，进行全文检索会很浪费时间。如果只检索字段的前面的若干个字符，这样可以提高检索速度。
8. 删除不再使用或者很少使用的索引
表中的数据被大量更新，或者数据的使用方式被改变后，原有的一些索引可能不再需要。应该定期找出这些索引，将它们删除，从而减少索引对更新操作的影响。
总结
选择索引的最终目的是为了使查询的速度变快，上面给出的原则是最基本的准则，但不能只拘泥于上面的准则。应该在学习和工作中不断的实践，根据应用的实际情况进行分析和判断，选择最合适的索引方式。