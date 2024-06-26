---
title: MyBatis 核心对象
date: 2021-04-09 16:24:15
tags: [MyBatis]
categories: [MyBatis]
---


使用 MyBatis 的主要 Java 接口就是`SqlSession`。可以通过这个接口来执行命令，获取映射器实例和管理事务。

`SqlSession`是由`SqlSessionFactory`实例创建的。`SqlSessionFactory`对象包含创建`SqlSession`实例的各种方法。而`SqlSessionFactory`本身是由`SqlSessionFactoryBuilder`创建的，它可以从 XML、注解或 Java 配置代码来创建`SqlSessionFactory`。

{% asset_img 1.png %}

当 Mybatis 与一些依赖注入框架（如 Spring ）搭配使用时，`SqlSession`将被依赖注入框架创建并注入，所以不需要使用`SqlSessionFactoryBuilder`或者`SqlSessionFactory`。
# SqlSessionFactoryBuilder
`SqlSessionFactoryBuilder`有五个`build()`方法，每一种都允许你从不同的资源中创建一个`SqlSessionFactory`实例。
```java
SqlSessionFactory build(InputStream inputStream)
SqlSessionFactory build(InputStream inputStream, String environment)
SqlSessionFactory build(InputStream inputStream, Properties properties)
SqlSessionFactory build(InputStream inputStream, String env, Properties props)
SqlSessionFactory build(Configuration config)
```
第一种方法是最常用的，它接受一个指向 XML 文件的`InputStream`实例。可选的参数是`environment`和`properties`。`environment`决定加载哪种环境，包括数据源和事务管理器。

如果你调用了带`environment`参数的`build`方法，那么 MyBatis 将使用该环境对应的配置。当然，如果你指定了一个无效的环境，会收到错误。如果你调用了不带`environment`参数的`build`方法，那么就会使用默认的环境配置。

如果你调用了接受`properties`实例的方法，那么 MyBatis 就会加载这些属性，并在配置中提供使用。绝大多数场合下，可以用`${propName}`形式引用这些配置值。

在`mybatis-config.xml`中，可以引用属性值，也可以直接指定属性值。因此，理解属性的优先级是很重要的。

如果一个属性存在于下面的多个位置，那么 MyBatis 将按照以下顺序来加载它们：
* 首先，读取在`properties`元素体中指定的属性；
* 其次，读取在`properties`元素的类路径`resource`或`url`指定的属性，且会覆盖已经指定了的重复属性；
* 最后，读取作为方法参数传递的属性，且会覆盖已经从`properties`元素体和`resource`或`url`属性中加载了的重复属性。

因此，通过方法参数传递的属性的优先级最高，`resource`或`url`指定的属性优先级中等，在`properties`元素体中指定的属性优先级最低。

```java
String resource = "org/mybatis/builder/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsReader(resource);
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
```
# SqlSessionFactory
`SqlSessionFactory`有六个方法创建`SqlSession`实例。
```java
SqlSession openSession()
SqlSession openSession(boolean autoCommit)
SqlSession openSession(Connection connection)
SqlSession openSession(TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType)
SqlSession openSession(ExecutorType execType, boolean autoCommit)
SqlSession openSession(ExecutorType execType, Connection connection)
Configuration getConfiguration();
```
默认的`openSession()`方法没有参数，它会创建具备如下特性的`SqlSession`：
* 事务作用域将会开启（也就是不自动提交）。
* 将由当前环境配置的`DataSource`实例中获取`Connection`对象。
* 事务隔离级别将会使用驱动或数据源的默认设置。
* 预处理语句不会被复用，也不会批量处理更新。

向`autoCommit`可选参数传递`true`值即可开启自动提交功能。若要使用自己的`Connection`实例，传递一个`Connection`实例给`connection`参数即可。注意，我们没有提供同时设置`Connection`和`autoCommit`的方法，这是因为 MyBatis 会依据传入的`Connection`来决定是否启用`autoCommit`。对于事务隔离级别，MyBatis 使用了一个 Java 枚举包装器来表示，称为`TransactionIsolationLevel`，事务隔离级别支持 JDBC 的五个隔离级别（`NONE、READ_UNCOMMITTED、READ_COMMITTED、REPEATABLE_READ`和`SERIALIZABLE`），并且与预期的行为一致。

`ExecutorType`这个枚举类型定义了三个值:
* `ExecutorType.SIMPLE`：该类型的执行器没有特别的行为。它为每个语句的执行创建一个新的预处理语句。
* `ExecutorType.REUSE`：该类型的执行器会复用预处理语句。
* `ExecutorType.BATCH`：该类型的执行器会批量执行所有更新语句，如果`SELECT`在多个更新中间执行，将在必要时将多条更新语句分隔开来，以方便理解。

在`SqlSessionFactory`中还有一个方法就是`getConfiguration()`。这个方法会返回一个`Configuration`实例，你可以在运行时使用它来检查 MyBatis 的配置。
# SqlSession
`SqlSession`类包含了所有执行语句、提交或回滚事务以及获取映射器实例的方法。`SqlSession`类的方法超过了 20 个，为了方便理解，我们将它们分成几种组别。
## 语句执行方法
这些方法被用来执行定义在 SQL 映射 XML 文件中的`SELECT、INSERT、UPDATE`和`DELETE`语句。每一方法都接受语句的 ID 以及参数对象，参数可以是原始类型（支持自动装箱或包装类）、`JavaBean`、`POJO`或`Map`。
```java
<T> T selectOne(String statement, Object parameter)
<E> List<E> selectList(String statement, Object parameter)
<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey)
int insert(String statement, Object parameter)
int update(String statement, Object parameter)
int delete(String statement, Object parameter)
```
`selectOne`和`selectList`的不同仅仅是`selectOne`必须返回一个对象或`null`值。如果返回值多于一个，就会抛出异常。如果你不知道返回对象会有多少，请使用`selectList`。如果需要查看某个对象是否存在，最好的办法是查询一个`count` 值（0 或 1）。`selectMap`稍微特殊一点，它会将返回对象的其中一个属性作为`key`值，将对象作为`value`值，从而将多个结果集转为`Map`类型值。由于并不是所有语句都需要参数，所以这些方法都具有一个不需要参数的重载形式。

`insert、update`以及`delete`方法返回的值表示受该语句影响的行数。
```java
<T> T selectOne(String statement)
<E> List<E> selectList(String statement)
<K,V> Map<K,V> selectMap(String statement, String mapKey)
int insert(String statement)
int update(String statement)
int delete(String statement)
```
最后，还有`select`方法的三个高级版本，它们允许你限制返回行数的范围，或是提供自定义结果处理逻辑，通常在数据集非常庞大的情形下使用。
```java
<E> List<E> selectList (String statement, Object parameter, RowBounds rowBounds)
<K,V> Map<K,V> selectMap(String statement, Object parameter, String mapKey, RowBounds rowbounds)
void select (String statement, Object parameter, ResultHandler<T> handler)
void select (String statement, Object parameter, RowBounds rowBounds, ResultHandler<T> handler)
```
`RowBounds`参数会告诉 MyBatis 略过指定数量的记录，并限制返回结果的数量。`RowBounds`类的`offset`和`limit`值只有在构造函数时才能传入，其它时候是不能修改的。
```java
int offset = 100;
int limit = 25;
RowBounds rowBounds = new RowBounds(offset, limit);
```
数据库驱动决定了略过记录时的查询效率。为了获得最佳的性能，建议将`ResultSet`类型设置为`SCROLL_SENSITIVE`或`SCROLL_INSENSITIVE`（换句话说：不要使用`FORWARD_ONLY`）。

`ResultHandler`参数允许自定义每行结果的处理过程。你可以将它添加到`List`中、创建`Map`和`Set`，甚至丢弃每个返回值，只保留计算后的统计结果。你可以使用`ResultHandler`做很多事，这其实就是 MyBatis 构建 结果列表的内部实现办法。

`ResultHandler`会在存储过程的`REFCURSOR`输出参数中传递使用的`CALLABLE`语句。
```java
package org.apache.ibatis.session;
public interface ResultHandler<T> {
  void handleResult(ResultContext<? extends T> context);
}
```
`ResultContext`参数允许你访问结果对象和当前已被创建的对象数目，另外还提供了一个返回值为`Boolean`的`stop`方法，你可以使用此`stop`方法来停止 MyBatis 加载更多的结果。

使用`ResultHandler`的时候需要注意以下两个限制：
* 使用带`ResultHandler`参数的方法时，收到的数据不会被缓存。
* 当使用高级的结果映射集（`resultMap`）时，MyBatis 很可能需要数行结果来构造一个对象。如果你使用了`ResultHandler`，你可能会接收到关联（`association`）或者集合（`collection`）中尚未被完整填充的对象。

## 立即批量更新方法
当你将`ExecutorType`设置为`ExecutorType.BATCH`时，可以使用这个方法清除（执行）缓存在 JDBC 驱动类中的批量更新语句。
```java
List<BatchResult> flushStatements()
```
## 事务控制方法
有四个方法用来控制事务作用域。当然，如果你已经设置了自动提交或你使用了外部事务管理器，这些方法就没什么作用了。然而，如果你正在使用由`Connection`实例控制的 JDBC 事务管理器，那么这四个方法就会派上用场：
```java
void commit()
void commit(boolean force)
void rollback()
void rollback(boolean force)
```
默认情况下 MyBatis 不会自动提交事务，除非它侦测到调用了插入、更新或删除方法改变了数据库。如果你没有使用这些方法提交修改，那么你可以在`commit`和`rollback`方法参数中传入`true`值，来保证事务被正常提交（注意，在自动提交模式或者使用了外部事务管理器的情况下，设置`force`值对`session`无效）。大部分情况下你无需调用`rollback()`，因为 MyBatis 会在你没有调用`commit`时替你完成回滚操作。不过，当你要在一个可能多次提交或回滚的`session`中详细控制事务，回滚操作就派上用场了。
## 本地缓存
Mybatis 使用到了两种缓存：本地缓存（`local cache`）和二级缓存（`second level cache`）。

每当一个新`session`被创建，MyBatis 就会创建一个与之相关联的本地缓存。任何在`session`执行过的查询结果都会被保存在本地缓存中，所以，当再次执行参数相同的相同查询时，就不需要实际查询数据库了。本地缓存将会在做出修改、事务提交或回滚，以及关闭`session`时清空。

默认情况下，本地缓存数据的生命周期等同于整个`session`的周期。由于缓存会被用来解决循环引用问题和加快重复嵌套查询的速度，所以无法将其完全禁用。但是你可以通过设置`localCacheScope=STATEMENT`来只在语句执行时使用缓存。

注意，如果`localCacheScope`被设置为`SESSION`，对于某个对象，MyBatis 将返回在本地缓存中唯一对象的引用。对返回的对象（例如`list`）做出的任何修改将会影响本地缓存的内容，进而将会影响到在本次`session`中从缓存返回的值。因此，不要对 MyBatis 所返回的对象作出更改，以防后患。

你可以随时调用以下方法来清空本地缓存：
```java
void clearCache()
```
## 确保 SqlSession 被关闭
```java
void close()
```
对于你打开的任何`session`，你都要保证它们被妥善关闭，这很重要。

和`SqlSessionFactory`一样，你可以调用当前使用的`SqlSession`的`getConfiguration`方法来获得`Configuration`实例。
```java
Configuration getConfiguration()
```
## 使用映射器
```java
<T> T getMapper(Class<T> type)
```
上述的各个`insert、update、delete`和`select`方法都很强大，但也有些繁琐，它们并不符合类型安全，对你的 IDE 和单元测试也不是那么友好。因此，使用映射器类来执行映射语句是更常见的做法。

一个映射器类就是一个仅需声明与`SqlSession`方法相匹配方法的接口。下面的示例展示了一些方法签名以及它们是如何映射到`SqlSession`上的。
```java
public interface AuthorMapper {
  // (Author) selectOne("selectAuthor",5);
  Author selectAuthor(int id);
  // (List<Author>) selectList(“selectAuthors”)
  List<Author> selectAuthors();
  // (Map<Integer,Author>) selectMap("selectAuthors", "id")
  @MapKey("id")
  Map<Integer, Author> selectAuthors();
  // insert("insertAuthor", author)
  int insertAuthor(Author author);
  // updateAuthor("updateAuthor", author)
  int updateAuthor(Author author);
  // delete("deleteAuthor",5)
  int deleteAuthor(int id);
}
```
总之，每个映射器方法签名应该匹配相关联的`SqlSession`方法，字符串参数`ID`无需匹配。而是由方法名匹配映射语句的`ID`。

此外，返回类型必须匹配期望的结果类型，返回单个值时，返回类型应该是返回值的类，返回多个值时，则为数组或集合类，另外也可以是游标（`Cursor`）。所有常用的类型都是支持的，包括：原始类型、`Map、POJO`和`JavaBean`。

映射器接口不需要去实现任何接口或继承自任何类。只要方法签名可以被用来唯一识别对应的映射语句就可以了。

映射器接口可以继承自其他接口。在使用 XML 来绑定映射器接口时，保证语句处于合适的命名空间中即可。唯一的限制是，不能在两个具有继承关系的接口中拥有相同的方法签名（这是潜在的危险做法，不可取）。

你可以传递多个参数给一个映射器方法。在多个参数的情况下，默认它们将会以`param`加上它们在参数列表中的位置来命名，比如：`#{param1}、#{param2}`等。如果你想（在有多个参数时）自定义参数的名称，那么你可以在参数上使用`@Param("paramName")`注解。

你也可以给方法传递一个`RowBounds`实例来限制查询结果。
