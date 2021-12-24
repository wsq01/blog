---
title: MyBatis 映射器
date: 2021-04-12 15:56:16
tags: [MyBatis]
categories: [MyBatis]
---

映射器文件中包含一组 SQL 语句（例如查询、添加、删除、修改），这些语句称为映射语句或映射 SQL 语句。

映射器由 Java 接口和 XML 文件（或注解）共同组成，它的作用如下：
* 定义参数类型
* 配置缓存
* 提供 SQL 语句和动态 SQL
* 定义查询结果和 POJO 的映射关系

映射器有两种实现方式：
* 通过 XML 文件方式实现，比如在`mybatis-config.xml`文件中描述的 XML 文件，用来生成`mapper`。
* 通过注解的方式实现，使用`Configuration`对象注册`Mapper`接口。

如果 SQL 语句存在动态 SQL 或者比较复杂，使用注解写在 Java 文件里可读性差，且增加了维护的成本。所以一般建议使用 XML 文件配置的方式，避免重复编写 SQL 语句。
# XML实现映射器
XML 定义映射器分为两个部分：接口和 XML。
```java
package net.biancheng.mapper;
import java.util.List;
import net.biancheng.po.Website;
public interface WebsiteMapper {
  public List<Website> selectAllWebsite();
}
```
```xml
<!-- WebsiteMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="net.biancheng.mapper.WebsiteMapper">
  <select id="selectAllWebsite" resultType="net.biancheng.po.Website">
    select * from website
  </select>
</mapper>
```
* `namespace`用来定义命名空间，该命名空间和定义接口的全限定名一致。
* `<select>`元素表明这是一条查询语句，属性`id`用来标识这条 SQL。`resultType`表示返回的是一个`Website`类型的值。

在 MyBatis 配置文件中添加以下代码。
```xml
<mapper resource="net/biancheng/mapper/WebsiteMapper.xml" />
```
该语句用来引入 XML 文件，MyBatis 会读取`WebsiteMapper.xml`文件，生成映射器。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml");
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    SqlSession ss = ssf.openSession();
    WebsiteMapper websiteMapper = ss.getMapper(WebsiteMapper.class);
    List<Website> websitelist = websiteMapper.selectAllWebsite();
    for (Website site : websitelist) {
      System.out.println(site);
    }
    ss.commit();
    ss.close();
  }
}
```
# 注解实现映射器
使用注解的方式实现映射器，只需要在接口中使用 Java 注解，注入 SQL 即可。
```java
package net.biancheng.mapper;
import java.util.List;
import org.apache.ibatis.annotations.Select;
import net.biancheng.po.Website;
public interface WebsiteMapper2 {
  @Select(value = "select * from website")
  public List<Website> selectAllWebsite();
}
```
> 如果使用注解和 XML 文件两种方式同时定义，那么 XML 方式将覆盖掉注解方式。

虽然这里注解的方式看起来比 XML 简单，但是现实中我们遇到的 SQL 会复杂得多。如果 SQL 语句中有多个表的关联、多个查询条件、级联、条件分支等，显然这条 SQL 就会复杂的多，所以并不建议使用这种方式。

此外，XML 可以相互引入，而注解是不可以的，所以在一些比较复杂的场景下，使用 XML 方式会更加灵活和方便。因此大部分的企业都以 XML 为主，当然在一些简单的表和应用中使用注解方式也会比较简单。

这个接口可以在 XML 中定义，将在`mybatis-config.xml`中配置 XML 的语句修改为以下语句即可。
```xml
<mapper resource="com/mybatis/mapper/WebsiteMapper2" />
```
也可以使用`configuration`对象注册这个接口：
```java
configuration.addMapper(WebsiteMapper2.class);
```
# 映射器的主要元素

| 元素名称      | 描述 | 备注 |
| :--: | :--: | :--: |
| mapper       | 映射文件的根节点，只有 namescape 一个属性 | namescape 作用如下：<br>1.用于区分不同的 mapper，全局唯一<br>2.绑定DAO接口，即面向接口编程。当 namescape 绑定某一接口后，可以不用写该接口的实现类，MyBatis 会通过接口的完整限定名查找到对应的 mapper 配置来执行 SQL 语句。因此 namescape 的命名必须要跟接口同名。 |
| select        | 查询语句 | 可以自定义参数，返回结果集等 | 
| insert        | 插入语句 | 执行后返回一个整数，代表插入的条数 | 
| update        | 更新语句 | 执行后返回一个整数，代表更新的条数 | 
| delete        | 删除语句 | 执行后返回一个整数，代表删除的条数 | 
| parameterMap	| 定义参数映射关系 | 即将被删除的元素，不建议使用 | 
| sql           | 允许定义一部分的 SQL，然后在各个地方引用它 | 例如，一张表列名，我们可以一次定义，在多个 SQL 语句中使用 | 
| resultMap     | 用来描述数据库结果集与对象的对应关系，它是最复杂、最强大的元素 | 提供映射规则 | 
| cache         | 配置给定命名空间的缓存 |  | 
| cache-ref     | 其它命名空间缓存配置的引用 |  | 

SQL 映射文件中的`mapper`元素的`namescape`属性有如下要求。
* `namescape`的命名必须跟某个 DAO 接口同名，同属于 DAO 层，因此代码结构上，映射文件与该接口应放置在同一`package`下（如`net.biancheng.dao.website`），并且习惯上是以`Mapper`结尾（如 `WebsiteMapper.java、WebsiteMapper.xml`）。
* 不同的`mapper`文件中子元素的`id`可以相同，MyBatis 通过`namescape`和子元素的`id`联合区分。接口中的方法与映射文件中的 SQL 语句`id`应一一对应。

## select 标签
在 SQL 映射文件中`<select>`标签用于映射 SQL 的`select`语句。
```xml
<select id="selectAllWebsite" resultType="net.biancheng.po.Website" parameterType="string">
  SELECT id,NAME,url FROM website WHERE NAME LIKE CONCAT ('%',#{name},'%')
</select>
```
以上是一个`id`为`selectAllWebsite`的映射语句，参数类型为`string`，返回结果类型为`Website`。

执行 SQL 语句时可以定义参数，参数可以是一个简单的参数类型，例如`int、float、String`；也可以是一个复杂的参数类型，例如`JavaBean、Map`等。MyBatis 提供了强大的映射规则，执行 SQL 后，MyBatis 会将结果集自动映射到`JavaBean`中。

> 为了使数据库的查询结果和返回值类型中的属性能够自动匹配，通常会对 MySQL 数据库和JavaBean采用同一套命名规则，即 Java 命名驼峰规则，这样就不需要再做映射了（数据库表字段名和属性名不一致时需要手动映射）。

参数的传递使用`#{参数名}`，相当于告诉 MyBatis 生成`PreparedStatement`参数。对于 JDBC，该参数会被标识为`?`。以上 SQL 语句可以使用 JDBC 实现。
```java
String sql = "SELECT id,NAME,url FROM website WHERE NAME LIKE CONCAT ('%',?,'%')";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1,userName);
```
### select 标签常用属性

| 属性 | 描述 |
| :--: | :--: |
| id | 在命名空间中唯一的标识符，可以被用来引用这条语句。 |
| parameterType | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。 |
| resultType | 期望从这条语句中返回结果的类全限定名或别名。 注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。 resultType 和 resultMap 之间只能同时使用一个。 |
| resultMap | 对外部 resultMap 的命名引用。resultType 和 resultMap 之间只能同时使用一个。 |
| flushCache | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。 |
| useCache | 将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。 |
| timeout | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| fetchSize | 这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。 |
| statementType | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| resultSetType | FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。 |
| databaseId | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |
| resultOrdered | 这个设置仅针对嵌套结果 select 语句：如果为 true，将会假设包含了嵌套结果集或是分组，当返回一个主结果行时，就不会产生对前面结果集的引用。 这就使得在获取嵌套结果集的时候不至于内存不够用。默认值：false。 |
| resultSets | 这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。 |

### 传递多个参数
现在需要根据`id`和`name`来模糊查询网站信息，显然这涉及到了两个参数。给映射器传递多个参数分为以下三种方法。
* 使用`Map`传递参数
* 使用注解传递参数
* 使用`JavaBean`传递参数

#### 使用 Map 传递参数
```xml
<!-- 根据name和url模糊查询网站信息 -->
<select id="selectWebsiteByMap" resultType="net.biancheng.po.Website" parameterType="map">
  SELECT id,NAME,url FROM website
  WHERE name LIKE CONCAT ('%',#{name},'%')
  AND url LIKE CONCAT ('%',#{url},'%')
</select>
```
在`WebsiteMapper`接口中，方法如下。
```java
public List<Website> selectWebsiteByMap(Map<String, String> params);
```
测试代码如下。
```java
Map<String,String> paramsMap = new HashMap<String,String>();
paramsMap.put("name","编程");
paramsMap.put("url","biancheng");
websiteMapper.selectWebsiteByMap(paramsMap);
```
使用`Map`传递参数虽然简单易用，但是由于这样设置参数需要键值对应，业务关联性不强，需要深入到程序中看代码，造成可读性下降。
#### 使用注解传递参数
使用 MyBatis 的注解`@Param()`传递参数。
```xml
<!-- 根据name和url模糊查询网站信息 -->
<select id="selectWebsiteByAn" resultType="net.biancheng.po.Website">
  SELECT id,NAME,url FROM website
  WHERE name LIKE CONCAT ('%',#{name},'%')
  AND url LIKE CONCAT ('%',#{url},'%')
</select>
```
`WebsiteMapper`接口中方法如下。
```java
public List<Website> selectWebsiteByAn(@Param("name") String name, @Param("url") String url);
```
当我们把参数传递给后台时，MyBatis 通过`@Param`提供的名称就会知道`#{name}`代表`name`参数，提高了参数可读性。但是如果这条 SQL 拥有 10 个参数的查询，就会造成可读性下降，增强了代码复杂性。
#### 使用JavaBean传递参数
在参数过多的情况下，MyBatis 允许组织一个`JavaBean`，通过简单的`setter`和`getter`方法设置参数，提高可读性。
```xml
<!-- 根据name和url模糊查询网站信息 -->
<select id="selectWebsiteByAn" resultType="net.biancheng.po.Website">
  SELECT id,NAME,url FROM website
  WHERE name LIKE CONCAT ('%',#{name},'%')
  AND url LIKE CONCAT ('%',#{url},'%')
</select>
```
`WebsiteMapper`接口中方法如下。
```
public List<Website> selectWebsiteByAn(Website website);
```
#### 区别
以上 3 种方式的区别如下。
* 使用`Map`传递参数会导致业务可读性的丧失，继而导致后续扩展和维护的困难，所以在实际应用中我们应该果断废弃该方式。
* 使用`@Param`注解传递参数会受到参数个数的影响。当`n≤5`时，它是最佳的传参方式，因为它更加直观；当`n>5`时，多个参数将给调用带来困难。
* 当参数个数大于 5 个时，建议使用`JavaBean`方式。

## insert 标签
`insert`标签用来定义插入语句，执行插入操作。当 MyBatis 执行完一条插入语句后，就会返回其影响数据库的行数。
```xml
<!-- WebsiteMapper.xml -->
<!-- 增加网站信息 -->
<insert id="addWebsite" parameterType="string">
  insert into website(name) values(#{name})
</insert>
```
在`WebsiteMapper`接口中定义一个`add()`方法。
```java
public int addWebsite(String name);
```
参数为`Sting`类型的字符串；返回值为`int`类型，即执行 SQL 后，插入记录的行数。

测试代码如下。
```java
//插入 name 为编程帮4 的记录
String name = "baidu";
int i = websiteMapper.addWebsite(name);
System.out.println("共插入了 " + i + " 条记录");
```
执行测试代码，控制台输出如下。
```
共插入了 1 条记录
```
### insert 标签常用属性

| 属性名称 | 描述 | 备注 |
| :--: | :--: | :--: |
| id             | 它和 Mapper 的命名空间组合起来使用，是唯一标识符，供 MyBatis 调用 | 如果命名空间+ id 不唯一，那么 MyBatis 抛出异常 |
| parameterType	 | 传入 SQL 语句的参数类型的全限定名或别名，它是一个可选属性。 | 支持基本数据类型和 JavaBean、Map 等复杂数据类型 |
| keyProperty    | 该属性的作用是将插入操作的返回值赋给 PO 类的某个属性，通常为主键对应的属性。如果是联合主键，可以将多个值用逗号隔开。 | |
| useGeneratedKeys | 该属性用来设置，是否使用 JDBC 提供的 getGenereatedKeys() 方法，获取数据库内部产生的主键并赋值到 keyProperty 属性设置的请求对象的属性中，例如 MySQL、SQL Server 等自动递增的字段，其默认值为 false。 | 该属性值设置为 true 后，会将数据库生成的主键回填到请求对象中，以供其他业务使用。 |
| flushCache     | 该属性用于设置执行该操作后，是否会清空二级缓存和本地缓存，默认值为 true。 |  |
| timeout        | 该属性用于设置执行该操作的最大时限，如果超时，就抛异常。 |  |
| databaseId     | 取值范围 oracle、mysql 等，表示数据库厂家；元素内部可通过 <if test="_databaseId = 'oracle'"> 来为特定数据库指定不同的 sql 语句。	MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。 MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库 databaseId 属性的所有语句。 如果同时找到带有 databaseId 和不带 databaseId 的相同语句，则后者会被舍弃。 |
keyColumn       | 该属性用于设置第几列是主键，当主键列不是表中的第 1 列时，就需要设置该属性。如果是联合主键，可以将多个值用逗号隔开。 |  |

> `insert`标签中没有`resultType`属性，只有查询操作才需要对返回结果类型进行相应的指定。

### 传递多个参数
Mybatis 为我们提供以下 3 种方式，来实现给映射器传递多个参数：
* 使用`Map`传递参数
* 使用注解传递参数
* 使用`JavaBean`传递参数

#### 使用 Map 传递参数
我们可以将参数封装到一个`Map`对象中，然后传递给 MyBatis 的映射器。

1. 在`WebsiteMapper`接口中，定义一个`addWebsiteByMap()`方法，并使用`Map`传递参数。
```
int addWebsiteByMap(Map<String, String> params);
```
2. 在`WebsiteMapper.xml`中，使用`insert`标签定义一条插入语句，并接收通过`Map`传递的参数。
```xml
<!--接收 Map 参数-->
<insert id="addWebsiteByMap" parameterType="map">
  insert into Website (name, url) values (#{name}, #{url})
</insert>
```
3. 测试代码如下。
```java
Map<String, String> params = new HashMap<>();
params.put("name", "baidu");
params.put("url", "https://www.baidu.com/");
int i = websiteMapper.addWebsiteByMap(params);
System.out.println("通过 Map 成功向数据库中添加了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 Map 成功向数据库中添加了 1 条记录
```

#### 使用注解传递参数
我们还可以使用 MyBatis 提供的`@Param`注解给注解器传递参数。

1. 在`WebsiteMapper`接口中，定义一个`addWebsiteByParam()`方法，并使用`@Param`注解传递参数。
```java
int addWebsiteByParam(@Param("name") String name, @Param("url") String url);
```
2. 在`WebsiteMapper.xml`中使用`insert`标签定义一条插入语句，并接收通过`@Param`注解传递的参数。
```xml
<!--接收 @Param 注解传递的参数-->
<insert id="addWebsiteByParam">
  insert into Website (name, url)  values (#{name}, #{url})
</insert>
```
3. 测试代码如下。
```java
int i = websiteMapper.addWebsiteByParam("baidu", "www.baidu.com");
System.out.println("通过 @Param 注解成功向数据库中添加了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 @Param 注解成功向数据库中添加了 1 条记录
```

#### 使用 JavaBean 传递参数
在参数过多的情况下，我们可以将参数通过`setter`方法封装到`JavaBean`（实体类）对象中 ，传递给映射器。

1. 在`WebsiteMapper`接口中,定义一个`addWebsiteByJavaBean()`方法，并使用`JavaBean`传递参数。
```java
int addWebsiteByJavaBean(Website website);
```
2. 在`WebsiteMapper.xml`中使用`insert`标签定义一条插入语句，并接收通过`JavaBean`传递的参数。
```xml
<!--接收通过 JavaBean 传递的参数-->
<insert id="addWebsiteByJavaBean" parameterType="net.biancheng.www.po.Website">
  insert into Website (name, url) values (#{name}, #{url})
</insert>
```
3. 测试代码如下。
```java
//创建 JavaBean 对象
Website website = new Website();
//通过 setter 方法将参数封装
website.setName("编程帮 JavaBean");
website.setUrl("www.biancheng.net");
int i = websiteMapper.addWebsiteByJavaBean(website);
System.out.println("通过 JavaBean 成功向数据库中添加了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 JavaBean 成功向数据库中添加了 1 条记录
```

#### 区别
以上 3 种方式的区别如下：
* 使用`Map`传递参数会导致业务可读性的丧失，继而导致后续扩展和维护的困难，所以在实际应用中我们应该果断废弃该方式。
* 使用`@Param`注解传递参数会受到参数个数的影响。当`n≤5`时，它是最佳的传参方式，因为它更加直观；当`n>5`时，多个参数将给调用带来困难。
* 当参数个数大于 5 个时，建议使用`JavaBean`方式。

### 主键（自动递增）回填
MySQL、SQL Server 等数据库表可以采用自动递增的字段作为其主键，当向这样的数据库表插入数据时，即使不指定自增主键的值，数据库也会根据自增规则自动生成主键并插入到表中。

一些特殊情况下，我们可能需要将这个刚刚生成的主键回填到请求对象（原本不包含主键信息的请求对象）中，供其他业务使用。此时，我们就可以通过在`insert`标签中添加`keyProperty`和`useGeneratedKeys`属性，来实现该功能。

1. 为`WebsiteMapper.xml`中`id`为`addWebsite`的`insert`标签添加`keyProperty`和`useGeneratedKeys`属性：
```xml
<!--添加一个网站信息，成功后将主键值返回填给id(po的属性)-->
<insert id="addWebsite" parameterType="net.biancheng.po.Website" keyProperty="id" useGeneratedKeys="true">
  insert into Website (name,url) values(#{name},#{url})
</insert>
```
2. 测试代码如下：
```java
// 添加一个网站信息
Website addsite = new Website();
//插入的对象中不包含主键 id 
addsite.setName("baidu");
addsite.setUrl("https://www.baidu.com/");
//执行插入
int num = websiteMapper.addWebsite(addsite);
System.out.println("添加了 " + num + " 条记录");
//获取回填的主键
System.out.println("添加记录的主键是:" + addsite.getId());
```
3. 执行测试代码，控制台输出如下。
```
添加了 1 条记录
添加记录的主键是:3
```

### 自定义主键
如果在实际工程中使用的数据库不支持主键自动递增，或者取消了主键自动递增的规则，可以使用 MyBatis 的`<selectKey>`元素来自定义生成主键。
```xml
<!-- 添加一个网站，#{name}为 net.biancheng.po.Website 的属性值 -->
<insert id="insertWebsite" parameterType="net.biancheng.po.Website">
  <!-- 先使用selectKey标签定义主键，然后再定义SQL语句 -->
  <selectKey keyProperty="id" resultType="Integer" order="BEFORE">
    select if(max(id) is null,1,max(id)+1) as newId from Website
  </selectKey>
  insert into Website (id,name,url) values(#{id},#{name},#{url})
</insert>
```
`<selectKey>`标签中属性说明如下：
* `keyProperty`：用于指定主键值对应的`PO`类的属性。
* `order`：该属性取值可以为`BEFORE`或`AFTER`。`BEFORE`表示先执行`<selectKey>`标签内的语句，再执行插入语句；`AFTER`表示先执行插入语句再执行`<selectKey>`标签内的语句。

## update标签
`update`标签用于定义更新语句，执行更新操作。当 MyBatis 执行完一条更新语句后，会返回一个整数，表示受影响的数据库记录的行数。

1. 在`WebsiteMapper.xml`中添加以下更新语句。
```xml
<!--update 标签-->
<update id="updateWebsite" parameterType="string">
    update website set name = #{name}
</update>
```
2. 在`WebsiteMapper`接口中增加一个`updateWebsite()`方法。
```java
int updateWebsite(String name);
```
参数为`String`类型的字符串；返回值为`int`类型，表示执行`sql`语句后受影响的记录的行数。

3. 测试代码如下。
```java
int i = websiteMapper.updateWebsite("baidu");
System.out.println("共更新了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
共更新了 9 条记录
```

### update 标签常用属性

| 属性名称	    | 描述 | 备注 |
| :--: | :--: | :--: |
| id            | 它和 Mapper 的命名空间组合起来使用，是唯一标识符，供 MyBatis 调用                           | 如果命名空间+ id 不唯一，那么 MyBatis 抛出异常 |
| parameterType	| 传入 SQL 语句的参数类型的全限定名或别名，它是一个可选属性。                                 | 支持基本数据类型和 JavaBean、Map 等复杂数据类型 |
| flushCache    | 该属性用于设置执行该操作后，是否会清空二级缓存和本地缓存，默认值为 true。                   |    |
| timeout       | 该属性用于设置 SQL 执行的超时时间，如果超时，就抛异常。                                     |  |
| statementType	| 执行 SQL 时使用的 statement 类型, 默认为 PREPARED，可选值：STATEMENT，PREPARED 和 CALLABLE。| |

> `update`标签中没有`resultType`属性，只有查询操作才需要对返回结果类型进行相应的指定。

### 传递多个参数
Mybatis 为我们提供以下 3 种方式，来实现给映射器传递多个参数：
* 使用`Map`传递参数
* 使用注解传递参数
* 使用`JavaBean`传递参数

#### 使用 Map 传递参数
我们可以将参数封装到一个`Map`对象中，然后传递给 MyBatis 的映射器。

1. 在`WebsiteMapper`接口中，定义一个`updateWebsiteByMap()`方法，并使用`Map`传递参数。
```java
int updateWebsiteByMap(Map<String, Object> params);
```
2. 在`WebsiteMapper.xml`使用`update`标签定义一个`update`语句，并接收通过`Map`传递的参数。
```xml
<!--更新语句接收 Map 传递的参数-->
<update id="updateWebsiteByMap" parameterType="map">
  update website set name = #{name},url= #{url} where id = #{id}
</update>
```
3. 测试代码如下。
```java
//使用 Map 向 update 标签传递参数
Map<String, Object> params = new HashMap<>();
params.put("id", 1);
params.put("name", "baidu");
params.put("url", "www.baidu.com");
int i = websiteMapper.updateWebsiteByMap(params);
System.out.println("通过 Map 传递参数，共更新了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 Map 传递参数，共更新了 1 条记录
```

#### 使用注解传递参数
我们还可以使用 MyBatis 提供的`@Param`注解给注解器传递参数。

1. 在`WebsiteMapper`接口中，定义一个`updateWebsiteByParam()`方法，并使用`@Param`注解传递参数。
```java
int updateWebsiteByParam(@Param("name") String name, @Param("url") String url, @Param("id") Integer id);
```
2. 在`WebsiteMapper.xml`中使用`update`标签定义一个`update`语句，并接收通过`@Param`注解传递的参数。
```xml
<!--更新语句接收 @Param 注解传递的参数-->
<update id="updateWebsiteByParam">
  update website set name = #{name},url= #{url} where id = #{id}
</update>
```
3. 测试代码如下。
```java
// 使用@Param 注解传递参数到更新语句中
String name = "编程帮2";
String url = "www.biancheng.net";
Integer id = 2;
int i = websiteMapper.updateWebsiteByParam(name, url, id);
System.out.println("通过 @Param 注解传递参数，共更新了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 @Param 注解传递参数，共更新了 1 条记录
```

#### 使用 JavaBean 传递参数
在参数过多的情况下，我们还可以将参数通过`setter`方法封装到`JavaBean`（实体类）对象中传递给映射器。

1. 在`WebsiteMapper`接口中，定义一个`updateWebsiteByJavaBean()`方法，并使用`JavaBean`传递参数。
```java
int updateWebsiteByJavaBean(Website website);
```
2. 在`WebsiteMapper.xml`中使用`update`标签定义一个`update`语句，并接收通过`JavaBean`传递的参数。
```xml
<!--更新语句接收 JavaBean 传递的参数-->
<update id="updateWebsiteByJavaBean" parameterType="net.biancheng.www.po.Website">
  update website set name = #{name},url= #{url} where id = #{id}
</update>
```
3. 测试代码如下。
```java
//使用 JavaBean 传递参数到更新语句中
Website website = new Website();
website.setName("baidu");
website.setUrl("www.baidu.com");
website.setId(3);
int i = websiteMapper.updateWebsiteByJavaBean(website);
System.out.println("通过 JavaBean 传递参数，共更新了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 JavaBean 传递参数，共更新了 1 条记录
```

#### 区别
以上 3 种方式的区别如下：
* 使用`Map`传递参数会导致业务可读性的丧失，继而导致后续扩展和维护的困难，所以在实际应用中我们应该果断废弃该方式。
* 使用`@Param`注解传递参数会受到参数个数的影响。当`n≤5`时，它是最佳的传参方式，因为它更加直观；当`n>5`时，多个参数将给调用带来困难。
* 当参数个数大于 5 个时，建议使用`JavaBean`方式。

## delete标签
`delete`标签用于定义`delete`语句，执行删除操作。当 MyBatis 执行完一条更新语句后，会返回一个整数，表示受影响的数据库记录的行数。

1. 在`WebsiteMapper.xml`中使用`delete`标签添加一条`delete`语句。
```xml
<delete id="deleteWebsite" parameterType="string">
  delete from website where name = #{name}
</delete>
```
2.  在`WebsiteMapper`接口中增加一个`deleteWebsite()`方法。
```java
int deleteWebsite(String name);
```
参数为`String`类型的字符串；返回值为`int`类型，表示执行`sql`语句后，被删除记录的行数。
3. 测试代码如下。
```java
//删除 name 为编程帮3 的记录
String name = "编程帮3";
int i = websiteMapper.deleteWebsite(name);
System.out.println("共删除了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
共删除了 3 条记录
```

### delete 标签常用属性

| 属性名称	     | 描述 | 备注 |
| :--: | :--: | :--: |
| id            | 它和 Mapper 的命名空间组合起来使用，是唯一标识符，供 MyBatis 调用 | 如果命名空间+ id 不唯一，那么 MyBatis 抛出异常 |
| parameterType	| 传入 SQL 语句的参数类型的全限定名或别名，它是一个可选属性。 | 支持基本数据类型和 JavaBean、Map 等复杂数据类型 |
| flushCache    | 该属性用于设置执行该操作后，是否会清空二级缓存和本地缓存，默认值为 true。 | |
| timeout       | 该属性用于设置 SQL 执行的超时时间，如果超时，就抛异常。 | |
| statementType	| 执行 SQL 时使用的 statement 类型, 默认为 PREPARED，可选值：STATEMENT，PREPARED 和 CALLABLE。  | |

> `delete`标签中没有`resultType`属性，只有查询操作才需要对返回结果类型进行相应的指定。

### 传递多个参数
Mybatis 为我们提供以下 3 种方式，来实现给映射器传递多个参数：
* 使用`Map`传递参数
* 使用注解传递参数
* 使用`JavaBean`传递参数

#### 使用 Map 传递参数
我们可以将参数封装到一个`Map`对象中，然后传递给 MyBatis 的映射器。

1. 在`WebsiteMapper`接口中，定义一个`deleteWebsiteByMap()`方法，并使用`Map`传递参数。
```java
int deleteWebsiteByMap(Map<String, Object> params);
```
2. 在`WebsiteMapper.xml`中使用`delete`标签定义一个`delete`语句，并接收通过`Map`传递的参数。
```xml
<!--通过 Map 传递参数，执行删除操作-->
<delete id="deleteWebsiteByMap" parameterType="map">
  delete from website where name = #{name} and url = #{url}
</delete>
```
3. 测试代码如下。
```java
//使用 Map 向 delete 标签传递参数
Map<String, Object> params = new HashMap<>();
params.put("name", "baidu");
params.put("url", "www.baidu.com");
int i = websiteMapper.deleteWebsiteByMap(params);
System.out.println("通过 Map 传递参数，共删除了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 Map 传递参数，共删除了 1 条记录
```

#### 使用注解传递参数
我们还可以使用 MyBatis 提供的`@Param`注解给注解器传递参数。

1. 在`WebsiteMapper`接口中，定义一个`deleteWebsiteByParam()`方法，并使用`@Param`注解传递参数。
```java
int deleteWebsiteByParam(@Param("name") String name, @Param("url") String url);
```
2. 在`WebsiteMapper.xml`中使用`delete`标签定义一个`delete`语句，并接收通过`@Param`注解传递的参数。
```xml
<!--通过 @Param 注解传递参数，执行删除操作-->
<delete id="deleteWebsiteByParam">
    delete from website where name = #{name} and url = #{url}
</delete>
```
3. 测试代码如下。
```java
//使用 @Param 注解传递参数
String name = "baidu";
String url = "www.baidu.com";
int i = websiteMapper.deleteWebsiteByParam(name, url);
System.out.println("通过 @Param 注解传递参数，共删除了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 @Param 注解传递参数，共删除了 1 条记录
```

#### 使用 JavaBean 传递参数
在参数过多的情况下，我们还可以将参数通过`setter`方法封装到`JavaBean`（实体类）对象中传递给映射器。

1. 在`WebsiteMapper`接口中,定义一个`deleteWebsiteByJavaBean()`方法，并使用`JavaBean`传递参数。
```
int deleteWebsiteByJavaBean(Website website);
```
2. 在`WebsiteMapper.xml`中使用`delete`标签定义一个`delete`语句，并接收通过`JavaBean`传递的参数。
```xml
<!--通过 JavaBean 传递参数，执行删除操作-->
<delete id="deleteWebsiteByJavaBean" parameterType="net.biancheng.www.po.Website">
  delete from website where name = #{name} and url = #{url}
</delete>
```
3. 测试代码如下。
```java
//使用 JavaBean 传递参数到更新语句中
Website website = new Website();
website.setName("baidu");
website.setUrl("https://www.baidu.com/");
int i = websiteMapper.deleteWebsiteByJavaBean(website);
System.out.println("通过 JavaBean 传递参数，共删除了 " + i + " 条记录");
```
4. 执行测试代码，控制台输出如下。
```
通过 JavaBean 传递参数，共删除了 3 条记录
```

#### 区别
以上 3 种方式的区别如下：
* 使用`Map`传递参数会导致业务可读性的丧失，继而导致后续扩展和维护的困难，所以在实际应用中我们应该果断废弃该方式。
* 使用`@Param`注解传递参数会受到参数个数的影响。当`n≤5`时，它是最佳的传参方式，因为它更加直观；当`n>5`时，多个参数将给调用带来困难。
* 当参数个数大于 5 个时，建议使用`JavaBean`方式。

## sql 标签
`<sql>`标签的作用在于可以定义 SQL 语句的一部分（代码片段），以方便后面的 SQL 语句引用它，例如反复使用的列名。

在 MyBatis 中只需使用`<sql>`标签编写一次便能在其他元素中引用它。
```xml
<sql id="comColumns">id,uname,usex</sql>
<select id="selectUser" resultType="com.po.MyUser">
  select <include refid="comColumns"> from user
</select>
```
## resultMap元素
`resultMap`主要用于解决实体类属性名与数据库表中字段名不一致的情况，可以将查询结果映射成实体对象。

现有的 MyBatis 版本只支持`resultMap`查询，不支持更新或者保存，更不必说级联的更新、删除和修改。
### resultMap元素的构成
`resultMap`元素还可以包含以下子元素。
```xml
<resultMap id="" type="">
  <constructor><!-- 类再实例化时用来注入结果到构造方法 -->
    <idArg/><!-- ID参数，结果为ID -->
    <arg/><!-- 注入到构造方法的一个普通结果 --> 
  </constructor>
  <id/><!-- 用于表示哪个列是主键 -->
  <result/><!-- 注入到字段或JavaBean属性的普通结果 -->
  <association property=""/><!-- 用于一对一关联 -->
  <collection property=""/><!-- 用于一对多、多对多关联 -->
  <discriminator javaType=""><!-- 使用结果值来决定使用哪个结果映射 -->
    <case value=""/><!-- 基于某些值的结果映射 -->
  </discriminator>
</resultMap>
```
其中：
* `<resultMap>`元素的`type`属性表示需要的 POJO，`id`属性是`resultMap`的唯一标识。
* 子元素`<constructor>`用于配置构造方法。当一个 POJO 没有无参数构造方法时使用。
* 子元素`<id>`用于表示哪个列是主键。允许多个主键，多个主键称为联合主键。
* 子元素`<result>`用于表示 POJO 和 SQL 列名的映射关系。
* 子元素`<association>、<collection>`和`<discriminator>`用在级联的情况下。

`id`和`result`元素都有以下属性。

| 元素	| 说明 |
| :--: | :--: |
| property	| 映射到列结果的字段或属性。如果 POJO 的属性和 SQL 列名（column元素）是相同的，那么 MyBatis 就会映射到 POJO 上 |
| column    | 对应 SQL 列 |
| javaType	| 配置 Java 类型。可以是特定的类完全限定名或 MyBatis 上下文的别名 |
| jdbcType	| 配置数据库类型。这是 JDBC 类型，MyBatis 已经为我们做了限定，基本支持所有常用数据库类型 |
| typeHandler	| 类型处理器。允许你用特定的处理器来覆盖 MyBatis 默认的处理器。需要指定 jdbcType 和 javaType 相互转化的规则 |

一条 SQL 查询语句执行后会返回结果集，结果集有两种存储方式，即使用`Map`存储和使用 POJO 存储。
### 使用Map存储结果集
任何`select`语句都可以使用`Map`存储。
```xml
<!-- 查询所有网站信息存到Map中 -->
<select id="selectAllWebsite" resultType="map">
  select * from website
</select>
```
在`WebsiteMapper`接口中添加以下方法。
```java
public List<Map<String,Object>> selectAllWebsite();
```
`Map`的`key`是`select`语句查询的字段名（必须完全一样），而`Map`的`value`是查询返回结果中字段对应的值，一条记录映射到一个`Map`对象中。

使用`Map`存储结果集很方便，但可读性稍差，所以一般推荐使用 POJO 的方式。
### 使用POJO存储结果集
因为 MyBatis 提供了自动映射，所以使用 POJO 存储结果集是最常用的方式。但有时候需要更加复杂的映射或级联，这时就需要使用`select`元素的`resultMap`属性配置映射集合。

修改`Website`类，代码如下。
```java
package net.biancheng.po;
import java.util.Date;
public class Website {
  private int id;
  private String uname;
  private String url;
  private int age;
  private String country;
  private Date createtime;
  /* setter和getter方法*/
  @Override
  public String toString() {
    return "Website[id=" + id + ",uname=" + uname + ",url=" + url + ",age=" + age + ",country=" + country
          + ",createtime=" + createtime + "]";
  }
}
```
`WebsiteMapper.xml`代码如下。
```xml
<!--使用自定义结果集类型 -->
<resultMap type="net.biancheng.po.Website" id="myResult">
    <!-- property 是 net.biancheng.po.Website 类中的属性 -->
    <!-- column是查询结果的列名，可以来自不同的表 -->
    <id property="id" column="id" />
    <result property="uname" column="name" />
</resultMap>
```
`resultMap`元素的属性`id`代表这个`resultMap`的标识，`type`标识需要映射的 POJO。我们可以使用 MyBatis 定义好的类的别名或自定义类的全限定名。

这里使用`property`元素指定`Website`的属性名称`uname，column`表示数据库中`website`表的 SQL 列名`name`，将 POJO 和 SQL 的查询结果一 一对应。

`WebsiteMapper.xml`中`select`元素配置代码如下。
```xml
<select id="selectAllWebsite" resultMap="myResult">
  select id,name,url from website
</select>
```
可以发现 SQL 语句的列名和`myResult`中的`column`一一对应。
### resultType和resultMap的区别
MyBatis 的每一个查询映射的返回类型都是`resultMap`，只是当我们提供的返回类型是`resultType`时，MyBatis 会自动把对应的值赋给`resultType`所指定对象的属性，而当我们提供的返回类型是`resultMap`时，MyBatis 会将数据库中的列数据复制到对象的相应属性上，可用于复制查询。

需要注意的是，`resultMap`和`resultType`不能同时使用。