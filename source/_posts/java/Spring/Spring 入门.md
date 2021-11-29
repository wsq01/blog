---
title: Spring 入门
date: 2021-05-15 16:24:15
tags: [Spring]
categories: [Spring]
---

Spring 包括许多框架，例如 Spring framework、SpringMVC、SpringBoot、Spring Cloud、Spring Data、Spring Security 等。Spring framework 就是我们平时说的 Spring 框架。

Spring 是分层的 Java SE/EE 一站式轻量级开源框架，以 IoC（`Inverse of Control`，控制反转）和 AOP（`Aspect Oriented Programming`，面向切面编程）为内核。

IoC 指的是将对象的创建权交给 Spring 去创建。使用 Spring 之前，对象的创建都是由我们使用`new`创建，而使用 Spring 之后，对象的创建都交给了 Spring 框架。AOP 用来封装多个类的公共行为，将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，减少系统的重复代码，降低模块间的耦合度。另外，AOP 还解决一些系统层面上的问题，比如日志、事务、权限等。


在实际开发中，通常服务器端采用三层体系架构，分别为表现层（`web`）、业务逻辑层（`service`）、持久层（`dao`）。

Spring 对每一层都提供了技术支持，在表现层提供了与 Spring MVC、Struts2 框架的整合，在业务逻辑层可以管理事务和记录日志等，在持久层可以整合 MyBatis、Hibernate 和 JdbcTemplate 等技术。

Spring 框架的主要优点：
1. 方便解耦，简化开发。Spring 就是一个大工厂，可以将所有对象的创建和依赖关系的维护交给 Spring 管理。
2. 方便集成各种优秀框架，Spring 其内部提供了对各种优秀框架的直接支持。
3. 降低 Java EE API 的使用难度，Spring 对 Java EE 开发中非常难用的一些 API都提供了封装，使这些 API 应用的难度大大降低。
4. 方便程序的测试。Spring 支持 JUnit4，可以通过注解方便地测试 Spring 程序。
5. AOP 编程的支持。Spring 提供面向切面编程，可以方便地实现对程序进行权限拦截和运行监控等功能。
6. 声明式事务的支持。只需要通过配置就可以完成对事务的管理，而无须手动编程。

Spring 框架采用分层架构，根据不同的功能被划分成了多个模块，这些模块大体可分为`Data Access/Integration、Web、AOP、Aspects、Messaging、Instrumentation、Core Container`和`Test`。

{% asset_img 1.png %}

1. Data Access/Integration（数据访问／集成）
数据访问/集成层包括 JDBC、ORM、OXM、JMS 和 Transactions 模块。
 * JDBC 模块(`spring-jdbc`)：提供了一个 JDBC 的抽象层，大幅度减少了在开发过程中对数据库操作的编码。
 * ORM 模块(`spring-orm`)：对流行的对象关系映射 API，包括 JPA、JDO、Hibernate 和 iBatis 提供了的集成层。
 * OXM 模块(`spring-oxm`)：提供了一个支持对象/XML 映射的抽象层实现，如 JAXB、Castor、XMLBeans、JiBX 和 XStream。
 * JMS 模块(`spring-jms`)：指 Java 消息服务，包含的功能为生产和消费的信息。
 * Transactions 事务模块(`spring-tx`)：支持编程和声明式事务管理实现特殊接口类，以及所有的 POJO。
2. Web 模块
Spring 的 Web 层包括 Web、Servlet、Struts 和 Portlet 组件。
 * Web 模块(`spring-web`)：提供了基本的 Web 开发集成特性，例如多文件上传功能、使用的 Servlet 监听器的 IoC 容器初始化以及 Web 应用上下文。
 * Servlet 模块(``spring-webmvc``)：包括 Spring 模型—视图—控制器（MVC）实现 Web 应用程序。
 * Struts 模块：包含支持类内的 Spring 应用程序，集成了经典的 Struts Web 层。
 * Portlet 模块(``spring-webmvc-portlet``)：提供了在 Portlet 环境中使用 MVC 实现，类似 Web-Servlet 模块的功能。
3. Core Container（核心容器）
Spring 的核心容器是其他模块建立的基础，由`Beans`模块、`Core`核心模块、`Context`上下文模块和`Expression Language`表达式语言模块组成。
 * Beans 模块(`spring-beans`)：提供了`BeanFactory`，是工厂模式的经典实现，Spring 将管理对象称为`Bean`。
 * Core 核心模块(`spring-core`)：提供了 Spring 框架的基本组成部分，包括 IoC 和 DI 功能。
 * Context 上下文模块(`spring-context`)：建立在核心和`Beans`模块的基础之上，它是访问定义和配置任何对象的媒介。`ApplicationContext`接口是上下文模块的焦点。
 * Expression Language 模块(`spring-expression`)：是运行时查询和操作对象图的强大的表达式语言。
4. 其他模块
Spring的其他模块还有 AOP、Aspects、Instrumentation 以及 Test 模块。
 * AOP 模块(`spring-aop`)：提供了面向切面编程实现，允许定义方法拦截器和切入点，将代码按照功能进行分离，以降低耦合性。
 * Aspects 模块(`spring-aspects`)：提供与 AspectJ 的集成，是一个功能强大且成熟的面向切面编程（AOP）框架。
 * Instrumentation 模块(`spring-instrument`)：提供了类工具的支持和类加载器的实现，可以在特定的应用服务器中使用。
 * Test 模块(`spring-test`)：支持 Spring 组件，使用 JUnit 或 TestNG 框架的测试。

