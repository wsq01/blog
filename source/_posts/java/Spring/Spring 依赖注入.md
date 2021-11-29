---
title: Spring 依赖注入
date: 2021-05-19 16:24:15
tags: [Spring]
categories: [Spring]
---


# Spring 依赖注入
依赖注入（`Dependency Injection，DI`）和控制反转含义相同，它们是从两个角度描述的同一个概念。

当某个 Java 实例需要另一个 Java 实例时，传统的方法是由调用者创建被调用者的实例（例如，使用`new`关键字获得被调用者实例），而使用 Spring 框架后，被调用者的实例不再由调用者创建，而是由 Spring 容器创建，这称为控制反转。

Spring 容器在创建被调用者的实例时，会自动将调用者需要的对象实例注入给调用者，这样，调用者通过 Spring 容器获得被调用者实例，这称为依赖注入。

依赖注入主要有两种实现方式，分别是`setter`注入（又称设值注入）和构造函数注入。
1. `setter`注入
指 IoC 容器使用`setter`方法注入被依赖的实例。通过调用无参构造器或无参`static`工厂方法实例化`bean`后，调用该`bean`的`setter`方法，即可实现基于`setter`的 DI。
2. 构造函数注入
指 IoC 容器使用构造函数注入被依赖的实例。可以通过调用带参数的构造函数实现依赖注入，每个参数代表一个依赖。

使用`setter`注入时，在 Spring 配置文件中，需要使用`<bean>`元素的子元素`<property>`为每个属性注入值。而使用构造注入时，在配置文件中，主要使用`<constructor-arg>`标签定义构造方法的参数，使用其`value`属性（或子元素）设置该参数的值。

## 构造函数注入
在`<constructor-arg>`标签中，包含`ref、value、type、index`等属性。`value`属性用于注入基本数据类型以及字符串类型的值；`ref`属性用于注入已经定义好的`Bean`；`type`属性用来指定对应的构造函数，当构造函数有多个参数时，可以使用`index`属性指定参数的位置，`index`属性值从 0 开始。
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
  <bean id="person" class="net.biancheng.Person">
    <constructor-arg ref="man" type="java.lang.String"/>
  </bean>
</beans>
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

//运行结果如下
//在man的有参构造函数内
//在Person的构造函数内
//名称：bianchengbang
//年龄：12
```
## setter注入
在`<property>`标签中，包含`name、ref、value`等属性。`name`用于指定参数名称；`value`属性用于注入基本数据类型以及字符串类型的值；`ref`属性用于注入已经定义好的`Bean`。
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
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
  <bean id="person" class="net.biancheng.Person">
    <property name="man" ref="man" />
  </bean>
  <bean id="man" class="net.biancheng.Man">
    <property name="name" value="bianchengbang" />
    <property name="age" value="12" />
  </bean>
</beans>

<!-- 运行结果如下。
在man的构造函数内
在setMan方法内
名称：bianchengbang
年龄：12 -->
```
# 注入内部Bean
Java 中在类内部定义的类称为内部类，同理在`Bean`中定义的`Bean`称为内部`Bean`。注入内部`Bean`使用`<property>`和`<constructor-arg>`中的`<bean>`标签。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
  <bean id="outerBean" class="...">
    <property name="target">
      <!-- 定义内部Bean -->
      <bean class="..." />
    </property>
  </bean>
</beans>
```
内部`Bean`的定义不需要指定`id`和`name`。如果指定了，容器也不会将其作为区分`Bean`的标识符，反而会无视内部`Bean`的`scope`属性。所以内部`Bean`总是匿名的，而且总是随着外部`Bean`创建。

在实际开发中很少注入内部`Bean`，因为开发者无法将内部的`Bean`注入外部`Bean`以外的其它`Bean`。
# 注入集合
如果需要传递类似于 Java `Collection`类型的值，例如`List、Set、Map`和`properties`，可以使用 Spring 提供的集合配置标签。

| 标签 | 说明 |
| :--: | :--: |
| `<list>` | 用于注入`list`类型的值，允许重复 |
| `<set>` | 用于注入`set`类型的值，不允许重复 |
| `<map>` | 用于注入`key-value`的集合，其中`key-value`可以是任意类型 |
| `<props>` | 用于注入`key-value`的集合，其中`key-value`都是字符串类型 |

```java
package net.biancheng;

import java.util.*;

public class JavaCollection {
  List manList;
  Set manSet;
  Map manMap;
  Properties manProp;

  public void setManList(List manList) {
    this.manList = manList;
  }

  public List getManList() {
    System.out.println("List Elements :" + manList);
    return manList;
  }

  public void setManSet(Set manSet) {
    this.manSet = manSet;
  }

  public Set getManSet() {
  System.out.println("Set Elements :" + manSet);
  return manSet;
  }

  public void setManMap(Map manMap) {
    this.manMap = manMap;
  }

  public Map getManMap() {
    System.out.println("Map Elements :" + manMap);
    return manMap;
  }

  public void setManProp(Properties manProp) {
    this.manProp = manProp;
  }

  public Properties getManProp() {
    System.out.println("Property Elements :" + manProp);
    return manProp;
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
    JavaCollection jc = (JavaCollection) context.getBean("javaCollection");

    jc.getManList();
    jc.getManSet();
    jc.getManMap();
    jc.getManProp();
  }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="javaCollection" class="net.biancheng.JavaCollection">

    <property name="manList">
      <list>
        <value>a</value>
        <value>b</value>
        <value>c</value>
        <value>d</value>
      </list>
    </property>

    <property name="manSet">
      <set>
        <value>a</value>
        <value>b</value>
        <value>c</value>
        <value>d</value>
      </set>
    </property>

    <property name="manMap">
      <map>
        <entry key="1" value="a" />
        <entry key="2" value="b" />
        <entry key="3" value="c" />
        <entry key="4" value="d" />
      </map>
    </property>

    <property name="manProp">
      <props>
        <prop key="one">a</prop>
        <prop key="two">b</prop>
        <prop key="three">C</prop>
        <prop key="four">C</prop>
      </props>
    </property>
  </bean>
</beans>
```
运行结果如下。
```
List Elements :[a, b, c, d]
Set Elements :[a, b, c]
Map Elements :{1=a, 2=b, 3=c, 4=d}
Property Elements :{two=b, one=a, three=c, four=d}
```
# 注入Bean引用
也可以在集合元素中注入`Bean`。
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="..." class="...">

    <property name="manList">
      <list>
        <ref bean="man1" />
        <ref bean="man2" />
        <value>a</value>
      </list>
    </property>

    <property name="manSet">
      <set>
        <ref bean="man1" />
        <ref bean="man2" />
        <value>a</value>
      </set>
    </property>

    <property name="manMap">
      <map>
        <entry key="one" value="a" />
        <entry key="two" value-ref="man1" />
        <entry key="three" value-ref="man2" />
      </map>
    </property>
  </bean>
</beans>
```
# 注入null和空字符串的值
Spring 会把属性的空参数直接当成空字符串来处理，如果您需要传递一个空字符串值，可以这样写：
```xml
<bean id = "..." class = "exampleBean">
    <property name = "email" value = ""/>
</bean>
```
等效于以下代码
```java
exampleBean.setEmail("")
```
如果需要传递`NULL`值，`<null/>`元素用来处理`Null`值。
```xml
<bean id = "..." class = "exampleBean">
  <property name = "email"><null/></property>
</bean>
```
等效于以下代码
```java
exampleBean.setEmail(null)
```