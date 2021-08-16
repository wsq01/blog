


Maven 是一款基于 Java 平台的项目管理和整合工具，它将项目的开发和管理过程抽象成一个项目对象模型（POM）。开发人员只需要做一些简单的配置，Maven 就可以自动完成项目的编译、测试、打包、发布以及部署等工作。

Maven 是使用 Java 语言编写的，因此它和 Java 一样具有跨平台性，这意味着无论是在 Windows ，还是在 Linux 或者 Mac OS 上，都可以使用相同的命令进行操作。

Maven 使用标准的目录结构和默认构建生命周期，因此开发者几乎不用花费多少时间就能够自动完成项目的基础构建工作。

Maven 能够帮助开发者完成以下任务：
构建项目
生成文档
创建报告
维护依赖
软件配置管理
发布
部署

总而言之，Maven 简化并标准化了项目构建过程。它将项目的编译，生成文档，创建报告，发布，部署等任务无缝衔接，构建成一套完整的生命周期。
约定优于配置
约定优于配置（Convention Over Configuration）是 Maven 最核心的涉及理念之一 ，Maven对项目的目录结构、测试用例命名方式等内容都做了规定，凡是使用 Maven 管理的项目都必须遵守这些规则。

Maven 项目构建过程中，会自动创建默认项目结构，开发人员仅需要在相应目录结构下放置相应的文件即可。

例如，下表显示了项目源代码文件，资源文件和其他配置在 Maven 项目中的默认位置。 

Maven 的特点
Maven 具有以下特点：
设置简单。
所有项目的用法一致。
可以管理和自动进行更新依赖。
庞大且不断增长的资源库。
可扩展，使用 Java 或脚本语言可以轻松的编写插件。
几乎无需额外配置，即可立即访问新功能。
基于模型的构建：Maven 能够将任意数量的项目构建为预定义的输出类型，例如 JAR，WAR。
项目信息采取集中式的元数据管理：使用与构建过程相同的元数据，Maven 能够生成一个网站（site）和一个包含完整文档的 PDF。
发布管理和发行发布：Maven 可以与源代码控制系统（例如 Git、SVN）集成并管理项目的发布。
向后兼容性：您可以轻松地将项目从旧版本的 Maven 移植到更高版本的 Maven 中。
并行构建：它能够分析项目依赖关系，并行构建工作，使用此功能，可以将性能提高 20%-50％。
更好的错误和完整性报告：Maven 使用了较为完善的错误报告机制，它提供了指向 Maven Wiki 页面的链接，您将在其中获得有关错误的完整描述。

# 在 windows 上安装 Maven
在安装 Maven 之前要确保安装了 JDK。

访问 Maven 下载页面：`https://maven.apache.org/download.cgi`，下载`apache-maven-3.6.3-bin.zip`。

{% asset_img img1.png %}

将压缩包解压到指定的目录中，比如：`C:\path\apache-maven-3.6.3`。

接着需要设置 Maven 的环境变量。

{% asset_img img2.png %}
{% asset_img img3.png %}

在 cmd 中输入`mvn -v`，检查 Maven 是否安装成功。

# POM
POM（Project Object Model，项目对象模型）是 Maven 的基本组件，它是以 xml 文件的形式存放在项目的根目录下，名称为 pom.xml。

POM 中定义了项目的基本信息，用于描述项目如何构建、声明项目依赖等等。

当 Maven 执行一个任务时，它会先查找当前项目的 POM 文件，读取所需的配置信息，然后执行任务。在 POM 中可以设置如下配置：
项目依赖
插件
目标
构建时的配置文件
版本 
开发者
邮件列表

在创建 POM 之前，首先要确定工程组（groupId），及其名称（artifactId）和版本，在仓库中这些属性是项目的唯一标识。
POM 示例
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>net.biancheng.www</groupId>
    <artifactId>maven</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</project>

所有的 Maven 项目都有一个 POM 文件，所有的 POM 文件都必须有 project 元素和 3 个必填字段：groupId、artifactId 以及 version。
Super POM
无论 POM 文件中是否显示的声明，所有的 POM 均继承自一个父 POM，这个父 POM 被称为 Super POM，它包含了一些可以被继承的默认设置。

Maven 使用 effective pom (Super POM 的配置加上项目的配置）来执行相关任务，它替开发人员在 pom.xml 中做了一些最基本的配置。当然，开发人员依然可以根据需要重写其中的配置信息。

执行以下命令 ，就可以查看 Super POM 的默认配置。
mvn help:effective-pom
示例
1. 在 D 盘创建一个名为 maven 的目录，将 POM 示例中的 pom.xml 文件移动至该文件夹下。

2. 打开命令行窗口，跳转到 pom.xml 所在的目录下，执行以下 mvn 命令。
mvn help:effective-pom

3. Maven 将会开始执行命令并显示 effective-pom 的内容，执行结果如下。


你可以看到 effective-pom 中包含了 Maven 在执行任务时需要用到的默认目录结构、输出目录、插件、仓库和报表目录等内容。
实际开发过程中，Maven 的 pom.xml 文件不需要手工编写，Maven 提供了大量的原型（Archetype）插件来创建项目，包括项目结构和 pom.xml。
# 创建Maven项目
Maven 提供了大量不同类型的 Archetype 模板，通过它们可以帮助用户快速的创建 Java 项目，其中最简单的模板就是 maven-archetype-quickstart，它只需要用户提供项目最基本的信息，就能生成项目的基本结构及 POM 文件。
创建 Maven 项目
下面我们将通过 maven-archetype-quickstart 原型，在 D:\maven 目录中创建一个基于 Maven 的 Java 项目。

打开命令行窗口，跳转到 D:\maven 目录，执行以下 mvn 命令。
​mvn archetype:generate -DgroupId=net.biancheng.www -DartifactId=helloMaven -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

参数说明：
-DgroupId: 项目组 ID，通常为组织名或公司网址的反写。
-DartifactId: 项目名。
-DarchetypeArtifactId: 指定 ArchetypeId，maven-archetype-quickstart 用于快速创建一个简单的 Maven 项目。
-DinteractiveMode: 是否使用交互模式。

Maven 开始进行处理，并创建一套完整的 Maven 项目目录结构。


目录结构
进入 D:\maven 目录， 我们看到 Maven 已经创建了一个名为 helloMaven 的 Java 项目（在 artifactId 中指定的），该项目使用一套标准的目录结构，如下图所示。

目录及文件说明：
helloMaven：项目名，包含 src 文件夹和 pom.xml。
src/main/java：用于存放项目的 Java 文件。
src/main/resources：用于存放项目资源文件。
src/test/java：用于存放所有测试 Java 文件，如 JUnit 测试类。
src/test/resources ：用于存放测试资源文件。
target：项目输出位置，用于存放编译后的文件。
pom.xml：Maven 项目核心配置文件。

Maven 创建项目时，还自动生成了两个Java 文件： App.java 和 AppTest.java。其中 App.java 位于 src/main/java 下 ，代码如下。
package net.biancheng.www;
/**
* Hello world!
*/
public class App {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}

AppTest.java 位于 src/test/java 下，代码如下。
package net.biancheng.www;
import junit.framework.Test;
import junit.framework.TestCase;
import junit.framework.TestSuite;
/**
* Unit test for simple App.
*/
public class AppTest
        extends TestCase {
    /**
     * Create the test case
     *
     * @param testName name of the test case
     */
    public AppTest(String testName) {
        super(testName);
    }
    /**
     * @return the suite of tests being tested
     */
    public static Test suite() {
        return new TestSuite(AppTest.class);
    }
    /**
     * Rigourous Test :-)
     */
    public void testApp() {
        assertTrue(true);
    }
}
注：本节侧重点在于使用 maven-archetype-quickstart 原型快速创建一个简单的 Maven 项目，为后续的学习做准备。对于 archetype 了解即可，在后面的 Maven Archetype 模板 中会详细讲解。
# Maven项目的构建与测试
# 构建项目
查看 helloMaven 项目的 pom.xml 文件，配置如下。
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>net.biancheng.www</groupId>
    <artifactId>helloMaven</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>helloMaven</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

从以上配置可知，Maven 已经添加了 Junit 作为该项目的测试框架，且 Maven 也在项目中自动生成了一个源码文件 App.java 和一个测试文件 AppTest.java 。

打开命令行窗口，跳转到 D:\maven\helloMaven 目录，执行以下 mvn 命令，对该项目进行构建。
mvn clean package

项目构建完成后，在该项目根目录中生成了一个名为 target 的目录，该目录包含以下内容。


说明：
Maven 命令中包含了两个命令：clean 和 package，其中 clean 负责清理 target 目录，package 负责将项目构建并打包输出为 jar 文件。
classes：源代码编译后存放在该目录中。
test-classes：测试源代码编译后并存放在该目录中。
surefire-reports：Maven 运行测试用例生成的测试报告存放在该目录中。
helloMaven-1.0-SNAPSHOT.jar：Maven 对项目进行打包生成的 jar 文件。

打开命令控制台，跳转到 D:\maven\helloMaven\target\classes 目录，执行 Java 命令
java net.biancheng.www.App

执行结果如下。
Hello World!
测试项目
下面我们介绍如何在 Maven 项目中添加其他的 Java 文件，并进行测试。

打开 D:\maven\helloMaven\src\main\java\net\biancheng\www 文件夹，在其中创建一个名为  Util.java 的文件。
package net.biancheng.www;
/**
* Maven 项目中添加的自定义java类
*
* @author 编程帮 www.biancheng.net
*/
public class Util {
    public static void printMessage(String message) {
        System.out.println(message);
    }
}

更新 App 类，调用 Util 类中的 printMessage() 方法，代码如下。
package net.biancheng.www;
/**
* @author 编程帮 www.biancheng.net
*/
public class App {
    public static void main(String[] args) {
        Util.printMessage("编程帮欢迎您，网址：www.biancheng.net");
    }
}

打开命令行窗口，跳转到 D:\maven\helloMaven 目录，并执行如下 Maven 命令，对项目进行编译。
mvn clean compile                                                                                  

命令执行结果如下。

编译成功后，跳转到 D:\maven\helloMaven\target\classes 目录下，执行以下 Java 命令。
java  net.biancheng.www.App                                                                                    

执行结果如下。
编程帮欢迎您，网址：www.biancheng.net

# Maven坐标
在 Maven 中，任何一个依赖、插件或者项目构建的输出，都可以称为构件。

Maven 坐标规定：任何一个构件都可以使用 Maven 坐标并作为其唯一标识，Maven 坐标包括`groupId、artifactId、version、packaging`等元素，只要用户提供了正确的坐标元素，Maven 就能找到对应的构件。 

任何一个构件都必须明确定义自己的坐标，这是 Maven 的强制要求。我们在开发 Maven 项目时，也需要为其定义合适的坐标，只有定义了坐标，其他项目才能引用该项目生成的构件。
```xml
<project> 
    <groupId>net.xxx.www</groupId>
    <artifactId>helloMaven</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
</project>
```
Maven 坐标主要由以下元素组成：
* `groupId`：项目组 ID，定义当前 Maven 项目隶属的组织或公司，通常是唯一的。它的取值一般是项目所属公司或组织的网址或 URL 的反写。
* `artifactId`：项目 ID，通常是项目的名称。
* `version`：版本。
* `packaging`：项目的打包方式，默认值为`jar`。

`groupId、artifactId`和`version`是必须定义的，`packaging`是可选的。
# Maven依赖
如果一个 Maven 构建所产生的构件（例如 Jar 文件）被其他项目引用，那么该构件就是其他项目的依赖。
## 依赖声明
Maven 坐标是依赖的前提，所有 Maven 项目必须明确定义自己的坐标，只有这样，它们才可能成为其他项目的依赖。

当 Maven 项目需要声明某一个依赖时，通常只需要在其 POM 中配置该依赖的坐标信息，Maven 会根据坐标自动将依赖下载到项目中。

某个项目中使用`servlet-api`作为其依赖，其配置如下。
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  ...
  <dependencies>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
</project>
```
`dependencies`元素可以包含一个或者多个`dependency`子元素，用以声明一个或者多个项目依赖，每个依赖都可以包含以下元素：
* `groupId、artifactId`和`version`：依赖的基本坐标，对于任何一个依赖来说，基本坐标是最重要的，Maven 根据坐标才能找到需要的依赖。
* `type`：依赖的类型，对应于项目坐标定义的`packaging`。大部分情况下，该元素不必声明，其默认值是`jar`。
* `scope`：依赖的范围。
* `optional`：标记依赖是否可选。
* `exclusions`：用来排除传递性依赖。

# Maven仓库
在 Maven 中，任何一个依赖、插件或者项目构建的输出，都可以称为构件。

Maven 在某个统一的位置存储所有项目的构件，这个统一的位置，我们就称之为仓库。换言之，仓库就是存放依赖和插件的地方。

任何的构件都有唯一的坐标，该坐标定义了构件在仓库中的唯一存储路径。当 Maven 项目需要某些构件时，只要其 POM 文件中声明了这些构件的坐标，Maven 就会根据这些坐标找自动到仓库中找到并使用它们。

项目构建完成生成的构件，也可以安装或者部署到仓库中，供其他项目使用。
## 仓库的分类
Maven 仓库可以分为 2 大类：本地仓库、远程仓库。

当 Maven 根据坐标寻找构件时，它会首先查看本地仓库，若本地仓库存在此构件，则直接使用；若本地仓库不存在此构件，Maven 就会去远程仓库查找，若发现所需的构件后，则下载到本地仓库使用。如果本地仓库和远程仓库都没有所需的构件，则 Maven 就会报错。

远程仓库还可以分为 3 个小类：中央仓库、私服、其他公共仓库。

中央仓库是由 Maven 社区提供的一种特殊的远程仓库，它包含了绝大多数流行的开源构件。在默认情况下，当本地仓库没有 Maven 所需的构件时，会首先尝试从中央仓库下载。

私服是一种特殊的远程仓库，它通常设立在局域网内，用来代理所有外部的远程仓库。它的好处是可以节省带宽，比外部的远程仓库更加稳定。

除了中央仓库和私服外，还有很多其他公共仓库，例如 JBoss Maven 库，Java.net Maven 库等等。

{% asset_img 4.png Maven 仓库分类 %}

## 本地仓库
Maven 本地仓库实际上就是本地计算机上的一个目录（文件夹），它会在第一次执行 Maven 命令时被创建。

Maven 本地仓库可以储存本地所有项目所需的构件。当 Maven 项目第一次进行构建时，会自动从远程仓库搜索依赖项，并将其下载到本地仓库中。当项目再进行构建时，会直接从本地仓库搜索依赖项并引用，而不会再次向远程仓库获取。

Maven 本地仓库默认地址为`C:\%USER_HOME%\.m2\repository`，但出于某些原因（例如 C 盘空间不够），我们通常会重新自定义本地仓库的位置。这时需要修改`%MAVEN_HOME%\conf`目录下的`settings.xml`文件，通过`localRepository`元素定义另一个本地仓库地址，例如：
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>D:/myRepository/repository</localRepository>
</settings>
 ```
构件只有储存在本地仓库中，才能被其他的 Maven 项目使用。构件想要进入本地仓库，除了从远程仓库下载到本地仓库外，还可以使用命令`mvn install`将本地项目的输出构件安装到本地仓库中。
## 中央仓库
中央仓库是由 Maven 社区提供的一种特殊的远程仓库，它包含了绝大多数流行的开源构件。在默认情况下，当本地仓库没有 Maven 所需的构件时，会首先尝试从中央仓库下载。

中央仓库具有如下特点：
* 这个仓库由 Maven 社区管理
* 不需要配置
* 需要通过网络才能访问

我们可以通过 Maven 社区提供的 URL：`http://search.maven.org/#browse`，浏览其中的构件。中央仓库包含了绝大多数流行的开源 Java 构件及其源码、作者信息和许可证信息等。一般来说，Maven 项目所依赖的构件都可以从中央仓库下载到。

虽然中央仓库属于远程仓库的范畴，但由于它的特殊性，一般会把它与其他远程仓库区分开。我们常说的远程仓库，一般不包括中央仓库。
## 远程仓库
如果 Maven 在本地仓库和中央仓库中都找不到依赖的库文件，它就会停止构建过程并输出错误信息到控制台。为避免这种情况的发生，Maven 还提供了远程仓库的概念，它是一种由开发人员自己定制的仓库，其中包含了供其他项目使用的代码库或者构件。
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>net.biancheng.www</groupId>
    <artifactId>maven</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>com.bangcheng.common-lib</groupId>
            <artifactId>common-lib</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>
    <repositories>
        <repository>
            <id>bianchengbang.lib1</id>
            <url>http://download.bianchengbang.org/maven2/lib1</url>
        </repository>
        <repository>
            <id>bianchengbang.lib2</id>
            <url>http://download.bianchengbang.org/maven2/lib2</url>
        </repository>
    </repositories>
</project>
```
## Maven 依赖搜索顺序
当通过 Maven 构建项目时，Maven 按照如下顺序查找依赖的构件。
1. 从本地仓库查找构件，如果没有找到，跳到第 2 步，否则继续执行其他处理。
2. 从中央仓库查找构件，如果没有找到，并且已经设置其他远程仓库，然后移动到第 4 步；如果找到，那么将构件下载到本地仓库中使用。
3. 如果没有设置其他远程仓库，Maven 则会停止处理并抛出错误。
4. 在远程仓库查找构件，如果找到，则会下载到本地仓库并使用，否则 Maven 停止处理并抛出错误。

# Maven生命周期
这个生命周期将项目的清理、初始化、编译、测试、打包、集成测试、验证、部署和站点生成等几乎所有构建过程进行了抽象和统一。
