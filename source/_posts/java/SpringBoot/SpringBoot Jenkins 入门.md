

# 项目打包
Spring Boot 提供了 Maven 插件`spring-boot-maven-plugin`，可以方便的将 Spring Boot 项目打成 jar 包或者 war 包。

考虑到部署的便利性，我们绝大多数 99.99% 的场景下，我们会选择打成 jar 包。这样，我们就无需在部署项目的服务器上，配置相应的 Tomcat、Jetty 等 Servlet 容器。所以本小节，我们也是将 Spring Boot 项目，打成 jar 包。
## 引入依赖
在 pom.xml 文件中，引入相关依赖。
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-41-demo01</artifactId>
    <!-- <1> 配置打成 jar 包 -->
    <packaging>jar</packaging>

    <dependencies>
        <!-- <2> 实现对 SpringMVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- <3> 实现对 Actuator 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

    <build>
        <!-- <4> 设置构建的 jar 包名 -->
        <finalName>${project.artifactId}</finalName>
        <!-- 使用 spring-boot-maven-plugin 插件打包 -->
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
<1> 处，通过 <packaging /> 设置为 jar，表示该项目构建打包成 jar 包。😈 如果胖友想要打成 war 包，可以将此处修改成 war。
<2> 处，引入 spring-boot-starter-web 依赖，实现对 SpringMVC 的自动化配置。
<3> 处，引入 spring-boot-starter-actuator 依赖，实现对 Spring Boot Actuator 的自动化配置。这样，我们就可以使用 Actuator 提供的 health 端点，获得健康检查所需的 HTTP API。
<4> 处，设置构建的 jar 包的名字。因为我们在《芋道 Jenkins 极简入门》的「2. 快速入门」小节中，提供的 deploy.sh 部署脚本，暂时不支持 jar 包的名字带有版本号，所以这里我们需要进行下设置。
<5> 处，引入 spring-boot-maven-plugin 插件，因为我们要使用它构建打包 Spring Boot 项目。

## 配置文件
在 resources 目录下，创建 5 个配置文件，对应不同的环境。如下：

application-local.yaml，本地环境。
application-dev.yaml，开发环境。
application-uat.yaml，UAT 环境。
application-pre.yaml，预发布环境。
application-prod.yaml，生产环境。
嘿嘿，现在暂时偷懒，所以每个配置文件的内容基本是一样的。如下：

server:
  port: 8079

management:
  server:
    port: 8078 # 自定义端口，避免 Nginx 暴露出去

  endpoints:
    web:
      exposure:
        include: '*' # 需要开放的端点。默认值只打开 health 和 info 两个端点。通过设置 * ，可以开放所有端点。
server.port 配置项，设置 SpringMVC 的服务器端口。
management.server.port 配置项，设置 Spring Boot Actuator 的服务器端口。
management.endpoints.web.exposure.include 配置项，设置 Spring Boot Actuator 的开放的端点。
通过多个不同的配置文件，搭配上我们在《芋道 Jenkins 极简入门》的「2. 快速入门」小节中，提供的 deploy.sh 部署脚本，设置使用 jar 包中不同的环境配置。

不过可能胖友会吐槽，一个 jar 包包含所有环境的配置，会不会不安全的问题？！确实存在，所以我们一般会做几个事情：

1、不同环境处于不同的网络环境中。例如说，我们测试环境使用一个[阿里云 VPC](专有网络 VPC)，正式环境使用另一个阿里云 VPC。这样，在测试环境下，即使使用 application-prod.yaml 配置文件，因为不同网络环境，也是无法连接正式环境的服务。
2、将 application-pre.yaml 和 application-prod.yaml 配置文件，放在对应环境中，避免泄露。又或者考虑采用配置中心。
3、敏感配置，进行加密处理。
😈 上述三个方案，可以一起采用，目前我们就是这么干的。

## DemoController
在 cn.iocoder.springboot.lab40.jenkinsdemo.controller 包路径下，创建 DemoController 类，提供示例 API 接口。代码如下：

@RestController
@RequestMapping("/demo")
public class DemoController {

    @GetMapping("/echo")
    public String echo() {
        return "echo";
    }

}

## Application
创建 Application.java 类，配置 @SpringBootApplication 注解即可。代码如下：

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Component
    public class Listener implements ApplicationListener<ApplicationEvent> {

        @Override
        public void onApplicationEvent(ApplicationEvent event) {
            if (event instanceof ContextClosedEvent) {
                this.sleep(10);
            }
        }

        private void sleep(int seconds) {
            try {
                Thread.sleep(seconds * 1000L);
            } catch (InterruptedException ignore) {
            }
        }

    }

}
这里创建的 Listener 监听器，主要是为了监听 ContextClosedEvent 事件，在 Spring 容器关闭之前，sleep 10 秒。这样，我们就能模拟 Spring Boot 应用，关闭时需要一段时间的过程。😈 毕竟，咱这个示例基本是一个“空”项目，关闭速度非常快，无法完整测试 deploy.sh 部署脚本的关闭流程。

## 简单测试
如果我们使用 IDEA，可以直接通过界面，使用 spring-boot-maven-plugin 插件打包。如下图所示：IDEA 打包

当然，我们也可以通过执行 mvn clean package -pl lab-41/lab-41-demo01 -Dmaven.test.skip=true 命令，构建 lab-41/lab-41-demo01 子 Maven 模块。

友情提示：如果胖友要构建整个项目，可以考虑使用 mvn clean package -Dmaven.test.skip=true 命令。

打包完成后，我们可以使用 java -jar 命令，进行启动。操作命令如下：

# 进入 jar 包所在目录，即指定项目的 target 目录下
$ cd lab-41/lab-41-demo01/target

# 启动 Java 服务。这里，我们使用 local 环境
$ java -jar lab-41-demo01.jar --spring.profiles.active=local
... 一堆 Spring Boot 项目的启动日志。

# 优雅上下线
示例代码对应仓库：lab-41-demo02。

在生产环境下，我们会部署相同 Spring Boot 项目的，启动多个 Java 服务。而后，通过 Nginx 对 Java 服务进行负载均衡，实现高可用。如下图所示：


一般情况下，我们是通过静态配置 Nginx Upstream，负载均衡到多个 Java 服务。示例如下：

upstream bck_testing_01 {
  server 192.168.250.220:8080
  server 192.168.250.221:8080
  server 192.168.250.222:8080
}
这样，就会存在两个问题？！

问题一，如果某个 Java 服务宕机后，如果在 Nginx Upstream 还配置着该 Java 服务，Nginx 还是会负载均衡到该节点，导致返回错误给用户。
问题二，在《芋道 Jenkins 极简入门》中我们提到过，Java 服务的启动和关闭是一个过程，需要一段时间，所以在此期间，Nginx 转发请求到该 Java 服务时，可能会响应较慢或者返回错误给用户。那么，每次我们在使用 Jenkins 部署 Spring Boot 项目时，必然会触发该问题。
在不考虑使用网关或者注册中心的情况下，我们需要动态化 Nginx Upstream 配置，实现如下效果：

在 Java 服务宕机时，又或者 Java 服务正在关闭时，Nginx 将该 Java 服务从 Nginx Upstream 配置中移除，避免转发请求到其上。
在 Java 服务完成启动时，Nginx 才将该 Java 服务在 Nginx Upstream 配置中添加，开始转发请求到其上。
如果我们要实现该效果，需要给 Nginx 增加健康检查功能，对 Nginx Upstream 配置的 Java 服务，进行不断的健康检查。在健康检查通过时，才真正将该 Java 服务添加到 Nginx Upstream 配置中，才会转发请求到其上。

😈 咳咳咳，感觉自己有点啰嗦。

在 Nginx 上，有多种健康检查插件，在[《Nginx 负载均衡中后端节点服务器健康检查 - 运维笔记》](TODO https://www.cnblogs.com/kevingrace/p/6685698.html)中，有详细的介绍。不过真正可用的，只有阿里开源的 Tengine 的 Upstream check module 插件。因为只有该插件，支持主动向 Nginx Upstream 配置的 Java 服务，进行健康检查。

下面，我们开始本小节的示例：

首先，我们会搭建一个 Spring Boot 项目。整体来说，会和「2. 项目打包」类似。不过为了更好的和 Nginx 的健康检查集成，我们会自定义一个 HealthIndicator 实现类，嘿嘿。
然后，我们会搭建一个 Tengine 服务，配置对搭建的 Spring Boot 项目的负载均衡与健康检查。
最后，我们会进行简单的测试。
一共三个步骤，走起~

## Spring Boot 项目搭建
和「2. 项目打包」基本相似的，主要是三个部分：

引入依赖：和「2.1 引入依赖」一致，可见 pom.xml 文件。
配置文件：和「2.2 配置文件」大致一致，可见 在 resources 目录。【重要】不过额外配置了 management.endpoint.health.show-details=always，设置 Actuator 的 health 端点，支持展示健康状态明细。同时因为该设置，我们可以只访问「3.1.1 ServerHealthIndicator」，详细后面会说。
DemoController：和和「2.3 DemoController」一致，可见 在 DemoController 类。
下面，我们来看看不同的部分。

### ServerHealthIndicator
在 cn.iocoder.springboot.lab40.jenkinsdemo.actuate 包下，创建 ServerHealthIndicator 类，自定义服务器状态的 HealthIndicator 实现类。代码如下：

@Component
public class ServerHealthIndicator extends AbstractHealthIndicator implements ApplicationListener<ApplicationEvent> {

    private Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 是否在服务中
     */
    private volatile boolean inService = false;

    @Override
    protected void doHealthCheck(Health.Builder builder) {
        if (inService) {
            builder.up();
        } else {
            builder.down();
        }
    }

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationReadyEvent) {
            this.handleApplicationReadyEvent((ApplicationReadyEvent) event);
        } else if (event instanceof ApplicationFailedEvent) {
            this.handleApplicationFailedEvent((ApplicationFailedEvent) event);
        } else if (event instanceof ContextClosedEvent) {
            this.handleContextClosedEvent((ContextClosedEvent) event);
        }
    }

    @SuppressWarnings("unused")
    private void handleApplicationReadyEvent(ApplicationReadyEvent event) {
        this.inService = true;
    }

    @SuppressWarnings("unused")
    private void handleApplicationFailedEvent(ApplicationFailedEvent event) {
        this.inService = false;
    }

    @SuppressWarnings("unused")
    private void handleContextClosedEvent(ContextClosedEvent event) {
        // 标记不提供服务
        this.inService = false;

        // sleep 等待负载均衡完成健康检查
        for (int i = 0; i < 20; i++) { // TODO 20 需要配置
            logger.info("[handleContextClosedEvent][优雅关闭，第 {} sleep 等待负载均衡完成健康检查]", i);
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException ignore) {
            }
        }
    }

}
① 继承 AbstractHealthIndicator 抽象类，所以实现 #doHealthCheck(Health.Builder builder) 方法，根据 inService 的状态，返回服务器是 UP 还是 DOWN 状态。

② 实现 ApplicationListener 接口，所以实现 #onApplicationEvent(ApplicationEvent event) 方法，监听 ApplicationEvent 事件。Spring Boot 在启动的过程中，会发布不同的 ApplicationEvent 事件，因此我们可以根据这些事件，标记服务器状态 inService，是否处于服务中。不同的 ApplicationEvent 时间处理如下：

如果是 ApplicationReadyEvent 事件，说明 Spring Boot 应用启动完毕，相应的 Bean 等都初始化完成，可以正常提供服务。因而，我们标记 inService = true。
如果是 ApplicationFailedEvent 事件，说明 Spring Boot 应用启动失败，无法正常提供服务。因此，我们标记 inService = false。
如果是 ContextClosedEvent 事件，说明 Spring 容器要开始关闭了，一旦开始关闭，无法正常提供服务。因此，我们标记 inService = false。😈 同时，这里非常关键，这里我们额外 sleep 了 20 秒，这样 Tengine 能够完成对该 Spring Boot 应用的健康检查，将其从 Upstream 配置移除，嘿嘿。
胖友在一起结合 ① 和 ②，思考下整个过程。后续，我们可以通过 http://127.0.0.1:8078/actuator/health/server 地址，返回服务器是否提供服务。当然，实际上也是可以使用 http://127.0.0.1:8078/actuator/health/server 地址，看自己喜好吧。

不过，ServerHealthIndicator 针对 ContextClosedEvent 的 sleep 20 秒，是一定要做的，保证 Tengine 有足够的时间，在该 Spring Boot 应用关闭前，完成对其的健康检查，从而从 Upstream 配置移除。咳咳咳，😈 好像又啰嗦了一遍。

### Demo02Application
创建 Demo02Application.java 类，配置 @SpringBootApplication 注解即可。代码如下：

@SpringBootApplication
public class Demo02Application {

    public static void main(String[] args) {
        SpringApplication.run(Demo02Application.class, args);
    }

}

