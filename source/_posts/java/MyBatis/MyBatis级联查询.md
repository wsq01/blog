---
title: MyBatis 级联查询
date: 2021-04-15 11:36:09
tags: [MyBatis]
categories: [MyBatis]
---

级联关系是一个数据库实体的概念，有 3 种级联关系，分别是一对一级联、一对多级联以及多对多级联。

级联的优点是获取关联数据十分便捷。但是级联过多会增加系统的复杂度，同时降低系统的性能，此增彼减。所以记录超过 3 层时，就不要考虑使用级联了，因为这样会造成多个对象的关联，导致系统的耦合、负载和难以维护。
# 一对一关联查询
通过`<resultMap>`元素的子元素`<association>`处理这种一对一级联关系。

在`<association>`元素中通常使用以下属性：
* `property`：指定映射到实体类的对象属性。
* `column`：指定表中对应的字段（即查询返回的列名）。
* `javaType`：指定映射到实体对象属性的类型。
* `select`：指定引入嵌套查询的子 SQL 语句，该属性用于关联映射中的嵌套查询。

```xml
<association property="studentCard" column="cardId"
    javaType="net.biancheng.po.StudentCard"
    select="net.biancheng.mapper.StudentCardMapper.selectStuCardById" />
```
一对一关联查询可采用以下两种方式：
* 单步查询，通过关联查询实现
* 分步查询，通过两次或多次查询，为一对一关系的实体`Bean`赋值

## 示例
### 1.创建数据表
创建`student`（学生）和`studentcard`（学号）数据表，SQL 语句如下。
```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) CHARACTER SET utf8 COLLATE utf8_unicode_ci DEFAULT NULL,
  `sex` tinyint(4) DEFAULT NULL,
  `cardId` int(20) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `cardId` (`cardId`),
  CONSTRAINT `student_ibfk_1` FOREIGN KEY (`cardId`) REFERENCES `studentcard` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;
insert  into `student`(`id`,`name`,`sex`,`cardId`) values (1,'张三',0,1),(2,'李四',0,2),(3,'赵小红',1,3),(4,'李晓明',0,4),(5,'李紫薇',1,5),(6,'钱百百',0,NULL);
DROP TABLE IF EXISTS `studentcard`;
CREATE TABLE `studentcard` (
  `id` int(20) NOT NULL AUTO_INCREMENT,
  `studentId` int(20) DEFAULT NULL,
  `startDate` date DEFAULT NULL,
  `endDate` date DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `studentId` (`studentId`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;
insert  into `studentcard`(`id`,`studentId`,`startDate`,`endDate`) values (1,20200311,'2021-03-01','2021-03-11'),(2,20200314,'2021-03-01','2021-03-11'),(3,20200709,'2021-03-01','2021-03-11'),(4,20200508,'2021-03-01','2021-03-11'),(5,20207820,'2021-03-01','2021-03-11');
```
### 2.创建持久化类
在`myBatisDemo`应用的`net.biancheng.po`包下创建数据表对应的持久化类`Student`和`StudentCard`。
```java
package net.biancheng.po;
public class Student {
  private int id;
  private String name;
  private int sex;
  private StudentCard studentCard;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "Student [id=" + id + ", name=" + name + ", sex=" + sex + ", studentCard=" + studentCard + "]";
  }
}
```
```java
package net.biancheng.po;
import java.util.Date;
public class StudentCard {
  private int id;
  private int studentId;
  private Date startDate;
  private Date endDate;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "StudentCard [id=" + id + ", studentId=" + studentId + "]";
  }
}
```
## 分步查询
新建`StudentCardMapper`类。
```java
package net.biancheng.mapper;
import net.biancheng.po.StudentCard;
public interface StudentCardMapper {
  public StudentCard selectStuCardById(int id);
}
```
`StudentCardMapper.xml`对应映射 SQL 语句代码如下。
```xml
<mapper namespace="net.biancheng.mapper.StudentCardMapper">
  <select id="selectStuCardById" resultType="net.biancheng.po.StudentCard">
    SELECT * FROM studentCard WHERE id = #{id}
  </select>
</mapper>
```
`StudentMapper`类方法代码如下。
```java
package net.biancheng.mapper;
import net.biancheng.po.Student;
public interface StudentMapper {
  public Student selectStuById1(int id);
  public Student selectStuById2(int id);
}
```
`StudentMapper.xml`代码如下。
```xml
<mapper namespace="net.biancheng.mapper.StudentMapper">
  <!-- 一对一根据id查询学生信息：级联查询的第一种方法（嵌套查询，执行两个SQL语句） -->
  <resultMap type="net.biancheng.po.Student" id="cardAndStu1">
    <id property="id" column="id" />
    <result property="name" column="name" />
    <result property="sex" column="sex" />
    <!-- 一对一级联查询 -->
    <association property="studentCard" column="cardId"
      javaType="net.biancheng.po.StudentCard"
      select="net.biancheng.mapper.StudentCardMapper.selectStuCardById" />
  </resultMap>
  <select id="selectStuById1" parameterType="Integer" resultMap="cardAndStu1">
    select * from student where id=#{id}
  </select>
</mapper>
```
测试代码如下。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    SqlSession ss = ssf.openSession();
    Student stu = ss.getMapper(StudentMapper.class).selectStuById1(2);
    System.out.println(stu);
  }
}
```
## 单步查询
在`StudentMapper.xml`中添加以下代码。
```xml
<resultMap type="net.biancheng.po.Student" id="cardAndStu2">
  <id property="id" column="id" />
  <result property="name" column="name" />
  <result property="sex" column="sex" />
  <!-- 一对一级联查询 -->
  <association property="studentCard" javaType="net.biancheng.po.StudentCard">
    <id property="id" column="id" />
    <result property="studentId" column="studentId" />
  </association>
</resultMap>
<select id="selectStuById2" parameterType="Integer" resultMap="cardAndStu2">
  SELECT s.*,sc.studentId FROM student s,studentCard sc
  WHERE
  s.cardId = sc.id AND s.id=#{id}
</select>
```
在`StudentMapper`中添加以下方法。
```java
public Student selectStuById2(int id);
```
# 一对多关联查询
但在实际生活中也有许多一对多级联关系，例如一个用户可以有多个订单，而一个订单只属于一个用户。同样，国家和城市也属于一对多级联关系。

通过`<resultMap>`元素的子元素`<collection>`处理一对多级联关系，`collection`可以将关联查询的多条记录映射到一个`list`集合属性中。
```xml
<collection property="orderList"
    ofType="net.biancheng.po.Order" column="id"
    select="net.biancheng.mapper.OrderMapper.selectOrderById" />
```
在`<collection>`元素中通常使用以下属性。
* `property`：指定映射到实体类的对象属性。
* `column`：指定表中对应的字段（即查询返回的列名）。
* `javaType`：指定映射到实体对象属性的类型。
* `select`：指定引入嵌套查询的子 SQL 语句，该属性用于关联映射中的嵌套查询。

一对多关联查询可采用以下两种方式：
* 分步查询，通过两次或多次查询，为一对多关系的实体 Bean 赋值
* 单步查询，通过关联查询实现

## 示例
### 1.创建数据表
两张数据表，一张是用户表`user`，一张是订单表`order`，这两张表具有一对多的级联关系。
```sql
CREATE TABLE `order` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `ordernum` int(25) DEFAULT NULL,
  `userId` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `userId` (`userId`),
  CONSTRAINT `order_ibfk_1` FOREIGN KEY (`userId`) REFERENCES `user` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8;
insert  into `order`(`id`,`ordernum`,`userId`) values (1,20200107,1),(2,20200806,2),(3,20206702,3),(4,20200645,1),(5,20200711,2),(6,20200811,2),(7,20201422,3),(8,20201688,4);
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  `pwd` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;
insert  into `user`(`id`,`name`,`pwd`) values (1,'编程帮','123'),(2,'C语言中文网','456'),(3,'赵小红','123'),(4,'李晓明','345'),(5,'杨小胤','123'),(6,'谷小乐','789');
```
### 2.创建持久化类
创建持久化类`User`和`Order`。
```java
package net.biancheng.po;
import java.util.List;
public class User {
  private int id;
  private String name;
  private String pwd;
  private List<Order> orderList;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "User [id=" + id + ", name=" + name + ", orderList=" + orderList + "]";
  }
}
```
```java
package net.biancheng.po;
public class Order {
  private int id;
  private int ordernum;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "Order [id=" + id + ", ordernum=" + ordernum + "]";
  }
}
```
## 分步查询
`OrderMapper`类代码如下。
```java
public List<Order> selectOrderById(int id);
```
`OrderMapper.xml`中相应的映射 SQL 语句如下。
```xml
<!-- 根据id查询订单信息 -->
<select id="selectOrderById" resultType="net.biancheng.po.Order" parameterType="Integer">
  SELECT * FROM `order` where userId=#{id}
</select>
```
`UserMapper`类代码如下。
```java
public User selectUserOrderById1(int id);
```
`UserMapper.xml`中相应的映射 SQL 语句如下。
```xml
<!-- 一对多 根据id查询用户及其关联的订单信息：级联查询的第一种方法（分步查询） -->
<resultMap type="net.biancheng.po.User" id="userAndOrder1">
  <id property="id" column="id" />
  <result property="name" column="name" />
  <result property="pwd" column="pwd" />
  <!-- 一对多级联查询，ofType表示集合中的元素类型，将id传递给selectOrderById -->
  <collection property="orderList"
    ofType="net.biancheng.po.Order" column="id"
    select="net.biancheng.mapper.OrderMapper.selectOrderById" />
</resultMap>
<select id="selectUserOrderById1" parameterType="Integer" resultMap="userAndOrder1">
  select * from user where id=#{id}
</select>
```
测试代码如下。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    SqlSession ss = ssf.openSession();
    User us = ss.getMapper(UserMapper.class).selectUserOrderById1(1);
    System.out.println(us);
  }
}
```
## 单步查询
该种方式实现一对多关联查询需要修改`Order`持久化类，因为`Order`中的`id`不能和`User`中的`id`重复。
```java
package net.biancheng.po;
public class Order {
  private int oId;
  private int ordernum;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "Order [id=" + oId+ ", ordernum=" + ordernum + "]";
  }
}
```
`UserMapper`类代码如下。
```java
public User selectUserOrderById2(int id);
```
`UserMapper.xml`中相关映射 SQL 语句如下。
```xml
<!-- 一对多 根据id查询用户及其关联的订单信息：级联查询的第二种方法（单步查询） -->
<resultMap type="net.biancheng.po.User" id="userAndOrder2">
  <id property="id" column="id" />
  <result property="name" column="name" />
  <result property="pwd" column="pwd" />
  <!-- 一对多级联查询，ofType表示集合中的元素类型 -->
  <collection property="orderList" ofType="net.biancheng.po.Order">
    <id property="oId" column="oId" />
    <result property="ordernum" column="ordernum" />
  </collection>
</resultMap>
<select id="selectUserOrderById2" parameterType="Integer" resultMap="userAndOrder2">
  SELECT u.*,o.id as oId,o.ordernum FROM `user` u,`order` o
  WHERE
  u.id=o.`userId` AND u.id=#{id}
</select>
```
测试代码修改调用方法，如下。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    SqlSession ss = ssf.openSession();
    User us = ss.getMapper(UserMapper.class).selectUserOrderById2(1);
    System.out.println(us);
  }
}
```
# 多对多关联查询
实际应用中，由于多对多的关系比较复杂，会增加理解和关联的复杂度，所以应用较少。MyBatis 没有实现多对多级联，推荐通过两个一对多级联替换多对多级联，以降低关系的复杂度，简化程序。

例如，一个订单可以有多种商品，一种商品可以对应多个订单，订单与商品就是多对多的级联关系。可以使用一个中间表（订单记录表）将多对多级联转换成两个一对多的关系。
## 示例
下面以订单和商品（实现“查询所有订单以及每个订单对应的商品信息”的功能）为例讲解多对多关联查询。
### 1. 创建数据表
创建`order`（订单），`product`（商品）和`order_details`（订单和商品中间表），SQL 语句如下。
```sql
CREATE TABLE `order` (
  `oid` int(11) NOT NULL AUTO_INCREMENT,
  `ordernum` int(25) DEFAULT NULL,
  `userId` int(11) DEFAULT NULL,
  PRIMARY KEY (`oid`),
  KEY `userId` (`userId`),
  CONSTRAINT `order_ibfk_1` FOREIGN KEY (`userId`) REFERENCES `user` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;
insert  into `order`(`oid`,`ordernum`,`userId`) values (1,20200107,1),(2,20200806,2),(3,20206702,3),(4,20200645,1),(5,20200711,2),(6,20200811,2),(7,20201422,3),(8,20201688,4),(9,NULL,5);
DROP TABLE IF EXISTS `orders_detail`;
CREATE TABLE `orders_detail` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `orderId` int(11) DEFAULT NULL,
  `productId` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;
insert  into `orders_detail`(`id`,`orderId`,`productId`) values (1,1,1),(2,1,2),(3,1,3),(4,2,3),(5,2,1),(6,3,2);
DROP TABLE IF EXISTS `product`;
CREATE TABLE `product` (
  `pid` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(25) DEFAULT NULL,
  `price` double DEFAULT NULL,
  PRIMARY KEY (`pid`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
insert  into `product`(`pid`,`name`,`price`) values (1,'Java教程',128),(2,'C语言教程',138),(3,'Python教程',132.35);
```
### 2. 创建持久化类
`Order`类代码如下。
```java
package net.biancheng.po;
import java.util.List;
public class Order {
  private int oid;
  private int ordernum;
  private List<Product> products;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "Order [id=" + oid + ", ordernum=" + ordernum + ", products=" + products + "]";
  }
}
```
`Product`类方法如下。
```java
package net.biancheng.po;
import java.util.List;
public class Product {
  private int pid;
  private String name;
  private Double price;
  // 多对多中的一个一对多
  private List<Order> orders;
  /*省略setter和getter方法*/
  @Override
  public String toString() {
    return "Product [id=" + pid + ", name=" + name + ", price=" + price + "]";
  }
}
```
### 3. 创建接口和映射文件
`OrderMapper`接口代码如下。
```java
package net.biancheng.mapper;
import java.util.List;
import net.biancheng.po.Order;
public interface OrderMapper {
  public List<Order> selectAllOrdersAndProducts();
}
```
`OrderMapper.xml`代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="net.biancheng.mapper.OrderMapper">
  <resultMap type="net.biancheng.po.Order" id="orderMap">
    <id property="oid" column="oid" />
    <result property="ordernum" column="ordernum" />
    <collection property="products" ofType="net.biancheng.po.Product">
      <id property="pid" column="pid" />
      <result property="name" column="name" />
      <result property="price" column="price" />
    </collection>
  </resultMap>
  <select id="selectAllOrdersAndProducts" parameterType="Integer" resultMap="orderMap">
    SELECT o.oid,o.`ordernum`,p.`pid`,p.`name`,p.`price` FROM
    `order` o
    INNER JOIN orders_detail od ON o.oid=od.`orderId`
    INNER JOIN
    product p
    ON p.pid = od.`productId`
  </select>
</mapper>
```
### 4. 创建测试类
测试类代码如下。
```java
package net.biancheng.test;
import java.io.IOException;
import java.io.InputStream;
import java.util.List;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import net.biancheng.mapper.OrderMapper;
import net.biancheng.po.Order;
public class Test {
  public static void main(String[] args) throws IOException {
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    SqlSession ss = ssf.openSession();
    List<Order> orderList = ss.getMapper(OrderMapper.class).selectAllOrdersAndProducts();
    for (Order or : orderList) {
      System.out.println(or);
    }
  }
}
```