


# 环境
需要使用 JDK 1.8 版本编译 Spring 的代码。

使用 IDEA 从官方仓库`https://github.com/spring-projects/spring-framework`克隆项目。我们使用的 Spring 版本是 5.3.10-SNAPSHOT。
# 下载依赖
克隆完 Spring 项目之后，用 IDEA 打开，IDEA 会自动下载需要的 Gradle 工具。我们使用的 Gradle 版本是 7.2.0。

下载完 Gradle 工具之后，IDEA 就会使用它自动下载相关的依赖库。

因为 Gradle 支持使用 Maven 依赖，所以我们可以使用阿里云的 Maven 镜像`https://maven.aliyun.com/nexus/content/groups/public/`。修改`build.gradle`文件：

{% asset_img 1.png %}

# 调试 Spring 示例
依赖下载完，我们就可以调试 Spring 的源码。虽然说 Spring 并没有直接提供`example`使用示例项目，但是我们通过调试 Spring 提供的单元测试类，了解 Spring 的执行流程。

例如说：
1. 通过 Debug 运行`XmlBeanDefinitionReaderTests`类的`withFreshInputStream()`的方法，调试 Spring 解析 XML 配合，获得`Bean`的定义。如下图所示：

{% asset_img 2.png %}

Spring 是先解析到`Bean`的定义，然后创建`Bean`对象。

2. 通过 Debug 运行`ClassPathXmlApplicationContextTests`类的`testSingleConfigLocation()`的方法，调试 Spring 容器的初始化过程，包括`Bean`的创建。如下图所示：

{% asset_img 3.png %}

