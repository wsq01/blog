


开发数据库应用程序的 Java API 称为 JDBC(Java Database Connectivity)。JDBC 提供访问和操作众多关系数据库的一个统一接口。JDBC API 是一个 Java 应用程序接口，通过 JDBC API，用 Java 编写的程序能够访问和操作关系数据库。

JDBC 通常由数据库厂商提供。访问 MySQL 数据库需要使用 MySQL JDBC 驱动程序，访问 Oracle 数据库需要使用 Oracle JDBC 驱动程序。
# 使用 JDBC
JDBC API 由类和接口构成，这些类和接口用于建立数据库的连接、把 SQL 语句发送到数据库、处理 SQL 语句的结果以及获取数据库的元数据。使用 Java 开发任何数据库应用程序都需要 4 个主要接口：`Driver`、`Collection`、`Statement`和`ResultSet`。这些接口定义了使用 SQL 访问数据库的一般架构。JDBC API 定义了这些接口，JDBC 驱动程序开发商为这些接口提供实现，程序员使用这些接口。

JDBC 应用程序使用`Driver`接口加载一个合适的驱动程序，使用`Collection`接口连接到数据库，使用`Statement`接口创建和执行 SQL 语句，如果语句返回结果的话，使用`ResultSet`接口处理结果。注意，有一些语句不返回结果，如 SQL 数据定义语句和 SQL 数据修改语句。

访问数据库的典型 Java 程序主要采用下列步骤：
## 1.加载驱动程序
在连接到数据库之前，必须使用下面的语句，加载一个合适的驱动程序。
```java
Class.forName("JDBCDriverClass");
```
驱动程序是一个实现接口`java.sql.Driver`的具体类。
| 数据库 | 驱动程序类 |
| :--: | :--: |
| Access | sun.jdbc.odbc.JdbcOdbcDriver |
| MySQL 5 | com.mysql.jdbc.Driver |
| MySQL 8 | com.mysql.cj.jdbc.Driver |
| Oracle | oracle.jdbc.driver.OracleDriver |

## 2.建立连接
为了连接到一个数据库，需要使用`DriverManager`类中的静态方法`getCollection(databaseURL)`。
```java
Collection collection = DriverManager.getCollection(databaseURL);
```
其中，`databaseURL`是数据库在 Internet 上的唯一标识符。

| 数据库 | URL 模式 |
| :--: | :--: |
| Access | jdbc:odbc:dataSource |
| MySQL | jdbc:mysql://hostname/dbname?k1=v1&k2=v2 |
| Oracle | jdbc:oracle:thin:@hostname:port#:oracleDBSID |

```java
Collection collection = DriverManager.getCollection(
  "jdbc:mysql://localhost:3306/test",
  "root",
  "123456"
);
```
## 3.创建语句
如果把一个`Collection`对象想象成一条连接程序和数据库的缆道，那么`Statement`对象或它的子类可以看作是一辆缆车，它为数据库传输 SQL 语句，并把运行结果返回程序。一旦创建了`Collection`对象，就可以创建执行 SQL 语句的语句。
```java
Statement statement = collection.createStatement();
```
## 4.执行语句
可以使用方法`executeUpdate(String sql)`来执行SQL DDL 或更新语句，可以使用`executeQuery(String sql)`来执行 SQL 查询语句。查询结果在`ResultSet`中返回。
```java
statement.executeUpdate("create table Temp (col1 char(5), col2 char(5))");

ResultSet resultSet = statement.executeQuery("select firstName, mi, lastName from Student where lastName = Smith");
```
## 5.处理结果
结果集`ResultSet`存有一个表，该表的当前行可以访问。当前行的初始位置是`null`。可以使用`next`方法移动到下一行，可以使用各种`get`方法从当前行获取值。
```java
while(resultSet.next()) {
  System.out.println(resultSet.getString(1) + " " + resultSet.getString(2) + ". " + resultSet.getString(3));
}
```
方法`getString(1)`、`getString(2)`和`getString(3)`分别获取`firstName`列、`mi`列和`lastName`列的值。

完整示例：
```java
package com.imooc.jdbc.sample;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class StandardJDBCSample {
    public static void main(String[] args) {
        Connection conn = null;
        try {
            // 1.加载并注册JDBC驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            // 2.创建数据库连接
            conn = DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/imooc?useSSL=false&useUnicode=true&charactorEncoding=UTF-8&serverTimezone=Asia/Shanghai",
                    "root",
                    "123456"
            );
            // 3.创建Statement对象
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("select * from employee");
            // 4.遍历查询结果
            while (rs.next()) {
                rs.getInt(1);
                String ename = rs.getString("ename");
                Float salary = rs.getFloat("salary");
                String dname = rs.getString("dname");
                System.out.println(ename + salary + dname);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            // 5.关闭连接，释放资源
            try {
                if (conn != null && conn.isClosed() == false) {
                    conn.close();
                }
            } catch (Exception ex) {
                ex.printStackTrace();
            }
        }
    }
}
```