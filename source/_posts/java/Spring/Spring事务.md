



事务（Transaction）是面向关系型数据库（RDBMS）企业应用程序的重要组成部分，用来确保数据的完整性和一致性。

事务具有以下 4 个特性，即原子性、一致性、隔离性和持久性，这 4 个属性称为 ACID 特性。
原子性（Atomicity）：一个事务是一个不可分割的工作单位，事务中包括的动作要么都做要么都不做。
一致性（Consistency）：事务必须保证数据库从一个一致性状态变到另一个一致性状态，一致性和原子性是密切相关的。
隔离性（Isolation）：一个事务的执行不能被其它事务干扰，即一个事务内部的操作及使用的数据对并发的其它事务是隔离的，并发执行的各个事务之间不能互相打扰。
持久性（Durability）：持久性也称为永久性，指一个事务一旦提交，它对数据库中数据的改变就是永久性的，后面的其它操作和故障都不应该对其有任何影响。
编程式和声明式
Spring 的事务管理有 2 种方式：
传统的编程式事务管理，即通过编写代码实现的事务管理；
基于 AOP 技术实现的声明式事务管理。
1. 编程式事务管理
编程式事务管理是通过编写代码实现的事务管理，灵活性高，但难以维护。
2. 声明式事务管理
Spring 声明式事务管理在底层采用了 AOP 技术，其最大的优点在于无须通过编程的方式管理事务，只需要在配置文件中进行相关的规则声明，就可以将事务规则应用到业务逻辑中。

Spring 实现声明式事务管理主要有 2 种方式：
基于 XML 方式的声明式事务管理。
通过 Annotation 注解方式的事务管理。

显然声明式事务管理要优于编程式事务管理。
事务管理接口
PlatformTransactionManager、TransactionDefinition 和 TransactionStatus 是事务的 3 个核心接口。
PlatformTransactionManager接口
PlatformTransactionManager 接口用于管理事务，接口定义如下：
```
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
    void commit(TransactionStatus status) throws TransactionException;
    void rollback(TransactionStatus status) throws TransactionException;
}
```
该接口中方法说明如下：

名称	说明
TransactionStatus getTransaction(TransactionDefinition definition)	用于获取事务的状态信息
void commit(TransactionStatus status)	用于提交事务
 void rollback(TransactionStatus status)	用于回滚事务
在项目中，Spring 将 xml 中配置的事务信息封装到对象 TransactionDefinition 中，然后通过事务管理器的 getTransaction() 方法获得事务的状态（TransactionStatus），并对事务进行下一步的操作。
TransactionDefinition接口
TransactionDefinition 接口提供了获取事务相关信息的方法，接口定义如下。
```
public interface TransactionDefinition {
    int getPropagationBehavior();
    int getIsolationLevel();
    String getName();
    int getTimeout();
    boolean isReadOnly();
}
```
该接口中方法说明如下。

方法	说明
String getName()	获取事务的名称
int getIsolationLevel()	获取事务的隔离级别
int getPropagationBehavior()	获取事务的传播行为
int getTimeout()	获取事务的超时时间
boolean isReadOnly()	获取事务是否只读以下是隔离级别的值。

方法	说明
ISOLATION_DEFAULT	使用后端数据库默认的隔离级别
ISOLATION_READ_UNCOMMITTED	允许读取尚未提交的更改，可能导致脏读、幻读和不可重复读
ISOLATION_READ_COMMITTED	（Oracle 默认级别）允许读取已提交的并发事务，防止脏读，可能出现幻读和不可重复读
ISOLATION_REPEATABLE_READ	（MySQL 默认级别），多次读取相同字段的结果是一致的，防止脏读和不可重复读，可能出现幻读
ISOLATION_SERIALIZABLE	完全服从 ACID 的隔离级别，防止脏读、不可重复读和幻读
关于事务隔离级别，详细介绍推荐阅读《数据库事务隔离级别》一节。

以下是传播行为的可能值，传播行为用来控制是否需要创建事务以及如何创建事务。

名称	说明
PROPAGATION_MANDATORY	支持当前事务，如果不存在当前事务，则引发异常
PROPAGATION_NESTED	如果当前事务存在，则在嵌套事务中执行
PROPAGATION_NEVER	不支持当前事务，如果当前事务存在，则引发异常
PROPAGATION_NOT_SUPPORTED	不支持当前事务，始终以非事务方式执行
PROPAGATION_REQUIRED	默认传播行为，支持当前事务，如果不存在，则创建一个新的
PROPAGATION_REQUIRES_NEW	创建新事务，如果已经存在事务则暂停当前事务
PROPAGATION_SUPPORTS	支持当前事务，如果不存在事务，则以非事务方式执行
TransactionStatus接口
TransactionStatus 接口提供了一些简单的方法来控制事务的执行和查询事务的状态，接口定义如下。
```java
public interface TransactionStatus extends SavepointManager {
    boolean isNewTransaction();
    boolean hasSavepoint();
    void setRollbackOnly();
    boolean isRollbackOnly();
    boolean isCompleted();
}
```
该接口中方法说明如下。

名称	说明
boolean hasSavepoint()	获取是否存在保存点
boolean isCompleted()	获取事务是否完成
boolean isNewTransaction()	获取是否是新事务
boolean isRollbackOnly()	获取事务是否回滚
void setRollbackOnly()	设置事务回滚
# 编程式事务管理
编程式事务管理是通过编写代码实现的事务管理，包括定义事务的开始、正常执行后的事务提交和异常时的事务回滚。

Spring 出现以前，编程式事务管理是基于 POJO 应用的唯一选择。在 Hibernate 中，我们需要在代码中显式调用 beginTransaction()、commit()、rollback() 等事务管理相关的方法，这就是编程式事务管理。而通过 Spring 提供的事务管理 API，我们可以在代码中灵活控制事务的执行。

下面根据在《Spring事务》一节讲解的 PlatformTransactionManager、TransactionDefinition 和 TransactionStatus 三个核心接口，通过编程的方式实现事务管理。
示例
下面使用 Eclipse IDE 演示 Spring 中编程式事务管理，步骤如下：
创建 SpringDemo 项目，并在 src 目录下创建 net.biancheng 包。
导入 Spring 相关 JAR 包及 mysql-connector-java.x.x.x.jar 包。
在 net.biancheng 包下创建 User、UserDao、UserDaoImpl、Beans.xml 和 MainApp。
运行 SpringDemo 项目。

User 类代码如下。
```java
package net.biancheng;
public class User {
    private int id;
    private String name;
    private int age;
    public User() {
    }
    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    
    // 省略set和get方法
}
```
UserDao 代码如下。
```java
package net.biancheng;
import java.util.List;
public interface UserDao {
    /**
     * 初始化User表
     */
    void createUserTable();
    /**
     * 保存用户
     */
    void saveUser(User user);
    /**
     * 查询用户
     */
    List<User> listUser();
}
```
UserDaoImpl 代码如下。
```
package net.biancheng;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.support.DefaultTransactionDefinition;
public class UserDaoImpl implements UserDao {
    private JdbcTemplate jdbcTemplate;
    private UserDao userDao;
    private PlatformTransactionManager transactionManager;
    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    public UserDao getUserDao() {
        return userDao;
    }
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    public void setDataSource(DataSource datasource) {
        this.jdbcTemplate = new JdbcTemplate(datasource);
    }
    public void setTransactionManager(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }
    @Override
    public void createUserTable() {
        this.jdbcTemplate.execute("CREATE TABLE `user` (\r\n" + "  `id` int(11) NOT NULL AUTO_INCREMENT,\r\n"
                + "  `name` varchar(50) DEFAULT NULL,\r\n" + "  `age` int(11) DEFAULT NULL,\r\n"
                + "  PRIMARY KEY (`id`)\r\n" + ") ENGINE=InnoDB DEFAULT CHARSET=utf8;");
    }
    @Override
    public void saveUser(User user) {
        TransactionDefinition def = new DefaultTransactionDefinition();
        // getTransaction()用于启动事务，返回TransactionStatus实例对象
        TransactionStatus status = transactionManager.getTransaction(def);
        try {
            this.jdbcTemplate.update("INSERT INTO USER(NAME,age) VALUES (?,?)", user.getName(), user.getAge());
            transactionManager.commit(status);
            System.out.println("commit!");
        } catch (Exception e) {
            System.out.println("Error in creating record, rolling back");
            transactionManager.rollback(status);
            throw e;
        }
    }
    @Override
    public List<User> listUser() {
        List<User> users = this.jdbcTemplate.query("SELECT NAME,age FROM USER", new RowMapper<User>() {
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setName(rs.getString("name"));
                user.setAge(rs.getInt("age"));
                return user;
            }
        });
        return users;
    }
}
```
Beans.xml 代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    <!-- 配置数据源 -->
    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--数据库驱动 -->
        <property name="driverClassName"
            value="com.mysql.jdbc.Driver" />
        <!--连接数据库的url -->
        <property name="url" value="jdbc:mysql://localhost/test" />
        <!--连接数据库的用户名 -->
        <property name="username" value="root" />
        <!--连接数据库的密码 -->
        <property name="password" value="root" />
    </bean>
    <!--配置JDBC模板 -->
    <bean id="jdbcTemplate"
        class="org.springframework.jdbc.core.JdbcTemplate">
        <!--默认必须使用数据源 -->
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userdao" class="net.biancheng.UserDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
        <property name="transactionManager" ref="transactionManager" />
    </bean>
</beans>
```
MainApp 类代码如下。
```
package net.biancheng;
import java.util.List;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("Beans.xml");
        UserDao dao = (UserDao) ctx.getBean("userdao");
        dao.createUserTable();
        dao.saveUser(new User("bianchengbang", 12));
        dao.saveUser(new User("baidu", 18));
        List<User> users = dao.listUser();
        for (User user : users) {
            System.out.println("姓名：" + user.getName() + "\t年龄：" + user.getAge());
        }
    }
}
```
事务提交，运行结果如下。
```
commit!
commit!
姓名：bianchengbang 年龄：12
姓名：baidu 年龄：18
```
如果您想模拟事务回滚场景，在提交事务时抛出异常即可。
# 基于XML实现事务管理
Spring 声明式事务管理是通过 AOP 实现的，其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

声明式事务最大的优点就是不需要通过编程的方式管理事务，可以将业务逻辑代码和事务管理代码很好的分开。

Spring 实现声明式事务管理主要有 2 种方式：
基于 XML 方式的声明式事务管理。
通过 Annotation 注解方式的事务管理。

下面介绍如何通过 XML 的方式实现声明式事务管理，步骤如下。
示例
下面使用 Eclipse IDE 演示通过 XML 方式实现声明式事务管理，步骤如下：
创建 SpringDemo 项目，并在 src 目录下创建 net.biancheng 包。
导入 Spring 相关 JAR 包及 mysql-connector-java.x.x.x.jar 包。
在 net.biancheng 包下创建 User、UserDao、UserDaoImpl、Beans.xml 和 MainApp。
运行 SpringDemo 项目。

User 类代码如下。
```java
package net.biancheng;
public class User {
    private int id;
    private String name;
    private int age;
    public User() {
    }
    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    
    // 省略set和get方法
}
```
UserDao 代码如下。
```java
package net.biancheng;
import java.util.List;
public interface UserDao {
    /**
     * 初始化User表
     */
    void createUserTable();
    /**
     * 保存用户
     */
    void saveUser(User user);
    /**
     * 查询用户
     */
    List<User> listUser();
}
```
UserDaoImpl 代码如下。
```java
package net.biancheng;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
public class UserDaoImpl implements UserDao {
    private JdbcTemplate jdbcTemplate;
    private UserDao userDao;
    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    public UserDao getUserDao() {
        return userDao;
    }
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    public void setDataSource(DataSource datasource) {
        this.jdbcTemplate = new JdbcTemplate(datasource);
    }
    @Override
    public void createUserTable() {
        this.jdbcTemplate.execute("CREATE TABLE `user` (\r\n" + "  `id` int(11) NOT NULL AUTO_INCREMENT,\r\n"
                + "  `name` varchar(50) DEFAULT NULL,\r\n" + "  `age` int(4) DEFAULT NULL,\r\n"
                + "  PRIMARY KEY (`id`)\r\n" + ") ENGINE=InnoDB DEFAULT CHARSET=utf8;");
    }
    @Override
    public void saveUser(User user) {
        try {
            this.jdbcTemplate.update("INSERT INTO USER(NAME,age) VALUES (?,?)", user.getName(), user.getAge());
            throw new RuntimeException("simulate Error condition");
        } catch (Exception e) {
            System.out.println("Error in creating record, rolling back");
            throw e;
        }
    }
    @Override
    public List<User> listUser() {
        List<User> users = this.jdbcTemplate.query("SELECT NAME,age FROM USER", new RowMapper<User>() {
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setName(rs.getString("name"));
                user.setAge(rs.getInt("age"));
                return user;
            }
        });
        return users;
    }
}
```
Beans.xml 代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    <!-- 配置数据源 -->
    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--数据库驱动 -->
        <property name="driverClassName"
            value="com.mysql.jdbc.Driver" />
        <!--连接数据库的url -->
        <property name="url" value="jdbc:mysql://localhost/test" />
        <!--连接数据库的用户名 -->
        <property name="username" value="root" />
        <!--连接数据库的密码 -->
        <property name="password" value="root" />
    </bean>
    <!-- 编写通知：对事务进行增强（通知），需要编写切入点和具体执行事务的细节 -->
    <tx:advice id="txAdvice"
        transaction-manager="transactionManager">
        <tx:attributes>
             <!-- 给切入点方法添加事务详情，name表示方法名称，*表示任意方法名称，propagation用于设置传播行为，read-only表示隔离级别，是否只读 -->
            <tx:method name="*" propagation="SUPPORTS" readOnly = "false"/>
        </tx:attributes>
    </tx:advice>
    <!-- aop编写，让Spring自动对目标生成代理，需要使用AspectJ的表达式 -->
    <aop:config>
        <!-- 切入点，execution 定义的表达式表示net.biencheng包下的所有类所有方法都应用该是事务 -->
        <aop:pointcut id="createOperation"
            expression="execution(* net.biancheng.*.*(..))" />
       
        <aop:advisor advice-ref="txAdvice"
            pointcut-ref="createOperation" />
    </aop:config>
    <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="jdbcTemplate"
        class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userdao" class="net.biancheng.UserDaoImpl">
        <property name="dataSource" ref="dataSource" />
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean>
</beans>
```
MainApp 类代码如下。
```java
package net.biancheng;
import java.util.List;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("Beans.xml");
        UserDao dao = (UserDao) ctx.getBean("userdao");
        dao.createUserTable();
        dao.saveUser(new User("bianchengbang", 12));
        dao.saveUser(new User("baidu", 18));
        List<User> users = dao.listUser();
        for (User user : users) {
            System.out.println("姓名：" + user.getName() + "\t年龄：" + user.getAge());
        }
    }
}
```
运行结果如下。
```
Error in creating record, rolling back
Exception in thread "main" java.lang.RuntimeException: simulate Error condition
...
```
# 基于注解实现事务管理
在 Spring 中，声明式事务除了可以使用 XML 实现外，还可以使用 Annotation 注解。使用注解实现可以减少代码之间的耦合度。

使用 Annotation 的方式非常简单，只需要在项目中做两件事，具体如下。
1）在 Spring 容器中注册驱动，代码如下所示：
```
<tx:annotation-driven transaction-manager="txManager"/>
```
2）在需要使用事务的业务类或者方法中添加注解 @Transactional，并配置 @Transactional 的参数。关于 @Transactional 的参数如图 1 所示。

常用属性说明如下：
propagation：设置事务的传播行为；
isolation：设置事务的隔离级别；
readOnly：设置是读写事务还是只读事务；
timeout：事务超时事件（单位：s）。
示例
下面使用 Eclipse IDE 演示使用注解实现声明式事务管理，步骤如下：
创建 SpringDemo 项目，并在 src 目录下创建 net.biancheng 包。
导入 Spring 相关 JAR 包及 mysql-connector-java.x.x.x.jar 包。
在 net.biancheng 包下创建 User、UserDao、UserDaoImpl、Beans.xml 和 MainApp。
运行 SpringDemo 项目。

User 类代码如下。
```java
package net.biancheng;
public class User {
    private int id;
    private String name;
    private int age;
    public User() {
    }
    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    
    // 省略set和get方法
}
```
UserDao 代码如下。
```java
package net.biancheng;
import java.util.List;
public interface UserDao {
    /**
     * 初始化User表
     */
    void createUserTable();
    /**
     * 保存用户
     */
    void saveUser(User user);
    /**
     * 查询用户
     */
    List<User> listUser();
}
```
UserDaoImpl 代码如下。
```java
package net.biancheng;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;
import javax.sql.DataSource;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, readOnly = false)
public class UserDaoImpl implements UserDao {
    private JdbcTemplate jdbcTemplate;
    private UserDao userDao;
    public JdbcTemplate getJdbcTemplate() {
        return jdbcTemplate;
    }
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    public UserDao getUserDao() {
        return userDao;
    }
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    public void setDataSource(DataSource datasource) {
        this.jdbcTemplate = new JdbcTemplate(datasource);
    }
    @Override
    public void createUserTable() {
        this.jdbcTemplate.execute("CREATE TABLE `user` (\r\n" + "  `id` int(11) NOT NULL AUTO_INCREMENT,\r\n"
                + "  `name` varchar(50) DEFAULT NULL,\r\n" + "  `age` int(4) DEFAULT NULL,\r\n"
                + "  PRIMARY KEY (`id`)\r\n" + ") ENGINE=InnoDB DEFAULT CHARSET=utf8;");
    }
    @Override
    public void saveUser(User user) {
        try {
            this.jdbcTemplate.update("INSERT INTO USER(NAME,age) VALUES (?,?)", user.getName(), user.getAge());
            this.jdbcTemplate.update("INSERT INTO USER(NAME,age) VALUES (?,?)", "google", 16);
            throw new RuntimeException("simulate Error condition");
        } catch (Exception e) {
            System.out.println("Error in creating record, rolling back");
            throw e;
        }
    }
    @Override
    public List<User> listUser() {
        List<User> users = this.jdbcTemplate.query("SELECT NAME,age FROM USER", new RowMapper<User>() {
            public User mapRow(ResultSet rs, int rowNum) throws SQLException {
                User user = new User();
                user.setName(rs.getString("name"));
                user.setAge(rs.getInt("age"));
                return user;
            }
        });
        return users;
    }
}
```
@Transactional 注解的参数之间用“，”进行分隔

Beans.xml 代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx-3.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
    <!-- 配置数据源 -->
    <bean id="dataSource"
        class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--数据库驱动 -->
        <property name="driverClassName"
            value="com.mysql.jdbc.Driver" />
        <!--连接数据库的url -->
        <property name="url" value="jdbc:mysql://localhost/test" />
        <!--连接数据库的用户名 -->
        <property name="username" value="root" />
        <!--连接数据库的密码 -->
        <property name="password" value="root" />
    </bean>
    <!-- 配置事务管理器 -->
    <bean id="transactionManager"
        class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="jdbcTemplate"
        class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userdao" class="net.biancheng.UserDaoImpl">
        <property name="dataSource" ref="dataSource" />
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean>
    <!-- 注册事务管理驱动 -->
    <tx:annotation-driven
        transaction-manager="transactionManager" />
</beans>
```
MainApp 类代码如下。
```java
package net.biancheng;
import java.util.List;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("Beans.xml");
        UserDao dao = (UserDao) ctx.getBean("userdao");
        dao.createUserTable();
        dao.saveUser(new User("bianchengbang", 12));
        dao.saveUser(new User("baidu", 18));
        List<User> users = dao.listUser();
        for (User user : users) {
            System.out.println("姓名：" + user.getName() + "\t年龄：" + user.getAge());
        }
    }
}
```
运行结果如下。
```
Error in creating record, rolling back
Exception in thread "main" java.lang.RuntimeException: simulate Error condition
...
```