---
title: JDBC 入门
date: 2021-03-03 18:01:52
tags: [JDBC]
categories: JDBC
---


`JDBC`代表 Java 数据库连接(`Java Database Connectivity`)，`JDBC`是用于在 Java 语言编程中与数据库连接的 API。

`JDBC`库中所包含的 API 通常使用于：
* 连接到数据库
* 创建`SQL`或 MySQL 语句
* 在数据库中执行`SQL`或 MySQL 查询
* 查看和修改数据库中的数据记录

# 常见的JDBC组件
JDBC API提供以下接口和类：
1. `DriverManager`
此类管理数据库驱动程序列表。 使用通信子协议将来自 Java 应用程序的连接请求与适当的数据库驱动程序进行匹配。在`JDBC`下识别某个子协议的第一个驱动程序将用于建立数据库连接。
2. `Driver`
此接口处理与数据库服务器的通信。`Driver`接口由数据库厂家提供，使用`DriverManager`对象来管理这种类型的对象。
3. `Connection`
此接口具有用于联系数据库的所有方法。 连接(`Connection`)对象表示通信上下文，即，与数据库的所有通信仅通过连接对象。
4. `Statement`
使用从此接口创建的对象将SQL语句提交到数据库。 除了执行存储过程之外，一些派生接口还接受参数。
5. `ResultSet`
在使用`Statement`对象执行SQL查询后，这些对象保存从数据库检索的数据。 它作为一个迭代器并可移动`ResultSet`对象查询的数据。
6. `SQLException`
此类处理数据库应用程序中发生的任何错误。

# 创建JDBC应用程序
建立一个`JDBC`应用程序，分六个步骤进行：
## 1. 导入包
在程序中导入编程所需的 JDBC 类。大多数情况下，使用`import java.sql.*`就足够了。
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
使用`DriverManager.getConnection()`方法来创建一个`Connection`对象，它代表一个数据库的物理连接：
```java
//STEP 3: Open a connection
//  Database credentials
static final String USER = "root";
static final String PASS = "pwd123456";
System.out.println("Connecting to database...");
Connection conn = DriverManager.getConnection(DB_URL,USER,PASS);
```
## 4. 执行一个查询
需要使用一个类型为`Statement`或`PreparedStatement`的对象，并提交一个 SQL 语句到数据库执行查询。
```java
//STEP 4: Execute a query
System.out.println("Creating statement...");
stmt = conn.createStatement();
String sql = "SELECT id, first, last, age FROM Employees";
ResultSet rs = stmt.executeQuery(sql);
```
如果要执行一个 SQL 语句：`UPDATE，INSERT`或`DELETE`语句，需要用`executeUpdate`：
```java
//STEP 4: Execute a query
System.out.println("Creating statement...");
stmt = conn.createStatement();
String sql = "DELETE FROM Employees";
ResultSet rs = stmt.executeUpdate(sql);
```
## 5. 从结果集中提取数据
从数据库中获取查询结果的数据。可以使用适当的`ResultSet.getXXX()`方法来检索的数据：
```java
//STEP 5: Extract data from result set
while(rs.next()) {
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
## 6. 释放资源
清理环境资源在使用 JDBC 与数据交互操作数据库中的数据后，应该明确地关闭所有的数据库资源以减少资源的浪费，对依赖于 JVM 的垃圾收集：
```java
//STEP 6: Clean-up environment
rs.close();
stmt.close();
conn.close();
```
## 实例
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
    try {
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
    } catch(SQLException se) {
      //Handle errors for JDBC
      se.printStackTrace();
    } catch(Exception e){
      //Handle errors for Class.forName
      e.printStackTrace();
    } finally{
      //finally block used to close resources
      try {
        if(stmt!=null)
          stmt.close();
      } catch(SQLException se2){
      }// nothing we can do
      try{
        if(conn!=null)
          conn.close();
      } catch(SQLException se){
        se.printStackTrace();
      }//end finally try
    }//end try
    System.out.println("Goodbye!");
  }//end main
}//end FirstExample
```
# JDBC 数据类型
JDBC驱动程序将 Java 数据类型转换为适当的JDBC类型，然后将其发送到数据库。它为大多数数据类型提供并使用默认映射。例如，Java `int`类型会被转换为 SQL `INTEGER`。 创建默认映射以提供到驱动程序时保持一致性。

下表总结了当调用`PreparedStatement`或`CallableStatement`对象或`ResultSet.updateXXX()`方法的`setXXX()`方法时，将 Java 数据类型转换为的默认 JDBC 数据类型。

| SQL类型 | JDBC/Java类型 | setXXX | updateXXX | getXXX |
| :--: | :--: | :--: | :--: | :--: |
| VARCHAR | java.lang.String | setString | updateString | getString |
| CHAR | java.lang.String | setString | updateString | getString |
| LONGVARCHAR java.lang.String | setString | updateString | getString |
| BIT | boolean | setBoolean | updateBoolean | getBoolean |
| NUMERIC | java.math.BigDecimal | setBigDecimal | updateBigDecimal | getBigDecimal |
| TINYINT | byte | setByte | updateByte | getByte |
| SMALLINT short | setShort | updateShort | getShort |
| INTEGER | int | setInt | updateInt | getInt |
| BIGINT | long | setLong | updateLong | getLong |
| REAL | float | setFloat | updateFloat | getFloat |
| FLOAT | float | setFloat | updateFloat | getFloat |
| DOUBLE | double | setDouble | updateDouble | getDouble |
| VARBINARY | byte[] | setBytes | updateBytes | getBytes |
| BINARY | byte[] | setBytes | updateBytes | getBytes |
| DATE | java.sql.Date | setDate | updateDate | getDate |
| TIME | java.sql.Time | setTime | updateTime | getTime |
| TIMESTAMP | java.sql.Timestamp | setTimestamp | updateTimestamp | getTimestamp |
| CLOB | java.sql.Clob | setClob | updateClob | getClob |
| BLOB | java.sql.Blob | setBlob | updateBlob | getBlob |
| ARRAY | java.sql.Array | setArray | updateArray| getArray |
| REF | java.sql.Ref | SetRef | updateRef | getRef |
| STRUCT | java.sql.Struct | SetStruct | updateStruct | getStruct |

## 日期和时间数据类型
`java.sql.Date`类映射到SQL `DATE`类型，`java.sql.Time`和`java.sql.Timestamp`类分别映射到SQL `TIME`和SQL `TIMESTAMP`数据类型。

以下示例显示了`Date`和`Time`类如何格式化为标准 Java 日期和时间值以匹配 SQL 数据类型要求。
```java
import java.sql.Date;
import java.sql.Time;
import java.sql.Timestamp;
import java.util.*;

public class SqlDateTime {
  public static void main(String[] args) {
    //Get standard date and time
    java.util.Date javaDate = new java.util.Date();
    long javaTime = javaDate.getTime();
    System.out.println("The Java Date is:" + javaDate.toString());

    //Get and display SQL DATE
    java.sql.Date sqlDate = new java.sql.Date(javaTime);
    System.out.println("The SQL DATE is: " + sqlDate.toString());

    //Get and display SQL TIME
    java.sql.Time sqlTime = new java.sql.Time(javaTime);
    System.out.println("The SQL TIME is: " + sqlTime.toString());
    //Get and display SQL TIMESTAMP
    java.sql.Timestamp sqlTimestamp =
    new java.sql.Timestamp(javaTime);
    System.out.println("The SQL TIMESTAMP is: " + sqlTimestamp.toString());
  }//end main
}//end SqlDateTime
```
## 处理NULL值
SQL使用`NULL`值和 Java 使用`null`是不同的概念。 所以，要在 Java 中处理SQL `NULL`值，可以使用三种策略：
* 避免使用返回原始数据类型的`getXXX()`方法。
* 对原始数据类型使用包装类，并使用`ResultSet`对象的`wasNull()`方法来测试接收`getXXX()`方法的返回值的包装器类变量是否应设置为`null`。
* 使用原始数据类型和`ResultSet`对象的`wasNull()`方法来测试接收到由`getXXX()`方法返回的值的原始变量是否应设置为表示`NULL`的可接受值。

```java
Statement stmt = conn.createStatement( );
String sql = "SELECT id, first, last, age FROM Employees";
ResultSet rs = stmt.executeQuery(sql);

int id = rs.getInt(1);
if( rs.wasNull( ) ) {
  id = 0;
}
```



