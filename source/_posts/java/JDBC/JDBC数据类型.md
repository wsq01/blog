


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

# 日期和时间数据类型
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
# 处理NULL值
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

