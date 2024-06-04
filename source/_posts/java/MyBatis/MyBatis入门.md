---
title: MyBatis 入门
date: 2021-04-07 16:24:15
tags: [MyBatis]
categories: MyBatis
---

MyBatis 是一个基于 Java 的持久层框架。MyBatis 提供的持久层框架包括 SQL Maps 和 Data Access Objects（DAO），它消除了几乎所有的 JDBC 代码和参数的手工设置以及结果集的检索。

MyBatis 使用简单的 XML 或注解用于配置和映射原始类型，将接口和 Java 的 POJOs（Plain Old Java Objects，普通的 Java 对象）映射成数据库中的记录。

Java 的持久层框架产品常见的有 Hibernate 和 MyBatis。
# 安装
要使用 MyBatis，只需将`mybatis-x.x.x.jar`文件置于类路径（`classpath`）中即可。

如果使用 Maven 来构建项目，则需将下面的依赖代码置于`pom.xml`文件中：
```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```
# MyBatis 的工作原理

{% asset_img 1.png MyBatis 框架的执行流程图 %}

流程说明：
1. 读取 MyBatis 配置文件：`mybatis-config.xml`为 MyBatis 的全局配置文件，配置了 MyBatis 的运行环境等信息，例如数据库连接信息。
2. 加载映射文件。映射文件即 SQL 映射文件，该文件中配置了操作数据库的 SQL 语句，需要在 MyBatis 配置文件`mybatis-config.xml`中加载。`mybatis-config.xml`文件可以加载多个映射文件，每个文件对应数据库中的一张表。
3. 构造会话工厂：通过 MyBatis 的环境等配置信息构建会话工厂`SqlSessionFactory`。
4. 创建会话对象：由会话工厂创建`SqlSession`对象，该对象中包含了执行 SQL 语句的所有方法。
5. `Executor`执行器：MyBatis 底层定义了一个`Executor`接口来操作数据库，它将根据`SqlSession`传递的参数动态地生成需要执行的 SQL 语句，同时负责查询缓存的维护。
6. `MappedStatement`对象：在`Executor`接口的执行方法中有一个`MappedStatement`类型的参数，该参数是对映射信息的封装，用于存储要映射的 SQL 语句的`id`、参数等信息。
7. 输入参数映射：输入参数类型可以是`Map、List`等集合类型，也可以是基本数据类型和 POJO 类型。输入参数映射过程类似于 JDBC 对`preparedStatement`对象设置参数的过程。
8. 输出结果映射：输出结果类型可以是`Map、 List`等集合类型，也可以是基本数据类型和 POJO 类型。输出结果映射过程类似于 JDBC 对结果集的解析过程。

# MyBatis的核心组件
MyBatis 的核心组件分为 4 个部分。
1. `SqlSessionFactoryBuilder`（构造器）：它会根据配置或者代码来生成`SqlSessionFactory`，采用的是分步构建的`Builder`模式。
2. `SqlSessionFactory`（工厂接口）：依靠它来生成`SqlSession`，使用的是工厂模式。
3. `SqlSession`（会话）：一个既可以发送 SQL 执行返回结果，也可以获取`Mapper`的接口。一般我们会让其在业务逻辑代码中“消失”，而使用的是 MyBatis 提供的`SQL Mapper`接口编程技术，它能提高代码的可读性和可维护性。
4. `SQL Mapper`（映射器）：它由一个 Java 接口和 XML 文件（或注解）构成，需要给出对应的 SQL 和映射规则。它负责发送 SQL 去执行，并返回结果。

注意，无论是映射器还是`SqlSession`都可以发送 SQL 到数据库执行。
# SqlSessionFactory
使用 MyBatis 首先是使用配置或者代码去生产`SqlSessionFactory`，MyBatis 提供了构造器`SqlSessionFactoryBuilder`。它提供了一个类`org.apache.ibatis.session.Configuration`作为引导，采用的是`Builder`模式。具体的分步则是在`Configuration`类里面完成的。

在 MyBatis 中，既可以通过读取配置的 XML 文件的形式生成`SqlSessionFactory`，也可以通过 Java 代码的形式去生成`SqlSessionFactory`。

当配置了 XML 或者提供代码后，MyBatis 会读取配置文件，通过`Configuration`类对象构建整个 MyBatis 的上下文。

注意，`SqlSessionFactory`是一个接口，在 MyBatis 中它存在两个实现类：`SqlSessionManager`和`DefaultSqlSessionFactory`。

一般而言，具体是由`DefaultSqlSessionFactory`去实现的，而`SqlSessionManager`使用在多线程的环境中，它的具体实现依靠`DefaultSqlSessionFactory`。

{% asset_img 2.png SqlSessionFactory 的生成 %}

每个基于 MyBatis 的应用都是以一个`SqlSessionFactory`的实例为中心的，而`SqlSessionFactory`唯一的作用就是生产 MyBatis 的核心接口对象`SqlSession`，所以它的责任是唯一的。我们往往会采用单例模式处理它。
## 使用 XML 构建 SqlSessionFactory
在 MyBatis 中的 XML 分为两类，一类是基础配置文件，通常只有一个，主要是配置一些最基本的上下文参数和运行环境；另一类是映射文件，它可以配置映射关系、SQL、参数等信息。

基础配置文件`mybatis-config.xml`，放在工程类路径下。
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <typeAliases><!--别名-->
    <typeAliases alias="user" type="com.mybatis.po.User"/>
  </typeAliases>
  <!-- 数据库环境 -->
  <environments default="development">
    <environment id="development">
      <!-- 使用JDBC的事务管理 -->
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
        <!-- MySQL数据库驱动 -->
        <property name="driver" value="com.mysql.jdbc.Driver" />
        <!-- 连接数据库的URL -->
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8" />
        <property name="username" value="root" />
        <property name="password" value="1128" />
      </dataSource>
    </environment>
  </environments>
  <!-- 将mapper文件加入到配置文件中 -->
  <mappers>
    <mapper resource="com/mybatis/mapper/UserMapper.xml" />
  </mappers>
</configuration>
```
有了基础配置文件，就可以生成`SqlSessionFactory`了。
```java
SqlSessionFactory factory = null;
String resource = "mybatis-config.xml";
InputStream is;
try {
  InputStream is = Resources.getResourceAsStream(resource);
  factory = new SqlSessionFactoryBuilder().build(is);
} catch (IOException e) {
  e.printStackTrace();
}
```
首先读取`mybatis-config.xml`，然后通过`SqlSessionFactoryBuilder`的`Builder`方法去创建`SqlSessionFactory`。整个过程比较简单，而里面的步骤还是比较烦琐的，只是 MyBatis 采用了`Builder`模式隐藏了这些细节。这样一个`SqlSessionFactory`就被创建出来了。
## 使用代码创建 SqlSessionFactory
通过代码来实现与使用 XML 构建`SqlSessionFactory`一样的功能。
```java
// 数据库连接池信息
PooledDataSource dataSource = new PooledDataSource();
dataSource.setDriver("com.mysql.jdbc.Driver");
dataSource.setUsername("root");
dataSource.setPassword ("1128");
dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis");
dataSource.setDefeultAutoCommit(false);
// 采用 MyBatis 的 JDBC 事务方式
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment ("development", transactionFactory, dataSource);
// 创建 Configuration 对象
Configuration configuration = new Configuration(environment);
// 注册一个 MyBatis 上下文别名
configuration.getTypeAliasRegistry().registerAlias("role", Role.class);
// 加入一个映射器
configuration.addMapper(RoleMapper.class);
//使用 SqlSessionFactoryBuilder 构建 SqlSessionFactory
SqlSessionFactory SqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
return SqlSessionFactory;
```
它和 XML 方式实现的功能是一致的。但是代码冗长，如果发生系统修改，那么有可能需要重新编译代码才能继续，所以这不是一个很好的方式。

除非有特殊的需要，比如在配置文件中，需要配置加密过的数据库用户名和密码，需要我们在生成`SqlSessionFactory`前解密为明文的时候，才会考虑使用这样的方式。
# SqlSession
在 MyBatis 中，`SqlSession`是其核心接口。在 MyBatis 中有两个实现类，`DefaultSqlSession`和`SqlSessionManager`。

`DefaultSqlSession`是单线程使用的，而`SqlSessionManager`在多线程环境下使用。`SqlSession`的作用类似于一个 JDBC 中的`Connection`对象，代表着一个连接资源的启用。它的作用有 3 个：获取`Mapper`接口、发送 SQL 给数据库、控制数据库事务。

创建`SqlSession`方法：
```java
SqlSession sqlSession = SqlSessionFactory.openSession();
```
注意，`SqlSession`只是一个门面接口，它有很多方法，可以直接发送 SQL。它就好像一家软件公司的商务人员，是一个门面，而实际干活的是软件工程师。在 MyBatis 中，真正干活的是`Executor`。

`SqlSession`控制数据库事务的方法：
```java
//定义 SqlSession
SqlSession sqlSession = null;
try {
  // 打开 SqlSession 会话
  sqlSession = SqlSessionFactory.openSession();
  // some code...
  sqlSession.commit();    // 提交事务
} catch (IOException e) {
  sqlSession.rollback();  // 回滚事务
} finally {
  // 在 finally 语句中确保资源被顺利关闭
  if (sqlSession != null){
    sqlSession.close();
  }
}
```
这里使用`commit`方法提交事务，或者使用`rollback`方法回滚事务。因为它代表着一个数据库的连接资源，使用后要及时关闭它，如果不关闭，那么数据库的连接资源就会很快被耗费光，整个系统就会瘫痪，所以用`finally`语句保证其顺利关闭。
# 实现映射器的2种方式
映射器由一个接口和对应的 XML 文件（或注解）组成。它可以配置以下内容：
* 描述映射规则。
* 提供 SQL 语句，并可以配置 SQL 参数类型、返回类型、缓存刷新等信息。
* 配置缓存。
* 提供动态 SQL。

两种实现映射器的方式，XML 文件形式和注解形式。
```sql
CREATE TABLE `role` (
  `id` bigint(20) NOT NULL,
  `role_name` varchar(20) DEFAULT NULL,
  `note` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
定义一个 POJO。
```java
package com.mybatis.po;
public class Role {
  private Long id;
  private String roleName;
  private String note;
/**setter and getter**/
}
```
映射器的主要作用就是将 SQL 查询到的结果映射为一个 POJO，或者将 POJO 的数据插入到数据库中，并定义一些关于缓存等的重要内容。

注意，开发只是一个接口，而不是一个实现类。接口不能直接运行。MyBatis 运用了动态代理技术使得接口能运行起来，MyBatis 会为这个接口生成一个代理对象，代理对象会去处理相关的逻辑。
## 用 XML 实现映射器
用 XML 定义映射器分为两个部分：接口和 XML。先定义一个映射器接口。
```java
package com.mybatis.mapper;
import com.mybatis.po.Role;
public interface RoleMapper {
  public Role getRole(Long id);
}
```
在用 XML 方式创建`SqlSession`的配置文件中有这样一段代码：
```xml
<mapper resource="com/mybatis/mapper/RoleMapper.xml" />
```
它的作用就是引入一个 XML 文件。用 XML 方式创建映射器。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.RoleMapper">
  <select id="getRole" parameterType="long" resultType="role">
    SELECT id,role_name as roleName,note FROM role WHERE id =#{id}
  </select>
</mapper>
```
有了这两个文件，就完成了一个映射器的定义。

`<mapper>`元素中的属性`namespace`所对应的是一个接口的全限定名，于是 MyBatis 上下文就可以通过它找到对应的接口。

`<select>`元素表明这是一条查询语句，而属性`id`标识了这条 SQL，属性`parameterType="long"`说明传递给 SQL 的是一个`long`型的参数，而`resultType="role"`表示返回的是一个`role`类型的返回值。而`role`是之前配置文件`mybatis-config.xml`配置的别名，指代的是`com.mybatis.po.Role`。

注意，我们并没有配置 SQL 执行后和 role 的对应关系，它是如何映射的呢？

其实这里采用的是一种被称为自动映射的功能，MyBatis 在默认情况下提供自动映射，只要 SQL 返回的列名能和 POJO 对应起来即可。

这里 SQL 返回的列名`id`和`note`是可以和之前定义的 POJO 的属性对应起来的，而表里的列`role_name`通过 SQL 别名的改写，使其成为`roleName`，也是和 POJO 对应起来的，所以此时 MyBatis 就可以把 SQL 查询的结果通过自动映射的功能映射成为一个 POJO。
## 注解实现映射器
除 XML 方式定义映射器外，还可以采用注解方式定义映射器，它只需要一个接口就可以通过 MyBatis 的注解来注入 SQL。
```java
package com.mybatis.mapper;
import org.apache.ibatis.annotations.Select;
import com.mybatis.po.Role;
public interface RoleMapper2 {
  @Select("select id,role_name as roleName,note from t_role where id=#{id}")
  public Role getRole(Long id);
}
```
这完全等同于 XML 方式创建映射器。如果它和 XML 方式同时定义时，XML 方式将覆盖掉注解方式。此外，XML 可以相互引入，而注解是不可以的，所以在一些比较复杂的场景下，使用 XML 方式会更加灵活和方便。

这个接口可以在 XML 中定义，我们仿造在`mybatis-config.xml`中配置 XML 语句：
```xml
<mapper resource="com/mybatis/mapper/RoleMapper.xml" />
```
把它修改为下面的形式即可。
```xml
<mapper resource="com/mybatis/mapper/RoleMapper2" />
```
也可以使用`configuration`对象注册这个接口：
```java
configuration.addMapper(RoleMapper2.class);
```
# 执行SQL的两种方式
## SqlSession 发送 SQL
有了映射器就可以通过`SqlSession`发送 SQL 了。
```java
Role role = (Role)sqlSession.select("com.mybatis.mapper.RoleMapper.getRole", 1L);
```
`selectOne`方法表示使用查询并且只返回一个对象，而参数则是一个`String`对象和一个`Object`对象。这里是一个`long`参数，`long`参数是它的主键。

`String`对象是由一个命名空间加上 SQL `id`组合而成的，它完全定位了一条 SQL，这样 MyBatis 就会找到对应的 SQL。如果在 MyBatis 中只有一个`id`为`getRole`的 SQL，那么也可以简写为：
```java
Role role = (Role)sqlSession.selectOne("getRole",1L);
```
这是 MyBatis 前身 iBatis 所留下的方式。
## 用 Mapper 接口发送 SQL
`SqlSession`还可以获取`Mapper`接口，通过`Mapper`接口发送 SQL。
```java
RoleMapper roleMapper = sqlSession.getMapper(RoleMapper.class);
Role role = roleMapper.getRole(1L);
```
通过`SqlSession`的`getMapper`方法来获取一个`Mapper`接口，就可以调用它的方法了。因为 XML 文件或者接口注解定义的 SQL 都可以通过“类的全限定名+方法名”查找，所以 MyBatis 会启用对应的 SQL 进行运行，并返回结果。
## 对比两种发送 SQL 方式
建议采用`SqlSession`获取`Mapper`的方式，理由如下：

使用`Mapper`接口编程可以消除`SqlSession`带来的功能性代码，提高可读性，而`SqlSession`发送 SQL，需要一个 SQL `id`去匹配 SQL，比较晦涩难懂。使用`Mapper`接口，类似`roleMapper.getRole（1L）`则是完全面向对象的语言，更能体现业务的逻辑。

使用`Mapper.getRole（1L）`方式，IDE 会提示错误和校验，而使用`sqlSession.selectOne（“getRole”,1L）`语法，只有在运行中才能知道是否会产生错误。

目前使用`Mapper`接口编程已成为主流，尤其在 Spring 中运用 MyBatis 时，`Mapper`接口的使用就更为简单
# 核心组件的作用域和生命周期
## SqlSessionFactoryBuilder
`SqlSessionFactoryBuilder`的作用在于创建`SqlSessionFactory`，创建成功后，`SqlSessionFactoryBuilder`就失去了作用，所以它只能存在于创建`SqlSessionFactory`的方法中，而不要让其长期存在。因此`SqlSessionFactoryBuilder`实例的最佳作用域是方法作用域（也就是局部方法变量）。
## SqlSessionFactory
`SqlSessionFactory`可以被认为是一个数据库连接池，它的作用是创建`SqlSession`接口对象。因为 MyBatis 的本质就是 Java 对数据库的操作，所以`SqlSessionFactory`的生命周期存在于整个 MyBatis 的应用之中，所以一旦创建了`SqlSessionFactory`，就要长期保存它，直至不再使用 MyBatis 应用，所以可以认为`SqlSessionFactory`的生命周期就等同于 MyBatis 的应用周期。

由于`SqlSessionFactory`是一个对数据库的连接池，所以它占据着数据库的连接资源。如果创建多个`SqlSessionFactory`，那么就存在多个数据库连接池，这样不利于对数据库资源的控制，也会导致数据库连接资源被消耗光，出现系统宕机等情况，所以尽量避免发生这样的情况。

因此在一般的应用中我们往往希望`SqlSessionFactory`作为一个单例，让它在应用中被共享。所以说`SqlSessionFactory`的最佳作用域是应用作用域。
## SqlSession
如果说`SqlSessionFactory`相当于数据库连接池，那么`SqlSession`就相当于一个数据库连接（`Connection`对象），你可以在一个事务里面执行多条 SQL，然后通过它的`commit、rollback`等方法，提交或者回滚事务。

所以它应该存活在一个业务请求中，处理完整个请求后，应该关闭这条连接，让它归还给`SqlSessionFactory`，否则数据库资源就很快被耗费精光，系统就会瘫痪，所以用`try...catch...finally...`语句来保证其正确关闭。

所以`SqlSession`的最佳的作用域是请求或方法作用域。
## Mapper
`Mapper`是一个接口，它由`SqlSession`所创建，所以它的最大生命周期至多和`SqlSession`保持一致，尽管它很好用，但是由于`SqlSession`的关闭，它的数据库连接资源也会消失，所以它的生命周期应该小于等于`SqlSession`的生命周期。`Mapper`代表的是一个请求中的业务处理，所以它应该在一个请求中，一旦处理完了相关的业务，就应该废弃它。

{% asset_img 3.png MyBatis 组件的生命周期 %}

# resultMap元素
`<resultMap>`元素表示结果映射集，是 MyBatis 中最重要也是最强大的元素，主要用来定义映射规则、级联的更新以及定义类型转化器等。
## resultMap 元素的结构
```xml
<resultMap id="" type="">
  <constructor><!-- 类再实例化时用来注入结果到构造方法 -->
    <idArg/><!-- ID参数，结果为ID -->
    <arg/><!-- 注入到构造方法的一个普通结果 -->  
  </constructor>
  <id/><!-- 用于表示哪个列是主键 -->
  <result/><!-- 注入到字段或JavaBean属性的普通结果 -->
  <association property=""/><!-- 用于一对一关联 -->
  <collection property=""/><!-- 用于一对多、多对多关联 -->
  <discriminator javaType=""><!-- 使用结果值来决定使用哪个结果映射 -->
    <case value=""/><!-- 基于某些值的结果映射 -->
  </discriminator>
</resultMap>
```
* `<resultMap>`元素的`type`属性表示需要的`POJO`，`id`属性是`resultMap`的唯一标识。
* 子元素`<constructor>`用于配置构造方法（当 POJO 未定义无参数的构造方法时使用）。
* 子元素`<id>`用于表示哪个列是主键。
* 子元素`<result>`用于表示POJO和数据表普通列的映射关系。
* 子元素`<association>、<collection>`和`<discriminator>`用在级联的情况下。

一条查询 SQL 语句执行后将返回结果，而结果可以使用`Map`存储，也可以使用`POJO`存储。
## 使用 Map 存储结果集
任何`select`语句都可以使用`Map`存储结果：
```xml
<!-- 查询所有用户信息存到Map中 -->
<select id="selectAllUserMap" resultType="map">
    select * from user
</select>
```
测试上述 SQL 配置文件的过程如下：

首先在`com.dao.UserDao`接口中添加以下接口方法。
```java
public List<Map<String,Object>> selectAllUserMap();
```
然后在`com.controller`包的`UserController`类中调用接口方法。
```java
// 查询所有用户信息存到Map中
List<Map<String, Object>> lmp = userDao.selectAllUserMap();
for (Map<String, Object> map : lmp) {
  System.out.println(map);
}
```
上述 Map 的 key 是 select 语句查询的字段名（必须完全一样），而 Map 的 value 是查询返回结果中字段对应的值，一条记录映射到一个 Map 对象中。Map 用起来很方便，但可读性稍差，有的开发者不太喜欢使用 Map，更多时候喜欢使用 POJO 的方式。
## 使用POJO存储结果集
使用`POJO`的方式存储结果集，一方面可以使用自动映射，例如使用`resultType`属性，但有时候需要更为复杂的映射或级联，这时候就需要使用`<select>`元素的`resultMap`属性配置映射集合：
### 1.创建 POJO 类
在应用的`com.pojo`包中创建`POJO`类`MapUser`。
```java
package com.pojo;
public class MapUser {
  private Integer m_uid;
  private String m_uname;
  private String m_usex;
  // 此处省略setter和getter方法
  @Override
  public String toString() {
    return "User[uid=" + m_uid + ",uname=" + m_uname + ",usex=" + m_usex + "]";
  }
}
```
### 2.配置 resultMap 元素
在 SQL 映射文件`UserMapper.xml`中配置`<resultMap>`元素，其属性`type`引用`POJO`类。
```xml
<!--使用自定义结果集类型-->
<resultMap type="com.pojo.MapUser" id="myResult">
  <!-- property 是 com.pojo.MapUser 类中的属性-->
  <!-- column是查询结果的列名，可以来自不同的表-->
  <id property="m_uid" column="uid"/>
  <result property="m_uname" column="uname"/>
  <result property="m_usex" column="usex"/>
</resultMap>
```
### 3.配置 select 元素
在 SQL 映射文件`UserMapper.xml`中配置`<select>`元素，其属性`resultMap`引用了`<resultMap>`元素的`id`。
```xml
<!-- 使用自定义结果集类型查询所有用户 -->
<select id="selectResultMap" resultMap="myResult">
    select * from user
</select>
```
### 4.添加接口方法
在`com.dao.UserDao`接口中添加以下接口方法：
```xml
public List<MapUser> selectResultMap();
```
### 5.调用接口方法
在`com.controller`包的`UserController`类中调用接口方法：
```java
// 使用resultMap映射结果集
List<MapUser> listResultMap = userDao.selectResultMap();
for (MapUser myUser : listResultMap) {
  System.out.println(myUser);
}
```