---
title: JDBC Statements
date: 2021-03-06 17:24:42
tags: [JDBC]
categories: JDBC
---



当获得了与数据库的连接后，就可以与数据库进行交互了。JDBC `Statement`，`CallableStatement`和`PreparedStatement`接口定义了可用于发送 SQL 的命令，并从数据库接收数据的方法和属性。还定义了有助于在 Java 和 SQL 数据类型的数据类型差异转换的方法。

| 接口 | 推荐使用 |
| :--: | :--: |
| Statement | 用于对数据库进行通用访问，在运行时使用静态SQL语句时很有用。Statement接口不能接受参数 |
| PreparedStatement | 当计划要多次使用SQL语句时使用。<br>PreparedStatement接口在运行时接受输入参数 |
| CallableStatement | 当想要访问数据库存储过程时使用。<br>CallableStatement接口也可以接受运行时输入参数 |

# Statement对象
## 1. 创建Statement对象
在使用`Statement`对象执行 SQL 语句之前，需要使用`Connection`对象的`createStatement()`方法创建一个`Statement`对象。
```java
Statement stmt = null;
try {
  stmt = conn.createStatement( );
  . . .
}
catch (SQLException e) {
  . . .
}
```
在创建`Statement`对象后，可以使用它来执行一个 SQL 语句，它有三个执行方法可以执行：
* `boolean execute (String SQL)`： 如果可以检索到`ResultSet`对象，则返回一个布尔值`true`; 否则返回`false`。使用此方法执行 SQL DDL 语句或需要使用真正的动态 SQL，可使用于执行创建数据库，创建表的 SQL 语句等等。
* `int executeUpdate (String SQL)`：返回受 SQL 语句执行影响的行数。使用此方法执行预期会影响多行的 SQL 语句，例如:`INSERT，UPDATE`或`DELETE`语句。
* `ResultSet executeQuery(String SQL)`：返回一个`ResultSet`对象。 当您希望获得结果集时，请使用此方法，就像使用`SELECT`语句一样。

## 2. 关闭Statement对象
就像关闭一个`Connection`对象一样，以保存数据库资源一样，由于同样的原因，还应该关闭`Statement`对象。

一个简单的调用`close()`方法将执行该工作。如果先关闭`Connection`对象，它也会关闭`Statement`对象。但是，应该始终显式关闭`Statement`对象，以确保正确的清理顺序。
```java
Statement stmt = null;
try {
  stmt = conn.createStatement();
  . . .
}
catch (SQLException e) {
  . . .
} finally {
  stmt.close();
}
```
# PreparedStatement对象
`PreparedStatement`接口扩展了`Statement`接口，它添加了比`Statement`对象更好一些优点的功能。此语句可以动态地提供/接受参数。
## 1 创建PreparedStatement对象
```java
PreparedStatement pstmt = null;
try {
  String SQL = "Update Employees SET age = ? WHERE id = ?";
  pstmt = conn.prepareStatement(SQL);
  pstmt.setInt(1, 16);
  pstmt.setInt(2, 3);
  int rows = stmt.executeUpdate();
  . . .
} catch (SQLException e) {
  . . .
}
```
JDBC 中的所有参数都由`?`符号作为占位符，这被称为参数标记。在执行SQL语句之前，必须为每个参数(占位符)提供值。

`setXXX()`方法将值绑定到参数，其中`XXX`表示要绑定到输入参数的值的 Java 数据类型。如果忘记提供绑定值，则将会抛出一个`SQLException`。

每个参数标记是它其顺序位置引用。第一个标记表示位置1，下一个位置2等等。

所有`Statement`对象与数据库交互的方法`execute()`，`executeQuery()`和`executeUpdate()`也可以用于`PreparedStatement`对象。 但是，这些方法被修改为可以使用输入参数的 SQL 语句。
## 2. 关闭PreparedStatement对象
就像关闭`Statement`对象一样，由于同样的原因，也应该关闭`PreparedStatement`对象。

简单的调用`close()`方法将执行关闭。如果先关闭`Connection`对象，它也会关闭`PreparedStatement`对象。但是，应该始终显式关闭`PreparedStatement`对象，以确保以正确顺序清理资源。
```java
PreparedStatement pstmt = null;
try {
  String SQL = "Update Employees SET age = ? WHERE id = ?";
  pstmt = conn.prepareStatement(SQL);
  . . .
} catch (SQLException e) {
  . . .
} finally {
  pstmt.close();
}
```
# CallableStatement对象
类似`Connection`对象创建`Statement`和`PreparedStatement`对象一样，它还可以使用同样的方式创建`CallableStatement`对象，该对象将用于执行对数据库存储过程的调用。
## 1. 创建CallableStatement对象
假设需要执行以下 MySQL 存储过程
```sql
DELIMITER $$

DROP PROCEDURE IF EXISTS `EMP`.`getEmpName` $$
CREATE PROCEDURE `EMP`.`getEmpName` 
  (IN EMP_ID INT, OUT EMP_FIRST VARCHAR(255))
BEGIN
  SELECT first INTO EMP_FIRST
  FROM Employees
  WHERE ID = EMP_ID;
END $$

DELIMITER ;
```
存在三种类型的参数：`IN，OUT`和`INOUT`。`PreparedStatement`对象只使用`IN`参数。`CallableStatement`对象可以使用上面三个参数类型。

| 参数 | 描述 |
| :--: | :--: |
| IN | 创建SQL语句时其参数值是未知的。 使用 setXXX() 方法将值绑定到IN参数 |
| OUT | 由SQL语句返回的参数值。可以使用 getXXX() 方法从OUT参数中检索值 |
| INOUT | 提供输入和输出值的参数。使用 setXXX() 方法绑定变量并使用 getXXX() 方法检索值 |

以下代码片段显示了如何使用`Connection.prepareCall()`方法根据上述存储过程来实例化一个`CallableStatement`对象。
```java
CallableStatement cstmt = null;
try {
  String strSQL = "{call getEmpName (?, ?)}";
  cstmt = conn.prepareCall (SQL);
  . . .
} catch (SQLException e) {
  . . .
}
```
`String`变量`strSQL`表示存储过程，带有两个参数占位符。

使用`CallableStatement`对象就像使用`PreparedStatement`对象一样。 在执行语句之前，必须将值绑定到所有参数，否则将抛出一个`SQLException`异常。

如果有`IN`参数，只需遵循适用于`PreparedStatement`对象的相同规则和技术; 使用与绑定的 Java 数据类型相对应的`setXXX()`方法。

使用`OUT`和`INOUT`参数时，必须使用一个额外的`CallableStatement`对象方法`registerOutParameter()`。`registerOutParameter()`方法将 JDBC 数据类型绑定到存储过程并返回预期数据类型。

当调用存储过程，可以使用适当的`getXXX()`方法从`OUT`参数中检索该值。此方法将检索到的 SQL 类型的值转换为对应的 Java 数据类型。
## 2. 关闭CallableStatement对象
就像关闭其他`Statement`对象一样，由于同样的原因，还应该关闭`CallableStatement`对象。

简单的调用`close()`方法将执行关闭`CallableStatement`对象。 如果先关闭`Connection`对象，它也会关闭`CallableStatement`对象。但是，应该始终显式关闭`CallableStatement`对象，以确保按正确顺序的清理资源。
```java
CallableStatement cstmt = null;
try {
  String SQL = "{call getEmpName (?, ?)}";
  cstmt = conn.prepareCall (SQL);
  . . .
} catch (SQLException e) {
  . . .
} finally {
  cstmt.close();
}
```
# 示例
## Statements示例
```java
//STEP 1. Import required packages
import java.sql.*;

public class JDBCStatment {
   // JDBC driver name and database URL
   static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
   static final String DB_URL = "jdbc:mysql://localhost/EMP";

   //  Database credentials
   static final String USER = "root";
   static final String PASS = "123456";

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
      String sql = "UPDATE Employees set age=30 WHERE id=103";

      // Let us check if it returns a true Result Set or not.
      Boolean ret = stmt.execute(sql);
      System.out.println("Return value is : " + ret.toString() );

      // Let us update age of the record with ID = 103;
      int rows = stmt.executeUpdate(sql);
      System.out.println("Rows impacted : " + rows );

      // Let us select all the records and display them.
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
   System.out.println("There are something wrong!");
}//end main
}//end JDBCStatment
```
## PreparedStatement示例
```java
//STEP 1. Import required packages
// detail at http://www.yiibai.com/jdbc/

import java.sql.*;

public class JDBCPreparedStatement {
   // JDBC driver name and database URL
   static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
   static final String DB_URL = "jdbc:mysql://localhost/EMP";

   //  Database credentials
   static final String USER = "root";
   static final String PASS = "123456";

   public static void main(String[] args) {
   Connection conn = null;
   PreparedStatement stmt = null;
   try{
      //STEP 2: Register JDBC driver
      Class.forName("com.mysql.jdbc.Driver");

      //STEP 3: Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);

      //STEP 4: Execute a query
      System.out.println("Creating statement...");
      String sql = "UPDATE Employees set age=? WHERE id=?";
      stmt = conn.prepareStatement(sql);

      //Bind values into the parameters.
      stmt.setInt(1, 35);  // This would set age
      stmt.setInt(2, 102); // This would set ID

      // Let us update age of the record with ID = 102;
      int rows = stmt.executeUpdate();
      System.out.println("Rows impacted : " + rows );

      // Let us select all the records and display them.
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
}//end JDBCPreparedStatement
```
## CallableStatement示例
```java
//STEP 1. Import required packages
import java.sql.*;

public class JDBCCallableStatement {
   // JDBC driver name and database URL
   static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
   static final String DB_URL = "jdbc:mysql://localhost/EMP";

   //  Database credentials
   static final String USER = "root";
   static final String PASS = "123456";

   public static void main(String[] args) {
   Connection conn = null;
   CallableStatement stmt = null;
   try{
      //STEP 2: Register JDBC driver
      Class.forName("com.mysql.jdbc.Driver");

      //STEP 3: Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);

      //STEP 4: Execute a query
      System.out.println("Creating statement...");
      String sql = "{call getEmpName (?, ?)}";
      stmt = conn.prepareCall(sql);

      //Bind IN parameter first, then bind OUT parameter
      int empID = 102;
      stmt.setInt(1, empID); // This would set ID as 102
      // Because second parameter is OUT so register it
      stmt.registerOutParameter(2, java.sql.Types.VARCHAR);

      //Use execute method to run stored procedure.
      System.out.println("Executing stored procedure..." );
      stmt.execute();

      //Retrieve employee name with getXXX method
      String empName = stmt.getString(2);
      System.out.println("Emp Name with ID:" + 
               empID + " is " + empName);
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
}//end JDBCExample
```
