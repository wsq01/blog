---
title: SpringMVC 类型转换器
date: 2021-07-18 17:36:55
tags: [SpringMVC]
categories: [SpringMVC]
---

# 类型转换器
Spring MVC 框架的`Converter<S，T>`是一个可以将一种数据类型转换成另一种数据类型的接口，这里`S`表示源类型，`T`表示目标类型。
## 内置的类型转换器
Spring MVC 框架提供的内置类型转换包括以下几种类型。
### 1.标量转换器
| 名称                           | 作用 |
| :--: | :--: |
| StringToBooleanConverter       | String 到 boolean 类型转换 |
| ObjectToStringConverter        | Object 到 String 转换，调用 toString 方法转换 |
| StringToNumberConverterFactory | String 到数字转换（例如 Integer、Long 等） |
| NumberToNumberConverterFactory | 数字子类型（基本类型）到数字类型（包装类型）转换 |
| StringToCharacterConverter     | String 到 Character 转换，取字符串中的第一个字符 |
| NumberToCharacterConverter     | 数字子类型到 Character 转换 |
| CharacterToNumberFactory       | Character 到数字子类型转换 |
| StringToEnumConverterFactory   | String 到枚举类型转换，通过 Enum.valueOf 将字符串转换为需要的枚举类型 |
| EnumToStringConverter          | 枚举类型到 String 转换，返回枚举对象的 name 值 |
| StringToLocaleConverter        | String 到 java.util.Locale 转换 |
| PropertiesToStringConverter    | java.util.Properties 到 String 转换，默认通过 ISO-8859-1 解码 |
| StringToPropertiesConverter    | String 到 java.util.Properties 转换，默认使用 ISO-8859-1 编码 |

### 2.集合、数组相关转换器
| 名称                            | 作用 |
| :--: | :--: | 
| ArrayToCollectionConverter      | 任意数组到任意集合（List、Set）转换 |
| CollectionToArrayConverter      | 任意集合到任意数组转换 |
| ArrayToArrayConverter           | 任意数组到任意数组转换 |
| CollectionToCollectionConverter	| 集合之间的类型转换 |
| MapToMapConverter               | Map之间的类型转换 |
| ArrayToStringConverter          | 任意数组到 String 转换 |
| StringToArrayConverter          | 字符串到数组的转换，默认通过“，”分割，且去除字符串两边的空格（trim） |
| ArrayToObjectConverter          | 任意数组到 Object 的转换，如果目标类型和源类型兼容，直接返回源对象；否则返回数组的第一个元素并进行类型转换 |
| ObjectToArrayConverter          | Object 到单元素数组转换 |
| CollectionToStringConverter     | 任意集合（List、Set）到 String 转换 |
| StringToCollectionConverter     | String 到集合（List、Set）转换，默认通过“，”分割，且去除字符串两边的空格（trim） |
| CollectionToObjectConverter     | 任意集合到任意 Object 的转换，如果目标类型和源类型兼容，直接返回源对象；否则返回集合的第一个元素并进行类型转换 |
| ObjectToCollectionConverter     | Object 到单元素集合的类型转换类型转换是在视图与控制器相互传递数据时发生的。Spring MVC 框架对于基本类型（例如 int、long、float、double、 boolean 以及 char 等）已经做好了基本类型转换。 |

注意：在使用内置类型转换器时，请求参数输入值与接收参数类型要兼容，否则会报 400 错误。
## 自定义类型转换器
当 Spring MVC 框架内置的类型转换器不能满足需求时，开发者可以开发自己的类型转换器。

例如需要用户在页面表单中输入信息来创建商品信息。当输入`bianchengbang，18，1.85`时表示在程序中自动创建一个`new User`，并将`bianchengbang`值自动赋给`name`属性，将“18”值自动赋给`age`属性，将“1.85”值自动赋给`height`属性。

如果想实现上述应用，需要做以下 5 件事：
* 创建实体类。
* 创建控制器类。
* 创建自定义类型转换器类。
* 注册类型转换器。
* 创建相关视图。

## 示例
1. 创建实体类
在`net.biancheng.po`包下创建`User`实体类。
```java
package net.biancheng.po;
public class User {
  private String name;
  private Integer age;
  private Double height;
  /**省略setter和getter方法*/
}
```
2. 创建控制器类
在`net.biancheng.controller`包下创建`UserController`控制器。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import net.biancheng.po.User;
@Controller
public class UserController {
  @RequestMapping("/addUser")
  public String addUser() {
    return "addUser";
  }
}
```
创建`ConverterController`控制器。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import net.biancheng.po.User;
@Controller
public class ConverterController {
  @RequestMapping("/converter")
  public String myConverter(@RequestParam("user") User user, Model model) {
    model.addAttribute("user", user);
    return "showUser";
  }
}
```
3. 创建自定义类型转换器
创建`net.biancheng.converter`，在该包下创建自定义类型转换器`UserConverter`。
```java
package net.biancheng.converter;
import org.springframework.core.convert.converter.Converter;
import org.springframework.stereotype.Component;
import net.biancheng.po.User;
@Component
public class UserConverter implements Converter<String, User> {
  @Override
  public User convert(String source) {
    // 创建User实例
    User user = new User();
    // 以“，”分隔
    String stringvalues[] = source.split(",");
    if (stringvalues != null && stringvalues.length == 3) {
      // 为user实例赋值
      user.setName(stringvalues[0]);
      user.setAge(Integer.parseInt(stringvalues[1]));
      user.setHeight(Double.parseDouble(stringvalues[2]));
      return user;
    } else {
      throw new IllegalArgumentException(String.format("类型转换失败， 需要格式'编程帮, 18,1.85',但格式是[% s ] ", source));
    }
  }
}
```
4. 配置转换器
在`springmvc-servlet.xml`文件中添加以下代码。
```xml
<mvc:annotation-driven conversion-service="conversionService" />
<!--注册类型转换器UserConverter -->
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
  <property name="converters">
    <list>
      <bean class="net.biancheng.converter.UserConverter" />
    </list>
  </property>
</bean>
```
5. 创建相关视图
创建添加用户页面`addUser.jsp`。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>添加用户</title>
</head>
<body>
  <form action="${pageContext.request.contextPath}/converter" method="post">
    请输入用户信息（格式为编程帮, 18,1.85）:
    <input type="text" name="user" />
    <br>
    <input type="submit" value="提交" />
  </form>
</body>
</html>
```
创建显示用户页面`showUser.jsp`。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>文件上传</title>
</head>
<body>
  您创建的用户信息如下：
  <br/>
  <!-- 使用EL表达式取出model中的user信息 -->
  用户名：${user.name } <br/>
  年龄：${user.age } <br/>
  身高：${user.height }
</body>
</html>
```

# 数据格式化
Spring MVC 框架的`Formatter<T>`与`Converter<S, T>`一样，也是一个可以将一种数据类型转换成另一种数据类型的接口。不同的是，`Formatter`的源类型必须是`String`类型，而`Converter`的源类型可以是任意数据类型。`Formatter`更适合 Web 层，而`Converter`可以在任意层中。所以对于需要转换表单中的用户输入的情况，应该选择`Formatter`，而不是`Converter`。

在 Web 应用中由 HTTP 发送的请求数据到控制器中都是以`String`类型获取，因此在 Web 应用中选择`Formatter<T>`比选择`Converter<S, T>`更加合理。
## 内置的格式化转换器
Spring MVC 提供了几个内置的格式化转换器，具体如下。
* `NumberFormatter`：实现`Number`与`String`之间的解析与格式化。
* `CurrencyFormatter`：实现`Number`与`String`之间的解析与格式化（带货币符号）。
* `PercentFormatter`：实现`Number`与`String`之间的解析与格式化（带百分数符号）。
* `DateFormatter`：实现`Date`与`String`之间的解析与格式化。

## 自定义格式化转换器
自定义格式化转换器就是编写一个实现`org.springframework.format.Formatter`接口的 Java 类。该接口声明如下。
```
public interface Formatter<T>
```
这里的`T`表示由字符串转换的目标数据类型。该接口有`parse`和`print`两个接口方法，自定义格式化转换器类必须覆盖它们。
```
public T parse(String s, java.util.Locale locale)
public String print(T object, java.util.Locale locale)
```
`parse`方法的功能是利用指定的`Locale`将一个`String`类型转换成目标类型，`print`方法与之相反，用于返回目标对象的字符串表示。
## 示例
1. 创建实体类
创建`net.biancheng.po`包，并在该包中创建`User`实体类。
```java
package net.biancheng.po;
import java.util.Date;
public class User {
  private String name;
  private Integer age;
  private Double height;
  private Date createDate;
  /**省略setter和getter方法*/
}
```
2. 创建控制器类
创建`net.biancheng.controller`包，并在该包中创建`FormatterController`控制器类。
```java
package net.biancheng.controller;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import net.biancheng.po.User;
@Controller
public class FormatterController {
  @RequestMapping("/formatter")
  public String myFormatter(User us, Model model) {
    model.addAttribute("user", us);
    return "showUser";
  }
}
```
3. 创建自定义格式化转换器类
创建`net.biancheng.formatter`包，并在该包中创建`MyFormatter`的自定义格式化转换器类。
```java
package net.biancheng.formatter;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;
import org.springframework.format.Formatter;
import org.springframework.stereotype.Component;
@Component
public class MyFormatter implements Formatter<Date> {
  SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
  public String print(Date object, Locale arg1) {
    return dateFormat.format(object);
  }
  public Date parse(String source, Locale arg1) throws ParseException {
    return dateFormat.parse(source); // Formatter只能对字符串转换
  }
}
```
4. 注册格式化转换器
在`springmvc-servlet.xml`配置文件中注册格式化转换器，具体代码如下：
```xml
<!--注册MyFormatter -->
<bean id="conversionService"
  class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
  <property name="formatters">
    <set>
      <bean class="net.biancheng.formatter.MyFormatter" />
    </set>
  </property>
</bean>
<mvc:annotation-driven conversion-service="conversionService" />
```
5. 创建相关视图
创建添加用户页面`addUser.jsp`。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>添加用户</title>
</head>
<body>
  <form action="${pageContext.request.contextPath}/formatter" method="post">
    用户名：<input type="text" name="name" />
    <br>
    年龄：<input type="text" name="age" />
    <br>
    身高：<input type="text" name="height" />
    <br>
    创建日期：<input type="text" name="createDate" />
    <br>
    <input type="submit" value="提交" />
  </form>
</body>
</html>
```
创建信息显示页面`showUser.jsp`。
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>用户信息</title>
</head>
<body>
  您创建的用户信息如下：
  <br />
  <!-- 使用EL表达式取出model中的user信息 -->
  用户名：${user.name }
  <br />
  年龄：${user.age }
  <br />
  身高：${user.height }
  <br />
  创建日期：${user.createDate }
</body>
</html>
```