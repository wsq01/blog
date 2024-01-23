

# 创建 Maven 项目
1. 打开 IDEA，点击菜单`File -> New -> Project...`来创建项目。如下图所示：
{% asset_img 1.jpg %}
2. 选择 Maven 类型，点击`Next`按钮，进入下一步。输入 Maven 的`GroupId、ArtifactId`，如下图所示：
{% asset_img 2.jpg %}
3. 点击`Next`按钮，继续进入下一步。如下图所示：
{% asset_img 3.jpg %}
4. 点击`Finish`按钮，完成 Maven 项目的创建。此时项目如下图所示：
{% asset_img 4.jpg %}

最终，我们的示例项目会如下图所示：

{% asset_img 5.jpg %}

# 引入依赖
在`pom.xml`文件中，引入相关依赖。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.iocoder</groupId>
    <artifactId>demo01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 从 Spring Boot 继承默认配置 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencies>
        <!-- 实现对 SpringMVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```
引入`spring-boot-starter-parent`作为父 POM，从而继承其默认配置。

引入`spring-boot-starter-web`依赖，实现对 SpringMVC 的自动化配置。同时该依赖会自动帮我们引入 SpringMVC 等相关依赖。
# 配置文件
在 Spring Boot 项目中，约定通过`application.yaml`配置文件，进行 Spring Boot 自动配置的`Bean`的自定义。

在`resource`目录下，创建`application.yaml`配置文件。内容如下：
```yaml
server:
  port: 8080 # 内嵌的 Tomcat 端口号。默认值为 8080。
```
通过`server.port`配置项，设置稍后启动的内嵌 Tomcat 端口为`8080`端口。
# DemoController
创建`DemoController`类，提供一个简单的 HTTP API。
```java
@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/echo")
    public String echo() {
        return "echo";
    }

}
```
# Application
创建`Application`类，提供 Spring Boot 应用的启动类。
```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
在类上，添加`@SpringBootApplication`注解，声明是一个 Spring Boot 应用。通过该注解，可以带来 Spring Boot 自动配置等等功能。

在`main(String[] args)`方法中，我们通过`SpringApplication#run(Class<?> primarySource, String... args)`方法，启动 Spring Boot 应用。
# 简单测试
执行`Application#main(String[] args)`方法，启动示例项目。

这里我们会发现，我们无需在部署 Web 项目到外部的 Tomcat 中，而是直接通过`Application#main(String[] args)`方法，就可以直接启动，非常方便。

此时，我们可以看到 IDEA 控制台输出 Spring Boot 启动日志如下：
```bash
// Spring Boot 自带 Banner
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.2.RELEASE)

// 启动 Java 进程使用的 PID 进程号
2020-02-08 15:38:25.724  INFO 10645 --- [           main] cn.iocoder.demo01.Application            : Starting Application on MacBook-Pro-8 with PID 10645 (/Users/yunai/Java/demo01/target/classes started by yunai in /Users/yunai/Java/demo01)
// Spring Boot Profile 机制，暂时可以忽略
2020-02-08 15:38:25.727  INFO 10645 --- [           main] cn.iocoder.demo01.Application            : No active profile set, falling back to default profiles: default
// 内嵌 Tomcat 启动
2020-02-08 15:38:26.503  INFO 10645 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-02-08 15:38:26.510  INFO 10645 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-02-08 15:38:26.510  INFO 10645 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.29]
2020-02-08 15:38:26.561  INFO 10645 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-02-08 15:38:26.561  INFO 10645 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 799 ms
2020-02-08 15:38:26.693  INFO 10645 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-02-08 15:38:26.839  INFO 10645 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-02-08 15:38:26.842  INFO 10645 --- [           main] cn.iocoder.demo01.Application            : Started Application in 1.396 seconds (JVM running for 1.955)
// SpringMVC DispatcherServlet 初始化
2020-02-08 15:38:44.992  INFO 10645 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2020-02-08 15:38:44.992  INFO 10645 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2020-02-08 15:38:44.996  INFO 10645 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 4 ms
// ... 暂时可以忽略
2020-02-08 15:39:37.113  INFO 10645 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
```
使用浏览器，访问`http://127.0.0.1:8080/demo/echo`接口，返回结果为`echo`。

这说明，SpringMVC 框架，也被 Spring Boot 自动配置完成。同时，使用的是内嵌的 Tomcat 服务器。
# IDEA x Spring Initializr
IDEA 内置了 Spring Boot 插件，提供了对 Spring Initializr 集成。

下面，我们来使用该插件，创建一个 Spring Boot 项目。
1. 打开 IDEA，点击菜单`File -> New -> Project...`来创建项目。
{% asset_img 6.jpg %}
1. 选择 Spring Initializr 类型，点击`Next`按钮，进入下一步。输入 Maven 的`GroupId、ArtifactId`：
{% asset_img 7.jpg %}
1. 点击`Next`按钮，选择需要的依赖，这里暂时我们只需要 Web 依赖。
{% asset_img 8.jpg %}
1. 点击`Next`按钮，之后点击`Finish`按钮，完成 Maven 项目的创建。此时项目如下图所示：
{% asset_img 9.jpg %}