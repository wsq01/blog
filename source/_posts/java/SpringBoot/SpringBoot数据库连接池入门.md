


# Druid 单数据源
## 引入依赖
在`pom.xml`文件中，引入相关依赖。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-19-datasource-pool-druid-single</artifactId>

    <dependencies>
        <!-- 保证 Spring JDBC 的依赖健全 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- 实现对 Druid 连接池的自动化配置 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.21</version>
        </dependency>
        <dependency> <!-- 本示例，我们使用 MySQL -->
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
        </dependency>

        <!-- 实现对 Spring MVC 的自动化配置，因为我们需要看看 Druid 的监控功能 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 方便等会写单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```
## 应用配置文件
在`application.yml`中，添加`Druid`配置：
```yaml
spring:
  # datasource 数据源配置内容，对应 DataSourceProperties 配置属性类
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/test?useSSL=false&useUnicode=true&characterEncoding=UTF-8
    driver-class-name: com.mysql.jdbc.Driver
    username: root # 数据库账号
    password: # 数据库密码
    type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
    # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
    druid:
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
      filter:
        stat: # 配置 StatFilter ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter
          log-slow-sql: true # 开启慢查询记录
          slow-sql-millis: 5000 # 慢 SQL 的标准，单位：毫秒
      stat-view-servlet: # 配置 StatViewServlet ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
        enabled: true # 是否开启 StatViewServlet
        login-username: yudaoyuanma # 账号
        login-password: javaniubi # 密码
```
`spring.datasource`配置项，设置 Spring 数据源的通用配置。其中，`spring.datasource.type`配置项，需要主动设置使用`DruidDataSource`。因为默认情况下，`spring-boot-starter-jdbc`的`DataSourceBuilder`会按照`DATA_SOURCE_TYPE_NAMES`的顺序，尝试推断数据源的类型。

`spring.datasource.druid`配置项，设置`Druid`连接池的自定义配置。然后`DruidDataSourceAutoConfigure`会自动化配置`Druid`连接池。
* `filter.stat`配置项，我们配置了`Druid StatFilter`，用于统计监控信息。要注意，`StatFilter`并不是我们说的`javax.servlet.Filter`，而是`Druid`提供的`Filter`拓展机制。
* `filter.stat-view-servlet`配置项，我们配置了`Druid StatViewServlet`，用于提供监控信息的展示的 html 页面和 JSON API。`StatViewServlet`就是我们说的`javax.servlet.Filter`。

## Application
创建`Application.java`类，配置`@SpringBootApplication`注解即可。
```java
@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Autowired
    private DataSource dataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        logger.info("[run][获得数据源：{}]", dataSource.getClass());
    }
}
```
执行日志如下：
```
2019-11-12 19:34:12.079  INFO 48359 --- [           main] c.a.d.s.b.a.DruidDataSourceAutoConfigure : Init DruidDataSource
2019-11-12 19:34:12.156  INFO 48359 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited

2019-11-12 19:34:12.560  INFO 48359 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得数据源：class com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceWrapper]
```
可以看到，`DruidDataSource`成功初始化。
## 监控功能
因为我们在应用配置中，做了如下操作：
通过`spring.datasource.filter.stat`配置了`StatFilter`，统计监控信息。
通过`spring.datasource.filter.stat-view-servlet`配置了`StatViewServlet`，提供监控信息的展示的 html 页面和 JSON API。

所以我们在启动项目后，访问`http://127.0.0.1:8080/druid`地址，可以看到监控 html 页面。如下图所示：

{% asset_img 1.png %}

在界面的顶部，提供了数据源、SQL 监控、SQL 防火墙等等功能。

每个界面上，可以通过 View JSON API 获得数据的来源。同时，我们可以在 JSON API(`http://127.0.0.1:8080/druid/api.html`) 菜单对应的界面中，看到`StatViewServlet`内置的监控信息的 JSON API 列表。

因为监控信息是存储在 JVM 内存中，在 JVM 进程重启时，信息将会丢失。如果我们希望持久化到 MySQL、Elasticsearch、HBase 等存储器中，可以通过`StatViewServlet`提供的 JSON API 接口，采集监控信息。

如果`StatViewServlet`提供的 JSON API 接口，无法满足我们的诉求，我们可以通过自定义 API 接口，使用`DruidStatManagerFacade`获得监控信息。使用示例`DruidStatController`代码如下：
```java
// DruidStatController.java

@RestController
public class DruidStatController {

    @GetMapping("/monitor/druid/stat")
    @Deprecated
    public Object druidStat(){
        // `DruidStatManagerFacade#getDataSourceStatDataList()` 方法，可以获取所有数据源的监控数据。
        // 除此之外，DruidStatManagerFacade 还提供了一些其他方法，你可以按需选择使用。
        return DruidStatManagerFacade.getInstance().getDataSourceStatDataList();
    }

}
```
当然，绝大多数情况下，我们并不需要做这方面的拓展。
# Druid 多数据源
## 应用配置
在`application.yml`中，添加`Druid`配置：
```yaml
spring:
  # datasource 数据源配置内容
  datasource:
    # 订单数据源配置
    orders:
      url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password:
      type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
      # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
    # 用户数据源配置
    users:
      url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
      driver-class-name: com.mysql.jdbc.Driver
      username: root
      password:
      type: com.alibaba.druid.pool.DruidDataSource # 设置类型为 DruidDataSource
      # Druid 自定义配置，对应 DruidDataSource 中的 setting 方法的属性
      min-idle: 0 # 池中维护的最小空闲连接数，默认为 0 个。
      max-active: 20 # 池中最大连接数，包括闲置和使用中的连接，默认为 8 个。
    # Druid 自定已配置
    druid:
      # 过滤器配置
      filter:
        stat: # 配置 StatFilter ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter
          log-slow-sql: true # 开启慢查询记录
          slow-sql-millis: 5000 # 慢 SQL 的标准，单位：毫秒
      # StatViewServlet 配置
      stat-view-servlet: # 配置 StatViewServlet ，对应文档 https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE
        enabled: true # 是否开启 StatViewServlet
        login-username: yudaoyuanma # 账号
        login-password: javaniubi # 密码
```
我们将 Druid 的自定义配置，和`url、driver-class-name`等数据源的通用配置放在同一级，这样后续我们只需要使用`@ConfigurationProperties(prefix = "spring.datasource.orders")`的方式，就能完成`DruidDataSource`的配置属性设置。

在`spring.datasource.druid`配置项下，我们还是配置了`filter.stat`和`stat-view-servlet`配置项，用于 Druid 监控功能。
## 数据源配置类
在`cn.iocoder.springboot.lab19.datasourcepool.config`包路径下，我们会创建`DataSourceConfig`配置类。代码如下：
```java
// DataSourceConfig.java

@Configuration
public class DataSourceConfig {

    /**
     * 创建 orders 数据源
     */
    @Primary
    @Bean(name = "ordersDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.orders") // 读取 spring.datasource.orders 配置到 HikariDataSource 对象
    public DataSource ordersDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    /**
     * 创建 users 数据源
     */
    @Bean(name = "usersDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.users")
    public DataSource usersDataSource() {
        return DruidDataSourceBuilder.create().build();
    }

}
```
因为我们在应用配置中，将 Druid 自定义的配置项，和数据源的通用配置放在了同一级，所以我们只需使用`@ConfigurationProperties(prefix = "spring.datasource.orders")`这样的方式即可。
## Application
创建`Application.java`类，配置`@SpringBootApplication`注解即可。代码如下：
```java
// Application.java

@SpringBootApplication
public class Application implements CommandLineRunner {

    private Logger logger = LoggerFactory.getLogger(Application.class);

    @Resource(name = "ordersDataSource")
    private DataSource ordersDataSource;

    @Resource(name = "usersDataSource")
    private DataSource usersDataSource;

    public static void main(String[] args) {
        // 启动 Spring Boot 应用
        SpringApplication.run(Application.class, args);
    }

    @Override
    public void run(String... args) {
        // orders 数据源
        logger.info("[run][获得数据源：{}]", ordersDataSource.getClass());

        // users 数据源
        logger.info("[run][获得数据源：{}]", usersDataSource.getClass());
    }
}
```
执行日志如下：
```
2019-11-12 21:39:24.063  INFO 49670 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得数据源：class com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceWrapper]
2019-11-12 21:39:24.064  INFO 49670 --- [           main] c.i.s.lab19.datasourcepool.Application   : [run][获得数据源：class com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceWrapper]
```
可以看到，两个`DruidDataSource`成功初始化。
