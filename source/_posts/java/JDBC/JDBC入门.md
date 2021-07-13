---
title: JDBC 入门
date: 2021-03-03 18:01:52
tags: [java]
categories: java
---


`JDBC`代表Java数据库连接(`Java Database Connectivity`)，`JDBC`是用于在 Java 语言编程中与数据库连接的API。

`JDBC`库中所包含的API通常使用于：
* 连接到数据库
* 创建`SQL`或 MySQL 语句
* 在数据库中执行`SQL`或 MySQL 查询
* 查看和修改数据库中的数据记录

# 常见的JDBC组件
JDBC API提供以下接口和类：
## DriverManager
此类管理数据库驱动程序列表。 使用通信子协议将来自 Java 应用程序的连接请求与适当的数据库驱动程序进行匹配。在`JDBC`下识别某个子协议的第一个驱动程序将用于建立数据库连接。
## Driver
此接口处理与数据库服务器的通信。`Driver`接口由数据库厂家提供，使用`DriverManager`对象来管理这种类型的对象。
## Connection
此接口具有用于联系数据库的所有方法。 连接(Connection)对象表示通信上下文，即，与数据库的所有通信仅通过连接对象。
## Statement
使用从此接口创建的对象将SQL语句提交到数据库。 除了执行存储过程之外，一些派生接口还接受参数。
## ResultSet
在使用Statement对象执行SQL查询后，这些对象保存从数据库检索的数据。 它作为一个迭代器并可移动ResultSet对象查询的数据。
## SQLException
此类处理数据库应用程序中发生的任何错误。

# 创建JDBC应用程序
建立一个`JDBC`应用程序，以 Java 连接 MySQL 为一个示例，分六个步骤进行：
## 1. 导入包
在程序中导入数据库编程所需的`JDBC`类。大多数情况下，使用`import java.sql.*`就足够了。
```java
//STEP 1. Import required packages
import java.sql.*;
```
## 2. 注册JDBC驱动程序
需要初始化驱动程序，这样就可以打开与数据库的通信。
```java
//STEP 2: Register JDBC driver
Class.forName("com.mysql.jdbc.Driver");
```
## 3. 打开一个连接
使用`DriverManager.getConnection()`方法来创建一个`Connection`对象，它代表一个数据库的物理连接，如下所示：
```java
//STEP 3: Open a connection
//  Database credentials
static final String USER = "root";
static final String PASS = "pwd123456";
System.out.println("Connecting to database...");
Connection conn = DriverManager.getConnection(DB_URL,USER,PASS);
```
## 4. 执行一个查询
需要使用一个类型为Statement或PreparedStatement的对象，并提交一个SQL语句到数据库执行查询。如下：
```java
//STEP 4: Execute a query
System.out.println("Creating statement...");
stmt = conn.createStatement();
String sql = "SELECT id, first, last, age FROM Employees";
ResultSet rs = stmt.executeQuery(sql);
```
如果要执行一个SQL语句：UPDATE，INSERT或DELETE语句，那么需要下面的代码片段：
```java
//STEP 4: Execute a query
System.out.println("Creating statement...");
stmt = conn.createStatement();
String sql = "DELETE FROM Employees";
ResultSet rs = stmt.executeUpdate(sql);
```
## 5. 从结果集中提取数据
从数据库中获取查询结果的数据。可以使用适当的ResultSet.getXXX()方法来检索的数据结果如下：
```java
//STEP 5: Extract data from result set
while(rs.next()){
    //Retrieve by column name
    int id  = rs.getInt("id");
    int age = rs.getInt("age");
    String first = rs.getString("first");
    String last = rs.getString("last");

    //Display values
    System.out.print("ID: " + id);
    System.out.print(", Age: " + age);
    System.out.print(", First: " + first);
    System.out.println(", Last: " + last);
}
```
6. 清理环境资源在使用JDBC与数据交互操作数据库中的数据后，应该明确地关闭所有的数据库资源以减少资源的浪费，对依赖于JVM的垃圾收集如下：
```java
//STEP 6: Clean-up environment
rs.close();
stmt.close();
conn.close();
```
# 实例
```java
//STEP 1. Import required packages
import java.sql.*;

public class FirstExample {
   // JDBC driver name and database URL
   static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
   static final String DB_URL = "jdbc:mysql://localhost/test";

   //  Database credentials -- 数据库名和密码自己修改
   static final String USER = "username";
   static final String PASS = "password";

   public static void main(String[] args) {
   Connection conn = null;
   Statement stmt = null;
   try{
      //STEP 2: Register JDBC driver
      Class.forName("com.mysql.jdbc.Driver");

      //STEP 3: Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);

      //STEP 4: Execute a query
      System.out.println("Creating statement...");
      stmt = conn.createStatement();
      String sql;
      sql = "SELECT id, first, last, age FROM Employees";
      ResultSet rs = stmt.executeQuery(sql);

      //STEP 5: Extract data from result set
      while(rs.next()){
         //Retrieve by column name
         int id  = rs.getInt("id");
         int age = rs.getInt("age");
         String first = rs.getString("first");
         String last = rs.getString("last");

         //Display values
         System.out.print("ID: " + id);
         System.out.print(", Age: " + age);
         System.out.print(", First: " + first);
         System.out.println(", Last: " + last);
      }
      //STEP 6: Clean-up environment
      rs.close();
      stmt.close();
      conn.close();
   }catch(SQLException se){
      //Handle errors for JDBC
      se.printStackTrace();
   }catch(Exception e){
      //Handle errors for Class.forName
      e.printStackTrace();
   }finally{
      //finally block used to close resources
      try{
         if(stmt!=null)
            stmt.close();
      }catch(SQLException se2){
      }// nothing we can do
      try{
         if(conn!=null)
            conn.close();
      }catch(SQLException se){
         se.printStackTrace();
      }//end finally try
   }//end try
   System.out.println("Goodbye!");
}//end main
}//end FirstExample
```
# JDBC批量处理
批处理允许执行一个批处理组相关的 SQL 语句，并将其一次提交到数据库中执行。当几个 SQL 语句一次发送到数据库中时，可以减少通信开销，从而提高性能。

JDBC 驱动程序不支持此功能。应该使用`DatabaseMetaData.supportsBatchUpdates()`方法来确定目标数据库支持批量更新处理。如果 JDBC 驱动程序支持此功能，则该方法返回`true`。`addBatch()`方法是`PreparedStatement`和`CallableStatementis`类中用于添加单个语句的批处理的声明。`executeBatch()`将开始将所有语句组合到一起并执行。

`executeBatch()`将返回一个整数数组，每个数组元素的表示为相应的更新语句的更新计数。

添加语句进行批处理时，可以使用`clearBatch()`方法删除它们。此方法将删除`addBatch()`方法添加的所有语句。但是不能有选择性地选择某个语句来删除。

JDBC 数据流`PreparedStatement`对象有能力使用提供参数数据的输入和输出流。这使您可以将整个文件到数据库中，可容纳较大的值，如`CLOB`和`BLOB`数据类型的列。

有下列方法可用于流数据：
* `setAsciiStream()`: 此方法用于提供大的 ASCII 数据值。
* `setCharacterStream()`: 此方法用于提供大的 UNICODE 数据值。
* `setBinaryStream()`: 使用此方法用于提供大的二进制数据值。

`setXXXStream()`方法需要一个额外的参数，文件大小(除了参数占位符)。此参数通知应发送多少数据到数据库来使用流的驱动程序。




