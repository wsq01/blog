




# 什么是MVC
MVC 英文是`Model View Controller`，是模型(`model`)－视图(`view`)－控制器(`controller`)的缩写，一种软件设计规范。本质上也是一种解耦。

用一种业务逻辑、数据、界面显示分离的方法，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

{% asset_img spring-springframework-mvc-4.png %}

* `Model`（模型）是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。
* `View`（视图）是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的。
* `Controller`（控制器）是应用程序中处理用户交互的部分。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

# 什么是Spring MVC
简单而言，Spring MVC 是 Spring 在 Spring Container Core 和 AOP 等技术基础上，遵循上述 Web MVC 的规范推出的 web 开发框架，目的是为了简化 Java 栈的 web 开发。

相关特性如下：
* 让我们能非常简单的设计出干净的 Web 层和薄薄的 Web 层；
* 进行更简洁的 Web 层的开发；
* 天生与 Spring 框架集成（如 IoC 容器、AOP 等）；
* 提供强大的约定大于配置的契约式编程支持；
* 能简单的进行 Web 层的单元测试；
* 支持灵活的 URL 到页面控制器的映射；
* 非常容易与其他视图技术集成，如 Velocity、FreeMarker 等等，因为模型数据不放在特定的 API 里，而是放在一个 Model 里（Map 数据结构实现，因此很容易被其他框架使用）；
* 非常灵活的数据验证、格式化和数据绑定机制，能使用任何对象进行数据绑定，不必实现特定框架的 API；
* 提供一套强大的 JSP 标签库，简化 JSP 开发；
* 支持灵活的本地化、主题等解析；
* 更加简单的异常处理；
* 对静态资源的支持；
* 支持 Restful 风格。

# Spring MVC的请求流程
Spring Web MVC 框架也是一个基于请求驱动的 Web 框架，并且也使用了前端控制器模式来进行设计，再根据请求映射 规则分发给相应的页面控制器（动作/处理器）进行处理。
## 核心架构的具体流程步骤
首先让我们整体看一下 Spring Web MVC 处理请求的流程：

{% asset_img spring-springframework-mvc-5.png %}

核心架构的具体流程步骤如下：
* 首先用户发送请求——>`DispatcherServlet`，前端控制器收到请求后自己不进行处理，而是委托给其他的解析器进行处理，作为统一访问点，进行全局的流程控制；
* `DispatcherServlet——>HandlerMapping`，`HandlerMapping`将会把请求映射为`HandlerExecutionChain`对象（包含一个`Handler`处理器（页面控制器）对象、多个`HandlerInterceptor`拦截器）对象，通过这种策略模式，很容易添加新的映射策略；
* `DispatcherServlet——>HandlerAdapter`，`HandlerAdapter`将会把处理器包装为适配器，从而支持多种类型的处理器，即适配器设计模式的应用，从而很容易支持很多类型的处理器；
* `HandlerAdapter`——>处理器功能处理方法的调用，`HandlerAdapter`将会根据适配的结果调用真正的处理器的功能处理方法，完成功能处理；并返回一个`ModelAndView`对象（包含模型数据、逻辑视图名）；
* `ModelAndView`的逻辑视图名——>`ViewResolver`，`ViewResolver`将把逻辑视图名解析为具体的`View`，通过这种策略模式，很容易更换其他视图技术；
* `View`——>渲染，`View`会根据传进来的`Model`模型数据进行渲染，此处的`Model`实际是一个`Map`数据结构，因此很容易支持其他视图技术；
* 返回控制权给`DispatcherServlet`，由`DispatcherServlet`返回响应给用户，到此一个流程结束。

## 对上述流程的补充
上述流程只是核心流程，这里我们再补充一些其它组件：
### Filter(ServletFilter)
进入`Servlet`前可以有`preFilter`，`Servlet`处理之后还可有`postFilter`

{% asset_img spring-springframework-mvc-8.png %}

### LocaleResolver
在视图解析/渲染时，还需要考虑国际化(`Local`)，显然这里需要有`LocaleResolver`。

{% asset_img spring-springframework-mvc-6.png %}

### ThemeResolver
如何控制视图样式呢？SpringMVC 中还设计了`ThemeSource`接口和`ThemeResolver`，包含一些静态资源的集合(样式及图片等），用来控制应用的视觉风格。

{% asset_img spring-springframework-mvc-9.png %}

### 对于文件的上传请求
对于常规请求上述流程是合理的，但是如果是文件的上传请求，那么就不太一样了；所以这里便出现了`MultipartResolver`。

{% asset_img spring-springframework-mvc-7.png %}

# Spring MVC案例
这里主要向你展示一个基本的 SpringMVC 例子，后文中将通过以 Debug 的方式分析源码。本例子中主要文件和结构如下：

{% asset_img spring-springframework-mvc-14.png %}

## Maven包引入
主要引入spring-webmvc包（spring-webmvc包中已经包含了Spring Core Container相关的包），以及servlet和jstl（JSP中使用jstl)的包。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tech-pdai-spring-demos</artifactId>
        <groupId>tech.pdai</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>011-spring-framework-demo-springmvc</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <spring.version>5.3.9</spring.version>
        <servlet.version>4.0.1</servlet.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>${servlet.version}</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
    </dependencies>

</project>
```
## 业务代码的编写
User实体
```java
package tech.pdai.springframework.springmvc.entity;

public class User {

    /**
     * user's name.
     */
    private String name;

    /**
     * user's age.
     */
    private int age;

    /**
     * init.
     *
     * @param name name
     * @param age  age
     */
    public User(String name, int age) {
        this.name = name;
        this.age = age;
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
Dao
```java
package tech.pdai.springframework.springmvc.dao;

import org.springframework.stereotype.Repository;
import tech.pdai.springframework.springmvc.entity.User;

import java.util.Collections;
import java.util.List;

@Repository
public class UserDaoImpl {

    /**
     * mocked to find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return Collections.singletonList(new User("pdai", 18));
    }
}
```
Service
```java
package tech.pdai.springframework.springmvc.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import tech.pdai.springframework.springmvc.dao.UserDaoImpl;
import tech.pdai.springframework.springmvc.entity.User;

import java.util.List;

@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    @Autowired
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return userDao.findUserList();
    }

}
```
Controller
```java
package tech.pdai.springframework.springmvc.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import tech.pdai.springframework.springmvc.service.UserServiceImpl;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.Date;

@Controller
public class UserController {

    @Autowired
    private UserServiceImpl userService;

    /**
     * find user list.
     *
     * @param request  request
     * @param response response
     * @return model and view
     */
    @RequestMapping("/user")
    public ModelAndView list(HttpServletRequest request, HttpServletResponse response) {
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("dateTime", new Date());
        modelAndView.addObject("userList", userService.findUserList());
        modelAndView.setViewName("userList"); // views目录下userList.jsp
        return modelAndView;
    }
}
```
# webapp下的web.xml
（创建上图的文件结构）webapp下的web.xml如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <display-name>SpringFramework - SpringMVC Demo @pdai</display-name>

    <servlet>
        <servlet-name>springmvc-demo</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc-demo</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
### springmvc.xml
web.xml中我们配置初始化参数contextConfigLocation，路径是classpath:springmvc.xml
```
<init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
</init-param>
```
在resources目录下创建
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 扫描注解 -->
    <context:component-scan base-package="tech.pdai.springframework.springmvc"/>

    <!-- 静态资源处理 -->
    <mvc:default-servlet-handler/>

    <!-- 开启注解 -->
    <mvc:annotation-driven/>

    <!-- 视图解析器 -->
    <bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

</beans>
```
# JSP视图创建userList.jsp
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>User List</title>

    <!-- Bootstrap -->
    <link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css">

</head>
<body>
    <div class="container">
        <c:if test="${!empty userList}">
            <table class="table table-bordered table-striped">
                <tr>
                    <th>Name</th>
                    <th>Age</th>
                </tr>
                <c:forEach items="${userList}" var="user">
                    <tr>
                        <td>${user.name}</td>
                        <td>${user.age}</td>
                    </tr>
                </c:forEach>
            </table>
        </c:if>
    </div>
</body>
</html>
```
# 部署测试
我们通过IDEA的tomcat插件来进行测试

下载Tomcat：tomcat地址在新窗口打开
下载后给tomcat/bin执行文件赋权
```
pdai@MacBook-Pro pdai % cd apache-tomcat-9.0.62 
pdai@MacBook-Pro apache-tomcat-9.0.62 % cd bin 
pdai@MacBook-Pro bin % ls
bootstrap.jar			makebase.sh
catalina-tasks.xml		setclasspath.bat
catalina.bat			setclasspath.sh
catalina.sh			shutdown.bat
ciphers.bat			shutdown.sh
ciphers.sh			startup.bat
commons-daemon-native.tar.gz	startup.sh
commons-daemon.jar		tomcat-juli.jar
configtest.bat			tomcat-native.tar.gz
configtest.sh			tool-wrapper.bat
daemon.sh			tool-wrapper.sh
digest.bat			version.bat
digest.sh			version.sh
makebase.bat
pdai@MacBook-Pro bin % chmod 777 *.sh
pdai@MacBook-Pro bin % 
```
配置Run Congfiuration

{% asset_img spring-springframework-mvc-15.png %}
添加Tomcat Server - Local

{% asset_img spring-springframework-mvc-16.png %}

将我们下载的Tomcat和Tomcat Server - Local关联

{% asset_img spring-springframework-mvc-17.png %}

在Deploy中添加我们的项目

{% asset_img spring-springframework-mvc-18.png %}

运行和管理Tomcat Sever（注意context路径）

{% asset_img spring-springframework-mvc-19.png %}

运行后访问我们的web程序页面（注意context路径）

{% asset_img spring-springframework-mvc-20.png %}


