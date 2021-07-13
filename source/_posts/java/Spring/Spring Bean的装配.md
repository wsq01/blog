---
title: Spring Bean的装配
date: 2021-04-07 16:24:15
tags: [Spring]
categories: [Spring]
---



# 基于XML装配Bean
`Bean`的装配可以理解为依赖关系注入，`Bean`的装配方式也就是`Bean`的依赖注入方式。Spring 容器支持多种形式的`Bean`的装配方式，如基于 XML 的`Bean`装配、基于`Annotation`的`Bean`装配和自动装配等。

Spring 基于 XML 的装配通常采用两种实现方式，即设值注入（`Setter Injection`）和构造注入（`Constructor Injection`）。

在 Spring 实例化`Bean`的过程中，首先会调用默认的构造方法实例化`Bean`对象，然后通过 Java 的反射机制调用`setXxx()`方法进行属性的注入。因此，设值注入要求一个`Bean`的对应类必须满足以下两点要求。
* 必须提供一个默认的无参构造方法。
* 必须为需要注入的属性提供对应的 setter 方法。

使用设值注入时，在 Spring 配置文件中，需要使用`<bean>`元素的子元素`<property>`元素为每个属性注入值。而使用构造注入时，在配置文件中，主要使用`<constructor-arg>`标签定义构造方法的参数，可以使用其`value`属性（或子元素）设置该参数的值。下面是基于 XML 方式的 Bean 的装配。
## 1. 创建 Person 类
在项目`springDemo02`中的`src`目录下，创建一个名称为`com.mengma.assembly`的包，在该包下创建一个`Person`类。
```java
package com.mengma.assembly;
public class Person {
  private String name;
  private int age;
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public int getAge() {
    return age;
  }
  public void setAge(int age) {
    this.age = age;
  }
  // 重写toString()方法
  public String toString() {
      return "Person[name=" + name + ",age=" + age + "]";
  }
  // 默认无参的构造方法
  public Person() {
    super();
  }
  // 有参的构造方法
  public Person(String name, int age) {
    super();
    this.name = name;
    this.age = age;
  }
}
```
上述代码中，定义了`name`和`age`两个属性，并为其提供了` getter`和`setter`方法，由于要使用构造注入，所以需要提供有参的构造方法。为了能更清楚地看到输出结果，这里还重写了`toString()`方法。
## 2. 创建 Spring 配置文件
在`com.mengma.assembly`包下创建一个名为`applicationContext.xml`的配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
  <!-- 使用设值注入方式装配Person实例 -->
  <bean id="person1" class="com.mengma.assembly.Person">
    <property name="name" value="zhangsan" />
    <property name="age" value="20" />
  </bean>
  <!-- 使用构造方法装配Person实例 -->
  <bean id="person2" class="com.mengma.assembly.Person">
    <constructor-arg index="0" value="lisi" />
    <constructor-arg index="1" value="21" />
  </bean>
</beans>
```
上述代码中，首先使用了设值注入方式装配`Person`类的实例，其中`<property>`子元素用于调用`Bean`实例中的`setXxx()`方法完成属性赋值。然后使用了构造方式装配了`Person`类的实例，其中`<constructor-arg>`元素用于定义构造方法的参数，其属性`index`表示其索引（从 0 开始），`value`属性用于设置注入的值。
## 3. 创建测试类
在`com.mengma.assembly`包下创建一个名称为`XmlBeanAssemblyTest`的测试类。
```java
package com.mengma.assembly;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class XmlBeanAssemblyTest {
  @Test
  public void test() {
    // 定义Spring配置文件路径
    String xmlPath = "com/mengma/assembly/applicationContext.xml";
    // 初始化Spring容器，加载配置文件，并对bean进行实例化
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(
            xmlPath);
    // 设值方式输出结果
    System.out.println(applicationContext.getBean("person1"));
    // 构造方式输出结果
    System.out.println(applicationContext.getBean("person2"));
  }
}
```
# 基于Annotation装配Bean
尽管使用 XML 配置文件可以实现 Bean 的装配工作，但如果应用中 Bean 的数量较多，会导致 XML 配置文件过于臃肿，从而给维护和升级带来一定的困难。

Java 提供了`Annotation`（注解）功能。Spring3 中定义了一系列的`Annotation`（注解），常用的注解如下。
1. @Component
可以使用此注解描述 Spring 中的 Bean，但它是一个泛化的概念，仅仅表示一个组件（Bean），并且可以作用在任何层次。使用时只需将该注解标注在相应类上即可。
2. @Repository
用于将数据访问层（DAO层）的类标识为 Spring 中的 Bean，其功能与 @Component 相同。
3. @Service
通常作用在业务层（Service 层），用于将业务层的类标识为 Spring 中的 Bean，其功能与 @Component 相同。
4. @Controller
通常作用在控制层（如 Struts2 的 Action），用于将控制层的类标识为 Spring 中的 Bean，其功能与 @Component 相同。
5. @Autowired
用于对 Bean 的属性变量、属性的 Set 方法及构造函数进行标注，配合对应的注解处理器完成 Bean 的自动配置工作。默认按照 Bean 的类型进行装配。
6. @Resource
其作用与 Autowired 一样。其区别在于 @Autowired 默认按照 Bean 类型装配，而 @Resource 默认按照 Bean 实例名称进行装配。

@Resource 中有两个重要属性：`name`和`type`。

Spring 将 name 属性解析为 Bean 实例名称，`type`属性解析为`Bean`实例类型。如果指定 name 属性，则按实例名称进行装配；如果指定`type`属性，则按`Bean`类型进行装配。

如果都不指定，则先按 Bean 实例名称装配，如果不能匹配，则再按照 Bean 类型进行装配；如果都无法匹配，则抛出`NoSuchBeanDefinitionException`异常。
7. @Qualifier
与 @Autowired 注解配合使用，会将默认的按`Bean`类型装配修改为按 Bean 的实例名称装配，`Bean`的实例名称由`@Qualifier`注解的参数指定。
1. 创建 DAO 层接口
在`src`目录下创建一个名为`com.mengma.annotation`的包，在该包下创建一个名为`PersonDao`的接口，并添加一个`add()`方法。
```java
package com.mengma.annotation;
public interface PersonDao {
  public void add();
}
```
2. 创建 DAO 层接口的实现类
在`com.mengma.annotation`包下创建`PersonDao`接口的实现类`PersonDaoImpl`。
```java
package com.mengma.annotation;
import org.springframework.stereotype.Repository;
@Repository("personDao")
public class PersonDaoImpl implements PersonDao {
    @Override
    public void add() {
        System.out.println("Dao层的add()方法执行了...");
    }
}
```
上述代码中，首先使用 @Repository 注解将 PersonDaoImpl 类标识为 Spring 中的 Bean，其写法相当于配置文件中 <bean id="personDao"class="com.mengma.annotation.PersonDaoImpl"/> 的书写。然后在 add() 方法中输出一句话，用于验证是否成功调用了该方法。
3. 创建 Service 层接口
在`com.mengma.annotation`包下创建一个名为`PersonService`的接口，并添加一个`add()`方法。
```java
package com.mengma.annotation;
public interface PersonService {
  public void add();
}
```
4. 创建 Service 层接口的实现类
在`com.mengma.annotation`包下创建`PersonService`接口的实现类`PersonServiceImpl`。
```java
package com.mengma.annotation;
import javax.annotation.Resource;
import org.springframework.stereotype.Service;
@Service("personService")
public class PersonServiceImpl implements PersonService {
  @Resource(name = "personDao")
  private PersonDao personDao;
  public PersonDao getPersonDao() {
    return personDao;
  }
  @Override
  public void add() {
    personDao.add();// 调用personDao中的add()方法
    System.out.println("Service层的add()方法执行了...");
  }
}
```
上述代码中，首先使用`@Service`注解将`PersonServiceImpl`类标识为 Spring 中的`Bean`，其写法相当于配置文件中`<bean id="personService"class="com.mengma.annotation.PersonServiceImpl"/>`的书写。

然后使用`@Resource`注解标注在属性`personDao`上（也可标注在`personDao`的`setPersonDao()`方法上），这相当于配置文件中`<property name="personDao"ref="personDao"/>`的写法。最后在该类的`add()`方法中调用`personDao`中的`add()`方法，并输出一句话。
5. 创建 Action
在`com.mengma.annotation`包下创建一个名为`PersonAction`的类。
```java
package com.mengma.annotation;
import javax.annotation.Resource;
import org.springframework.stereotype.Controller;
@Controller("personAction")
public class PersonAction {
  @Resource(name = "personService")
  private PersonService personService;
  public PersonService getPersonService() {
    return personService;
  }
  public void add() {
    personService.add(); // 调用personService中的add()方法
    System.out.println("Action层的add()方法执行了...");
  }
}
```
上述代码中，首先使用`@Controller`注解标注`PersonAction`类，其写法相当于在配置文件中编写`<bean id="personAction"class="com.mengma.annotation.PersonAction"/>`。

然后使用了`@Resource`注解标注在`personService`上，这相当于在配置文件内编写`<property name="personService"ref="personService"/>`。

最后在其`add()`方法中调用了`personService`中的`add()`方法，并输出一句话。
6. 创建 Spring 配置文件
在`com.mengma.annotation`包下创建一个名为`applicationContext.xml`的配置文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:p="http://www.springframework.org/schema/p"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
          http://www.springframework.org/schema/beans
          http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
          http://www.springframework.org/schema/aop
          http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
          http://www.springframework.org/schema/tx
          http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
  <!--使用context命名空间，通知spring扫描指定目录，进行注解的解析-->
  <context:component-scan base-package="com.mengma.annotation"/>
</beans>
```
与之前的配置文件相比，上述代码的`<beans>`元素中增加了第 7 行、第 15 行和第 16 行中包含有`context`的代码，然后在第 18 行代码中，使用`context`命名空间的`component-scan`元素进行注解的扫描，其`base-package`属性用于通知 spring 所需要扫描的目录。
7. 创建测试类
在 com.mengma.annotation 包下创建一个名为 AnnotationTest 的测试类，编辑后如下所示。
```java
package com.mengma.annotation;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class AnnotationTest {
  @Test
  public void test() {
    // 定义Spring配置文件路径
    String xmlPath = "com/mengma/annotation/applicationContext.xml";
    // 初始化Spring容器，加载配置文件，并对bean进行实例化
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
    // 获得personAction实例
    PersonAction personAction = (PersonAction) applicationContext.getBean("personAction");
    // 调用personAction中的add()方法
    personAction.add();
  }
}
```
上述代码中，首先通过加载配置文件并获取`personAction`的实例，然后调用该实例的`add()`方法。
# 自动装配Bean
自动装配就是指 Spring 容器可以自动装配（autowire）相互协作的 Bean 之间的关联关系，将一个 Bean 注入其他 Bean 的 Property 中。

要使用自动装配，就需要配置 <bean> 元素的 autowire 属性。autowire 属性有五个值，具体说明如表 1 所示。

| 名称 | 说明 |
| :--: | :--: |
| byName | 根据 Property 的 name 自动装配，如果一个 Bean 的 name 和另一个 Bean 中的 Property 的 name 相同，则自动装配这个 Bean 到 Property 中。 |
| byType | 根据 Property 的数据类型（Type）自动装配，如果一个 Bean 的数据类型兼容另一个 Bean 中 Property 的数据类型，则自动装配。 |
| constructor | 根据构造方法的参数的数据类型，进行 byType 模式的自动装配。 |
| autodetect | 如果发现默认的构造方法，则用 constructor 模式，否则用 byType 模式。 |
| no | 默认情况下，不使用自动装配，Bean 依赖必须通过 ref 元素定义。 |

将 applicationContext.xml 配置文件修改成自动装配形式。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:p="http://www.springframework.org/schema/p" 
  xmlns:tx="http://www.springframework.org/schema/tx"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="  
          http://www.springframework.org/schema/beans 
          http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
          http://www.springframework.org/schema/aop 
          http://www.springframework.org/schema/aop/spring-aop-2.5.xsd  
          http://www.springframework.org/schema/tx 
          http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
          http://www.springframework.org/schema/context
          http://www.springframework.org/schema/context/spring-context.xsd">
  <bean id="personDao" class="com.mengma.annotation.PersonDaoImpl" />
  <bean id="personService" class="com.mengma.annotation.PersonServiceImpl" autowire="byName" />
  <bean id="personAction" class="com.mengma.annotation.PersonAction" autowire="byName" />
</beans>
```
在上述配置文件中，用于配置 personService 和 personAction 的 <bean> 元素中除了 id 和 class 属性以外，还增加了 autowire 属性，并将其属性值设置为 byName（按属性名称自动装配）。

默认情况下，配置文件中需要通过`ref`装配`Bean`，但设置了`autowire="byName"`，Spring 会在配置文件中自动寻找与属性名字`personDao`相同的`<bean>`，找到后，通过调用`setPersonDao（PersonDao personDao）`方法将`id`为`personDao`的`Bean`注入`id`为`personService`的`Bean`中，这时就不需要通过`ref`装配了。