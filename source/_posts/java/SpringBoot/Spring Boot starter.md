


# starter入门
Spring Boot 将日常企业应用研发中的各种场景都抽取出来，做成一个个的`starter`（启动器），starter 中整合了该场景下各种可能用到的依赖，用户只需要在 Maven 中引入`starter`依赖，SpringBoot 就能自动扫描到要加载的信息并启动相应的默认配置。starter 提供了大量的自动配置，让用户摆脱了处理各种依赖和配置的困扰。所有这些`starter`都遵循着约定成俗的默认配置，并允许用户调整这些配置，即遵循“约定大于配置”的原则。

以`spring-boot-starter-web`为例，它能够为提供 Web 开发场景所需要的几乎所有依赖，因此在使用 Spring Boot 开发 Web 项目时，只需要引入该`Starter`即可，而不需要额外导入 Web 服务器和其他的 Web 依赖。

在`pom.xml`中引入`spring-boot-starter-web`。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<!--SpringBoot父项目依赖管理-->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.4.5</version>
		<relativePath/>
	</parent>
	....
	<dependencies>
		<!--导入 spring-boot-starter-web-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		...
	</dependencies>
	...
</project>
```
在该项目中执行以下`mvn`命令查看器依赖树。
```
mvn dependency:tree
```
执行结果如下。
```
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< net.biancheng.www:helloworld >--------------------
[INFO] Building helloworld 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:3.1.2:tree (default-cli) @ helloworld ---
[INFO] net.biancheng.www:helloworld:jar:0.0.1-SNAPSHOT
[INFO] \- org.springframework.boot:spring-boot-starter-web:jar:2.4.5:compile
[INFO]    +- org.springframework.boot:spring-boot-starter:jar:2.4.5:compile
[INFO]    |  +- org.springframework.boot:spring-boot:jar:2.4.5:compile
[INFO]    |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.4.5:compile
[INFO]    |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.4.5:compile
[INFO]    |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO]    |  |  |  +- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO]    |  |  |  \- org.slf4j:slf4j-api:jar:1.7.30:compile
[INFO]    |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.13.3:compile
[INFO]    |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.13.3:compile
[INFO]    |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.30:compile
[INFO]    |  +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
[INFO]    |  +- org.springframework:spring-core:jar:5.3.6:compile
[INFO]    |  |  \- org.springframework:spring-jcl:jar:5.3.6:compile
[INFO]    |  \- org.yaml:snakeyaml:jar:1.27:compile
[INFO]    +- org.springframework.boot:spring-boot-starter-json:jar:2.4.5:compile
[INFO]    |  +- com.fasterxml.jackson.core:jackson-databind:jar:2.11.4:compile
[INFO]    |  |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.11.4:compile
[INFO]    |  |  \- com.fasterxml.jackson.core:jackson-core:jar:2.11.4:compile
[INFO]    |  +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.11.4:compile
[INFO]    |  +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.11.4:compile
[INFO]    |  \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.11.4:compile
[INFO]    +- org.springframework.boot:spring-boot-starter-tomcat:jar:2.4.5:compile
[INFO]    |  +- org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.45:compile
[INFO]    |  +- org.glassfish:jakarta.el:jar:3.0.3:compile
[INFO]    |  \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:9.0.45:compile
[INFO]    +- org.springframework:spring-web:jar:5.3.6:compile
[INFO]    |  \- org.springframework:spring-beans:jar:5.3.6:compile
[INFO]    \- org.springframework:spring-webmvc:jar:5.3.6:compile
[INFO]       +- org.springframework:spring-aop:jar:5.3.6:compile
[INFO]       +- org.springframework:spring-context:jar:5.3.6:compile
[INFO]       \- org.springframework:spring-expression:jar:5.3.6:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.505 s
[INFO] Finished at: 2021-04-30T14:52:30+08:00
[INFO] ------------------------------------------------------------------------
```
从以上结果中，我们可以看到 Spring Boot 导入了 springframework、logging、jackson 以及 Tomcat 等依赖，而这些正是我们在开发 Web 项目时所需要的。
## spring-boot-starter-parent
`spring-boot-starter-parent`是所有 Spring Boot 项目的父级依赖，它被称为 Spring Boot 的版本仲裁中心，可以对项目内的部分常用依赖进行统一管理。
```xml
<!--SpringBoot父项目依赖管理-->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.4.5</version>
	<relativePath/>
</parent>
```
Spring Boot 项目可以通过继承`spring-boot-starter-parent`来获得一些合理的默认配置，它主要提供了以下特性：
* 默认 JDK 版本（Java 8）
* 默认字符集（UTF-8）
* 依赖管理功能
* 资源过滤
* 默认插件配置
* 识别`application.properties`和`application.yml`类型的配置文件

查看`spring-boot-starter-parent`的底层代码，可以发现其有一个父级依赖`spring-boot-dependencies`。
```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-dependencies</artifactId>
	<version>2.4.5</version>
</parent>
```
`spring-boot-dependencies`的底层代码如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
				xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
				xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
	<modelVersion>4.0.0</modelVersion>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-dependencies</artifactId>
	<version>2.4.5</version>
	<packaging>pom</packaging>
	....
	<properties>
		<activemq.version>5.16.1</activemq.version>
		<antlr2.version>2.7.7</antlr2.version>
		<appengine-sdk.version>1.9.88</appengine-sdk.version>
		<artemis.version>2.15.0</artemis.version>
		<aspectj.version>1.9.6</aspectj.version>
		<assertj.version>3.18.1</assertj.version>
		<atomikos.version>4.0.6</atomikos.version>
		....
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.apache.activemq</groupId>
				<artifactId>activemq-amqp</artifactId>
				<version>${activemq.version}</version>
			</dependency>
			<dependency>
				<groupId>org.apache.activemq</groupId>
				<artifactId>activemq-blueprint</artifactId>
				<version>${activemq.version}</version>
			</dependency>
			...
		</dependencies>
	</dependencyManagement>
	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.codehaus.mojo</groupId>
					<artifactId>build-helper-maven-plugin</artifactId>
					<version>${build-helper-maven-plugin.version}</version>
				</plugin>
				<plugin>
					<groupId>org.flywaydb</groupId>
					<artifactId>flyway-maven-plugin</artifactId>
					<version>${flyway.version}</version>
				</plugin>
				...
			</plugins>
		</pluginManagement>
	</build>
</project>
```
以上配置中，部分元素说明如下：
* `dependencyManagement`：负责管理依赖；
* `pluginManagement`：负责管理插件；
* `properties`：负责定义依赖或插件的版本号。

`spring-boot-dependencies`通过`dependencyManagement、pluginManagement`和`properties`等元素对一些常用技术框架的依赖或插件进行了统一版本管理，例如`Activemq、Spring、Tomcat`等。

# spring-boot-starter-web
Spring Boot 为 Spring MVC 提供了自动配置，并在 Spring MVC 默认功能的基础上添加了以下特性：

引入了 ContentNegotiatingViewResolver 和 BeanNameViewResolver（视图解析器）
对包括 WebJars 在内的静态资源的支持
自动注册 Converter、GenericConverter 和 Formatter （转换器和格式化器）
对 HttpMessageConverters 的支持（Spring MVC 中用于转换 HTTP 请求和响应的消息转换器）
自动注册 MessageCodesResolver（用于定义错误代码生成规则）
支持对静态首页（index.html）的访问
自动使用 ConfigurableWebBindingInitializer

只要我们在 Spring  Boot 项目中的 pom.xml 中引入了 spring-boot-starter-web ，即使不进行任何配置，也可以直接使用 Spring MVC 进行 Web 开发。
示例 
1. 创建一个名为 spring-boot-springmvc-demo1 的 Spring Boot 工程，并在其 pom.xml 的dependencies 节点中添加 spring-boot-starter-web 的依赖，代码如下。
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

2. 在 net.biancheng.www 包下创建一个名为 HelloController，代码如下。
package net.biancheng.www.controller;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello() {
        return "www.biancheng.net";
    }
}

3. 启动 Spring Boot，浏览器访问“http://localhost:8080/hello”，结果如下图。


注意：由于 spring-boot-starter-web 默认替我们引入了核心启动器 spring-boot-starter，因此，当 Spring Boot  项目中的 pom.xml 引入了 spring-boot-starter-web 的依赖后，就无须在引入 spring-boot-starter 核心启动器的依赖了。