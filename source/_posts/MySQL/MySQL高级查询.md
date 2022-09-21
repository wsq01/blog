---
title: MySQL 高级查询
date: 2020-05-10 11:42:54
tags: [MySQL]
categories: [MySQL]
---
<!-- MySQL必知必会 -->

# 使用子查询
SQL还允许创建子查询，即嵌套在其他查询中的查询。
## 利用子查询进行过滤
订单存储在两个表中。对于包含订单号、客户ID、订单日期的每个订单，`orders`表存储一行。各订单的物品存储在相关的`orderitems`表中。`orders`表不存储客户信息。它只存储客户的 ID。实际的客户信息存储在`customers`表中。

现在，假如需要列出订购物品 TNT2 的所有客户，具体的步骤：
1. 检索包含物品 TNT2 的所有订单的编号。
2. 检索具有前一步骤列出的订单编号的所有客户的 ID。
3. 检索前一步骤返回的所有客户 ID 的客户信息。

上述每个步骤都可以单独作为一个查询来执行。可以把一条`SELECT`语句返回的结果用于另一条`SELECT`语句的`WHERE`子句。

也可以使用子查询来把3个查询组合成一条语句。

第一条`SELECT`语句的含义很明确，对于`prod_id`为`TNT2`的所有订单物品，它检索其`order_num`列。输出列出两个包含此物品的订单：
```sql
SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';

/*
输出结果
order_num
20005
20007
*/
```
下一步，查询具有订单 20005 和 20007 的客户 ID。利用`IN`子句，编写如下的`SELECT`语句：
```sql
SELECT cust_id FROM orders WHERE order_num IN (20005, 20007);
/*
输出结果
cust_id
10001
10004
*/
```
现在，把第一个查询（返回订单号的那一个）变为子查询组合两个查询。
```sql
SELECT cust_id
FROM orders
WHERE order_num
IN (
  SELECT order_num FROM orderitems WHERE prod_id = 'TNT2'
);
```
现在得到了订购物品`TNT2`的所有客户的 ID。下一步是检索这些客户 ID 的客户信息。检索两列的 SQL 语句为：
```sql
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN (10001, 10004);
```
可以把其中的`WHERE`子句转换为子查询：
```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id
IN (
  SELECT cust_id
  FROM orders
  WHERE order_num
  IN (
    SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';
  );
);
```
可见，在`WHERE`子句中使用子查询能够编写出功能很强并且很灵活的 SQL 语句。对于能嵌套的子查询的数目没有限制，不过在实际使用时由于性能的限制，不能嵌套太多的子查询。
## 作为计算字段使用子查询
使用子查询的另一方法是创建计算字段。假如需要显示`customers`表中每个客户的订单总数。订单与相应的客户 ID 存储在`orders`表中。为了执行这个操作，遵循下面的步骤。
1. 从`customers`表中检索客户列表。
2. 对于检索出的每个客户，统计其在`orders`表中的订单数目。

可使用`SELECT COUNT(*)`对表中的行进行计数，并且通过提供一条`WHERE`子句来过滤某个特定的客户 ID，可仅对该客户的订单进行计数。例如，下面的代码对客户 10001 的订单进行计数：
```sql
SELECT COUNT(*) AS orders FROM orders WHERE cust_id = 10001;
```
为了对每个客户执行`COUNT(*)`计算，应该将`COUNT(*)`作为一个子查询。
```sql
SELECT
  cust_name,
  cust_state,
  (
    SELECT COUNT(*)
    FROM orders
    WHERE orders.cust_id = customer.cust_id
  ) AS orders
FROM
  customers
ORDER BY
  cust_name;
```
这条`SELECT`语句对`customers`表中每个客户返回 3 列：`cust_name、cust_state`和`orders`。`orders`是一个计算字段，它是由圆括号中的子查询建立的。该子查询对检索出的每个客户执行一
次。在此例子中，该子查询执行了5次，因为检索出了5个客户。

子查询中的WHERE子句使用了完全限定列名。这种类型的子查询称为相关子查询。任何时候只要列名可能有多义性，就必须使用这种语法（表名和列名由一个句点分隔）。
# 联结表
## 联结
SQL最强大的功能之一就是能在数据检索查询的执行中联结（`join`）表。
#### 关系表
理解关系表的最好方法是来看一个现实世界中的例子。

假如有一个包含产品目录的数据库表，其中每种类别的物品占一行。对于每种物品要存储的信息包括产品描述和价格，以及生产该产品的供应商信息。

现在，假如有由同一供应商生产的多种物品，那么在何处存储供应商信息（如，供应商名、地址、联系方法等）呢？将这些数据与产品信息分开存储的理由如下。
* 因为同一供应商生产的每个产品的供应商信息都是相同的，对每个产品重复此信息既浪费时间又浪费存储空间。
* 如果供应商信息改变（例如，供应商搬家或电话号码变动），只需改动一次即可。
* 如果有重复数据（即每种产品都存储供应商信息），很难保证每次输入该数据的方式都相同。不一致的数据在报表中很难利用。

关键是，相同数据出现多次决不是一件好事，此因素是关系数据库设计的基础。关系表的设计就是要保证把信息分解成多个表，一类数据一个表。各表通过某些常用的值（即关系设计中的关系）互相关联。

在这个例子中，可建立两个表，一个存储供应商信息，另一个存储产品信息。`vendors`表包含所有供应商信息，每个供应商占一行，每个供应商具有唯一的标识。此标识称为主键（`primary key`），可以是供应商 ID 或任何其他唯一值。

`products`表只存储产品信息，它除了存储供应商 ID（`vendors`表的主键）外不存储其他供应商信息。`vendors`表的主键又叫作`products`的外键，它将`vendors`表与`products`表关联，利用供应商 ID 能从`vendors`表中找出相应供应商的详细信息。

外键为某个表中的一列，它包含另一个表的主键值，定义了两个表之间的关系。

这样做的好处如下：
* 供应商信息不重复，从而不浪费时间和空间；
* 如果供应商信息变动，可以只更新vendors表中的单个记录，相关表中的数据不用改动；
* 由于数据无重复，显然数据是一致的，这使得处理数据更简单。

总之，关系数据可以有效地存储和方便地处理。因此，关系数据库的可伸缩性远比非关系数据库要好。
#### 为什么要使用联结
正如所述，分解数据为多个表能更有效地存储，更方便地处理，并且具有更大的可伸缩性。但这些好处是有代价的。如果数据存储在多个表中，怎样用单条`SELECT`语句检索出数据？答案是使用联结。简单地说，联结是一种机制，用来在一条`SELECT`语句中关联表，因此称之为联结。使用特殊的语法，可以联结多个表返回一组输出，联结在运行时关联表中正确的行。
## 创建联结
联结的创建非常简单，规定要联结的所有表以及它们如何关联即可。
```sql
SELECT vend_name, prod_name, prod_price
FROM vendors, products
WHERE vendors.vend_id = products.vend_id
ORDER BY vend_name, prod_name;
```
利用`WHERE`子句建立联结关系似乎有点奇怪，但实际上，有一个很充分的理由。请记住，在一条`SELECT`语句中联结几个表时，相应的关系是在运行中构造的。在数据库表的定义中不存在能指示 MySQL 如何对表进行联结的东西。你必须自己做这件事情。在联结两个表时，你实际上做的是将第一个表中的每一行与第二个表中的每一行配对。`WHERE`子句作为过滤条件，它只包含那些匹配给定条件（这里是联结条件）的行。没有`WHERE`子句，第一个表中的每个行将与第二个表中的每个行配对，而不管它们逻辑上是否可以配在一起。

应该保证所有联结都有`WHERE`子句，否则 MySQL 将返回比想要的数据多得多的数据。同理，应该保证`WHERE`子句的正确性。不正确的过滤条件将导致 MySQL 返回不正确的数据。
#### 内部联结
目前为止所用的联结称为等值联结，它基于两个表之间的相等测试。这种联结也称为内部联结。其实，对于这种联结可以使用稍微不同的语法来明确指定联结的类型。下面的`SELECT`语句返回与前面例
子完全相同的数据：
```sql
SELECT vend_name, prod_name, prod_price
FROM vendors INNER JOIN products
ON vendors.vend_id = products.vend_id;
```
此语句中的`SELECT`与前面的`SELECT`语句相同，但`FROM`子句不同。这里，两个表之间的关系是`FROM`子句的组成部分，以`INNER JOIN`指定。在使用这种语法时，联结条件用特定的`ON`子句而不是`WHERE`子句给出。传递给`ON`的实际条件与传递给`WHERE`的相同。
#### 联结多个表
SQL 对一条`SELECT`语句中可以联结的表的数目没有限制。创建联结的基本规则也相同。首先列出所有表，然后定义表之间的关系。
```sql
SELECT prod_name, vend_name, prod_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id AND orderitems.prod_id = products.prod_id AND order_num = 20005;
```
此例子显示编号为 20005 的订单中的物品。订单物品存储在`orderitems`表中。每个产品按其产品 ID 存储，它引用`products`表中的产品。这些产品通过供应商ID联结到`vendors`表中相应的供应商，供应商 ID 存储在每个产品的记录中。这里的`FROM`子句列出了 3 个表，而`WHERE`子句定义了这两个联结条件，而第三个联结条件用来过滤出订单 20005 中的物品。

```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id
IN (
  SELECT cust_id
  FROM orders
  WHERE order_num
  IN (
    SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';
  );
);
```
子查询并不总是执行复杂`SELECT`操作的最有效的方法，下面是使用联结的相同查询：
```sql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
AND orderitems.order_num = orders.order_num
AND prod_id = 'TNT2';
```
# 创建高级联结
## 使用表别名
别名除了用于列名和计算字段外，SQL 还允许给表名起别名。这样做有两个主要理由：
* 缩短 SQL 语句；
* 允许在单条`SELECT`语句中多次使用相同的表。

```sql
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
AND oi.order_num = o.order_num
AND prod_id = 'TNT2';
```
在此例子中，表别名只用于`WHERE`子句。但是，表别名不仅能用于`WHERE`子句，它还可以用于`SELECT`的列表、`ORDER BY`子句以及语句的其他部分。

应该注意，表别名只在查询执行中使用。与列别名不一样，表别名不返回到客户机。
## 使用不同类型的联结
#### 自联结
使用表别名的主要原因之一是能在单条`SELECT`语句中不止一次引用相同的表。下面举一个例子。

假如你发现某物品（其 ID 为`DTNTR`）存在问题，因此想知道生产该物品的供应商生产的其他物品是否也存在这些问题。此查询要求首先找到生产 ID 为`DTNTR`的物品的供应商，然后找出这个供应商生产的其他物品。下面是解决此问题的一种方法：
```sql
SELECT prod_id, prod_name
FROM products
WHERE vend_id = (
  SELECT vend_id
  FROM products
  WHERE prod_id = 'DTNTR'
);
```
这是第一种解决方案，它使用了子查询。内部的`SELECT`语句做了一个简单的检索，返回生产 ID 为`DTNTR`的物品供应商的`vend_id`。该 ID 用于外部查询的`WHERE`子句中，以便检索出这个供应商生产的所有物品。现在来看使用联结的相同查询：
```sql
SELECT p1.prod_id, p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id AND p2.prod_id = 'DTNTR';
```
此查询中需要的两个表实际上是相同的表，因此`products`表在`FROM`子句中出现了两次。虽然这是完全合法的，但对`products`的引用具有二义性，因为 MySQL 不知道你引用的是`products`表中的哪个实例。

为解决此问题，使用了表别名。`products`的第一次出现为别名`p1`，第二次出现为别名`p2`。现在可以将这些别名用作表名。例如，`SELECT`语句使用`p1`前缀明确地给出所需列的全名。如果不这样，MySQL 将返回错误，因为分别存在两个名为`prod_id`、`prod_name`的列。MySQL 不知道想要的是哪一个列（即使它们事实上是同一个列）。`WHERE`（通过匹配`p1`中的`vend_id`和`p2`中的`vend_id`）首先联结两个表，然后按第二个表中的`prod_id`过滤数据，返回所需的数据。

自联结通常作为外部语句用来替代从相同表中检索数据时使用的子查询语句。虽然最终的结果是相同的，但有时候处理联结远比处理子查询快得多。应该试一下两种方法，以确定哪一种的性能更好。
#### 自然联结
无论何时对表进行联结，应该至少有一个列出现在不止一个表中（被联结的列）。标准的联结返回所有数据，甚至相同的列多次出现。自然联结排除多次出现，使每个列只返回一次。

怎样完成这项工作呢？答案是，系统不完成这项工作，由你自己完成它。自然联结是这样一种联结，其中你只能选择那些唯一的列。这一般是通过对表使用通配符（`SELECT *`），对所有其他表的列使用明确的子集来完成的。下面举一个例子：
```sql
SELECT c.*, o.order_num, o.order_date, oi.prod_id, oi.quantity, oi.item_price
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
AND oi.order_num = o.order_num
AND prod_id = 'FB';
```
在这个例子中，通配符只对第一个表使用。所有其他列明确列出，所以没有重复的列被检索出来。

事实上，迄今为止我们建立的每个内部联结都是自然联结，很可能我们永远都不会用到不是自然联结的内部联结。
#### 外部联结
许多联结将一个表中的行与另一个表中的行相关联。但有时候会需要包含没有关联行的那些行。例如，可能需要使用联结来完成以下工作：
* 对每个客户下了多少订单进行计数，包括那些至今尚未下订单的客户；
* 列出所有产品以及订购数量，包括没有人订购的产品；
* 计算平均销售规模，包括那些至今尚未下订单的客户。

在上述例子中，联结包含了那些在相关表中没有关联行的行。这种类型的联结称为外部联结。

下面的`SELECT`语句给出一个简单的内部联结。它检索所有客户及其订单：
```sql
SELECT customers.cust_id, orders.order_num
FROM customers INNER JOIN orders
ON customers.cust_id = orders.cust_id;
```
外部联结语法类似。为了检索所有客户，包括那些没有订单的客户，可如下进行：
```sql
SELECT customers.cust_id, orders.order_num
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id;
```
这条`SELECT`语句使用了关键字`OUTER JOIN`来指定联结的类型（而不是在`WHERE`子句中指定）。但是，与内部联结关联两个表中的行不同的是，外部联结还包括没有关联行的行。在使用`OUTER JOIN`语法时，必须使用`RIGHT`或`LEFT`关键字指定包括其所有行的表（`RIGHT`指出的是`OUTER JOIN`右边的表，而`LEFT`指出的是`OUTER JOIN`左边的表）。上面的例子使用``LEFT OUTER JOIN`从`FROM`子句的左边表（`customers`表）中选择所有行。为了从右边的表中选择所有行，应该使用`RIGHT OUTER JOIN`，如下例所示：
```sql
SELECT customers.cust_id, orders.order_num
FROM customers RIGHT OUTER JOIN orders
ON customers.cust_id = orders.cust_id;
```
存在两种基本的外部联结形式：左外部联结和右外部联结。它们之间的唯一差别是所关联的表的顺序不同。换句话说，左外部联结可通过颠倒`FROM`或`WHERE`子句中表的顺序转换为右外部联结。因此，两种类型的外部联结可互换使用，而究竟使用哪一种纯粹是根据方便而定。
## 使用带聚集函数的联结
聚集函数用来汇总数据。虽然至今为止聚集函数的所有例子只是从单个表汇总数据，但这些函数也可以与联结一起使用。

为说明这一点，请看一个例子。如果要检索所有客户及每个客户所下的订单数，下面使用了`COUNT()`函数的代码可完成此工作：
```sql
SELECT customers.cust_name, customers.cust_id, COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```
此`SELECT`语句使用`INNER JOIN`将`customers`和`orders`表互相关联。`GROUP BY`子句按客户分组数据，因此，函数调用`COUNT(orders.order_num)`对每个客户的订单计数，将它作为`num_ord`返回。

聚集函数也可以方便地与其他联结一起使用。
```sql
SELECT customers.cust_name, customers.cust_id, COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders
ON customers.cust_id = orders.cust_id
GROUP BY customers.cust_id;
```
这个例子使用左外部联结来包含所有客户，甚至包含那些没有任何下订单的客户。结果显示也包含了客户`Mouse House`，它有 0 个订单。
## 使用联结和联结条件
* 注意所使用的联结类型。一般我们使用内部联结，但使用外部联结也是有效的。
* 保证使用正确的联结条件，否则将返回不正确的数据。
* 应该总是提供联结条件，否则会得出笛卡儿积。
* 在一个联结中可以包含多个表，甚至对于每个联结可以采用不同的联结类型。虽然这样做是合法的，一般也很有用，但应该在一起测试它们前，分别测试每个联结。这将使故障排除更为简单。

# 组合查询
多数 SQL 查询都只包含从一个或多个表中返回数据的单条`SELECT`语句。MySQL也允许执行多个查询（多条`SELECT`语句），并将结果作为单个查询结果集返回。这些组合查询通常称为并（`union`）或复合查询（`compound query`）。

有两种基本情况需要使用组合查询：
* 在单个查询中从不同的表返回类似结构的数据；
* 对单个表执行多个查询，按单个查询返回数据。


多数情况下，组合相同表的两个查询完成的工作与具有多个`WHERE`子句条件的单条查询完成的工作相同。换句话说，任何具有多个`WHERE`子句的`SELECT`语句都可以作为一个组合查询给出。

这两种技术在不同的查询中性能也不同。因此，应该试一下这两种技术，以确定对特定的查询哪一种性能更好。
## 创建组合查询
可用`UNION`操作符来组合数条 SQL 查询。利用`UNION`，可给出多条`SELECT`语句，将它们的结果组合成单个结果集。
#### 使用UNION
`UNION`的使用很简单。所需做的只是给出每条`SELECT`语句，在各条语句之间放上关键字`UNION`。

举一个例子，假如需要价格小于等于 5 的所有物品的一个列表，而且还想包括供应商1001和1002生产的所有物品（不考虑价格）。当然，可以利用`WHERE`子句来完成此工作，不过这次我们将使用`UNION`。

正如所述，创建`UNION`涉及编写多条`SELECT`语句。首先来看单条语句：
```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5;

SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002);
```
第一条`SELECT`检索价格不高于5的所有物品。第二条`SELECT`使 用IN找出供应商 1001 和 1002 生产的所有物品。

为了组合这两条语句，按如下进行：
```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002);
```
`UNION`指示 MySQL 执行两条`SELECT`语句，并把输出组合成单个查询结果集。

作为参考，这里给出使用多条`WHERE`子句而不是使用`UNION`的相同查询：
```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
OR vend_id IN (1001, 1002);
```
#### UNION规则
* `UNION`必须由两条或两条以上的`SELECT`语句组成，语句之间用关键字`UNION`分隔（因此，如果组合 4 条`SELECT`语句，将要使用 3 个`UNION`关键字）。
* `UNION`中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）。
* 列数据类型必须兼容：类型不必完全相同，但必须是 DBMS 可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）。

如果遵守了这些基本规则或限制，则可以将并用于任何数据检索任务。
#### 包含或取消重复的行
在使用`UNION`时，重复的行被自动取消。这是`UNION`的默认行为，但是如果需要，可以改变它。事实上，如果想返回所有匹配行，可使用`UNION ALL`而不是`UNION`。
#### 对组合查询结果排序
`SELECT`语句的输出用`ORDER BY`子句排序。在用`UNION`组合查询时，只能使用一条`ORDER BY`子句，它必须出现在最后一条`SELECT`语句之后。对于结果集，不存在用一种方式排序一部分，而又用另一种方式排序另一部分的情况，因此不允许使用多条`ORDER BY`子句。
```sql
SELECT vend_id, prod_id, prod_price
FROM products
WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price
FROM products
WHERE vend_id IN (1001, 1002)
ORDER BY vend_id, prod_price;
```
虽然`ORDER BY`子句似乎只是最后一条`SELECT`语句的组成部分，但实际上 MySQL 将用它来排序所有`SELECT`语句返回的所有结果。
