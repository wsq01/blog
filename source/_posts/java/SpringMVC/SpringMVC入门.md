


# MVC设计模式
MVC 是 Model、View 和 Controller 的缩写，分别代表 Web 应用程序中的 3 种职责。
模型：用于存储数据以及处理用户请求的业务逻辑。
视图：向控制器提交数据，显示模型中的数据。
控制器：根据视图提出的请求判断将请求和数据交给哪个模型处理，将处理后的有关结果交给哪个视图更新显示。

基于 Servlet 的 MVC 模式的具体实现如下。
模型：一个或多个 JavaBean 对象，用于存储数据（实体模型，由 JavaBean 类创建）和处理业务逻辑（业务模型，由一般的 Java 类创建）。
视图：一个或多个 JSP 页面，向控制器提交数据和为模型提供数据显示，JSP 页面主要使用 HTML 标记和 JavaBean 标记来显示数据。
控制器：一个或多个 Servlet 对象，根据视图提交的请求进行控制，即将请求转发给处理业务逻辑的 JavaBean，并将处理结果存放到实体模型 JavaBean 中，输出给视图显示。

基于 Servlet 的 MVC 模式的流程如图。

{% asset_img 1.gif JSP 中的 MVC 模式 %}

# Spring MVC 工作流程
Spring MVC 框架主要由 DispatcherServlet、处理器映射、控制器、视图解析器、视图组成，其工作原理如图。

{% asset_img 2.png Spring MVC 工作原理图 %}

从图中可总结出 Spring MVC 的工作流程如下：
客户端请求提交到 DispatcherServlet。
由 DispatcherServlet 控制器寻找一个或多个 HandlerMapping，找到处理请求的 Controller。
DispatcherServlet 将请求提交到 Controller。
Controller 调用业务逻辑处理后返回 ModelAndView。
DispatcherServlet 寻找一个或多个 ViewResolver 视图解析器，找到 ModelAndView 指定的视图。
视图负责将结果显示到客户端。

# Spring MVC接口
在上图中包含 4 个 Spring MVC 接口，即 DispatcherServlet、HandlerMapping、Controller 和 ViewResolver。

Spring MVC 所有的请求都经过`DispatcherServlet`来统一分发，在`DispatcherServlet`将请求分发给`Controller`之前需要借助 Spring MVC 提供的`HandlerMapping`定位到具体的`Controller`。

`HandlerMapping`接口负责完成客户请求到`Controller`映射。

`Controller`接口将处理用户请求，这和 Java Servlet 扮演的角色是一致的。一旦`Controller`处理完用户请求，将返回`ModelAndView`对象给`DispatcherServlet`前端控制器，`ModelAndView`中包含了模型（`Model`）和视图（`View`）。

从宏观角度考虑，`DispatcherServlet`是整个 Web 应用的控制器；从微观考虑，`Controller`是单个 Http 请求处理过程中的控制器，而`ModelAndView`是 Http 请求过程中返回的模型（`Model`）和视图（`View`）。

`ViewResolver`接口（视图解析器）在 Web 应用中负责查找`View`对象，从而将相应结果渲染给客户。


# 视图解析器
Spring 视图解析器是 Spring MVC 中的重要组成部分，用户可以在配置文件中定义 Spring MVC 的一个视图解析器（`ViewResolver`）：
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
  <!--前缀-->
  <property name="prefix" value="/WEB-INF/jsp/"/>
  <!--后缀-->
  <property name="suffix" value=".jsp"/>
</bean>
```
上述视图解析器配置了前缀和后缀两个属性。

`InternalResourceViewResolver`是`URLBasedViewResolver`的子类，所以`URLBasedViewResolver`支持的特性它都支持。

在实际应用中`InternalResourceViewResolver`也是使用的最广泛的一个视图解析器。

单从字面意思来看，我们可以把`InternalResourceViewResolver`解释为内部资源视图解析器，这就是`InternalResourceViewResolver`的一个特性。

`InternalResourceViewResolver`会把返回的视图名称都解析为`InternalResourceView`对象，`InternalResourceView`会把`Controller`处理器方法返回的模型属性都存放到对应的`request`属性中，然后通过`RequestDispatcher`在服务器端把请求`forword`重定向到目标 URL。

比如在`InternalResourceViewResolver`中定义了`prefix=/WEB-INF/`，`suffix=.jsp`，然后请求的`Controller`处理器方法返回的视图名称为`login`，那么这个时候`InternalResourceViewResolver`就会把`login`解析为一个`InternalResourceView`对象，先把返回的模型属性都存放到对应的`HttpServletRequest`属性中，然后利用`RequestDispatcher`在服务器端把请求`forword`到`/WEB-INF/test.jsp`。

这就是`InternalResourceViewResolver`一个非常重要的特性，我们都知道存放在`/WEB-INF/`下面的内容是不能直接通过`request`请求的方式请求到的，为了安全性考虑，我们通常会把`jsp`文件放在`WEB-INF`目录下，而`InternalResourceView`在服务器端跳转的方式可以很好的解决这个问题。


# Controller 注解类型
在 Spring MVC 中使用`org.springframework.stereotype.Controller`注解类型声明某类的实例是一个控制器。例如，在`springMVCDemo02`应用的`src`目录下创建`controller`包，并在该包中创建`Controller`注解的控制器类`IndexController`：
```java
package controller;
import org.springframework.stereotype.Controller;
/**
* “@Controller”表示 IndexController 的实例是一个控制器
*
* @Controller相当于@Controller(@Controller) 或@Controller(value="@Controller")
*/
@Controller
public class IndexController {
  // 处理请求的方法
}
```
在 Spring MVC 中使用扫描机制找到应用中所有基于注解的控制器类，所以，为了让控制器类被 Spring MVC 框架扫描到，需要在配置文件中声明`spring-context`，并使用`<context：component-scan/>`元素指定控制器类的基本包（请确保所有控制器类都在基本包及其子包下）。

例如，在`springMVCDemo02`应用的`/WEB-INF/`目录下创建配置文件`springmvc-servlet.xml`：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:p="http://www.springframework.org/schema/p" 
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="
      http://www.springframework.org/schema/beans
      http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/context
      http://www.springframework.org/schema/context/spring-context.xsd
      http://www.springframework.org/schema/mvc
      http://www.springframework.org/schema/mvc/spring-mvc.xsd">
  <!-- 使用扫描机制扫描控制器类，控制器类都在controller包及其子包下 -->
  <context:component-scan base-package="controller" />
  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/" />
    <property name="suffix" value=".jsp" />
  </bean>
</beans>
```
# RequestMapping 注解类型
在基于注解的控制器类中可以为每个请求编写对应的处理方法。那么如何将请求与处理方法一一对应呢？

需要使用`org.springframework.web.bind.annotation.RequestMapping`注解类型将请求与处理方法一一对应。
## 1.方法级别注解
```java
package controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
/**
* “@Controller”表示 IndexController 的实例是一个控制器
*
* @Controller相当于@Controller(@Controller) 或@Controller(value="@Controller")
*/
@Controller
public class IndexController {
  @RequestMapping(value = "/index/login")
  public String login() {
    /**
      * login代表逻辑视图名称，需要根据Spring MVC配置
      * 文件中internalResourceViewResolver的前缀和后缀找到对应的物理视图
      */
    return "login";
  }
  @RequestMapping(value = "/index/register")
  public String register() {
    return "register";
  }
}
```
上述示例中有两个 RequestMapping 注解语句，它们都作用在处理方法上。注解的 value 属性将请求 URI 映射到方法，value 属性是 RequestMapping 注解的默认属性，如果只有一个 value 属性，则可以省略该属性。

用户可以使用如下 URL 访问 login 方法（请求处理方法），在访问`login`方法之前需要事先在`/WEB-INF/jsp/`目录下创建`login.jsp`。
```
http://localhost:8080/springMVCDemo02/index/login
```
2）类级别注解
```java
package controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/login")
    public String login() {
        return "login";
    }
    @RequestMapping("/register")
    public String register() {
        return "register";
    }
}
```
在类级别注解的情况下，控制器类中的所有方法都将映射为类级别的请求。用户可以使用如下 URL 访问 login 方法。
```
http://localhost:8080/springMVCDemo02/index/login
```
# 编写请求处理方法
在控制类中每个请求处理方法可以有多个不同类型的参数，以及一个多种类型的返回结果。
1）请求处理方法中常出现的参数类型
如果需要在请求处理方法中使用 Servlet API 类型，那么可以将这些类型作为请求处理方法的参数类型。Servlet API 参数类型的示例代码如下：
```java
package controller;
import javax.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
  @RequestMapping("/login")
  public String login(HttpSession session,HttpServletRequest request) {
    session.setAttribute("skey", "session范围的值");
    session.setAttribute("rkey", "request范围的值");
    return "login";
  }
}
```
除了 Servlet API 参数类型以外，还有输入输出流、表单实体类、注解类型、与 Spring 框架相关的类型等，这些类型在后续章节中使用时再详细介绍。

其中特别重要的类型是 org.springframework.ui.Model 类型，该类型是一个包含 Map 的 Spring 框架类型。在每次调用请求处理方法时 Spring MVC 都将创建 org.springframework.ui.Model 对象。Model 参数类型的示例代码如下：
```java
package controller;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
  @RequestMapping("/register")
  public String register(Model model) {
    /*在视图中可以使用EL表达式${success}取出model中的值*/
    model.addAttribute("success", "注册成功");
    return "register";
  }
}
```
2）请求处理方法常见的返回类型
最常见的返回类型就是代表逻辑视图名称的 String 类型。除了 String 类型以外，还有 ModelAndView、Model、View 以及其他任意的 Java 类型。
# 获取参数的几种常见方式
