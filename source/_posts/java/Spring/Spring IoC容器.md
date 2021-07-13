---
title: Spring Ioc容器
date: 2021-04-06 17:43:22
tags: [Spring]
categories: [Spring]
---

# IoC容器
IoC 容器是 Spring 的核心，也可以称为 Spring 容器。Spring 通过 IoC 容器来管理对象的实例化和初始化，以及对象从创建到销毁的整个生命周期。

Spring 中使用的对象都由 IoC 容器管理，不需要手动使用`new`运算符创建对象。由 IoC 容器管理的对象称为`Spring Bean`，`Spring Bean`就是 Java 对象，和使用`new`运算符创建的对象没有区别。

Spring 通过读取 XML 或 Java 注解中的信息来获取哪些对象需要实例化。

Spring 提供 2 种不同类型的 IoC 容器，即`BeanFactory`和`ApplicationContext`容器。
## BeanFactory容器
`BeanFactory`是最简单的 IoC 容器，由`org.springframework.beans.facytory.BeanFactory`接口定义，采用懒加载，所以容器启动比较快。

为了兼容 Spring 集成的第三方框架（如`BeanFactoryAware`、`InitializingBean`），所以目前仍然保留了该接口。

简单来说，`BeanFactory`就是一个管理`Bean`的工厂，它主要负责初始化各种`Bean`，并调用它们的生命周期方法。

`BeanFactory`接口有多个实现类，最常见的是`org.springframework.beans.factory.xml.XmlBeanFactory`，它是根据 XML 配置文件中的定义装配`Bean`的。

使用`BeanFactory`需要创建`XmlBeanFactory`类的实例，通过`XmlBeanFactory`类的构造函数来传递`Resource`对象。
```java
Resource resource = new ClassPathResource("applicationContext.xml"); 
BeanFactory factory = new XmlBeanFactory(resource);  
```
## ApplicationContext容器
`ApplicationContext`继承了`BeanFactory`接口，由`org.springframework.context.ApplicationContext`接口定义，对象在启动容器时加载。

`ApplicationContext`在`BeanFactory`的基础上增加了很多企业级功能，例如 AOP、国际化、事件支持等。

`ApplicationContext`接口有两个常用的实现类。
### 1.ClassPathXmlApplicationContext
该类从类路径`ClassPath`中寻找指定的 XML 配置文件，并完成`ApplicationContext`的实例化工作。
```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext(String configLocation);
```
`configLocation`参数用于指定 Spring 配置文件的名称和位置，如`applicationContext.xml`。
### 2. FileSystemXmlApplicationContext
该类从指定的文件系统路径中寻找指定的 XML 配置文件，并完成`ApplicationContext`的实例化工作。
```java
ApplicationContext applicationContext = new FileSystemXmlApplicationContext(String configLocation);
```
它与`ClassPathXmlApplicationContext`的区别是：在读取 Spring 的配置文件时，`FileSystemXmlApplicationContext`不再从类路径中读取配置文件，而是通过参数指定配置文件的位置，它可以获取类路径之外的资源，如`F:/workspaces/applicationContext.xml`。

在使用 Spring 框架时，可以通过实例化其中任何一个类创建 Spring 的`ApplicationContext`容器。

通常在 Java 项目中，会采用通过`ClassPathXmlApplicationContext`类实例化`ApplicationContext`容器的方式，而在 Web 项目中，`ApplicationContext`容器的实例化工作会交由 Web 服务器完成。Web 服务器实例化`ApplicationContext`容器通常使用基于`ContextLoaderListener`实现的方式，它只需要在`web.xml`中添加如下代码：
```xml
<!--指定Spring配置文件的位置，有多个配置文件时，以逗号分隔-->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <!--spring将加载spring目录下的applicationContext.xml文件-->
  <param-value>
    classpath:spring/applicationContext.xml
  </param-value>
</context-param>
<!--指定以ContextLoaderListener方式启动Spring容器-->
<listener>
  <listener-class>
    org.springframework.web.context.ContextLoaderListener
  </listener-class>
</listener>
```
需要注意的是，`BeanFactory`和`ApplicationContext`都是通过 XML 配置文件加载`Bean`的。

二者的主要区别在于，如果`Bean`的某一个属性没有注入，则使用`BeanFacotry`加载后，在第一次调用`getBean()`方法时会抛出异常，而`ApplicationContext`则在初始化时自检，这样有利于检查所依赖的属性是否注入。

因此，在实际开发中，通常都选择使用`ApplicationContext`，而只有在系统资源较少时，才考虑使用`BeanFactory`。
# Bean定义
由 Spring IoC 容器管理的对象称为`Bean`，`Bean`根据 Spring 配置文件中的信息创建。

Spring 容器可以被看作一个大工厂，而 Spring 容器中的`Bean`就相当于该工厂的产品。如果希望这个大工厂能够生产和管理`Bean`，则需要告诉容器需要哪些`Bean`，以及需要以何种方式将这些`Bean`装配到一起。

Spring 配置文件支持两种格式，分别是`XML`文件格式和`Properties`文件格式。
* `Properties`配置文件主要以`key-value`键值对的形式存在，只能赋值，不能进行其他操作，适用于简单的属性配置。
* `XML`配置文件是树形结构，相对于`Properties`文件来说更加灵活。`XML`配置文件结构清晰，但是内容比较繁琐，适用于大型复杂的项目。

通常情况下，Spring 的配置文件使用`XML`格式，`XML`配置文件的根元素是`<beans>`，该元素包含了多个`<bean>`子元素，每一个`<bean>`子元素定义了一个`Bean`，并描述了该`Bean`如何被装配到 Spring 容器中。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
  <!-- 使用id属性定义person1，其对应的实现类为com.mengma.person1 -->
  <bean id="person1" class="com.mengma.damain.Person1" />
  <!--使用name属性定义person2，其对应的实现类为com.mengma.domain.Person2-->
  <bean name="Person2" class="com.mengma.domain.Person2"/>
</beans>
```
`<bean>`元素常用属性：

| 属性名称 | 描述 |
| :--: | :--: |
| id | 是一个 Bean 的唯一标识符，Spring 容器对 Bean 的配置和管理都通过该属性完成 |
| name | Spring 容器同样可以通过此属性对容器中的 Bean 进行配置和管理，name 属性中可以为 Bean 指定多个名称，每个名称之间用逗号或分号隔开 |
| class | 该属性指定了 Bean 的具体实现类，它必须是一个完整的类名，使用类的全限定名 |
| scope | 用于设定 Bean 实例的作用域，其属性值有 singleton（单例）、prototype（原型）、request、session 和 global Session。默认值是 singleton |
| constructor-arg | `<bean>`元素的子元素，可以使用此元素传入构造参数进行实例化。该元素的 index 属性指定构造参数的序号（从 0 开始），type 属性指定构造参数的类型 |
| property | `<bean>`元素的子元素，用于调用 Bean 实例中的 Set 方法完成属性赋值，从而完成依赖注入。该元素的 name 属性指定 Bean 实例中的相应属性名 |
| ref | `<property>`和`<constructor-arg>`等元素的子元素，该元素中的 bean 属性用于指定对 Bean 工厂中某个 Bean 实例的引用 |
| value | `<property>`和`<constractor-arg>`等元素的子元素，用于直接指定一个常量值 |
| list | 用于封装 List 或数组类型的依赖注入 |
| set | 用于封装 Set 类型属性的依赖注入 |
| map | 用于封装 Map 类型属性的依赖注入 |
| entry | `<map>`元素的子元素，用于设置一个键值对。其 key 属性指定字符串类型的键值，ref 或 value 子元素指定其值 |
| init-method | 容器加载 Bean 时调用该方法，类似于 Servlet 中的 init() 方法 |
| destroy-method | 容器删除 Bean 时调用该方法，类似于 Servlet 中的 destroy() 方法。该方法只在 scope=singleton 时有效 |
| lazy-init | 懒加载，值为 true，容器在首次请求时才会创建 Bean 实例；值为 false，容器在启动时创建 Bean 实例。该方法只在 scope=singleton 时有效 |

# Bean作用域
在配置文件中，除了可以定义`Bean`的属性值和相互之间的依赖关系，还可以声明`Bean`的作用域。例如，如果每次获取`Bean`时，都需要一个`Bean`实例，那么应该将`Bean`的`scope`属性定义为`prototype`，如果 Spring 需要每次都返回一个相同的`Bean`实例，则应将`Bean`的`scope`属性定义为`singleton`。
## 作用域的种类
Spring 容器在初始化一个`Bean`的实例时，同时会指定该实例的作用域。Spring 5 支持以下 6 种作用域。
1. `singleton`
默认值，单例模式，表示在 Spring 容器中只有一个`Bean`实例，`Bean`以单例的方式存在。
2. `prototype`
原型模式，表示每次通过 Spring 容器获取`Bean`时，容器都会创建一个`Bean`实例。
3. `request`
每次 HTTP 请求，容器都会创建一个`Bean`实例。该作用域只在当前 HTTP `Request`内有效。
4. `session`
同一个 HTTP Session 共享一个`Bean`实例，不同的`Session`使用不同的`Bean`实例。该作用域仅在当前 HTTP `Session`内有效。
5. `application`
同一个 Web 应用共享一个`Bean`实例，该作用域在当前`ServletContext`内有效。
类似于`singleton`，不同的是，`singleton`表示每个 IoC 容器中仅有一个`Bean`实例，而同一个 Web 应用中可能会有多个 IoC 容器，但一个 Web 应用只会有一个`ServletContext`，也可以说`application`才是 Web 应用中货真价实的单例模式。
6. `websocket`
`websocket`的作用域是`WebSocket`，即在整个`WebSocket`中有效。

Spring 5 版本之前还支持`global Session`，该值表示在一个全局的 HTTP `Session`中，容器会返回该`Bean`的同一个实例。一般用于 Portlet 应用环境。Spring 5.2.0 版本中已经将该值移除了。

`request、session、application、websocket`和`global Session`作用域只能在 Web 环境下使用，如果使用`ClassPathXmlApplicationContext`加载这些作用域中的任意一个的`Bean`，就会抛出以下异常。
```
java.lang.IllegalStateException: No Scope registered for scope name 'xxx'
```
## singleton
`singleton`是 Spring 容器默认的作用域，当`Bean`的作用域为`singleton`时，Spring 容器中只会存在一个共享的`Bean`实例，该`Bean`实例将存储在高速缓存中，并且所有对`Bean`的请求，只要`id`与该`Bean`定义相匹配，都会返回该缓存对象。

通常情况下，这种单例模式对于无会话状态的`Bean`（如 DAO 层、Service 层）来说，是最理想的选择。

在 Spring 配置文件中，可以使用`<bean>`元素的`scope`属性，将`Bean`的作用域定义成`singleton`：
```xml
<bean id="..." class="..." scope="singleton"/>
```
## prototype
使用`prototype`作用域的`Bean`会在每次请求该`Bean`时都会创建一个新的`Bean`实例。`prototype`作用域适用于需要保持会话状态的 `Bean`。

在 Spring 配置文件中，可以使用`<bean>`元素的`scope`属性，将`Bean`的作用域定义成`prototype`。
```xml
<bean id="..." class="..." scope="prototype"/>
```
# Bean生命周期
Spring 根据`Bean`的作用域来选择管理方式。对于`singleton`作用域的`Bean`，Spring 能够精确地知道该`Bean`何时被创建，何时初始化完成，以及何时被销毁；而对于`prototype`作用域的`Bean`，Spring 只负责创建，当容器创建了`Bean`的实例后，`Bean`的实例就交给客户端代码管理，Spring 容器将不再跟踪其生命周期。

了解 Spring 生命周期的意义就在于，可以利用`Bean`在其存活期间的指定时刻完成一些相关操作。这种时刻可能有很多，但一般情况下，会在`Bean`被初始化后和被销毁前执行一些相关操作。
## Bean生命周期执行流程
Spring 容器在确保一个`Bean`能够使用之前，会进行很多工作。Spring 容器中 Bean 的生命周期流程：

{% asset_img 1.png  Bean 的生命周期 %}

`Bean`生命周期的整个执行过程描述如下。
1. Spring 启动，查找并加载需要被 Spring 管理的`Bean`，并实例化`Bean`。
2. 利用依赖注入完成`Bean`中所有属性值的配置注入。
3. 如果`Bean`实现了`BeanNameAware`接口，则 Spring 调用`Bean`的`setBeanName()`方法传入当前`Bean`的`id`值。
4. 如果`Bean`实现了`BeanFactoryAware`接口，则 Spring 调用`setBeanFactory()`方法传入当前工厂实例的引用。
5. 如果`Bean`实现了`ApplicationContextAware`接口，则 Spring 调用`setApplicationContext()`方法传入当前`ApplicationContext`实例的引用。
6. 如果`Bean`实现了`BeanPostProcessor`接口，则 Spring 将调用该接口的预初始化方法`postProcessBeforeInitialzation()`对`Bean`进行加工操作，此处非常重要，Spring 的 AOP 就是利用它实现的。
7. 如果`Bean`实现了`InitializingBean`接口，则 Spring 将调用`afterPropertiesSet()`方法。
8. 如果在配置文件中通过`init-method`属性指定了初始化方法，则调用该初始化方法。
9. 如果`BeanPostProcessor`和`Bean`关联，则 Spring 将调用该接口的初始化方法`postProcessAfterInitialization()`。此时，`Bean`已经可以被应用系统使用了。
10. 如果在`<bean>`中指定了该`Bean`的作用范围为`scope="singleton"`，则将该`Bean`放入 Spring IoC 的缓存池中，将触发 Spring 对该`Bean`的生命周期管理；如果在`<bean>`中指定了该`Bean`的作用范围为`scope="prototype"`，则将该`Bean`交给调用者，调用者管理该`Bean`的生命周期，Spring 不再管理该`Bean`。
11. 如果`Bean`实现了`DisposableBean`接口，则 Spring 会调用`destory()`方法将 Spring 中的`Bean`销毁；如果在配置文件中通过`destory-method`属性指定了`Bean`的销毁方法，则 Spring 将调用该方法对`Bean`进行销毁。

Spring 为`Bean`提供了细致全面的生命周期过程，通过实现特定的接口或`<bean>`的属性设置，都可以对`Bean`的生命周期过程产生影响。建议不要过多地使用`Bean`实现接口，因为这样会导致代码的耦合性过高。

Spring 官方提供了 3 种方法实现初始化回调和销毁回调：
* 实现`InitializingBean`和`DisposableBean`接口；
* 在 XML 中配置`init-method`和`destory-method`；
* 使用`@PostConstruct`和`@PreDestory`注解。

在一个`Bean`中有多种生命周期回调方法时，优先级为：注解 > 接口 > XML。不建议使用接口和注解，这会让`pojo`类和 Spring 框架紧耦合。
## 初始化回调
### 1. 使用接口
`org.springframework.beans.factory.InitializingBean`接口提供了以下方法：`void afterPropertiesSet() throws Exception;`。

可以实现以上接口，在`afterPropertiesSet`方法内指定`Bean`初始化后需要执行的操作。
```java
<bean id="..." class="..." />

public class User implements InitializingBean {
  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("调用接口：InitializingBean，方法：afterPropertiesSet，无参数");
  }
}
```
### 2. 配置XML
可以通过`init-method`属性指定`Bean`初始化后执行的方法。
```java
<bean id="..." class="..." init-method="init"/>

public class User {
  public void init() {
    System.out.println("调用init-method指定的初始化方法:init" );
  }
}
```
### 3. 使用注解
使用`@PostConstruct`注解标明该方法为`Bean`初始化后的方法。
```java
public class ExampleBean {
  @PostConstruct
  public void init() {
    System.out.println("@PostConstruct注解指定的初始化方法:init" );
  }
}
```
## 销毁回调
### 1. 使用接口
`org.springframework.beans.factory.DisposableBean`接口提供了以下方法：
```java
void destroy() throws Exception;
```
您可以实现以上接口，在`destroy`方法内指定`Bean`初始化后需要执行的操作。
```java
<bean id="..." class="..." />
public class User implements InitializingBean {
  @Override
  public void afterPropertiesSet() throws Exception {
    System.out.println("调用接口：InitializingBean，方法：afterPropertiesSet，无参数");
  }
}
```
### 2. 配置XML
可以通过`destroy-method`属性指定`Bean`销毁后执行的方法。
```java
<bean id="..." class="..." destroy-method="destroy"/>
public class User {
  public void destroy() {
    System.out.println("调用destroy-method指定的销毁方法:destroy" );
  }
}
```
### 3. 使用注解
使用 @PreDestory 注解标明该方法为 Bean 销毁前执行的方法。
```java
public class ExampleBean {
  @PreDestory 
  public void destroy() {
    System.out.println("@PreDestory注解指定的初始化方法:destroy" );
  }
}
```
## 默认的初始化和销毁方法
如果多个`Bean`需要使用相同的初始化或者销毁方法，不用为每个`bean`声明初始化和销毁方法，可以使用`default-init-method`和`default-destroy-method`属性。
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init"
    default-destroy-method="destroy">
    <bean id="..." class="...">
        ...
    </bean>
</beans>
```
# 后置处理器BeanPostProcessor
`BeanPostProcessor`接口也被称为后置处理器，通过该接口可以自定义调用初始化前后执行的操作方法。

`BeanPostProcessor`接口源码如下：
```java
public interface BeanPostProcessor {
  Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
  Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```
`postProcessBeforeInitialization`在`Bean`实例化、依赖注入后，初始化前调用。`postProcessAfterInitialization`在`Bean`实例化、依赖注入、初始化都完成后调用。

当需要添加多个后置处理器实现类时，默认情况下 Spring 容器会根据后置处理器的定义顺序来依次调用。也可以通过实现`Ordered`接口的`getOrder`方法指定后置处理器的执行顺序。该方法返回值为整数，默认值为 0，值越大优先级越低。

# Bean继承
`Bean`定义可以包含很多配置信息，包括构造函数参数、属性值和容器的一些具体信息，如初始化方法、销毁方法等。子`Bean`可以继承父`Bean`的配置数据，根据需要，子`Bean`可以重写值或添加其它值。

需要注意的是，`Spring Bean`定义的继承与 Java 中的继承无关。您可以将父`Bean`的定义作为一个模板，其它子`Bean`从父`Bean`中继承所需的配置。

在配置文件中通过`parent`属性来指定继承的父`Bean`。
## 示例
`HelloWorld`类代码如下。
```java
package net.biancheng;

public class HelloWorld {
  private String message1;
  private String message2;

  public void setMessage1(String message) {
    this.message1 = message;
  }

  public void setMessage2(String message) {
    this.message2 = message;
  }

  public void getMessage1() {
    System.out.println("World Message1 : " + message1);
  }

  public void getMessage2() {
    System.out.println("World Message2 : " + message2);
  }
}
```
`HelloChina`类代码如下。
```java
package net.biancheng;

public class HelloChina {
  private String message1;
  private String message2;
  private String message3;

  public void setMessage1(String message) {
    this.message1 = message;
  }

  public void setMessage2(String message) {
    this.message2 = message;
  }

  public void setMessage3(String message) {
    this.message3 = message;
  }

  public void getMessage1() {
    System.out.println("China Message1 : " + message1);
  }

  public void getMessage2() {
    System.out.println("China Message2 : " + message2);
  }

  public void getMessage3() {
    System.out.println("China Message3 : " + message3);
  }
}
```
在配置文件中，分别为`HelloWorld`中的`message1`和`message2`赋值。使用`parent`属性将`HelloChain`定义为`HelloWorld`的子类，并为`HelloChain`中的`message1`和`message3`赋值。`Beans.xml`文件代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="helloWorld" class="net.biancheng.HelloWorld">
    <property name="message1" value="Hello World！" />
    <property name="message2" value="Hello World2！" />
  </bean>
  
  <bean id="helloChina" class="net.biancheng.HelloChina" parent="helloWorld">
    <property name="message1" value="Hello China！" />
    <property name="message3" value="Hello China3！" />
  </bean>
</beans>
```
MainApp 类代码如下。
```java
package net.biancheng;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainApp {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");

        HelloWorld objA = (HelloWorld) context.getBean("helloWorld");
        objA.getMessage1();
        objA.getMessage2();

        HelloChina objB = (HelloChina) context.getBean("helloChina");
        objB.getMessage1();
        objB.getMessage2();
        objB.getMessage3();
    }
}

//运行结果如下。
//World Message1 : Hello World！
//World Message2 : Hello World2！
//China Message1 : Hello China！
//China Message2 : Hello World2！
//China Message3 : Hello China3！
```
由结果可以看出，我们在创建`helloChina`时并没有给`message2`赋值，但是由于`Bean`的继承，将值传递给了`message2`。
## Bean定义模板
您可以创建一个`Bean`定义模板，该模板只能被继承，不能被实例化。创建`Bean`定义模板时，不用指定`class`属性，而是指定`abstarct="true"`将该`Bean`定义为抽象`Bean`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="beanTeamplate" abstract="true">
    <property name="message1" value="Hello World!" />
    <property name="message2" value="Hello World2!" />
    <property name="message3" value="Hello World3!" />
  </bean>

  <bean id="helloChina" class="net.biancheng.HelloChina" parent="beanTeamplate">
    <property name="message1" value="Hello China!" />
    <property name="message3" value="Hello China!" />
  </bean>
</beans>
```
# Spring实例化Bean的三种方法
在 Spring 中，实例化`Bean`有三种方式，分别是构造器实例化、静态工厂方式实例化和实例工厂方式实例化。
## 构造器实例化
构造器实例化是指 Spring 容器通过`Bean`对应的类中默认的构造函数实例化`Bean`。
### 1. 创建项目并导入 JAR 包
创建一个名称为`springDemo02`的 Web 项目，然后将 Spring 支持和依赖的 JAR 包复制到项目的`lib`目录中，并发布到类路径下。
### 2. 创建实体类
在项目的`src`目录下创建一个名为`com.mengma.instance.constructor`的包，在该包下创建一个实体类`Person1`。
```java
package com.mengma.instance.constructor;
public class Person1 {}
```
### 3. 创建 Spring 配置文件
在`com.mengma.instance.constructor`包下创建 Spring 的配置文件`applicationContext.xml`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
  <bean id="person1" class="com.mengma.instance.constructor.Person1" />
</beans>
```
在上述配置中，定义了一个`id`为`person1`的`Bean`，其中`class`属性指定了其对应的类为`Person1`。
### 4. 创建测试类
在`com.mengma.instance.constructor`包下创建一个名为`InstanceTest1`的测试类。
```java
package com.mengma.instance.constructor;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class InstanceTest1 {
  @Test
  public void test() {
    // 定义Spring配置文件的路径
    String xmlPath = "com/mengma/instance/constructor/ApplicationContext.xml";
    // 初始化Spring容器，加载配置文件，并对bean进行实例化
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
    // 通过容器获取id为person1的实例
    System.out.println(applicationContext.getBean("person1"));
  }
}
```
上述文件中，首先在`test()`方法中定义了 Spring 配置文件的路径，然后 Spring 容器会加载配置文件。在加载的同时，Spring 容器会通过实现类`Person1`中默认的无参构造函数对`Bean`进行实例化。
## 静态工厂方式实例化
也可以使用静态工厂的方式实例化`Bean`。此种方式需要提供一个静态工厂方法创建`Bean`的实例。
### 1. 创建实体类
在项目的`src`目录下创建一个名为`com.mengma.instance.static_factory `的包，并在该包下创建一个实体类`Person2`。
### 2. 创建静态工厂类
在`com.mengma.instance.static_factory`包下创建一个名为`MyBeanFactory`的类，并在该类中创建一个名为`createBean()`的静态方法，用于创建`Bean`的实例。
```java
package com.mengma.instance.static_factory;
public class MyBeanFactory {
  // 创建Bean实例的静态工厂方法
  public static Person2 createBean() {
      return new Person2();
  }
}
```
### 3. 创建 Spring 配置文件
在`com.mengma.instance.static_factory`包下创建 Spring 的配置文件`applicationContext.xml`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
    <bean id="person2" class="com.mengma.instance.static_factory.MyBeanFactory"
        factory-method="createBean" />
</beans>
```
上述代码中，定义了一个`id`为`person2`的`Bean`，其中`class`属性指定了其对应的工厂实现类为`MyBeanFactory`，而`factory-method`属性用于告诉 Spring 容器调用工厂类中的`createBean()`方法获取`Bean`的实例。
### 4. 创建测试类
在`com.mengma.instance.static_factory`包下创建一个名为`InstanceTest2`的测试类。
```java
package com.mengma.instance.static_factory;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class InstanceTest2 {
  @Test
  public void test() {
    // 定义Spring配置文件的路径
    String xmlPath = "com/mengma/instance/static_factory/applicationContext.xml";
    // 初始化Spring容器，加载配置文件，并对bean进行实例化
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
    // 通过容器获取id为person2实例
    System.out.println(applicationContext.getBean("person2"));
  }
}
```
## 实例工厂方式实例化
还有一种实例化`Bean`的方式就是采用实例工厂。在这种方式中，工厂类不再使用静态方法创建`Bean`的实例，而是直接在成员方法中创建`Bean`的实例。

同时，在配置文件中，需要实例化的`Bean`也不是通过`class`属性直接指向其实例化的类，而是通过`factory-bean`属性配置一个实例工厂，然后使用`factory-method`属性确定使用工厂中的哪个方法。
### 1. 创建实体类
在`src`目录下创建一个名为`com.mengma.instance.factory`的包，在该包下创建一个`Person3`类。
### 2. 创建实例工厂类
在`com.mengma.instance.factory`包下创建一个名为`MyBeanFactory`的类。
```java
package com.mengma.instance.factory;
public class MyBeanFactory {
  public MyBeanFactory() {
    System.out.println("person3工厂实例化中");
  }
  // 创建Bean的方法
  public Person3 createBean() {
    return new Person3();
  }
}
```
上述代码中，使用默认无参的构造方法输出`person3`工厂实例化中语句，使用`createBean`成员方法创建`Bean`的实例。
### 3. 创建 Spring 配置文件
在`com.mengma.instance.factory`包下创建 Spring 的配置文件`applicationContext.xml`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">
    <!-- 配置实例工厂 -->
    <bean id="myBeanFactory" class="com.mengma.instance.factory.MyBeanFactory" />
    <!-- factory-bean属性指定一个实例工厂，factory-method属性确定使用工厂中的哪个方法 -->
    <bean id="person3" factory-bean="myBeanFactory" factory-method="createBean" />
</beans>
```
上述代码中，首先配置了一个实例工厂`Bean`，然后配置了需要实例化的`Bean`。在`id`为`person3`的`Bean`中，使用`factory-bean`属性指定一个实例工厂，该属性值就是实例工厂的`id`属性值。使用`factory-method`属性确定使用工厂中的`createBean()`方法。
### 4. 创建测试类
在`com.mengma.instance.factory`包下创建一个名为`InstanceTest3`的测试类。
```java
package com.mengma.instance.factory;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class InstanceTest3 {
  @Test
  public void test() {
    // 定义Spring配置文件的路径
    String xmlPath = "com/mengma/instance/factory/applicationContext.xml";
    // 初始化Spring容器，加载配置文件，并对bean进行实例化
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext(xmlPath);
    // 通过容器获取id为person3实例
    System.out.println(applicationContext.getBean("person3"));
  }
}
```
