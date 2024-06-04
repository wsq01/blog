


# 动态SQL官方使用参考
动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。

在 MyBatis 之前的版本中，需要花时间了解大量的元素。借助功能强大的基于 OGNL 的表达式，MyBatis 3 替换了之前的大部分元素，大大精简了元素种类，现在要学习的元素种类比原来的一半还要少。
* `if`
* `choose (when, otherwise)`
* `trim (where, set)`
* `foreach`

## if
使用动态 SQL 最常见情景是根据条件包含`where`子句的一部分。比如：
```xml
<select id="findActiveBlogWithTitleLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = 'ACTIVE'
  <if test="title != null">
    AND title like #{title}
  </if>
</select>
```
这条语句提供了可选的查找文本功能。如果不传入`title`，那么所有处于`ACTIVE`状态的`BLOG`都会返回；如果传入了`title`参数，那么就会对`title`一列进行模糊查找并返回对应的`BLOG`结果。

如果希望通过`title`和`author`两个参数进行可选搜索该怎么办呢？首先，我想先将语句名称修改成更名副其实的名称；接下来，只需要加入另一个条件即可。
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = 'ACTIVE'
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```
## choose、when、otherwise
有时候，我们不想使用所有的条件，而只是想从多个条件中选择一个使用。针对这种情况，MyBatis 提供了`choose`元素，它有点像 Java 中的`switch`语句。

还是上面的例子，但是策略变为：传入了`title`就按`title`查找，传入了`author`就按`author`查找的情形。若两者都没有传入，就返回标记为`featured`的`BLOG`（这可能是管理员认为，与其返回大量的无意义随机`Blog`，还不如返回一些由管理员挑选的`Blog`）。
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG WHERE state = 'ACTIVE'
  <choose>
    <when test="title != null">
      AND title like #{title}
    </when>
    <when test="author != null and author.name != null">
      AND author_name like #{author.name}
    </when>
    <otherwise>
      AND featured = 1
    </otherwise>
  </choose>
</select>
```
## trim、where、set
前面几个例子已经合宜地解决了一个臭名昭著的动态 SQL 问题。现在回到之前的`if`示例，这次我们将`state = 'ACTIVE'`设置成动态条件，看看会发生什么。
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG
  WHERE
  <if test="state != null">
    state = #{state}
  </if>
  <if test="title != null">
    AND title like #{title}
  </if>
  <if test="author != null and author.name != null">
    AND author_name like #{author.name}
  </if>
</select>
```
如果没有匹配的条件会怎么样？最终这条 SQL 会变成这样：
```
SELECT * FROM BLOG
WHERE
```
这会导致查询失败。如果匹配的只是第二个条件又会怎样？这条 SQL 会是这样:
```
SELECT * FROM BLOG
WHERE
AND title like 'someTitle'
```
这个查询也会失败。这个问题不能简单地用条件元素来解决。

MyBatis 有一个简单且适合大多数场景的解决办法。而在其他场景中，可以对其进行自定义以符合需求。而这，只需要一处简单的改动：
```xml
<select id="findActiveBlogLike" resultType="Blog">
  SELECT * FROM BLOG
  <where>
    <if test="state != null">
        state = #{state}
    </if>
    <if test="title != null">
        AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
        AND author_name like #{author.name}
    </if>
  </where>
</select>
```
`where`元素只会在子元素返回任何内容的情况下才插入`WHERE`子句。而且，若子句的开头为`AND`或`OR`，`where`元素也会将它们去除。如果`where`元素与你期望的不太一样，你也可以通过自定义`trim`元素来定制`where`元素的功能。比如，和`where`元素等价的自定义`trim`元素为：
```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ...
</trim>
```
`prefixOverrides`属性会忽略通过管道符分隔的文本序列（注意此例中的空格是必要的）。上述例子会移除所有`prefixOverrides`属性中指定的内容，并且插入`prefix`属性中指定的内容。

用于动态更新语句的类似解决方案叫做`set`。`set`元素可以用于动态包含需要更新的列，忽略其它不更新的列。比如：
```xml
<update id="updateAuthorIfNecessary">
  update Author
    <set>
      <if test="username != null">username=#{username},</if>
      <if test="password != null">password=#{password},</if>
      <if test="email != null">email=#{email},</if>
      <if test="bio != null">bio=#{bio}</if>
    </set>
  where id=#{id}
</update>
```
这个例子中，`set`元素会动态地在行首插入`SET`关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）。来看看与`set`元素等价的自定义`trim`元素吧：
```xml
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```
注意，我们覆盖了后缀值设置，并且自定义了前缀值。
## foreach
动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建`IN`条件语句的时候）。比如：
```xml
<select id="selectPostIn" resultType="domain.blog.Post">
  SELECT * FROM POST P WHERE ID in
  <foreach item="item" index="index" collection="list" open="(" separator="," close=")">
    #{item}
  </foreach>
</select>
```
`foreach`标签主要有以下属性：
* `item`：表示集合中每一个元素进行迭代时的别名。
* `index`：指定一个名字，表示在迭代过程中每次迭代到的位置。
* `open`：表示该语句以什么开始（既然是`in`条件语句，所以必然以`(`开始）。
* `separator`：表示在每次进行迭代之间以什么符号作为分隔符（既然是`in`条件语句，所以必然以`,`作为分隔符）。
* `close`：表示该语句以什么结束（既然是`in`条件语句，所以必然以`)`结束）。

使用`foreach`标签时，最关键、最容易出错的是`collection`属性，该属性是必选的，但在不同情况下该属性的值是不一样的，主要有以下 3 种情况：
* 如果传入的是单参数且参数类型是一个`List`，`collection`属性值为`list`。
* 如果传入的是单参数且参数类型是一个`array`数组，`collection`的属性值为`array`。
* 如果传入的参数是多个，需要把它们封装成一个`Map`，当然单参数也可以封装成`Map`。`Map`的`key`是参数名，`collection`属性值是传入的`List`或`array`对象在自己封装的`Map`中的`key`。

`foreach`元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（`item`）和索引（`index`）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符。这个元素也不会错误地添加多余的分隔符。

> 提示 你可以将任何可迭代对象（如`List、Set`等）、`Map`对象或者数组对象作为集合参数传递给`foreach`。当使用可迭代对象或者数组时，`index`是当前迭代的序号，`item`的值是本次迭代获取到的元素。当使用`Map`对象（或者`Map.Entry`对象的集合）时，`index`是键，`item`是值。
## script
要在带注解的映射器接口类中使用动态 SQL，可以使用`script`元素。比如:    
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
## bind
`bind`元素允许你在 OGNL 表达式以外创建一个变量，并将其绑定到当前的上下文。比如：
```xml
<select id="selectBlogsLike" resultType="Blog">
  <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
  SELECT * FROM BLOG WHERE title LIKE #{pattern}
</select>
```
以上代码中的`_parameter`代表传递进来的参数，它和通配符连接后，赋给了`pattern`，然后就可以在`select`语句中使用这个变量进行模糊查询，不管是 MySQL 数据库还是 Oracle 数据库都可以使用这样的语句，提高了可移植性。
## 多数据库支持
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
## 动态 SQL 中的插入脚本语言
MyBatis 从 3.2 版本开始支持插入脚本语言，这允许你插入一种语言驱动，并基于这种语言来编写动态 SQL 查询语句。可以通过实现以下接口来插入一种语言：
```java
public interface LanguageDriver {
  ParameterHandler createParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql);
  SqlSource createSqlSource(Configuration configuration, XNode script, Class<?> parameterType);
  SqlSource createSqlSource(Configuration configuration, String script, Class<?> parameterType);
}
```
实现自定义语言驱动后，你就可以在`mybatis-config.xml`文件中将它设置为默认语言：
```xml
<typeAliases>
  <typeAlias type="org.sample.MyLanguageDriver" alias="myLanguage"/>
</typeAliases>
<settings>
  <setting name="defaultScriptingLanguage" value="myLanguage"/>
</settings>
```
或者，你也可以使用`lang`属性为特定的语句指定语言：
```xml
<select id="selectBlog" lang="myLanguage">
  SELECT * FROM BLOG
</select>
```
或者，在你的`mapper`接口上添加`@Lang`注解：
```java
public interface Mapper {
  @Lang(MyLanguageDriver.class)
  @Select("SELECT * FROM BLOG")
  List<Blog> selectBlog();
}
```
前面看到的所有`xml`标签都由默认 MyBatis 语言提供，而它由语言驱动`org.apache.ibatis.scripting.xmltags.XmlLanguageDriver`（别名为`xml`）所提供。
# 动态SQL解析原理
我们在使用 mybatis 的时候，会在`xml`中编写`sql`语句。比如这段动态`sql`代码：
```xml
<update id="update" parameterType="org.format.dynamicproxy.mybatis.bean.User">
    UPDATE users
    <trim prefix="SET" prefixOverrides=",">
        <if test="name != null and name != ''">
            name = #{name}
        </if>
        <if test="age != null and age != ''">
            , age = #{age}
        </if>
        <if test="birthday != null and birthday != ''">
            , birthday = #{birthday}
        </if>
    </trim>
    where id = ${id}
</update>
```
mybatis 底层是如何构造这段`sql`的？下面带着这个疑问，我们一步一步分析。
## 关于动态SQL的接口和类
`SqlNode`接口，简单理解就是`xml`中的每个标签，比如上述`sql`的`update,trim,if`标签：
```java
public interface SqlNode {
    boolean apply(DynamicContext context);
}
```
`SqlSource Sql`源接口，代表从`xml`文件或注解映射的`sql`内容，主要就是用于创建`BoundSql`，有实现类`DynamicSqlSource`(动态`Sql`源)，`StaticSqlSource`(静态`Sql`源)等：
```java
public interface SqlSource {
    BoundSql getBoundSql(Object parameterObject);
}
```
`BoundSql`类，封装 mybatis 最终产生`sql`的类，包括`sql`语句，参数，参数源数据等参数：


XNode，一个Dom API中的Node接口的扩展类：


BaseBuilder接口及其实现类(属性，方法省略了，大家有兴趣的自己看),这些Builder的作用就是用于构造sql：


下面我们简单分析下其中4个Builder：
XMLConfigBuilder：解析mybatis中configLocation属性中的全局xml文件，内部会使用XMLMapperBuilder解析各个xml文件。
XMLMapperBuilder：遍历mybatis中mapperLocations属性中的xml文件中每个节点的Builder，比如user.xml，内部会使用XMLStatementBuilder处理xml中的每个节点。
XMLStatementBuilder：解析xml文件中各个节点，比如select,insert,update,delete节点，内部会使用XMLScriptBuilder处理节点的sql部分，遍历产生的数据会丢到Configuration的mappedStatements中。
XMLScriptBuilder：解析xml中各个节点sql部分的Builder。

LanguageDriver接口及其实现类(属性，方法省略了，大家有兴趣的自己看)，该接口主要的作用就是构造sql:

简单分析下XMLLanguageDriver(处理xml中的sql，RawLanguageDriver处理静态sql)：XMLLanguageDriver内部会使用XMLScriptBuilder解析xml中的sql部分。
## 源码分析
Spring与Mybatis整合的时候需要配置SqlSessionFactoryBean，该配置会加入数据源和mybatis xml配置文件路径等信息：
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatisConfig.xml"/>
    <property name="mapperLocations" value="classpath*:org/format/dao/*.xml"/>
</bean>
```
我们就分析这一段配置背后的细节：

SqlSessionFactoryBean实现了Spring的InitializingBean接口，InitializingBean接口的afterPropertiesSet方法中会调用buildSqlSessionFactory方法 该方法内部会使用XMLConfigBuilder解析属性configLocation中配置的路径，还会使用XMLMapperBuilder属性解析mapperLocations属性中的各个xml文件。部分源码如下：


由于XMLConfigBuilder内部也是使用XMLMapperBuilder，我们就看看XMLMapperBuilder的解析细节：


我们关注一下，增删改查节点的解析：


XMLStatementBuilder的解析：


默认会使用XMLLanguageDriver创建SqlSource（Configuration构造函数中设置）。

XMLLanguageDriver创建SqlSource：


XMLScriptBuilder解析sql：

得到SqlSource之后，会放到Configuration中，有了SqlSource，就能拿BoundSql了，BoundSql可以得到最终的sql。
## 实例分析
以下面的xml解析大概说下parseDynamicTags的解析过程：
```xml
<update id="update" parameterType="org.format.dynamicproxy.mybatis.bean.User">
    UPDATE users
    <trim prefix="SET" prefixOverrides=",">
        <if test="name != null and name != ''">
            name = #{name}
        </if>
        <if test="age != null and age != ''">
            , age = #{age}
        </if>
        <if test="birthday != null and birthday != ''">
            , birthday = #{birthday}
        </if>
    </trim>
    where id = ${id}
</update>
```
parseDynamicTags方法的返回值是一个List，也就是一个Sql节点集合。SqlNode本文一开始已经介绍，分析完解析过程之后会说一下各个SqlNode类型的作用。首先根据update节点(Node)得到所有的子节点，分别是3个子节点：文本节点 \n UPDATE userstrim子节点 ...文本节点 \n where id = #

遍历各个子节点：如果节点类型是文本或者CDATA，构造一个TextSqlNode或StaticTextSqlNode；如果节点类型是元素，说明该update节点是个动态sql，然后会使用NodeHandler处理各个类型的子节点。这里的NodeHandler是XMLScriptBuilder的一个内部接口，其实现类包括TrimHandler、WhereHandler、SetHandler、IfHandler、ChooseHandler等。看类名也就明白了这个Handler的作用，比如我们分析的trim节点，对应的是TrimHandler；if节点，对应的是IfHandler...这里子节点trim被TrimHandler处理，TrimHandler内部也使用parseDynamicTags方法解析节点。

遇到子节点是元素的话，重复以上步骤：trim子节点内部有7个子节点，分别是文本节点、if节点、是文本节点、if节点、是文本节点、if节点、文本节点。文本节点跟之前一样处理，if节点使用IfHandler处理。遍历步骤如上所示，下面我们看下几个Handler的实现细节。

IfHandler处理方法也是使用parseDynamicTags方法，然后加上if标签必要的属性：
```java
private class IfHandler implements NodeHandler {
    public void handleNode(XNode nodeToHandle, List<SqlNode> targetContents) {
      List<SqlNode> contents = parseDynamicTags(nodeToHandle);
      MixedSqlNode mixedSqlNode = new MixedSqlNode(contents);
      String test = nodeToHandle.getStringAttribute("test");
      IfSqlNode ifSqlNode = new IfSqlNode(mixedSqlNode, test);
      targetContents.add(ifSqlNode);
    }
}
```
TrimHandler处理方法也是使用parseDynamicTags方法，然后加上trim标签必要的属性：
```java
private class TrimHandler implements NodeHandler {
    public void handleNode(XNode nodeToHandle, List<SqlNode> targetContents) {
      List<SqlNode> contents = parseDynamicTags(nodeToHandle);
      MixedSqlNode mixedSqlNode = new MixedSqlNode(contents);
      String prefix = nodeToHandle.getStringAttribute("prefix");
      String prefixOverrides = nodeToHandle.getStringAttribute("prefixOverrides");
      String suffix = nodeToHandle.getStringAttribute("suffix");
      String suffixOverrides = nodeToHandle.getStringAttribute("suffixOverrides");
      TrimSqlNode trim = new TrimSqlNode(configuration, mixedSqlNode, prefix, prefixOverrides, suffix, suffixOverrides);
      targetContents.add(trim);
    }
}
```
以上update方法最终通过parseDynamicTags方法得到的SqlNode集合如下：

trim节点：

由于这个update方法是个动态节点，因此构造出了DynamicSqlSource。DynamicSqlSource内部就可以构造sql了:


DynamicSqlSource内部的SqlNode属性是一个MixedSqlNode。然后我们看看各个SqlNode实现类的apply方法。下面分析一下各个SqlNode实现类的apply方法实现：

MixedSqlNode：MixedSqlNode会遍历调用内部各个sqlNode的apply方法。
```java
public boolean apply(DynamicContext context) {
   for (SqlNode sqlNode : contents) {
     sqlNode.apply(context);
   }
   return true;
}
```
StaticTextSqlNode：直接append sql文本。
```java
public boolean apply(DynamicContext context) {
   context.appendSql(text);
   return true;
}
```
IfSqlNode：这里的evaluator是一个ExpressionEvaluator类型的实例，内部使用了OGNL处理表达式逻辑。
```java
public boolean apply(DynamicContext context) {
   if (evaluator.evaluateBoolean(test, context.getBindings())) {
     contents.apply(context);
     return true;
   }
   return false;
}
```
TrimSqlNode：
```java
public boolean apply(DynamicContext context) {
    FilteredDynamicContext filteredDynamicContext = new FilteredDynamicContext(context);
    boolean result = contents.apply(filteredDynamicContext);
    filteredDynamicContext.applyAll();
    return result;
}

public void applyAll() {
    sqlBuffer = new StringBuilder(sqlBuffer.toString().trim());
    String trimmedUppercaseSql = sqlBuffer.toString().toUpperCase(Locale.ENGLISH);
    if (trimmedUppercaseSql.length() > 0) {
        applyPrefix(sqlBuffer, trimmedUppercaseSql);
        applySuffix(sqlBuffer, trimmedUppercaseSql);
    }
    delegate.appendSql(sqlBuffer.toString());
}

private void applyPrefix(StringBuilder sql, String trimmedUppercaseSql) {
    if (!prefixApplied) {
        prefixApplied = true;
        if (prefixesToOverride != null) {
            for (String toRemove : prefixesToOverride) {
                if (trimmedUppercaseSql.startsWith(toRemove)) {
                    sql.delete(0, toRemove.trim().length());
                    break;
                }
            }
        }
        if (prefix != null) {
            sql.insert(0, " ");
            sql.insert(0, prefix);
        }
   }
}
```
`TrimSqlNode`的`apply`方法也是调用属性`contents`(一般都是`MixedSqlNode`)的`apply`方法，按照实例也就是 7 个`SqlNode`，都是`StaticTextSqlNode`和`IfSqlNode`。最后会使用`FilteredDynamicContext`过滤掉`prefix`和`suffix`。
