---
title: MyBatis 动态SQL
date: 2021-04-012 16:24:15
tags: [MyBatis]
categories: [MyBatis]
---

开发人员通常根据需求手动拼接 SQL 语句，这是一个极其麻烦的工作，而 MyBatis 提供了对 SQL 语句动态组装的功能，恰能解决这一问题。

MyBatis 的动态 SQL 元素与 JSTL 或 XML 文本处理器相似，常用`<if>、<choose>、<when>、<otherwise>、<trim>、<where>、<set>、<foreach>`和`<bind>`等元素。
# if
使用动态 SQL 最常见情景是根据条件包含`where`子句的一部分。
```xml
<select id="selectAllWebsite" resultMap="myResult">
  select id,name,url from website
  <if test="name != null">
    where name like #{name}
  </if>
</select>
```
以上代表表示根据网站名称去查找相应的网站信息，但是网站名称是一个可填可不填的条件，不填写的时候不作为查询条件。

可多个`if`语句同时使用。以下语句表示为可以按照网站名称（`name`）或者网址（`url`）进行模糊查询。如果您不输入名称或网址，则返回所有的网站记录。但是，如果你传递了任意一个参数，它就会返回与给定参数相匹配的记录。
```xml
<select id="selectAllWebsite" resultMap="myResult">
  select id,name,url from website where 1=1
  <if test="name != null">
    AND name like #{name}
  </if>
  <if test="url!= null">
    AND url like #{url}
  </if>
</select>
```
# choose、when、otherwise
动态语句`choose-when-otherwise`类似于 Java 中的`switch-case-default`语句。由于 MyBatis 并没有为`if`提供对应的`else`标签，如果想要达到`<if>...<else>...</else> </if>`的效果，可以借助`<choose>、<when>、<otherwise>`来实现。
```xml
<choose>
  <when test="判断条件1">
    SQL语句1
  </when >
  <when test="判断条件2">
    SQL语句2
  </when >
  <when test="判断条件3">
    SQL语句3
  </when >
  <otherwise>
    SQL语句4
  </otherwise>
</choose>
```
`choose`标签按顺序判断其内部`when`标签中的判断条件是否成立，如果有一个成立，则执行相应的 SQL 语句，`choose`执行结束；如果都不成立，则执行`otherwise`中的 SQL 语句。这类似于 Java 的`switch`语句，`choose`为`switch`，`when`为`case`，`otherwise`则为`default`。
## 示例
以下示例要求：
* 当网站名称不为空时，只用网站名称作为条件进行模糊查询；
* 当网站名称为空，而网址不为空时，则用网址作为条件进行模糊查询；
* 当网站名称和网址都为空时，则要求网站年龄不为空。

`WebsiteMapper.xml`代码如下。
```xml
<mapper namespace="net.biancheng.mapper.WebsiteMapper">
  <select id="selectWebsite" parameterType="net.biancheng.po.Website" resultType="net.biancheng.po.Website">
    SELECT id,name,url,age,country FROM website WHERE 1=1
    <choose>
      <when test="name != null and name !=''">
        AND name LIKE CONCAT('%',#{name},'%')
      </when>
      <when test="url != null and url !=''">
        AND url LIKE CONCAT('%',#{url},'%')
      </when>
      <otherwise>
        AND age is not null
      </otherwise>
    </choose>
  </select>
</mapper>
```
`WebsiteMapper`类中方法如下。
```java
public List<Website> selectWebsite(Website website);
```
测试类代码如下。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    // 读取配置文件mybatis-config.xml
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml"); // 根据配置文件构建
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    // 通过SqlSessionFactory创建SqlSession
    SqlSession ss = ssf.openSession();
    Website site = new Website();
    site.setname("编程");
    List<Website> siteList = ss.selectList("net.biancheng.mapper.WebsiteMapper.selectWebsite", site);
    for (Website ws : siteList) {
      System.out.println(ws);
    }
  }
}
```
输出结果如下。
```
DEBUG [main] - ==>  Preparing: SELECT id,name,url,age,country FROM website WHERE 1=1 AND name LIKE CONCAT('%',?,'%')
DEBUG [main] - ==> Parameters: 编程(String)
DEBUG [main] - <==      Total: 1
Website[id=1,name=百度,url=https://www.baidu.com/,age=10,country=CN,createtime=null]
```
# where
```xml
<mapper namespace="net.biancheng.mapper.WebsiteMapper">
  <select id="selectWebsite" parameterType="net.biancheng.po.Website" resultType="net.biancheng.po.Website">
    SELECT id,name,url,age,country FROM website WHERE 1=1
    <choose>
      <when test="name != null and name !=''">
        AND name LIKE CONCAT('%',#{name},'%')
      </when>
      <when test="url != null and url !=''">
        AND url LIKE CONCAT('%',#{url},'%')
      </when>
      <otherwise>
        AND age is not null
      </otherwise>
    </choose>
  </select>
</mapper>
```
上面的 SQL 语句中加入了一个条件`1=1`，如果没有加入这个条件，那么可能就会变成下面这样一条错误的语句。
```sql
SELECT id,name,url,age,country FROM website AND name LIKE CONCAT('%',#{name},'%')
```
显然以上语句会出现 SQL 语法异常，但加入`1=1`这样的条件又非常奇怪，所以 MyBatis 提供了`where`标签。

`where`标签主要用来简化 SQL 语句中的条件判断，可以自动处理`AND/OR`条件。
```xml
<where>
  <if test="判断条件">
    AND/OR ...
  </if>
</where>
```
`if`语句中判断条件为`true`时，`where`关键字才会加入到组装的 SQL 里面，否则就不加入。`where`会检索语句，它会将`where`后的第一个 SQL 条件语句的`AND`或者`OR`关键词去掉。
## 示例
根据网站名称或网址对网站进行模糊查询。
```xml
<select id="selectWebsite" resultType="net.biancheng.po.Website">
  select id,name,url from website
  <where>
    <if test="name != null">
      AND name like #{name}
    </if>
    <if test="url!= null">
      AND url like #{url}
    </if>
  </where>
</select>
```
# trim
在 MyBatis 中除了使用`if+where`实现多条件查询，还有一个更为灵活的元素`trim`能够替代之前的做法。

`trim`一般用于去除 SQL 语句中多余的`AND`关键字、逗号，或者给 SQL 语句前拼接`where、set`等后缀，可用于选择性插入、更新、删除或者条件查询等操作。
```xml
<trim prefix="前缀" suffix="后缀" prefixOverrides="忽略前缀字符" suffixOverrides="忽略后缀字符">
  SQL语句
</trim>
```
`trim`中属性说明如下。

| 属性 | 描述 |
| :--: | :--: |
| prefix	        | 给 SQL 语句拼接的前缀，为 trim 包含的内容加上前缀 |
| suffix	        | 给 SQL 语句拼接的后缀，为 trim 包含的内容加上后缀 |
| prefixOverrides	| 去除 SQL 语句前面的关键字或字符，该关键字或者字符由 prefixOverrides 属性指定。 |
| suffixOverrides	| 去除 SQL 语句后面的关键字或者字符，该关键字或者字符由 suffixOverrides 属性指定。 |

```xml
<select id="selectWebsite" resultType="net.biancheng.po.Website">
  SELECT id,name,url,age,country FROM website
  <trim prefix="where" prefixOverrides="and">
    <if test="name != null and name !=''">
      AND name LIKE CONCAT ('%',#{name},'%')
    </if>
    <if test="url!= null">
      AND url like concat ('%',#{url},'%')
    </if>
  </trim>
</select>
```
# set
`update`语句可以使用`set`标签动态更新列。`set`标签可以为 SQL 语句动态的添加`set`关键字，剔除追加到条件末尾多余的逗号。
## 示例
根据`id`修改网站名称或网址。
```xml
<!--使用set元素动态修改一个网站记录 -->
<update id="updateWebsite" parameterType="net.biancheng.po.Website">
  UPDATE website
  <set>
    <if test="name!=null">name=#{name}</if>
    <if test="url!=null">url=#{url}</if>
  </set>
  WHERE id=#{id}
</update>
```
# foreach
对于一些 SQL 语句中含有`in`条件，需要迭代条件集合来生成的情况，可以使用`foreach`来实现 SQL 条件的迭代。 

`foreach`标签用于循环语句，它很好的支持了数据和`List、set`接口的集合，并对此提供遍历的功能。
```xml
<foreach item="item" index="index" collection="list|array|map key" open="(" separator="," close=")">
  参数值
</foreach>
```
`foreach`标签主要有以下属性：
* `item`：表示集合中每一个元素进行迭代时的别名。
* `index`：指定一个名字，表示在迭代过程中每次迭代到的位置。
* `open`：表示该语句以什么开始（既然是`in`条件语句，所以必然以(开始）。
* `separator`：表示在每次进行迭代之间以什么符号作为分隔符（既然是`in`条件语句，所以必然以,作为分隔符）。
* `close`：表示该语句以什么结束（既然是`in`条件语句，所以必然以)开始）。

使用`foreach`标签时，最关键、最容易出错的是`collection`属性，该属性是必选的，但在不同情况下该属性的值是不一样的，主要有以下 3 种情况：
* 如果传入的是单参数且参数类型是一个`List`，`collection`属性值为`list`。
* 如果传入的是单参数且参数类型是一个`array`数组，`collection`的属性值为`array`。
* 如果传入的参数是多个，需要把它们封装成一个`Map`，当然单参数也可以封装成`Map`。`Map`的`key`是参数名，`collection`属性值是传入的`List`或`array`对象在自己封装的`Map`中的`key`。

## 示例
现有`website`表包含以下记录。
```
+----+----------------+----------------------------+-----+---------+---------------------+
| id | name           | url                        | age | country | createtime          |
+----+----------------+----------------------------+-----+---------+---------------------+
|  1 | 百度           | https://www.baidu.com/     |  18 | CN      | 2021-03-08 11:23:53 |
|  2 | 淘宝           | https://www.taobao.com/    |  17 | CN      | 2021-03-10 10:33:54 |
|  3 | Google         | https://www.google.com/    |  23 | US      | 2021-03-10 10:34:34 |
|  4 | GitHub         | https://github.com/        |  13 | US      | 2021-03-10 10:34:34 |
|  5 | Stack Overflow | https://stackoverflow.com/ |  16 | US      | 2021-03-10 10:34:34 |
|  6 | Yandex         | http://www.yandex.ru/      |  11 | RU      | 2021-03-10 10:34:34 |
+----+----------------+----------------------------+-----+---------+---------------------+
```
`WebsiteMapper.xml`中代码如下。
```xml
<select id="selectWebsite" parameterType="net.biancheng.po.Website" resultType="net.biancheng.po.Website">
  SELECT id,name,url,age,country FROM website WHERE age in
  <foreach item="age" index="index" collection="list" open="(" separator="," close=")">
    #{age}
  </foreach>
</select>
```
`WebsiteMapper`类中相应方法如下。
```java
public List<Website> selectWebsite(List<Integer> ageList);
```
测试代码如下。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    // 读取配置文件mybatis-config.xml
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml"); // 根据配置文件构建
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    // 通过SqlSessionFactory创建SqlSession
    SqlSession ss = ssf.openSession();
    List<Integer> ageList = new ArrayList<Integer>();
    ageList.add(17);
    ageList.add(18);
    List<Website> siteList = ss.selectList("net.biancheng.mapper.WebsiteMapper.selectWebsite", ageList);
    for (Website ws : siteList) {
      System.out.println(ws);
    }
  }
}
```
# script
要在带注解的映射器接口类中使用动态 SQL，可以使用`script`元素。
```java
@Update({"<script>",
  "update Author",
  "  <set>",
  "    <if test='username != null'>username=#{username},</if>",
  "    <if test='password != null'>password=#{password},</if>",
  "    <if test='email != null'>email=#{email},</if>",
  "    <if test='bio != null'>bio=#{bio}</if>",
  "  </set>",
  "where id=#{id}",
  "</script>"})
void updateAuthorValues(Author author);
```
# bind
每个数据库的拼接函数或连接符号都不同，例如 MySQL 的`concat`函数、Oracle 的连接符号`||`等。这样 SQL 映射文件就需要根据不同的数据库提供不同的实现，显然比较麻烦，且不利于代码的移植。MyBatis 提供了`bind`标签来解决这一问题。

`bind`标签可以通过 OGNL 表达式自定义一个上下文变量。

比如，按照网站名称进行模糊查询，SQL 映射文件如下。
```xml
<select id="selectWebsite" resultType="net.biancheng.po.Website">
  <bind name="pattern" value="'%'+_parameter+'%'" />
  SELECT id,name,url,age,country
  FROM website
  WHERE name like #{pattern}
</select>
```
`bind`元素属性如下。
* `value`：对应传入实体类的某个字段，可以进行字符串拼接等特殊处理。
* `name`：给对应参数取的别名。

以上代码中的`_parameter`代表传递进来的参数，它和通配符连接后，赋给了`pattern`，然后就可以在`select`语句中使用这个变量进行模糊查询，不管是 MySQL 数据库还是 Oracle 数据库都可以使用这样的语句，提高了可移植性。

大部分情况下需要传递多个参数，下面为传递多个参数时`bind`的用法示例。
## 示例
`WebsiteMapper`类中方法代码如下。
```
public List<Website> selectWebsite(Website site);
```
SQL 映射文件代码如下。
```xml
<select id="selectWebsite" resultType="net.biancheng.po.Website">
  <bind name="pattern_name" value="'%'+name+'%'" />
  <bind name="pattern_url" value="'%'+url+'%'" />
  SELECT id,name,url,age,country
  FROM website
  WHERE name like #{pattern_name}
  AND url like #{pattern_url}
</select>
```
测试代码如下。
```java
public class Test {
  public static void main(String[] args) throws IOException {
    // 读取配置文件mybatis-config.xml
    InputStream config = Resources.getResourceAsStream("mybatis-config.xml"); // 根据配置文件构建
    SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(config);
    // 通过SqlSessionFactory创建SqlSession
    SqlSession ss = ssf.openSession();
    Website site = new Website();
    site.setname("编程");
    site.setUrl("http");
    List<Website> siteList = ss.selectList("net.biancheng.mapper.WebsiteMapper.selectWebsite", site);
    for (Website ws : siteList) {
      System.out.println(ws);
    }
  }
}
```
运行结果如下。
```
DEBUG [main] - ==>  Preparing: SELECT id,name,url,age,country FROM website WHERE name like ? AND url like ?
DEBUG [main] - ==> Parameters: %编程%(String), %http%(String)
DEBUG [main] - <==      Total: 1
Website[id=1,name=百度,url=https://www.baidu.com/,age=10,country=CN]
```
# 多数据库支持
如果配置了`databaseIdProvider`，你就可以在动态代码中使用名为`_databaseId`的变量来为不同的数据库构建特定的语句。
```xml
<insert id="insert">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    <if test="_databaseId == 'oracle'">
      select seq_users.nextval from dual
    </if>
    <if test="_databaseId == 'db2'">
      select nextval for seq_users from sysibm.sysdummy1"
    </if>
  </selectKey>
  insert into users values (#{id}, #{name})
</insert>
```