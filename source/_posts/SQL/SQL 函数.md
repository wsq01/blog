---
title: SQL 函数
date: 2020-02-07 14:11:02
tags: [SQL]
categories: [数据库, SQL]
---

在 SQL 中我们也可以使用函数对检索出来的数据进行函数操作，比如求某列数据的平均值，或者求字符串的长度等。从函数定义的角度出发，我们可以将函数分成内置函数和自定义函数。在 SQL 语言中，同样也包括了内置函数和自定义函数。内置函数是系统内置的通用函数，而自定义函数是我们根据自己的需要编写的。
# 什么是 SQL 函数
SQL 中的函数一般是在数据上执行的，可以很方便地转换和处理数据。一般来说，当我们从数据表中检索出数据之后，就可以进一步对这些数据进行操作，得到更有意义的结果，比如返回指定条件的函数，或者求某个字段的平均值等。
# 常用的 SQL 函数有哪些
SQL 提供了一些常用的内置函数，当然你也可以自己定义 SQL 函数。SQL 的内置函数对于不同的数据库软件来说具有一定的通用性，我们可以把内置函数分成四类：
* 算术函数
* 字符串函数
* 日期函数
* 转换函数

这 4 类函数分别代表了算术处理、字符串处理、日期处理、数据类型转换，它们是 SQL 函数常用的划分形式。

函数是对提取出来的数据进行操作，那么数据表中字段类型的定义有哪几种呢？

我们经常会保存一些数值，不论是整数类型，还是浮点类型，实际上对应的就是数值类型。同样我们也会保存一些文本内容，可能是人名，也可能是某个说明，对应的就是字符串类型。此外我们还需要保存时间，也就是日期类型。那么针对数值、字符串和日期类型的数据，我们可以对它们分别进行算术函数、字符串函数以及日期函数的操作。如果想要完成不同类型数据之间的转换，就可以使用转换函数。
## 算术函数
算术函数，顾名思义就是对数值类型的字段进行算术运算。常用的算术函数及含义如下表所示：

| 函数名 | 定义 |
| :--: | :--: |
| `ABS()` | 取绝对值 |
| `MOD()` | 取余 |
| `ROUND()` | 四舍五入为指定的小数位数，需要有两个参数，分别为字段名称、小数位数 |

例：

`SELECT ABS(-2)`，运行结果为 2。

`SELECT MOD(101,3)`，运行结果 2。

`SELECT ROUND(37.25,1)`，运行结果 37.3。
## 字符串函数
常用的字符串函数操作包括了字符串拼接，大小写转换，求长度以及字符串替换和截取等。具体的函数名称及含义如下表所示：

| 函数名 | 定义 |
| :--: | :--: |
| `CONCAT()` | 将多个字符串拼接起来 |
| `LENGTH()` | 计算字段的长度，一个汉字算3个字符，一个数字或字母算一个字符 |
| `CHAR_LENGTH()` | 计算字段的长度，汉字、数字、字母都算一个字符 |
| `LOWER()` | 将字符串中的字符转化为小写 |
| `UPPER()` | 将字符串中的字符转化为大写 |
| `REPLACE()` | 替换函数，有3个参数，分别为：要替换的表达式或字段名、想要查找的被替换字符串、替换成哪个字符串 |
| `SUBSTRING()` | 截取字符串，有3个参数，分别为：待截取的表达式或字段名、开始截取的位置、想要截取的字符串长度 |

例：

`SELECT CONCAT('abc', 123)`，运行结果为`abc123`。

`SELECT LENGTH('你好')`，运行结果为 6。

`SELECT CHAR_LENGTH('你好')`，运行结果为 2。

`SELECT LOWER('ABC')`，运行结果为`abc`。

`SELECT UPPER('abc')`，运行结果`ABC`。

`SELECT REPLACE('fabcd', 'abc', 123)`，运行结果为 f123d。

`SELECT SUBSTRING('fabcd', 1, 3)`，运行结果为`fab`。
## 日期函数
日期函数是对数据表中的日期进行处理，常用的函数包括：

| 函数名 | 定义 |
| :--: | :--: |
| `CURRENT_DATE()` | 系统当前日期 |
| `CURRENT_TIME()` | 系统当前时间，没有具体的日期 |
| `CURRENT_TIMESTAMP()` | 系统当前时间，包括具体的日期+时间 |
| `EXTRACT()` | 抽取具体的年、月、日 |
| `DATE()` | 返回时间的日期部分 |
| `YEAR()` | 返回时间的年份部分 |
| `MONTH()` | 返回时间的月份部分 |
| `DAY()` | 返回时间的天数部分 |
| `HOUR()` | 返回时间的小时部分 |
| `MINUTE()` | 返回时间的分钟部分 |
| `SECOND()` | 返回时间的秒部分 |

例：

`SELECT CURRENT_DATE()`，运行结果为`2019-04-03`。

`SELECT CURRENT_TIME()`，运行结果为`21:26:34`。

`SELECT CURRENT_TIMESTAMP()`，运行结果为`2019-04-03 21:26:34`。

`SELECT EXTRACT(YEAR FROM '2019-04-03')`，运行结果为 2019。

`SELECT DATE('2019-04-01 12:00:05')`，运行结果为`2019-04-01`。

这里需要注意的是，`DATE`日期格式必须是`yyyy-mm-dd`的形式。如果要进行日期比较，就要使用`DATE`函数，不要直接使用日期与字符串进行比较。
## 转换函数
转换函数可以转换数据之间的类型，常用的函数如下表所示：

| 函数名 | 定义 |
| :--: | :--: |
| `CAST()` | 数据类型转换，参数是一个表达式，表达式通过AS关键词分割了2个参数，分别是原始数据和目标数据类型 |
| `COALESCE()` | 返回第一个非空数值 |

例：

`SELECT CAST(123.123 AS INT)`，运行结果会报错。

`SELECT CAST(123.123 AS DECIMAL(8,2))`，运行结果为 123.12。

`SELECT COALESCE(null,1,2)`，运行结果为 1。

`CAST`函数在转换数据类型的时候，不会四舍五入，如果原数值有小数，那么转换为整数类型的时候就会报错。不过你可以指定转化的小数类型，在 MySQL 和 SQL Server 中，你可以用`DECIMAL(a,b)`来指定，其中`a`代表整数部分和小数部分加起来最大的位数，`b`代表小数位数，比如`DECIMAL(8,2)`代表的是精度为 8 位（整数加小数位数最多为 8 位），小数位数为 2 位的数据类型。所以`SELECT CAST(123.123 AS DECIMAL(8,2))`的转换结果为 123.12。

# 用 SQL 函数对王者荣耀英雄数据做处理
王者荣耀英雄数据库，一共有 69 个英雄，23 个属性值。SQL 文件见[Github 地址](https://github.com/cystanford/sql_heros_data)。

我们现在把这个文件导入到 MySQL 中，你可以使用 Navicat 可视化数据库管理工具将`.sql`文件导入到数据库中。数据表为`heros`，然后使用 SQL 函数，对这个英雄数据表进行处理。

首先显示英雄以及他的物攻成长，对应字段为`attack_growth`。我们让这个字段精确到小数点后一位，需要使用的是算术函数里的`ROUND`函数。
```sql
SQL：SELECT name, ROUND(attack_growth,1) FROM heros
```
代码中，`ROUND(attack_growth,1)`中的`attack_growth`代表想要处理的数据，“1”代表四舍五入的位数，也就是我们这里需要精确到的位数。

运行结果为：

| `name` | `ROUND(attack_growth, 1)` |
| :--: | :--: |
| 夏侯惇 | 12.0 |
| 钟无艳 | 11.0 |
| 张飞 | 11.0 |
| ... | ... |
| 百里守约 | 16.0 |

假设我们想显示英雄最大生命值的最大值，就需要用到`MAX`函数。在数据中，“最大生命值”对应的列数为`hp_max`，在代码中的格式为`MAX(hp_max)`。
```sql
SQL：SELECT MAX(hp_max) FROM heros
```
运行结果为 9328。

假如我们想要知道最大生命值最大的是哪个英雄，以及对应的数值，就需要分成两个步骤来处理：首先找到英雄的最大生命值的最大值，即`SELECT MAX(hp_max) FROM heros`，然后再筛选最大生命值等于这个最大值的英雄，如下所示。
```sql
SQL：SELECT name, hp_max FROM heros WHERE hp_max = (SELECT MAX(hp_max) FROM heros)
```
运行结果：

| `name` | `hp_max` |
| :--: | :--: |
| 廉颇 | 9328 |

假如我们想显示英雄的名字，以及他们的名字字数，需要用到`CHAR_LENGTH`函数。
```sql
SQL：SELECT CHAR_LENGTH(name), name FROM heros
```
运行结果为：

| `CHAR_LENGTH(name)` | `name` |
| :--: | :--: |
| 3 | 夏侯惇 |
| 3 | 钟无艳 |
| ... | ... |
| 4 | 百里守约 |

假如想要提取英雄上线日期（对应字段`birthdate`）的年份，只显示有上线日期的英雄即可（有些英雄没有上线日期的数据，不需要显示），这里我们需要使用`EXTRACT`函数，提取某一个时间元素。所以我们需要筛选上线日期不为空的英雄，即`WHERE birthdate is not null`，然后再显示他们的名字和上线日期的年份，即：
```sql
SQL： SELECT name, EXTRACT(YEAR FROM birthdate) AS birthdate FROM heros WHERE birthdate is NOT NULL
```
或者使用如下形式：
```sql
SQL: SELECT name, YEAR(birthdate) AS birthdate FROM heros WHERE birthdate is NOT NULL
```
运行结果为：

| `name` | `birthdate` |
| :--: | :--: |
| 夏侯惇 | 2016 |
| 牛魔 | 2015 |
| ... | ... |
| 百里守约 | 2017 |

假设我们需要找出在 2016 年 10 月 1 日之后上线的所有英雄。这里我们可以采用`DATE`函数来判断`birthdate`的日期是否大于`2016-10-01`，即`WHERE DATE(birthdate)>'2016-10-01'`，然后再显示符合要求的全部字段信息，即：
```sql
SQL： SELECT * FROM heros WHERE DATE(birthdate)>'2016-10-01'
```
需要注意的是下面这种写法是不安全的：
```sql
SELECT * FROM heros WHERE birthdate>'2016-10-01'
```
因为很多时候你无法确认`birthdate`的数据类型是字符串，还是`datetime`类型，如果你想对日期部分进行比较，那么使用`DATE(birthdate)`来进行比较是更安全的。

假设我们需要知道在 2016 年 10 月 1 日之后上线英雄的平均最大生命值、平均最大法力和最高物攻最大值。同样我们需要先筛选日期条件，即`WHERE DATE(birthdate)>'2016-10-01'`，然后再选择`AVG(hp_max), AVG(mp_max), MAX(attack_max)`字段进行显示。
```sql
SQL： SELECT AVG(hp_max), AVG(mp_max), MAX(attack_max) FROM heros WHERE DATE(birthdate)>'2016-10-01'
```
运行结果为：

| `AVG(hp_max)` | `AVG(mp_max)` | `MAX(attack_max)` |
| :--: | :--: | :--: |
| 6611.5000 | 1821.5000 | 410 |

# 为什么使用 SQL 函数会带来问题
尽管 SQL 函数使用起来会很方便，但我们使用的时候还是要谨慎，因为你使用的函数很可能在运行环境中无法工作，这是为什么呢？

语言是有不同版本的，比如 Python 会有 2.7 版本和 3.x 版本，不过它们之间的函数差异不大，也就在 10% 左右。但我们在使用 SQL 语言的时候，不是直接和这门语言打交道，而是通过它使用不同的数据库软件，即 DBMS。DBMS 之间的差异性很大，远大于同一个语言不同版本之间的差异。实际上，只有很少的函数是被 DBMS 同时支持的。比如，大多数 DBMS 使用（||）或者（+）来做拼接符，而在 MySQL 中的字符串拼接函数为`Concat()`。大部分 DBMS 会有自己特定的函数，这就意味着采用 SQL 函数的代码可移植性是很差的，因此在使用函数的时候需要特别注意。


在 SQL 中，使用函数的时候需要格外留意。不过如果工程量不大，使用的是同一个 DBMS 的话，还是可以使用函数简化操作的，这样也能提高代码效率。只是在系统集成，或者在多个 DBMS 同时存在的情况下，使用函数的时候就需要慎重一些。

比如`CONCAT()`是字符串拼接函数，在 MySQL 和 Oracle 中都有这个函数，但是在这两个 DBMS 中作用却不一样，`CONCAT`函数在 MySQL 中可以连接多个字符串，而在 Oracle 中`CONCAT`函数只能连接两个字符串，如果要连接多个字符串就需要用（||）连字符来解决。
# 关于大小写的规范
在 SQL 中，关键字和函数名是不用区分字母大小写的，比如`SELECT、WHERE、ORDER、GROUP BY`等关键字，以及`ABS、MOD、ROUND、MAX `等函数名。

不过在 SQL 中，你还是要确定大小写的规范，因为在 Linux 和 Windows 环境下，你可能会遇到不同的大小写问题。

比如 MySQL 在 Linux 的环境下，数据库名、表名、变量名是严格区分大小写的，而字段名是忽略大小写的。

而 MySQL 在 Windows 的环境下全部不区分大小写。

这就意味着如果你的变量名命名规范没有统一，就可能产生错误。这里有一个有关命名规范的建议：
* 关键字和函数名称全部大写；
* 数据库名、表名、字段名称全部小写；
* SQL 语句必须以分号结尾。

虽然关键字和函数名称在 SQL 中不区分大小写，也就是如果小写的话同样可以执行，但是数据库名、表名和字段名在 Linux MySQL 环境下是区分大小写的，因此建议你统一这些字段的命名规则，比如全部采用小写的方式。同时将关键词和函数名称全部大写，以便于区分数据库名、表名、字段名。
# 聚集函数
SQL 函数还有一种，叫做聚集函数，它是对一组数据进行汇总的函数，输入的是一组数据的集合，输出的是单个值。通常我们可以利用聚集函数汇总表的数据，如果稍微复杂一些，我们还需要先对数据做筛选，然后再进行聚集，比如先按照某个条件进行分组，对分组条件进行筛选，然后得到筛选后的分组的汇总信息。
## 聚集函数都有哪些
SQL 中的聚集函数一共包括 5 个，可以帮我们求某列的最大值、最小值和平均值等，它们分别是：

| 函数 | 说明 |
| :--: | :--: |
| `COUNT()` | 总行数 |
| `MAX()` | 最大值 |
| `MIN()` | 最小值 |
| `SUM()` | 求和 |
| `AVG()` | 平均值 |

说明：
* `COUNT`、`SUM`、`AVG`函数中可以使用`DISTINCT`关键字去除指定列中的重复项。使用`DISTINCT`关键字后只是对不同行的值进行统计。
* `MIN`、`MAX`函数中的列或表达式可以是数字型、字符型或日期类型的值。如果是字符型的，则按照首字母从 A 到 Z 的顺序排序，如果首字母相同，则比较字符串中第二个字母的大小，以此类推。汉字则是按照其汉语拼音的全拼来排序。
* `SUM`和`AVG`函数中的表达式只能是数字类型的值。
* 除了`COUNT(*)`外，其他几个函数在计算时都忽略表达式中的空行（`NULL`行）。
* `COUNT`函数是用来计算数据表中的总行数，`SUM`函数是用来计算数据表中某一列的属性值的总和。

我们继续使用`heros`数据表，对王者荣耀的英雄数据进行聚合。

如果我们想要查询最大生命值大于 6000 的英雄数量。
```sql
SQL：SELECT COUNT(*) FROM heros WHERE hp_max > 6000
```
运行结果为 41。

如果想要查询最大生命值大于 6000，且有次要定位的英雄数量，需要使用`COUNT`函数。
```sql
SQL：SELECT COUNT(role_assist) FROM heros WHERE hp_max > 6000
```
运行结果是 23。

需要说明的是，有些英雄没有次要定位，即`role_assist`为`NULL`，这时`COUNT(role_assist)`会忽略值为`NULL`的数据行，而`COUNT(*)`只是统计数据行数，不管某个字段是否为`NULL`。

如果我们想要查询射手（主要定位或者次要定位是射手）的最大生命值的最大值是多少，需要使用 MAX 函数。
```sql
SQL：SELECT MAX(hp_max) FROM heros WHERE role_main = '射手' or role_assist = '射手'
```
运行结果为 6014。

上面的例子里，都是在一条`SELECT`语句中使用了一次聚集函数，实际上我们也可以在一条`SELECT`语句中进行多项聚集函数的查询，比如我们想知道射手（主要定位或者次要定位是射手）的英雄数、平均最大生命值、法力最大值的最大值、攻击最大值的最小值，以及这些英雄总的防御最大值等汇总数据。

如果想要知道英雄的数量，我们使用的是`COUNT(*)`函数，求平均值、最大值、最小值，以及总的防御最大值，我们分别使用的是`AVG、MAX、MIN`和`SUM`函数。另外我们还需要对英雄的主要定位和次要定位进行筛选，使用的是`WHERE role_main = '射手' or role_assist = '射手'`。
```sql
SQL: SELECT COUNT(*), AVG(hp_max), MAX(mp_max), MIN(attack_max), SUM(defense_max) FROM heros WHERE role_main = '射手' or role_assist = '射手'
```
运行结果：

| `COUNT(*)` | `AVG(hp_max)` | `MAX(mp_max)` | `MIN(attack_max)` | `SUM(defense_max)` |
| :--: | :--: | :--: | :--: | :--: |
| 10 | 5798.5 | 1784 | 362 | 3333|

需要说明的是`AVG、MAX、MIN`等聚集函数会自动忽略值为`NULL`的数据行，`MAX`和`MIN`函数也可以用于字符串类型数据的统计，如果是英文字母，则按照`A—Z`的顺序排列，越往后，数值越大。如果是汉字则按照全拼拼音进行排列。比如：
```sql
SQL：SELECT MIN(CONVERT(name USING gbk)), MAX(CONVERT(name USING gbk)) FROM heros
```