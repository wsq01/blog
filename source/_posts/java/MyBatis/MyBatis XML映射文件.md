---
title: MyBatis 映射文件
date: 2021-04-12 15:56:16
tags: [MyBatis]
categories: [MyBatis]
---


# XML 映射器
SQL 映射文件的几个顶级元素（按照应被定义的顺序列出）：

`cache`：该命名空间的缓存配置。
`cache-ref`：引用其它命名空间的缓存配置。
`resultMap`：描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
`sql`：可被其它语句引用的可重用语句块。
`insert`：映射插入语句。
`update`：映射更新语句。
`delete`：映射删除语句。
`select`：映射查询语句。

# select元素
在 SQL 映射文件中`<select>`元素用于映射 SQL 的`select`语句。
```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```
这个语句名为`selectPerson`，接受一个`int`（或`Integer`）类型的参数，并返回一个`HashMap`类型的对象，其中的键是列名，值便是结果行中的对应值。

注意参数符号：`#{id}`。这就告诉 MyBatis 创建一个预处理语句（`PreparedStatement`）参数，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：
```java
// 近似的 JDBC 代码，非 MyBatis 代码...
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```
`<select>`元素常用的属性：

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

## 使用 Map 接口传递多个参数
在实际开发中，查询 SQL 语句经常需要多个参数，例如多条件查询。当传递多个参数时，`<select>`元素的`parameterType`属性值的类型是什么呢？在 MyBatis 中允许`Map`接口通过键值对传递多个参数。

假设数据操作接口中有个实现查询陈姓男性用户信息功能的方法：
```java
public List<MyUser> selectAllUser(Map<String,Object> param);
```
此时，传递给映射器的是一个 Map 对象，使用它在 SQL 文件中设置对应的参数，对应 SQL 文件的代码如下：
```xml
<!-- 查询陈姓男性用户信息 -->
<select id="selectAllUser" resultType="com.mybatis.po.MyUser">
    select * from user
    where uname like concat('%',#{u_name},'%')
    and usex = #{u_sex}
</select>
```
在上述 SQL 文件中，参数名`u_name`和`u_sex`是`Map`的`key`。

`com.controller`包中`UserController`的代码片段如下：
```java
@Controller("UserController")
public class UserController {
  private UserDao userDao;
  public void test(){
    ...
    //查询多个用户
    Map<String,Object> map = new HashMap<>();
    map.put("u_name","陈");
    map.put("u_sex","男");
    List<MyUser> list = userDao.seleceAllUser(map);
    for(MyUser myUser : list) {
      System.out.println(myUser);
    }
    ...
  }
}
```
Map 是一个键值对应的集合，使用者要通过阅读它的键才能了解其作用。另外，使用`Map`不能限定其传递的数据类型，所以业务性不强，可读性较差。如果 SQL 语句很复杂，参数很多，使用`Map`将很不方便。MyBatis 还提供了使用 Java Bean 传递多个参数的形式。
## 使用 Java Bean 传递多个参数
首先在应用的`src`目录下创建一个名为`com.pojo`的包，在包中创建一个 POJO 类`SeletUserParam`：
```java
package com.pojo;
public class SeletUserParam {
  private String u_name;
  private String u_sex;
  // 此处省略setter和getter方法
}
```
接着将`Dao`接口中的`selectAllUser`方法修改为如下：
```java
public List<MyUser> selectAllUser(SelectUserParam param);
```
然后将`com.mybatis`包中的 SQL 映射文件`UserMapper.xml`中的“查询陈姓男性用户信息”的代码修改为如下：
```xml
<select id="selectAllUser" resultType="com.po.MyUser" parameterType="com.pojo.SeletUserParam">
  select * from user
  where uname like concat('%',#{u_name},'%')
  and usex=#{u_sex}
</select>
```
最后将`com.controller`包中`UserController`的“查询多个用户”的代码片段做如下修改：
```java
SeletUserParam su = new SelectUserParam();
su.setU_name("陈");
su.setU_sex("男");
List<MyUser> list = userDao.selectAllUser(su);
for (MyUser myUser : list) {
    System.out.println(myUser);
}
```
在实际应用中是选择`Map`还是选择`Java Bean`传递多个参数应根据实际情况而定，如果参数较少，建议选择`Map`；如果参数较多，建议选择`Java Bean`。
# insert 元素
`<insert>`元素用于映射插入语句，MyBatis 执行完一条插入语句后将返回一个整数表示其影响的行数。它的属性与`<select>`元素的属性大部分相同，它的几个特有属性。
* `keyProperty`：该属性的作用是将插入或更新操作时的返回值赋给`PO`类的某个属性，通常会设置为主键对应的属性。如果是联合主键，可以将多个值用逗号隔开。
* `keyColumn`：该属性用于设置第几列是主键，当主键列不是表中的第 1 列时需要设置。如果是联合主键，可以将多个值用逗号隔开。
* `useGeneratedKeys`：该属性将使 MyBatis 使用 JDBC 的`getGeneratedKeys()`方法获取由数据库内部产生的主键，例如 MySQL、SQL Server 等自动递增的字段，其默认值为`false`。

## 主键（自动递增）回填
MySQL、SQL Server 等数据库的表格可以采用自动递增的字段作为主键，有时可能需要使用这个刚刚产生的主键，用于关联其他业务。

首先为`com.mybatis`包中的 SQL 映射文件`UserMapper.xml`中`id`为`addUser`的`<insert>`元素添加`keyProperty`和`useGeneratedKeys`属性：
```xml
<!--添加一个用户，成功后将主键值返回填给uid(po的属性)-->
<insert id="addUser" parameterType="com.po.MyUser" keyProperty="uid" useGeneratedKeys="true">
  insert into user (uname,usex) values(#{uname},#{usex})
</insert>
```
然后在`com.controller`包的`UserController`类中进行调用：
```java
// 添加一个用户
MyUser addmu = new MyUser();
addmu.setUname("陈恒");
addmu.setUsex("男");
int add = userDao.addUser(addmu);
System.out.println("添加了" + add + "条记录");
System.out.println("添加记录的主键是" + addmu.getUid());
```
## 自定义主键
如果在实际工程中使用的数据库不支持主键自动递增，或者取消了主键自动递增的规则，可以使用 MyBatis 的`<selectKey>`元素来自定义生成主键。
```xml
<!-- 添加一个用户，#{uname}为 com.mybatis.po.MyUser 的属性值 -->
<insert id="insertUser" parameterType="com.po.MyUser">
  <!-- 先使用selectKey元素定义主键，然后再定义SQL语句 -->
  <selectKey keyProperty="uid" resultType="Integer" order="BEFORE">
  select if(max(uid) is null,1,max(uid)+1) as newUid from user)
  </selectKey>
  insert into user (uid,uname,usex) values(#{uid},#{uname},#{usex})
</insert>
```
在执行上述示例代码时，`<selectKey>`元素首先被执行，该元素通过自定义的语句设置数据表的主键，然后执行插入语句。

`<selectKey>`元素的`keyProperty`属性指定了新生主键值返回给`PO`类（`com.po.MyUser`）的哪个属性。
* `order`属性可以设置为`BEFORE`或`AFTER`。
* `BEFORE`表示先执行`<selectKey>`元素然后执行插入语句。
* `AFTER`表示先执行插入语句再执行`<selectKey>`元素。

# update与delete元素
`<update>`和`<delete>`元素比较简单，它们的属性和`<insert>`元素、`<select>`元素的属性差不多，执行后也返回一个整数，表示影响了数据库的记录行数。
```xml
<!-- 修改一个用户 -->
<update id="updateUser" parameterType="com.po.MyUser">
  update user set uname = #{uname},usex = #{usex} where uid = #{uid}
</update>
<!-- 删除一个用户 -->
<delete id="deleteUser" parameterType="Integer">
  delete from user where uid = #{uid}
</delete>
```
# sql 元素
`<sql>`元素的作用在于可以定义 SQL 语句的一部分（代码片段），以方便后面的 SQL 语句引用它，例如反复使用的列名。

在 MyBatis 中只需使用`<sql>`元素编写一次便能在其他元素中引用它。
```xml
<sql id="comColumns">id,uname,usex</sql>
<select id="selectUser" resultType="com.po.MyUser">
  select <include refid="comColumns"> from user
</select>
```
在上述代码中使用`<include>`元素的`refid`属性引用了自定义的代码片段。


