---
title: MySQL 存储过程
date: 2020-05-08 16:44:13
tags: [MySQL]
categories: [MySQL]
---


# 存储过程是什么？
在数据库的实际操作中，经常会有需要多条 SQL 语句处理多个表才能完成的操作。

存储过程是一组为了完成特定功能的 SQL 语句集合。使用存储过程的目的是将常用或复杂的工作预先用 SQL 语句写好并用一个指定名称存储起来，这个过程经编译和优化后存储在数据库服务器中，因此称为存储过程。当以后需要数据库提供与已定义好的存储过程的功能相同的服务时，只需调用“CALL存储过程名字”即可自动完成。

常用操作数据库的 SQL 语句在执行的时候需要先编译，然后执行。存储过程则采用另一种方式来执行 SQL 语句。

一个存储过程是一个可编程的函数，它在数据库中创建并保存，一般由 SQL 语句和一些特殊的控制结构组成。当希望在不同的应用程序或平台上执行相同的特定功能时，存储过程尤为合适。

存储过程是数据库中的一个重要功能，存储过程可以用来转换数据、数据迁移、制作报表，它类似于编程语言，一次执行成功，就可以随时被调用，完成指定的功能操作。

使用存储过程不仅可以提高数据库的访问效率，同时也可以提高数据库使用的安全性。

对于调用者来说，存储过程封装了 SQL 语句，调用者无需考虑逻辑功能的具体实现过程。只是简单调用即可，它可以被 Java 和 C# 等编程语言调用。

存储过程有如下优点：
1. 封装性
通常完成一个逻辑功能需要多条 SQL 语句，而且各个语句之间很可能传递参数，所以，编写逻辑功能相对来说稍微复杂些，而存储过程可以把这些 SQL 语句包含到一个独立的单元中，使外界看不到复杂的 SQL 语句，只需要简单调用即可达到目的。并且数据库专业人员可以随时对存储过程进行修改，而不会影响到调用它的应用程序源代码。
2. 可增强 SQL 语句的功能和灵活性
存储过程可以用流程控制语句编写，有很强的灵活性，可以完成复杂的判断和较复杂的运算。
3. 可减少网络流量
由于存储过程是在服务器端运行的，且执行速度快，因此当客户计算机上调用该存储过程时，网络中传送的只是该调用语句，从而可降低网络负载。
4. 高性能
当存储过程被成功编译后，就存储在数据库服务器里了，以后客户端可以直接调用，这样所有的 SQL 语句将从服务器执行，从而提高性能。但需要说明的是，存储过程不是越多越好，过多的使用存储过程反而影响系统性能。
5. 提高数据库的安全性和数据的完整性
存储过程提高安全性的一个方案就是把它作为中间组件，存储过程里可以对某些表做相关操作，然后存储过程作为接口提供给外部程序。这样，外部程序无法直接操作数据库表，只能通过存储过程来操作对应的表，因此在一定程度上，安全性是可以得到提高的。
6. 使数据独立
数据的独立可以达到解耦的效果，也就是说，程序可以调用存储过程，来替代执行多条的 SQL 语句。这种情况下，存储过程把数据同用户隔离开来，优点就是当数据表的结构改变时，调用表不用修改程序，只需要数据库管理者重新编写存储过程即可。

# 创建存储过程
可以使用`CREATE PROCEDURE`语句创建存储过程：
```sql
CREATE PROCEDURE <过程名> ( [过程参数[,…] ] ) <过程体>
[过程参数[,…] ] 格式
[ IN | OUT | INOUT ] <参数名> <类型>
```
语法说明：
### 1.过程名
存储过程的名称，默认在当前数据库中创建。若需要在特定数据库中创建存储过程，则要在名称前面加上数据库的名称，即`db_name.sp_name`。

需要注意的是，名称应当尽量避免选取与 MySQL 内置函数相同的名称，否则会发生错误。
### 2.过程参数
存储过程的参数列表。其中，<参数名>为参数名，<类型>为参数的类型（可以是任何有效的 MySQL 数据类型）。当有多个参数时，参数列表中彼此间用逗号分隔。存储过程可以没有参数（此时存储过程的名称后仍需加上一对括号），也可以有 1 个或多个参数。

MySQL 存储过程支持三种类型的参数，即输入参数、输出参数和输入/输出参数，分别用`IN、OUT`和`INOUT`三个关键字标识。其中，输入参数可以传递给一个存储过程，输出参数用于存储过程需要返回一个操作结果的情形，而输入/输出参数既可以充当输入参数也可以充当输出参数。

需要注意的是，参数的取名不要与数据表的列名相同，否则尽管不会返回出错信息，但是存储过程的 SQL 语句会将参数名看作列名，从而引发不可预知的结果。
### 3.过程体
存储过程的主体部分，也称为存储过程体，包含在过程调用的时候必须执行的 SQL 语句。这个部分以关键字`BEGIN`开始，以关键字`END`结束。若存储过程体中只有一条 SQL 语句，则可以省略`BEGIN-END`标志。

在存储过程的创建中，经常会用到`DELIMITER`命令。

在 MySQL 中，服务器处理 SQL 语句默认是以分号作为语句结束标志的。然而，在创建存储过程时，存储过程体可能包含有多条 SQL 语句，这些 SQL 语句如果仍以分号作为语句结束符，那么 MySQL 服务器在处理时会以遇到的第一条 SQL 语句结尾处的分号作为整个程序的结束符，而不再去处理存储过程体中后面的 SQL 语句，这样显然不行。

为解决以上问题，通常使用`DELIMITER`命令将结束命令修改为其他字符。语法格式如下：
```
DELIMITER $$
```
语法说明如下：
* `$$`是用户定义的结束符，通常这个符号可以是一些特殊的符号，如两个`?`或两个`￥`等。
* 当使用`DELIMITER`命令时，应该避免使用反斜杠`\`字符，因为它是 MySQL 的转义字符。

在 MySQL 命令行客户端输入如下 SQL 语句。
```sql
mysql > DELIMITER ??
```
成功执行这条 SQL 语句后，任何命令、语句或程序的结束标志就换为两个问号“??”了。

若希望换回默认的分号“;”作为结束标志，则在 MySQL 命令行客户端输入下列语句即可：
```sql
mysql > DELIMITER ;
```
注意：`DELIMITER`和分号`;`之间一定要有一个空格。在创建存储过程时，必须具有`CREATE ROUTINE`权限。

创建名称为`ShowStuScore`的存储过程，存储过程的作用是从学生成绩信息表中查询学生的成绩信息，输入的 SQL 语句和执行过程如下所示。
```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE ShowStuScore()
    -> BEGIN
    -> SELECT * FROM tb_students_score;
    -> END //
Query OK， 0 rows affected (0.09 sec)
```
结果显示 ShowStuScore 存储过程已经创建成功。

创建名称为`GetScoreByStu`的存储过程，输入参数是学生姓名。存储过程的作用是通过输入的学生姓名从学生成绩信息表中查询指定学生的成绩信息，输入的 SQL 语句和执行过程如下所示。
```sql
mysql> DELIMITER //
mysql> CREATE PROCEDURE GetScoreByStu
    -> (IN name VARCHAR(30))
    -> BEGIN
    -> SELECT student_score FROM tb_students_score
    -> WHERE student_name=name;
    -> END //
Query OK, 0 rows affected (0.01 sec)
```
# 查看存储过程
创建好存储过程后，用户可以通过`SHOW ATATUS`语句来查看存储过程的状态，也可以通过`SHOW CREATE`语句来查看存储过程的定义。
## 查看存储过程的状态
MySQL 中可以通过`SHOW STATUS`语句查看存储过程的状态：
```sql
SHOW PROCEDURE STATUS LIKE 存储过程名;
```
`LIKE`存储过程名用来匹配存储过程的名称，`LIKE`不能省略。
```sql
CREATE TABLE `studentinfo` (
    `ID` int(11) NOT NULL,
    `NAME` varchar(20) DEFAULT NULL,
    `SCORE` decimal(4,2) DEFAULT NULL,
    `SUBJECT` varchar(20) DEFAULT NULL,
    `TEACHER` varchar(20) DEFAULT NULL,
    PRIMARY KEY (`ID`)
);

mysql> INSERT INTO studentinfo(id,name,score) VALUES(1,"zhangsan",80),(2,"lisi","70");
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> DELIMITER //
mysql> CREATE PROCEDURE showstuscore()
    -> BEGIN
    -> SELECT id,name,score FROM studentinfo;
    -> END //
Query OK, 0 rows affected (0.07 sec)

mysql> SHOW PROCEDURE STATUS LIKE 'showstuscore' \G
*************************** 1. row ***************************
                  Db: test
                Name: showstuscore
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2020-02-20 13:34:50
             Created: 2020-02-20 13:34:50
       Security_type: DEFINER
             Comment:
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.01 sec)

mysql> SHOW PROCEDURE STATUS LIKE 'show%' \G
*************************** 1. row ***************************
                  Db: test
                Name: showstuscore
                Type: PROCEDURE
             Definer: root@localhost
            Modified: 2020-02-21 09:34:50
             Created: 2020-02-21 09:34:50
       Security_type: DEFINER
             Comment:
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.00 sec)
```
查询结果显示了存储过程的创建时间、修改时间和字符集等信息。
## 查看存储过程的定义
MySQL 中可以通过`SHOW CREATE`语句查看存储过程的状态：
```sql
SHOW CREATE PROCEDURE 存储过程名;
```
```sql
mysql> SHOW CREATE PROCEDURE showstuscore \G
*************************** 1. row ***************************
           Procedure: showstuscore
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `showstuscore`()
BEGIN
SELECT id,name,score FROM studentinfo;
END
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.01 sec)
```
查询结果显示了存储过程的定义和字符集信息等。

`SHOW STATUS`语句只能查看存储过程是操作的哪一个数据库、存储过程的名称、类型、谁定义的、创建和修改时间、字符编码等信息。但是，这个语句不能查询存储过程的集体定义，如果需要查看详细定义，需要使用`SHOW CREATE`语句。

存储过程的信息都存储在`information_schema`数据库下的`Routines`表中，可以通过查询该表的记录来查询存储过程的信息，SQL 语句如下：
```sql
SELECT * FROM information_schema.Routines WHERE ROUTINE_NAME=存储过程名;
```
在`information_schema`数据库下的 routines 表中，存储着所有存储过程的定义。所以，使用`SELECT`语句查询`routines`表中的存储过程和函数的定义时，一定要使用`routine_name`字段指定存储过程的名称，否则，将查询出所有的存储过程的定义。
# 修改存储过程
MySQL 中通过`ALTER PROCEDURE`语句来修改存储过程。
```sql
ALTER PROCEDURE 存储过程名 [ 特征 ... ]
```
特征指定了存储过程的特性，可能的取值有：
* `CONTAINS SQL`表示子程序包含 SQL 语句，但不包含读或写数据的语句。
* `NO SQL`表示子程序中不包含 SQL 语句。
* `READS SQL DATA`表示子程序中包含读数据的语句。
* `MODIFIES SQL DATA`表示子程序中包含写数据的语句。
* `SQL SECURITY { DEFINER |INVOKER }`指明谁有权限来执行。
* `DEFINER`表示只有定义者自己才能够执行。
* `INVOKER`表示调用者可以执行。
* `COMMENT 'string'`表示注释信息。

下面修改存储过程 showstuscore 的定义，将读写权限改为 MODIFIES SQL DATA，并指明调用者可以执行，代码如下：
```sql
mysql> ALTER PROCEDURE showstuscore MODIFIES SQL DATA SQL SECURITY INVOKER;
Query OK, 0 rows affected (0.01 sec)
执行代码，并查看修改后的信息，运行结果如下：
mysql> SHOW CREATE PROCEDURE showstuscore \G
*************************** 1. row ***************************
           Procedure: showstuscore
            sql_mode: STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    Create Procedure: CREATE DEFINER=`root`@`localhost` PROCEDURE `showstuscore`()
    MODIFIES SQL DATA
    SQL SECURITY INVOKER
BEGIN
SELECT id,name,score FROM studentinfo;
END
character_set_client: gbk
collation_connection: gbk_chinese_ci
  Database Collation: latin1_swedish_ci
1 row in set (0.00 sec)
```
结果显示，存储过程修改成功。从运行结果可以看到，访问数据的权限已经变成了`MODIFIES SQL DATA`，安全类型也变成了`INVOKE`。

提示：`ALTER PROCEDURE`语句用于修改存储过程的某些特征。如果要修改存储过程的内容，可以先删除原存储过程，再以相同的命名创建新的存储过程；如果要修改存储过程的名称，可以先删除原存储过程，再以不同的命名创建新的存储过程。
# 删除存储过程
存储过程被创建后，就会一直保存在数据库服务器上，直至被删除。当 MySQL 数据库中存在废弃的存储过程时，我们需要将它从数据库中删除。

MySQL 中使用`DROP PROCEDURE`语句来删除数据库中已经存在的存储过程。
```sql
DROP PROCEDURE [ IF EXISTS ] <过程名>
```
语法说明：
* 过程名：指定要删除的存储过程的名称。
* `IF EXISTS`：指定这个关键字，用于防止因删除不存在的存储过程而引发的错误。

注意：存储过程名称后面没有参数列表，也没有括号，在删除之前，必须确认该存储过程没有任何依赖关系，否则会导致其他与之关联的存储过程无法运行。

下面删除存储过程`ShowStuScore`：
```sql
mysql> DROP PROCEDURE ShowStuScore;
Query OK, 0 rows affected (0.08 sec)
```
删除后，可以通过查询`information_schema`数据库下的`routines`表来确认上面的删除是否成功。
```sql
mysql> SELECT * FROM information_schema.routines WHERE routine_name='ShowStuScore';
Empty set (0.03 sec)
```
结果显示，没有查询出任何记录，说明存储过程`ShowStuScore`已经被删除了。
# 存储函数
存储函数和存储过程一样，都是在数据库中定义一些 SQL 语句的集合。存储函数可以通过`return`语句返回函数值，主要用于计算并返回一个值。而存储过程没有直接返回值，主要用于执行操作。

在 MySQL 中，使用`CREATE FUNCTION`语句来创建存储函数：
```sql
CREATE FUNCTION sp_name ([func_parameter[...]])
RETURNS type
[characteristic ...] routine_body
```
其中：
* `sp_name`参数：表示存储函数的名称；
* `func_parameter`：表示存储函数的参数列表；
* `RETURNS type`：指定返回值的类型；
* `characteristic`参数：指定存储函数的特性，该参数的取值与存储过程是一样的；
* `routine_body`参数：表示 SQL 代码的内容，可以用`BEGIN...END`来标示 SQL 代码的开始和结束。

注意：在具体创建函数时，函数名不能与已经存在的函数名重名。除了上述要求外，推荐函数名命名（标识符）为`function_xxx`或者`func_xxx`。

`func_parameter`可以由多个参数组成，其中每个参数由参数名称和参数类型组成，其形式如下：
```sql
[IN | OUT | INOUT] param_name type;
```
其中：
* `IN`表示输入参数，`OUT`表示输出参数，`INOUT`表示既可以输入也可以输出；
* `param_name`参数是存储函数的参数名称；
* `type`参数指定存储函数的参数类型，该类型可以是 MySQL 数据库的任意数据类型。

```sql
mysql> USE test;
Database changed
mysql> DELIMITER //
mysql> CREATE FUNCTION func_student(id INT(11))
    -> RETURNS VARCHAR(20)
    -> COMMENT '查询某个学生的姓名'
    -> BEGIN
    -> RETURN(SELECT name FROM tb_student WHERE tb_student.id = id);
    -> END //
Query OK, 0 rows affected (0.10 sec)
mysql> DELIMITER ;
```
上述代码中，创建了`func_student`函数，该函数拥有一个类型为`INT(11)`的参数`id`，返回值为`VARCHAR(20)`类型。`SELECT`语句从`tb_student`表中查询`id`字段值等于所传入参数`id`值的记录，同时返回该条记录的`name`字段值。

创建函数与创建存储过程一样，需要通过命令`DELIMITER //`将 SQL 语句的结束符由“;”修改为“//”，最后通过命令`DELIMITER`; 将结束符号修改成 SQL 语句中默认的结束符号。

如果在存储函数中的`RETURN`语句返回一个类型不同于函数的`RETURNS`子句中指定类型的值，返回值将被强制为恰当的类型。比如，如果一个函数返回一个`ENUM`或`SET`值，但是`RETURN`语句返回一个整数，对于`SET`成员集的相应的`ENUM`成员，从函数返回的值是字符串。

查看存储函数的语法如下：
```
SHOW FUNCTION STATUS LIKE 存储函数名;
SHOW CREATE FUNCTION 存储函数名;
SELECT * FROM information_schema.Routines WHERE ROUTINE_NAME=存储函数名;
```
可以发现，操作存储函数和操作存储过程不同的是将`PROCEDURE`替换成了`FUNCTION`。同样，修改存储函数的语法如下：
```
ALTER FUNCTION 存储函数名 [ 特征 ... ]
```
存储函数的特征与存储过程的基本一样。

删除存储过程的语法如下：
```
DROP FUNCTION [ IF EXISTS ] <函数名>
```
# 调用存储过程和函数
存储过程和存储函数都是存储在服务器端的 SQL 语句集合。要想使用这些已经定义好的存储过程和存储函数就必须要通过调用的方式来实现。

存储过程通过`CALL`语句来调用，存储函数的使用方法与 MySQL 内部函数的使用方法相同。执行存储过程和存储函数需要拥有`EXECUTE`权限（`EXECUTE`权限的信息存储在`information_schema`数据库下的`USER_PRIVILEGES`表中）。
## 调用存储过程
MySQL 中使用`CALL`语句来调用存储过程。调用存储过程后，数据库系统将执行存储过程中的 SQL 语句，然后将结果返回给输出值。

`CALL`语句接收存储过程的名字以及需要传递给它的任意参数，基本语法形式如下：
```
CALL sp_name([parameter[...]]);
```
其中，`sp_name`表示存储过程的名称，`parameter`表示存储过程的参数。

```sql
mysql> DELIMITER ;
mysql> CALL ShowStuScore();
+--------------+---------------+
| student_name | student_score |
+--------------+---------------+
| Dany         |            90 |
| Green        |            99 |
| Henry        |            95 |
| Jane         |            98 |
| Jim          |            88 |
| John         |            94 |
| Lily         |           100 |
| Susan        |            96 |
| Thomas       |            93 |
| Tom          |            89 |
+--------------+---------------+
10 rows in set (0.00 sec)
Query OK, 0 rows affected (0.02 sec)

mysql> CALL GetScoreByStu('Green');
+---------------+
| student_score |
+---------------+
|            99 |
+---------------+
1 row in set (0.03 sec)
Query OK, 0 rows affected (0.03 sec)
```
因为存储过程实际上也是一种函数，所以存储过程名后需要有( )符号，即使不传递参数也需要。
## 调用存储函数
在 MySQL 中，存储函数的使用方法与 MySQL 内部函数的使用方法是一样的。换言之，用户自己定义的存储函数与 MySQL 内部函数是一个性质的。区别在于，存储函数是用户自己定义的，而内部函数是 MySQL 开发者定义的。
```sql
mysql> SELECT func_student(3);
+-----------------+
| func_student(3) |
+-----------------+
| 王五            |
+-----------------+
1 row in set (0.10 sec)
```
通过示例的比较，可以看出虽然存储函数和存储过程的定义稍有不同，但它们都可以实现相同的功能，我们应该在实际应用中灵活选择。
# 变量的定义和赋值
在 MySQL 中，除了支持标准的存储过程和函数外，还引入了表达式。表达式与其它高级语言的表达式一样，由变量、运算符和流程控制来构成。

变量是表达式语句中最基本的元素，可以用来临时存储数据。在存储过程和函数中都可以定义和使用变量。用户可以使用`DECLARE`关键字来定义变量，定义后可以为变量赋值。这些变量的作用范围是`BEGIN...END`程序段中。

## 1. 定义变量
MySQL 中可以使用`DECLARE`关键字来定义变量，其基本语法如下：
```
DECLARE var_name[,...] type [DEFAULT value]
```
其中：
* `DECLARE`关键字是用来声明变量的；
* `var_name`参数是变量的名称，这里可以同时定义多个变量；
* `type`参数用来指定变量的类型；
* `DEFAULT value`子句将变量默认值设置为`value`，没有使用`DEFAULT`子句时，默认值为`NULL`。

下面定义变量`my_sql`，数据类型为`INT`类型，默认值为 10。
```
DECLARE my_sql INT DEFAULT 10;
```
## 2. 为变量赋值
MySQL 中可以使用`SET`关键字来为变量赋值：
```
SET var_name = expr[,var_name = expr]...
```
其中：
* `SET`关键字用来为变量赋值；
* `var_name`参数是变量的名称；
* `expr`参数是赋值表达式。

注意：一个`SET`语句可以同时为多个变量赋值，各个变量的赋值语句之间用逗号隔开。

下面为变量`my_sql`赋值为 30。
```
SET my_sql=30;
```
MySQL 中还可以使用`SELECT..INTO`语句为变量赋值。
```sql
SELECT col_name [...] INTO var_name[,...]
FROM table_name WEHRE condition
```
其中：
* `col_name`参数表示查询的字段名称；
* `var_name`参数是变量的名称；
* `table_name`参数指表的名称；
* `condition`参数指查询条件。

注意：当将查询结果赋值给变量时，该查询语句的返回结果只能是单行。

下面从`tb_student`表中查询`id`为 2 的记录，将该记录的`id`值赋给变量`my_sql`。
```sql
SELECT id INTO my_sql FROM tb_student WEHRE id=2;
```
# 定义条件和处理程序
在程序的运行过程中可能会遇到问题，此时我们可以通过定义条件和处理程序来事先定义这些问题。

定义条件是指事先定义程序执行过程中遇到的问题，处理程序定义了在遇到这些问题时应当采取的处理方式和解决办法，保证存储过程和函数在遇到警告或错误时能继续执行，从而增强程序处理问题的能力，避免程序出现异常被停止执行。
## 1. 定义条件
MySQL 中可以使用`DECLARE`关键字来定义条件。
```sql
DECLARE condition_name CONDITION FOR condition_value
condition value:
SQLSTATE [VALUE] sqlstate_value | mysql_error_code
```
其中：
* `condition_name`参数表示条件的名称；
* `condition_value`参数表示条件的类型；
* `sqlstate_value`参数和`mysql_error_code`参数都可以表示 MySQL 的错误。`sqlstate_value`表示长度为 5 的字符串类型错误代码，`mysql_error_code`表示数值类型错误代码。例如`ERROR 1146(42S02)`中，`sqlstate_value`值是 42S02，`mysql_error_code`值是 1146。

下面定义“ERROR 1146 (42S02)”这个错误，名称为 can_not_find。 可以用两种不同的方法来定义，代码如下：
```
//方法一：使用sqlstate_value
DECLARE can_not_find CONDITION FOR SQLSTATE '42S02';

//方法二：使用 mysql_error_code
DECLARE can_not_find CONDITION FOR 1146;
```
## 2. 定义处理程序
MySQL 中可以使用`DECLARE`关键字来定义处理程序。
```SQL
DECLARE handler_type HANDLER FOR condition_value[...] sp_statement
handler_type:
CONTINUE | EXIT | UNDO
condition_value:
SQLSTATE [VALUE] sqlstate_value | condition_name | SQLWARNING | NOT FOUND | SQLEXCEPTION | mysql_error_code
```
其中，handler_type 参数指明错误的处理方式，该参数有 3 个取值。这 3 个取值分别是 CONTINUE、EXIT 和 UNDO。
* `CONTINUE`表示遇到错误不进行处理，继续向下执行；
* `EXIT`表示遇到错误后马上退出；
* `UNDO`表示遇到错误后撤回之前的操作，MySQL 中暂时还不支持这种处理方式。

注意：通常情况下，执行过程中遇到错误应该立刻停止执行下面的语句，并且撤回前面的操作。但是，MySQL 中现在还不能支持 UNDO 操作。因此，遇到错误时最好执行 EXIT 操作。如果事先能够预测错误类型，并且进行相应的处理，那么可以执行 CONTINUE 操作。

参数指明错误类型，该参数有 6 个取值：
* `sqlstate_value`：包含 5 个字符的字符串错误值；
* `condition_name`：表示`DECLARE`定义的错误条件名称；
* `SQLWARNING`：匹配所有以 01 开头的`sqlstate_value`值；
* `NOT FOUND`：匹配所有以 02 开头的`sqlstate_value`值；
* `SQLEXCEPTION`：匹配所有没有被`SQLWARNING`或`NOT FOUND`捕获的`sqlstate_value`值；
* `mysql_error_code`：匹配数值类型错误代码。

`sp_statement`参数为程序语句段，表示在遇到定义的错误时，需要执行的一些存储过程或函数。

下面是定义处理程序的几种方式，代码如下：
```SQL
//方法一：捕获 sqlstate_value
DECLARE CONTINUE HANDLER FOR SQLSTATE '42S02' SET @info='CAN NOT FIND';

//方法二：捕获 mysql_error_code
DECLARE CONTINUE HANDLER FOR 1146 SET @info='CAN NOT FIND';

//方法三：先定义条件，然后调用
DECLARE can_not_find CONDITION FOR 1146;
DECLARE CONTINUE HANDLER FOR can_not_find SET @info='CAN NOT FIND';

//方法四：使用 SQLWARNING
DECLARE EXIT HANDLER FOR SQLWARNING SET @info='ERROR';

//方法五：使用 NOT FOUND
DECLARE EXIT HANDLER FOR NOT FOUND SET @info='CAN NOT FIND';

//方法六：使用 SQLEXCEPTION
DECLARE EXIT HANDLER FOR SQLEXCEPTION SET @info='ERROR';
```
上述代码是 6 种定义处理程序的方法。
捕获 sqlstate_value 值。如果遇到 sqlstate_value 值为 42S02，执行 CONTINUE 操作，并且输出“CAN NOT FIND”信息。
捕获 mysql_error_code 值。如果遇到 mysql_error_code 值为 1146， 执行 CONTINUE 操作，并且输出“CAN NOT FIND”信息。
先定义条件，然后再调用条件。这里先定义 can_not_find 条件，遇到 1146 错误就执行 CONTINUE 操作。
使用 SQLWARNING。SQLWARNING 捕获所有以 01 开头的 sqlstate_value 值，然后执行 EXIT 操作，并且输出“ERROR"信息。
使用 NOT FOUND。NOT FOUND 捕获所有以 02 开头的 sqlstate_value 值，然后执行 EXIT 操作，并且输出“CAN NOT FIND”信息。
使用 SQLEXCEPTION。 SQLEXCEPTION 捕获所有没有被 SQLWARNING 或 NOT FOUND 捕获的 sqlstate_value 值，然后执行 EXIT 操作，并且输出“ERROR”信息。

定义条件和处理顺序，具体的执行过程如下：
```
mysql> CREATE TABLE t8(s1 INT,PRIMARY KEY(s1));
Query OK, 0 rows affected (0.07 sec)

mysql> DELIMITER //
mysql> CREATE PROCEDURE handlerdemo()
    -> BEGIN
    -> DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' SET @X2=1;
    -> SET @X=1;
    -> INSERT INTO t8 VALUES(1);
    -> SET @X=2;
    -> INSERT INTO t8 VALUES(1);
    -> SET @X=3;
    -> END //
Query OK, 0 rows affected (0.02 sec)

mysql> DELIMITER ;
mysql> CALL handlerdemo();
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT @X;
+------+
| @X   |
+------+
|    3 |
+------+
1 row in set (0.00 sec)
```
上述代码中，@X 是一个用户变量，执行结果 @X 等于 3，这表明 MySQL 执行到程序的末尾。

如果`DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' SET @X2=1;`这一行不存在，第二个 INSERT 因 PRIMARY KEY 约束而失败之后，MySQL 可能已经采取 EXIT 策略，且 SELECT @X 可能已经返回 2。

注意：@X 表示用户变量，使用 SET 语句为其赋值，用户变量与连接有关，一个客户端定义的变量不能被其他客户端所使用，当客户端退出时，该客户端连接的所有变量将自动释放。
# 游标
在 MySQL 中，存储过程或函数中的查询有时会返回多条记录，而使用简单的`SELECT`语句，没有办法得到第一行、下一行或前十行的数据，这时可以使用游标来逐条读取查询结果集中的记录。游标在部分资料中也被称为光标。

关系数据库管理系统实质是面向集合的，在 MySQL 中并没有一种描述表中单一记录的表达形式，除非使用 WHERE 子句来限制只有一条记录被选中。所以有时我们必须借助于游标来进行单条记录的数据处理。

一般通过游标定位到结果集的某一行进行数据修改。
结果集是符合 SQL 语句的所有记录的集合。

个人理解游标就是一个标识，用来标识数据取到了什么地方，如果你了解编程语言，可以把他理解成数组中的下标。

不像多数 DBMS，MySQL 游标只能用于存储过程和函数。
 

下面介绍游标的使用，主要包括游标的声明、打开、使用和关闭。
1. 声明游标
MySQL 中使用 DECLARE 关键字来声明游标，并定义相应的 SELECT 语句，根据需要添加 WHERE 和其它子句。其语法的基本形式如下：
```
DECLARE cursor_name CURSOR FOR select_statement;
```
其中，`cursor_name`表示游标的名称；`select_statement`表示`SELECT`语句，可以返回一行或多行数据。
例 1
下面声明一个名为 nameCursor 的游标，代码如下：
```
mysql> DELIMITER //
mysql> CREATE PROCEDURE processnames()
    -> BEGIN
    -> DECLARE nameCursor CURSOR
    -> FOR
    -> SELECT name FROM tb_student;
    -> END//
Query OK, 0 rows affected (0.07 sec)
```
以上语句定义了 nameCursor 游标，游标只局限于存储过程中，存储过程处理完成后，游标就消失了。
2. 打开游标
声明游标之后，要想从游标中提取数据，必须首先打开游标。在 MySQL 中，打开游标通过`OPEN`关键字来实现：
```
OPEN cursor_name;
```
其中，`cursor_name`表示所要打开游标的名称。需要注意的是，打开一个游标时，游标并不指向第一条记录，而是指向第一条记录的前边。

在程序中，一个游标可以打开多次。用户打开游标后，其他用户或程序可能正在更新数据表，所以有时会导致用户每次打开游标后，显示的结果都不同。
3. 使用游标
游标顺利打开后，可以使用`FETCH...INTO`语句来读取数据：
```
FETCH cursor_name INTO var_name [,var_name]...
```
上述语句中，将游标 cursor_name 中 SELECT 语句的执行结果保存到变量参数 var_name 中。变量参数 var_name 必须在游标使用之前定义。使用游标类似高级语言中的数组遍历，当第一次使用游标时，此时游标指向结果集的第一条记录。

MySQL 的游标是只读的，也就是说，你只能顺序地从开始往后读取结果集，不能从后往前，也不能直接跳到中间的记录。
4. 关闭游标
游标使用完毕后，要及时关闭，在 MySQL 中，使用`CLOSE`关键字关闭游标：
```
CLOSE cursor_name;
```
`CLOSE`释放游标使用的所有内部内存和资源，因此每个游标不再需要时都应该关闭。

在一个游标关闭后，如果没有重新打开，则不能使用它。但是，使用声明过的游标不需要再次声明，用`OPEN`语句打开它就可以了。

如果你不明确关闭游标，MySQL 将会在到达 END 语句时自动关闭它。游标关闭之后，不能使用 FETCH 来使用该游标。

创建 users 数据表，并插入数据，SQL 语句和运行结果如下：
```
mysql> CREATE TABLE `users`
    ->  (
    ->  `ID` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
    ->  `user_name` VARCHAR(60),
    ->  `user_pass` VARCHAR(64),
    ->  PRIMARY KEY (`ID`)
    -> );
Query OK, 0 rows affected (0.06 sec)

mysql> INSERT INTO users VALUES(null,'sheng','sheng123'),
    -> (null,'yu','yu123'),
    -> (null,'ling','ling123');
Query OK, 3 rows affected (0.01 sec)
```
创建存储过程 test_cursor，并创建游标 cur_test，查询 users 数据表中的第 3 条记录，SQL 语句和执行过程如下：
```
mysql> DELIMITER //
mysql> CREATE PROCEDURE test_cursor (in param INT(10),out result VARCHAR(90))
    -> BEGIN
    -> DECLARE name VARCHAR(20);
    -> DECLARE pass VARCHAR(20);
    -> DECLARE done INT;
    -> DECLARE cur_test CURSOR FOR SELECT user_name,user_pass FROM users;
    -> DECLARE continue handler FOR SQLSTATE '02000' SET done = 1;
    -> IF param THEN INTO result FROM users WHERE id = param;
    -> ELSE
    -> OPEN cur_test;
    -> repeat
    -> FETCH cur_test into name,pass;
    -> SELECT concat_ws(',',result,name,pass) INTO result;
    -> until done
    -> END repeat;
    -> CLOSE cur_test;
    -> END IF;
    -> END //
Query OK, 0 rows affected (0.10 sec)

mysql> call test_cursor(3,@test)//
Query OK, 1 row affected (0.03 sec)

mysql> select @test//
+-----------+
| @test     |
+-----------+
| ling,ling123 |
+-----------+
1 row in set (0.00 sec)
```
创建 pro_users() 存储过程，定义 cur_1 游标，将表 users 中的 user_name 字段全部修改为 MySQL，SQL 语句和执行过程如下。
```
mysql> CREATE PROCEDURE pro_users()
    -> BEGIN
    -> DECLARE result VARCHAR(100);
    -> DECLARE no INT;
    -> DECLARE cur_1 CURSOR FOR SELECT user_name FROM users;
    -> DECLARE CONTINUE HANDLER FOR NOT FOUND SET no=1;
    -> SET no=0;
    -> OPEN cur_1;
    -> WHILE no=0 do
    -> FETCH cur_1 into result;
    -> UPDATE users SET user_name='MySQL'
    -> WHERE user_name=result;
    -> END WHILE;
    -> CLOSE cur_1;
    -> END //
Query OK, 0 rows affected (0.05 sec)

mysql> call pro_users() //
Query OK, 0 rows affected (0.03 sec)

mysql> SELECT * FROM users //
+----+-----------+-----------+
| ID | user_name | user_pass |
+----+-----------+-----------+
|  1 | MySQL     | sheng      |
|  2 | MySQL     | zhang     |
|  3 | MySQL     | ying      |
+----+-----------+-----------+
3 rows in set (0.00 sec)
```
结果显示，users 表中的 user_name 字段已经全部修改为 MySQL。
# 流程控制语句
在存储过程和自定义函数中可以使用流程控制语句来控制程序的流程。MySQL 中流程控制语句有：`IF`语句、`CASE`语句、`LOOP`语句、`LEAVE`语句、`ITERATE`语句、`REPEAT`语句和`WHILE`语句等。
## 1. IF语句
IF 语句用来进行条件判断，根据是否满足条件（可包含多个条件），来执行不同的语句，是流程控制中最常用的判断语句。其语法的基本形式如下：
```SQL
IF search_condition THEN statement_list
    [ELSEIF search_condition THEN statement_list]...
    [ELSE statement_list]
END IF
```
其中，`search_condition`参数表示条件判断语句，如果返回值为`TRUE`，相应的 SQL 语句列表（`statement_list`）被执行；如果返回值为`FALSE`，则`ELSE`子句的语句列表被执行。`statement_list`可以包括一个或多个语句。

注意：MySQL 中的`IF()`函数不同于这里的`IF`语句。
```
IF age>20 THEN SET @count1=@count1+1;
    ELSEIF age=20 THEN @count2=@count2+1;
    ELSE @count3=@count3+1;
END lF;
```
`IF`语句都需要使用`END IF`来结束。
## 2. CASE语句
`CASE`语句也是用来进行条件判断的，它提供了多个条件进行选择，可以实现比`IF`语句更复杂的条件判断。
```
CASE case_value
    WHEN when_value THEN statement_list
    [WHEN when_value THEN statement_list]...
    [ELSE statement_list]
END CASE
```
其中：
* `case_value`参数表示条件判断的变量，决定了哪一个`WHEN`子句会被执行；
* `when_value`参数表示变量的取值，如果某个`when_value`表达式与`case_value`变量的值相同，则执行对应的`THEN`关键字后的`statement_list`中的语句；
* `statement_list`参数表示`when_value`值没有与`case_value`相同值时的执行语句。
* `CASE`语句都要使用`END CASE`结束。

`CASE`语句还有另一种形式。
```
CASE
    WHEN search_condition THEN statement_list
    [WHEN search_condition THEN statement_list] ...
    [ELSE statement_list]
END CASE
```
其中，`search_condition`参数表示条件判断语句；`statement_list`参数表示不同条件的执行语句。

与上述语句不同的是，该语句中的 WHEN 语句将被逐个执行，直到某个 search_condition 表达式为真，则执行对应 THEN 关键字后面的 statement_list 语句。如果没有条件匹配，ELSE 子句里的语句被执行。

这里介绍的 CASE 语句与“控制流程函数”里描述的 SQL CASE 表达式的 CASE 语句有轻微的不同。这里的 CASE 语句不能有 ELSE NULL 语句，并且用 END CASE 替代 END 来终止。

```
CASE age
    WHEN 20 THEN SET @count1=@count1+1;
    ELSE SET @count2=@count2+1;
END CASE;
# 代码也可以是下面的形式：
CASE
    WHEN age=20 THEN SET @count1=@count1+1;
    ELSE SET @count2=@count2+1;
END CASE;
```
## 3. LOOP 语句
`LOOP`语句可以使某些特定的语句重复执行。与`IF`和`CASE`语句相比，`LOOP`只实现了一个简单的循环，并不进行条件判断。

`LOOP`语句本身没有停止循环的语句，必须使用`LEAVE`语句等才能停止循环，跳出循环过程。
```SQL
[begin_label:]LOOP
    statement_list
END LOOP [end_label]
```
其中，`begin_label`参数和`end_label`参数分别表示循环开始和结束的标志，这两个标志必须相同，而且都可以省略；`statement_list`参数表示需要循环执行的语句。
```SQL
add_num:LOOP
    SET @count=@count+1;
END LOOP add_num;
```
该示例循环执行`count`加 1 的操作。因为没有跳出循环的语句，这个循环成了一个死循环。`LOOP`循环都以`END LOOP`结束。
## 4. LEAVE 语句
`LEAVE`语句主要用于跳出循环控制。
```
LEAVE label
```
其中，`label`参数表示循环的标志，`LEAVE`语句必须跟在循环标志前面。

下面是一个 LEAVE 语句的示例。代码如下：
```
add_num:LOOP
    SET @count=@count+1;
    IF @count=100 THEN
        LEAVE add_num;
END LOOP add num;
```
该示例循环执行 count 加 1 的操作。当 count 的值等于 100 时，跳出循环。
## 5. ITERATE 语句
`ITERATE`是“再次循环”的意思，用来跳出本次循环，直接进入下一次循环。
```
ITERATE label
```
其中，`label`参数表示循环的标志，`ITERATE`语句必须跟在循环标志前面。
```SQL
add_num:LOOP
    SET @count=@count+1;
    IF @count=100 THEN
        LEAVE add_num;
    ELSE IF MOD(@count,3)=0 THEN
        ITERATE add_num;
    SELECT * FROM employee;
END LOOP add_num;
```
该示例循环执行 count 加 1 的操作，count 值为 100 时结束循环。如果 count 的值能够整除 3，则跳出本次循环，不再执行下面的 SELECT 语句。

说明：LEAVE 语句和 ITERATE 语句都用来跳出循环语句，但两者的功能是不一样的。LEAVE 语句是跳出整个循环，然后执行循环后面的程序。而 ITERATE 语句是跳出本次循环，然后进入下一次循环。使用这两个语句时一定要区分清楚。
## 6. REPEAT 语句
`REPEAT`语句是有条件控制的循环语句，每次语句执行完毕后，会对条件表达式进行判断，如果表达式返回值为`TRUE`，则循环结束，否则重复执行循环中的语句。
```SQL
[begin_label:] REPEAT
    statement_list
    UNTIL search_condition
END REPEAT [end_label]
```
其中：
* `begin_label`为`REPEAT`语句的标注名称，该参数可以省略；
* `REPEAT`语句内的语句被重复，直至`search_condition`返回值为`TRUE`。
* `statement_list`参数表示循环的执行语句；
* `search_condition`参数表示结束循环的条件，满足该条件时循环结束。

`REPEAT`循环都用`END REPEAT`结束。
```SQL
REPEAT
    SET @count=@count+1;
    UNTIL @count=100
END REPEAT;
```
该示例循环执行`count`加 1 的操作，`count`值为 100 时结束循环。
## 7. WHILE 语句
`WHILE`语句也是有条件控制的循环语句。`WHILE`语句和`REPEAT`语句不同的是，`WHILE`语句是当满足条件时，执行循环内的语句，否则退出循环。`WHILE`语句的基本语法形式如下：
```SQL
[begin_label:] WHILE search_condition DO
    statement list
END WHILE [end label]
```
其中，`search_condition`参数表示循环执行的条件，满足该条件时循环执行；`statement_list`参数表示循环的执行语句。`WHILE`循环需要使用`END WHILE`来结束。
```SQL
WHILE @count<100 DO
    SET @count=@count+1;
END WHILE;
```
该示例循环执行`count`加 1 的操作，`count`值小于 100 时执行循环。如果`count`值等于 100 了，则跳出循环。