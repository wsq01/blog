---
title: JDBC 数据库连接
date: 2021-03-05 11:01:11
tags: [JDBC]
categories: JDBC
---


安装相应的驱动程序后，就可以使用JDBC建立数据库连接了。

建立JDBC连接的四个步骤：
* 导入JDBC包：使用`import`语句在 Java 代码开头位置导入所需的类。
* 注册JDBC驱动程序：使 JVM 将所需的驱动程序实现加载到内存中，从而可以满足 JDBC 请求。
* 数据库URL配置：创建一个正确格式化的地址，指向要连接到的数据库。
* 创建连接对象：最后，调用`DriverManager`对象的`getConnection()`方法来建立实际的数据库连接。

# 导入JDBC包
`import`语句告诉 Java 编译器在哪里找到在代码中引用的类，`import`语句一般放置在源代码的开头。
```java
import java.sql.* ;  // for standard JDBC programs
```
# 注册JDBC驱动程序
在使用程序之前，必须先注册该驱动程序。注册驱动程序是将数据库驱动程序的类文件加载到内存中的过程，因此可以将其用作JDBC接口的实现。

只需在程序中一次注册就可以。可以通过两种方式之一来注册驱动程序。
## 方法1 - Class.forName()
注册驱动程序最常见的方法是使用 Java 的`Class.forName()`方法，将驱动程序的类文件动态加载到内存中，并将其自动注册。这个方法是推荐使用的方法，因为它使驱动程序注册可配置和便携。
```java
Class.forName("com.mysql.jdbc.Driver");
Connection conn = null;
conn = DriverManager.getConnection("jdbc:mysql://hostname:port/db_name","db_username", "db_password");
conn.close();
```
使用`getInstance()`方法来解决不合规的 JVM，但是必须编写两个额外的异常：
```java
try {
  Class.forName("oracle.jdbc.driver.OracleDriver").newInstance();
} catch(ClassNotFoundException ex) {
  System.out.println("Error: unable to load driver class!");
  System.exit(1);
} catch(IllegalAccessException ex) {
  System.out.println("Error: access problem while loading!");
  System.exit(2);
} catch(InstantiationException ex) {
  System.out.println("Error: unable to instantiate driver!");
  System.exit(3);
}
```
## 方法2 - DriverManager.registerDriver()
第二种方法是使用静态`DriverManager.registerDriver()`方法来注册驱动程序。

如果使用的是非 JDK 兼容的 JVM(如 Microsoft 提供的)，则应使用`registerDriver()`方法。
```java
try {
  Driver myDriver = new oracle.jdbc.driver.OracleDriver();
  DriverManager.registerDriver( myDriver );
} catch(ClassNotFoundException ex) {
  System.out.println("Error: unable to load driver class!");
  System.exit(1);
}
```
# 数据库 URL 配置
加载驱动程序后，可以使用`DriverManager.getConnection()`方法建立连接。

三个重载的`DriverManager.getConnection()`方法：
* `getConnection(String url)`
* `getConnection(String url, Properties prop)`
* `getConnection(String url, String user, String password)`

这里每个格式都需要一个数据库 URL。数据库URL是指向数据库的地址。

下表列出了常用的 JDBC 驱动程序名称和数据库 URL。

| RDBMS | JDBC驱动程序名称 | URL格式 |
| :--: | :--: | :--: |
| MySQL | com.mysql.jdbc.Driver | jdbc:mysql://hostname/databaseName |
| ORACLE | oracle.jdbc.driver.OracleDriver | jdbc:oracle:thin:@hostname:portNumber:databaseName |
| DB2 | com.ibm.db2.jdbc.net.DB2Driver | jdbc:db2:hostname:port Number/databaseName |

# 创建连接对象
上面列出了三种形式的`DriverManager.getConnection()`方法来创建一个连接对象。
## 使用具有用户名和密码的数据库URL
`getConnection()`最常用的形式要求传递数据库URL，用户名和密码。

假设使用Oracle thin驱动程序，那么需要为 URL 的数据库部分指定：`host:port:databaseName`值。

如果主机名为`amrood`的TCP/IP地址为`192.0.0.10`，并且 Oracle 侦听器配置为侦听端口 1521，并且要连接的数据库名称是`EMP`，则完整的数据库 URL 将是：
```
jdbc:oracle:thin:@amrood:1521:EMP
// 或者
jdbc:oracle:thin:@192.0.0.10:1521:EMP
```
现在必须使用适当的用户名和密码调用`getConnection()`方法获取一个`Connection`对象：
```java
String URL = "jdbc:oracle:thin:@amrood:1521:EMP";
// String URL = "jdbc:oracle:thin:@192.0.0.10:1521:EMP";
String USER = "username";
String PASS = "password"
Connection conn = DriverManager.getConnection(URL, USER, PASS);
```
## 仅使用数据库URL
`DriverManager.getConnection()`方法的第二种形式只需要数据库 URL。
```java
DriverManager.getConnection(String url);
```
但是，数据库 URL 包括用户名和密码，并具有以下一般形式：
```
jdbc:oracle:driver:username/password@database
```
所以，上述连接可以使用如下方式创建：
```java
String URL = "jdbc:oracle:thin:username/password@192.168.0.10:1521:EMP";
Connection conn = DriverManager.getConnection(URL);
```
使用数据库 URL 和`Properties`对象`DriverManager.getConnection()`方法的第三种形式需要一个数据库 URL 和一个`Properties`对象。
```java
DriverManager.getConnection(String url, Properties info);
```
`Properties`对象包含一组键-值对。在调用`getConnection()`方法时，它用于将驱动程序属性传递给驱动程序。
```java
import java.util.*;

String URL = "jdbc:oracle:thin:@amrood:1521:EMP";
Properties info = new Properties( );
info.put( "user", "root" );
info.put( "password", "password12321" );

Connection conn = DriverManager.getConnection(URL, info);
```
# 关闭JDBC连接
在JDBC程序结束之后，显式地需要关闭与数据库的所有连接以结束每个数据库会话。

但是，如果在编写程序中忘记了关闭也没有关系，Java的垃圾收集器将在清除过时的对象时也会关闭这些连接。依靠垃圾收集，是一个非常差的实践。所以应该要使用与连接对象关联的`close()`方法关闭连接。

要确保连接已关闭，可以将关闭连接的代码中编写在`finally`块中。
```java
conn.close();
```
# 示例
```java
//STEP 1. Import required packages
import java.sql.*;

public class SelectDatabase {
  // JDBC driver name and database URL
  static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";  
  static final String DB_URL = "jdbc:mysql://localhost/jdbc_db";

  //  Database credentials
  static final String USER = "root";
  static final String PASS = "123456";

  public static void main(String[] args) {
  Connection conn = null;
  try{
    //STEP 2: Register JDBC driver
    Class.forName("com.mysql.jdbc.Driver");

    //STEP 3: Open a connection
    System.out.println("Connecting to a selected database...");
    conn = DriverManager.getConnection(DB_URL, USER, PASS);
    System.out.println("Connected database successfully...");
  }catch(SQLException se){
    //Handle errors for JDBC
    se.printStackTrace();
  }catch(Exception e){
    //Handle errors for Class.forName
    e.printStackTrace();
  }finally{
    //finally block used to close resources
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
