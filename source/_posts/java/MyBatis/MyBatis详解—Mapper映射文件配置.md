---
title: MyBatis详解—Mapper映射文件配置
date: 2024-05-24 17:50:15
tags: [MyBatis]
categories: MyBatis
---


在`mapper`文件中，以`mapper`作为根节点，其下面可以配置的元素节点有：`select, insert, update, delete, cache, cache-ref, resultMap, sql`; 本文将对`Mapper`映射文件配置进行详解。
# insert, update, delete 的配置及使用
```xml
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<!-- mapper 为根元素节点， 一个namespace对应一个dao -->
<!-- 
Mapper元素只有一个属性namespace，它有两个作用：
`一是用于区分不同的mapper`（在不同的mapper文件里，子元素的id可以相同，mybatis通过namespace和子元素的id联合区分），
`二是与接口关联`（应用程序通过接口访问mybatis时，mybatis通过接口的完整名称查找对应的mapper配置，
因此namespace的命名务必小心一定要某接口同名）。
-->
<mapper namespace="com.dy.dao.UserDao">
    <!-- 
    cache- 配置本定命名空间的缓存。
        type- cache实现类，默认为PERPETUAL，可以使用自定义的cache实现类（别名或完整类名皆可）
        eviction- 回收算法，默认为LRU，可选的算法有：
            LRU– 最近最少使用的：移除最长时间不被使用的对象。
            FIFO– 先进先出：按对象进入缓存的顺序来移除它们。
            SOFT– 软引用：移除基于垃圾回收器状态和软引用规则的对象。
            WEAK– 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
        flushInterval- 刷新间隔，默认为1个小时，单位毫秒
        size- 缓存大小，默认大小1024，单位为引用数
        readOnly- 只读
    -->
    <cache type="PERPETUAL" eviction="LRU" flushInterval="60000" size="512" readOnly="true" />
    
    <!-- 
    cache-ref–从其他命名空间引用缓存配置。
        如果你不想定义自己的cache，可以使用cache-ref引用别的cache。
        因为每个cache都以namespace为id，所以cache-ref只需要配置一个namespace属性就可以了。
        需要注意的是，如果cache-ref和cache都配置了，以cache为准。
    -->
    <cache-ref namespace="com.someone.application.data.SomeMapper"/>
    
    <insert
      <!-- 1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致 -->
      
      id="insertUser"
      
      <!-- 2. parameterType (可选配置, 默认为mybatis自动选择处理)
        将要传入语句的参数的完全限定类名或别名，
        如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
        parameterType 主要指定参数类型，可以是int,short,long,string等类型，也可以是复杂类型（如对象） -->
      
      parameterType="com.demo.User"
      
      <!-- 3. flushCache （可选配置，默认配置为true）
        将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，
        默认值：true（对应插入、更新和删除语句） -->
      
      flushCache="true"
      
      <!-- 4. statementType （可选配置，默认配置为PREPARED）
        STATEMENT，PREPARED 或 CALLABLE 的一个。
        这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 -->
      
      statementType="PREPARED"
      
      <!-- 5. keyProperty (可选配置， 默认为unset)
        （仅对 insert 和 update 有用）
        唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。
        如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      
      keyProperty=""
      
      <!-- 6. keyColumn (可选配置)
        （仅对 insert 和 update 有用）
        通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。
        如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      
      keyColumn=""
      
      <!-- 7. useGeneratedKeys (可选配置， 默认为false)
        （仅对 insert 和 update 有用）
        这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键
        （比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。  -->
      
      useGeneratedKeys="false"
      
      <!-- 8. timeout  (可选配置， 默认为unset, 依赖驱动)
        这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 -->
      timeout="20">

    <update
      id="updateUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">

    <delete
      id="deleteUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">
</mapper>
```
以上就是一个模板配置，哪些是必要配置，哪些是根据自己实际需求，看一眼就知道了。看一个真实的`UserDao-Mapper.xml`配置：
```xml
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<mapper namespace="com.dy.dao.UserDao">
   
   <!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
      insert into user(id, name, password, age, deleteFlag) values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
   </insert>
   
   <!-- 对应userDao中的updateUser方法 -->
   <update id="updateUser" parameterType="com.dy.entity.User">
      update user set name = #{name}, password = #{password}, age = #{age}, deleteFlag = #{deleteFlag} where id = #{id};
   </update>
    
   <!-- 对应userDao中的deleteUser 方法 --> 
   <delete id="deleteUser" parameterType="com.dy.entity.User">
      delete from user where id = #{id};
   </delete>
</mapper>
```
这样，一个简单的映射关系就建立了。仔细观察上面`parameterType`, `"com.dy.entity.User"`，每次都这样写，得蛋疼了。别忘了`typeAliases`（别名）。咱们配上别名，在哪儿配？当然是在 mybatis 的全局配置文件（我这儿名字是`mybatis-conf.xml`），不要认为是在`mapper`的配置文件里面配置哈。
```xml
<typeAliases>
    <!--
      通过package, 可以直接指定package的名字， mybatis会自动扫描你指定包下面的javabean,
      并且默认设置一个别名，默认的名字为： javabean 的首字母小写的非限定类名来作为它的别名。
      也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user) 
      <package name="com.dy.entity"/>
    -->
    <typeAlias alias="user" type="com.dy.entity.User"/>
</typeAliases>
```
这样，一个别名就取好了，咱们可以把上面的`com.dy.entity.User`都直接改为`user`了。这多方便呀！我这儿数据库用的是 mysql, 我把`user`表的主键id 设置了自动增长，以上代码运行正常，那么问题来了，我要是换成oracle数据库怎么办？oracle 可是不支持`id`自增长啊？请看下面：
```xml
<!-- 对应userDao中的insertUser方法，  -->
<insert id="insertUser" parameterType="com.dy.entity.User">
    <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id -->  
    <selectKey resultType="int" order="BEFORE" keyProperty="id">
          select seq_user_id.nextval as id from dual
    </selectKey>
    
    insert into user(id, name, password, age, deleteFlag) 
    values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
</insert>
```
同理，如果我们在使用 mysql 的时候，想在数据插入后返回插入的`id`, 我们也可以使用`selectKey`这个元素：   
```xml
<!-- 对应userDao中的insertUser方法，  -->
<insert id="insertUser" parameterType="com.dy.entity.User">
    <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id 
    <selectKey resultType="int" order="BEFORE" keyProperty="id">
          select seq_user_id.nextval as id from dual
    </selectKey>
    --> 
    
    <!-- mysql插入数据后，获取id，该方法LAST_INSERT_ID()与数据库连接绑定，同属统一会话级别。-->
    <selectKey keyProperty="id" resultType="int" order="AFTER" >
          SELECT LAST_INSERT_ID() as id
    </selectKey>
      
    insert into user(id, name, password, age, deleteFlag) 
    values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
</insert>
```
这儿，我们就简单提一下`<selectKey>`这个元素节点吧。

`selectKey`给了你一个简单的行为在你的数据库中来处理自动生成的主键，而不需要使你的 Java 代码变得复杂。在上面的示例中，`selectKey`元素将会首先运行，`userid`会被设置，然后插入语句会被调用。另外，`selectKey`节点生成的`KeyGenerator`优先级高于`statement`节点的`useGeneratedKeys`属性生成的`KeyGenerator`对象，也就是说配置了`SelectKey`子节点就不需要再配置`useGeneratedKeys`属性了。
```xml
<selectKey
      <!-- selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
      keyProperty="id"
      <!-- 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。
      MyBatis 允许任何简单类型用作主键的类型，包括字符串。
      如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 -->
      resultType="int"
      <!-- 这可以被设置为 BEFORE 或 AFTER。
      如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。
      如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 -->
      order="BEFORE"
      <!-- 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，
      分别代表 PreparedStatement 和 CallableStatement 类型。 -->
      statementType="PREPARED">
```
# select、resultMap的配置及使用
`select`无疑是我们最常用，mybatis 通过`resultMap`能帮助我们很好地进行高级映射。先看`select`的配置吧：
```xml
<select
     <!--  1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致
     -->
     
     id="selectPerson"
     
     <!-- 2. parameterType（可选配置, 默认为mybatis自动选择处理）
        将要传入语句的参数的完全限定类名或别名，
        如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
     parameterType="int"
     
     <!-- 3. resultType (resultType 与 resultMap 二选一配置)
         resultType用以指定返回类型，指定的类型可以是基本类型，可以是java容器，也可以是javabean -->
     resultType="hashmap"
     
     <!-- 4. resultMap (resultType 与 resultMap 二选一配置)
         resultMap用于引用我们通过 resultMap标签定义的映射类型，这也是mybatis组件高级复杂映射的关键 -->
     resultMap="personResultMap"
     
     <!-- 5. flushCache (可选配置)
         将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false -->
     flushCache="false"
     
     <!-- 6. useCache (可选配置)
         将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true -->
     useCache="true"
     
     <!-- 7. timeout (可选配置) 
         这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）-->
     timeout="10000"
     
     <!-- 8. fetchSize (可选配置) 
         这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动)-->
     fetchSize="256"
     
     <!-- 9. statementType (可选配置) 
         STATEMENT，PREPARED 或 CALLABLE 的一个。
         这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED-->
     statementType="PREPARED"
     
     <!-- 10. resultSetType (可选配置) 
         FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）-->
     resultSetType="FORWARD_ONLY">
```
配置看起来这么多，实际常用的配置也就那么几个，根据自己的需要吧，上面都已注明是否必须配置。看一个`CourseDao-Mapper.xml`配置：
```xml
<mapper namespace="com.dy.dao.CourseDao">
    
    <!-- 
         1.此处直接将resultType 设置为course, 一看就知道我设置了别名吧，如果没有设置别名，那么resultType = com.dy.entity.Course。
         2.可能细心的你会发现：Course.java中的属性名与数据库字段名不一致，下面，我就在sql语句中用了as, 使之匹配，当然方法不止一种，在学习了resultMap之后，你能看到一种更直观优雅的方式去将javabean中的属性与数据库字段名保持一致
         3.findCourseById 与CourseDao中findCourseById方法对应， 那么传入的参数名称以及类型也应该保持对应关系。
         4.可以看到，在sql语句中，通过#{}表达式可以获取参数。
         5.下面这条sql语句，实际上的形式是怎么样的？还记得之前说过，mybatis默认为preparedStatement吧，那么，用我们jdbc代码来看，它其实就是：
             select course_id as id, course_name as name, course_delete_flg as deleteFlag from t_course where course_id=?
     -->
    <select id="findCourseById"  resultType="course" >
        select course_id as id, course_name as name, course_delete_flg as deleteFlag from t_course where course_id=#{courseId}
    </select>
</mapper>
```
上面的示例，我们针对`course`, 简单演示了`select`的用法，不过有个问题值得思考：一个`student`可以对应多个`course`, 那么，在 mybatis 中如何处理这种一对多，甚至于多对多，一对一的关系呢？这儿，就不得不提到`resultMap`这个东西，mybatis 的`resultMap`功能可谓十分强大，能够处理复杂的关系映射：    
```xml
<!-- 
    resultMap –结果映射，用来描述如何从数据库结果集映射到你想要的对象。

    1.type 对应类型，可以是javabean, 也可以是其它
    2.id 必须唯一， 用于标示这个resultMap的唯一性，在使用resultMap的时候，就是通过id指定
  -->
<resultMap type="" id="">

    <!-- id, 唯一性，注意啦，这个id用于标示这个javabean对象的唯一性，不一定会是数据库的主键（不要把它理解为数据库对应表的主键） 
        property属性对应javabean的属性名，column对应数据库表的列名
        （这样，当javabean的属性与数据库对应表的列名不一致的时候，就能通过指定这个保持正常映射了）
    -->
    <id property="" column=""/>
    
    <!-- result与id相比， 对应普通属性 -->    
    <result property="" column=""/>
    
    <!-- 
        constructor对应javabean中的构造方法
      -->
    <constructor>
        <!-- idArg 对应构造方法中的id参数；-->
        <idArg column=""/>
        <!-- arg 对应构造方法中的普通参数；-->
        <arg column=""/>
    </constructor>
    
    <!-- 
        聚集元素用来处理“一对多”的关系。需要指定映射的Java实体类的属性，属性的javaType（一般为ArrayList）；
        列表中对象的类型ofType（Java实体类）；对应的数据库表的列名称；

        collection，对应javabean中容器类型, 是实现一对多的关键 
        property 为javabean中容器对应字段名
        column 为体现在数据库中列名
        ofType 就是指定javabean中容器指定的类型

        不同情况需要告诉MyBatis 如何加载一个聚集。MyBatis 可以用两种方式加载：
            1. select: 执行一个其它映射的SQL 语句返回一个Java实体类型。较灵活；
            2. resultMap: 使用一个嵌套的结果映射来处理通过join查询结果集，映射成Java实体类型。
    -->
    <collection property="" column="" ofType=""></collection>
    
    <!-- 
        联合元素用来处理“一对一”的关系。需要指定映射的Java实体类的属性，属性的javaType（通常MyBatis 自己会识别）。
        对应的数据库表的列名称。如果想覆写的话返回结果的值，需要指定typeHandler。

        association 为关联关系，是实现N对一的关键。
        property 为javabean中容器对应字段名
        column 为体现在数据库中列名
        javaType 指定关联的类型

        不同情况需要告诉MyBatis 如何加载一个联合。MyBatis可以用两种方式加载：
            3. select: 执行一个其它映射的SQL 语句返回一个Java实体类型。较灵活；
            4. resultMap: 使用一个嵌套的结果映射来处理，通过join查询结果集，映射成Java实体类型。
      -->
    <association property="" column="" javaType=""></association>

    <!-- 
        有时一个单独的数据库查询也许返回很多不同（但是希望有些关联）数据类型的结果集。
        鉴别器元素就是被设计来处理这个情况的，还有包括类的继承层次结构。鉴别器非常容易理解，因为它的表现很像Java语言中的switch语句。

        定义鉴别器指定了column和javaType属性。列是MyBatis查找比较值的地方。
        JavaType是需要被用来保证等价测试的合适类型（尽管字符串在很多情形下都会有用）。

        下面这个例子为，当classId为20000001时，才映射classId属性。
    -->
    <discriminator column="CLASS_ID" javaType="String" jdbcType="VARCHAR">  
        <case value="20000001" resultType="liming.student.manager.data.model.StudentEntity" >  
            <result property="classId" column="CLASS_ID" javaType="String" jdbcType="VARCHAR"/>  
        </case> 
    </discriminator>
</resultMap>
```
知道`resutMap`怎么配置后，咱们接着上面的 demo 来练习一下，一个`student`对应多个`course`，典型的一对多，咱们就来看看 mybatis 怎么配置这种映射吧：`StudentDao-Mapper.xml`
```xml
<mapper namespace="com.dy.dao.StudentDao">

    <!-- 这儿定义一个resultMap -->
    <resultMap type="student" id="studentMap">
    
        <!-- 
            数据库中主键是id, 但是我这儿却是指定idCard为主键，为什么？ 
            刚刚讲了，id用来表示唯一性， 我们可以认为只要idCard一样，那么他就是同一个学生。
            如果此处用数据库中id， 那么mybatis将会认为数据库中每条记录都是一个student, 这显然不符合逻辑
        -->
        <id property="idCard" column="stu_id_card"/>
        <result property="id" column="stu_id"/>
        <result property="name" column="stu_name"/>
        <result property="deleteFlag" column="stu_delete_flg"/>
  
        <constructor>  
            <idArg javaType="String" column="STUDENT_ID"/>  
            <arg javaType="String" column="STUDENT_NAME"/>  
            <arg javaType="String" column="STUDENT_SEX"/>  
            <arg javaType="Date" column="STUDENT_BIRTHDAY"/>  
        </constructor>
        
        <!-- 
            这儿就是实现一对多的关键。 
            在Student中，courseList为List<Course>, 因此，ofType也应该与之对应（当然，我用了别名，不然要蛋疼的写全名了）。
            collection的子标签是在指定Course的映射关系（由于Course的javabean的属性名与数据库的列名不一致）
        -->
        <collection property="courseList" column="stu_course_id" ofType="Course">
            <id property="id" column="course_id"/>
            <result property="name" column="course_name"/>
            <result property="deleteFlag" column="course_delete_flg"/>
        </collection>
    </resultMap>
    
    <!-- 这儿将返回类型设置成了上面指定的studentMap -->
    <select id="findStudentById" resultMap="studentMap">
        SELECT s.*, c.* FROM t_student s LEFT JOIN t_course c ON s.stu_course_id=c.course_id WHERE s.stu_id_card=#{idCard}
    </select>

    <!-- 
        sql –可以重用的SQL块，可以被其他数据库操作语句引用。
    -->
    <sql id="userColumns"> userid,username,password</sql>
    
    <select id="queryUsers" parameterType="UserDto" resultType="UserDto" useCache="false">
	select <include refid="userColumns"/> from t_user t where t.username = #{username}
    </select>
    
</mapper>
```
当然，我们需要定义`StudentEntity`实体类的构造方法：
```java
public StudentEntity(String studentID, String studentName, String studentSex, Date studentBirthday){  
    this.studentID = studentID;  
    this.studentName = studentName;  
    this.studentSex = studentSex;  
    this.studentBirthday = studentBirthday;  
}
```
上面只演示了一对多的映射，其实多对一、多对多也与它类似，所以我就没演示了，有兴趣的可以自己动手再做做。

# 字符串代入法
默认的情况下，使用`#{}`语法会促使 MyBatis 生成`PreparedStatement`属性并且使用`PreparedStatement`的参数（`=？`）来安全的设置值。尽量这些是快捷安全，也是经常使用的。但有时候你可能想直接未更改的字符串代入到SQL 语句中。比如说，对于`ORDER BY`，你可能会这样使用：`ORDER BY ${columnName}`但 MyBatis 不会修改和规避掉这个字符串。

注意：这样地接收和应用一个用户输入到未更改的语句中，是非常不安全的。这会让用户能植入破坏代码，所以，要么要求字段不要允许客户输入，要么你直接来检测他的合法性 。
# 子元素之cache解析
`Mapper`配置文件是由`XMLMapperBuilder`解析的，其中`cacheElement`方法负责解析`cache`元素，它通过调用`CacheBuilder`的相应方法完成`cache`的创建。每个`cache`内部都有一个唯一的 ID，这个`id`的值就是`namespace`。创建好的`cache`对象存入`configuration`的`cache`缓存中（该缓存以`cache`的`ID`属性即`namespace`为`key`，这里再次体现了 mybatis 的`namespace`的强大用处）。  
```java
private void cacheElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type", "PERPETUAL");
      Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
      String eviction = context.getStringAttribute("eviction", "LRU");
      Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
      Long flushInterval = context.getLongAttribute("flushInterval");
      Integer size = context.getIntAttribute("size");
      boolean readWrite = !context.getBooleanAttribute("readOnly", false);
      Properties props = context.getChildrenAsProperties();
      builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, props);
    }
  }
```
# 子元素之cache-ref解析
`cacheRefElement`方法负责解析`cache-ref`元素，它通过调用`CacheRefResolver`的相应方法完成`cache`的引用。创建好的`cache-ref`引用关系存入`configuration`的`cacheRefMap`缓存中。  
```java
private void cacheRefElement(XNode context) {
    if (context != null) {
      configuration.addCacheRef(builderAssistant.getCurrentNamespace(), context.getStringAttribute("namespace"));
      CacheRefResolver cacheRefResolver = new CacheRefResolver(builderAssistant, context.getStringAttribute("namespace"));
      try {
    	  cacheRefResolver.resolveCacheRef();
      } catch (IncompleteElementException e) {
    	  configuration.addIncompleteCacheRef(cacheRefResolver);
      }
    }
  }
```
# 子元素之resultMap解析
`resultMapElement`方法负责解析`resultMap`元素，它通过调用`ResultMapResolver`的相应方法完成`resultMap`的解析。创建好的`resultMap`存入`configuration`的`resultMaps`缓存中（该缓存以`namespace+resultMap`的`id`为`key`，这里再次体现了 mybatis 的`namespace`的强大用处）。  
```java
private ResultMap resultMapElement(XNode resultMapNode) throws Exception {
    return resultMapElement(resultMapNode, Collections.<ResultMapping> emptyList());
  }

  private ResultMap resultMapElement(XNode resultMapNode, List<ResultMapping> additionalResultMappings) throws Exception {
    ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());
    String id = resultMapNode.getStringAttribute("id",
        resultMapNode.getValueBasedIdentifier());
    String type = resultMapNode.getStringAttribute("type",
        resultMapNode.getStringAttribute("ofType",
            resultMapNode.getStringAttribute("resultType",
                resultMapNode.getStringAttribute("javaType"))));
    String extend = resultMapNode.getStringAttribute("extends");
    Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");
    Class<?> typeClass = resolveClass(type);
    Discriminator discriminator = null;
    List<ResultMapping> resultMappings = new ArrayList<ResultMapping>();
    resultMappings.addAll(additionalResultMappings);
    List<XNode> resultChildren = resultMapNode.getChildren();
    for (XNode resultChild : resultChildren) {
      if ("constructor".equals(resultChild.getName())) {
        processConstructorElement(resultChild, typeClass, resultMappings);
      } else if ("discriminator".equals(resultChild.getName())) {
        discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
      } else {
        ArrayList<ResultFlag> flags = new ArrayList<ResultFlag>();
        if ("id".equals(resultChild.getName())) {
          flags.add(ResultFlag.ID);
        }
        resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));
      }
    }
    ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);
    try {
      return resultMapResolver.resolve();
    } catch (IncompleteElementException  e) {
      configuration.addIncompleteResultMap(resultMapResolver);
      throw e;
    }
  }
```
# 子元素之sql解析
`sqlElement`方法负责解析`sql`元素。`id`属性用于区分不同的`sql`元素，在同一个`mapper`配置文件中可以配置多个`sql`元素。  
```java
private void sqlElement(List<XNode> list) throws Exception {
    if (configuration.getDatabaseId() != null) {
      sqlElement(list, configuration.getDatabaseId());
    }
    sqlElement(list, null);
  }

  private void sqlElement(List<XNode> list, String requiredDatabaseId) throws Exception {
    for (XNode context : list) {
      String databaseId = context.getStringAttribute("databaseId");
      String id = context.getStringAttribute("id");
      id = builderAssistant.applyCurrentNamespace(id, false);
      if (databaseIdMatchesCurrent(id, databaseId, requiredDatabaseId)) sqlFragments.put(id, context);
    }
  }
  
  private boolean databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) {
    if (requiredDatabaseId != null) {
      if (!requiredDatabaseId.equals(databaseId)) {
        return false;
      }
    } else {
      if (databaseId != null) {
        return false;
      }
      // skip this fragment if there is a previous one with a not null databaseId
      if (this.sqlFragments.containsKey(id)) {
        XNode context = this.sqlFragments.get(id);
        if (context.getStringAttribute("databaseId") != null) {
          return false;
        }
      }
    }
    return true;
  }
```
# 子元素之statement解析
`buildStatementFromContext`方法负责解析`statement`元素。`id`属性用于区分不同的`statement`元素，在同一个配置文件中可以配置多个`statement`元素。通过调用`XMLStatementBuilder`的`parseStatementNode`方法完成解析。在这个方法内有几个重要的步骤，理解他们对正确的配置`statement`元素很有帮助。
## 动态解析子元素
`statement`节点可以配置各种子元素，比如前面提到的`include`子元素和`selectKey`子元素等（在动态`sql`里还有更多的子元素，具体参考 mybatis 的官方文档）。动态解析子元素通过`parseDynamicTags`方法完成。该方法根据子元素的类型递归的解析成一个个的`SqlNode`，这些`SqlNode`对象提供了`apply`方法，供后续调用时生成sql语句所需。需要注意的是`SelectKey`没有对应的`SqlNode`对象，因为它的功能是用来生成`KeyGenerator`对象的（具体来说是`SelectKeyGenerator`对象）。另外，`SelectKey`节点生成的`KeyGenerator`优先级高于`statement`节点的`useGeneratedKeys`属性生成的`KeyGenerator`对象，也就是说配置了`SelectKey`子节点就不需要再配置`useGeneratedKeys`属性了。
## 生成SqlSource
`SqlSource`用于后续调用时根据`SqlNode`和参数对象生成`sql`语句。它接收一个叫做`rootsqlNode`的对象作为构造参数。
## 生成KeyGenerator
如果配置了`selectKey`子元素，`KeyGenerator`直接使用`selectKey`子元素里生成的`KeyGenerator`对象（具体来说是`SelectKeyGenerator`对象）。若没配置，则如果`useGeneratedKeys`属性的值为`true`且配置了`keyProperty`属性，则生成默认的`Jdbc3KeyGenerator`对象，该对象调用JDBC驱动的`getGeneratedKeys`方法返回`insert`语句执行后生成的自增长主键。
## 创建MappedStatement
`MappedStatement`对象封装了`statement`元素的所有属性以及子节点值，`MappedStatement`对象有一个`id`属性用于唯一标记它，这个`id`由`namespace`加`statement`元素的`id`属性值构成。创建好的`MappedStatement`对象存入`Configuration`对象的`mappedStatements`缓存中，`key`为`MappedStatement`对象的`id`值。
```java
// XMLMapperBuilder.java
private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
      buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
  }

  private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    for (XNode context : list) {
      final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
      try {
        statementParser.parseStatementNode();
      } catch (IncompleteElementException e) {
        configuration.addIncompleteStatement(statementParser);
      }
    }
  }
```
```java
// XMLStatementBuilder.java
public void parseStatementNode() {
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");

    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) return;

    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultType = context.getStringAttribute("resultType");
    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    Class<?> resultTypeClass = resolveClass(resultType);
    String resultSetType = context.getStringAttribute("resultSetType");
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);

    String nodeName = context.getNode().getNodeName();
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    // Parse selectKey after includes and remove them.
    processSelectKeyNodes(id, parameterTypeClass, langDriver);
    
    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
    String resultSets = context.getStringAttribute("resultSets");
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    KeyGenerator keyGenerator;
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
      keyGenerator = configuration.getKeyGenerator(keyStatementId);
    } else {
      keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
          configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
          ? new Jdbc3KeyGenerator() : new NoKeyGenerator();
    }

    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered, 
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
  }
```
# 注册mapper类型
我们知道每个`mapper`配置文件的`namespace`属性对应于某个接口，应用程序通过接口访问 mybatis 时，mybatis 会为这个接口生成一个代理对象，这个对象就叫`mapper`对象，在生成代理对象前 mybatis 会校验接口是否已注册，未注册的接口会产生一个异常。为了避免这种异常，就需要注册`mapper`类型。这个步骤是在`XMLMapperBuilder`的`bindMapperForNamespace`方法中完成的。它通过调用`Configuration`对象的`addMapper`方法完成，而`Configuration`对象的`addMapper`方法是通过`MapperRegistry`的`addMapper`方法完成的，它只是简单的将`namespace`属性对应的接口类型存入本地缓存中。

`Configuration`对象提供了一个重载的`addMappers(StringpackageName)`方法，该方法以包路径名为参数，它的功能是自动扫描包路径下的接口并注册到`MapperRegistry`的缓存中，同时扫描包路径下的`mapper`配置文件并解析之。解析配置文件是在`MapperAnnotationBuilder`类的`parse`方法里完成的，该方法先解析配置文件，然后再解析接口里的注解配置，且注解里的配置会覆盖配置文件里的配置，也就是说注解的优先级高于配置文件，这点需要注意。采用自动扫描会大大简化配置，只不过需要应用程序自己调用，mybatis 默认是不会调用这个方法的（后续spring集成mybatis就用到了自动扫描）。  
```java
private void bindMapperForNamespace() {
    String namespace = builderAssistant.getCurrentNamespace();
    if (namespace != null) {
      Class<?> boundType = null;
      try {
        boundType = Resources.classForName(namespace);
      } catch (ClassNotFoundException e) {
        //ignore, bound type is not required
      }
      if (boundType != null) {
        if (!configuration.hasMapper(boundType)) {
          // Spring may not know the real resource name so we set a flag
          // to prevent loading again this resource from the mapper interface
          // look at MapperAnnotationBuilder#loadXmlResource
          configuration.addLoadedResource("namespace:" + namespace);
          configuration.addMapper(boundType);
        }
      }
    }
  }
```