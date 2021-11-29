



# 导入Spring配置
默认情况下，Spring Boot 中是不包含任何的 Spring 配置文件的，即使我们手动添加 Spring 配置文件到项目中，也不会被识别。我们可以通过以下 2 种方式来导入 Spring 配置：
* 使用`@ImportResource`注解加载 Spring 配置文件
* 使用全注解方式加载 Spring 配置

## @ImportResource 导入 Spring 配置文件
在主启动类上使用`@ImportResource`注解可以导入一个或多个 Spring 配置文件，并使其中的内容生效。
```java
package net.biancheng.www.service;
import net.biancheng.www.bean.Person;
public interface PersonService {
  public Person getPersonInfo();
}
```
```java
package net.biancheng.www.service.impl;
import net.biancheng.www.bean.Person;
import net.biancheng.www.service.PersonService;
import org.springframework.beans.factory.annotation.Autowired;
public class PersonServiceImpl implements PersonService {
  @Autowired
  private Person person;
  @Override
  public Person getPersonInfo() {
    return person;
  }
}
```
在该项目的`resources`下添加一个名为`beans.xml`的 Spring 配置文件，配置代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="persionService" class="net.biancheng.www.service.impl.PersonServiceImpl"></bean>
</beans>
```
修改该项目的单元测试类`HelloworldApplicationTests`，校验 IOC 容器是否已经`personService`。
```java
package net.biancheng.www;
import net.biancheng.www.bean.Person;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
@SpringBootTest
class HelloworldApplicationTests {
  @Autowired
  Person person;
  //IOC 容器
  @Autowired
  ApplicationContext ioc;
  @Test
  public void testHelloService() {
    //校验 IOC 容器中是否包含组件 personService
    boolean b = ioc.containsBean("personService");
    if (b) {
      System.out.println("personService 已经添加到 IOC 容器中");
    } else {
      System.out.println("personService 没添加到 IOC 容器中");
    }
  }
  @Test
  void contextLoads() {
    System.out.println(person);
  }
}
```
在主启动程序类上使用`@ImportResource`注解，将 Spring 配置文件`beans.xml`加载到项目中。
```java
package net.biancheng.www;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.ImportResource;
//将 beans.xml 加载到项目中
@ImportResource(locations = {"classpath:/beans.xml"})
@SpringBootApplication
public class HelloworldApplication {
  public static void main(String[] args) {
    SpringApplication.run(HelloworldApplication.class, args);
  }
}
```
8. 执行测试代码，结果如下图。

{% asset_img 4.png 使用 @ImportResource 注解加载 Spring 配置文件结果 %}

## 全注解方式加载 Spring 配置
全注解的方式加载 Spring 配置实现方式如下：
* 使用`@Configuration`注解定义配置类，替换 Spring 的配置文件；
* 配置类内部可以包含有一个或多个被`@Bean`注解的方法，这些方法会被`AnnotationConfigApplicationContext`或`AnnotationConfigWebApplicationContext`类扫描，构建`bean`定义（相当于 Spring 配置文件中的`<bean></bean>`标签），方法的返回值会以组件的形式添加到容器中，组件的`id`就是方法名。

删除主启动类的`@ImportResource`注解，在`net.biancheng.www.config`包下添加一个名为`MyAppConfig`的配置类。
```java
package net.biancheng.www.config;
import net.biancheng.www.service.PersonService;
import net.biancheng.www.service.impl.PersonServiceImpl;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
/**
* @Configuration 注解用于定义一个配置类，相当于 Spring 的配置文件
* 配置类中包含一个或多个被 @Bean 注解的方法，该方法相当于 Spring 配置文件中的 <bean> 标签定义的组件。
*/
@Configuration
public class MyAppConfig {
  /**
    * 与 <bean id="personService" class="PersonServiceImpl"></bean> 等价
    * 该方法返回值以组件的形式添加到容器中
    * 方法名是组件 id（相当于 <bean> 标签的属性 id)
    */
  @Bean
  public PersonService personService() {
    System.out.println("在容器中添加了一个组件:peronService");
    return new PersonServiceImpl();
  }
}
```
执行测试代码，执行结果如下图。

{% asset_img 5.png 全注解方式添加 Spring 配置 %}

# 默认配置文件
通常情况下，Spring Boot 在启动时会将`resources`目录下的`application.properties`或`apllication.yml`作为其默认配置文件，我们可以在该配置文件中对项目进行配置。

Spring Boot 项目中可以存在多个`application.properties`或`apllication.yml`。

Spring Boot 启动时会扫描以下 5 个位置的`application.properties`或`apllication.yml`文件，并将它们作为 Spring boot 的默认配置文件。
* `file:./config/`
* `file:./config/*/`
* `file:./`
* `classpath:/config/`
* `classpath:/`

> 注：`file`: 指当前项目根目录；`classpath`: 指当前项目的类路径，即`resources`目录。

以上所有位置的配置文件都会被加载，且它们优先级依次降低，序号越小优先级越高。其次，位于相同位置的`application.properties`的优先级高于`application.yml`。

所有位置的文件都会被加载，高优先级配置会覆盖低优先级配置，形成互补配置，即：
* 存在相同的配置内容时，高优先级的内容会覆盖低优先级的内容；
* 存在不同的配置内容时，高优先级和低优先级的配置内容取并集。

## 示例
创建一个名为`springbootdemo`的 Spring Boot 项目，并在当前项目根目录下、类路径下的`config`目录下、以及类路径下分别创建一个配置文件`application.yml`，该项目结构如下图。

{% asset_img 1.png Spring Boot 项目配置文件位置 %}

项目根路径下配置文件`application.yml`配置如下。
```
#项目更目录下
#上下文路径为 /abc
server:
  servlet:
    context-path: /abc
```
项目类路径下`config`目录下配置文件`application.yml`配置如下。
```
#类路径下的 config 目录下
#端口号为8084
#上下文路径为 /helloWorld
server:
  port: 8084
  servlet:
    context-path: /helloworld
```
项目类路径下的`application.yml`配置如下。
```
#默认配置
server:
  port: 8080
```
创建一个名为`MyController`的类。
```java
package net.biancheng.www.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
@RestController
public class MyController {
  @ResponseBody
  @RequestMapping("/test")
  public String hello() {
    return "hello Spring Boot!";
  }
}
```
启动 Spring Boot，查看控制台输出。

{% asset_img 2.png Spring Boot 项目启动控制台输出 %}

根据 Spring Boot 默认配置文件优先级进行分析：
* 该项目中存在多个默认配置文件，其中根目录下`/config`目录下的配置文件优先级最高，因此项目的上下文路径为`/abc`；
* 类路径（`classpath`）下`config`目录下的配置文件优先级高于类路径下的配置文件，因此该项目的端口号为`8084`；
* 以上所有配置项形成互补，所以访问路径为`http://localhost:8084/abc`。

# 外部配置文件
除了默认配置文件，Spring Boot 还可以加载一些位于项目外部的配置文件。我们可以通过如下 2 个参数，指定外部配置文件的路径：
```
spring.config.location 
spring.config.additional-location 
```
## spring.config.location
我们可以先将 Spring Boot 项目打包成 JAR 文件，然后在命令行启动命令中，使用命令行参数 --spring.config.location，指定外部配置文件的路径。
java -jar {JAR}  --spring.config.location={外部配置文件全路径}

需要注意的是，使用该参数指定配置文件后，会使项目默认配置文件（application.properties 或 application.yml ）失效，Spring Boot 将只加载指定的外部配置文件。
示例 1
1. 在本地目录 D:\myConfig 下，创建一个配置文件 my-application.yml，配置如下。
```
#指定配置文件
server:
  port: 8088
```
2. 执行以下 mvn 命令，将 springbootdemo 项目打包成 JAR。
```
mvn clean package
```
命令执行结果如下图。

Spring Boot  打包结果
图1：打包结果

3. 打开命令行窗口，跳转到 JAR 文件所在目录，执行以下命令，其中 --spring.config.location 用于指定配置文件的新位置。
```
java -jar springbootdemo-0.0.1-SNAPSHOT.jar  --spring.config.location=D:\myConfig\my-application.yml
```
项目运行结果如下图。

Spring Boot --spring.config.location 指定外部配置文件
图2：使用外部配置运行项目控制台输出

从控制台输出可以看出：
服务器端口号从“8084”被修改为“8088”，表示外部配置文件已生效；
上下文路径则从“/abc”被修改为默认值（‘ ’），表示项目内部的默认配置文件已失效。

4. 使用浏览器访问 “http://localhost:8088/test”,结果如下图。

Spring Boot 指定外部配置文件访问结果
图3：spring.config.location 指定外部配置文件访问结果
spring.config.additional-location
我们还可以在 Spring Boot 启动时，使用命令行参数 --spring.config.additional-location 来加载外部配置文件。
```
java -jar {JAR}  --spring.config.additional-location={外部配置文件全路径}
```
但与 --spring.config.location 不同，--spring.config.additional-location 不会使项目默认的配置文件失效，使用该命令行参数添加的外部配置文件会与项目默认的配置文件共同生效，形成互补配置，且其优先级是最高的，比所有默认配置文件的优先级都高。
示例 2
1. 将 springbootdemo 打包为 JAR 文件，打开命令行窗口，跳转到该项目 JAR 所在目录下，执行以下命令启动该项目。
java -jar springbootdemo-0.0.1-SNAPSHOT.jar  --spring.config.additional-location=D:\myConfig\my-application.yml

结果如下图。


图4：Spring Boot  spring.config.additional-location 指定外部配置文件项目启动结果

注意：Maven 对项目进行打包时，位于项目根目录下的配置文件是无法被打包进项目的 JAR 包的，因此位于根目录下的默认配置文件无法在 JAR 中生效，即该项目将只加载指定的外部配置文件和项目类路径（classpath）下的默认配置文件，它们的加载优先级顺序为：

spring.config.additional-location 指定的外部配置文件 my-application.yml
classpath:/config/application.yml
classpath:/application.yml

根据配置文件优先级分析可知：
以上三个配置文件中 my-application.yml 的优先级最高，因此该项目的服务器端口号为 “8088”；
只有 classpath:/config/application.yml 中配置了上下文路径（context-path），因此该项目的上下文路径为 “/helloworld”；
基于以上配置分析，得出该项目访问路径为“http://localhost:8088/helloWorld”。

# 配置加载顺序
Spring Boot 不仅可以通过配置文件进行配置，还可以通过环境变量、命令行参数等多种形式进行配置。这些配置都可以让开发人员在不修改任何代码的前提下，直接将一套 Spring Boot  应用程序在不同的环境中运行。
## Spring Boot 配置优先级
以下是常用的 Spring Boot 配置形式及其加载顺序（优先级由高到低）：
* 命令行参数
* 来自`java:comp/env`的 JNDI 属性
* Java 系统属性（`System.getProperties()`）
* 操作系统环境变量
* `RandomValuePropertySource`配置的`random.*`属性值
* 配置文件（`YAML`文件、`Properties`文件）
* `@Configuration`注解类上的`@PropertySource`指定的配置文件
* 通过`SpringApplication.setDefaultProperties`指定的默认属性

以上所有形式的配置都会被加载，当存在相同配置内容时，高优先级的配置会覆盖低优先级的配置；存在不同的配置内容时，高优先级和低优先级的配置内容取并集，共同生效，形成互补配置。
### 命令行参数
Spring Boot 中的所有配置，都可以通过命令行参数进行指定，其配置形式如下。
```
java -jar {Jar文件名} --{参数1}={参数值1} --{参数2}={参数值2}
```
1. 在 springbootdemo  项目启动时，使用以下命令。
```
java -jar springbootdemo-0.0.1-SNAPSHOT.jar --server.port=8081 --server.servlet.context-path=/bcb
```
命令行参数说明如下：
* `--server.port`：指定服务器端口号；
* `--server.servlet.context-path`：指定上下文路径（项目的访问路径）。

执行结果如下图。

Spring Boot 指定命令行参数
图1：Spring Boot 指定命令行参数
### 配置文件
Spring Boot 启动时，会自动加载 JAR 包内部及 JAR 包所在目录指定位置的配置文件（Properties 文件、YAML 文件），下图中展示了 Spring Boot 自动加载的配置文件的位置及其加载顺序，同一位置下，Properties 文件优先级高于 YAML 文件。

Spring Boot 配置文件加载顺序

图2：Spring Boot 配置文件加载位置及优先级
 
 
图 2 说明如下：
/myBoot：表示 JAR 包所在目录，目录名称自定义；
/childDir：表示 JAR 包所在目录下 config 目录的子目录，目录名自定义；
JAR：表示 Spring Boot 项目打包生成的 JAR；
其余带有“/”标识的目录的目录名称均不能修改。
红色数字：表示该配置文件的优先级，数字越小优先级越高。

这些配置文件得优先级顺序，遵循以下规则：
* 先加载 JAR 包外的配置文件，再加载 JAR 包内的配置文件；
* 先加载`config`目录内的配置文件，再加载 config 目录外的配置文件；
* 先加载`config`子目录下的配置文件，再加载 config 目录下的配置文件；
* 先加载`appliction-{profile}.properties/yml`，再加载`application.properties/yml`；
* 先加载`.properties`文件，再加载`.yml`文件。

1. 创建一个名为 mybootdemo 的 Spring Boot 项目，并在 src/main/resoources 下创建以下 4 个配置文件。
* applcation.yml：默认配置
* application-dev.yml：开发环境配置
* application-test.yml：测试环境配置
* application-prod.yml：生产环境配置

1）在 applcation.yml 文件中，指定默认服务端口号（port）为 “8080”，上下文路径（context-path）为“/mybootdemo”，并激活开发环境（dev）的 profile。
```
server:
  port: 8080 #端口号
  servlet:
    context-path: /mybootdemo #上下文路径或项目访问路径
spring:
  profiles:
    active: dev #激活开发环境配置
```
2）在 application-dev.yml 中，指定开发环境端口号为 “8081”，上下文路径为“/in-dev”，配置如下。
```
server:
  port: 8081 #开发环境端口号 8081
  servlet:
    context-path: /in-dev #开发环境上下文路径为 in-dev
spring:
  config:
    activate:
      on-profile: dev #开发环境
```
3）在 application-test.yml 中，指定测试环境端口号为 “8082”，上下文路径为“/in-test”，配置如下。
```
#测试环境配置
server:
  port: 8082 #测试环境端口 8082
  servlet:
    context-path: /in-test #测试环境上下文路径 /in-test
spring:
  config:
    activate:
      on-profile: test
```
4）在 application-prod.yml 中，指定生产环境端口号为 “8083”，上下文路径为“/in-prod”，配置如下。
```
#生产环境配置
server:
  port: 8083 #端口号
  servlet:
    context-path: /in-prod #上下文路径
spring:
  config:
    activate:
      on-profile: prod
```
2. 执行以下 mvn 命令，将 mybootdemo 打包成 JAR，并将该 JAR 包移动到本次磁盘的某个目录下（例如 mySpringBoot 目录）。
```
mvn clean package
```

3. 在 JAR 包所在目录下创建 application.yml ，并设置上下文路径为“/out-default”，并激活生产环境（prod）Profile。
```
#JAR 包外默认配置
server:
  servlet:
    context-path: /out-default
#切换配置
spring:
  profiles:
    active: prod #激活开发环境配置
```
4. 打开命令行窗口，跳转到 mySpringBoot 目录下，执行以下命令启动 Spring Boot。
java -jar mybootdemo-0.0.1-SNAPSHOT.jar
启动结果如下图。


图3：Spring Boot 使用 JAR 包外配置启动
示例分析：

Spring Boot 在启动时会加载全部的 5 个配置文件，其中位于 JAR 包外的 application.yml 优先级最高；
在 JAR 包外的 application.yml 中，配置激活了生产环境（prod）Profile，即 JAR 包内部的 application-prod.yml 生效。此时，该项目中的配置文件优先级顺序为：JAR 包外 application.yml > JAR 包内 application-prod.yml >JAR 包内其他配置文件;
application-prod.yml 的配置内容会覆盖 JAR 包内所有其他配置文件的配置内容，即端口号（port）为“8083”，上下文路径（context-path）为“/in-prod”;
JAR 包内的 application-prod.yml 中的上下文路径会被 JAR 包外的 application.yml 覆盖为“/out-default”;
JAR 包内的 application-prod.yml 与 JAR 包外的 application.yml，形成互补配置，即，端口号为“8083”，上下文路径为“/out-default”。

# 自动配置原理
