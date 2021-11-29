---
title: Spring AOP
date: 2021-05-26 16:24:15
tags: [Spring]
categories: [Spring]
---

AOP(`Aspect Oriented Programming`) 采取横向抽取机制（动态代理），取代了传统纵向继承机制的重复性代码，其应用主要体现在事务处理、日志管理、权限控制、异常处理等方面。主要作用是分离功能性需求和非功能性需求，使开发人员可以集中处理某一个关注点或者横切逻辑，减少对业务代码的侵入，增强代码的可读性和可维护性。

简单的说，AOP 的作用就是保证开发者在不修改源代码的前提下，为系统中的业务组件添加某种通用功能。AOP 就是代理模式的典型应用。

目前最流行的 AOP 框架有两个，分别为 Spring AOP 和 AspectJ。

Spring AOP 是基于 AOP 编程模式的一个框架，它能够有效的减少系统间的重复代码，达到松耦合的目的。Spring AOP 使用纯 Java 实现，不需要专门的编译过程和类加载器，在运行期间通过代理方式向目标类植入增强的代码。有两种实现方式：基于接口的 JDK 动态代理和基于继承的 CGLIB 动态代理。

AspectJ 是一个基于 Java 语言的 AOP 框架。AspectJ 扩展了 Java 语言，提供了一个专门的编译器，在编译时提供横向代码的植入。
# AOP术语

| 名称 | 说明 |
| :--: | :--: |
| Joinpoint（连接点）	| 指那些被拦截到的点，在 Spring 中，可以被动态代理拦截目标类的方法。 |
| Pointcut（切入点） | 指要对哪些 Joinpoint 进行拦截，即被拦截的连接点。 |
| Advice（通知） | 指拦截到 Joinpoint 之后要做的事情，即对切入点增强的内容。 |
| Target（目标） | 指代理的目标对象。 |
| Weaving（植入） | 指把增强代码应用到目标上，生成代理对象的过程。 |
| Proxy（代理） | 指生成的代理对象。 |
| Aspect（切面） | 切入点和通知的结合。 |

`Advice`直译为通知，也有的资料翻译为“增强处理”，共有 5 种类型。

| 通知 | 说明 |
| :--: | :--: |
| before（前置通知） | 通知方法在目标方法调用之前执行 |
| after（后置通知） | 通知方法在目标方法返回或异常后调用 |
| after-returning（返回后通知） | 通知方法会在目标方法返回后调用 |
| after-throwing（抛出异常通知） | 通知方法会在目标方法抛出异常后调用 |
| around（环绕通知） | 通知方法会将目标方法封装起来 |

使用 AOP 主要有以下优势。
* 提供声明式企业服务，这种服务是声明式事务管理。
* 允许用户实现自定义切面。在某些不适合用 OOP 编程的场景中，采用 AOP 来补充。
* 可以对业务逻辑的各个部分进行隔离，从而使业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时也提高了开发效率。

# JDK动态代理
Spring JDK 动态代理需要实现`InvocationHandler`接口，重写`invoke`方法，客户端使用`Java.lang.reflect.Proxy`类产生动态代理类的对象。
## 示例
步骤如下：
* 创建`SpringDemo`项目，并在`src`目录下创建`net.biancheng`包。
* 在`net.biancheng`包下创建`UserManager`（用户管理接口）、`UserManagerImpl`（用户管理接口实现类）、`MyAspect`（切面类）和`JdkProxy`（动态代理类）。
* 运行`SpringDemo`项目。

```java
package net.biancheng;
public interface UserManager {
  // 新增用户抽象方法
  void addUser(String userName, String password);
  // 删除用户抽象方法
  void delUser(String userName);
}
```
```java
package net.biancheng;
public class UserManagerImpl implements UserManager {
  @Override
  public void addUser(String userName, String password) {
    System.out.println("正在执行添加用户方法");
    System.out.println("用户名称: " + userName + " 密码: " + password);
  }
  @Override
  public void delUser(String userName) {
    System.out.println("正在执行删除用户方法");
    System.out.println("用户名称: " + userName);
  }
}
```
```java
package net.biancheng;
public class MyAspect {
  public void myBefore() {
    System.out.println("方法执行之前");
  }
  public void myAfter() {
    System.out.println("方法执行之后");
  }
}
```
```java
package net.biancheng;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
/**
* JDK动态代理实现InvocationHandler接口
*
*/
public class JdkProxy implements InvocationHandler {
  private Object target; // 需要代理的目标对象
  final MyAspect myAspect = new MyAspect();
  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    myAspect.myBefore();
    Object result = method.invoke(target, args);
    myAspect.myAfter();
    return result;
  }
  // 定义获取代理对象方法
  private Object getJDKProxy(Object targetObject) {
    // 为目标对象target赋值
    this.target = targetObject;
    
    // JDK动态代理只能代理实现了接口的类，从 newProxyInstance 函数所需的参数就可以看出来
    return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(), targetObject.getClass().getInterfaces(),this);
  }
  public static void main(String[] args) {
    JdkProxy jdkProxy = new JdkProxy();    // 实例化JDKProxy对象
    UserManager user = (UserManager) jdkProxy.getJDKProxy(new UserManagerImpl());    // 获取代理对象
    user.addUser("zhangsan", "123456");    // 执行新增方法
    user.delUser("zhangsan");    // 执行删除方法
  }
}
```
运行结果如下。
```
方法执行之前
正在执行添加用户方法
用户名称: zhangsan 密码: 123456
方法执行之后
方法执行之前
正在执行删除用户方法
用户名称: zhangsan
方法执行之后
```
# CGLlB动态代理
JDK 动态代理使用起来非常简单，但是 JDK 动态代理的目标类必须要实现一个或多个接口，具有一定的局限性。如果不希望实现接口，可以使用 CGLIB代理。

CGLIB（`Code Generation Library`）是一个高性能开源的代码生成包，它被许多 AOP 框架所使用，其底层是通过使用一个小而快的字节码处理框架 ASM（Java 字节码操控框架）转换字节码并生成新的类。使用 CGLIB 需要导入 CGLIB 和 ASM 包，即`asm-x.x.jar`和`CGLIB-x.x.x.jar`。如果您已经导入了 Spring 的核心包`spring-core-x.x.x.RELEASE.jar`，就不用再导入`asm-x.x.jar`和`cglib-x.x.x.jar`了。

Spring 核心包中包含 CGLIB 和 asm，也就是说 Spring 核心包已经集成了 CGLIB 所需要的包，所以在开发中不需要另外导入`asm-x.x.jar`和`cglib-x.x.x.jar`包了。
## 示例
步骤如下：
* 创建`SpringDemo`项目，并在 src 目录下创建`net.biancheng`包。
* 导入相关 JAR 包。
* 在`net.biancheng`包下创建`UserManager`（用户管理接口）、`UserManagerImpl`（用户管理接口实现类）、`MyAspect`（切面类）和`CGLIBProxy`（动态代理类）。
* 运行`SpringDemo`项目。

```java
package net.biancheng;
public interface UserManager {
    // 新增用户抽象方法
    void addUser(String userName, String password);
    // 删除用户抽象方法
    void delUser(String userName);
}
```
```java
package net.biancheng;
public class UserManagerImpl implements UserManager {
    @Override
    public void addUser(String userName, String password) {
        System.out.println("正在执行添加用户方法");
        System.out.println("用户名称: " + userName + " 密码: " + password);
    }
    @Override
    public void delUser(String userName) {
        System.out.println("正在执行删除用户方法");
        System.out.println("用户名称: " + userName);
    }
}
```
```java
package net.biancheng;
public class MyAspect {
    public void myBefore() {
        System.out.println("方法执行之前");
    }
    public void myAfter() {
        System.out.println("方法执行之后");
    }
}
```
```java
package net.biancheng;
import java.lang.reflect.Method;
import org.springframework.CGLIB.proxy.Enhancer;
import org.springframework.CGLIB.proxy.MethodInterceptor;
import org.springframework.CGLIB.proxy.MethodProxy;
/**
* CGLIB动态代理，实现MethodInterceptor接口
*
*/
public class CglibProxy implements MethodInterceptor {
    private Object target;// 需要代理的目标对象
    final MyAspect myAspect = new MyAspect();
    // 重写拦截方法
    @Override
    public Object intercept(Object obj, Method method, Object[] arr, MethodProxy proxy) throws Throwable {
        myAspect.myBefore();
        Object invoke = method.invoke(target, arr);// 方法执行，参数：target目标对象 arr参数数组
        myAspect.myAfter();
        return invoke;
    }
    // 定义获取代理对象方法
    public Object getCglibProxy(Object objectTarget) {
        // 为目标对象target赋值
        this.target = objectTarget;
        Enhancer enhancer = new Enhancer();
        // 设置父类,因为CGLIB是针对指定的类生成一个子类，所以需要指定父类
        enhancer.setSuperclass(objectTarget.getClass());
        enhancer.setCallback(this);// 设置回调
        Object result = enhancer.create();// 创建并返回代理对象
        return result;
    }
    public static void main(String[] args) {
        CglibProxy cglib= new CglibProxy();// 实例化CglibBProxy对象
        UserManager user = (UserManager) cglib.getCglibProxy(new UserManagerImpl());// 获取代理对象
        user.addUser("zhangsan", "123456"); // 执行新增方法
        user.delUser("zhangsan"); // 执行删除方法
    }
}
```
运行结果如下。
```
方法执行之前
正在执行添加用户方法
用户名称: zhangsan 密码: 123456
方法执行之后
方法执行之前
正在执行删除用户方法
用户名称: zhangsan
方法执行之后
```
## JDK代理和CGLIB代理的区别
JDK 动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用`InvokeHandler`来处理。而 CGLIB 动态代理是利用 ASM 开源包，加载代理对象类的`class`文件，通过修改其字节码生成子类来处理。

JDK 动态代理只能对实现了接口的类生成代理，而不能针对类。

CGLIB 是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法。因为是继承，所以该类或方法不能声明成`final`类型。

JDK动态代理特点：
* 代理对象必须实现一个或多个接口
* 以接口的形式接收代理实例，而不是代理类

CGLIB动态代理特点：
* 代理对象不能被`final`修饰
* 以类或接口形式接收代理实例

JDK与CGLIB动态代理的性能比较：
* 生成代理实例性能：JDK > CGLIB
* 代理实例运行性能：JDK > CGLIB

# 基于AspectJ XML开发
AspectJ 是一个基于 Java 语言的 AOP 框架，它扩展了 Java 语言，提供了强大的 AOP 功能。

基于 XML 的声明式是指通过 Spring 配置文件的方式来定义切面、切入点及通知，而所有的切面和通知都必须定义在`<aop:config>`元素中。

在使用`<aop:config>`元素之前，我们需要先导入 Spring aop 命名空间。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">
    ...
</beans>
```
## 定义切面\<aop:aspect\>
在 Spring 配置文件中，使用`<aop:aspect>`元素定义切面，该元素可以将定义好的`Bean`转换为切面`Bean`，所以使用`<aop:aspect>`之前需要先定义一个普通的`Spring Bean`。
```xml
<aop:config>
  <aop:aspect id="myAspect" ref="aBean">
    ...
  </aop:aspect>
</aop:config>
```
其中，`id`用来定义该切面的唯一表示名称，`ref`用于引用普通的`Spring Bean`。
## 定义切入点 \<aop:pointcut\>
`<aop:pointcut>`用来定义一个切入点，当`<aop:pointcut>`元素作为`<aop:config>`元素的子元素定义时，表示该切入点是全局切入点，它可被多个切面所共享；当`<aop:pointcut>`元素作为 `<aop:aspect>`元素的子元素时，表示该切入点只对当前切面有效。
```xml
<aop:config>
  <aop:pointcut id="myPointCut" expression="execution(* net.biancheng.service.*.*(..))"/>
</aop:config>
```
其中，`id`用于指定切入点的唯一标识名称，`execution`用于指定切入点关联的切入点表达式。

`execution`格式为：
```
execution(modifiers-pattern returning-type-pattern declaring-type-pattern name-pattern(param-pattern)throws-pattern)
```
其中：
* `returning-type-pattern、name-pattern、param-pattern`是必须的，其它参数为可选项。
* `modifiers-pattern`：指定修饰符，如`private、public`。
* `returning-type-pattern`：指定返回值类型，*表示可以为任何返回值。如果返回值为对象，则需指定全路径的类名。
* `declaring-type-pattern`：指定方法的包名。
* `name-pattern`：指定方法名，`*`代表所有，`set*`代表以`set`开头的所有方法。
* `param-pattern`：指定方法参数（声明的类型），`(..)`代表所有参数，`(*)`代表一个参数，`(*,String)`代表第一个参数可以为任何值，第二个为`String`类型的值。
* `throws-pattern`：指定抛出的异常类型。

例如：`execution(* net.biancheng.*.*(..))`表示匹配`net.biancheng`包中任意类的任意方法。
## 定义通知
AspectJ 支持 5 种类型的`advice`。
```xml
<aop:aspect id="myAspect" ref="aBean">
  <!-- 前置通知 -->
  <aop:before pointcut-ref="myPointCut" method="..."/>
  <!-- 后置通知 -->
  <aop:after-returning pointcut-ref="myPointCut" method="..."/>
  <!-- 环绕通知 -->
  <aop:around pointcut-ref="myPointCut" method="..."/>
  <!-- 异常通知 -->
  <aop:after-throwing pointcut-ref="myPointCut" method="..."/>
  <!-- 最终通知 -->
  <aop:after pointcut-ref="myPointCut" method="..."/>
  .... 
</aop:aspect>
```
## 示例
步骤如下：
* 创建`SpringDemo`项目，并在`src`目录下创建`net.biancheng`包。
* 导入 Spring 相关 JAR 包及`Aspectjrt.jar、Aspectjweaver.jar、Aspectj.jar`。
* 在`net.biancheng`包下创建`Logging、Man、Beans.xml`和`MainApp`。
* 运行`SpringDemo`项目。

```java
package net.biancheng;
public class Logging {
    /**
     * 前置通知
     */
    public void beforeAdvice() {
        System.out.println("前置通知");
    }
    /**
     * 后置通知
     */
    public void afterAdvice() {
        System.out.println("后置通知");
    }
    /**
     * 返回后通知
     */
    public void afterReturningAdvice(Object retVal) {
        System.out.println("返回值为：" + retVal.toString());
    }
    /**
     * 抛出异常通知
     */
    public void afterThrowingAdvice(IllegalArgumentException ex) {
        System.out.println("这里的异常为：" + ex.toString());
    }
}
```
```java
package net.biancheng;
public class Man {
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
    public void throwException() {
        System.out.println("抛出异常");
        throw new IllegalArgumentException();
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">
    <aop:config>
        <aop:aspect id="log" ref="logging">
            <aop:pointcut id="selectAll"
                expression="execution(* net.biancheng.*.*(..))" />
            <aop:before pointcut-ref="selectAll" method="beforeAdvice" />
            <aop:after pointcut-ref="selectAll" method="afterAdvice" />
            <aop:after-returning pointcut-ref="selectAll"
                returning="retVal" method="afterReturningAdvice" />
            <aop:after-throwing pointcut-ref="selectAll"
                throwing="ex" method="afterThrowingAdvice" />
        </aop:aspect>
    </aop:config>
    <bean id="man" class="net.biancheng.Man">
        <property name="name" value="bianchengbang" />
        <property name="age" value="12" />
    </bean>
    <bean id="logging" class="net.biancheng.Logging" />
</beans>
```
```java
package net.biancheng;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
        Man man = (Man) context.getBean("man");
        man.getName();
        man.getAge();
        man.throwException();
    }
}
```
运行结果如下。
```
前置通知
后置通知
返回值为：bianchengbang
前置通知
后置通知
返回值为：12
前置通知
抛出异常
后置通知
这里的异常为：java.lang.IllegalArgumentException
Exception in thread "main" java.lang.IllegalArgumentException
```
# 基于AspectJ注解开发
AspectJ 框架为 AOP 开发提供了一套注解。AspectJ 允许使用注解定义切面、切入点和增强处理，Spring 框架可以根据这些注解生成 AOP 代理。

| 名称            | 说明 |
| :--: | :--: |
| @Aspect         | 用于定义一个切面。 |
| @Pointcut       | 用于定义一个切入点。 |
| @Before         | 用于定义前置通知，相当于 BeforeAdvice。 |
| @AfterReturning | 用于定义后置通知，相当于 AfterReturningAdvice。 |
| @Around         | 用于定义环绕通知，相当于MethodInterceptor。 |
| @AfterThrowing  | 用于定义抛出通知，相当于ThrowAdvice。 |
| @After          | 用于定义最终final通知，不管是否异常，该通知都会执行。 |
| @DeclareParents | 用于定义引介通知，相当于IntroductionInterceptor（不要求掌握）。 |

启用`@AspectJ`注解有以下两种方法：
1. 使用`@Configuration`和`@EnableAspectJAutoProxy`注解
```
@Configuration 
@EnableAspectJAutoProxy
public class Appconfig {}
```
2. 基于XML配置
在 XML 文件中添加以下内容启用`@AspectJ`。
```
<aop:aspectj-autoproxy>
```

## 定义切面@Aspect
`AspectJ`类和其它普通的`Bean`一样，可以有方法和字段，不同的是`AspectJ`类需要使用`@Aspect`注解。
```java
package net.biancheng;
import org.aspectj.lang.annotation.Aspect;
@Aspect
public class AspectModule {}
```
`AspectJ`类也可以像其它`Bean`一样在 XML 中配置。
```xml
<bean id = "myAspect" class = "net.biancheng.AspectModule">
   ...
</bean>
```
## 定义切入点@Pointcut
`@Pointcut`注解用来定义一个切入点。
```java
// 要求：方法必须是private，返回值类型为void，名称自定义，没有参数
@Pointcut("execution(*net.biancheng..*.*(..))")
private void myPointCut() {}
```
相当于以下代码
```
<aop:pointcut expression="execution(*net.biancheng..*.*(..))"  id="myPointCut"/>
```
## 定义通知advice
`@AspectJ`支持 5 种类型的`advice`，以下为使用`@Before`的示例。
```java
@Before("myPointCut()")
public void beforeAdvice(){
    ...
}
```
## 示例
步骤如下：
* 创建`SpringDemo`项目，并在`src`目录下创建`net.biancheng`包。
* 导入 Spring 相关 JAR 包及`Aspectjrt.jar、Aspectjweaver.jar、Aspectj.jar`。
* 在`net.biancheng`包下创建`Logging、Man、Beans.xml`和`MainApp`。
* 运行`SpringDemo`项目。

```java
package net.biancheng;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
@Aspect
public class Logging {
    /**
     * 定义切入点
     */
    @Pointcut("execution(* net.biancheng.*.*(..))")
    private void selectAll() {
    }
    /**
     * 前置通知
     */
    @Before("selectAll()")
    public void beforeAdvice() {
        System.out.println("前置通知");
    }
    /**
     * 后置通知
     */
    @After("selectAll()")
    public void afterAdvice() {
        System.out.println("后置通知");
    }
    /**
     * 返回后通知
     */
    @AfterReturning(pointcut = "selectAll()", returning = "retVal")
    public void afterReturningAdvice(Object retVal) {
        System.out.println("返回值为：" + retVal.toString());
    }
    /**
     * 抛出异常通知
     */
    @AfterThrowing(pointcut = "selectAll()", throwing = "ex")
    public void afterThrowingAdvice(IllegalArgumentException ex) {
        System.out.println("这里的异常为：" + ex.toString());
    }
}
```
```java
package net.biancheng;
public class Man {
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
    public void throwException() {
        System.out.println("抛出异常");
        throw new IllegalArgumentException();
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop-3.0.xsd ">
    <aop:aspectj-autoproxy />
    <bean id="man" class="net.biancheng.Man">
        <property name="name" value="bianchengbang" />
        <property name="age" value="12" />
    </bean>
    <bean id="logging" class="net.biancheng.Logging" />
</beans>
```
```java
package net.biancheng;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class MainApp {
  public static void main(String[] args) {
    ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
    Man man = (Man) context.getBean("man");
    man.getName();
    man.getAge();
    man.throwException();
  }
}
```
运行结果如下。
```
前置通知
后置通知
返回值为：bianchengbang
前置通知
后置通知
返回值为：12
前置通知
抛出异常
后置通知
这里的异常为：java.lang.IllegalArgumentException
```