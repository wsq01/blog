---
title: Spring AOP
date: 2021-04-09 16:24:15
tags: [Spring]
categories: [Spring]
---


面向切面编程（AOP）和面向对象编程（OOP）类似，也是一种编程模式。Spring AOP 是基于 AOP 编程模式的一个框架，它的使用有效减少了系统间的重复代码，达到了模块间的松耦合目的。

AOP 的全称是“Aspect Oriented Programming”，即面向切面编程，它将业务逻辑的各个部分进行隔离，使开发人员在编写业务逻辑时可以专心于核心业务，从而提高了开发效率。

AOP 采取横向抽取机制，取代了传统纵向继承体系的重复性代码，其应用主要体现在事务处理、日志管理、权限控制、异常处理等方面。

目前最流行的 AOP 框架有两个，分别为 Spring AOP 和 AspectJ。

Spring AOP 使用纯 Java 实现，不需要专门的编译过程和类加载器，在运行期间通过代理方式向目标类植入增强的代码。

AspectJ 是一个基于 Java 语言的 AOP 框架，从 Spring 2.0 开始，Spring AOP 引入了对 AspectJ 的支持。AspectJ 扩展了 Java 语言，提供了一个专门的编译器，在编译时提供横向代码的植入。
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

Advice 直译为通知，也有的资料翻译为“增强处理”，共有 5 种类型，如下表所示。

| 通知 | 说明 |
| :--: | :--: |
| before（前置通知） | 通知方法在目标方法调用之前执行 |
| after（后置通知） | 通知方法在目标方法返回或异常后调用 |
| after-returning（返回后通知） | 通知方法会在目标方法返回后调用 |
| after-throwing（抛出异常通知） | 通知方法会在目标方法抛出异常后调用 |
| around（环绕通知） | 通知方法会将目标方法封装起来 |

在 Spring 框架中使用 AOP 主要有以下优势。
提供声明式企业服务，特别是作为 EJB 声明式服务的替代品。最重要的是，这种服务是声明式事务管理。
允许用户实现自定义切面。在某些不适合用 OOP 编程的场景中，采用 AOP 来补充。
可以对业务逻辑的各个部分进行隔离，从而使业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时也提高了开发效率。
# JDK动态代理
JDK 动态代理是通过 JDK 中的`java.lang.reflect.Proxy`类实现的。下面通过具体的案例演示 JDK 动态代理的使用。
## 1. 创建项目
创建一个名称为`springDemo03`的 Web 项目，将 Spring 支持和依赖的 JAR 包复制到 Web 项目的 WEB-INF/lib 目录中，并发布到类路径下。
## 2. 创建接口 CustomerDao
在项目的`src`目录下创建一个名为`com.mengma.dao`的包，在该包下创建一个`CustomerDao`接口。
```java
package com.mengma.dao;
public interface CustomerDao {
  public void add(); // 添加
  public void update(); // 修改
  public void delete(); // 删除
  public void find(); // 查询
}
```
## 3. 创建实现类 CustomerDaoImpl
在`com.mengma.dao`包下创建`CustomerDao`接口的实现类`CustomerDaoImpl`，并实现该接口中的所有方法。
```java
package com.mengma.dao;
public class CustomerDaoImpl implements CustomerDao {
  @Override
  public void add() {
    System.out.println("添加客户...");
  }
  @Override
  public void update() {
    System.out.println("修改客户...");
  }
  @Override
  public void delete() {
    System.out.println("删除客户...");
  }
  @Override
  public void find() {
    System.out.println("修改客户...");
  }
}
```
## 4. 创建切面类 MyAspect
在`src`目录下，创建一个名为`com.mengma.jdk`的包，在该包下创建一个切面类`MyAspect`。
```java
package com.mengma.jdk;
public class MyAspect {
  public void myBefore() {
    System.out.println("方法执行之前");
  }
  public void myAfter() {
    System.out.println("方法执行之后");
  }
}
```
上述代码中，在切面中定义了两个增强的方法，分别为 myBefore() 方法和 myAfter() 方法，用于对目标类（CustomerDaoImpl）进行增强。
## 5. 创建代理类 MyBeanFactory
在 com.mengma.jdk 包下创建一个名为 MyBeanFactory 的类，在该类中使用 java.lang.reflect.Proxy 实现 JDK 动态代理，如下所示。
```java
package com.mengma.jdk;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import com.mengma.dao.CustomerDao;
import com.mengma.dao.CustomerDaoImpl;
public class MyBeanFactory {
  public static CustomerDao getBean() {
    // 准备目标类
    final CustomerDao customerDao = new CustomerDaoImpl();
    // 创建切面类实例
    final MyAspect myAspect = new MyAspect();
    // 使用代理类，进行增强
    return (CustomerDao) Proxy.newProxyInstance(
            MyBeanFactory.class.getClassLoader(),
            new Class[] { CustomerDao.class }, new InvocationHandler() {
              public Object invoke(Object proxy, Method method,
                      Object[] args) throws Throwable {
                  myAspect.myBefore(); // 前增强
                  Object obj = method.invoke(customerDao, args);
                  myAspect.myAfter(); // 后增强
                  return obj;
              }
            });
  }
}
```
上述代码中，定义了一个静态的 getBean() 方法，这里模拟 Spring 框架的 IoC 思想，通过调用 getBean() 方法创建实例，第 14 行代码创建了 customerDao 实例。

第 16 行代码创建的切面类实例用于调用切面类中相应的方法；第 18～26 行就是使用代理类对创建的实例 customerDao 中的方法进行增强的代码，其中 Proxy 的 newProxyInstance() 方法的第一个参数是当前类的类加载器，第二参数是所创建实例的实现类的接口，第三个参数就是需要增强的方法。

在目标类方法执行的前后，分别执行切面类中的 myBefore() 方法和 myAfter() 方法。
## 6. 创建测试类 JDKProxyTest
在`com.mengma.jdk`包下创建一个名为 JDKProxyTest 的测试类。
```java
package com.mengma.jdk;
import org.junit.Test;
import com.mengma.dao.CustomerDao;
public class JDKProxyTest {
  @Test
  public void test() {
    // 从工厂获得指定的内容（相当于spring获得，但此内容时代理对象）
    CustomerDao customerDao = MyBeanFactory.getBean();
    // 执行方法
    customerDao.add();
    customerDao.update();
    customerDao.delete();
    customerDao.find();
  }
}
```
上述代码中，在调用 getBean() 方法时，获取的是 CustomerDao 类的代理对象，然后调用了该对象中的方法。
## 7. 运行项目并查看结果
使用 JUnit 测试运行 test() 方法，运行成功后，控制台的输出结果如图 1 所示。

从图 1 的输出结果中可以看出，在调用目标类的方法前后，成功调用了增强的代码，由此说明，JDK 动态代理已经实现。