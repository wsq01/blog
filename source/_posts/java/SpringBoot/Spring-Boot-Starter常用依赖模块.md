


SpringBoot 提供的依赖模块都约定以`spring-boot-starter-`作为命名的前缀，并且皆位于`org.springframework.boot`包或者命名空间下。

所有的`spring-boot-starter`都有约定俗成的默认配置，但允许我们调整这些配置以改变默认的配置行为，即“约定优先于配置”。
# 应用日志和spring-boot-starter-logging
Java 的日志系统多种多样，从`java.util`默认提供的日志支持，到`log4j，log4j2，commons logging`等，复杂繁多，所以，应用日志系统的配置就会比较特殊，从而`spring-boot-starter-logging`也比较特殊一些。

假如 maven 依赖中添加了`spring-boot-starter-logging`：
```xml
<dependency>
    <groupId> org.springframework.boot </groupId>
    <artifactId> spring-boot-starter-logging </artifactId>
</dependency>
```
那么，我们的 SpringBoot 应用将自动使用 logback 作为应用日志框架，SpringBoot 启动的时候，由 org.springframework.boot.logging.Logging-Application-Listener 根据情况初始化并使用。

SpringBoot 为我们提供了很多默认的日志配置，所以，只要将 spring-boot-starter-logging 作为依赖加入到当前应用的 classpath，则“开箱即用”，不需要做任何多余的配置，但假设我们要对默认 SpringBoot 提供的应用日志设定做调整，则可以通过几种方式进行配置调整：
遵循 logback 的约定，在 classpath 中使用自己定制的 logback.xml 配置文件。
在文件系统中任何一个位置提供自己的 logback.xml 配置文件，然后通过 logging.config 配置项指向这个配置文件来启用它，比如在 application.properties 中指定如下的配置。
```
logging.config=/{some.path.you.defined}/any-logfile-name-I-like.log
```
SpringBoot 默认允许我们通过在配置文件或者命令行等方式使用 logging.file 和 logging.path 来自定义日志文件的名称和存放路径，不过，这只是允许我们在 SpringBoot 框架预先定义的默认日志系统设定的基础上做有限的设置，如果我们希望更灵活的配置，最好通过框架特定的配置方式提供相应的配置文件，然后通过 logging.config 来启用。

如果大家更习惯使用 log4j 或者 log4j2，那么也可以采用类似的方式将它们对应的`spring-boot-starter`依赖模块加到 Maven 依赖中即可：
```xml
<dependency>
    <groupId> org.springframework.boot </groupId>
    <artifactId> spring-boot-starter-log4j </artifactId>
</dependency>
<!-- 或者 -->
<dependency>
    <groupId> org.springframework.boot </groupId>
    <artifactId> spring-boot-starter-log4j2 </artifactId>
</dependency>
```
但一定不要将这些完成同一目的的 spring-boot-starter 都加到依赖中。
# 快速 Web 应用开发与 spring-boot-starter-web
为了帮我们简化快速搭建并开发一个 Web 项目，SpringBoot 提供了`spring-boot-starter-web`自动配置模块。

只要将`spring-boot-starter-web`加入项目的 maven 依赖：
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
我们就得到了一个直接可执行的 Web 应用，当前项目下运行`mvn spring-boot：run`就可以直接启动一个使用了嵌入式 tomcat 服务请求的 Web 应用，只不过，我们还没有提供任何服务 Web 请求的`Controller`，所以，访问任何路径都会返回一个 SpringBoot 默认提供的错误页面（一般称其为 whitelabel error page），我们可以在当前项目下新建一个服务根路径 Web 请求的 Controller 实现：
```java
@RestController
public class IndexController {
  @RequestMapping("/")
  public String index() {
    return "hello, there";
  }
}
```
重新运行 mvn spring-boot：run 并访问`http://localhost：8080`，错误页面将被我们的`Controller`返回的消息所替代，一个简单的 Web 应用就这样完成了。
## 项目结构层面的约定
项目结构层面与传统打包为 war 的 Java Web 应用的差异在于，静态文件和页面模板的存放位置变了，原来是放在`src/main/webapp `目录下的一系列资源，现在都统一放在`src/main/resources`相应子目录下，比如：
* `src/main/resources/static`用于存放各类静态资源，比如`css，js`等。
* `src/main/resources/templates`用于存放模板文件，比如`*.vm`。

当然，如果还是希望以 war 包的形式，而不是 SpringBoot 推荐使用的独立 jar 包形式发布 Web 应用，也可以继续原来 Java Web 应用的项目结构约定。
## SpringMVC 框架层面的约定和定制
`spring-boot-starter-web`默认将为我们自动配置如下一些 SpringMVC 必要组件：
* 必要的`ViewResolver`，比如`ContentNegotiatingViewResolver`和`Bean-NameViewResolver`。
* 将必要的`Converter、GenericConverter`和`Formatter`等`bean`注册到 IoC 容器。
* 添加一系列的`HttpMessageConverter`以便支持对 Web 请求和相应的类型转换。
* 自动配置和注册`MessageCodesResolver`。
* 其他。

任何时候，如果我们对默认提供的 SpringMVC 组件设定不满意，都可以在 IoC 容器中注册新的同类型的`bean`定义来替换，或者直接提供一个基于`WebMvcConfigurerAdapter`类型的`bean`定义来定制，甚至直接提供一个标注了`@EnableWebMvc`的`@Configuration`配置类完全接管所有 SpringMVC 的相关配置，自己完全重新配置。
# spring-boot-starter-jdbc与数据访问
若想 SpringBoot 为我们自动配置数据访问的基础设施，那么，我们需要直接或者间接地依赖 spring-jdbc，一旦 spring-jdbc 位于我们 SpringBoot 应用的 classpath，即会触发数据访问相关的自动配置行为，最简单的做法就是把 spring-boot-starter-jdbc 加为应用的依赖。

默认情况下，如果我们没有配置任何 DataSource，那么，SpringBoot 会为我们自动配置一个基于嵌入式数据库的 DataSource，这种自动配置行为其实很适合于测试场景，但对实际的开发帮助不大，基本上我们会自己配置一个 DataSource 实例，或者通过自动配置模块提供的配置参数对 DataSource 实例进行自定义的配置。

假设我们的 SpringBoot 应用只依赖一个数据库，那么，使用`DataSource`自动配置模块提供的配置参数是最方便的：
```
spring.datasource.url=jdbc:mysql://{database host}:3306/{databaseName}
spring.datasource.username={database username}
spring.datasource.password={database password}
```
当然，自己配置一个 DataSource 也是可以的，SpringBoot 也会智能地选择我们自己配置的这个 DataSource 实例（只不过必要性真不大）。

除了 DataSource 会自动配置，SpringBoot 还会自动配置相应的 JdbcTemplate、DataSourceTransactionManager 等关联“设施”，可谓服务周到，我们只要在使用的地方注入就可以了：
```java
class SomeDao {
  @Autowired
  JdbcTemplate jdbcTemplate;
  public <T> List<T> queryForList(String sql){
    // ...
  }
  // ...
}
```
不过，spring-boot-starter-jdbc 以及与其相关的自动配置也不总是带来便利，在某些场景下，我们可能会在一个应用中需要依赖和访问多个数据库，这个时候就会出现问题了。

假设我们在 ApplicationContext 中配置了多个 DataSource 实例指向多个数据库：
```java
@Bean
public DataSource dataSource1() throws Throwable {
  DruidDataSource dataSource = new DruidDataSource();
  dataSource.setUrl(...);
  dataSource.setUsername(...);
  dataSource.setPassword(...);
  // TODO other settings if necessary in the future.
  return dataSource;
}
@Bean
public DataSource dataSource2() throws Throwable {
  DruidDataSource dataSource = new DruidDataSource();
  dataSource.setUrl(...);
  dataSource.setUsername(...);
  dataSource.setPassword(...);
  // TODO other settings if necessary in the future.
  return dataSource;
}
```
那么，启动 SpringBoot 应用的时候会抛出类似如下的异常（Exception）：
```
Exception）：No qualifying bean of type [javax.sql.DataSource] is defined: expected single matching bean but found 2 
```
为了避免这种情况的发生，我们需要在 SpringBoot 的启动类上做点儿“手脚”：
```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class,
    DataSourceTransactionManagerAutoConfiguration.class })
public class UnveilSpringChapter3Application {
  public static void main(String[] args) {
    SpringApplication.run(UnveilSpringChapter3Application.class, args);
  }
}
```
也就是说，我们需要在这种场景下排除掉对 SpringBoot 默认提供的`DataSource`相关的自动配置。但如果我们还是想要享受 SpringBoot 提供的自动配置`DataSource`的机能，也可以通过为其中一个`DataSource`配置添加`org.springframework.context.annotation.Primary`这个`Annotation`的方式以实现两全其美：
```java
@Bean
@Primary
public DataSource dataSource1() throws Throwable {
  DruidDataSource dataSource = new DruidDataSource();
  dataSource.setUrl(...);
  dataSource.setUsername(...);
  dataSource.setPassword(...);
  // TODO other settings if necessary in the future.
  return dataSource;
}
@Bean
public DataSource dataSource2() throws Throwable {
  DruidDataSource dataSource = new DruidDataSource();
  dataSource.setUrl(...);
  dataSource.setUsername(...);
  dataSource.setPassword(...);
  // TODO other settings if necessary in the future.
  return dataSource;
}
```
另外，SpringBoot 还提供了很多其他数据访问相关的自动配置模块，比如`spring-boot-starter-data-jpa、spring-boot-starter-data-mongodb`等。

如果选择了`spring-boot-starter-data-jpa`等关系数据库相关的数据访问自动配置模块，并且还需要同时依赖访问多个数据库，那么，也需要相应的在 SpringBoot 启动类中排除掉这些自动配置模块中的`AutoConfiguration`实现类（对应`spring-boot-starter-data-jpa`是`JpaRepositoriesAutoConfiguration`），或者标注某个`DataSource`为`@Primary`。
# spring-boot-starter-aop
现在 Spring 框架提供的 AOP 方案倡导了一种各取所长的方案，即使用 SpringAOP 的面向对象的方式来编写和组织织入逻辑，并使用 AspectJ 的 Pointcut 描述语言配合 Annotation 来标注和指明织入点（Jointpoint）。

SpringBoot 为我们提供了一个`spring-boot-starter-aop`自动配置模块。

spring-boot-starter-aop 自动配置行为由两部分内容组成：
位于 spring-boot-autoconfigure的org.springframework.boot.autoconfigure.aop.AopAutoConfiguration 提供 @Configuration 配置类和相应的配置项。
spring-boot-starter-aop 模块自身提供了针对 spring-aop、aspectjrt 和 aspectjweaver 的依赖。

一般情况下，只要项目依赖中加入了 spring-boot-starter-aop，其实就会自动触发 AOP 的关联行为，包括构建相应的 AutoProxyCreator，将横切关注点织入（Weave）相应的目标对象等，不过 AopAutoConfiguration 依然为我们提供了可怜的两个配置项，用来有限地干预 AOP 相关配置：
spring.aop.auto=true
spring.aop.proxy-target-class=false

对我们来说，这两个配置项的最大意义在于：允许我们投反对票，比如可以选择关闭自动的 aop 配置（spring.aop.auto=false），或者启用针对 class 而不是 interface 级别的 aop 代理（aop proxy）。

AOP 的应用场景很多，我们不妨以当下最热门的 APM（Application Performance Monitoring）为实例场景，尝试使用 spring-boot-starter-aop 的支持打造一个应用性能监控的工具原型。
spring-boot-starter-aop 在构建 spring-boot-starter-metrics 自定义模块中的应用
对于应用性能监控来说，架构逻辑上其实很简单，基本上就是三步走（如图 1 所示）。

本节暂时只构建一个 spring-boot-starter-metrics 自定义的自动配置模块用来解决“应用性能数据采集”的问题。
应用性能监控关键环节示意图
图 1  应用性能监控关键环节示意图

在此之前，有几个原则我们需要先说明一下：

虽然说采集应用性能数据可以帮助我们更好地分析和改进应用的性能指标，但这不意味着可以借着 APM 的名义对应用的核心职能形成侵害，加上应用性能数据采集功能一定会对应用的性能本身带来拖累，你拿到的所谓性能数据是分摊了你的数据采集方案带来的负担，所以，一般情况下，最好把应用性能数据采集模块的性能损耗控制在 10% 以内甚至更小。

SpringAOP 其实提供了多种横切逻辑织入机制（Weaving），性能损耗上也是各有差别，从运行期间的动态代理和字节码增强 Weavng，到类加载期间的 Weaving，甚至高冷的 AspectJ 二次静态编译 Weaving，大家可以根据情况灵活把握。

针对应用性能数据的采集，最好对应用开发者是透明的，通过配置外部化的形式，可以最大限度地剥离这部分对应用开发者来说非核心的关注点，只在部署和运行之前确定采集点并配置上线即可。

虽然本节实例采用基于 @Annotation 的方式来标注性能采集点，但不意味着这是最优的方式，更多是基于技术方案（SpringAOP）的现状给出的一种实践方式。

下面我们正式着手构建 spring-boot-starter-metrics 自定义的自动配置模块的设计和实现方案。

笔者一向是只在有必要的时候才重新“造轮子”，绝不会为了炫技而去“造轮子”，所以，本次的主角我们选择 Java 中的 Dropwizard Metrics 这个类库作为打造我们 APM 原型的起点。

Dropwizard Metrics 为我们提供了多种不同类型的应用数据度量方案，且通过相应的数据处理算法在性能和批量状态的管理上做了很优秀的工作，只不过，如果我们直接用它的 API 来对自己的应用代码进行度量的话，那写起来代码太多，而且这些性能代码混杂在应用的核心逻辑执行路径上，一个是界面不友好，另外一个就是不容易维护：
```java
public class MockService implements InitializingBean {
  @Autowired
  MetricRegistry metricRegistry;
  private Timer timer;
  private Counter counter;
  // define more other metrics...
  public void doSth() {
    counter.inc();
    Timer.Context context = timer.time();
    try {
      System.out.println("just do something.");
    } finally {
      context.stop();
    }
  }
  @Override
  public void afterPropertiesSet() throws Exception {
    timer = metricRegistry.timer("timerToProfilingDoSthMethod");
    counter = metricRegistry.counter("counterForDoSthMethod");
  }
}
```
所以，对于这些非功能性的性能度量代码，我们可以使用 AOP 的方式剥离到相应的 Aspect 中单独维护，而为了能够将这些性能度量的 Aspect 挂接到指定的待度量代码上，基于现有的方案选型。

可以使用 metrics-annotation 提供的一系列 Annotation 来标注织入位置，这样，开发者只要在需要度量的代码位置上标注相应的 Annotation，我们提供的 spring-boot-starter-metrics 自定义的自动配置模块就会自动地收集这些位置上指定的性能度量数据。

首先，我们通过 http://start.spring.io/ 构建一个 SpringBoot 的脚手架项目，选择以 Maven 编译（选择用 Gradle 的同学自行甄别后面的配置如何具体进行），然后在创建好的 SpringBoot 脚手架项目的 pom.xml 中添加如下必要配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.keevol</groupId>
  <artifactId>spring-boot-starter-metrics</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>spring-boot-starter-metrics</name>
  <description>auto configuration module for dropwizard metrics</description>
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>1.3.0.RELEASE</version>
      <relativePath /> <!-- lookup parent from repository -->
  </parent>
  <properties>
      <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      <java.version>1.8</java.version>
      <metrics.version>3.1.2</metrics.version>
  </properties> <!--其他配置 -->
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
      <groupId>io.dropwizard.metrics</groupId>
      <artifactId>metrics-core</artifactId>
      <version>
        ${metrics.version}
      </version>
    </dependency>
    <dependency>
      <groupId>io.dropwizard.metrics</groupId>
      <artifactId>metrics-annotation</artifactId>
      <version>${metrics.version}</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.8.7</version>
    </dependency>
  </dependencies>
</project>
```
pom.xml 中有几个关键配置需要关注：
继承了 spring-boot-starter-parent，用于加入 springboot 的相关依赖。
添加了 spring-boot-starter-aop 依赖。
添加了 io.dropwizard.metrics 下相应的依赖，用来引入 dropwizard metrics 类库和必要的 Annotations。
添加了 spring-boot-starter-actuator，这个自动配置模块教程后面会跟大家进一步介绍，在这里我们主要是引入它对 dropwizard metrics 和 JMX 的一部分自动配置逻辑，比如针对 MetricRegistry 和 MBeanServer 的自动配置，这样我们就可以直接 @Autowired 来注入使用 MetricRegistry 和 MBeanServer。

至于 aspectjrt，是使用了最新的版本，原则上spring-boot-starter-aop已经有依赖，这里可以不用明确添加配置。

如果单单是一个提供必要依赖的自动配置模块，那么到这里其实就可以结束了，但我们的 spring-boot-starter-metrics 需要使用 AOP 提供相应的横切关注点逻辑。

所以，还需要编写并提供一些必要的代码组件，因此，最少我们先要提供一个 @Configuration 配置类，用于将我们即将提供的这些 AOP 逻辑暴露给使用者：
```java
@Configuration
@ComponentScan({ "com.keevol.springboot.metrics.lifecycle",
    "com.keevol.springboot.metrics.aop" })
@AutoConfigureAfter(AopAutoConfiguration.class)
public class DropwizardMetricsMBeansAutoConfiguration {
  @Value("${metrics.mbeans.domain.name:com.keevol.metrics}")
  String metricsMBeansDomainName;
  @Autowired
  MBeanServer mbeanServer;
  @Autowired
  MetricRegistry metricRegistry;
  @Bean
  public JmxReporter jmxReporter() {
    JmxReporter reporter = JmxReporte.forRegistry(metricRegistry)
        .inDomain(metricsMBeansDomainName).registerWith(mbeanServer)
        .build();
    return reporter;
  }
}
```
然后就是将这个配置类添加到 META-INF/spring.factories：

org.springframework.boot.autoconfigure.EnableAutoConfiguration=\com.keevol.springboot.metrics.autocfg.DropwizardMetricsMBeansAuto-ConfigurationOK， 
不要认为将 spring-boot-starter-metrics 打包作为类库发布出去就可以了，AOP 相关的代码还没写。

我们回头来看 DropwizardMetricsMBeansAutoConfiguration 配置类，这个配置类的实现很简单，注入了 MBeanServer 和 MetricRegistry 的实例，并开放了一个 metrics.mbeans.domain.name 配置属性（默认值 com.keevol.metrics）便于使用者指定自定义的 MBean 暴露和访问的命名空间。

当然，以上给这些其实都不是重点，因为它们都只是为了将我们要采集的性能数据指标以 JMX 的形式暴露出去而服务的，重点在于 DropwizardMetricsMBeansAutoConfiguration 头顶上的那几顶“帽子”：
@Configuration 自然不必说了，这是一个 JavaConfig 配置类。
@ComponentScan（{"com.keevol.springboot.metrics.lifecycle"，"com.keevol.springboot.metrics.aop"}），为了简便，让 @ComponentScan 把这两个 java package 下的所有组件都加载到 IoC 容器中，这些组件就包括我们要提供的一系列与 AOP 和 Dropwizard Metrics 相关的实现逻辑。
@AutoConfigureAfter（AopAutoConfiguration.class）告诉 SpringBoot：“我希望 DropwizardMetricsMBeansAutoConfiguration 在 AopAutoConfiguration 完成之后进行配置”。

现在，最后的秘密就隐藏在 @ComponentScan 背后的两个 java package 之下了。

首先是 com.keevol.springboot.metrics.aop，在这个 java package 下面，我们只提供了一个 AutoMetricsAspect，其定义如下：
```java
@Component
@Aspectpublic
class AutoMetricsAspect {
  protected ConcurrentMap<String, Meter> meters = new ConcurrentHashMap<>();
  protected ConcurrentMap<String, Meter> exceptionMeters = new ConcurrentHashMap<>();
  protected ConcurrentMap<String, Timer> timers = new ConcurrentHashMap<>();
  protected ConcurrentMap<String, Counter> counters = new ConcurrentHashMap<>();
  @Autowired
  MetricRegistry metricRegistry;
  @Pointcut(value = "execution(public * *(..))")
  public void publicMethods() {
  }
  @Before("publicMethods() && @annotation(countedAnnotation)")
  public void instrumentCounted(JoinPoint jp, Counted countedAnnotation) {
      String name = name(jp.getTarget().getClass(), StringUtils.hasLength(countedAnnotation.name()) ? countedAnnotation.name() : jp.getSignature().getName(), "counter");
      Counter counter = counters.computeIfAbsent(name, key -> metricRegistry.counter(key));
      counter.inc();
  }   
  
  @Before("publicMethods() && @annotation(meteredAnnotation)")
  public void instrumentMetered(JoinPoint jp, Metered meteredAnnotation) {
      String name = name(jp.getTarget().getClass(), StringUtils.hasLength(meteredAnnotation.name()) ? meteredAnnotation.name() : jp.getSignature().getName(), "meter");
      Meter meter = meters.computeIfAbsent(name, key -> metricRegistry.meter(key));
      meter.mark();
  }   
  @AfterThrowing(pointcut = "publicMethods() && @annotation(exMe-teredAnnotation)", throwing = "ex")
  public void instrumentExceptionMetered(JoinPoint jp, Throwable ex, ExceptionMetered exMeteredAnnotation) {
      String name = name(jp.getTarget().getClass(), StringUtils.hasLength(exMeteredAnnotation.name()) ? exMeteredAnnotation.name() : jp.getSignature().getName(), "meter", "exception");
      Meter meter = exceptionMeters.computeIfAbsent(name, meterName -> metricRegistry.meter(meterName));
      meter.mark();
  }   
  @Around("publicMethods() && @annotation(timedAnnotation)")
  public Object instrumentTimed(ProceedingJoinPoint pjp, Timed timedAnnotation) throws Throwable {
    String name = name(pjp.getTarget().getClass(), StringUtils.hasLength(timedAnnotation.name()) ? timedAnnotation.name() : pjp.getSignature().getName(), "timer");
    Timer timer = timers.computeIfAbsent(name, inputName -> metricRegistry.timer(inputName));
    Timer.Context tc = timer.time();
    try {
      return pjp.proceed();
    } finally {
      tc.stop();
    }
  }
}
```
@Aspect+@Component 的目的在于告诉 Spring 框架：“我是一个 AOP 的 Aspect 实现类并且你可以通过 @ComponentScan 把我加入 IoC 容器之中。”当然，这不是重点。

io.dropwizard.metrics：metrics-annotation 这个依赖包为我们提供了几个有趣的 Annotation：
```
Timed
Gauge
Counted
Metered
ExceptionMetered
```
这些语义良好的 Annotation 定义可以用来标注相应的 AOP 逻辑扩展点，比如，针对同一个 MockService，我们可以将性能数据的度量和采集简化为只标注一两个 Annotation 就可以了：
```java
@Component
public class MockService {
  @Timed
  @Counted
  public void doSth() {
    System.out.println("just do something.");
  }
}
```
但是，Annotation 注定只是 Annotation，它们只是一些标记信息，要让它们发挥作用，需要有“伯乐”的眷顾，所以，AutoMetricsAspect 在这里就是这些 Dropwizard Metrics Annotation 的“伯乐”。

通过拦截每一个 public 方法并检查方法上是否存在某个 metrics annotation，我们就可以根据具体的 metrics annotation 的类型，为匹配的方法注入相应性能数据采集代码逻辑，从而完成整个基于 AOP 和 dropwizard metrics 的应用性能数据采集方案的实现。

受限于 SpringAOP 自身的一些限制，并不是所有 AOP 的 Joinpoint 类型都支持，而且，以上原型代码方向也不见得是性能最优的方案，大家需要结合自己的目标和手上可用的技术手段，根据自己的具体应用场景具体分析和权衡。
# spring-boot-starter-security与应用安全
spring-boot-starter-security 主要面向 Web 应用安全，配合 spring-boot-starter-web。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.keevol.unveilspring.chapter3</groupId>
    <artifactId>web-security-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>web-security-demo</name>
    <description>web security demo project for Spring Boot</description>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.3.1.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency> <!--其他依赖 -->
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
在当前项目中只要添加需要的 Controller 实现，一个添加了基本安全防护的 Web 应用就诞生了。spring-boot-starter-security 默认会提供一个基于 HTTP Basic 认证的安全防护策略，默认用户名为 user，访问密码则在当前 Web 应用启动的时候，打印到控制台，类似于：
2017-01-01 13:57:00.596 INFO 17966 --- [ost-startStop-1] b.a.s.Au-thenticationManagerConfiguration : Using default security password: 560ff91b-0ae7-492c-ad16-603e1adec54c 

如果我们希望对 HTTP Basic 认证的用户名和密码进行定制，可以通过如下配置项进行：
```
security.user.name={个人希望设置的用户名}
security.user.password={个人希望使用的访问密码} 
```
除此之外，spring-boot-starter-security 还会默认启用一些必要的 Web 安全防护，比如针对 XSS、CSRF 等常见针对 Web 应用的攻击，同时，也会将一些常见的静态资源路径排除在安全防护之外。

但是，说实话，spring-boot-starter-security 提供的默认安全策略相对于真正的生产环境来说，还是太弱了。但也没办法，既要安全，又要便利，spring-boot-starter-security 默认情况下已经尽量做到够好了。

不过好在 SpringSecurity 扩展性不错，要在其上构建一套真正严谨有效的 Web 应用安全防护体系也并非难事，只不过，需要我们先能够从其架构设计上理解并把握它，然后再在 SpringSecurity 和 SpringBoot 的基础上构建一套符合自身需要的 Web 应用安全方案。
## 了解 SpringSecurity 基本设计
SpringSecurity 框架不但囊括了基本的认证和授权功能，而且还提供了加密解密、统一登录等一系列相关支持。

我们可以将 Spring Security 的几个核心概念按照图 1 所示勾勒在一起：
SpringSecurity核心概念示意图
图 1  SpringSecurity 核心概念示意图

访问者（Accessor）需要访问某个资源（Resources）是这个场景中最原始的需求，但并不是谁都可以访问资源，也不是任何资源都允许任何人来访问，所以，中间我们要加入一些检查和防护。

在访问资源的所经之路上，可能需要上山、过桥、下海行船，不管怎么样，这些所经之路对于我们要防护的资源来说都是比较好的设置关卡点，对应上图就是 FilterInvocation（对应 Web 应用场景）、MethodInvocation 以及 Joinpoint，这些在 Spring Security 框架中统称 Secured Object（s）。

我们知道了在哪里设置关卡最合适，下一步就是设置关卡，对应不同的所经之路，我们分别设置类似 FilterSecurityInterceptor、Method-SecurityInterceptor 以及 AspectJSecurityInterceptor 这样的关卡来负责拦截非法资源访问的闯入者们。

而在 Spring Security 框架的设计中，关卡的概念统一抽象为 AbstractSecurityInterceptor，而 FilterSecurity-Interceptor、MethodSecurityInterceptor 以及 AspectJSecurityInterceptor 都是它的具体实现类。

现在把门儿的倒是有了，可是他们不知道该拦谁，不该拦谁，所以，我们需要有类似神盾局（S.H.I.E.L.D）这样的机构，由这个机构来决定谁可以放行，谁必须阻截，而在 SpringSecurity 框架中 AccessDecisionManager 就是这个控制机构，AccessDecisionManager 将决定谁可以访问哪些资源。

现在剩下最后一个问题，这个谁怎么定义？我们总得知道当前这个访问者是谁才能告知 AccessDecisionManager 阻截还是放行，所以，SpringSecurity 框架中的 AuthenticationManager 将解决的是访问者身份认证的问题，只有确定你在册了，才可以给你授权访问（除非匿名访问某些公共资源）。

AuthenticationManager、AccessDecisionManager 和 AbstractSecurityInterceptor 属于 Spring Security 框架的基础铁三角。AuthenticationManager 和 Access-DecisionManager 负责制定规则，AbstractSecurityInterceptor 负责执行。

所有针对不同应用场景的安全方案，基本上都是在这个基础核心的基础上衍生出来的，比如，Web 安全。

Spring Security 的 Web 安全方案基于 Java 的 Servlet API 规范进行构建，所以，像 Play Framework 这种脱离 Servlet 规范的 Web 框架，则无法享受到 SpringSecurity 提供的默认的 Web 安全方案（当然，依然可以基于基本模型实现扩展方案）。

既然是基于 Servlet API 规范，那么，要实现关卡的“特效”，则非 javax.servlet.Filter 莫属了。在使用 Spring 框架开发 Filter 的时候，为了让 Filter 可以享受到依赖注入的好处，我们一般是实现 GenericFilterBean 并注册到 IoC 容器。

为了能够启用这些注册到 IoC 容器的 Filter，我们一般要在 web.xml 或者相应的 JavaConfig 的配置中声明一个 org.springframework.web.filter.DelegatingFilterProxy，使其 filter-name 与 IoC 容器中我们希望启用的 Filter 对应“挂钩”，SpringSecurity 的 Web 安全方案的启用也是这个原理。

SpringSecurity 默认会需要声明一个默认名称为“springSecurityFilterChain”的 org.springframework.web.filter.DelegatingFilterProxy（web.xml 方式或者 JavaConfig 方式），然后指向 IoC 容器中注册的一个 org.springframework.security.web.FilterChainProxy 实例。

FilterChainProxy 通过扩展 GenericFilterBean 间接实现了 Filter 接口，同时持有一组 SecurityFilterChain，使它可以针对不同的 Web 资源进行特定的防护，这些“角儿”之间的关系大体上如图 2 所示。
FilterChainProxy相关组件关系示意图
图 2  FilterChainProxy 相关组件关系示意图

当然，这些还只是“骨架”，真正执行防护任务的其实是一个个 org.springframework.security.web.SecurityFilterChain 中定义的一系列 Filter：
```
public interface SecurityFilterChain {
    boolean matches(HttpServletRequest request);
    List<Filter> getFilters();
}
```
当我们经常看到如下的 xml schema 形式的配置格式的时候：
```xml
<http auto-config='true'>
    <intercept-url pattern="/login.jsp*" access="IS_AUTHENTICATED_ANONYMOUSLY"/>
    <intercept-url pattern="/**" access="ROLE_USER" />
    <form-login login-page='/login.jsp'/>
</http>
```
其实一个个 http 元素背后对应的就是一个个 SecurityFilterChain 实例，而 http 元素的那些子元素，比如 intercept-url，则对应的就是一个个 Filter。

默认情况下，Spring Security 为 SecurityFilterChain 中的 Filter 序列设定了一个注册框架，以 100 为间隔步长，按照一个合理的顺序来规划和排布常用的 Filter 实现（代码参考FilterComparator）：
```
int order = 100;
put(ChannelProcessingFilter.class, order);
order += STEP;
put(ConcurrentSessionFilter.class, order);
order += STEP;
put(WebAsyncManagerIntegrationFilter.class, order);
order += STEP;
put(SecurityContextPersistenceFilter.class, order);
order += STEP;
put(HeaderWriterFilter.class, order);
order += STEP;
put(CsrfFilter.class, order);
order += STEP;
put(LogoutFilter.class, order);
order += STEP;
put(X509AuthenticationFilter.class, order);
order += STEP;
put(AbstractPreAuthenticatedProcessingFilter.class, order);
order += STEP;
filterToOrder.put("org.springframework.security.cas.web.CasAuthenticationFilter", order);
order += STEP;
put(UsernamePasswordAuthenticationFilter.class, order);
order += STEP;
put(ConcurrentSessionFilter.class, order);
order += STEP;
filterToOrder.put("org.springframework.security.openid.OpenIDAuthenticationFilter", order);
order += STEP;
put(DefaultLoginPageGeneratingFilter.class, order);
order += STEP;
put(ConcurrentSessionFilter.class, order);
order += STEP;
put(DigestAuthenticationFilter.class, order);
order += STEP;
put(BasicAuthenticationFilter.class, order);
order += STEP;
put(RequestCacheAwareFilter.class, order);
order += STEP;
put(SecurityContextHolderAwareRequestFilter.class, order);
order += STEP;
put(JaasApiIntegrationFilter.class, order);
order += STEP;
put(RememberMeAuthenticationFilter.class, order);
order += STEP;
put(AnonymousAuthenticationFilter.class, order);
order += STEP;
put(SessionManagementFilter.class, order);
order += STEP;
put(ExceptionTranslationFilter.class, order);
order += STEP;
put(FilterSecurityInterceptor.class, order);
order += STEP;
put(SwitchUserFilter.class, order); 
```
这些 Filter 虽然很多，但可以简单划分为几类，除个别 Filter 在每个 SecurityFilterChain 都需要，其他可以根据需要选用并添加：
可以认为是信道与状态管理，比如 ChannelProcessingFilter 用于处理 http 或者 https 之间的切换，而 SecurityContextPersistenceFilter 用于重建或者销毁必要的 SecurityContext 状态。
是常见 Web 安全防护类，比如 CsrfFilter。
是认证和授权，比如 BasicAuthenticationFilter、CasAuthen-ticationFilter 等。

最需要重点关注的是 FilterSecurityInterceptor，还记得我们前面说到的 secured object 吧。FilterSecurityInterceptor 就属于放在 Web 访问路径上的那道“关卡”，现在，它的真实位置和效能终于浮出水面了。

ExceptionTranslationFilter 属于另一个需要关注的核心类，它负责接待或者送客，如果访客来访，对方没有报上名来，那么，它会让访客去登记认证（去找 AuthenticationManager 做认证），如果对方报上名了，但认证失败，那么不好意思，请重新认证或者走人。当然，它拒绝访客的方式是抛出相应的 Exception，所以名字叫 ExceptionTranslationFilter。

最后，这个 Filter 序列因为间隔了 100 的步长，所以，我们可以在其中穿插自己的 Filter 实现类，为定制和扩展 SpringSecurity 的防护体系提供了机会。

## 进一步定制spring-boot-starter-security
除了使用 SecurityProperties 暴露的配置项（以 security.* 开头）对 spring-boot-starter-security 进行简单的配置，我们还可以通过给出一个继承了 WebSecurityConfigurerAdapter 的 JavaConfig 配置类对 spring-boot-starter-security 的行为进行更深一级的定制。

使用 WebSecurityConfigurerAdapter 的好处在于，我们依然可以使用 spring-boot-starter-security 默认约定的一些行为，只需要对必要的行为进行调整，比如：
使用其他的 AuthenticationManager 实例。
对默认 HttpSecurity 定义的资源访问的规则进行重新定义。
对默认提供的 WebSecurity 行为进行调整。

为了能够让这些调整生效，我们定义的 WebSecurityConfigurerAdapter 实现类一般在顺序上需要先于 spring-boot-starter-security 默认提供的配置，故此，一般配合@Order（SecurityProperties.ACCESS_OVERRIDE_ORDER）进行标注，代码如下所示：
```java
@Configuration
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class DemoSecurityConfiguration extends WebSecurity-ConfigurerAdapter {
    protected DemoSecurityConfiguration() {
        super(true); // 取消默认提供的安全相关Filters配置
    }
    @Override
    public void configure(WebSecurity web) throws Exception {
        // ...
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // ...
    }
    // 通过Override其他方法实现对web安全的定制
}
```
WebSecurityConfigurerAdapter 其实是为我们预先设定了一个框架，并开放了有限的一些扩展点允许我们对 Web 安全相关的设定进行定制，某些场景下还是会感觉“掣肘”，或者，某些有“洁癖”的开发者，往往不想使用在某些场景下显得并非必要的默认设定。

这个时候，我们可以直接实现并注册一个标注了 @EnableWebSecurity 的 JavaConfig 配置类到 IoC 容器，从而实现一种“颠覆性”的定制，即跟 spring-boot-starter-security 默认提供的 Web 安全相关配置一刀两断，完全自建：
```java
@Configuration
@EnableWebSecurity
public class OverhaulSecurityConfiguration {
  @Bean
  public AuthenticationManager authenticationManager() {
    // ...
  }
  @Bean
  public AccessDecisionManager accessDecisionManager() {
    // ...
  }
  @Bean
  public SecurityFilterChain mySecurityFilterChain() {
    // ...
  }
  // 其他web安全相关组件和依赖配置}
}
```
# spring-boot-starter-actuator与应用监控
所有的应用开发完成之后，其最终目的都是为了上线运行，SpringBoot 应用也不例外，而在应用运行的漫长生命周期内，为了保障其可以持续稳定的服务，我们通常需要对其进行监控，从而可以了解应用的运行状态，并根据情况决定是否需要对其运行状态进行调整。

顺应需求，SpringBoot 框架提供了`spring-boot-starter-actuator`自动配置模块用于支持 SpringBoot 应用的监控。

为了能够感知应用的运行状态，我们通常会设置一些监控指标并采集分析，这些监控指标的采集需要在应用内部设置相应的监控点，这类监控点一般只是读取状态数据，我们通常称它们为`Sensor`，即中文一般称为“传感器”的东西。

应用的运行状态数据通过`Sensors`采集上来之后，我们通常会有专门的系统对这些数据进行分析和判断，一旦某个指标数据超出了预定的阈值，这往往意味着应用的运行状态在这个指标上出现了“不健康”的现象，我们希望对这个指标进行调整，而为了能够执行调整，我们需要预先在应用内部设置对应的执行调整逻辑的控制器。

比如，直接关闭的开关，或者可以执行微调甚至像刹车一样直接快速拉低某个指标值的装置，这些控制器就称为`Actuator`。虽然我们日常天天在说“监控，监控”，但实际上“监”跟“控”是两个概念，`Sensor`更多服务于“监”的场景，而`Actuator`则服务于“控”的场景。

`spring-boot-starter-actuator`自动配置模块默认提供了很多`endpoint`，虽然自动配置模块名为`spring-boot-starter-actuator`，但实际上这些`endpoint`可以按照“监”和“控”划分为两类：
1. `Sensor`类`endpoints`


2. `Actuator`类`endpoints`
* `shutdown`：用于关闭当前 SpringBoot 应用的`endpoint`。
* `dump`：用于执行线程的`dump`操作。

默认情况下，除了`shutdown`这个`endpoint`（因为比较危险，如果没有安全防护，谁都可以访问它，然后关闭应用），其他`endpoints`都是默认启用的。

生产环境下，如果没有启用安全防护（比如没有依赖`spring-boot-starter-security`），那么，建议遵循`Deny By Default`原则，将所有的`endpoints`都关掉，然后根据具体情况单独启用某些 `endpoint`：
```
endpoints.enabled=falseendpoints.info.enabled=trueendpoints.health.enabled=true...
```
所有配置项以`endpoints.`为前缀，然后根据`endpoint`名称划分具体配置项。大部分`endpoints`都是开箱即用，但依然有些`endpoint`提供给我们进一步扩展的权利，比如健康状态检查相关的`endpoint（health endpoint）`。
## 自定义应用的健康状态检查
应用的健康状态检查是很普遍的监控需求，SpringBoot 也预先通过`org.springframework.boot.actuate.autoconfigure.HealthIndicatorAutoConfiguration`为我们提供了一些常见服务的监控检查支持，比如：
```
DataSourceHealthIndicator
DiskSpaceHealthIndicator
RedisHealthIndicator
SolrHealthIndicator
MongoHealthIndicator
```
如果这些默认提供的健康检查支持依然无法满足我们的需要，SpringBoot 还允许我们提供更多的`HealthIndicator`实现，只要将这些`HealthIndicator`实现类注册到 IoC 容器，SpringBoot 会自动发现并使用它们。

假设需要检查依赖的 dubbo 服务是否处于健康状态，我们可以实现一个`DubboHealthIndicator`：
```java
import com.alibaba.dubbo.config.spring.ReferenceBean;
import com.alibaba.dubbo.rpc.service.EchoService;
import org.springframework.boot.actuate.health.AbstractHealthIndicator;
import org.springframework.boot.actuate.health.Health;
public class DubboHealthIndicator extends AbstractHealthIndicator {
  private final ReferenceBean bean;
  public DubboHealthIndicator(ReferenceBean bean) {
    this.bean = bean;
  }
  @Override
  protected void doHealthCheck(Health.Builder builder) throws Exception {
    builder.withDetail("interface", bean.getObjectType());
    final EchoService service = (EchoService) bean.getObject();
    service.$echo("hi");
    builder.up();
  }
}
```
要实现一个自定义的`HealthIndicator`，一般我们不会直接实现（`Implements`）`HealthIndicator`接口，而是继承`AbstractHealthIndicator`：
```java
public abstract class AbstractHealthIndicator implements HealthIndicator {
  @Override
  public final Health health() {
    Health.Builder builder = new Health.Builder();
    try {
      doHealthCheck(builder);
    } catch (Exception ex) {
      builder.down(ex);
    }
    return builder.build();
  }
  protected abstract void doHealthCheck(Health.Builder builder) throws Exception;
}
```
好处就是，我们只需实现`doHealthCheck`，在其中实现我们面向的具体服务的健康检查逻辑就可以了，因此，在`DubboHealthIndicator`实现类中，我们通过 dubbo 框架提供的`EchoService`直接检查相应的 dubbo 服务健康状态即可，只要没有任何异常抛出，我们就认为检查的 dubbo 服务是状态健康的，所以，最后会通过 Health.Builder 的 up() 方法标记服务状态为正常运行。

为了完成对 dubbo 服务的健康检查，只实现一个 DubboHealthIndicator 是不够的，我们还需要将其注册到 IoC 容器中，但是一个一个单独注册太费劲了，而且还要自己提供针对某个 dubbo 服务的 ReferenceBean 依赖实例。

所以，为了一劳永逸，也为了其他人能够同样方便地使用针对 dubbo 服务的健康检查支持，我们可以在 DubboHealthIndicator 的基础上实现一个 spring-boot-starter-dubbo-health-indicator 自动配置模块，即：
```java
@Configuration
@ConditionalOnClass(name = { "com.alibaba.dubbo.rpc.Exporter" })
public class DubboHealthIndicatorConfiguration {
  @Autowired
  HealthAggregator healthAggregator;
  @Autowired(required = false)
  Map<String, ReferenceBean> references;
  @Bean
  public HealthIndicator dubboHealthIndicator() {
    Map<String, HealthIndicator> indicators = new HashMap<>();
    for (String key : references.keySet()) {
        final ReferenceBean bean = references.get(key);
        indicators.put(key.startsWith("&") ? key.replaceFirst("&", "")
                : key, new DubboHealthIndicator(bean));
    }
    return new CompositeHealthIndicator(healthAggregator, indicators);
  }
}
```
然后我们在 spring-boot-starter-dubbo-health-indicator 的 META-INF/spring.factories 文件中添加如下配置：
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\com.keevol...DubboHealthIndicatorConfiguration 

现在，发布 spring-boot-starter-dubbo-health-indicator 并依赖它就可以自动享受到针对当前应用引用的所有 dubbo 服务进行健康检查的服务。

那么针对 Map<String，ReferenceBean>references 的依赖注入是从哪里来的？

其实 Spring 框架支持依赖注入 Key 的类型为 String 的 Map，遇到这种类型的 Map 声明（Map），Spring 框架将扫描容器中所有类型为 T 的 bean，然后以该 bean 的 name 作为 Map 的 Key，以 bean 实例作为对应的 Value，从而构建一个 Map 并注入到依赖处。
开放的 endpoints 才真正“有用”
不管是 spring-boot-starter-actuator 默认提供的 endpoint 实现，还是我们自己给出的 endpoint 实现，如果只是实现了放在 SpringBoot 应用的“身体内部”，那么它们不会发挥任何作用，只有将它们采集的信息暴露开放给外部监控者，或者允许外部监控者访问它们，这些 endpoints 才会真正发挥出它们的最大“功效”。

首先，spring-boot-starter-actuator 会通过 org.springframework.boot.actuate.autoconfigure.EndpointMBeanExportAutoConfiguration 将所有的 org.springframework.boot.actuate.endpoint.Endpoint 实例以 JMX MBean 的形式开放给外部监控者使用。

默认情况下，这些 Endpoint 对应的 JMX MBean 会放在 org.springframework.boot 命名空间下面，不过可以通过 endpoints.jmx.domain 配置项进行更改，比如 endpoints.jmx.domain=com.keevol.management。

EndpointMBeanExportAutoConfiguration 为我们提供了一条很好的应用监控实践之路，既然它会把所有的 org.springframework.boot.actuate.endpoint.Endpoint 实例都作为 JMX Mbean 开放出去，那么，我们就可以提供一批用于某些场景下的自定义 Endpoint 实现类，比如：
```java
public class HelloEndpoint extends AbstractEndpoint<String> {
  public HelloEndpoint(String id) {
    super(id, false);
  }
  @Override
  public String invoke() {
    return "Hello, SpringBoot";
  }
}
```
然后，将像 HelloEndpoint 这样的实现类注册到 SpringBoot 应用的 IoC 容器，就可以扩展 SpringBoot 的 endpoints 功能了。

Endpoint 其实更适合简单的 Sensor 场景（即用于读取或者提供信息），或者简单功能的 actuator 场景（不需要行为参数），如果需要对 SpringBoot 进行更细粒度的监控，可以考虑直接使用 Spring 框架的 JMX 支持。

除了可以使用 JMX 将 spring-boot-starter-actuator 提供的（或者我们自己提供的）endpoints 开放访问，如果当前 SpringBoot 应用恰好又是一个 Web 应用。那么，这些 endpoints 还会通过 HTTP 协议开放给外部访问，与一般的 Web 请求处理一样，使用的也是 Web 应用使用的 HTTP 服务器和地址端口。

因为每个 Endpoint 都有一个 id 作为唯一标识，所以，这些 endpoints 的默认访问路径其实就是它们的 id，比如 info 这个 endpoint 的 HTTP 访问路径就是 /info，而 beans 这个 endpoint 的 HTTP 访问路径则是 /beans，以此类推。

SpringBoot 允许我们通过 management. 为前缀的配置项对 endpoints 的 HTTP 开放行为进行调整：
使用 management.context-path=设置自定义的 endpoints 访问上下文路径，默认直接根路径，即 /info，/beans 等形式。
使用 management.address= 配置单独的 HTTP 服务监听地址，比如只允许本地访问。

management.address=127.0.0.1 使用 management.port=设置单独的监听端口，默认与 web 应用的对外服务端口相同。

我们可以通过 management.port=8888 将管理接口的 HTTP 对外监听端口设置为 8888，但如果 management.port=-1，则意味着我们将关闭管理接口的 HTTP 对外服务。

JMX 和 HTTP 是 endpoints 对外开放访问最常用的方式，鉴于 Java 的序列化漏洞以及 JMX 的远程访问防火墙问题，建议用 HTTP 并配合安全防护将 SpringBoot 应用的 endpoints 开放给外部监控者使用。
用还是不用，这是个问题
endpoints 属于 spring-boot-starter-actuator 提供的主要功能之一，除此之外，spring-boot-starter-actuator 还提供了更多针对应用监控的支持和实现方案。
1. CrshAutoConfiguration 与 spring-boot-starter-remote-shell
spring-boot-starter-actuator 提供了基于 CRaSH（http://www.crashub.org/）的远程 Shell（Remote Shell）支持，从笔者角度来看，这是一把双刃剑，不建议在生产环境使用，因为提供给自己便利的同时，也为黑客朋友们提供了便利。如果实在要用，请加强安全认证和防护。

不过，这里我们还是会为大家分析一下 spring-boot-starter-actuator 是如何提供针对 CRaSH 的支持的。

spring-boot-starter-actuator 提供了 org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration 自动配置类，该类会在 org.crsh.plugin.PluginLifeCycle 出现在 classpath 中的时候生效。

所以，只要将 CRaSH 作为依赖加入应用的 classpath 依赖就可以了，最简单直接的做法是让需要启用 CRaSH 的 SpringBoot 应用依赖 spring-boot-starter-remote-shell 自动配置模块，spring-boot-starter-remote-shell 的主要功效就是提供了针对 CRaSH 的各项依赖。
2. SpringBoot 的 Metrics 与 Dropwizard 的 Metrics
SpringBoot 提供了一套自己的针对系统指标的度量框架，这个框架的核心设计如图 2 所示。
SpringBoot框架的Metrics核心类设计示意图
图 2  SpringBoot 框架的 Metrics 核心类设计示意图

基本上，我们只需关注 org.springframework.boot.actuate.endpoint.PublicMetrics 即可，它可以理解为提供一组 Metric 的集合，我们既可以通过 PublicMetrics 来汇总和管理 Metric，也可以通过 MetricRepository 来存储和管理 Metric。

一旦使用了 spring-boot-starter-actuator，只要当前 SpringBoot 应用的 ApplicationContext 中存在任何 PublicMetrics 实例，EndpointAutoConfiguration 就会将这些 PublicMetrics 采集汇总到一起，然后通过 MetricsEndpoint 将它们开放出去。

spring-boot-starter-actuator 提供的 org.springframework.boot.actuate.autoconfigure.PublicMetricsAutoConfiguration 默认会把一个 SystemPublicMetrics 开放出来，用于提供系统各项指标的度量和状态采集，另外一个就是会把当前 SpringBoot 应用的 ApplicationContext 的 org.springframework.boot.actuate.metrics.repository.MetricRepository 实例中的所有 Metric 汇总并开放出去。

默认如果用户没有给出任何自定义的 MetricRepository，spring-boot-starter-actuator 会提供一个 InMemoryMetricRepository 实现，如果我们将 Dropwizard 的 Metrics 类库作为依赖加入 classpath，那么，Dropwizard Metrics 的 MetricRegistry 中所有的度量指标项也会通过 PublicMetrics 的形式开发暴露出来。

虽然 SpringBoot 提供的 metrics 框架也能帮助我们完成系统和应用指标的度量，但笔者更倾向于使用 Dropwizard 这种特定场景下比较完善的方案，从 metrics 的类型，到外围系统的集成，Dropwizard metrics 都更加成熟和完备。
3. Auditing 与 Trace
SpringBoot 的 Auditing 和 Trace 支持都遵循数据/事件+Repository 的设计（如图 3 所示）。
SpringBoot框架Audit和Trace功能支持核心类示意图
图 3  SpringBoot 框架 Audit 和 Trace 功能支持核心类示意图

我们可能只是通过打印日志时候的 Logger 名称来区分并记录 Audit 事件，然后通过日志采集通道汇总分析就可以了，而不用非要实现一个 LogFileBasedAuditEventRepository 或者 ElasticSearchBasedAuditEventRepository 之类的实现，否则看起来难免有些“学究”气。对于 Trace 来说也是同样道理，我们可能直接使用完备的 APM 方案而不是单一或者少量 Trace 事件的记录。