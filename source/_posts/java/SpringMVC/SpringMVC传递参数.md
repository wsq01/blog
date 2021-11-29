---
title: SpringMVC 传递参数
date: 2021-07-14 19:16:21
tags: [SpringMVC]
categories: [SpringMVC]
---


# 传递参数
`Controller`接收请求参数的方式有很多种，有的适合`get`请求方式，有的适合`post`请求方式，有的两者都适合。主要有以下几种方式：
* 通过实体`Bean`接收请求参数
* 通过处理方法的形参接收请求参数
* 通过`HttpServletRequest`接收请求参数
* 通过`@PathVariable`接收 URL 中的请求参数
* 通过`@RequestParam`接收请求参数
* 通过`@ModelAttribute`接收请求参数

## 通过实体 Bean 接收请求参数
通过实体`Bean`接收请求参数适用于`get`和`post`提交请求方式。需要注意的是，`Bean`的属性名称必须与请求参数名称相同。
```java
@RequestMapping("/login")
public String login(User user, Model model) {
  if ("zhangsan".equals(user.getName()) && "123456".equals(user.getPwd())) {
    model.addAttribute("message", "登录成功");
    return "main"; // 登录成功，跳转到 main.jsp
  } else {
    model.addAttribute("message", "用户名或密码错误");
    return "login";
  }
}
```
## 通过处理方法的形参接收请求参数
通过处理方法的形参接收请求参数就是直接把表单参数写在控制器类相应方法的形参中，即形参名称与请求参数名称完全相同。该接收参数方式适用于`get`和`post`提交请求方式。
```java
@RequestMapping("/login")
public String login(String name, String pwd, Model model) {
  if ("zhangsan".equals(user.getName()) && "123456".equals(user.getPwd())) { 
    model.addAttribute("message", "登录成功");
    return "main"; // 登录成功，跳转到 main.jsp
  } else {
    model.addAttribute("message", "用户名或密码错误");
    return "login";
  }
}
```
## 通过HttpServletRequest接收请求参数
通过`HttpServletRequest`接收请求参数适用于`get`和`post`提交请求方式。
```java
@RequestMapping("/login")
public String login(HttpServletRequest request, Model model) {
  String name = request.getParameter("name");
  String pwd = request.getParameter("pwd");
  
  if ("zhangsan".equals(name) && "123456".equals(pwd)) {
    model.addAttribute("message", "登录成功");
    return "main"; // 登录成功，跳转到 main.jsp
  } else {
    model.addAttribute("message", "用户名或密码错误");
    return "login";
  }
}
```
## 通过@PathVariable接收URL中的请求参数
通过`@PathVariable`获取 URL 中的参数。
```java
@RequestMapping("/login/{name}/{pwd}")
public String login(@PathVariable String name, @PathVariable String pwd, Model model) {
  if ("zhangsan".equals(name) && "123456".equals(pwd)) {
    model.addAttribute("message", "登录成功");
    return "main"; // 登录成功，跳转到 main.jsp
  } else {
    model.addAttribute("message", "用户名或密码错误");
    return "login";
  }
}
```
在访问`http://localhost:8080/springMVCDemo02/user/register/zhangsan/123456`路径时，上述代码会自动将 URL 中的模板变量`{name}`和`{pwd}`绑定到通过`@PathVariable`注解的同名参数上，即`name=zhangsan、pwd=123456`。
## 通过@RequestParam接收请求参数
在方法入参处使用`@RequestParam`注解指定其对应的请求参数。`@RequestParam`有以下三个参数：
* `value`：参数名
* `required`：是否必须，默认为`true`，表示请求中必须包含对应的参数名，若不存在将抛出异常
* `defaultValue`：参数默认值

通过`@RequestParam`接收请求参数适用于`get`和`post`提交请求方式。
```java
@RequestMapping("/login")
public String login(@RequestParam String name, @RequestParam String pwd, Model model) {
  if ("zhangsan".equals(name) && "123456".equals(pwd)) {
    model.addAttribute("message", "登录成功");
    return "main"; // 登录成功，跳转到 main.jsp
  } else {
    model.addAttribute("message", "用户名或密码错误");
    return "login";
  }
}
```
该方式与“通过处理方法的形参接收请求参数”部分的区别：当请求参数与接收参数名不一致时，“通过处理方法的形参接收请求参数”不会报 404 错误，而“通过`@RequestParam`接收请求参数”会报 404 错误。
## 通过@ModelAttribute接收请求参数
`@ModelAttribute`注解用于将多个请求参数封装到一个实体对象中，从而简化数据绑定流程，而且自动暴露为模型数据，在视图页面展示时使用。

而“通过实体`Bean`接收请求参数”中只是将多个请求参数封装到一个实体对象，并不能暴露为模型数据（需要使用`model.addAttribute`语句才能暴露为模型数据）。

通过`@ModelAttribute`注解接收请求参数适用于`get`和`post`提交请求方式。
```java
@RequestMapping("/login")
public String login(@ModelAttribute("user") User user, Model model) {
  if ("zhangsan".equals(name) && "123456".equals(pwd)) {
    model.addAttribute("message", "登录成功");
    return "main"; // 登录成功，跳转到 main.jsp
  } else {
    model.addAttribute("message", "用户名或密码错误");
    return "login";
  }
}
```
# 重定向和转发
Spring MVC 请求方式分为转发、重定向 2 种，分别使用`forward`和`redirect`关键字在`controller`层进行处理。

重定向是将用户从当前处理请求定向到另一个视图（例如 JSP）或处理请求，以前的请求（`request`）中存放的信息全部失效，并进入一个新的`request`作用域；转发是将用户对当前处理的请求转发给另一个视图或处理请求，以前的`request`中存放的信息不会失效。

转发是服务器行为，重定向是客户端行为。
## 转发过程
客户浏览器发送`http`请求，Web 服务器接受此请求，调用内部的一个方法在容器内部完成请求处理和转发动作，将目标资源发送给客户；在这里转发的路径必须是同一个 Web 容器下的 URL，其不能转向到其他的 Web 路径上，中间传递的是自己的容器内的`request`。

在客户浏览器的地址栏中显示的仍然是其第一次访问的路径，也就是说客户是感觉不到服务器做了转发的。转发行为是浏览器只做了一次访问请求。
## 重定向过程
客户浏览器发送`http`请求，Web 服务器接受后发送 302 状态码响应及对应新的`location`给客户浏览器，客户浏览器发现是 302 响应，则自动再发送一个新的`http`请求，请求 URL 是新的`location`地址，服务器根据此请求寻找资源并发送给客户。

在这里`location`可以重定向到任意 URL，既然是浏览器重新发出了请求，那么就没有什么`request`传递的概念了。在客户浏览器的地址栏中显示的是其重定向的路径，客户可以观察到地址的变化。重定向行为是浏览器做了至少两次的访问请求。

在 Spring MVC 框架中，控制器类中处理方法的`return`语句默认就是转发实现，只不过实现的是转发到视图。
```java
@RequestMapping("/register")
public String register() {
  return "register";  //转发到register.jsp
}
```
在 Spring MVC 框架中，重定向与转发的示例代码如下：
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
@Controller
@RequestMapping("/index")
public class IndexController {
  @RequestMapping("/login")
  public String login() {
    //转发到一个请求方法（同一个控制器类可以省略/index/）
    return "forward:/index/isLogin";
  }
  @RequestMapping("/isLogin")
  public String isLogin() {
    //重定向到一个请求方法
    return "redirect:/index/isRegister";
  }
  @RequestMapping("/isRegister")
  public String isRegister() {
    //转发到一个视图
    return "register";
  }
}
```
在 Spring MVC 框架中，不管是重定向或转发，都需要符合视图解析器的配置，如果直接转发到一个不需要`DispatcherServlet`的资源，例如：
`return "forward:/html/my.html"`;

则需要使用`mvc:resources`配置：
```xml
<mvc:resources location="/html/" mapping="/html/**" />
```