---
title: SpringMVC 入门
date: 2021-07-08 16:24:15
tags: [SpringMVC]
categories: [SpringMVC]
---


# MVC设计模式
MVC 设计模式一般指 MVC 框架，M（`Model`）指数据模型层，V（`View`）指视图层，C（`Controller`）指控制层。使用 MVC 的目的是将 M 和 V 的实现代码分离，使同一个程序可以有不同的表现形式。

在 Web 项目的开发中，能够及时、正确地响应用户的请求是非常重要的。用户在网页上单击一个 URL 路径，这对 Web 服务器来说，相当于用户发送了一个请求。而获取请求后如何解析用户的输入，并执行相关处理逻辑，最终跳转至正确的页面显示反馈结果，这些工作往往是控制层（`Controller`）来完成的。

在请求的过程中，用户的信息被封装在`User`实体类中，该实体类在 Web 项目中属于数据模型层（`Model`）。

在请求显示阶段，跳转的结果网页就属于视图层（`View`）。

像这样，控制层负责前台与后台的交互，数据模型层封装用户的输入/输出数据，视图层选择恰当的视图来显示最终的执行结果，这样的层次分明的软件开发和处理流程被称为 MVC 模式。

总结如下：
* 视图层（`View`）：负责格式化数据并把它们呈现给用户，包括数据展示、用户交互、数据验证、界面设计等功能。
* 控制层（`Controller`）：负责接收并转发请求，对请求进行处理后，指定视图并将响应结果发送给客户端。
* 数据模型层（`Model`）：模型对象拥有最多的处理任务，是应用程序的主体部分，它负责数据逻辑（业务规则）的处理和实现数据操作（即在数据库中存取数据）。

SUN 公司推出 JSP 技术的同时，也推出了两种 Web 应用程序的开发模式。即 JSP+JavaBean 和 Servlet+JSP+JavaBean。
## JSP+JavaBean
JSP+JavaBean 中 JSP 用于处理用户请求，JavaBean 用于封装和处理数据。该模式只有视图和模型，一般把控制器的功能交给视图来实现，适合业务流程比较简单的 Web 程序。

{% asset_img 1.png JSP+JavaBean %}

通过上图可以发现 JSP 从 HTTP Request（请求）中获得所需的数据，并进行业务逻辑的处理，然后将结果通过 HTTP Response（响应）返回给浏览器。从中可见，JSP+JavaBean 模式在一定程度上实现了 MVC，即 JSP 将控制层和视图合二为一，JavaBean 为模型层。

JSP+JavaBean 模式中 JSP 身兼数职，既要负责视图层的数据显示，又要负责业务流程的控制，结构较为混乱，并且也不是我们所希望的松耦合架构模式，所以当业务流程复杂的时候并不推荐使用。
## Servlet+JSP+JavaBean
Servlet+JSP+JavaBean 中 Servlet 用于处理用户请求，JSP 用于数据显示，JavaBean 用于数据封装，适合复杂的 Web 程序。

{% asset_img 2.png Servlet+JSP+JavaBean %}

相比 JSP+JavaBean 模式来说，Servlet+JSP+JavaBean 模式将控制层单独划分出来负责业务流程的控制，接收请求，创建所需的 JavaBean 实例，并将处理后的数据返回视图层（JSP）进行界面数据展示。

Servlet+JSP+JavaBean 模式的结构清晰，是一个松耦合架构模式，一般情况下，建议使用该模式。
## MVC优缺点
优点：
* 多视图共享一个模型，大大提高了代码的可重用性
* MVC 三个模块相互独立，松耦合架构
* 控制器提高了应用程序的灵活性和可配置性
* 有利于软件工程化管理

总之，我们通过 MVC 设计模式最终可以打造出一个松耦合+高可重用性+高可适用性的完美架构。

缺点：
* 原理复杂
* 增加了系统结构和实现的复杂性
* 视图对模型数据的低效率访问

MVC 并不适合小型甚至中型规模的项目，花费大量时间将 MVC 应用到规模并不是很大的应用程序，通常得不偿失，所以对于 MVC 设计模式的使用要根据具体的应用场景来决定。
# Spring MVC是什么
Spring MVC 是 Spring 提供的一个基于 MVC 设计模式的轻量级 Web 开发框架，本质上相当于 Servlet。Spring MVC 是结构最清晰的 Servlet+JSP+JavaBean 的实现，是一个典型的教科书式的 MVC 构架。

在 Spring MVC 框架中，Controller 替换 Servlet 来担负控制器的职责，用于接收请求，调用相应的 Model 进行处理，处理器完成业务处理后返回处理结果。Controller 调用相应的 View 并对处理结果进行视图渲染，最终客户端得到响应信息。

Spring MVC 框架采用松耦合可插拔的组件结构，具有高度可配置性，比起其它 MVC 框架更具有扩展性和灵活性。

此外，Spring MVC 的注解驱动和对 REST 风格的支持，也是它最具特色的功能。并且由于 Spring MVC 本身就是 Spring 框架的一部分，所以可以说与 Spring 框架是无缝集成，性能方面具有先天的优越性，对于开发者来说，开发效率也高于其它的 Web 框架。

Spring MVC优点：
* 清晰地角色划分，Spring MVC 在 Model、View 和 Controller 方面提供了一个非常清晰的角色划分，这 3 个方面真正是各司其职，各负其责。
* 灵活的配置功能，可以把类当作 Bean 通过 XML 进行配置。
* 提供了大量的控制器接口和实现类，开发者可以使用 Spring 提供的控制器实现类，也可以自己实现控制器接口。
* 真正做到与 View 层的实现无关。它不会强制开发者使用 JSP，可以根据项目需求使用 Velocity、FreeMarker 等技术。
* 国际化支持
* 面向接口编程
* 与 Spring 框架无缝集成

# 第一个Spring MVC程序
搭建步骤：
* 创建 Web 应用并引入 JAR 包
* Spring MVC 配置：在`web.xml`中配置 Servlet，创建 Spring MVC 的配置文件
* 创建`Controller`（处理请求的控制器）
* 创建`View`
* 部署运行

## 1. 创建Web应用并引入JAR包
创建 Web 应用`springmvcDemo`，在`springmvcDemo`的`lib`目录中添加 Spring MVC 所依赖的 JAR 包。

Maven 项目在`pom.xml`文件中添加以下内容：
```xml
<!--测试-->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.11</version>
    <scope>test</scope>
</dependency>
<!--日志-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.21</version>
</dependency>
<!--J2EE-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <version>2.2</version>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
<!--mysql驱动包-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.35</version>
</dependency>
<!--springframework-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.github.stefanbirkner</groupId>
    <artifactId>system-rules</artifactId>
    <version>1.16.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.9</version>
</dependency>
<!--其他需要的包-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.4</version>
</dependency>
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```
## 2. Spring MVC配置
Spring MVC 是基于 Servlet 的，`DispatcherServlet`是整个 Spring MVC 框架的核心，主要负责截获请求并将其分派给相应的处理器处理。所以配置 Spring MVC，首先要定义`DispatcherServlet`。跟所有 Servlet 一样，用户必须在`web.xml`中进行配置。
### 定义DispatcherServlet
在开发 Spring MVC 应用时需要在`web.xml`中部署`DispatcherServlet`：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
    <display-name>springMVC</display-name>
    <!-- 部署 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 表示容器再启动时立即加载servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- 处理所有URL -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```
Spring MVC 初始化时将在应用程序的`WEB-INF`目录下查找配置文件，该配置文件的命名规则是`servletName-servlet.xml`，例如`springmvc-servlet.xml`。

也可以将 Spring MVC 的配置文件存放在应用程序目录中的任何地方，但需要使用 servlet 的`init-param`元素加载配置文件，通过`contextConfigLocation`参数来指定 Spring MVC 配置文件的位置。
```xml
<!-- 部署 DispatcherServlet -->
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet </servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc-servlet.xml</param-value>
    </init-param>
    <!-- 表示容器再启动时立即加载servlet -->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```
此处使用 Spring 资源路径的方式进行指定，即`classpath:springmvc-servlet.xml`。

上述代码配置了一个名为`springmvc`的 Servlet。该 Servlet 是`DispatcherServlet`类型，它就是 Spring MVC 的入口，并通过`<load-on-startup>1</load-on-startup>`配置标记容器在启动时就加载此`DispatcherServlet`，即自动启动。然后通过`servlet-mapping`映射到“/”，即`DispatcherServlet`需要截获并处理该项目的所有 URL 请求。
### 创建Spring MVC配置文件
在`WEB-INF`目录下创建`springmvc-servlet.xml`文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- LoginController控制器类，映射到"/login" -->
    <bean name="/login" class="net.biancheng.controller.LoginController"/>
    <!-- LoginController控制器类，映射到"/register" -->
    <bean name="/register" class="net.biancheng.controller.RegisterController"/>
</beans>
```
### 创建Controller
在`src`目录下创建`net.biancheng.controller`包，并在该包中创建`RegisterController`和`LoginController`两个传统风格的控制器类（实现`Controller`接口），分别处理首页中“注册”和“登录”超链接的请求。

`Controller`是控制器接口，接口中只有一个方法`handleRequest`，用于处理请求和返回`ModelAndView`。

`RegisterController`的具体代码如下。
```java
package controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
public class LoginController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest arg0,
            HttpServletResponse arg1) throws Exception {
        return new ModelAndView("/WEB-INF/jsp/register.jsp");
    }
}
```
`LoginController`的具体代码如下。
```java
package controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;
public class RegisterController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest arg0,
            HttpServletResponse arg1) throws Exception {
        return new ModelAndView("/WEB-INF/jsp/login.jsp");
    }
}
```
### 创建View
`index.jsp`代码如下。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    未注册的用户，请
    <a href="${pageContext.request.contextPath }/register"> 注册</a>！
    <br /> 已注册的用户，去
    <a href="${pageContext.request.contextPath }/login"> 登录</a>！
</body>
</html>
```
在`WEB-INF`下创建`jsp`文件夹，将`login.jsp`和`register.jsp`放到`jsp`文件夹下。`login.jsp`代码如下。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    登录页面！
</body>
</html>
```
`register.jsp`代码如下。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
<body>
    注册页面！
</body>
</html>
</head>
```
### 部署运行
将`springmvcDemo`项目部署到 Tomcat 服务器。

{% asset_img 3.png %}
{% asset_img 4.png %}
{% asset_img 5.png %}

在上图所示的页面中，当用户单击“注册”超链接时，根据`springmvc-servlet.xml`文件中的映射将请求转发给`RegisterController`控制器处理，处理后跳转到`/WEB-INF/jsp`下的`register.jsp`视图。同理，当单击“登录”超链接时，控制器处理后转到`/WEB-INF/jsp`下的`login.jsp`视图。
# 视图解析器（ViewResolver）
视图解析器（`ViewResolver`）是 Spring MVC 的重要组成部分，负责将逻辑视图名解析为具体的视图对象。

Spring MVC 提供了很多视图解析类，其中每一项都对应 Java Web 应用中特定的某些视图技术。
## URLBasedViewResolver
`UrlBasedViewResolver`是对`ViewResolver`的一种简单实现，主要提供了一种拼接`URL`的方式来解析视图。

`UrlBasedViewResolver`通过`prefix`属性指定前缀，`suffix`属性指定后缀。当`ModelAndView`对象返回具体的`View`名称时，它会将前缀`prefix`和后缀`suffix`与具体的视图名称拼接，得到一个视图资源文件的具体加载路径，从而加载真正的视图文件并反馈给用户。

使用`UrlBasedViewResolver`除了要配置前缀和后缀属性之外，还需要配置`viewClass`，表示解析成哪种视图。
```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">            
    <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceViewResolver"/> <!--不能省略-->
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>  
</bean>
 ```
上述视图解析器配置了前缀和后缀两个属性，这样缩短了`view`路径。

上述`viewClass`值为`InternalResourceViewResolver`，它用来展示 JSP 页面。如果需要使用`jstl`标签展示数据，将`viewClass`属性值指定为`JstlView`即可。

另外，存放在`/WEB-INF/`目录下的内容不能直接通过`request`请求得到，所以为了安全性考虑，通常把`jsp`文件放在`WEB-INF`目录下。
## InternalResourceViewResolver
`InternalResourceViewResolver`为“内部资源视图解析器”。它是`URLBasedViewResolver`的子类，拥有`URLBasedViewResolver`的一切特性。

`InternalResourceViewResolver`能自动将返回的视图名称解析为`InternalResourceView`类型的对象。`InternalResourceView`会把`Controller`处理器方法返回的模型属性都存放到对应的`request`属性中，然后通过`RequestDispatcher`在服务器端把请求`forword`重定向到目标 URL。也就是说，使用`InternalResourceViewResolver`视图解析时，无需再单独指定`viewClass`属性。
```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
  <property name="viewClass" value="org.springframework.web.servlet.view.InternalResourceViewResolver"/> <!--可以省略-->
  <!--前缀-->
  <property name="prefix" value="/WEB-INF/jsp/"/>
  <!--后缀-->
  <property name="suffix" value=".jsp"/>
</bean>
```
## FreeMarkerViewResolver
`FreeMarkerViewResolver`是`UrlBasedViewResolver`的子类，可以通过`prefix`属性指定前缀，通过`suffix`属性指定后缀。

`FreeMarkerViewResolver`最终会解析逻辑视图配置，返回`freemarker`模板。不需要指定`viewClass`属性。

`FreeMarkerViewResolver`配置如下。
```xml
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerViewResolver">
  <property name="prefix" value="fm_"/>
  <property name="suffix" value=".ftl"/>
</bean>
```
下面指定 FreeMarkerView 类型最终生成的实体视图（模板文件）的路径以及其他配置。需要给 FreeMarkerViewResolver 设置一个 FreeMarkerConfig 的 bean 对象来定义 FreeMarker 的配置信息，代码如下。
```xml
<bean class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
  <property name="templateLoaderPath" value="/WEB-INF/ftl" />
</bean>
```
定义了`templateLoaderPath`属性后，Spring 可以通过该属性找到`FreeMarker`模板文件的具体位置。当有模板位于不同的路径时，可以配置 `templateLoaderPath`属性，来指定多个资源路径。

然后定义一个`Controller`，让其返回`ModelAndView`，同时定义一些返回参数和视图信息。
```java
@Controller
@RequestMapping("viewtest")
public class ViewController {
  @RequestMapping("freemarker")
  public ModelAndView freemarker() {
    ModelAndView mv = new ModelAndView();
    mv.addObject("username", "BianChengBang");
    mv.setViewName("freemarker");
    return mv;
  }
}
```
当`FreeMarkerViewResolver`解析逻辑视图信息时，会生成一个 URL 为“前缀+视图名+后缀”（这里即`fm_freemarker.ftl`）的`FreeMarkerView`对象，然后通过`FreeMarkerConfigurer`的配置找到`templateLoaderPath`对应文本文件的路径，在该路径下找到该文本文件，从而`FreeMarkerView`就可以利用该模板文件进行视图的渲染，并将`model`数据封装到即将要显示的页面上，最终展示给用户。

在`/WEB-INF/ftl`文件夹下创建`fm_freemarker.ftl`，代码如下。
```html
<html>
<head>
<title>FreeMarker</title>
</head>
<body>
    <b>Welcome!</b>
    <i>${username }</i>
</body>
</html>
```
# Spring MVC执行流程
Spring MVC 框架是高度可配置的，包含多种视图技术，例如 JSP、FreeMarker、Tiles。Spring MVC 框架并不关心使用的视图技术，也不会强迫开发者只使用 JSP。

Spring MVC 执行流程如图

{% asset_img 6.png %}

SpringMVC 的执行流程如下。
* 用户点击某个请求路径，发起一个 HTTP request 请求，该请求会被提交到`DispatcherServlet`（前端控制器）；
* 由`DispatcherServlet`请求一个或多个`HandlerMapping`（处理器映射器），并返回一个执行链（`HandlerExecutionChain`）。
* `DispatcherServlet`将执行链返回的`Handler`信息发送给`HandlerAdapter`（处理器适配器）；
* `HandlerAdapter`根据`Handler`信息找到并执行相应的`Handler`（常称为`Controller`）；
* `Handler`执行完毕后会返回给`HandlerAdapter`一个`ModelAndView`对象（Spring MVC的底层对象，包括`Model`数据模型和`View`视图信息）；
* `HandlerAdapter`接收到`ModelAndView`对象后，将其返回给`DispatcherServlet`；
* `DispatcherServlet`接收到`ModelAndView`对象后，会请求`ViewResolver`（视图解析器）对视图进行解析；
* `ViewResolver`根据`View`信息匹配到相应的视图结果，并返回给`DispatcherServlet`；
* `DispatcherServlet`接收到具体的`View`视图后，进行视图渲染，将`Model`中的模型数据填充到`View`视图中的`request`域，生成最终的`View`（视图）；
* 视图负责将结果显示到浏览器（客户端）。

## Spring MVC接口
Spring MVC 涉及到的组件有`DispatcherServlet`（前端控制器）、`HandlerMapping`（处理器映射器）、`HandlerAdapter`（处理器适配器）、`Handler`（处理器）、`ViewResolver`（视图解析器）和`View`（视图）。下面对各个组件的功能说明如下。
1. `DispatcherServlet`
`DispatcherServlet`是前端控制器，从图可以看出，Spring MVC 的所有请求都要经过`DispatcherServlet`来统一分发。`DispatcherServlet`相当于一个转发器或中央处理器，控制整个流程的执行，对各个组件进行统一调度，以降低组件之间的耦合性，有利于组件之间的拓展。
2. `HandlerMapping`
`HandlerMapping`是处理器映射器，其作用是根据请求的 URL 路径，通过注解或者 XML 配置，寻找匹配的处理器（`Handler`）信息。
3. `HandlerAdapter`
`HandlerAdapter`是处理器适配器，其作用是根据映射器找到的处理器（`Handler`）信息，按照特定规则执行相关的处理器（`Handler`）。
4. `Handler`
`Handler`是处理器，和 Java Servlet 扮演的角色一致。其作用是执行相关的请求处理逻辑，并返回相应的数据和视图信息，将其封装至`ModelAndView`对象中。
5. `View Resolver`
`View Resolver`是视图解析器，其作用是进行解析操作，通过`ModelAndView`对象中的`View`信息将逻辑视图名解析成真正的视图`View`（如通过一个 JSP 路径返回一个真正的 JSP 页面）。
6. `View`
`View`是视图，其本身是一个接口，实现类支持不同的`View`类型（JSP、FreeMarker、Excel 等）。

以上组件中，需要开发人员进行开发的是处理器（`Handler`，常称`Controller`）和视图（`View`）。通俗的说，要开发处理该请求的具体代码逻辑，以及最终展示给用户的界面。

