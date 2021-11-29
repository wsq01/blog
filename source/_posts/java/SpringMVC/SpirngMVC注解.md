---
title: SpringMVC 注解
date: 2021-07-16 12:33:42
tags: [SpringMVC]
categories: [SpringMVC]
---

# @Controller注解
`@Controller`注解用于声明某类的实例是一个控制器。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
@Controller
public class IndexController {
  // 处理请求的方法
}
```
Spring MVC 使用扫描机制找到应用中所有基于注解的控制器类，所以，为了让控制器类被 Spring MVC 框架扫描到，需要在配置文件中声明`spring-context`，并使用`<context:component-scan/>`元素指定控制器类的基本包（请确保所有控制器类都在基本包及其子包下）。

在应用的配置文件`springmvc-servlet.xml`中添加以下代码：
```xml
<!-- 使用扫描机制扫描控制器类，控制器类都在net.biancheng.controller包及其子包下 -->
<context:component-scan base-package="net.biancheng.controller" />
```
# @RequestMapping注解
一个控制器内有多个处理请求的方法，如`UserController`里通常有增加用户、修改用户信息、删除指定用户、根据条件获取用户列表等。每个方法负责不同的请求操作，而`@RequestMapping`就负责将请求映射到对应的控制器方法上。

在基于注解的控制器类中可以为每个请求编写对应的处理方法。使用`@RequestMapping`注解将请求与处理方法一一对应即可。

`@RequestMapping`注解可用于类或方法上。用于类上，表示类中的所有响应请求的方法都以该地址作为父路径。

`@RequestMapping`注解常用属性如下。
## 1. value 属性
`value`属性是`@RequestMapping`注解的默认属性，因此如果只有`value`属性时，可以省略该属性名，如果有其它属性，则必须写上`value`属性名称。
```java
@RequestMapping(value="toUser")
@RequestMapping("toUser")
```
`value`属性支持通配符匹配，如`@RequestMapping(value="toUser/*")`表示`http://localhost:8080/toUser/1`或`http://localhost:8080/toUser/hahaha`都能够正常访问。
## 2. path属性
`path`属性和`value`属性都用来作为映射使用。即`@RequestMapping(value="toUser")`和`@RequestMapping(path="toUser")`都能访问`toUser()`方法。

`path`属性支持通配符匹配，如`@RequestMapping(path="toUser/*")`表示`http://localhost:8080/toUser/1`或`http://localhost:8080/toUser/hahaha`都能够正常访问。
## 3. name属性
`name`属性相当于方法的注释。如`@RequestMapping(value = "toUser",name = "获取用户信息")`。
## 4. method属性
`method`属性用于表示该方法支持哪些`HTTP`请求。如果省略`method`属性，则说明该方法支持全部的`HTTP`请求。

`@RequestMapping(value = "toUser",method = RequestMethod.GET)`表示该方法只支持`GET`请求。也可指定多个`HTTP`请求，如`@RequestMapping(value = "toUser",method = {RequestMethod.GET,RequestMethod.POST})`，说明该方法同时支持`GET`和`POST`请求。
## 5. params属性
`params`属性用于指定请求中规定的参数。
```java
@RequestMapping(value = "toUser",params = "type")
public String toUser() {
  return "showUser";
}
```
以上代码表示请求中必须包含`type`参数时才能执行该请求。即`http://localhost:8080/toUser?type=xxx`能够正常访问`toUser()`方法，而`http://localhost:8080/toUser`则不能正常访问`toUser()`方法。
```java
@RequestMapping(value = "toUser",params = "type=1")
public String toUser() {
  return "showUser";
}
```
以上代码表示请求中必须包含`type`参数，且`type`参数为 1 时才能够执行该请求。即`http://localhost:8080/toUser?type=1`能够正常访问`toUser()`方法，而`http://localhost:8080/toUser?type=2`则不能正常访问`toUser()`方法。
## 6. header属性
`header`属性表示请求中必须包含某些指定的`header`值。

`@RequestMapping(value = "toUser",headers = "Referer=http://www.xxx.com")`表示请求的`header`中必须包含了指定的`Referer`请求头，以及值为`http://www.xxx.com`时，才能执行该请求。
## 7. consumers属性
`consumers`属性用于指定处理请求的提交内容类型（`Content-Type`），例如：`application/json、text/html`。如`@RequestMapping(value = "toUser",consumes = "application/json")`。
## 8. produces属性
`produces`属性用于指定返回的内容类型，返回的内容类型必须是`request`请求头（`Accept`）中所包含的类型。如`@RequestMapping(value = "toUser",produces = "application/json")`。

除此之外，`produces`属性还可以指定返回值的编码。如`@RequestMapping(value = "toUser",produces = "application/json,charset=utf-8")`，表示返回`utf-8`编码。

使用`@RequestMapping`来完成映射，具体包括 4 个方面的信息项：请求 URL、请求参数、请求方法和请求头。
## 通过请求URL进行映射
### 方法级别注解
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
public class IndexController {
  @RequestMapping(value = "/index/login")
  public String login() {
    return "login";
  }
  @RequestMapping(value = "/index/register")
  public String register() {
    return "register";
  }
}
```
上述示例中有两个`RequestMapping`注解语句，它们都作用在处理方法上。在整个 Web 项目中，`@RequestMapping`映射的请求信息必须保证全局唯一。

用户可以使用如下 URL 访问`login`方法（请求处理方法），在访问`login`方法之前需要事先在`/WEB-INF/jsp/`目录下创建`login.jsp`。
```
http://localhost:8080/springmvcDemo/index/login
```
### 类级别注解
```java
package net.biancheng.controller;
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
在类级别注解的情况下，控制器类中的所有方法都将映射为类级别的请求。用户可以使用如下 URL 访问`login`方法。
```
http://localhost:8080/springmvcDemo/index/login
```
## 通过请求参数、请求方法进行映射
`@RequestMapping`除了可以使用请求 URL 映射请求之外，还可以使用请求参数、请求方法来映射请求，通过多个条件可以让请求映射更加精确。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
public class IndexController {
  @RequestMapping(value = "/index/success" method=RequestMethod.GET, params="username")
  public String success(@RequestParam String username) {
    return "index";
  }
}
```
# @Autowired和@Service注解
将依赖注入到 Spring MVC 控制器时需要用到`@Autowired`和`@Service`注解。

`@Autowired`注解属于`org.springframework.beans.factory. annotation`包，可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。

`@Service`注解属于`org.springframework.stereotype`包，会将标注类自动注册到 Spring 容器中。

在配置文件中需要添加`<component-scan/>`元素来扫描依赖基本包。
```xml
<context:component-scan base-package="net.biancheng.service"/>
```
## 示例
新建 Web 应用`springmvcDemo3`进一步说明 Spring MVC 如何应用依赖注入。
```java
package net.biancheng.po;
public class User {
    private String name;
    private String pwd;
    /*省略setter和getter方法*/
}
```
```java
package net.biancheng.service;
import net.biancheng.po.User;
public interface UserService {
    boolean login(User user);
    boolean register(User user);
}
```
```java
package net.biancheng.service;
import org.springframework.stereotype.Service;
import net.biancheng.po.User;
@Service
public class UserServiceImpl implements UserService {
  @Override
  public boolean login(User user) {
    if ("bianchengbang".equals(user.getName()) && "123456".equals(user.getPwd())) {
      return true;
    }
    return false;
  }
  @Override
  public boolean register(User user) {
    if ("bianchengbang".equals(user.getName()) && "123456".equals(user.getPwd())) {
      return true;
    }
    return false;
  }
}
```
注意：为了使类能被 Spring 扫描到，必须为其标注`@Service`。
```java
package net.biancheng.controller;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import net.biancheng.po.User;
import net.biancheng.service.UserService;
@Controller
@RequestMapping("/user")
public class UserController {
  @Autowired
  private UserService userService;
  @RequestMapping("/login")
  public String getLogin(Model model) {
    User us = new User();
    us.setName("bianchengbang");
    userService.login(us);
    model.addAttribute("user", us);
    return "login";
  }
  @RequestMapping("/register")
  public String getRegister(Model model) {
    User us = new User();
    us.setName("bianchengbang");
    userService.login(us);
    model.addAttribute("user", us);
    return "register";
  }
}
```
在`UserService`上添加`@Autowired`注解会使`UserService`的一个实例被注入到`UserController`实例中。

`springmvc-servlet.xml`代码如下。
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
    <context:component-scan
        base-package="net.biancheng" />
    <mvc:annotation-driven />
    <bean id="viewResolver"
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!--前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀 -->
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```
`web.xml`代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
    <display-name>springMVC</display-name>
    <!-- 部署 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/springmvc-servlet.xml</param-value>
        </init-param>
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
`index.jsp`文件内容如下。
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
    <a href="${pageContext.request.contextPath }/user/register"> 注册</a>！
    <br /> 已注册的用户，去
    <a href="${pageContext.request.contextPath }/user/login"> 登录</a>！
</body>
</html>
```
`login.jsp`文件内容如下。
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
    登录页面！ 欢迎 ${user.name} 登录
</body>
</html>
```
`register.jsp`文件内容如下。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
</head>
<body>
    注册页面！
    注册账号成功，用户名为： ${user.name }
</body>
</html>
```
# @ModelAttribute注解
`@ModelAttribute`用来将请求参数绑定到`Model`对象。

在`Controller`中使用`@ModelAttribute`时，有以下几种应用情况。
* 应用在方法上
* 应用在方法的参数上
* 应用在方法上，并且方法也使用了`@RequestMapping`

需要注意的是，因为模型对象要先于`controller`方法之前创建，所以被`@ModelAttribute`注解的方法会在`Controller`每个方法执行之前都执行。因此一个`Controller`映射多个 URL 时，要谨慎使用。
## 应用在方法上
### 1.应用在无返回值的方法
示例 1：创建`ModelAttributeController`。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
@Controller
public class ModelAttributeController {
  // 方法无返回值
  @ModelAttribute
  public void myModel(@RequestParam(required = false) String name, Model model) {
    model.addAttribute("name", name);
  }
  @RequestMapping(value = "/model")
  public String model() {
    return "index";
  }
}
```
创建`index.jsp`页面。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>测试</title>
</head>
<body>
    ${name}
</body>
</html>
```
访问地址：`http://localhost:8080/springmvcDemo2/model?name=%E7%BC%96%E7%A8%8B%E5%B8%AE`。

在请求`/model?name=%E7%BC%96%E7%A8%8B%E5%B8%AE`后，Spring MVC 会先执行`myModel`方法，将`name`的值存入到`Model`中。然后执行`model`方法，这样`name`的值就被带到了`model`方法中。

将`myModel`和`model`方法合二为一后，代码如下。
```java
@RequestMapping(value = "/model")
public String model(@RequestParam(required = false) String name, Model model) {
  model.addAttribute("name", name);
  return "index";
}
```
### 2.应用在有返回值的方法
示例 2：修改`ModelAttributeController`控制类，代码如下。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
@Controller
public class ModelAttributeController {
  // 方法有返回值
  @ModelAttribute("name")
  public String myModel(@RequestParam(required = false) String name) {
    return name;
  }
  @RequestMapping(value = "/model")
  public String model() {
    return "index";
  }
}
```
修改 index.jsp，代码如下。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>编程帮</title>
</head>
<body>
  ${string }
</body>
</html>
```
访问地址和运行结果与示例 1 相同。

对于以上情况，返回值对象`name`会被默认放到隐含的`Model`中，在`Model`中`key`为返回值首字母小写，`value`为返回的值。等同于`model.addAttribute("string", name);`。

但正常情况下，程序中尽量不要出现`key`为`string、int、float`等这样数据类型的返回值。使用`@ModelAttribute`注解`value`属性可以自定义`key`，代码如下。
```java
// 方法有返回值
@ModelAttribute("name")
public String myModel(@RequestParam(required = false) String name) {
  return name;
}
// 等同于
model.addAttribute("name", name);
```
## 应用在方法的参数上
`@ModelAttribute`注解在方法的参数上，调用方法时，模型的值会被注入。这在实际使用时非常简单，常用于将表单属性映射到模型对象。
```java
@RequestMapping("/register")
public String register(@ModelAttribute("user") UserForm user) {
  if ("zhangsan".equals(uname) && "123456".equals(upass)) {
    logger.info("成功");
    return "login";
  } else {
    logger.info("失败");
    return "register";
  }
}
```
上述代码中`@ModelAttribute("user") UserForm user`语句的功能有两个：
* 将请求参数的输入封装到`user`对象中
* 创建`UserForm`实例

以`user`为键值存储在`Model`对象中，和`model.addAttribute("user",user)`语句的功能一样。如果没有指定键值，即`@ModelAttribute UserForm user`，那么在创建`UserForm`实例时以`userForm`为键值存储在`Model`对象中，和`model.addAtttribute("userForm", user)`语句的功能一样。
## ModelAttribute+RequestMapping
示例 3：修改`ModelAttributeController`，代码如下。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
@Controller
public class ModelAttributeController {
  // @ModelAttribute和@RequestMapping同时放在方法上
  @RequestMapping(value = "/index")
  @ModelAttribute("name")
  public String model(@RequestParam(required = false) String name) {
    return name;
  }
}
```
`index.jsp`代码如下。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>编程帮</title>
</head>
<body>
    ${name }
</body>
</html>
```
访问地址：`http://localhost:8080/springmvcDemo2/index?name=%E7%BC%96%E7%A8%8B%E5%B8%AE`。

`@ModelAttribute`和`@RequestMapping`注解同时应用在方法上时，有以下作用：
1. 方法的返回值会存入到`Model`对象中，`key`为`ModelAttribute`的`value`属性值。
2. 方法的返回值不再是方法的访问路径，访问路径会变为`@RequestMapping`的`value`值，例如：`@RequestMapping(value = "/index")`跳转的页面是`index.jsp`页面。

总而言之，`@ModelAttribute`注解的使用方法有很多种，非常灵活，可以根据业务需求选择使用。

`Model`和`ModelView`的区别：
`Model`：每次请求中都存在的默认参数，利用其`addAttribute()`方法即可将服务器的值传递到客户端页面中。

`ModelAndView`：包含`model`和`view`两部分，使用时需要自己实例化，利用`ModelMap`来传值，也可以设置`view`的名称。
## 拓展
`@ModelAttribute`注解的方法会在每次调用该控制器类的请求处理方法前被调用。这种特性可以用来控制登录权限。
控制登录权限的方法有很多，例如拦截器、过滤器等。

创建`BaseController`。
```java
package net.biancheng.controller;
import javax.servlet.http.HttpSession;
import org.springframework.web.bind.annotation.ModelAttribute;
public class BaseController {
  @ModelAttribute
  public void isLogin(HttpSession session) throws Exception {
    if (session.getAttribute("user") == null) {
      throw new Exception("没有权限");
    }
  }
}
```
创建`ModelAttributeController`：
```java
package net.biancheng.controller;
import org.springframework.web.bind.annotation.RequestMapping;
@RequestMapping("/admin")
public class ModelAttributeController extends BaseController {
  @RequestMapping("/add")
  public String add() {
    return "addSuccess";
  }
  @RequestMapping("/update")
  public String update() {
    return "updateSuccess";
  }
  @RequestMapping("/delete")
  public String delete() {
    return "deleteSuccess";
  }
}
```
在上述`ModelAttributeController`类中的`add、update、delete`请求处理方法执行时，首先执行父类`BaseController`中的`isLogin`方法判断登录权限，可以通过地址`http://localhost:8080/springMVCDemo2/admin/add`测试登录权限。