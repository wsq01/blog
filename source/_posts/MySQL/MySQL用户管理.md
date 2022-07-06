---
title: MySQL 用户管理
date: 2020-05-31 11:48:16
tags: [MySQL]
categories: [MySQL]
---

MySQL 是一个多用户数据库，具有功能强大的访问控制系统，可以为不同用户指定不同权限。
# user权限表
MySQL 在安装时会自动创建一个名为`mysql`的数据库，`mysql`数据库中存储的都是用户权限表。用户登录以后，MySQL 会根据这些权限表的内容为每个用户赋予相应的权限。

`user`表是 MySQL 中最重要的一个权限表，用来记录允许连接到服务器的账号信息。需要注意的是，在`user`表里启用的所有权限都是全局级的，适用于所有数据库。

user 表中的字段大致可以分为 4 类，分别是用户列、权限列、安全列和资源控制列。
## 用户列
用户列存储了用户连接 MySQL 数据库时需要输入的信息。需要注意的是 MySQL 5.7 版本不再使用`Password`来作为密码的字段，而改成了`authentication_string`。

MySQL 5.7 版本的用户列：

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| Host | char(60) | NO | 无 | 主机名 |
| User | char(32) | NO | 无 | 用户名 |
| authentication_string | text | YES | 无 | 密码 |

用户登录时，如果这 3 个字段同时匹配，MySQL 数据库系统才会允许其登录。创建新用户时，也是设置这 3 个字段的值。修改用户密码时，实际就是修改`user`表的`authentication_string`字段的值。因此，这 3 个字段决定了用户能否登录。
## 权限列
权限列的字段决定了用户的权限，用来描述在全局范围内允许对数据和数据库进行的操作。

权限大致分为两大类，分别是高级管理权限和普通权限：
* 高级管理权限主要对数据库进行管理，例如关闭服务的权限、超级权限和加载用户等；
* 普通权限主要操作数据库，例如查询权限、修改权限等。

`user`表的权限列包括`Select_priv、Insert_ priv`等以`priv`结尾的字段，这些字段值的数据类型为`ENUM`，可取的值只有`Y`和`N`：`Y`表示该用户有对应的权限，`N`表示该用户没有对应的权限。从安全角度考虑，这些字段的默认值都为`N`。

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| Select_priv | enum('N','Y') | NO | N | 是否可以通过SELECT 命令查询数据 |
| Insert_priv | enum('N','Y') | NO | N | 是否可以通过 INSERT 命令插入数据 |
| Update_priv | enum('N','Y') | NO | N | 是否可以通过UPDATE 命令修改现有数据 |
| Delete_priv | enum('N','Y') | NO | N | 是否可以通过DELETE 命令删除现有数据 |
| Create_priv | enum('N','Y') | NO | N | 是否可以创建新的数据库和表 |
| Drop_priv | enum('N','Y') | NO | N | 是否可以删除现有数据库和表 |
| Reload_priv | enum('N','Y') | NO | N | 是否可以执行刷新和重新加载MySQL所用的各种内部缓存的特定命令，包括日志、权限、主机、查询和表 |
| Shutdown_priv | enum('N','Y') | NO | N | 是否可以关闭MySQL服务器。将此权限提供给root账户之外的任何用户时，都应当非常谨慎 |
| Process_priv | enum('N','Y') | NO | N | 是否可以通过SHOW PROCESSLIST命令查看其他用户的进程 |
| File_priv | enum('N','Y') | NO | N | 是否可以执行SELECT INTO OUTFILE和LOAD DATA INFILE命令 |
| Grant_priv | enum('N','Y') | NO | N | 是否可以将自己的权限再授予其他用户 |
| References_priv | enum('N','Y') | NO | N | 是否可以创建外键约束 |
| Index_priv | enum('N','Y') | NO | N | 是否可以对索引进行增删查 |
| Alter_priv | enum('N','Y') | NO | N | 是否可以重命名和修改表结构 |
| Show_db_priv | enum('N','Y') | NO | N | 是否可以查看服务器上所有数据库的名字，包括用户拥有足够访问权限的数据库 |
| Super_priv | enum('N','Y') | NO | N | 是否可以执行某些强大的管理功能，例如通过KILL命令删除用户进程；使用SET GLOBAL命令修改全局MySQL变量，执行关于复制和日志的各种命令。（超级 权限） |
| Create_tmp_table_priv | enum('N','Y') | NO | N | 是否可以创建临时表 |
| Lock_tables_priv | enum('N','Y') | NO | N | 是否可以使用LOCK TABLES命令阻止对表的访问/修改 |
| Execute_priv | enum('N','Y') | NO | N | 是否可以执行存储过程 |
| Repl_slave_priv | enum('N','Y') | NO | N | 是否可以读取用于维护复制数据库环境的二进制日志文件 |
| Repl_client_priv | enum('N','Y') | NO | N | 是否可以确定复制从服务器和主服务器的位置 |
| Create_view_priv | enum('N','Y') | NO | N | 是否可以创建视图 |
| Show_view_priv | enum('N','Y') | NO | N | 是否可以查看视图 |
| Create_routine_priv | enum('N','Y') | NO | N | 是否可以更改或放弃存储过程和函数 |
| Alter_routine_priv | enum('N','Y') | NO | N | 是否可以修改或删除存储函数及函数 |
| Create_user_priv | enum('N','Y') | NO | N | 是否可以执行CREATE USER命令，这个命令用于创建新的MySQL账户 |
| Event_priv | enum('N','Y') | NO | N | 是否可以创建、修改和删除事件 |
| Trigger_priv | enum('N','Y') | NO | N | 是否可以创建和删除触发器 |
| Create_tablespace_priv | enum('N','Y') | NO | N | 是否可以创建表空间 |

如果要修改权限，可以使用`GRANT`语句为用户赋予一些权限，也可以通过`UPDATE`语句更新`user`表的方式来设置权限。
## 安全列
安全列主要用来判断用户是否能够登录成功。

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| ssl_type | enum('','ANY',<br>'X509', 'SPECIFIED') | NO | | 支持ssl标准加密安全字段 |
| ssl_cipher | blob | NO | | 支持ssl标准加密安全字段 |
| x509_issuer | blob | NO | | 支持x509标准字段 |
| x509_subject | blob | NO | | 支持x509标准字段 |
| plugin | char(64) | NO | mysql_native_<br>password | 引入plugins以进行用户连接时的密码验证，plugin创建外部/代理用户 |
| password_expired | enum('N','Y') | NO | N | 密码是否过期 (N 未过期，y 已过期) |
| password_last_changed | timestamp | YES | | 记录密码最近修改的时间 |
| password_lifetime | smallint(5) | unsigned | YES | | 设置密码的有效时间，单位为天数 |
| account_locked | enum('N','Y') | NO | N | 用户是否被锁定（Y 锁定，N 未锁定） |

注意：即使`password_expired`为`Y`，用户也可以使用密码登录 MySQL，但是不允许做任何操作。

通常标准的发行版不支持`ssl`，可以使用`SHOW VARIABLES LIKE "have_openssl"`语句来查看是否具有`ssl`功能。如果`have_openssl`的值为`DISABLED`，那么则不支持`ssl`加密功能。
## 资源控制列
资源控制列的字段用来限制用户使用的资源，user 表中的资源控制列：

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| max_questions | int(11) | unsigned | NO | 0 | 规定每小时允许执行查询的操作次数 |
| max_updates | int(11) | unsigned | NO | 0 | 规定每小时允许执行更新的操作次数 |
| max_connections | int(11) | unsigned | NO | 0 | 规定每小时允许执行的连接操作次数 |
| max_user_connections | int(11) | unsigned | NO | 0 | 规定允许同时建立的连接次数  |

以上字段的默认值为 0，表示没有限制。一个小时内用户查询或者连接数量超过资源控制限制，用户将被锁定，直到下一个小时才可以在此执行对应的操作。可以使用`GRANT`语句更新这些字段的值。
# 其它权限表
在 MySQL 数据库中，权限表除了`user`表外，还有`db`表、`tables_priv`表、`columns_priv`表和`procs_priv`表。
## db表
`db`表中存储了用户对某个数据库的操作权限。表中的字段大致可以分为两类，分别是用户列和权限列。
### 用户列
`db`表用户列有 3 个字段，分别是`Host、User、Db`，标识从某个主机连接某个用户对某个数据库的操作权限，这 3 个字段的组合构成了`db`表的主键。

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| Host | char(60) | NO | 无 | 主机名 |
| Db | char(64) | NO | 无 | 数据库名 |
| User | char(32) | NO | 无 | 用户名 |

### 权限列
`db`表中的权限列和`user`表中的权限列大致相同，只是`user`表中的权限是针对所有数据库的，而`db`表中的权限只针对指定的数据库。如果希望用户只对某个数据库有操作权限，可以先将`user`表中对应的权限设置为`N`，然后在`db`表中设置对应数据库的操作权限。
## tables_priv表和columns_priv表
`tables_priv`表用来对单个表进行权限设置，`columns_priv`表用来对单个数据列进行权限设置。`tables_priv`表结构如下表所示：

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| Host | char(60) | NO | 无 | 主机 |
| Db | char(64) | NO | 无 | 数据库名 |
| User | char(32) | NO | 无 | 用户名 |
| Table_name | har(64) | NO | 无 | 表名 |
| Grantor | char(93) | NO | 无 | 修改该记录的用户 |
| Timestamp | timestamp | NO | CURRENT_TIMESTAMP | 修改该记录的时间 |
| Table_priv | set('Select','Insert',<br>'Update','Delete',<br>'Create','Drop',<br>'Grant','References',<br>'Index','Alter','Create View',<br>'Show view','Trigger') | NO | 无 | 表示对表的操 作权限，包括 Select、Insert、Update、Delete、Create、Drop、Grant、References、Index 和 Alter 等 |
| Column_priv | set('Select','Insert',<br>'Update','References') | NO | 无 | 表示对表中的列的操作权限，包括 Select、Insert、Update 和 References |

`columns_priv`表结构如下表所示：

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| Host | char(60) | NO | 无 | 主机 |
| Db | char(64) | NO | 无 | 数据库名 |
| User | char(32) | NO | 无 | 用户名 |
| Table_name | char(64) | NO | 无 | 表名 |
| Column_name | char(64) | NO | 无 | 数据列名称，用来指定对哪些数据列具有操作权限 |
| Timestamp | timestamp | NO | CURRENT_TIMESTAMP | 修改该记录的时间 |
| Column_priv | set('Select',<br>'Insert',<br>'Update',<br>'References') | NO | 无 | 表示对表中的列的操作权限，包括 Select、Insert、Update 和 References |

## procs_priv表
`procs_priv`表可以对存储过程和存储函数进行权限设置，`procs_priv`的表结构如表所示：

| 字段名 | 字段类型 | 是否为空 | 默认值 | 说明 |
| :--: | :--: | :--: | :--: | :--: |
| Host | char(60) | NO | 无 | 主机名 |
| Db | char(64) | NO | 无 | 数据库名 |
| User | char(32) | NO | 无 | 用户名 |
| Routine_name | char(64) | NO | 无 | 表示存储过程或函数的名称 |
| Routine_type | enum(<br>'FUNCTION',<br>'PROCEDURE') | NO | 无 | 表示存储过程或函数的类型，Routine_type 字段有两个值，分别是 FUNCTION 和 PROCEDURE。FUNCTION 表示这是一个函数；PROCEDURE 表示这是一个存储过程。 |
| Grantor | char(93) | NO | 无 | 插入或修改该记录的用户 |
| Proc_priv | set(<br>'Execute',<br>'Alter Routine',<br>'Grant') | NO | 无 | 表示拥有的权限，包括 Execute、Alter Routine、Grant 3种 |
| Timestamp | timestamp | NO | CURRENT_TIMESTAMP | 表示记录更新时间 |

# 创建用户
MySQL 在安装时，会默认创建一个名为`root`的用户，该用户拥有超级权限，可以控制整个 MySQL 服务器。

在对 MySQL 的日常管理和操作中，为了避免有人恶意使用 root 用户控制数据库，我们通常创建一些具有适当权限的用户，尽可能地不用或少用 root 用户登录系统，以此来确保数据的安全访问。

MySQL 提供了以下 3 种方法创建用户：
* 使用`CREATE USER`语句创建用户
* 在`mysql.user`表中添加用户
* 使用`GRANT`语句创建用户

## 使用CREATE USER语句创建用户
可以使用`CREATE USER`语句来创建 MySQL 用户，并设置相应的密码。
```
CREATE USER <用户> [ IDENTIFIED BY [ PASSWORD ] 'password' ] [ ,用户 [ IDENTIFIED BY [ PASSWORD ] 'password' ]]
```
参数说明如下：
1. 用户
指定创建用户账号，格式为`user_name'@'host_name`。这里的`user_name`是用户名，`host_name`为主机名，即用户连接 MySQL 时所用主机的名字。如果在创建的过程中，只给出了用户名，而没指定主机名，那么主机名默认为“%”，表示一组主机，即对所有主机开放权限。
2. `IDENTIFIED BY`子句
用于指定用户密码。新用户可以没有初始密码，若该用户不设密码，可省略此子句。
3. `PASSWORD 'password'`
`PASSWORD`表示使用哈希值设置密码，该参数可选。如果密码是一个普通的字符串，则不需要使用`PASSWORD`关键字。`'password'`表示用户登录时使用的密码，需要用单引号括起来。

使用`CREATE USER`语句时应注意以下几点：
* `CREATE USER`语句可以不指定初始密码。但是从安全的角度来说，不推荐这种做法。
* 使用`CREATE USER`语句必须拥有`mysql`数据库的`INSERT`权限或全局`CREATE USER`权限。
* 使用`CREATE USER`语句创建一个用户后，MySQL 会在 mysql 数据库的`user`表中添加一条新记录。
* `CREATE USER`语句可以同时创建多个用户，多个用户用逗号隔开。

新创建的用户拥有的权限很少，它们只能执行不需要权限的操作。如登录 MySQL、使用 SHOW 语句查询所有存储引擎和字符集的列表等。如果两个用户的用户名相同，但主机名不同，MySQL 会将它们视为两个用户，并允许为这两个用户分配不同的权限集合。

使用`CREATE USER`创建一个用户，用户名是`test1`，密码是`test1`，主机名是`localhost`。
```sql
mysql> CREATE USER 'test1'@'localhost' IDENTIFIED BY 'test1';
```
在实际应用中，我们应避免明文指定密码，可以通过`PASSWORD`关键字使用密码的哈希值设置密码。

在 MySQL 中，可以使用`password()`函数获取密码的哈希值：
```sql
mysql> SELECT password('test1');
+-------------------------------------------+
| password('test1')                         |
+-------------------------------------------+
| *06C0BF5B64ECE2F648B5F048A71903906BA08E5C |
+-------------------------------------------+
```
`*06C0BF5B64ECE2F648B5F048A71903906BA08E5C`就是`test1`的哈希值。下面创建用户`test1`，SQL 语句和执行过程如下：
```sql
mysql> CREATE USER 'test1'@'localhost'IDENTIFIED BY PASSWORD '*06C0BF5B64ECE2F648B5F048A71903906BA08E5C';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
执行成功后就可以使用密码`test1`登录了。
## 使用 INSERT 语句新建用户
可以使用`INSERT`语句将用户的信息添加到`mysql.user`表中，但必须拥有对`mysql.user`表的`INSERT`权限。通常`INSERT`语句只添加`Host、User`和`authentication_string`这 3 个字段的值。

MySQL 5.7 的`user`表中的密码字段从`Password`变成了`authentication_string`，如果是 MySQL 5.7 之前的版本，将`authentication_string`字段替换成`Password`即可。

使用`INSERT`语句创建用户的代码如下：
```sql
INSERT INTO mysql.user(Host, User,  authentication_string, ssl_cipher, x509_issuer, x509_subject) VALUES ('hostname', 'username', PASSWORD('password'), '', '', '');
```
由于`mysql`数据库的`user`表中，`ssl_cipher、x509_issuer`和`x509_subject`这 3 个字段没有默认值，所以向`user`表插入新记录时，一定要设置这 3 个字段的值，否则`INSERT`语句将不能执行。

下面使用`INSERT`语句创建名为`test2`的用户，主机名是`localhost`，密码也是`test2`。
```sql
mysql> INSERT INTO mysql.user(Host, User, authentication_string, ssl_cipher, x509_issuer, x509_subject)
       VALUES ('localhost', 'test2', PASSWORD('test2'), '', '', '');
Query OK, 1 row affected, 1 warning (0.02 sec)
```
结果显示，新建用户成功。但是这时如果通过该账户登录 MySQL 服务器，不会登录成功，因为`test2`用户还没有生效。

可以使用`FLUSH`命令让用户生效：
```sql
FLUSH PRIVILEGES;
```
使用以上命令可以让 MySQL 刷新系统权限相关表。执行`FLUSH`命令需要`RELOAD`权限。

注意：`user`表中的`User`和`Host`字段区分大小写，创建用户时要指定正确的用户名称或主机名。
## 使用GRANT语句新建用户
虽然`CREATE USER`和`INSERT INTO`语句都可以创建普通用户，但是这两种方式不便授予用户权限。于是 MySQL 提供了`GRANT`语句。
```sql
GRANT priv_type ON database.table TO user [IDENTIFIED BY [PASSWORD] 'password']
```
其中：
* `priv_type`参数表示新用户的权限；
* `database.table`参数表示新用户的权限范围，即只能在指定的数据库和表上使用自己的权限；
* `user`参数指定新用户的账号，由用户名和主机名构成；
* `IDENTIFIED BY`关键字用来设置密码；
* `password`参数表示新用户的密码。

下面使用`GRANT`语句创建名为`test3`的用户，主机名为`localhost`，密码为`test3`。该用户对所有数据库的所有表都有`SELECT`权限。
```sql
mysql> GRANT SELECT ON*.* TO 'test3'@localhost IDENTIFIED BY 'test3';
Query OK, 0 rows affected, 1 warning (0.01 sec)
```
其中，`*.*`表示所有数据库下的所有表。结果显示创建用户成功，且`test3`用户对所有表都有查询（`SELECT`）权限。
# 修改用户
在 MySQL 中，我们可以使用`RENAME USER`语句修改一个或多个已经存在的用户账号。
```
RENAME USER <旧用户> TO <新用户>
```
其中：
* <旧用户>：系统中已经存在的 MySQL 用户账号。
* <新用户>：新的 MySQL 用户账号。

使用`RENAME USER`语句时应注意以下几点：
* `RENAME USER`语句用于对原有的 MySQL 用户进行重命名。
* 若系统中旧账户不存在或者新账户已存在，该语句执行时会出现错误。
* 使用`RENAME USER`语句，必须拥有`mysql`数据库的`UPDATE`权限或全局`CREATE USER`权限。

使用`RENAME USER`语句将用户名`test1`修改为`testUser1`，主机是`localhost`。
```sql
mysql> RENAME USER 'test1'@'localhost' TO 'testUser1'@'localhost';
Query OK, 0 rows affected (0.03 sec)
```
# 删除用户
在 MySQL 数据库中，可以使用`DROP USER`语句删除用户，也可以直接在`mysql.user`表中删除用户以及相关权限。
## 1. 使用DROP USER语句删除普通用户
```
DROP USER <用户1> [ , <用户2> ]…
```
其中，用户用来指定需要删除的用户账号。

使用`DROP USER`语句应注意以下几点：
* `DROP USER`语句可用于删除一个或多个用户，并撤销其权限。
* 使用`DROP USER`语句必须拥有`mysql`数据库的`DELETE`权限或全局`CREATE USER`权限。
* 在`DROP USER`语句的使用中，若没有明确地给出账户的主机名，则该主机名默认为“%”。

注意：用户的删除不会影响他们之前所创建的表、索引或其他数据库对象，因为 MySQL 并不会记录是谁创建了这些对象。

```sql
mysql> DROP USER 'test1'@'localhost';
```
## 2. 使用DELETE语句删除普通用户
可以使用`DELETE`语句直接删除`mysql.user`表中相应的用户信息，但必须拥有`mysql.user`表的`DELETE`权限。
```sql
DELETE FROM mysql.user WHERE Host='hostname' AND User='username';
```
`Host`和`User`这两个字段都是`mysql.user`表的主键。因此，需要两个字段的值才能确定一条记录。

下面使用 DELETE 语句删除用户'test2'@'localhost'。SQL 语句和执行过程如下所示。
```sql
DELETE FROM mysql.user WHERE Host='localhost'AND User='test2';
Query OK, 1 rows affected (0.00 sec)
```
结果显示删除成功。可以使用`SELETE`语句查询`mysql.user`表，以确定该用户是否已经成功删除。
# 查看用户权限
在 MySQL 中，可以通过查看`mysql.user`表中的数据记录来查看相应的用户权限，也可以使用`SHOW GRANTS`语句查询用户的权限。

`mysql`数据库下的`user`表中存储着用户的基本权限，可以使用`SELECT`语句来查看。
```sql
SELECT * FROM mysql.user;
```
要执行该语句，必须拥有对`user`表的查询权限。

注意：新创建的用户只有登录 MySQL 服务器的权限，没有任何其它权限，不能查询`user`表。

除了使用`SELECT`语句之外，还可以使用`SHOW GRANTS FOR`语句查看权限。
```sql
SHOW GRANTS FOR 'username'@'hostname';
```
下面创建 testuser1 用户并查询权限，SQL 语句和执行过程如下：
```
mysql> CREATE USER 'testuser1'@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GRANTS FOR 'testuser1'@'localhost';
+-----------------------------------------------+
| Grants for testuser1@localhost                |
+-----------------------------------------------+
| GRANT USAGE ON *.* TO 'testuser1'@'localhost' |
+-----------------------------------------------+
1 row in set (0.00 sec)
```
其中，`USAGE ON *.*`表示该用户对任何数据库和任何表都没有权限。

下面查询`root`用户的权限，代码如下：
```
mysql> SHOW GRANTS FOR 'root'@'localhost';
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+
2 rows in set (0.00 sec)
```
# GRANT：用户授权
授权就是为某个用户赋予某些权限。例如，可以为新建的用户赋予查询所有数据库和表的权限。MySQL 提供了`GRANT`语句来为用户设置权限。

在 MySQL 中，拥有`GRANT`权限的用户才可以执行`GRANT 语句`，其语法格式如下：
```
GRANT priv_type [(column_list)] ON database.table
TO user [IDENTIFIED BY [PASSWORD] 'password']
[, user[IDENTIFIED BY [PASSWORD] 'password']] ...
[WITH with_option [with_option]...]
```
其中：
* `priv_type`参数表示权限类型；
* `columns_list`参数表示权限作用于哪些列上，省略该参数时，表示作用于整个表；
* `database.table`用于指定权限的级别；
* `user`参数表示用户账户，由用户名和主机名构成，格式是`'username'@'hostname'`；
* `IDENTIFIED BY`参数用来为用户设置密码；
* `password`参数是用户的新密码。

`WITH`关键字后面带有一个或多个`with_option`参数。这个参数有 5 个选项：
* `GRANT OPTION`：被授权的用户可以将这些权限赋予给别的用户；
* `MAX_QUERIES_PER_HOUR count`：设置每个小时可以允许执行`count`次查询；
* `MAX_UPDATES_PER_HOUR count`：设置每个小时可以允许执行`count`次更新；
* `MAX_CONNECTIONS_PER_HOUR count`：设置每小时可以建立`count`个连接;
* `MAX_USER_CONNECTIONS count`：设置单个用户可以同时具有的`count`个连接。

MySQL 中可以授予的权限有如下几组：
* 列权限，和表中的一个具体列相关。例如，可以使用`UPDATE`语句更新表`students`中 name 列的值的权限。
* 表权限，和一个具体表中的所有数据相关。例如，可以使用`SELECT`语句查询表`students`的所有数据的权限。
* 数据库权限，和一个具体的数据库中的所有表相关。例如，可以在已有的数据库`mytest`中创建新表的权限。
* 用户权限，和 MySQL 中所有的数据库相关。例如，可以删除已有的数据库或者创建一个新的数据库的权限。

对应地，在 GRANT 语句中可用于指定权限级别的值有以下几类格式：
* `*`：表示当前数据库中的所有表。
* `*.*`：表示所有数据库中的所有表。
* `db_name.*`：表示某个数据库中的所有表，`db_name`指定数据库名。
* `db_name.tbl_name`：表示某个数据库中的某个表或视图，`db_name`指定数据库名，`tbl_name`指定表名或视图名。
* `db_name.routine_name`：表示某个数据库中的某个存储过程或函数，`routine_name`指定存储过程名或函数名。
* `TO`子句：如果权限被授予给一个不存在的用户，MySQL 会自动执行一条`CREATE USER`语句来创建这个用户，但同时必须为该用户设置密码。

## 权限类型说明
1. 授予数据库权限时，<权限类型>可以指定为以下值：

| 权限名称 | 对应user表中的字段 | 说明 |
| :--: | :--: | :--: |
| SELECT                         | Select_priv          | 表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限。 |
| INSERT                         | Insert_priv          | 表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限。 |
| DELETE                         | Delete_priv          | 表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限。 |
| UPDATE                         | Update_priv          | 表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限。 |
| REFERENCES                     | References_priv      | 表示授予用户可以创建指向特定的数据库中的表外键的权限。 |
| CREATE                         | Create_priv          | 表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限。 |
| ALTER                          | Alter_priv           | 表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限。 |
| SHOW VIEW                      | Show_view_priv       | 表示授予用户可以查看特定数据库中已有视图的视图定义的权限。 |
| CREATE ROUTINE                 | Create_routine_priv  | 表示授予用户可以为特定的数据库创建存储过程和存储函数的权限。 |
| ALTER ROUTINE                  | Alter_routine_priv   | 表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限。 |
| INDEX                          | Index_priv           | 表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限。 |
| DROP                           | Drop_priv            | 表示授予用户可以删除特定数据库中所有表和视图的权限。 |
| CREATE TEMPORARY TABLES        | Create_tmp_table_priv| 表示授予用户可以在特定数据库中创建临时表的权限。 |
| CREATE VIEW                    | Create_view_priv	    | 表示授予用户可以在特定数据库中创建新的视图的权限。 |
| EXECUTE ROUTINE                | Execute_priv	        | 表示授予用户可以调用特定数据库的存储过程和存储函数的权限。 |
| LOCK TABLES                    | Lock_tables_priv     | 表示授予用户可以锁定特定数据库的已有数据表的权限。 |
| ALL 或<br>ALL PRIVILEGES<br>或 SUPER | Super_priv           | 表示以上所有权限/超级权限 |

2. 授予表权限时，<权限类型>可以指定为以下值：

| 权限名称 | 对应user表中的字段 | 说明 |
| :--: | :--: | :--: |
| SELECT                          | Select_priv     | 授予用户可以使用 SELECT 语句进行访问特定表的权限 |
| INSERT                          | Insert_priv     | 授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限 |
| DELETE                          | Delete_priv     | 授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限 |
| DROP                            | Drop_priv       | 授予用户可以删除数据表的权限 |
| UPDATE                          | Update_priv     | 授予用户可以使用 UPDATE 语句更新特定数据表的权限 |
| ALTER                           | Alter_priv      | 授予用户可以使用 ALTER TABLE 语句修改数据表的权限 |
| REFERENCES                      | References_priv | 授予用户可以创建一个外键来参照特定数据表的权限 |
| CREATE                          | Create_priv     | 授予用户可以使用特定的名字创建一个数据表的权限 |
| INDEX                           | Index_priv      | 授予用户可以在表上定义索引的权限 |
| ALL 或<br>ALL PRIVILEGES<br>或 SUPER  | Super_priv      | 所有的权限名 |

3. 授予列权限时，<权限类型>的值只能指定为`SELECT、INSERT`和`UPDATE`，同时权限的后面需要加上列名列表`column-list`。
4. 最有效率的权限是用户权限。
授予用户权限时，<权限类型>除了可以指定为授予数据库权限时的所有值之外，还可以是下面这些值：
 * `CREATE USER`：表示授予用户可以创建和删除新用户的权限。
 * `SHOW DATABASES`：表示授予用户可以使用`SHOW DATABASES`语句查看所有已有的数据库的定义的权限。

使用`GRANT`语句创建一个新的用户`testUser`，密码为`testPwd`。用户`testUser`对所有的数据有查询、插入权限，并授予`GRANT`权限。
```sql
mysql> GRANT SELECT,INSERT ON *.*
    -> TO 'testUser'@'localhost'
    -> IDENTIFIED BY 'testPwd'
    -> WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.05 sec)
```
使用`SHOW GRANTS`语句查询用户`testUser`的权限。
```sql
mysql> SHOW GRANTS FOR 'testUser'@'localhost';
+-------------------------------------------------------------------------+
| Grants for testUser@localhost                                           |
+-------------------------------------------------------------------------+
| GRANT SELECT, INSERT ON *.* TO 'testUser'@'localhost' WITH GRANT OPTION |
+-------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
结果显示，`testUser`对所有数据库的所有表有查询、插入权限，并可以将这些权限赋予给别的用户。
# REVOKE：删除用户权限
在 MySQL 中，可以使用`REVOKE`语句删除某个用户的某些权限（此用户不会被删除），在一定程度上可以保证系统的安全性。

使用`REVOKE`语句删除权限的语法格式有两种形式：
## 第一种
删除用户某些特定的权限，语法格式如下：
```sql
REVOKE priv_type [(column_list)]...
ON database.table
FROM user [, user]...
```
`REVOKE`语句中的参数与`GRANT`语句的参数意思相同。其中：
* `priv_type`参数表示权限的类型；
* `column_list`参数表示权限作用于哪些列上，没有该参数时作用于整个表上；
* `user`参数由用户名和主机名构成，格式为`username'@'hostname'`。

## 第二种
删除特定用户的所有权限：
```sql
REVOKE ALL PRIVILEGES, GRANT OPTION FROM user [, user] ...
```
删除用户权限需要注意以下几点：
* `REVOKE`语法和`GRANT`语句的语法格式相似，但具有相反的效果。
* 要使用`REVOKE`语句，必须拥有 MySQL 数据库的全局`CREATE USER`权限或`UPDATE`权限。

使用`REVOKE`语句取消用户`testUser`的插入权限。
```sql
mysql> REVOKE INSERT ON *.* FROM 'testUser'@'localhost';
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW GRANTS FOR 'testUser'@'localhost';
+-----------------------------------------------------------------+
| Grants for testUser@localhost                                   |
+-----------------------------------------------------------------+
| GRANT SELECT ON *.* TO 'testUser'@'localhost' WITH GRANT OPTION |
+-----------------------------------------------------------------+
1 row in set (0.00 sec)
```
# 登录和退出服务器
启动 MySQL 服务后，可以使用以下命令来登录。
```
mysql -h hostname|hostlP -p port -u username -p DatabaseName -e "SQL语句"
```
参数说明：
* `-h`：指定连接 MySQL 服务器的地址。可以用两种方式表示，`hostname`为主机名，`hostIP`为主机 IP 地址。
* `-p`：指定连接 MySQL 服务器的端口号，`port`为连接的端口号。MySQL 的默认端口号是 3306，因此如果不指定该参数，默认使用 3306 连接 MySQL 服务器。
* `-u`：指定连接 MySQL 服务器的用户名，`username`为用户名。
* `-p`：提示输入密码，即提示`Enter password`。
* `DatabaseName`：指定连接到 MySQL 服务器后，登录到哪一个数据库中。如果没有指定，默认为 mysql 数据库。
* `-e`：指定需要执行的 SQL 语句，登录 MySQL 服务器后执行这个 SQL 语句，然后退出 MySQL 服务器。

下面使用 root 用户登录到 test 数据库中，命令和运行过程如下：
```
C:\Users\11645>mysql -h localhost -u root -p test
Enter password: ****
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.29-log MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
上述命令中，通过值 localhost 指定 MySQL 服务器的地址，参数 -u 指定了登录 MySQL 服务器的用户账户，参数 -p 表示会出现输入密码提示信息，最后值“test”指定了登录成功后要使用的数据库。

由结果可以看到，输入命令后，会出现“Enter password”提示信息，在这条信息之后输入密码，然后按 Enter 键。密码正确后，就成功登录到 MySQL 服务器了。

下面使用 root 用户登录到自己计算机的 mysql 数据库，同时查询 student 表的表结构，命令和运行过程如下：
```
C:\Users\11645>mysql -h localhost -u root -p test -e"DESC student"
Enter password: ****
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(4)      | NO   | PRI | NULL    |       |
| name  | varchar(20) | YES  |     | NULL    |       |
| age   | int(4)      | YES  |     | NULL    |       |
| stuno | int(11)     | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
```
结果显示，执行命令并输入正确密码后，窗口中就会显示出 student 表的基本结构。

用户也可以直接在 mysql 命令的 -p 后加上登录密码，登录密码与 -p 之间没有空格。

下面使用 root 用户登录到自己计算机的 MySQL 服务器中，密码直接加在 mysql 命令中。命令如下：
```
C:\Users\11645>mysql -h localhost -u root -proot
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.29-log MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
上述命令执行后，后面不会提示输入密码。因为 -p 后面有密码，MySQL 会直接使用这个密码。

退出 MySQL 服务器的方式很简单，只要在命令行输入 EXIT 或 QUIT 即可。“\q”是 QUIT 的缩写，也可以用来退出 MySQL 服务器。退出后就会显示 Bye。如下所示：
```
mysql> QUIT;
Bye
```
# root修改普通用户密码
在 MySQL 中，`root`用户拥有很高的权限，不仅可以修改自己的密码，还可以修改其他用户的密码。
## 使用SET语句修改普通用户的密码
在 MySQL 中，只有`root`用户可以通过更新 MySQL 数据库来更改密码。使用`root`用户登录到 MySQL 服务器后，可以使用`SET`语句来修改普通用户密码。
```sql
SET PASSWORD FOR 'username'@'hostname' = PASSWORD ('newpwd');
```
其中，`username`参数是普通用户的用户名，`hostname`参数是普通用户的主机名，`newpwd`是要更改的新密码。

> 注意：新密码必须使用`PASSWORD()`函数来加密，如果不使用`PASSWORD()`加密，也会执行成功，但是用户会无法登录。

如果是普通用户修改密码，可省略`FOR`子句来更改自己的密码。
```sql
SET PASSWORD = PASSWORD('newpwd');
```
首先创建一个没有密码的 testuser 用户：
```sql
mysql> CREATE USER 'testuser'@'localhost';
Query OK, 0 rows affected (0.14 sec)
```
`root`用户登录 MySQL 服务器后，再使用`SET`语句将`testuser`用户的密码修改为`newpwd`：
```sql
mysql> SET PASSWORD FOR 'testuser'@'localhost' = PASSWORD("newpwd");
Query OK, 0 rows affected, 1 warning (0.01 sec)
```
由运行结果可以看出，SET 语句执行成功，testuser 用户的密码被成功设置为“newpwd”。

下面验证 testuser 用户密码是否修改成功。退出 MySQL 服务器，使用 testuser 用户登录，输入密码“newpwd”，SQL 语句和运行结果如下：
```
C:\Users\leovo>mysql -utestuser -p
Enter password: ******
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 5.7.29-log MySQL Community Server (GPL)
 
Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
 
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
由运行结果可以看出，testuser 用户登录成功，修改密码成功。

使用 testuser 用户登录 MySQL 服务器，再使用 SET 语句将密码更改为“newpwd1”，SQL 语句和运行结果如下所示：
```
mysql> SET PASSWORD = PASSWORD('newpwd1');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
由运行结果可以看出，修改密码成功。
## 使用UPDATE语句修改普通用户的密码
使用 root 用户登录 MySQL 服务器后，可以使用 UPDATE 语句修改 MySQL 数据库的 user 表的 authentication_string 字段，从而修改普通用户的密码。UPDATA 语句的语法如下：
```sql
UPDATE MySQL.user SET authentication_string = PASSWORD("newpwd") WHERE User = "username" AND Host = "hostname";
```
其中，username 参数是普通用户的用户名，hostname 参数是普通用户的主机名，newpwd 是要更改的新密码。

> 注意，执行`UPDATE`语句后，需要执行`FLUSH PRIVILEGES`语句重新加载用户权限。

使用 root 用户登录 MySQL 服务器，再使用 UPDATE 语句将 testuser 用户的密码修改为“newpwd2”的 SQL 语句和运行结果如下：
```
mysql> UPDATE MySQL.user SET authentication_string = PASSWORD ("newpwd2")
    -> WHERE User = "testuser" AND Host = "localhost";
Query OK, 1 row affected, 1 warning (0.07 sec)
Rows matched: 1  Changed: 1  Warnings: 1
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.03 sec)
```
由运行结果可以看出，密码修改成功。testuser 的密码被修改成了 newpwd2。使用 FLUSH PRIVILEGES 重新加载权限后，就可以使用新的密码登录 testuser 用户了。
## 使用 GRANT 语句修改普通用户密码
除了前面介绍的方法，还可以在全局级别使用 GRANT USAGE 语句指定某个账户的密码而不影响账户当前的权限。需要注意的是，使用 GRANT 语句修改密码，必须拥有 GRANT 权限。一般情况下最好使用该方法来指定或修改密码。语法格式如下：
```
GRANT USAGE ON *.* TO 'user'@’hostname’ IDENTIFIED BY 'newpwd';
```
其中，username 参数是普通用户的用户名，hostname 参数是普通用户的主机名，newpwd 是要更改的新密码。

使用 root 用户登录 MySQL 服务器，再使用 GRANT 语句将 testuser 用户的密码修改为“newpwd3”，SQL 语句和运行结果如下：
```
mysql> GRANT USAGE ON *.* TO 'testuser'@'localhost' IDENTIFIED BY 'newpwd3';
Query OK, 0 rows affected, 1 warning (0.05 sec)
```
由运行结果可以看出，密码修改成功。
# 修改root密码
在 MySQL 中，`root`用户拥有很高的权限，因此必须保证`root`用户密码的安全。修改`root`用户密码的方式有很多种。
## 使用mysqladmin命令在命令行指定新密码
root 用户可以使用 mysqladmin 命令来修改密码，mysqladmin 的语法格式如下：
```sql
mysqladmin -u username -h hostname -p password "newpwd"
```
参数说明：
* `usermame`指需要修改密码的用户名称，在这里指定为 root 用户；
* `hostname`指需要修改密码的用户主机名，该参数可以不写，默认是 localhost；
* `password`为关键字，而不是指旧密码；
* `newpwd`为新设置的密码，必须用双引号括起来。如果使用单引号会引发错误，可能会造成修改后的密码不是你想要的。

执行完上面的语句，root 用户的密码将被修改为“newpwd”。

下面使用 mysqladmin 将 root 用户的密码修改为“rootpwd”，在 Windows 命令行窗口（cmd）中执行命令和运行结果如下：
```
C:\Users\leovo>mysqladmin -u root -p password "rootpwd"
Enter password: ****
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.
```
输入 mysqladmin 命令后，按回车键，然后输入 root 用户原来的密码。执行完毕后，密码修改成功，root 用户登录时将使用新的密码。

运行结果中，输入密码后会提示在命令行界面上使用密码可能不安全的警告信息，因为在命令行输入密码时，MySQL 服务器就会提示这些安全警告信息。

下面使用修改后的“rootpwd”密码登录 root 用户，SQL 语句和运行结果如下：
```
C:\Users\leovo>mysql -uroot -p
Enter password: *******
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 23
Server version: 5.7.29-log MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```
结果显示，root 用户登录成功，所以使用 mysqladmin 命令修改 root 用户密码成功。
## 修改MySQL数据库的user表
因为所有账户信息都保存在`user`表中，因此可以直接通过修改`user`表来改变`root`用户的密码。

`root`用户登录到 MySQL 服务器后，可以使用`UPDATE`语句修改 MySQL 数据库的`user`表的`authentication_string`字段，从而修改用户的密码。

使用 UPDATA 语句修改 root 用户密码的语法格式如下：
```
UPDATE mysql.user set authentication_string = PASSWORD ("rootpwd) WHERE User = "root" and Host="localhost";
```
新密码必须使用 PASSWORD() 函数来加密。执行UPDATE语句后，需要执行FLUSH PRIVILEGES语句重新加载用户权限。

下面使用 UPDATE 语句将 root用户的密码修改为“rootpwd2”。

使用 root 用户登录到 MySQL 服务器后，SQL 语句和运行结果如下所示：
``` sql
mysql> UPDATE mysql.user set authentication_string = password ("rootpwd2")
    -> WHERE User = "root" and Host = "localhost";
Query OK, 1 row affected, 0 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings:0
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.06 sec)
```
结果显示，密码修改成功。而且使用了FLUSH PRIVILEGES;语句加载权限。退出后就必须使用新密码来登录了。
## 使用SET语句修改root用户的密码
`SET PASSWORD`语句可以用来重新设置其他用户的登录密码或者自己使用的账户的密码。使用 SET 语句修改密码的语法结构如下：
```
SET PASSWORD = PASSWORD ("rootpwd");
```
下面使用 SET 语句将 root 用户的密码修改为“rootpwd3”。

使用 root 用户登录到 MySQL 服务器后，SQL 语句和运行结果如下所示：
```
MySQL> SET PASSWORD = password ("rootpwd3");
Query OK, 0 rows affected (0.00 sec)
```
结果显示，SET 语句执行成功，root 用户的密码被成功设置为“rootpwd3”。
# 忘记root密码后如何重置
在忘记 MySQL 密码的情况下，可以通过`--skip-grant-tables`关闭服务器的认证，然后重置`root`的密码，具体操作步骤如下。
1. 关闭正在运行的 MySQL 服务。打开 cmd 进入 MySQL 的 bin 目录。
2. 输入`mysqld --console --skip-grant-tables --shared-memory`命令。`–skip-grant-tables`会让 MySQL 服务器跳过验证步骤，允许所有用户以匿名的方式，无需做密码验证就可以直接登录 MySQL 服务器，并且拥有所有的操作权限。
3. 上一个 DOS 窗口不要关闭，打开一个新的 DOS 窗口，此时仅输入 mysql 命令，不需要用户名和密码，即可连接到 MySQL。
4. 输入命令 update mysql.user set authentication_string=password('root') where user='root' and Host ='localhost'; 设置新密码。
注意：MySQL 5.7 版本中的 user 表里已经去掉了 password 字段，改为了 authentication_string。
5. 刷新权限（必须步骤），输入flush privileges;命令。
6. 因为之前使用 --skip-grant-tables 启动，所以需要重启 MySQL 服务器去掉 --skip-grant-tables。输入无误后输入quit;命令退出 MySQL 服务。
7. 重启 MySQL 服务，使用用户名 root 和刚才设置的新密码 root 登录就可以了。

# 修改密码的3种方式
在使用数据库时，我们也许会遇到 MySQL 需要修改密码的情况，比如密码太简单需要修改等。3 种修改 MySQL 数据库密码的方法。
## 1. 使用 SET PASSWORD 命令
1. 输入命令`mysql -u root -p`指定 root 用户登录 MySQL，输入后按回车键输入密码。如果没有配置环境变量，请在 MySQL 的 bin 目录下登录操作。
2. 使用 SET PASSWORD 修改密码命令格式为 set password for username @localhost = password(newpwd);
，其中 username 为要修改密码的用户名，newpwd 为要修改的新密码。如图所示。
3. 输入quit;命令退出 MySQL 重新登录，输入新密码“root”登录就可以了；

## 2. 使用mysqladmin修改密码
使用 mysqladmin 命令修改 MySQL 的 root 用户密码格式为 mysqladmin -u用户名 -p旧密码 password 新密码。

注意：下图修改密码的命令中 -uroot 和 -proot 是整体，不要写成 -u root -p root，-u 和 root 间可以加空格，但是会有警告出现，所以就不要加空格了。

## 3. UPDATE直接编辑user表
1. 输入命令mysql -u root -p指定 root 用户登录 MySQL，输入后按回车键输入密码。如果没有配置环境变量，请在 MySQL 的 bin 目录下登录操作。
2. 输入use mysql;命令连接权限数据库。
3. 输入命令update mysql.user set authentication_string=password('新密码') where user='用户名' and Host ='localhost';设置新密码。
4. 输入 flush privileges; 命令刷新权限。
5. 输入quit;命令退出 MySQL 重新登录，此时密码已经修改为刚才输入的新密码了。


# 权限控制实现原理
MySQL 权限表在数据库启动时载入内存，用户通过身份认证后，系统会在内存中进行相应权限的存取。当 MySQL 允许一个用户执行各种操作时，它将首先核实该用户向 MySQL 服务器发送的连接请求，然后确认用户的操作请求是否被允许。

当用户进行连接时，MySQL 实现权限控制主要有以下两个阶段：
1）连接核实阶段
登录 MySQL 服务器时，客户端连接请求中会提供用户名称、主机地址和密码，MySQL 服务器会使用`user`表中的`Host、User`和`authentication_string`字段执行身份检查。

只有客户端请求的主机名和用户名在`user`表中有匹配的记录，并且密码正确时，MySQL 服务器才会通过身份认证，接受连接，否则拒绝连接。

MySQL 通过 IP 地址和用户名联合进行身份认证。例如 MySQL 安装后默认创建的用户`root@localhost`，表示用户`root`只能从本地（`localhost`）进行连接时才能通过认证。此用户从其它任何主机对数据库进行连接时都将被拒绝。也就是说，用户名相同，IP 地址不同，MySQL 则将其视为不同的用户。

服务器接受连接后进入请求核实阶段等待用户请求。如果连接核实没有通过，服务器则完全拒绝访问。
2）请求核实阶段
建立连接后，服务器进入请求核实阶段，对在此连接上的每个请求，服务器都会检查用户是否有足够的权限来执行它。这正是授权表中的权限列发挥作用的地方。

权限按照以下权限表的顺序得到数据库权限：`user→db→tables_priv→columns_priv→procs_priv`。在这几个权限表中，权限范围依次递减，全局权限覆盖局部权限。

请求核实的过程如下所示：
1. 用户向 MySQL 发出操作请求。
2. MySQL 首先检查 user 表，匹配 User、Host 字段值，查看请求的全局权限在 user 表中是否被授权。授权则允许操作执行，如果指定的权限在 user 表中没有被授权。MySQL 将检查 db 表。
3. db 表是下一安全层级，其中的权限限定于数据库层级，在该层级的 SELECT 权限允许用户查看指定数据库的所有表中的数据。
MySQL 检查 db 权限表中的权限信息，匹配 User、Host 字段值，查看请求的数据库级别的权限在 db 表中是否被授权。授权则允许操作执行，否则 MySQL 继续向下查找。
4. MySQL 检查 tables_priv 权限表中的权限信息，匹配 User、Host 字段值，查看请求的数据表级别的权限在 tables_priv 表中是否被授权。授权则允许操作执行，否则 MySQL 继续向下查找。
5. MySQL 检查 columns_priv 权限表中的权限信息，匹配  User、Host 字段值，查看请求的列级别的权限在 columns_priv 表中是否被授权。授权则允许操作执行，否则 MySQL 继续向下查找。
6. 如果所有权限表都检查完毕，还是没有找到允许的权限操作，那么 MySQL 将返回错误信息，即用户请求的操作不能执行，操作失败。

提示：上面提到 MySQL 通过向下层级的顺序检查权限表，但并不意味着所有的权限都要执行该过程。例如，一个用户登录到 MySQL 服务器之后只执行对 MySQL 的管理操作，此时只涉及管理权限，因此 MySQL 只检查 user 表。