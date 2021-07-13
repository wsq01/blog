---
title: MyBatis XML配置文件
date: 2021-04-10 12:11:21
tags: [MyBatis]
categories: MyBatis
---

MyBatis 配置文件的所有元素：
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration><!-- 配置 -->
  <properties /><!-- 属性 -->
  <settings /><!-- 设置 -->
  <typeAliases /><!-- 类型命名 -->
  <typeHandlers /><!-- 类型处理器 -->
  <objectFactory /><!-- 对象工厂 -->
  <plugins /><!-- 插件 -->
  <environments><!-- 配置环境 -->
      <environment><!-- 环境变量 -->
          <transactionManager /><!-- 事务管理器 -->
          <dataSource /><!-- 数据源 -->
      </environment>
  </environments>
  <databaseIdProvider /><!-- 数据库厂商标识 -->
  <mappers /><!-- 映射器 -->
</configuration>
```
需要注意的是，MyBatis 配置项的顺序不能颠倒。如果颠倒了它们的顺序，那么在 MyBatis 启动阶段就会发生异常，导致程序无法运行。
# properties元素
`properties`属性可以给系统配置一些运行参数，可以放在 XML 文件或者 properties 文件中。一般而言，MyBatis 提供了 3 种方式让我们使用`properties`：`property`子元素、`properties`文件、程序代码传递。
## property 子元素
以下面代码为基础，使用`property`子元素将数据库连接的相关配置进行改写。
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <properties>
    <property name="driver" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8" />
    <property name="username" value="root" />
    <property name="password" value="1128" />
  </properties>
  <typeAliases>
    <typeAlias alias="role" type="com.mybatis.po.Role"/>
  </typeAliases>
  <!--数据库环境-->
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC" />
      <dataSource type="POOLED">
        <property name="driver" value="${driver}" />
        <property name="url"    value="${url}" />
        <property name="username" value="${username}" />
        <property name="password" value="${password}" />
      </dataSource>
    </environment>
  </environments>
  <!-- 映射文件 -->
  <mappers>
    <mapper resource="com/mybatis/mapper/RoleMapper.xml" />
  </mappers>
</configuration>
```
这里使用了元素`<properties>`下的子元素 <property> 定义，用字符串`username`定义数据库用户名，然后就可以在数据库定义中引入这个已经定义好的属性参数，如`${username}`，这样定义一次就可以到处引用了。但是如果属性参数有成百上千个，显然使用这样的方式不是一个很好的选择，这个时候可以使用`properties`文件。
## 使用 properties 文件
我们创建一个文件`jdbc.properties`放到`classpath`的路径下。
```js
database.driver=com.mysql.jdbc.Driver
database.url=jdbc:mysql://localhost:3306/mybatis
database.username=root
database.password=1128
```
在 MyBatis 中通过`<properties>`的属性`resource`来引入`properties`文件。
```xml
<properties resource="jdbc.properties"/>
```
也可以按`${username}`的方法引入`properties`文件的属性参数到 MyBatis 配置文件中。这个时候通过维护`properties`文件就可以维护我们的配置内容了。
## 使用程序传递方式传递参数
在真实的生产环境中，数据库的用户密码是对开发人员和其他人员保密的。运维人员为了保密，一般都需要把用户和密码经过加密成为密文后，配置到`properties`文件中。

对于开发人员及其他人员而言，就不知道其真实的用户密码了，数据库也不可能使用已经加密的字符串去连接，此时往往需要通过解密才能得到真实的用户和密码了。

现在假设系统已经为提供了这样的一个`CodeUtils.decode(str)`进行解密，那么我们在创建`SqlSessionFactory`前，就需要把用户名和密码解密，然后把解密后的字符串重置到`properties`属性中。
```java
String resource = "mybatis-config.xml";
InputStream inputStream;
Inputstream in = Resources.getResourceAsStream("jdbc.properties");
Properties props = new Properties();
props.load(in);
String username = props.getProperty("database.username");
String password = props.getProperty("database.password");
//解密用户和密码，并在属性中重置
props.put("database.username", CodeUtils.decode(username));
props.put ("database.password", CodeUtils.decode(password)); 
inputstream = Resources.getResourceAsStream(resource);
//使用程序传递的方式覆盖原有的properties属性参数
SqlSessionFactory = new SqlSessionFactoryBuilder().build(inputstream, props);
```
首先使用`Resources`对象读取了一个`jdbc.properties`配置文件，然后获取了它原来配置的用户和密码，进行解密并重置，最后使用`SqlSessionFactoryBuilder`的`build`方法，传递多个`properties`参数来完成。

这将覆盖之前配置的密文，这样就能连接数据库了，同时也满足了运维人员对数据库用户和密码安全的要求。
# settings
在 MyBatis 中`settings`是最复杂的配置，它能深刻影响 MyBatis 底层的运行，但是在大部分情况下使用默认值便可以运行，所以在大部分情况下不需要大量配置它，只需要修改一些常用的规则即可，比如自动映射、驼峰命名映射、级联规则、是否启动缓存、执行器（`Executor`）类型等。

| 配置项 | 作用 | 配置选项 | 默认值 |
| :--: | :--: | :--: | :--: |
| cacheEnabled | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存 | true/false | true |
| lazyLoadingEnabled | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态 | true/false | false |
| multipleResultSetsEnabled | 是否允许单个语句返回多结果集（需要数据库驱动支持） | true/false | true |
| useColumnLabel | 使用列标签代替列名。不同的驱动会有不同的表现 | true/false | true |
| useGeneratedKeys | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作 | true/false | false |
| autoMappingBehavior | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE<br>PARTIAL<br>FULL | PARTIAL |
| autoMappingUnkno wnColumnBehavior | 指定自动映射当中未知列（或未知属性类型）时的行为。 默认是不处理，只有当日志级别达到 WARN 级别或者以下，才会显示相关日志，如果处理失败会抛出 SqlSessionException 异常 | NONE<br>WARNING<br>FAILING | NONE |
| defaultExecutorType | 配置默认的执行器。SIMPLE 是普通的执行器；REUSE 会重用预处理语句（prepared statements）；BATCH 执行器将重用语句并执行批量更新 | SIMPLE<br>REUSE<br>BATCH | SIMPLE |
| defaultStatementTimeout | 设置超时时间，它决定驱动等待数据库响应的秒数 | 任何正整数 | 未设置 (null) |
| defaultFetchSize | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数 | 未设置 (null) |
| safeRowBoundsEnabled | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true/false | false |
| mapUnderscoreToCamelCase | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true/false | false |
| localCacheScope | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION<br>STATEMENT | SESSION |
| jdbcTypeForNull | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量<br>常用值：NULL<br>VARCHAR<br>OTHER。 | OTHER |
| lazyLoadTriggerMethods | 指定对象的哪些方法触发一次延迟加载。 | 用逗号分隔的方法列表。 | equals<br>clone<br>hashCode<br>toString |
| logPrefix | 指定 MyBatis 增加到日志名称的前缀 | 任何字符串 | 未设置 |
| loglmpl | 指定 MyBatis 所用日志的具体实现，未指定时将自动査找 | SLF4J/LOG4J/LOG4J2<br>JDK_LOGGING<br>COMMONS_LOGGING<br>ST DOUT_LOGGING<br>NO_LOGGING	| 未设置 |
| proxyFactory | 指定 MyBatis 创建具有延迟加栽能力的对象所用到的代理工具 | CGLIB<br>JAVASSIST | JAVASSIST |

全量的配置样例：
```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```
# typeAliases
由于类的全限定名称很长，需要大量使用的时候，总写那么长的名称不方便。在 MyBatis 中允许定义一个简写来代表这个类，这就是别名，别名分为系统定义别名和自定义别名。

在 MyBatis 中别名由类`TypeAliasRegistry（org.apache.ibatis.type.TypeAliasRegistry）`去定义。注意，在 MyBatis 中别名不区分大小写。
## 系统定义别名
在 MyBatis 的初始化过程中，系统自动初始化了一些别名。

| 别名 | Java 类型 | 是否支持数组 |
| :--: | :--: | :--: |
| _byte | byte | 是 |
| ... | ...| 是 |
| string | String | 是 |
| ... | ...| 是 |
| map | Map | 否 |
| ... | ...| 否 |
| iterator | Iterator | 否 |

如果需要使用对应类型的数组型，要看其是否能支持数据，如果支持只需要使用别名加`[]`即可，比如`_int`数组的别名就是`_int[]`。而类似`list`这样不支持数组的别名，则不能那么写。

有时候要通过代码来实现注册别名，让我们看看 MyBatis 是如何初始化这些别名的。
```java
public TypeAliasRegistry() {
  registerAlias("string", String.class);
  registerAlias("byte", Byte.class);
  registerAlias("long", Long.class);
  ......
  registerAlias("byte[]",Byte[].class); registerAlias("long[]",Long[].class);
  ......
  registerAlias("map", Map.class);
  registerAlias("hashmap", HashMap.class);
  registerAlias("list", List.class); registerAlias("arraylist", ArrayList.class);
  registerAlias("collection", Collection.class);
  registerAlias("iterator", Iterator.class);
  registerAlias("ResultSet", ResultSet.class);
}
```
所以使用`TypeAliasRegistry`的`registerAlias`方法就可以注册别名了。一般是通过`Configuration`获取`TypeAliasRegistry`类对象，其中有一个`getTypeAliasRegistry`方法可以获得别名，如`configuration.getTypeAliasRegistry()`。

然后就可以通过`registerAlias`方法对别名注册了。而事实上`Configuration`对象也对一些常用的配置项配置了别名。
```java
//事务方式别名
typeAliasRegistry.registerAlias("JDBC",JdbcTransactionFactory.class);
typeAliasRegistry.registerAlias("MANAGED",ManagedTransactionFactory.class);
//数据源类型别名
typeAliasRegistry.registerAlias("JNDI",JndiDataSourceFactory.class);
typeAliasRegistry.registerAlias("POOLED",
PooledDataSourceFactory.class);
typeAliasRegistry.registerAlias("UNPOOLED",UnpooledDataSourceFactory.class);
//缓存策略别名
typeAliasRegistry.registerAlias("PERPETUAL",PerpetualCache.class);
typeAliasRegistry.registerAlias("FIFO",FifoCache.class);
typeAliasRegistry.registerAlias("LRU",LruCache.class); typeAliasRegistry.registerAlias("SOFT", SoftCache.class); typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
//数据库标识别名
typeAliasRegistry.registerAlias("DB_VENDOR",
VendorDatabaseIdProvider.class);
//语言驱动类别名
typeAliasRegistry.registerAlias("XML",XMLLanguageDriver.class);
typeAliasRegistry.registerAlias("RAW",RawLanguageDriver.class);
//日志类别名
typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
typeAliasRegistry.registerAlias("COMMONS_LOGGTNG",JakartmCommonsLogginglmpl.class);
typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
typeAliasRegistry.registerAlias("NO_LOGGING",NoLoggingImpl.class);
//动态代理别名
typeAliasRegistry.registerAlias("CGLIB",CglibProxyFactory.class);
typeAliasRegistry.registerAlias("JAVASSIST",JavassistProxyFactory.class);
```
这些配置为的是让我们更容易配置 MyBatis 的相关信息。以上就是 MyBatis 系统定义的别名，我们在使用的时候，不要重复命名，导致出现其他问题。
## 自定义别名
MyBatis 也提供了用户自定义别名的规则。我们可以通过`TypeAliasRegistry`类的`registerAlias`方法注册，也可以采用配置文件或者扫描方式来自定义它。

使用配置文件定义：
```xml
<typeAliases><!--别名-->
  <typeAlias alias="role" type="com.mybatis.po.Role"/>
  <typeAlias alias="role" type="com.mybatis.po.User"/>
</typeAliases>
```
MyBatis 还支持扫描别名。比如上面的两个类都在包`com.mybatis.po`之下，那么就可以定义为：
```xml
<typeAliases><!--别名-->
  <package name="com.mybatis.po"/>
</typeAliases>
```
这样 MyBatis 将扫描这个包里面的类，将其第一个字母变为小写作为其别名，比如类`Role`的别名会变为`role`，而`User`的别名会变为`user`。使用这样的规则，有时候会出现重名。

比如`com.mybatis.po.User`这个类，MyBatis 还增加了对包`com.mybatis.po`的扫描，那么就会出现异常，这个时候可以使用 MyBatis 提供的注解`@Alias("user3")`进行区分。
```java
package com.mybatis.po;
@Alias("user3")
public Class User {
  ......
}
```
这样就能够避免因为别名重名导致的扫描失败的问题。
# environments
在 MyBatis 中，运行环境主要的作用是配置数据库信息，它可以配置多个数据库，一般而言只需要配置其中的一个就可以了。

它下面又分为两个可配置的元素：事务管理器（`transactionManager`）、数据源（`dataSource`）。

在实际的工作中，大部分情况下会采用 Spring 对数据源和数据库的事务进行管理。

运行环境配置：
```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC" />
    <dataSource type="POOLED">
      <property name="driver" value="${database.driver}" />
      <property name="url" value="${database.url}" />
      <property name="username" value="${database.username}" />
      <property name="password" value="${database.password}" />
    </dataSource>
  </environment>
</environments>
```
## transactionManager（事务管理器）
在 MyBatis 中，`transactionManager`提供了两个实现类，它需要实现接口`Transaction（org.apache.ibatis.transaction.Transaction）`，它的定义代码如下所示。
```java
public interface Transaction {
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
  Integer getTimeout() throws SQLException;
}
```
从方法可知，它主要的工作就是提交（`commit`）、回滚（`rollback`）和关闭（`close`）数据库的事务。MyBatis 为`Transaction`提供了两个实现类`JdbcTransaction`和`ManagedTransaction`。

于是它对应着两种工厂：`JdbcTransactionFactory`和`ManagedTransactionFactory`，这个工厂需要实现`TransactionFactory`接口，通过它们会生成对应的`Transaction`对象。于是可以把事务管理器配置成为以下两种方式：
```xml
<transactionManager type="JDBC"/>
<transactionManager type="MANAGED"/>
```
JDBC 使用`JdbcTransactionFactory`生成的`JdbcTransaction`对象实现。它是以 JDBC 的方式对数据库的提交和回滚进行操作。

`MANAGED`使用`ManagedTransactionFactory`生成的`ManagedTransaction`对象实现。它的提交和回滚方法不用任何操作，而是把事务交给容器处理。在默认情况下，它会关闭连接，然而一些容器并不希望这样，因此需要将`closeConnection`属性设置为`false`来阻止它默认的关闭行为。

不想采用 MyBatis 的规则时，我们可以这样配置：
```xml
<transactionManager type="com.mybatis.transaction.MyTransactionFactory"/>
```
实现一个自定义事务工厂，代码如下所示。
```java
public class MyTransactionFactory implements TransactionFactory {
  @Override
  public void setProperties(Properties props) {
  }
  @Override
  public Transaction newTransaction(Connection conn) {
    return new MyTransaction(conn);
  }
  @Override
  public Transaction newTransaction(DataSource dataSource, TransactionlsolationLevel level, boolean autoCommit) {
    return new MyTransaction(dataSource, level, autoCommit);
  }
}
```
这里就实现了`TransactionFactory`所定义的工厂方法，这个时候还需要事务实现类`MyTransaction`，它用于实现`Transaction`接口。
```java
public class MyTransaction extends JdbcTransaction implements Transaction {
  public MyTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
    super(ds, desiredLevel, desiredAutoCommit);
  }
  public MyTransaction(Connection connection) {
    super(connection);
  }
  public Connection getConnection() throws SQLException {
    return super.getConnection();
  }
  public void commit() throws SQLException {
    super.commit();
  }
  public void rollback() throws SQLException {
    super.rollback();
  }
  public void close() throws SQLException {
    super.close();
  }
  public Integer getTimeout() throws SQLException {
    return super.getTimeout();
  }
}
```
这样就能够通过自定义事务规则，满足特殊的需要了。
## environment 数据源环境
`environment`的主要作用是配置数据库，在 MyBatis 中，数据库通过`PooledDataSource Factory、UnpooledDataSourceFactory`和`JndiDataSourceFactory`三个工厂类来提供，前两者对应产生`PooledDataSource、UnpooledDataSource`类对象，而`JndiDataSourceFactory`则会根据`JNDI`的信息拿到外部容器实现的数据库连接对象。

无论如何这三个工厂类，最后生成的产品都会是一个实现了`DataSource`接口的数据库连接对象。

由于存在三种数据源，所以可以按照下面的形式配置它们。
```xml
<dataSource type="UNPOOLED">
<dataSource type="POOLED">
<dataSource type="JNDI">
```
### 1. UNPOOLED
`UNPOOLED`采用非数据库池的管理方式，每次请求都会打开一个新的数据库连接，所以创建会比较慢。在一些对性能没有很高要求的场合可以使用它。

对有些数据库而言，使用连接池并不重要，那么它也是一个比较理想的选择。`UNPOOLED`类型的数据源可以配置以下几种属性：
* `driver`数据库驱动名。
* `url`连接数据库的 URL。
* `username`用户名。
* `password`密码。
* `defaultTransactionIsolationLevel`默认的连接事务隔离级别。

传递属性给数据库驱动也是一个可选项，注意属性的前缀为`driver.`，例如`driver.encoding=UTF8`。它会通过`DriverManager.getConnection(url,driverProperties)`方法传递值为`UTF8`的`encoding`属性给数据库驱动。
### 2. POOLED
数据源`POOLED`利用“池”的概念将 JDBC 的`Connection`对象组织起来，它开始会有一些空置，并且已经连接好的数据库连接，所以请求时，无须再建立和验证，省去了创建新的连接实例时所必需的初始化和认证时间。它还控制最大连接数，避免过多的连接导致系统瓶颈。

除了`UNPOOLED`下的属性外，还有更多属性用来配置`POOLED`的数据源：

| 名称 | 说明 |
| :--: | :--: |
| poolMaximumActiveConnections | 	是在任意时间都存在的活动（也就是正在使用）连接数量，默认值为 10 |
| poolMaximumIdleConnections | 是任意时间可能存在的空闲连接数 |
| poolMaximumCheckoutTime | 在被强制返回之前，池中连接被检出（checked out）的时间，默认值为 20 000 毫秒（即 20 秒） |
| poolPingQuery | 为发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中，并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。 |
| poolPingEnabled | 为是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值为 false。 |
| poolPingConnectionsNotUsedFor | 为配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值为 0（即所有连接每一时刻都被侦测——仅当 poolPingEnabled 为 true 时适用）。 |

### 3. JNDI
数据源 JNDI 的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性：
1. `initial_context`
用来在`InitialContext`中寻找上下文（即，`initialContext.lookup（initial_context）`）。`initial_context`是个可选属性，如果忽略，那么`data_source`属性将会直接从`InitialContext`中寻找。
2. `data_source`
是引用数据源实例位置上下文的路径。当提供`initial_context`配置时，`data_source`会在其返回的上下文中进行查找；当没有提供`initial_context`时`data_source`直接在`InitialContext`中查找。

与其他数据源配置类似，它可以通过添加前缀`env.`直接把属性传递给初始上下文（`InitialContext`）。比如`env.encoding=UTF8`，就会在初始上下文实例化时往它的构造方法传递值为`UTF8`的`encoding`属性。

MyBatis 也支持第三方数据源，例如使用 DBCP 数据源，那么需要提供一个自定义的`DataSourceFactory`。
```java
public class DbcpDataSourceFactory implements DataSourceFactory {
  private Properties props = null;
  public void setProperties(Properties props) {
    this.props = props;
  }
  public DataSource getDataSource() {
    DataSource dataSource = null;
    dataSource = BasicDataSourceFactory.createDataSource(props);
    return dataSource;
  }
}
```
然后进行如下配置：
```xml
<dataSource type="com.mybatis.dataSource.DbcpDataSourceFactory">
  <property name="driver" value="${database.driver}" />
  <property name="url" value="${database.url}" />
  <property name="username" value="${database.username}" />
  <property name="password" value="${database.password}" />
</dataSource>
```
这样 MyBatis 就会采用配置的数据源工厂来生成数据源了。
# 映射器（mappers）
既然 MyBatis 的行为已经由上述元素配置完了，我们现在就要来定义 SQL 映射语句了。但首先，我们需要告诉 MyBatis 到哪里去找到这些语句。 在自动查找资源方面，Java 并没有提供一个很好的解决方案，所以最好的办法是直接告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用，或完全限定资源定位符（包括`file:///`形式的 URL），或类名和包名等。
```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```
这些配置会告诉 MyBatis 去哪里找映射文件，剩下的细节就应该是每个 SQL 映射文件了。