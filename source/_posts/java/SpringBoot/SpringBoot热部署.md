

# 概述
在日常开发中，我们需要经常修改 Java 代码，手动重启项目，查看修改后的效果。如果在项目小时，重启速度比较快，等待的时间是较短的。但是随着项目逐渐变大，重启的速度变慢，等待时间 1-2 min 是比较常见的。

这样就导致我们开发效率降低，那么是否有方式能够实现，在我们修改完 Java 代码之后，能够不重启项目呢？

答案是有的，通过热部署的方式。并且实现的方式还是非常多。
# spring-boot-devtools
`spring-boot-devtools`是 Spring Boot 提供的开发者工具，它会监控当前应用所在的`classpath`下的文件发生变化，进行自动重启。

注意，`spring-boot-devtools`并没有采用热部署的方式，而是一种较快的重启方式。

在项目中，我们需要在`pom.xml`中，引入`spring-boot-devtools`依赖如下：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 可选 -->
</dependency>
```
## 演示
下面，我们来演示下`spring-boot-devtools`的使用。

Run 或者 Debug 运行 Spring Boot 应用。

使用浏览器，访问`http://127.0.0.1:8080/demo/echo`接口，返回结果为`echo`。

修改`DemoController`的`echo()`方法，设置返回值为`none`。

我们现在仅仅需要修改了 Java 代码，需要重新编译下代码。点击 IDEA 的菜单`Build -> Build Project`，手动进行编译。此时，IDEA 控制台会看到 Spring Boot 重新启动的日志如下：
```
2020-02-09 09:22:52.082  INFO 36495 --- [      Thread-10] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.4.RELEASE)

2020-02-09 09:22:52.195  INFO 36495 --- [  restartedMain] cn.iocoder.demo03.Demo03Application      : Starting Demo03Application on MacBook-Pro-8 with PID 36495 (/Users/yunai/Downloads/demo03/target/classes started by yunai in /Users/yunai/Downloads/demo03)
2020-02-09 09:22:52.195  INFO 36495 --- [  restartedMain] cn.iocoder.demo03.Demo03Application      : No active profile set, falling back to default profiles: default
2020-02-09 09:22:52.335  INFO 36495 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-02-09 09:22:52.336  INFO 36495 --- [  restartedMain] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-02-09 09:22:52.336  INFO 36495 --- [  restartedMain] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.30]
2020-02-09 09:22:52.342  INFO 36495 --- [  restartedMain] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-02-09 09:22:52.342  INFO 36495 --- [  restartedMain] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 145 ms
2020-02-09 09:22:52.382  INFO 36495 --- [  restartedMain] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-02-09 09:22:52.409  INFO 36495 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2020-02-09 09:22:52.418  INFO 36495 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-02-09 09:22:52.419  INFO 36495 --- [  restartedMain] cn.iocoder.demo03.Demo03Application      : Started Demo03Application in 0.244 seconds (JVM running for 169.162)
2020-02-09 09:22:52.420  INFO 36495 --- [  restartedMain] .ConditionEvaluationDeltaLoggingListener : Condition evaluation unchanged
```
如果觉得手动 Build Project 有点麻烦，IDEA 还提供的自动编译的选项。设置方式，点击 IDEA 的菜单`IntelliJ IDEA -> Preference...`，然后选择`Compiler`选项卡，将`Build project automatically`勾选上。如下图所示：

{% asset_img 1.png %}

注意，`Build project automatically`后面的一行提示，自动编译仅在项目不处于运行，或者处于 Debug 运行中时，才会自动生效。

所以一定要 Debug 运行 Spring Boot 项目。

因为`spring-boot-devtools`提供的本质是重启的方式，所以还是会存在开头所提到的问题。不过 IDEA 自带了热部署的方式。
# IDEA 热部署
IDEA 提供了`HotSwap`插件，可以实现真正的热部署。如下图所示：

{% asset_img 2.png %}

## 演示
下面，我们来演示下`HotSwap`插件的使用。

Run 或者 Debug 运行 Spring Boot 应用。

使用浏览器，访问`http://127.0.0.1:8080/demo/echo`接口，返回结果为`echo`。

修改`DemoController`的`echo()`方法，设置返回值为`none`。

我们现在仅仅需要修改了 Java 代码，需要重新编译下代码。点击 IDEA 的菜单`Build -> Build Project`，手动进行编译。此时，我们在 IDEA 中可以看修改的类被重载的提示。如下图所示：

{% asset_img 3.png %}