

# Spring Boot 的特点
Spring Boot 具有以下特点：
1. 独立运行的 Spring 项目
Spring Boot 可以以 jar 包的形式独立运行，Spring Boot 项目只需通过命令`java–jar xx.jar`即可运行。
2. 内嵌 Servlet 容器
Spring Boot 使用嵌入式的 Servlet 容器（例如 Tomcat、Jetty等），应用无需打成 WAR 包。
3. 提供 starter 简化 Maven 配置
Spring Boot 提供了一系列的“starter”项目对象模型（POMS）来简化 Maven 配置。
4. 提供了大量的自动配置
Spring Boot 提供了大量的默认自动配置，来简化项目的开发，开发人员也通过配置文件修改默认配置。
5. 自带应用监控
Spring Boot 可以对正在运行的项目提供监控。
6. 无代码生成和 xml 配置
Spring Boot 不需要任何 xml 配置即可实现 Spring 的所有配置。

# @SpringBootApplication
`@SpringBootApplication`是一个“三体”结构，实际上它是一个复合`Annotation`：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Configuration
@EnableAutoConfiguration
@ComponentScanpublic
@interface
SpringBootApplication{...}
```
虽然它的定义使用了多个`Annotation`进行元信息标注，但实际上对于 SpringBoot 应用来说，重要的只有三个`Annotation`，而“三体”结构实际上指的就是这三个`Annotation`：
```
@Configuration
@EnableAutoConfiguration
@ComponentScan
```
所以，如果我们使用如下的 SpringBoot 启动类，整个 SpringBoot 应用依然可以与之前的启动类功能对等：
```java
@Configuration
@EnableAutoConfiguration
@ComponentScanpublic
class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```
但每次都写三个`Annotation`显然过于繁琐，所以写一个`@SpringBootApplication`这样的一站式复合`Annotation`显然更方便些。
## @Configuration
这里的`@Configuration`是 JavaConfig 形式的 Spring IoC 容器的配置类使用的那个`@Configuration`，既然 SpringBoot 应用就是一个 Spring 应用，那么，自然也需要加载某个 IoC 容器的配置，而 SpringBoot 社区推荐使用基于 JavaConfig 的配置形式，所以，很明显，这里的启动类标注了`@Configuration`之后，本身其实也是一个 IoC 容器的配置类！

如果我们将上面的 SpringBoot 启动类拆分为两个独立的 Java 类，整个形势就明朗了：
```java
@Configuration
@EnableAutoConfiguration
@ComponentScan
public class DemoConfiguration {
  @Bean
  public Controller controller() {
    return new Controller();
  }
}

public class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoConfiguration.class, args);
  }
}
```
所以，启动类`DemoApplication`其实就是一个标准的`Standalone`类型 Java 程序的`main`函数启动类，没有什么特殊的。而`@Configuration`标注的`DemoConfiguration`定义其实也是一个普通的 JavaConfig 形式的 IoC 容器配置类。
## @EnableAutoConfiguration
`@EnableAutoConfiguration`借助`@Import`，将所有符合自动配置条件的`bean`定义加载到 IoC 容器。

`@EnableAutoConfiguration`作为一个复合`Annotation`，其自身定义关键信息如下：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(EnableAutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {...}
```
其中，最关键的要属`@Import（EnableAutoConfigurationImportSelector.class）`，借助`EnableAutoConfigurationImportSelector`，`@EnableAutoConfiguration`可以帮助 SpringBoot 应用将所有符合条件的`@Configuration`配置都加载到当前 SpringBoot 创建并使用的 IoC 容器。

{% asset_img 1.png %}

借助于 Spring 框架原有的一个工具类：`SpringFactoriesLoader`的支持，`@EnableAutoConfiguration`可以“智能”地自动配置功效才得以大功告成！
## SpringFactoriesLoader
`SpringFactoriesLoader`属于 Spring 框架私有的一种扩展方案（类似于 Java 的 SPI 方案`java.util.ServiceLoader`），其主要功能就是从指定的配置文件`META-INF/spring.factories`加载配置，`spring.factories`是一个典型的 java `properties`文件，配置的格式为`Key=Value`形式，只不过`Key`和`Value`都是 Java 类型的完整类名，比如：
```java
example.MyService=example.MyServiceImpl1,example.MyServiceImpl2 然后框架就可以根据某个类型作为 Key 来查找对应的类型名称列表了：
public abstract class SpringFactoriesLoader {
  // ...
  public static <T> List<T> loadFactories(Class<T> factoryClass, ClassLoader classLoader) {
    ...
  }
  public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
    ...
  }
  // ...
}
```
对于`@EnableAutoConfiguration`来说，`SpringFactoriesLoader`的用途稍微不同一些，其本意是为了提供 SPI 扩展的场景，而在`@EnableAutoConfiguration`的场景中，它更多是提供了一种配置查找的功能支持，即根据`@EnableAutoConfiguration`的完整类名`org.springframework.boot.autoconfigure.EnableAutoConfiguration`作为查找的`Key`，获取对应的一组`@Configuration`类：
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=
\org.springframework.boot.autoconfigure.admin.SpringApplicationAdmin- JmxAutoConfiguration,
\org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,
\org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,
\org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,
\org.springframework.boot.autoconfigure.PropertyPlaceholderAuto- Configuration,
\org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,
\org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,
\org.springframework.boot.autoconfigure.cassandra.CassandraAuto-Configuration,
\org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,
\org.springframework.boot.autoconfigure.context.ConfigurationProperties-AutoConfiguration,
\org.springframework.boot.autoconfigure.dao.PersistenceException-TranslationAutoConfiguration,
\org.springframework.boot.autoconfigure.data.cassandra.Cassandra-DataAutoConfiguration,
\org.springframework.boot.autoconfigure.data.cassandra.Cassandra-RepositoriesAutoConfiguration,
\...
```
以上是从 SpringBoot 的`autoconfigure`依赖包中的`META-INF/spring.factories`配置文件中摘录的一段内容，可以很好地说明问题。

所以，`@EnableAutoConfiguration`自动配置的魔法其实就变成了：从`classpath`中搜寻所有`META-INF/spring.factories`配置文件，并将其中`org.spring-framework.boot.autoconfigure.EnableAutoConfiguration`对应的配置项通过反射实例化为对应的标注了`@Configuration`的`JavaConfig`形式的 IoC 容器配置类，然后汇总为一个并加载到 IoC 容器。
## 可有可无的@ComponentScan
为啥说`@ComponentScan`是可有可无的？

因为原则上来说，`@ComponentScan`的功能其实就是自动扫描并加载符合条件的组件或`bean`定义，最终将这些`bean`定义加载到容器中。加载`bean`定义到 Spring 的 IoC 容器，我们可以手工单个注册，不一定非要通过批量的自动扫描完成，所以说`@ComponentScan`是可有可无的。

对于 SpringBoot 应用来说，同样如此。
```java
@Configuration
@EnableAutoConfiguration
@ComponentScanpublic
class DemoApplication {
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```
如果我们当前应用没有任何`bean`定义需要通过`@ComponentScan`加载到当前 SpringBoot 应用对应使用的 IoC 容器，那么，除去`@ComponentScan`的声明，当前 SpringBoot 应用依然可以照常运行，功能对等。