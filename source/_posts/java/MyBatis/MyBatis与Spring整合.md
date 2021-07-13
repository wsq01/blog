




# 导入相关JAR包
实现 MyBatis 与 Spring 的整合需要导入相关 JAR 包，包括 MyBatis、Spring 以及其他 JAR 包。
1）MyBatis 框架所需的 JAR 包
MyBatis 框架所需的 JAR 包包括它的核心包和依赖包。
2）Spring 框架所需的 JAR 包
Spring 框架所需的 JAR 包包括它的核心模块 JAR、AOP 开发使用的 JAR、JDBC 和事务的 JAR 包（其中依赖包不需要再导入，因为 MyBatis 已提供），具体如下：
aopalliance-1.0.jar
aspectjweaver-1.6.9.jar
spring-aop-3.2.13.RELEASE.jar
spring-aspects-3.2.13.RELEASE.jar
spring-beans-3.2.13.RELEASE.jar
spring-context-3.2.13.RELEASE.jar
spring-core-3.2.13.RELEASE.jar
spring-expression-3.2.13.RELEASE.jar
spring-jdbc-3.2.13.RELEASE.jar
spring-tx-3.2.13.RELEASE.jar
3）MyBatis 与 Spring 整合的中间 JAR 包
该中间 JAR 包的版本为 mybatis-spring-1.3.1.jar。
4）数据库驱动 JAR 包
MySQL 数据库驱动包为 mysql-connector-java-5.1.25-bin.jar。
5）数据源所需的 JAR 包
在整合时使用的是 DBCP 数据源，需要准备 DBCP 和连接池的 JAR 包。

DBCP 的 JAR 包为 commons-dbcp2-2.2.0.jar。

连接池的 JAR 包为 commons-pool2-2.5.0.jar。
# 在Spring中配置MyBatis工厂
通过与 Spring 的整合，MyBatis 的 SessionFactory 交由 Spring 来构建，在构建时需要在 Spring 的配置文件中添加如下代码：
```xml
<!--配置数据源-->
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://127.0.0.1:3306/springtest?seUnicode=true&amp;characterEncoding=utf-8" />
    <property name="username" value="root" />
    <property name="password" value="1128" />
    <!-- 最大连接数 -->
    <property name="maxTotal" value="30"/>
    <!-- 最大空闲连接数 -->
    <property name="maxIdle" value="10"/>
    <!-- 初始化连接数 -->
    <property name="initialSize" value="5"/>
</bean>
<!-- 配置SqlSessionFactoryBean -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 引用数据源组件 -->
    <property name="dataSource" ref="dataSource" />
    <!-- 引用MyBatis配置文件中的配置 -->
    <property name="configLocation" value="classpath:mybatis-config.xml" />
</bean>
```
使用 Spring 管理 MyBatis 的数据操作接口
使用 Spring 管理 MyBatis 数据操作接口的方式有多种，其中最常用、最简洁的一种是基于 MapperScannerConfigurer 的整合。该方式需要在 Spring 的配置文件中加入以下内容：
```xml
<!-- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配 （Sprinh将指定包中的所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口） -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <!-- mybatis-spring组件的扫描器，com.dao只需要接口（接口方法与SQL映射文件中的相同） -->
  <property name="basePackage" value="com.dao" />
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
</bean>
```
# 实例
### 1.创建应用并导入相关 JAR 包
创建一个名为 MyBatis-Spring 的 Web 应用，并将《MyBatis与Spring的整合步骤》教程的 JAR 导入 /WEB-INF/lib 目录下。
### 2.创建持久化类
在 src 目录下创建一个名为 com.po 的包，将《第一个MyBatis程序》教程的持久化类复制到包中。
### 3.创建 SQL 映射文件和 MyBatis 核心配置文件
在 src 目录下创建一个名为 com.mybatis 的包，在该包中创建 MyBatis 核心配置文件 mybatis-config.xml 和 SQL 映射文件 UserMapper.xml。

UserMapper.xml 的代码如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dao.UserDao">
    <!--根据uid查询一个用户信息 -->
    <select id="selectUserById" parameterType="Integer" resultType="com..po.MyUser">
        select * from user where uid = #{uid}
    </select>
    <!-- 查询所有用户信息 -->
    <select id="selectAllUser" resultType="com.po.MyUser">
        select * from user
    </select>
    <!-- 添加一个用户，#{uname}为 com.mybatis.po.MyUser 的属性值 -->
    <insert id="addUser" parameterType="com.po.MyUser">
    insert into user (uname,usex) values(#{uname},#{usex})
    </insert>
    <!--修改一个用户 -->
    <update id="updateUser" parameterType="com..po.MyUser">
        update user set uname = #{uname},usex = #{usex} where uid = #{uid}
    </update>
    <!-- 删除一个用户 -->
    <delete id="deleteUser" parameterType="Integer">
        delete from user where uid = #{uid}
    </delete>
</mapper>
```
`mybatis-config.xml`的代码如下：
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 告诉MyBatis到哪里去找映射文件 -->
  <mappers>
    <mapper resource="com/mybatis/UserMapper.xml" />
  </mappers>
</configuration>
```
### 创建数据访问接口
在`src`目录下创建一个名为`com.dao`的包，在该包中创建`UserDao`接口，并将接口使用`@Mapper`注解为`Mapper`，接口中的方法与 SQL 映射文件一致。
```java
package com.dao;
import java.util.List;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;
import com.po.MyUser;
@Repository("userDao")
@Mapper
/*
* 使用Spring自动扫描MyBatis的接口并装配 （Spring将指定包中所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口
*/
public interface UserDao {
  /**
    * 接口方法对应的SQL映射文件UserMapper.xml中的id
    */
  public MyUser selectUserById(Integer uid);
  public List<MyUser> selectAllUser();
  public int addUser(MyUser user);
  public int updateUser(MyUser user);
  public int deleteUser(Integer uid);
}
```
### 创建日志文件
在`src`目录下创建日志文件`log4j.properties`：
```yml
# Global logging configuration
log4j.rootLogger=ERROR,stdout
# MyBatis logging configuration...
log4j.logger.com.mybatis=DEBUG
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```
### 6.创建控制层
在`src`目录下创建一个名为`com.controller`的包，在包中创建`UserController`类，在该类中调用数据访问接口中的方法。
```java
package com.controller;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import com.dao.UserDao;
import com.po.MyUser;
@Controller("userController")
public class UserController {
  @Autowired
  private UserDao userDao;
  public void test() {
    // 查询一个用户
    MyUser auser = userDao.selectUserById(1);
    System.out.println(auser);
    System.out.println("============================");
    // 添加一个用户
    MyUser addmu = new MyUser();
    addmu.setUname("陈恒");
    addmu.setUsex("男");
    int add = userDao.addUser(addmu);
    System.out.println("添加了" + add + "条记录");
    System.out.println("============================");
    // 修改一个用户
    MyUser updatemu = new MyUser();
    updatemu.setUid(1);
    updatemu.setUname("张三");
    updatemu.setUsex("女");
    int up = userDao.updateUser(updatemu);
    System.out.println("修改了" + up + "条记录");
    System.out.println("============================");
    // 删除一个用户
    int dl = userDao.deleteUser(9);
    System.out.println("删除了" + dl + "条记录");
    System.out.println("============================");
    // 查询所有用户
    List<MyUser> list = userDao.selectAllUser();
    for (MyUser myUser : list) {
      System.out.println(myUser);
    }
  }
}
```
### 7.创建 Spring 的配置文件
在`src`目录下创建配置文件`applicationContext.xml`，在配置文件中配置数据源、MyBatis 工厂以及 Mapper 代理开发等信息。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd
          http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-2.5.xsd">
  <!-- 指定需要扫描的包（包括子包），使注解生效 -->
  <context:component-scan base-package="com.dao" />
  <context:component-scan base-package="com.controller" />
  <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver" />
    <property name="url" value="jdbc:mysql://127.0.0.1:3306/smbms?useUnicode=true&amp;characterEncoding=utf-8" />
    <property name="username" value="root" />
    <property name="password" value="1128" />
    <!-- 最大连接数 -->
    <property name="maxTotal" value="30" />
    <!-- 最大空闲连接数 -->
    <property name="maxIdle" value="10" />
    <!-- 初始化连接数 -->
    <property name="initialSize" value="5" />
  </bean>
  <!-- 添加事务支持 -->
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
  </bean>
  <!-- 注册事务管理驱动 -->
  <tx:annotation-driven transaction-manager="txManager" />
  <!-- 配置SqlSessionFactoryBean -->
  <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 引用数据源组件 -->
    <property name="dataSource" ref="dataSource" />
    <!-- 引用MyBatis配置文件中的配置 -->
    <property name="configLocation" value="classpath:com/mybatis/mybatis-config.xml" />
  </bean>
  <!-- Mapper代理开发，使用Spring自动扫描MyBatis的接口并装配 （Sprinh将指定包中的所有被@Mapper注解标注的接口自动装配为MyBatis的映射接口） -->
  <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- mybatis-spring组件的扫描器，com.dao只需要接口（接口方法与SQL映射文件中的相同） -->
    <property name="basePackage" value="com.dao" />
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
  </bean>
</beans>
```
### 8.创建测试类
在`com.controller`包中创建测试类`TestController`：
```java
package com.controller;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class TestController {
  public static void main(String[] args) {
    String xmlPath = "applicationContext.xml";
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
    UserController uc = (UserController) applicationContext.getBean("userController");
    uc.test();
  }
}
```