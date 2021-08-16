---
title: Spring Bean的装配
date: 2021-05-20 16:24:15
tags: [Spring]
categories: [Spring]
---



# 基于XML装配Bean
`Bean`的装配可以理解为依赖关系注入，`Bean`的装配方式也就是`Bean`的依赖注入方式。Spring 容器支持多种形式的`Bean`的装配方式，如基于 XML 的`Bean`装配、基于`Annotation`的`Bean`装配和自动装配等。

Spring 基于 XML 的装配通常采用两种实现方式，即设值注入（`Setter Injection`）和构造注入（`Constructor Injection`）。

# 自动装配Bean
自动装配就是指 Spring 容器在不使用`<constructor-arg>`和`<property>`标签的情况下，可以自动装配（`autowire`）相互协作的`Bean`之间的关联关系，将一个`Bean`注入其他`Bean`的`Property`中。

使用自动装配需要配置`<bean>`元素的`autowire`属性。autowire 属性有五个值。

| 名称 | 说明 |
| :--: | :--: |
| `byName` | 根据`Property`的`name`自动装配，如果一个`Bean`的`name`和另一个`Bean`中的`Property`的`name`相同，则自动装配这个`Bean`到`Property`中。 |
| `byType` | 根据`Property`的数据类型（`Type`）自动装配，如果一个`Bean`的数据类型兼容另一个`Bean`中`Property`的数据类型，则自动装配。 |
| `constructor` | 根据构造方法的参数的数据类型，进行`byType`模式的自动装配。 |
| `autodetect` | 如果发现默认的构造方法，则用`constructor`模式，否则用`byType`模式。 |
| `no` | 默认值，表示不使用自动装配，`Bean`依赖必须通过`ref`元素定义。 |

```java
package net.biancheng;
public class Person {
  private Man man;
  public Person(Man man) {
    System.out.println("在Person的构造函数内");
    this.man = man;
  }
  public void man() {
    man.show();
  }
}
```
```java
package net.biancheng;
public class Man {
  private String name;
  private int age;
  public Man() {
    System.out.println("在man的构造函数内");
  }
  public Man(String name, int age) {
    System.out.println("在man的有参构造函数内");
    this.name = name;
    this.age = age;
  }
  public void show() {
    System.out.println("名称：" + name + "\n年龄：" + age);
  }
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
}
```
```java
package net.biancheng;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
    Person person = (Person) context.getBean("person");
    person.man();
  }
}
```
### 1.不使用自动装配（autowire="no"）
`autowire="no"`表示不使用自动装配，需要手动注入，`Bean`依赖通过`ref`元素定义，`Beans.xml`配置文件如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
  <bean id="man" class="net.biancheng.Man">
    <constructor-arg value="bianchengbang" />
    <constructor-arg value="12" type="int" />
  </bean>
  <bean id="person" class="net.biancheng.Person" autowire="no">
    <constructor-arg ref="man" type="java.lang.String"/>
  </bean>
</beans>
```
### 2.按名称自动装配（autowire="byName"）
`autowire="byName"`表示按属性名称自动装配，XML 文件中`Bean`的`id`必须与类中的属性名称相同。配置文件内容修改如下。
```xml
<bean id="man" class="net.biancheng.Man">
    <constructor-arg value="bianchengbang" />
    <constructor-arg value="12" type="int" />
</bean>
<bean id="person" class="net.biancheng.Person" autowire="byName"/>
```
运行结果如下。
```
在man的有参构造函数内
在Person的构造函数内
名称：bianchengbang
年龄：12
```
如果更改`Bean`的名称，很可能不会注入依赖项。

将`Bean`的名称更改为`man1`，配置文件如下：
```xml
<bean id="man1" class="net.biancheng.Man">
  <constructor-arg value="bianchengbang" />
  <constructor-arg value="12" type="int" />
</bean>
<bean id="person" class="net.biancheng.Person" autowire="byName"/>
```
注入失败，异常信息为：
```
在man的有参构造函数内
Exception in thread "main" java.lang.NullPointerException
at net.biancheng.Person.man(Person.java:16)
at net.biancheng.MainApp.main(MainApp.java:10)
```
### 3.按类型自动装配（autowire="byType"）
XML 文件中`Bean`的`id`与类中的属性名称可以不同，但必须只有一个类型的`Bean`。配置文件内容修改如下。
```xml
<bean id="man1" class="net.biancheng.Man">
  <constructor-arg value="bianchengbang" />
  <constructor-arg value="12" type="int" />
</bean>
<bean id="person" class="net.biancheng.Person" autowire="byType"/>
```
运行结果如下。
```
在man的有参构造函数内
在Person的构造函数内
名称：bianchengbang
年龄：12
```
如果您有相同类型的多个`Bean`，则注入失败，并且引发异常。

添加`id`为`man2`的`Bean`，配置文件代码如下。
```xml
<bean id="man1" class="net.biancheng.Man">
  <constructor-arg value="bianchengbang" />
  <constructor-arg value="12" type="int" />
</bean>
<bean id="man2" class="net.biancheng.Man">
  <constructor-arg value="bianchengbang" />
  <constructor-arg value="12" type="int" />
</bean>
<bean id="person" class="net.biancheng.Person" autowire="byType"/>
```
异常信息为：
```
在man的有参构造函数内
在man的有参构造函数内
一月 26, 2021 1:34:14 下午 org.springframework.context.support.AbstractApplicationContext refresh
警告: Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'person' defined in class path resource [Beans.xml]: Unsatisfied dependency expressed through bean property 'man'; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'net.biancheng.Man' available: expected single matching bean but found 2: man1,man2
...
```
### 4.构造函数自动装配（autowire="constructor"）
修改`Person`类的代码。
```java
package net.biancheng;
public class Person {
  private Man man;
  public Person(Man man) {
    System.out.println("在Person的构造方法内");
    this.man = man;
  }
  public Man getMan() {
    return man;
  }
  public void man() {
    man.show();
  }
}
```
类中构造函数的参数必须在配置文件中有相同的类型，配置文件内容修改如下。
```xml
<bean id="man" class="net.biancheng.Man">
  <constructor-arg value="bianchengbang" />
  <constructor-arg value="12" type="int" />
</bean>
<bean id="person" class="net.biancheng.Person" autowire="constructor"/>
```
运行结果如下。
```
在man的有参构造函数内
在Person的构造函数内
名称：bianchengbang
年龄：12
```
## 自动装配的优缺点
优点：自动装配只需要较少的代码就可以实现依赖注入。

缺点：不能自动装配简单数据类型，比如`int、boolean、String`等。相比较显示装配，自动装配不受程序员控制。
# 基于注解装配Bean
尽管使用 XML 配置文件可以实现 Bean 的装配工作，但如果应用中`Bean`的数量较多，会导致 XML 配置文件过于臃肿，从而给维护和升级带来一定的困难。我们可以使用注解来配置依赖注入。

Spring 默认不使用注解装配`Bean`，因此需要在配置文件中添加`<context:annotation-config/>`，启用注解。

Spring 常用的注解如下：
1. `@Component`
可以使用此注解描述 Spring 中的`Bean`，但它是一个泛化的概念，仅仅表示一个组件（`Bean`），并且可以作用在任何层次。使用时只需将该注解标注在相应类上即可。
2. `@Repository`
用于将数据访问层（DAO层）的类标识为 Spring 中的`Bean`，其功能与`@Component`相同。
3. `@Service`
通常作用在业务层（Service 层），用于将业务层的类标识为 Spring 中的`Bean`，其功能与`@Component`相同。
4. `@Controller`
通常作用在控制层（如 SpringMVC 的`Controller`），用于将控制层的类标识为 Spring 中的`Bean`，其功能与`@Component`相同。
5. `@Autowired`
可以应用到`Bean`的属性变量、属性的`setter`方法、非`setter`方法及构造函数等，配合对应的注解处理器完成`Bean`的自动配置工作。默认按照`Bean`的类型进行装配。
6. `@Resource`
作用与`Autowired`一样。区别在于`@Autowired`默认按照`Bean`类型装配，而`@Resource`默认按照`Bean`实例名称进行装配。
`@Resource`中有两个重要属性：`name`和`type`。
Spring 将`name`属性解析为`Bean`实例名称，`type`属性解析为`Bean`实例类型。如果指定`name`属性，则按实例名称进行装配；如果指定`type`属性，则按`Bean`类型进行装配。
如果都不指定，则先按`Bean`实例名称装配，如果不能匹配，则再按照`Bean`类型进行装配；如果都无法匹配，则抛出`NoSuchBeanDefinitionException`异常。
7. `@Qualifier`
与`@Autowired`注解配合使用，会将默认的按`Bean`类型装配修改为按`Bean`的实例名称装配，`Bean`的实例名称由`@Qualifier`注解的参数指定。

## 示例
```java
package net.biancheng;
public interface UserDao {
  /**
    * 输出方法
    */
  public void outContent();
}
```
```java
package net.biancheng;
import org.springframework.stereotype.Repository;
@Repository("userDao")
public class UserDaoImpl implements UserDao {
  @Override
  public void outContent() {
    System.out.println("hello");
  }
}
```
```java
package net.biancheng;
public interface UserService {
  /**
    * 输出方法
    */
  public void outContent();
}
```
```java
package net.biancheng;
import javax.annotation.Resource;
import org.springframework.stereotype.Service;
@Service("userService")
public class UserServiceImpl implements UserService{
   
  @Resource(name="userDao")
  private UserDao userDao;
  
  public UserDao getUserDao() {
    return userDao;
  }
  public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
  }
  @Override
  public void outContent() {
    userDao.outContent();
    System.out.println("world");
  }
}
```
```java
package net.biancheng;
import javax.annotation.Resource;
import org.springframework.stereotype.Controller;
@Controller("userController")
public class UserController {
  @Resource(name = "userService")
  private UserService userService;
  public UserService getUserService() {
    return userService;
  }
  public void setUserService(UserService userService) {
    this.userService = userService;
  }
  public void outContent() {
    userService.outContent();
    System.out.println("test");
  }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    <!--使用context命名空间，通知spring扫描指定目录，进行注解的解析 -->
    <context:component-scan base-package="net.biancheng" />
</beans>
```
```java
package net.biancheng;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
  public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("Beans.xml");
    UserController uc = (UserController) ctx.getBean("userController");
    uc.outContent();
  }
}
```
运行结果如下。
```
hello
world
test
```