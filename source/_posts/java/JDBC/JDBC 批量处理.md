---
title: JDBC 批量处理
date: 2021-03-011 15:51:24
tags: [JDBC]
categories: JDBC
---


批量处理允许将相关的 SQL 语句分组到批处理中，并通过对数据库的一次调用来提交它们，一次执行完成与数据库之间的交互。一次向数据库发送多个 SQL 语句时，可以减少通信开销，从而提高性能。

不需要JDBC驱动程序来支持此功能。应该使用`DatabaseMetaData.supportsBatchUpdates()`方法来确定目标数据库是否支持批量更新处理。如果 JDBC 驱动程序支持此功能，该方法将返回`true`。

`Statement，PreparedStatement`和`CallableStatement`的`addBatch()`方法用于将单个语句添加到批处理。`executeBatch()`用于执行组成批量的所有语句。

`executeBatch()`返回一个整数数组，数组的每个元素表示相应更新语句的更新计数。就像将批处理语句添加到处理中一样，可以使用`clearBatch()`方法删除它们。此方法将删除所有使用`addBatch()`方法添加的语句。但是，无法指定选择某个要删除的语句。

# 使用Statement对象进行批处理
* 使用`Statement`对象的批处理的典型步骤序列：
* 使用`createStatement()`方法创建`Statement`对象。
* 使用`setAutoCommit()`将自动提交设置为`false`。
* 使用`addBatch()`方法在创建的`Statement`对象上添加 SQL 语句到批处理中。
* 在创建的`Statement`对象上使用`executeBatch()`方法执行所有 SQL 语句。
* 最后，使用`commit()`方法提交所有更改。

## 实例
```java
// Import required packages
import java.sql.*;

public class BatchingWithStatement {
  // JDBC driver name and database URL
  static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";  
  static final String DB_URL = "jdbc:mysql://localhost/EMP";

  //  Database credentials
  static final String USER = "root";
  static final String PASS = "123456";

  public static void main(String[] args) {
  Connection conn = null;
  Statement stmt = null;
  try {
    // Register JDBC driver
    Class.forName("com.mysql.cj.jdbc.Driver");

    // Open a connection
    System.out.println("Connecting to database...");
    conn = DriverManager.getConnection(DB_URL,USER,PASS);

    // Create statement
    System.out.println("Creating statement...");
    stmt = conn.createStatement();

    // Set auto-commit to false
    conn.setAutoCommit(false);

    // First, let us select all the records and display them.
    printRows(stmt);

    // Create SQL statement
    String SQL = "INSERT INTO Employees (id, first, last, age) VALUES(200,'Curry', 'Stephen', 30)";
    // Add above SQL statement in the batch.
    stmt.addBatch(SQL);

    // Create one more SQL statement
    SQL = "INSERT INTO Employees (id, first, last, age) VALUES(201,'Kobe', 'Bryant', 35)";
    // Add above SQL statement in the batch.
    stmt.addBatch(SQL);

    // Create one more SQL statement
    SQL = "UPDATE Employees SET age = 35 WHERE id = 100";
    // Add above SQL statement in the batch.
    stmt.addBatch(SQL);

    // Create an int[] to hold returned values
    int[] count = stmt.executeBatch();

    //Explicitly commit statements to apply changes
    conn.commit();

    // Again, let us select all the records and display them.
    printRows(stmt);

    // Clean-up environment
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

public static void printRows(Statement stmt) throws SQLException{
   System.out.println("Displaying available rows...");
   // Let us select all the records and display them.
   String sql = "SELECT id, first, last, age FROM Employees";
   ResultSet rs = stmt.executeQuery(sql);

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
   System.out.println();
   rs.close();
}//end printRows()
}//end JDBCExample
```
# 使用PrepareStatement对象进行批处理
使用`PrepareStatement`对象进行批处理的典型步骤顺序：
* 使用占位符创建 SQL 语句。
* 使用`prepareStatement()`方法创建`PrepareStatement`对象。
* 使用`setAutoCommit()`将自动提交设置为`false`。
* 使用`addBatch()`方法在创建的`Statement`对象上添加 SQL 语句到批处理中。
* 在创建的`Statement`对象上使用`executeBatch()`方法执行所有 SQL 语句。
* 最后，使用`commit()`方法提交所有更改。

## 示例
```java
// Import required packages
import java.sql.*;

public class BatchingWithPrepareStatement {
  // JDBC driver name and database URL
  static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";  
  static final String DB_URL = "jdbc:mysql://localhost/EMP";

  //  Database credentials
  static final String USER = "root";
  static final String PASS = "123456";

  public static void main(String[] args) {
    Connection conn = null;
    PreparedStatement stmt = null;
    try{
      // Register JDBC driver
      Class.forName("com.mysql.cj.jdbc.Driver");

      // Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);

      // Create SQL statement
      String SQL = "INSERT INTO Employees(id,first,last,age) VALUES(?, ?, ?, ?)";

      // Create preparedStatemen
      System.out.println("Creating statement...");
      stmt = conn.prepareStatement(SQL);

      // Set auto-commit to false
      conn.setAutoCommit(false);

      // First, let us select all the records and display them.
      printRows(stmt);

      // Set the variables
      stmt.setInt( 1, 400 );
      stmt.setString( 2, "Python" );
      stmt.setString( 3, "Zhang" );
      stmt.setInt( 4, 33 );
      // Add it to the batch
      stmt.addBatch();

      // Set the variables
      stmt.setInt( 1, 401 );
      stmt.setString( 2, "C++" );
      stmt.setString( 3, "Huang" );
      stmt.setInt( 4, 31 );
      // Add it to the batch
      stmt.addBatch();

      // Create an int[] to hold returned values
      int[] count = stmt.executeBatch();

      //Explicitly commit statements to apply changes
      conn.commit();

      // Again, let us select all the records and display them.
      printRows(stmt);

      // Clean-up environment
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

  public static void printRows(Statement stmt) throws SQLException{
    System.out.println("Displaying available rows...");
    // Let us select all the records and display them.
    String sql = "SELECT id, first, last, age FROM Employees";
    ResultSet rs = stmt.executeQuery(sql);

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
    System.out.println();
    rs.close();
  }//end printRows()
}//end JDBCExample
```
