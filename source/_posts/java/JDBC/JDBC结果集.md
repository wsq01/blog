---
title: JDBC 结果集
date: 2021-03-08 15:51:24
tags: [JDBC]
categories: JDBC
---


SQL语句执行后从数据库查询读取数据，返回的数据放在结果集中。`java.sql.ResultSet`接口表示数据库查询的结果集。

`ResultSet`对象维护指向结果集中当前行的游标。 术语“结果集”是指包含在`ResultSet`对象中的行和列数据。

`ResultSet`接口的方法可以分为三类：
* 浏览方法：用于移动光标。
* 获取方法：用于查看光标指向的当前行的列中的数据。
* 更新方法：用于更新当前行的列中的数据。 然后在基础数据库中更新数据。

光标可以基于`ResultSet`的属性移动。当创建生成`ResultSet`的相应`Statement`时，将指定这些属性。

JDBC 提供以下连接方法来创建具有所需`ResultSet`的语句：
* `createStatement(int RSType, int RSConcurrency);`
* `prepareStatement(String SQL, int RSType, int RSConcurrency);`
* `prepareCall(String sql, int RSType, int RSConcurrency);`

第一个参数表示`ResultSet`对象的类型，第二个参数是两个`ResultSet`常量之一，用于指定结果集是只读还是可更新。
## ResultSet类型
可能的`RSType`值如下。如果不指定任何`ResultSet`类型，将自动分配一个`TYPE_FORWARD_ONLY`值。

| 类型 | 描述 |
| :--: | :--: |
| ResultSet.TYPE_FORWARD_ONLY | 光标只能在结果集中向前移动。 |
| ResultSet.TYPE_SCROLL_INSENSITIVE | 光标可以向前和向后滚动，结果集对创建结果集后发生的数据库所做的更改不敏感。 |
| ResultSet.TYPE_SCROLL_SENSITIVE | 光标可以向前和向后滚动，结果集对创建结果集之后发生的其他数据库的更改敏感。 |

## ResultSet的并发性
可能的`RSConcurrency`如下。 如果不指定任何并发类型，将自动获得一个`CONCUR_READ_ONLY`值。

| 并发 | 描述 |
| :--: | :--: |
| ResultSet.CONCUR_READ_ONLY | 创建只读结果集，这是默认值。 |
| ResultSet.CONCUR_UPDATABLE | 创建可更新的结果集 |

## 浏览结果集
`ResultSet`接口中有几种涉及移动光标的方法，包括：

| 方法 | 描述 |
| :--: | :--: |
| public void beforeFirst() throws SQLException | 将光标移动到第一行之前 |
| public void afterLast() throws SQLException | 将光标移动到最后一行之后 |
| public boolean first() throws SQLException | 将光标移动到第一行 |
| public void last() throws SQLException | 将光标移动到最后一行 |
| public boolean absolute(int row) throws SQLException | 将光标移动到指定的行 |
| public boolean relative(int row) throws SQLException | 从当前指向的位置，将光标向前或向后移动给定行数 |
| public boolean previous() throws SQLException | 将光标移动到上一行。 如果上一行关闭结果集，此方法返回false |
| public boolean next() throws SQLException | 将光标移动到下一行。 如果结果集中没有更多行，则此方法返回false |
| public int getRow() throws SQLException | 返回光标指向的行号 |
| public void moveToInsertRow() throws SQLException | 将光标移动到结果集中的特殊行，该行可用于将新行插入数据库。当前光标位置被记住 |
| public void moveToCurrentRow() throws SQLException | 如果光标当前位于插入行，则将光标移回当前行; 否则，此方法什么也不做 |

## 查看结果集
`ResultSet`接口包含数十种获取当前行数据的方法。

每个可能的数据类型都有一个`get`方法，每个`get`方法有两个版本，一个是采用列名称。另一个采用列索引。

例如，如果对查看感兴趣的列包含一个`int`，则需要使用`ResultSet`的其中一个`getInt()`方法：

| 方法 | 描述 |
| :--: | :--: |
| public int getInt(String columnName) throws SQLException | 返回名为columnName的列中当前行中的int值 |
| public int getInt(int columnIndex) throws SQLException | 返回指定列索引当前行中的int值。列索引从1开始，意味着行的第一列为1，行的第二列为2，依此类推 |

类似地，在八个 Java 基元类型中的每一个的`ResultSet`接口中都有`get`方法，以及常见的类型，如`java.lang.String`，`java.lang.Object`和`java.net.URL`等。
## 更新结果集
`ResultSet`接口包含用于更新结果集的数据的更新方法的集合。

与`get`方法一样，每种数据类型都有两种更新方法：一个是采用列名称。另一个采用列索引。

例如，要更新结果集当前行的`String`列，可以使用以下`updateString()`方法之一：

| 方法 | 描述 |
| :--: | :--: |
| public void updateString(int columnIndex, String s) throws SQLException | 将指定列中的String值更改为指定的s值 |
| public void updateString(String columnName, String s) throws SQLException | 与前面的方法类似，除了使用列的名称而不是列的索引指定 |

有八种基本数据类型的更新方法，以及`java.sql`包中的`String，Object，URL`和 SQL 数据类型。

更新结果集中的一行会更改`ResultSet`对象中当前行的列，但不会更改底层数据库中的列的值。要更新数据库中的行，需要调用以下方法之一。

| 方法 | 描述 |
| :--: | :--: |
| public void updateRow() | 更新数据库中当前行 |
| public void deleteRow() | 从数据库中删除当前行 | 
| public void refreshRow() | 刷新结果集中的数据以反映数据库中最近的任何更改 |
| public void cancelRowUpdates() | 取消对当前行所做的任何更新 |
| public void insertRow() | 在数据库中插入一行。只有当光标指向插入行时，才能调用此方法 |

# 示例
## 浏览结果集
```java
//STEP 1. Import required packages
import java.sql.*;

public class NavigatingResultSet {
  // JDBC driver name and database URL
  static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
  static final String DB_URL = "jdbc:mysql://localhost/EMP";

  //  Database credentials
  static final String USER = "root";
  static final String PASS = "123456";

  public static void main(String[] args) {
    Connection conn = null;
    Statement stmt = null;
    try {
      //STEP 2: Register JDBC driver
      Class.forName("com.mysql.jdbc.Driver");
      //STEP 3: Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);
      //STEP 4: Execute a query to create statment with
      // required arguments for RS example.
      System.out.println("Creating statement...");
      stmt = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY);
      String sql;
      sql = "SELECT id, first, last, age FROM Employees";
      ResultSet rs = stmt.executeQuery(sql);

      // Move cursor to the last row.
      System.out.println("Moving cursor to the last...");
      rs.last();

      //STEP 5: Extract data from result set
      System.out.println("Displaying record...");
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

      // Move cursor to the first row.
      System.out.println("Moving cursor to the first row...");
      rs.first();

      //STEP 6: Extract data from result set
      System.out.println("Displaying record...");
      //Retrieve by column name
      id  = rs.getInt("id");
      age = rs.getInt("age");
      first = rs.getString("first");
      last = rs.getString("last");

      //Display values
      System.out.print("ID: " + id);
      System.out.print(", Age: " + age);
      System.out.print(", First: " + first);
      System.out.println(", Last: " + last);
      // Move cursor to the first row.

      System.out.println("Moving cursor to the next row...");
      rs.next();

      //STEP 7: Extract data from result set
      System.out.println("Displaying record...");
      id  = rs.getInt("id");
      age = rs.getInt("age");
      first = rs.getString("first");
      last = rs.getString("last");

      //Display values
      System.out.print("ID: " + id);
      System.out.print(", Age: " + age);
      System.out.print(", First: " + first);
      System.out.println(", Last: " + last);

      //STEP 8: Clean-up environment
      rs.close();
      stmt.close();
      conn.close();
    } catch(SQLException se){
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
      }catch(SQLException se){
        se.printStackTrace();
      }//end finally try
    }//end try
    System.out.println("Goodbye!");
  }//end main
}//end JDBCExample
```
## 更新结果集
```java
//STEP 1. Import required packages
import java.sql.*;

public class JDBCExample {
  // JDBC driver name and database URL
  static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
  static final String DB_URL = "jdbc:mysql://localhost/EMP";

  //  Database credentials
  static final String USER = "username";
  static final String PASS = "password";

  public static void main(String[] args) {
    Connection conn = null;
    try{
      //STEP 2: Register JDBC driver
      Class.forName("com.mysql.jdbc.Driver");

      //STEP 3: Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);

      //STEP 4: Execute a query to create statment with
      // required arguments for RS example.
      System.out.println("Creating statement...");
      Statement stmt = conn.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);
      //STEP 5: Execute a query
      String sql = "SELECT id, first, last, age FROM Employees";
      ResultSet rs = stmt.executeQuery(sql);

      System.out.println("List result set for reference....");
      printRs(rs);

      //STEP 6: Loop through result set and add 5 in age
      //Move to BFR postion so while-loop works properly
      rs.beforeFirst();
      //STEP 7: Extract data from result set
      while(rs.next()){
        //Retrieve by column name
        int newAge = rs.getInt("age") + 5;
        rs.updateDouble( "age", newAge );
        rs.updateRow();
      }
      System.out.println("List result set showing new ages...");
      printRs(rs);
      // Insert a record into the table.
      //Move to insert row and add column data with updateXXX()
      System.out.println("Inserting a new record...");
      rs.moveToInsertRow();
      rs.updateInt("id",104);
      rs.updateString("first","John");
      rs.updateString("last","Paul");
      rs.updateInt("age",40);
      //Commit row
      rs.insertRow();

      System.out.println("List result set showing new set...");
      printRs(rs);

      // Delete second record from the table.
      // Set position to second record first
      rs.absolute( 2 );
      System.out.println("List the record before deleting...");
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

     //Delete row
      rs.deleteRow();
      System.out.println("List result set after deleting one records...");
      printRs(rs);

      //STEP 8: Clean-up environment
      rs.close();
      stmt.close();
      conn.close();
   } catch(SQLException se) {
      //Handle errors for JDBC
      se.printStackTrace();
   } catch(Exception e) {
      //Handle errors for Class.forName
      e.printStackTrace();
   } finally {
      //finally block used to close resources
      try {
        if(conn!=null)
          conn.close();
      } catch(SQLException se){
        se.printStackTrace();
      }//end finally try
   }//end try
   System.out.println("Goodbye!");
  }//end main
  public static void printRs(ResultSet rs) throws SQLException {
    //Ensure we start with first row
    rs.beforeFirst();
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
  }//end printRs()
}//end JDBCExample
```