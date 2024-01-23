

# 概述
Spring Boot 自动配置，顾名思义，是希望能够自动配置，将我们从配置的苦海中解脱出来。那么既然要自动配置，它需要解三个问题：
* 满足什么样的条件？
* 创建哪些 Bean？
* 创建的 Bean 的属性？

我们来举个示例，对照下这三个问题。在我们引入`spring-boot-starter-web`依赖，会创建一个 8080 端口的内嵌 Tomcat，同时可以通过`application.yaml`配置文件中的`server.port`配置项自定义端口。那么这三个问题的答案如下：
* 满足什么样的条件？因为我们引入了`spring-boot-starter-web`依赖。
* 创建哪些`Bean`？创建了一个内嵌的`Tomcat Bean`，并进行启动。
* 创建的`Bean`的属性？通过`application.yaml`配置文件的`server.port`配置项，定义`Tomcat Bean`的启动端口属性，并且默认值为 8080。

我们来看看 Spring Boot 提供的`EmbeddedWebServerFactoryCustomizerAutoConfiguration`类，负责创建内嵌的 Tomcat、Jetty 等等 Web 服务器的配置类。
```java
@Configuration // <1.1>
@ConditionalOnWebApplication // <2.1>
@EnableConfigurationProperties(ServerProperties.class) // <3.1>
public class  EmbeddedWebServerFactoryCustomizerAutoConfiguration {

	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration // <1.2>
	@ConditionalOnClass({ Tomcat.class, UpgradeProtocol.class })
	public static class TomcatWebServerFactoryCustomizerConfiguration {

		@Bean
		public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			// <3.2>
			return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration // <1.3>
	@ConditionalOnClass({ Server.class, Loader.class, WebAppContext.class })
	public static class JettyWebServerFactoryCustomizerConfiguration {

		@Bean
		public JettyWebServerFactoryCustomizer jettyWebServerFactoryCustomizer(
				Environment environment, ServerProperties serverProperties) {
			 // <3.3>
			return new JettyWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	// ... 省略 UndertowWebServerFactoryCustomizerConfiguration 代码

	/**
	 * Nested configuration if Netty is being used.
	 */
	// ... 省略 NettyWebServerFactoryCustomizerConfiguration 代码

}
```
在开始看代码之前，我们先来简单科普下 Spring JavaConfig 的小知识。在 Spring3.0 开始，Spring 提供了`JavaConfig`的方式，允许我们使用 Java 代码的方式，进行 Spring Bean 的创建。示例代码如下：
```java
@Configuration
public class DemoConfiguration {

    @Bean
    public void object() {
        return new Obejct();
    }

}
```
通过在类上添加`@Configuration`注解，声明这是一个 Spring 配置类。

通过在方法上添加`@Bean`注解，声明该方法创建一个 Spring Bean。

现在我们在回过头看看`EmbeddedWebServerFactoryCustomizerAutoConfiguration`的代码，我们分成三块内容来讲，刚好解决我们上面说的三个问题：
* 配置类
* 条件注解
* 配置属性

## 配置类
`<1.1>`处，在类上添加了`@Configuration`注解，声明这是一个配置类。因为它的目的是自动配置，所以类名以`AutoConfiguration`作为后缀。

`<1.2>、<1.3>`处，分别是用于初始化 Tomcat、Jetty 相关`Bean`的配置类。
* `TomcatWebServerFactoryCustomizerConfiguration`配置类，负责创建`TomcatWebServerFactoryCustomizer Bean`，从而初始化内嵌的 Tomcat 并进行启动。
* `JettyWebServerFactoryCustomizer`配置类，负责创建`JettyWebServerFactoryCustomizer Bean`，从而初始化内嵌的 Jetty 并进行启动。

如此，我们可以得到结论一，通过`@Configuration`注解的配置类，可以解决“创建哪些 Bean”的问题。

实际上，Spring Boot 的`spring-boot-autoconfigure`项目，提供了大量框架的自动配置类。
## 条件注解
`<2>`处，在类上添加了`@ConditionalOnWebApplication`条件注解，表示当前配置类需要在当前项目是 Web 项目的条件下，才能生效。在 Spring Boot 项目中，会将项目类型分成 Web 项目（使用 SpringMVC 或者 WebFlux）和非 Web 项目。这样我们就很容易理解，为什么`EmbeddedWebServerFactoryCustomizerAutoConfiguration`配置类会要求在项目类型是 Web 项目，只有 Web 项目才有必要创建内嵌的 Web 服务器呀。

`<2.1>、<2.2>`处，在类上添加了`@ConditionalOnClass`条件注解，表示当前配置类需要在当前项目有指定类的条件下，才能生效。
* `TomcatWebServerFactoryCustomizerConfiguration`配置类，需要有`tomcat-embed-core`依赖提供的`Tomcat、UpgradeProtocol`依赖类，才能创建内嵌的 Tomcat 服务器。
* `JettyWebServerFactoryCustomizer`配置类，需要有`jetty-server`依赖提供的`Server、Loader、WebAppContext`类，才能创建内嵌的 Jetty 服务器。

如此，我们可以得到结论二，通过条件注解，可以解决“满足什么样的条件？”的问题。

实际上，Spring Boot 的`condition`包下，提供了大量的条件注解。
## 配置属性
`<3.1>`处，使用`@EnableConfigurationProperties`注解，让`ServerProperties`配置属性类生效。在 Spring Boot 定义了`@ConfigurationProperties`注解，用于声明配置属性类，将指定前缀的配置项批量注入到该类中。例如`ServerProperties`代码如下：
```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties
		implements EmbeddedServletContainerCustomizer, EnvironmentAware, Ordered {

	/**
	 * Server HTTP port.
	 */
	private Integer port;

	/**
	 * Context path of the application.
	 */
	private String contextPath;
	
	// ... 省略其它属性
	
}
```
通过`@ConfigurationProperties`注解，声明将`server`前缀的配置项，设置到`ServerProperties`配置属性类中。
`<3.2>、<3.3>`处，在创建`TomcatWebServerFactoryCustomizer`和`JettyWebServerFactoryCustomizer`对象时，都会将`ServerProperties`传入其中，作为后续创建的 Web 服务器的配置。也就是说，我们通过修改在配置文件的配置项，就可以自定义 Web 服务器的配置。

如此，我们可以得到结论三，通过配置属性，可以解决“创建的 Bean 的属性？”的问题。

至此，我们已经比较清晰的理解 Spring Boot 是怎么解决我们上面提出的三个问题，但是这样还是无法实现自动配置。例如说，我们引入的`spring-boot-starter-web`等依赖，Spring Boot 是怎么知道要扫码哪些配置类的。
# 自动配置类
在 Spring Boot 的`spring-boot-autoconfigure`项目，提供了大量框架的自动配置，如下图所示：

{% asset_img 1.png %}

在我们通过`SpringApplication#run(Class<?> primarySource, String... args)`方法，启动 Spring Boot 应用的时候，有个非常重要的组件`SpringFactoriesLoader`类，会读取`META-INF`目录下的`spring.factories`文件，获得每个框架定义的需要自动配置的配置类。

我们以`spring-boot-autoconfigure`项目的 Spring Boot `spring.factories`文件来举个例子，如下图所示：

{% asset_img 2.png %}

如此，原先`@Configuration`注解的配置类，就升级成类自动配置类。这样，Spring Boot 在获取到需要自动配置的配置类后，就可以自动创建相应的`Bean`，完成自动配置的功能。

因为`spring-boot-autoconfigure`项目提供的是它选择的主流框架的自动配置，所以其它框架需要自己实现。例如说，Dubbo 通过`dubbo-spring-boot-project`项目，提供 Dubbo 的自动配置。如下图所示：

{% asset_img 3.png %}

# 条件注解
条件注解并不是 Spring Boot 所独有，而是在 Spring3.1 版本时，为了满足不同环境注册不同的`Bean`，引入了`@Profile`注解。示例代码如下：
```java
@Configuration
public class DataSourceConfiguration {

    @Bean
    @Profile("DEV")
    public DataSource devDataSource() {
        // ... 单机 MySQL
    }

    @Bean
    @Profile("PROD")
    public DataSource prodDataSource() {
        // ... 集群 MySQL
    }
    
}
```
* 在测试环境下，我们注册单机 MySQL 的`DataSource Bean`。
* 在生产环境下，我们注册集群 MySQL 的`DataSource Bean`。

在 Spring4 版本时，提供了`@Conditional`注解，用于声明在配置类或者创建`Bean`的方法上，表示需要满足指定条件才能生效。示例代码如下：
```java
@Configuration
public class TestConfiguration {

    @Bean
    @Conditional(XXXCondition.class)
    public Object xxxObject() {
        return new Object();
    }
    
}
```
其中，`XXXCondition`需要我们自己实现`Condition`接口，提供具体的条件实现。

显然，Spring4 提交的`@Conditional`注解非常不方便，需要我们自己去拓展。因此，Spring Boot 进一步增强，提供了常用的条件注解：
* `@ConditionalOnBean`：当容器里有指定`Bean`的条件下
* `@ConditionalOnMissingBean`：当容器里没有指定`Bean`的情况下
* `@ConditionalOnSingleCandidate`：当指定`Bean`在容器中只有一个，或者虽然有多个但是指定首选`Bean`
* `@ConditionalOnClass`：当类路径下有指定类的条件下
* `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
* `@ConditionalOnProperty`：指定的属性是否有指定的值
* `@ConditionalOnResource`：类路径是否有指定的值
* `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
* `@ConditionalOnJava`：基于 Java 版本作为判断条件
* `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
* `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
* `@ConditionalOnWebApplication`：当前项目是 Web 项 目的条件下

# 配置属性
Spring Boot 约定读取`application.yaml、application.properties`等配置文件，从而实现创建`Bean`的自定义属性配置，甚至可以搭配`@ConditionalOnProperty`注解来取消`Bean`的创建。
# 内置 Starter
我们在使用 Spring Boot 时，并不会直接引入`spring-boot-autoconfigure`依赖，而是使用 Spring Boot 内置提供的`Starter`依赖。例如说，我们想要使用 SpringMVC 时，引入的是`spring-boot-starter-web`依赖。这是为什么呢？

因为 Spring Boot 提供的自动配置类，基本都有`@ConditionalOnClass`条件注解，判断我们项目中存在指定的类，才会创建对应的`Bean`。而拥有指定类的前提，一般是需要我们引入对应框架的依赖。

因此，在我们引入`spring-boot-starter-web`依赖时，它会帮我们自动引入相关依赖，从而保证自动配置类能够生效，创建对应的`Bean`。如下图所示：

{% asset_img 4.png %}

Spring Boot 内置了非常多的`Starter`，方便我们引入不同框架，并实现自动配置。如下图所示：

{% asset_img 5.png %}

# 自定义 Starter
在一些场景下，我们需要自己实现自定义`Starter`来达到自动配置的目的。例如说：
* 三方框架并没有提供`Starter`，比如说`Swagger、XXL-JOB`等。
* Spring Boot 内置的`Starter`无法满足自己的需求，比如说`spring-boot-starter-jdbc`不提供多数据源的配置。
* 随着项目越来越大，想要提供适合自己团队的`Starter`来方便配置项目。

`Spring Boot Starter`的命名规则：

| 场景                      | 命名规则                     | 示例 |
| :--: | :--: | :--: |
| Spring Boot 内置 Starter  | spring-boot-starter-{框架}   | spring-boot-starter-web |
| 框架自定义 Starter        | {框架}-spring-boot-starter   | mybatis-spring-boot-starter |
