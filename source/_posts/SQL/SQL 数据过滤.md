---
title: SQL 数据过滤
date: 2020-02-06 16:23:03
tags: [SQL]
categories: [数据库, SQL]
---


# 比较运算符
在 SQL 中，我们可以使用`WHERE`子句对条件进行筛选，在此之前，你需要了解`WHERE`子句中的比较运算符。这些比较运算符的含义你可以参见下面这张表格：

| 含义 | 运算符 |
| :--: | :--: |
| 等于、小于、大于、不等于 | `=、<、>、!=` |
| 不等于 | `<>`或`!=` |
| 小于等于 | `<=`或`!>` |
| 大于等于 | `>=`或`!<` |
| 在指定的两个数值之间 | `BETWEEN` |
| 为空值 | `IS NULL` |

不同的 DBMS 支持的运算符可能是不同的，比如 Access 不支持（`!=`），不等于应该使用（`<>`）。在 MySQL 中，不支持（`!>`）（`!<`）等。

`WHERE`子句的基本格式是：`SELECT ……(列名) FROM ……(表名) WHERE ……(子句条件)`

比如我们想要查询所有最大生命值大于 6000 的英雄：
```sql
SQL：SELECT name, hp_max FROM heros WHERE hp_max > 6000
```
| name | hp_max |
| :--: | :--: |
| 夏侯淳 | 7350 |
| ... | ... |
| 凯 | 6700 |

想要查询所有最大生命值在 5399 到 6811 之间的英雄：
```sql
SQL：SELECT name, hp_max FROM heros WHERE hp_max BETWEEN 5399 AND 6811
```
| name | hp_max |
| :--: | :--: |
| 芈月 | 6164 |
| ... | ... |
| 百里守约 | 5611 |

需要注意的是`hp_max`可以取值到最小值和最大值，即 5399 和 6811。

我们也可以对`heros`表中的`hp_max`字段进行空值检查。
```sql
SQL：SELECT name, hp_max FROM heros WHERE hp_max IS NULL
```
运行结果为空，说明`heros`表中的`hp_max`字段没有存在空值的数据行。
# 逻辑运算符
如果我们存在多个`WHERE`条件子句，可以使用逻辑运算符：

| 含义 | 逻辑运算符 |
| :--: | :--: |
| 并且 | `AND` |
| 或者 | `OR` |
| 在指定条件范围内 | `IN` |
| 非 | `NOT` |

我们还是通过例子来看下这些逻辑运算符的使用，同样采用`heros`这张表的数据查询。

假设想要筛选最大生命值大于 6000，最大法力大于 1700 的英雄，然后按照最大生命值和最大法力值之和从高到低进行排序。
```sql
SQL：SELECT name, hp_max, mp_max FROM heros WHERE hp_max > 6000 AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC
```
| name | hp_max | mp_max |
| :--: | :--: | :--: |
| 廉颇 | 9328 | 1708 |
| ... | ... | ... |
| 孙尚香 | 6014 | 1756 |

如果`AND`和`OR`同时存在`WHERE`子句中会是怎样的呢？假设我们想要查询最大生命值加最大法力值大于 8000 的英雄，或者最大生命值大于 6000 并且最大法力值大于 1700 的英雄。
```sql
SQL：SELECT name, hp_max, mp_max FROM heros WHERE (hp_max+mp_max) > 8000 OR hp_max > 6000 AND mp_max > 1700 ORDER BY (hp_max+mp_max) DESC
```
这次的条件查询多出来了 10 个英雄，这是因为我们放宽了条件，允许最大生命值 + 最大法力值大于 8000 的英雄显示出来。另外你需要注意到，当`WHERE`子句中同时存在`OR`和`AND`的时候，`AND`执行的优先级会更高，也就是说 SQL 会优先处理`AND`操作符，然后再处理`OR`操作符。

如果我们对这条查询语句`OR`两边的条件增加一个括号，结果会是怎样的呢？
```sql
SQL：SELECT name, hp_max, mp_max FROM heros WHERE ((hp_max+mp_max) > 8000 OR hp_max > 6000) AND mp_max 
```
所以当`WHERE`子句中同时出现`AND`和`OR`操作符的时候，你需要考虑到执行的先后顺序，也就是两个操作符执行的优先级。一般来说`()`优先级最高，其次优先级是`AND`，然后是`OR`。

如果我想要查询主要定位或者次要定位是法师或是射手的英雄，同时英雄的上线时间不在`2016-01-01`到`2017-01-01`之间。
```sql
SQL：
SELECT name, role_main, role_assist, hp_max, mp_max, birthdate
FROM heros 
WHERE (role_main IN ('法师', '射手') OR role_assist IN ('法师', '射手')) 
AND DATE(birthdate) NOT BETWEEN '2016-01-01' AND '2017-01-01'
ORDER BY (hp_max + mp_max) DESC
```
你能看到`WHERE`子句分成了两个部分。第一部分是关于主要定位和次要定位的条件过滤，使用的是`role_main in ('法师', '射手') OR role_assist in ('法师', '射手')`。这里用到了`IN`逻辑运算符，同时`role_main`和`role_assist`是`OR`（或）的关系。

第二部分是关于上线时间的条件过滤。`NOT`代表否，因为我们要找到不在`2016-01-01`到`2017-01-01`之间的日期，因此用到了`NOT BETWEEN '2016-01-01' AND '2017-01-01'`。同时我们是在对日期类型数据进行检索，所以使用到了`DATE`函数，将字段`birthdate`转化为日期类型再进行比较。

| name | role_main | role_assist | hp_max | mp_max | birthdate |
| :--: | :--: | :--: | :--: | :--: | :--: |
| 张良 | 法师 | | 5799 | 1988 | 2015-10-26 |
| 貂蝉 | 法师 | 刺客 | 5611 | 1960 | 2015-12-15 |
| ... | ... | ... | ... | ... | ... |
| 芈月 | 法师 | 坦克 | 6164 | 100 | 2015-12-08 |

# 使用通配符进行过滤
刚才的条件过滤都是对已知值进行的过滤，还有一种情况是我们要检索文本中包含某个词的所有数据，这里就需要使用通配符。通配符就是我们用来匹配值的一部分的特殊字符。这里我们需要使用到`LIKE`操作符。

如果我们想要匹配任意字符串出现的任意次数，需要使用（`%`）通配符。比如我们想要查找英雄名中包含“太”字的英雄都有哪些：
```sql
SQL：SELECT name FROM heros WHERE name LIKE '% 太 %'
```

| name |
| :--: |
| 东皇太一 |
| 太乙真人 |

需要说明的是不同 DBMS 对通配符的定义不同，在 Access 中使用的是（`*`）而不是（`%`）。另外关于字符串的搜索可能是需要区分大小写的，比如`'liu%'`就不能匹配上`'LIU BEI'`。具体是否区分大小写还需要考虑不同的 DBMS 以及它们的配置。

如果我们想要匹配单个字符，就需要使用下划线 (`_`) 通配符。（`%`）和（`_`）的区别在于，（`%`）代表一个或多个字符，而（`_`）只代表一个字符。比如我们想要查找英雄名除了第一个字以外，包含“太”字的英雄有哪些。
```sql
SQL：SELECT name FROM heros WHERE name LIKE '_% 太 %'
```
因为太乙真人的太是第一个字符，而`_%太%`中的太不是在第一个字符，所以匹配不到“太乙真人”，只可以匹配上“东皇太一”。

同样需要说明的是，在 Access 中使用（`?`）来代替（`_`），而且在 DB2 中是不支持通配符（`_`）的，因此你需要在使用的时候查阅相关的 DBMS 文档。

通配符还是很有用的，尤其是在进行字符串匹配的时候。不过在实际操作过程中，还是尽量少用通配符，因为它需要消耗数据库更长的时间来进行匹配。即使你对`LIKE`检索的字段进行了索引，索引的价值也可能会失效。如果要让索引生效，那么`LIKE`后面就不能以（`%`）开头，比如使用`LIKE '%太%'或LIKE '%太'`的时候就会对全表进行扫描。如果使用`LIKE '太%'`，同时检索的字段进行了索引的时候，则不会进行全表扫描。
