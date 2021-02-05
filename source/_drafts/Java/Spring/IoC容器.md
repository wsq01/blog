


# IoC 概述
控制反转(`Inverse of Control, Ioc`)就是通过容器来控制业务对象之间的依赖关系，而非传统实现中由代码直接操控。这也就是控制反转概念的所在：控制权由应用代码中转到了外部容器，控制权的转移，就是反转。控制权转移带来的好处就是降低了业务对象之间的依赖程度。
# BeanFactory 和 ApplicationContext
Spring 通过一个配置文件描述了`Bean`及`Bean`之间的依赖关系，利用反射功能实例化`Bean`并建立`Bean`之间的依赖关系。Spring 的 IoC 容器在完成这些底层工作的基础上，还提供了`Bean`实例缓存、生命周期管理、`Bean`实例代理、事件发布、资源装载等高级服务。

`Bean`工厂(`com.springframework.beans.factory.BeanFactory`)是 Spring 框架最核心的接口，它提供了高级 IoC 的配置机制。`BeanFactory`使管理不同类型的 Java 对象成为可能，应用上下文(`com.springframework.context.ApplicationContext`)建立在`BeanFactory`基础之上，提供了更多面向应用的功能，它还提供了国际化支持和框架事件体系，更易于创建实际应用。一般称`BeanFactory`为 IoC 容器，`ApplicationContext`为应用上下文。
## BeanFactory介绍



# Bean 装配
## Bean 基本配置
```xml
<bean id="foo" class="com.smart.bean.Foo"></bean>
```
其中，`id`为这个`Bean`的名称，通过容器的`getBean("id")`即可获得对应的`Bean`，在容器中起到定位查找的作用。`class`属性指定了`Bean`对应的实现类。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="car" class="com.smart.simple.Car"></bean>
  <bean id="boss" class="com.smart.simple.Boss"></bean>
</beans>
```
上面基于 XML 的配置文件定义了两个`Bean`，这段配置信息提供了实例化`Car`和`Boss`这两个`Bean`必须的信息，Spring IoC 容器完全可以据此创建这两个`Bean`的实例。

一般在配置一个`Bean`时，需要为其指定一个`id`属性作为`Bean`的名称。`id`在 IoC 容器中必须是唯一的。`id`的命名需要满足 XML 对`id`的命名规范。如果希望用到一些特殊字符对`Bean`命名，可以使用`name`属性，`name`属性没有字符上的限制，几乎可以使用任何字符。

`id`和`name`都可以指定多个名字，名字之间可用逗号、分号或空格进行分隔。
```xml
<bean name="#car1,123,$car" class="com.smart.simple.Car"></bean>
```
Spring 配置文件不允许出现两个相同`id`的`Bean`，但却可以出现两个相同`name`的`Bean`。如果有多个`name`相同的`Bean`，通过`getBean()`获取`Bean`时，将返回最后声明的那个`Bean`。

如果`id`和`name`都未指定，如`<bean class="com.smart.simple.Car"></bean>`Spring 自动将全限定类名作为`Bean`的名称，就可以通过`getBean("com.smart.simple.Car")`获取`Car Bean`。

如果存在多个实现类相同的匿名`Bean`，如：
```xml
<bean class="com.smart.simple.Car"></bean>
<bean class="com.smart.simple.Car"></bean>
<bean class="com.smart.simple.Car"></bean>
```
第一个`Bean`通过`getBean("com.smart.simple.Car")`获得；第二个`Bean`通过`getBean("com.smart.simple.Car#1")`获得；第三个`Bean`通过`getBean("com.smart.simple.Car#2")`获得，以此类推。一般匿名`Bean`出现在通过内部`Bean`为外层`Bean`提供注入值时使用。

## 依赖注入
### 属性注入
属性注入即通过`setXxx()`方法注入`Bean`的属性值或依赖对象，由于属性注入方式具有可选择性和灵活性高的优点，因此属性注入是最常用的注入方式。

属性注入要求`Bean`提供一个默认的构造函数，并为需要注入的属性提供对应的`Setter`方法。Spring 先调用`Bean`的默认构造函数实例化`Bean`对象，然后通过反射的方式调用`Setter`方法注入属性值。
```java
public class Car {
  private int maxSpeed;
  public String brand;
  private double price;
  public void setBrand(String brand) {
    this.brand = brand;
  }
  public void setMaxSpeed(int maxSpeed) {
    this.maxSpeed = maxSpeed;
  }
  public vooid setPrice(double price) {
    this.price = price;
  }
}
```
注意，默认构造函数是不带参数的构造函数，如果类中没有定义任何构造函数，JVM 自动为其生成一个默认的构造函数。反之，如果类中显示定义了构造函数，JVM 不会为其生成默认的构造函数。所以假设`Car`类中显示定义了一个带参的构造函数，则需要同时提供一个默认构造函数`public Car()`，否则使用属性注入时将抛出异常。

```xml
<bean id="car" class="com.smart.simple.Car">
  <property name="maxSpeed" value="200"></property>
  <property name="brand" value="红旗 SV"></property>
  <property name="price" value="200000"></property>
</bean>
```
`Bean`的每一个属性对应一个`<property>`标签，`name`为属性的名称，在`Bean`实现类中拥有与其对应的`Setter`方法。

需要指出的是：Spring 只会检查`Bean`中是否有对应的`Setter`方法，至于`Bean`中是否有对应的属性变量则不作要求。虽然如此，但在一般情况下，还是会在`Bean`中提供同名的属性变量。

### 构造函数注入
构造函数注入保证一些必要的属性在`Bean`实例化时就得到设置，并且确保了`Bean`实例在实例化后就可以使用。

如果任何可用的`Car`对象都必须提供`brand`和`price`的值，使用属性注入方式只能人为在配置时提供保证，而无法在语法级提供保证，这时通过构造函数注入就能很好满足这一要求。

使用构造函数注入的前提是`Bean`必须提供带参的构造函数。
```java
public class Car {
  ...
  public Car(String brand, double price) {
    this.brand = brand;
    this.price = price;
  }
}
```
```xml
<bean id="car" class="com.smart.ditype.Car">
  <constructor-arg name="brand" value="红旗 SV" />
  <constructor-arg name="price" value="200000" />
</bean>
```


## Bean 作用域
