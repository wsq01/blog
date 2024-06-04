---
title: MyBatis详解—总体框架设计
date: 2024-05-06 20:08:14
tags: [MyBatis]
categories: MyBatis
---


# MyBatis架构概览
MyBatis 框架整体设计如下:

{% asset_img mybatis-y-arch-1.png %}

# 接口层-和数据库交互的方式
MyBatis 和数据库的交互有两种方式：
* 使用传统的 MyBatis 提供的API；
* 使用`Mapper`接口；

## 使用传统的MyBatis提供的API
这是传统的传递`Statement Id`和查询参数给`SqlSession`对象，使用`SqlSession`对象完成和数据库的交互；MyBatis 提供了非常方便和简单的API，供用户实现对数据库的增删改查数据操作，以及对数据库连接信息和 MyBatis 自身配置信息的维护操作。

{% asset_img mybatis-y-arch-2.png %}

上述使用 MyBatis 的方法，是创建一个和数据库打交道的`SqlSession`对象，然后根据`Statement Id`和参数来操作数据库，这种方式固然很简单和实用，但是它不符合面向对象语言的概念和面向接口编程的编程习惯。由于面向接口的编程是面向对象的大趋势，MyBatis 为了适应这一趋势，增加了第二种使用 MyBatis 支持接口（`Interface`）调用方式。
## 使用Mapper接口
MyBatis 将配置文件中的每一个`<mapper>`节点抽象为一个`Mapper`接口，而这个接口中声明的方法和跟`<mapper>`节点中的`<select|update|delete|insert>`节点项对应，即`<select|update|delete|insert>`节点的`id`值为`Mapper`接口中的方法名称，`parameterType`值表示`Mapper`对应方法的入参类型，而`resultMap`值则对应了`Mapper`接口表示的返回值类型或者返回结果集的元素类型。

{% asset_img mybatis-y-arch-3.png %}

根据 MyBatis 的配置规范配置好后，通过`SqlSession.getMapper(XXXMapper.class)`方法，MyBatis 会根据相应的接口声明的方法信息，通过动态代理机制生成一个`Mapper`实例，我们使用`Mapper`接口的某一个方法时，MyBatis 会根据这个方法的方法名和参数类型，确定`Statement Id`，底层还是通过`SqlSession.select("statementId",parameterObject);`或者`SqlSession.update("statementId",parameterObject);`等等来实现对数据库的操作，MyBatis 引用`Mapper`接口这种调用方式，纯粹是为了满足面向接口编程的需要。（其实还有一个原因是在于，面向接口的编程，使得用户在接口上可以使用注解来配置 SQL 语句，这样就可以脱离 XML 配置文件，实现“0配置”）。
# 数据处理层
数据处理层可以说是 MyBatis 的核心，从大的方面上讲，它要完成两个功能：
* 通过传入参数构建动态 SQL 语句；
* SQL 语句的执行以及封装查询结果集成`List<E>`

## 参数映射和动态SQL语句生成
动态语句生成可以说是MyBatis框架非常优雅的一个设计，MyBatis 通过传入的参数值，使用 Ognl 来动态地构造 SQL 语句，使得 MyBatis 有很强的灵活性和扩展性。

参数映射指的是对于 java 数据类型和 jdbc 数据类型之间的转换：这里有包括两个过程：查询阶段，我们要将 java 类型的数据，转换成 jdbc 类型的数据，通过`preparedStatement.setXXX()`来设值；另一个就是对`resultset`查询结果集的`jdbcType`数据转换成 java 数据类型。
## SQL语句的执行以及封装查询结果集成List
动态SQL语句生成之后，MyBatis 将执行SQL语句，并将可能返回的结果集转换成List<E> 列表。MyBatis 在对结果集的处理中，支持结果集关系一对多和多对一的转换，并且有两种支持方式，一种为嵌套查询语句的查询，还有一种是嵌套结果集的查询。
# 框架支撑层
* 事务管理机制
事务管理机制对于 ORM 框架而言是不可缺少的一部分，事务管理机制的质量也是考量一个 ORM 框架是否优秀的一个标准。
* 连接池管理机制
由于创建一个数据库连接所占用的资源比较大，对于数据吞吐量大和访问量非常大的应用而言，连接池的设计就显得非常重要。
* 缓存机制
为了提高数据利用率和减小服务器和数据库的压力，MyBatis 会对于一些查询提供会话级别的数据缓存，会将对某一次查询，放置到`SqlSession`中，在允许的时间间隔内，对于完全相同的查询，MyBatis 会直接将缓存结果返回给用户，而不用再到数据库中查找。
* SQL语句的配置方式
传统的 MyBatis 配置 SQL 语句方式就是使用 XML 文件进行配置的，但是这种方式不能很好地支持面向接口编程的理念，为了支持面向接口的编程，MyBatis 引入了`Mapper`接口的概念，面向接口的引入，对使用注解来配置 SQL 语句成为可能，用户只需要在接口上添加必要的注解即可，不用再去配置 XML 文件了，但是，目前的 MyBatis 只是对注解配置 SQL 语句提供了有限的支持，某些高级功能还是要依赖 XML 配置文件配置 SQL 语句。

# 引导层
引导层是配置和启动 MyBatis 配置信息的方式。MyBatis 提供两种方式来引导 MyBatis：基于 XML 配置文件的方式和基于 Java API 的方式。
# 主要构件及其相互关系
从 MyBatis 代码实现的角度来看，主体构件和关系如下：

{% asset_img mybatis-y-arch-4.png %}

主要的核心部件解释如下：
* `SqlSession`作为 MyBatis 工作的主要顶层 API，表示和数据库交互的会话，完成必要数据库增删改查功能；
* `Executor` MyBatis 执行器，是 MyBatis 调度的核心，负责 SQL 语句的生成和查询缓存的维护；
* `StatementHandler`封装了JDBC `Statement`操作，负责对JDBC `statement`的操作，如设置参数、将`Statement`结果集转换成`List`集合；
* `ParameterHandler`负责对用户传递的参数转换成 JDBC `Statement`所需要的参数；
* `ResultSetHandler`负责将 JDBC 返回的`ResultSet`结果集对象转换成`List`类型的集合；
* `TypeHandler`负责 java 数据类型和 jdbc 数据类型之间的映射和转换`MappedStatement`；
* `MappedStatement`维护了一条`<select|update|delete|insert>`节点的封装；
* `SqlSource`负责根据用户传递的`parameterObject`，动态地生成 SQL 语句，将信息封装到`BoundSql`对象中，并返回；
* `BoundSql`表示动态生成的 SQL 语句以及相应的参数信息；
* `Configuration` MyBatis 所有的配置信息都维持在`Configuration`对象之中；
