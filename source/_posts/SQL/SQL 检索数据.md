---
title: SQL 检索数据
date: 2020-02-05 15:15:09
tags: [SQL]
categories: [数据库, SQL]
---


`SELECT`的作用是从一个表或多个表中检索出想要的数据行。
# SELECT 查询的基础语法
`SELECT`可以帮助我们从一个表或多个表中进行数据查询。我们知道一个数据表是由列（字段名）和行（数据行）组成的，我们要返回满足条件的数据行，就需要在`SELECT`后面加上我们想要查询的列名，可以是一列，也可以是多个列。如果你不知道所有列名都有什么，也可以检索所有列。

下面这个王者荣耀英雄数据表里一共有 69 个英雄，23 个属性值（不包括英雄名`name`）。SQL 文件见[Github 地址](https://github.com/cystanford/sql_heros_data)。

| `id` | `name` | `hp_max` | `hp_growth` | ... | `role_assist` | `birthdate` |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| 10000 | 夏侯淳 | 7350 | 288.8 | | 战士 | 2016-07-19 |
| 10001 | 钟无艳 | 7000 | 275 | | 坦克 |  |
| 10002 | 张飞 | 8341 | 329 | | 辅助 |  |
| ... | ... | ... | ... | ... | ... | ... |
| 10068 | 百里守约 | 5611 | 185 | | 战士 | 2017-08-08 |

数据表中这 24 个字段（除了`id`以外），分别代表的含义见下图。

| | | | |
| :--: | :--: | :--: | :--: |
| name <br> 英雄名称 | hp_max <br> 最大生命 | hp_growth <br> 生命成长 | hp_start <br> 初始生命 |
| mp_max <br> 最大法力 | mp_growth <br> 法力成长 | mp_start <br> 初始法力 | attack_max <br> 最高物攻 |
| attack_growth <br> 物攻成长 | attack_start <br> 初始物攻 | defense_max <br> 最大物防 | defense_growth` <br> 物防成长 |
| defense_start <br> 初始物防 | hp_5s_max <br> 最大没5秒回血 | hp_5s_growth <br> 每5秒回血成长 | hp_5s_start <br> 初始每5秒回血 |
| mp_5s_max <br> 最大每5秒回蓝 | mp_5s_growth <br> 每5s回蓝成长 | mp_5s_start <br> 初始每5秒回蓝 | attack_range <br> 攻击范围 |
| attack_speed_max <br> 最大攻速 | role_main <br> 主要定位 | role_assist <br> 次要定位 | birthdate <br> 上线时间 |

## 查询列
如果我们想要对数据表中的某一列进行检索，在`SELECT`后面加上这个列的字段名即可。比如我们想要检索数据表中都有哪些英雄。
```sql
SQL：SELECT name FROM heros
```
你可以看到这样就等于单独输出了`name`这一列。

| `name` |
| :--: |
| 夏侯淳 |
| 钟无艳 |
| ... |
| 百里守约 |

我们也可以对多个列进行检索，在列名之间用逗号 (,) 分割即可。比如我们想要检索有哪些英雄，他们的最大生命、最大法力、最大物攻和最大物防分别是多少。
```sql
SQL：SELECT name, hp_max, mp_max, attack_max, defense_max FROM heros
```
这个表中一共有 25 个字段，除了`id`和英雄名`name`以外，还存在 23 个属性值，如果我们记不住所有的字段名称，可以使用`SELECT *`帮我们检索出所有的列：
```sql
SQL：SELECT * FROM heros
```
我们在做数据探索的时候，`SELECT *`还是很有用的，这样我们就不需要写很长的`SELECT`语句了。但是在生产环境时要尽量避免使用`SELECT *`。
## 起别名
我们在使用`SELECT`查询的时候，还有一些技巧可以使用，比如你可以给列名起别名。我们在进行检索的时候，可以给英雄名、最大生命、最大法力、最大物攻和最大物防等取别名：
```sql
SQL：SELECT name AS n, hp_max AS hm, mp_max AS mm, attack_max AS am, defense_max AS dm FROM heros
```
运行结果和上面多列检索的运行结果是一样的，只是将列名改成了`n、hm、mm、am`和`dm`。当然这里的列别名只是举例，一般来说起别名的作用是对原有名称进行简化，从而让 SQL 语句看起来更精简。同样我们也可以对表名称起别名，这个在多表连接查询的时候会用到。
## 查询常数
`SELECT`查询还可以对常数进行查询。就是在`SELECT`查询结果中增加一列固定的常数列。这列的取值是我们指定的，而不是从数据表中动态取出的。你可能会问为什么我们还要对常数进行查询呢？SQL 中的`SELECT`语法的确提供了这个功能，一般来说我们只从一个表中查询数据，通常不需要增加一个固定的常数列，但如果我们想整合不同的数据源，用常数列作为这个表的标记，就需要查询常数。

比如说，我们想对`heros`数据表中的英雄名进行查询，同时增加一列字段`platform`，这个字段固定值为“王者荣耀”，可以这样写：
```sql
SQL：SELECT '王者荣耀' as platform, name FROM heros
```
| `plantform` | `name` |
| :--: | :--: |
| 王者荣耀 | 夏侯淳 |
| 王者荣耀 | 钟无艳 |
| 王者荣耀 | ... |
| 王者荣耀 | 百里守约 |

在这个 SQL 语句中，我们虚构了一个`platform`字段，并且把它设置为固定值“王者荣耀”。

需要说明的是，如果常数是个字符串，那么使用单引号（''）就非常重要了，比如‘王者荣耀’。单引号说明引号中的字符串是个常数，否则 SQL 会把王者荣耀当成列名进行查询，但实际上数据表里没有这个列名，就会引起错误。如果常数是英文字母，比如`'WZRY'`也需要加引号。如果常数是个数字，就可以直接写数字，不需要单引号，比如：
```sql
SQL：SELECT 123 as platform, name FROM heros
```
## 去除重复行
关于单个表的`SELECT`查询，还有一个非常实用的操作，就是从结果中去掉重复的行。使用的关键字是`DISTINCT`。比如我们想要看下`heros`表中关于攻击范围的取值都有哪些：
```sql
SQL：SELECT DISTINCT attack_range FROM heros
```
这样我们就能直观地看到攻击范围其实只有两个值，那就是近战和远程。

| `attack_range` |
| :--: |
| 近战 |
| 远程 |

如果我们带上英雄名称：
```sql
SQL：SELECT DISTINCT attack_range, name FROM heros
```
| `attack_range` | `name` |
| :--: | :--: |
| 近战 | 夏侯淳 |
| 近战 | 钟无艳 |
| ... | ... |
| 远程 | 百里守约 |

这里有两点需要注意：
`DISTINCT`需要放到所有列名的前面，如果写成`SELECT name, DISTINCT attack_range FROM heros`会报错。
`DISTINCT`其实是对后面所有列名的组合进行去重，你能看到最后的结果是 69 条，因为这 69 个英雄名称不同，都有攻击范围（`attack_range`）这个属性值。如果你想要看都有哪些不同的攻击范围（`attack_range`），只需要写`DISTINCT attack_range`即可，后面不需要再加其他的列名了。
# 如何排序检索数据
当我们检索数据的时候，有时候需要按照某种顺序进行结果的返回，比如我们想要查询所有的英雄，按照最大生命从高到低的顺序进行排列，就需要使用`ORDER BY`子句。使用`ORDER BY`子句有以下几个点需要掌握：
* 排序的列名：`ORDER BY`后面可以有一个或多个列名，如果是多个列名进行排序，会按照后面第一个列先进行排序，当第一列的值相同的时候，再按照第二列进行排序，以此类推。
* 排序的顺序：`ORDER BY`后面可以注明排序规则，`ASC`代表递增排序，`DESC`代表递减排序。如果没有注明排序规则，默认情况下是按照`ASC`递增排序。我们很容易理解`ORDER BY`对数值类型字段的排序规则，但如果排序字段类型为文本数据，就需要参考数据库的设置方式了，这样才能判断`A`是在`B`之前，还是在`B`之后。比如使用 MySQL 在创建字段的时候设置为`BINARY`属性，就代表区分大小写。
* 非选择列排序：`ORDER BY`可以使用非选择列进行排序，所以即使在`SELECT`后面没有这个列名，你同样可以放到`ORDER BY`后面进行排序。
* `ORDER BY`的位置：`ORDER BY`通常位于`SELECT`语句的最后一条子句，否则会报错。

假设我们想要显示英雄名称及最大生命值，按照最大生命值从高到低的方式进行排序：
```sql
SQL：SELECT name, hp_max FROM heros ORDER BY hp_max DESC 
```
| `name` | `hp_max` |
| :--: | :--: |
| 廉颇 | 9328 |
| 白起 | 8638 |
| 程咬金 | 8611 |
| ... | ... |
| 武则天 | 5037 |

如果想要显示英雄名称及最大生命值，按照第一排序最大法力从低到高，当最大法力值相等的时候则按照第二排序进行，即最大生命值从高到低的方式进行排序：
```sql
SQL：SELECT name, hp_max FROM heros ORDER BY mp_max, hp_max DESC  
```
| `name` | `hp_max` |
| :--: | :--: |
| 程咬金 | 8611 |
| 亚瑟 | 8050 |
| 曹操 | 7473 |
| ... | ... |
| 妲己 | 5824 |

# 约束返回结果的数量
另外在查询过程中，我们可以约束返回结果的数量，使用`LIMIT`关键字。比如我们想返回英雄名称及最大生命值，按照最大生命值从高到低排序，返回 5 条记录即可。
```sql
SQL：SELECT name, hp_max FROM heros ORDER BY hp_max DESC LIMIT 5
```
| `name` | `hp_max` |
| :--: | :--: |
| 廉颇 | 9328 |
| 白起 | 8638 |
| 程咬金 | 8611 |
| 刘婵 | 8581 |
| 牛魔 | 8476 |

有一点需要注意，约束返回结果的数量，在不同的 DBMS 中使用的关键字可能不同。在 MySQL、PostgreSQL、MariaDB 和 SQLite 中使用`LIMIT`关键字，而且需要放到`SELECT`语句的最后面。如果是 SQL Server 和 Access，需要使用`TOP`关键字，比如：
```sql
SQL：SELECT TOP 5 name, hp_max FROM heros ORDER BY hp_max DESC
```
如果是 DB2，使用`FETCH FIRST 5 ROWS ONLY`这样的关键字：
```sql
SQL：SELECT name, hp_max FROM heros ORDER BY hp_max DESC FETCH FIRST 5 ROWS ONLY
```
如果是 Oracle，你需要基于`ROWNUM`来统计行数：
```sql
SQL：SELECT name, hp_max FROM heros WHERE ROWNUM <=5 ORDER BY hp_max DESC
```
需要说明的是，这条语句是先取出来前 5 条数据行，然后再按照`hp_max`从高到低的顺序进行排序。但这样产生的结果和上述方法的并不一样。可以使用`SELECT name, hp_max FROM (SELECT name, hp_max FROM heros ORDER BY hp_max) WHERE ROWNUM <=5`得到与上述方法一致的结果。

约束返回结果的数量可以减少数据表的网络传输量，也可以提升查询效率。如果我们知道返回结果只有 1 条，就可以使用`LIMIT 1`，告诉`SELECT`语句只需要返回一条记录即可。这样的好处就是`SELECT`不需要扫描完整的表，只需要检索到一条符合条件的记录即可返回。
# SELECT 的执行顺序
查询是`RDBMS`中最频繁的操作。我们在理解`SELECT`语法的时候，还需要了解`SELECT`执行时的底层原理。只有这样，才能让我们对 SQL 有更深刻的认识。

其中你需要记住`SELECT`查询时的两个顺序：
## 1. 关键字的顺序是不能颠倒的：
```sql
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ...
```
## 2.SELECT 语句的执行顺序（在 MySQL 和 Oracle 中，SELECT 执行顺序基本相同）：
```sql
FROM > WHERE > GROUP BY > HAVING > SELECT 的字段 > DISTINCT > ORDER BY > LIMIT
```
比如你写了一个 SQL 语句，那么它的关键字顺序和执行顺序是下面这样的：
```sql
SELECT DISTINCT player_id, player_name, count(*) as num # 顺序 5
FROM player JOIN team ON player.team_id = team.team_id # 顺序 1
WHERE height > 1.80 # 顺序 2
GROUP BY player.team_id # 顺序 3
HAVING num > 2 # 顺序 4
ORDER BY num DESC # 顺序 6
LIMIT 2 # 顺序 7
```
在`SELECT`语句执行这些步骤的时候，每个步骤都会产生一个虚拟表，然后将这个虚拟表传入下一个步骤中作为输入。需要注意的是，这些步骤隐含在 SQL 的执行过程中，对于我们来说是不可见的。

SQL 的执行原理。

首先，你可以注意到，`SELECT`是先执行`FROM`这一步的。在这个阶段，如果是多张表联查，还会经历下面的几个步骤：
1. 首先先通过`CROSS JOIN`求笛卡尔积，相当于得到虚拟表`vt（virtual table）1-1`；
2. 通过`ON`进行筛选，在虚拟表`vt1-1`的基础上进行筛选，得到虚拟表`vt1-2`；
3. 添加外部行。如果我们使用的是左连接、右链接或者全连接，就会涉及到外部行，也就是在虚拟表`vt1-2`的基础上增加外部行，得到虚拟表`vt1-3`。

当然如果我们操作的是两张以上的表，还会重复上面的步骤，直到所有表都被处理完为止。这个过程得到是我们的原始数据。

当我们拿到了查询数据表的原始数据，也就是最终的虚拟表`vt1`，就可以在此基础上再进行`WHERE`阶段。在这个阶段中，会根据`vt1`表的结果进行筛选过滤，得到虚拟表`vt2`。

然后进入第三步和第四步，也就是`GROUP`和`HAVING`阶段。在这个阶段中，实际上是在虚拟表`vt2`的基础上进行分组和分组过滤，得到中间的虚拟表`vt3`和`vt4`。

当我们完成了条件筛选部分之后，就可以筛选表中提取的字段，也就是进入到`SELECT`和`DISTINCT`阶段。

首先在`SELECT`阶段会提取想要的字段，然后在`DISTINCT`阶段过滤掉重复的行，分别得到中间的虚拟表`vt5-1`和`vt5-2`。

当我们提取了想要的字段数据之后，就可以按照指定的字段进行排序，也就是`ORDER BY`阶段，得到虚拟表`vt6`。

最后在`vt6`的基础上，取出指定行的记录，也就是`LIMIT`阶段，得到最终的结果，对应的是虚拟表`vt7`。

当然我们在写`SELECT`语句的时候，不一定存在所有的关键字，相应的阶段就会省略。

同时因为 SQL 是一门类似英语的结构化查询语言，所以我们在写`SELECT`语句的时候，还要注意相应的关键字顺序。
