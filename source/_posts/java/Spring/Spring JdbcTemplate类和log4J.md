


# Spring JdbcTemplate类
Spring 针对数据库开发提供了 JdbcTemplate 类，该类封装了 JDBC，支持对数据库的所有操作。

JdbcTemplate 位于 spring-jdbc-x.x.x.jar 包中，其全限定命名为 org.springframework.jdbc.core.JdbcTemplate。此外使用 JdbcTemplate 还需要导入 spring-tx-x.x.x.jar 包，该包用来处理事务和异常。

在 Spring 中，JDBC 的相关信息在配置文件中完成，其配置模板如下所示。
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http:/www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd"> 
   
    <!-- 配置数据源 --> 
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <!--数据库驱动-->
        <property name="driverClassName" value="com.mysql.jdbc.Driver" /> 
        <!--连接数据库的url-->
        <property name= "url" value="jdbc:mysql://localhost/xx" />
        <!--连接数据库的用户名-->
        <property name="username" value="root" />
        <!--连接数据库的密码-->
        <property name="password" value="root" />
    </bean>
    <!--配置JDBC模板-->
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <!--默认必须使用数据源-->
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置注入类-->
    <bean id="xxx" class="xxx">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
    ...
</beans>
本节使用 MySQL 数据库，如果您使用的是其它数据库，需要对内容进行相应的修改。

上述代码中定义了 3 个 Bean，分别是 dataSource、jdbcTemplate 和需要注入类的 Bean。其中 dataSource 对应的是 DriverManagerDataSource 类，用于对数据源进行配置；jdbcTemplate 对应 JdbcTemplate 类，该类中定义了 JdbcTemplate 的相关配置。

在 dataSource 中，定义了 4 个连接数据库的属性，如下表所示。

属性名	说明
driverClassName	所使用的驱动名称，对应驱动 JAR 包中的 Driver 类
url	数据源所在地址
username	访问数据库的用户名
password	访问数据库的密码
上表中的属性值需要根据数据库类型或者机器配置的不同进行相应设置。如果数据库类型不同，则需要更改驱动名称。如果数据库不在本地，则需要将 localhost 替换成相应的主机 IP。

在定义 JdbcTemplate 时，需要将 dataSource 注入到 JdbcTemplate 中。而在其他的类中要使用 JdbcTemplate，也需要将 JdbcTemplate 注入到使用类中（通常注入 dao 类中）。

在 JdbcTemplate 类中，提供了大量的查询和更新数据库的方法，如 query()、update() 等，如下表所示。

方法	说明
public int update(String sql)	用于执行新增、修改、删除等语句
args 表示需要传入到 query 中的参数
public int update(String sql,Object... args)
public void execute(String sql)	可以执行任意 SQL，一般用于执行 DDL 语句
action 表示执行完 SQL 语句后，要调用的函数
public T execute(String sql, PreparedStatementCallback action)
public T query(String sql, ResultSetExtractor rse)	用于执行查询语句
以 ResultSetExtractor 作为参数的 query 方法返回值为 Object，使用查询结果需要对其进行强制转型
以 RowMapper 作为参数的 query 方法返回值为 List
public List query(String sql, RowMapper rse)
示例
下面通过实例演示使用 JdbcTemplate 类操作数据库，步骤如下：
创建 SpringDemo 项目，并在 src 目录下创建 net.biancheng 包。
导入 Spring 相关 JAR 包及 mysql-connector-java.x.x.x.jar 包。
在 net.biancheng 包下创建 User、UserDao、UserDaoImpl、Beans.xml 和 MainApp。
运行 SpringDemo 项目。

User 类代码如下。
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
UserDao 代码如下。
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
UserDaoImpl 代码如下。
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
                + "  `name` varchar(50) DEFAULT NULL,\r\n" + "  `age` int(11) DEFAULT NULL,\r\n"
                + "  PRIMARY KEY (`id`)\r\n" + ") ENGINE=MyISAM DEFAULT CHARSET=utf8;");
    }
    @Override
    public void saveUser(User user) {
        this.jdbcTemplate.update("INSERT INTO USER(NAME,age) VALUES (?,?)", user.getName(), user.getAge());
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
Beans.xml 代码如下。
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context.xsd
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
    <!--配置JDBC模板 -->
    <bean id="jdbcTemplate"
        class="org.springframework.jdbc.core.JdbcTemplate">
        <!--默认必须使用数据源 -->
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="userdao" class="net.biancheng.UserDaoImpl">
        <property name="jdbcTemplate" ref="jdbcTemplate" />
    </bean>
</beans>
MainApp 类代码如下。
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
运行结果如下。
姓名：bianchengbang 年龄：12
姓名：baidu 年龄：18

# Spring集成Log4J
日志是应用软件中不可缺少的部分，Apache 的开源项目 Log4J 是一个功能强大的日志组件。在 Spring 中使用 Log4J 是非常容易的，下面通过例子演示 Log4J 和 Spring 的集成。

使用 Log4J 之前，需要先导入 log4j-x.y.z.jar 包，Log4J 下载地址：https://logging.apache.org/log4j。
示例
下面使用 Eclipse IDE 演示 Log4J 的使用，步骤如下：
创建 SpringDemo 项目，并在 src 目录下创建 net.biancheng 包。
导入 Spring 相关 JAR 包及 log4j-x.y.z.jar。
在 net.biancheng 包下创建 HelloWorld、MainApp、Beans.xml 和 log4j.properties。
运行 SpringDemo 项目。

HelloWorld 类代码如下。
package net.biancheng;
public class HelloWorld {
    private String message;
    public void setMessage(String message) {
        this.message = message;
    }
    public void getMessage() {
        System.out.println("消息：" + message);
    }
}
MainApp 类代码如下。
package net.biancheng;
import org.apache.log4j.Logger;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
    static Logger log = Logger.getLogger(MainApp.class.getName());
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
        log.info("Going to create HelloWord Obj");
        HelloWorld obj = (HelloWorld) context.getBean("helloWorld");
        obj.getMessage();
        log.info("Exiting the program");
    }
}
Beans.xml 代码如下。
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
    <bean id="helloWorld" class="net.biancheng.HelloWorld">
        <property name="message" value="Hello,bianchengbang!" />
    </bean>
</beans>
log4j.properties 内容如下。
# Define the root logger with appender file
log4j.rootLogger = DEBUG, FILE

# Define the file appender
log4j.appender.FILE=org.apache.log4j.FileAppender
# Set the name of the file
log4j.appender.FILE.File=D:\\log.out

# Set the immediate flush to true (default)
log4j.appender.FILE.ImmediateFlush=true

# Set the threshold to debug mode
log4j.appender.FILE.Threshold=debug

# Set the append to false, overwrite
log4j.appender.FILE.Append=false

# Define the layout for file appender
log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
log4j.appender.FILE.layout.conversionPattern=%m%n

运行结果如下。
消息：Hello,bianchengbang!

log.out 文件内容如下。
Going to create HelloWord Obj
Exiting the program
