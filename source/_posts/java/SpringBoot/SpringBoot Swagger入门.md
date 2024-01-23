

# 快速入门 Swagger
## 引入依赖
在 pom.xml 文件中，引入相关依赖。

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

    <artifactId>lab-24-apidoc-swagger</artifactId>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 引入 Swagger 依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>

        <!-- 引入 Swagger UI 依赖，以实现 API 接口的 UI 界面 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>

    </dependencies>

</project>
## SwaggerConfiguration
因为 Spring Boot 暂未提供 Swagger 内置的支持，所以我们需要自己定义配置类。

在 cn.iocoder.springboot.lab24.apidoc.config 包路径下，创建 SwaggerConfiguration 配置类，用于配置 Swagger 。代码如下：

// SwaggerConfiguration.java

@Configuration
@EnableSwagger2 // 标记项目启用 Swagger API 接口文档
public class SwaggerConfiguration {

    @Bean
    public Docket createRestApi() {
        // 创建 Docket 对象
        return new Docket(DocumentationType.SWAGGER_2) // 文档类型，使用 Swagger2
                .apiInfo(this.apiInfo()) // 设置 API 信息
                // 扫描 Controller 包路径，获得 API 接口
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.iocoder.springboot.lab24.apidoc.controller"))
                .paths(PathSelectors.any())
                // 构建出 Docket 对象
                .build();
    }

    /**
     * 创建 API 信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试接口文档示例")
                .description("我是一段描述")
                .version("1.0.0") // 版本号
                .contact(new Contact("芋艿", "http://www.iocoder.cn", "zhijiantianya@gmail.com")) // 联系人
                .build();
    }

}
在类上，添加 @EnableSwagger2 注解， 标记项目启用 Swagger API 接口文档。
通过 #createRestApi() 方法，创建 Swagger Docket Bean 。每个属性的作用，胖友看看艿艿的注释。大多数情况下，胖友使用这些属性是足够的。不过如果想看看其它配置，胖友可以自己去如下两个类翻翻：
Docket.java
ApiInfo.java

## Application
创建 Application.java 类，配置 @SpringBootApplication 注解即可。代码如下：

// Application.java

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
## UserController
在 cn.iocoder.springboot.lab24.apidoc.controller 包路径下，创建 UserController 类，提供用户 API 接口。代码如下：

// UserController.java

@RestController
@RequestMapping("/users")
@Api(tags = "用户 API 接口")
public class UserController {

    @GetMapping("/list")
    @ApiOperation(value = "查询用户列表", notes = "目前仅仅是作为测试，所以返回用户全列表")
    public List<UserVO> list() {
        // 查询列表
        List<UserVO> result = new ArrayList<>();
        result.add(new UserVO().setId(1).setUsername("yudaoyuanma"));
        result.add(new UserVO().setId(2).setUsername("woshiyutou"));
        result.add(new UserVO().setId(3).setUsername("chifanshuijiao"));
        // 返回列表
        return result;
    }

    @GetMapping("/get")
    @ApiOperation("获得指定用户编号的用户")
    @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
    public UserVO get(@RequestParam("id") Integer id) {
        // 查询并返回用户
        return new UserVO().setId(id).setUsername(UUID.randomUUID().toString());
    }

    @PostMapping("add")
    @ApiOperation("添加用户")
    public Integer add(UserAddDTO addDTO) {
        // 插入用户记录，返回编号
        Integer returnId = UUID.randomUUID().hashCode();
        // 返回用户编号
        return returnId;
    }

    @PostMapping("/update")
    @ApiOperation("更新指定用户编号的用户")
    public Boolean update(UserUpdateDTO updateDTO) {
        // 更新用户记录
        Boolean success = true;
        // 返回更新是否成功
        return success;
    }

    @PostMapping("/delete")
    @ApiOperation(value = "删除指定用户编号的用户")
    @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
    public Boolean delete(@RequestParam("id") Integer id) {
        // 删除用户记录
        Boolean success = false;
        // 返回是否更新成功
        return success;
    }

}
相比我们之前使用 SpringMVC 来说，我们在类和接口上，额外增加了 Swagger 提供的注解。
从使用习惯上，我比较喜欢先添加 SpringMVC 的注解，再添加 Swagger 的注解。
因为已经使用了 Swagger 的注解，所以类和方法上的注释，一般可以删除了，除非有特殊诉求。
其中涉及到的 POJO 类，有 UserAddDTO、UserUpdateDTO、UserVO 。
执行 Application 启动项目。然后浏览器访问 http://127.0.0.1:8080/swagger-ui.html 地址，就可以看到 Swagger 生成的 API 接口文档。如下图所示：Swagger-UI 示例

至此，我们已经完成了 Swagger 的快速入门。不过考虑到胖友能够更好的使用，我们来一个一个注解了解。

## 注解
在 swagger-annotations 库中，在 io.swagger.annotations 包路径下，提供了我们会使用到的所有 Swagger 注解。Swagger 提供的注解还是比较多的，大多数场景下，只需要使用到我们在 「2.4 UserController」 中用到的注解。
### @Api
@Api 注解，添加在 Controller 类上，标记它作为 Swagger 文档资源。

示例如下：

// UserController.java

@RestController
@RequestMapping("/users")
@Api(tags = "用户 API 接口")
public class UserController {

    // ... 省略
}
效果如下：@API 示例

@Api 注解的常用属性，如下：

tags 属性：用于控制 API 所属的标签列表。[] 数组，可以填写多个。
可以在一个 Controller 上的 @Api 的 tags 属性，设置多个标签，那么这个 Controller 下的 API 接口，就会出现在这两个标签中。
如果在多个 Controller 上的 @Api 的 tags 属性，设置一个标签，那么这些 Controller 下的 API 接口，仅会出现在这一个标签中。
本质上，tags 就是为了分组 API 接口，和 Controller 本质上是一个目的。所以绝大数场景下，我们只会给一个 Controller 一个唯一的标签。例如说，UserController 的 tags 设置为 "用户 API 接口" 。
@Api 注解的不常用属性，如下：

produces 属性：请求请求头的可接受类型( Accept )。如果有多个，使用 , 分隔。
consumes 属性：请求请求头的提交内容类型( Content-Type )。如果有多个，使用 , 分隔。
protocols 属性：协议，可选值为 "http"、"https"、"ws"、"wss" 。如果有多个，使用 , 分隔。
authorizations 属性：授权相关的配置，[] 数组，使用 @Authorization 注解。
hidden 属性：是否隐藏，不再 API 接口文档中显示。
@Api 注解的废弃属性，不建议使用，有 value、description、basePath、position 。
### @ApiOperation
@ApiOperation 注解，添加在 Controller 方法上，标记它是一个 API 操作。

示例如下：

// UserController.java

@GetMapping("/list")
@ApiOperation(value = "查询用户列表", notes = "目前仅仅是作为测试，所以返回用户全列表")
public List<UserVO> list() {
    // 查询列表
    List<UserVO> result = new ArrayList<>();
    result.add(new UserVO().setId(1).setUsername("yudaoyuanma"));
    result.add(new UserVO().setId(2).setUsername("woshiyutou"));
    result.add(new UserVO().setId(3).setUsername("chifanshuijiao"));
    // 返回列表
    return result;
}
效果如下：@ApiOperation 示例

@ApiOperation 注解的常用属性，如下：

value 属性：API 操作名。
notes 属性：API 操作的描述。
@ApiOperation 注解的不常用属性，如下：

tags 属性：和 @API 注解的 tags 属性一致。
nickname 属性：API 操作接口的唯一标识，主要用于和第三方工具做对接。
httpMethod 属性：请求方法，可选值为 GET、HEAD、POST、PUT、DELETE、OPTIONS、PATCH 。因为 Swagger 会解析 SpringMVC 的注解，所以一般无需填写。
produces 属性：和 @API 注解的 produces 属性一致。
consumes 属性：和 @API 注解的 consumes 属性一致。
protocols 属性：和 @API 注解的 protocols 属性一致。
authorizations 属性：和 @API 注解的 authorizations 属性一致。
hidden 属性：和 @API 注解的 hidden 属性一致。
response 属性：响应结果类型。因为 Swagger 会解析方法的返回类型，所以一般无需填写。
responseContainer 属性：响应结果的容器，可选值为 List、Set、Map 。
responseReference 属性：指定对响应类型的引用。这个引用可以是本地，也可以是远程。并且，当设置了它时，会覆盖 response 属性。说人话，就是可以忽略这个属性，哈哈哈。
responseHeaders 属性：响应头，[] 数组，使用 @ResponseHeader 注解。
code 属性：响应状态码，默认为 200 。
extensions 属性：拓展属性，[] 属性，使用 @Extension 注解。
ignoreJsonView 属性：在解析操作和类型，忽略 JsonView 注释。主要是为了向后兼容。
@ApiOperation 注解的废弃属性，不建议使用，有 position 。

### @ApiImplicitParam
@ApiImplicitParam 注解，添加在 Controller 方法上，声明每个请求参数的信息。

示例如下：

// UserController.java

@PostMapping("/delete")
@ApiOperation(value = "删除指定用户编号的用户")
@ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
public Boolean delete(@RequestParam("id") Integer id) {
    // 删除用户记录
    Boolean success = false;
    // 返回是否更新成功
    return success;
}
效果如下：@ApiImplicitParam 示例

@ApiImplicitParam 注解的常用属性，如下：

name 属性：参数名。
value 属性：参数的简要说明。
required 属性：是否为必传参数。默认为 false 。
dataType 属性：数据类型，通过字符串 String 定义。
dataTypeClass 属性：数据类型，通过 dataTypeClass 定义。在设置了 dataTypeClass 属性的情况下，会覆盖 dataType 属性。推荐采用这个方式。
paramType 属性：参数所在位置的类型。有如下 5 种方式：
"path" 值：对应 SpringMVC 的 @PathVariable 注解。
【默认值】"query" 值：对应 SpringMVC 的 @PathVariable 注解。
"body" 值：对应 SpringMVC 的 @RequestBody 注解。
"header" 值：对应 SpringMVC 的 @RequestHeader 注解。
"form" 值：Form 表单提交，对应 SpringMVC 的 @PathVariable 注解。
😈 绝大多数情况下，使用 "query" 值这个类型即可。
example 属性：参数值的简单示例。
examples 属性：参数值的复杂示例，使用 @Example 注解。
@ApiImplicitParam 注解的不常用属性，如下：

defaultValue 属性：默认值。
allowableValues 属性：允许的值。如果要设置多个值，有两种方式：
数组方式，即 {value1, value2, value3} 。例如说，{1, 2, 3} 。
范围方式，即 [value1, value2] 或 [value1, value2) 。例如说 [1, 5] 表示 1 到 5 的所有数字。如果有无穷的，可以使用 (-infinity, value2] 或 [value1, infinity) 。
allowEmptyValue 属性：是否允许空值。
allowMultiple 属性：是否允许通过多次传递该参数来接受多个值。默认为 false 。
type 属性：搞不懂具体用途，对应英文注释为 Adds the ability to override the detected type 。
readOnly 属性：是否只读。
format 属性：自定义的格式化。
collectionFormat 属性：针对 Collection 集合的，自定义的格式化。
当我们需要添加在方法上添加多个 @ApiImplicitParam 注解时，可以使用 @ApiImplicitParams 注解中添加多个。示例如下：

@ApiImplicitParams({ // 参数数组
        @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024"), // 参数一
        @ApiImplicitParam(name = "name", value = "昵称", paramType = "query", dataTypeClass = String.class, required = true, example = "芋道"), // 参数二
})

### @ApiModel
@ApiModel 注解，添加在 POJO 类，声明 POJO 类的信息。而在 Swagger 中，把这种 POJO 类称为 Model 类。所以，我们下文就统一这么称呼。

示例如下：

// UserVO.java

@ApiModel("用户 VO")
public class UserVO {

    // ... 省略

}
效果如下：@ApiModel 示例

@ApiModel 注解的常用属性，如下：

value 属性：Model 名字。
description 属性：Model 描述。
@ApiModel 注解的不常用属性，如下：

parent 属性：指定该 Model 的父 Class 类，用于继承父 Class 的 Swagger 信息。
subTypes 属性：定义该 Model 类的子类 Class 们。
discriminator 属性：搞不懂具体用途，对应英文注释为 Supports model inheritance and polymorphism.
reference 属性：搞不懂具体用途，对应英文注释为 Specifies a reference to the corresponding type definition, overrides any other metadata specified

### @ApiModelProperty
@ApiModelProperty 注解，添加在 Model 类的成员变量上，声明每个成员变量的信息。

示例如下：

// UserVO.java

@ApiModel("用户 VO")
public class UserVO {

    @ApiModelProperty(value = "用户编号", required = true, example = "1024")
    private Integer id;
    @ApiModelProperty(value = "账号", required = true, example = "yudaoyuanma")
    private String username;

    // ... 省略 set/get 方法
}
效果如下：@ApiModelProperty 示例

@ApiModelProperty 注解的常用属性，如下：

value 属性：属性的描述。
dataType 属性：和 @ApiImplicitParam 注解的 dataType 属性一致。不过因为 @ApiModelProperty 是添加在成员变量上，可以自动获得成员变量的类型。
required 属性：和 @ApiImplicitParam 注解的 required 属性一致。
example 属性：@ApiImplicitParam 注解的 example 属性一致。
@ApiModelProperty 注解的不常用属性，如下：

name 属性：覆盖成员变量的名字，使用该属性进行自定义。
allowableValues 属性：和 @ApiImplicitParam 注解的 allowableValues 属性一致。
position 属性：成员变量排序位置，默认为 0 。
hidden 属性：@ApiImplicitParam 注解的 hidden 属性一致。
accessMode 属性：访问模式，有 AccessMode.AUTO、AccessMode.READ_ONLY、AccessMode.READ_WRITE 三种，默认为 AccessMode.AUTO 。
reference 属性：和 @ApiModel 注解的 reference 属性一致。
allowEmptyValue 属性：和 @ApiImplicitParam 注解的 allowEmptyValue 属性一致。
extensions 属性：和 @ApiImplicitParam 注解的 extensions 属性一致。
@ApiModelProperty 注解的废弃属性，不建议使用，有 readOnly 。

### @ApiResponse
在大多数情况下，我们并不需要使用 @ApiResponse 注解，因为我们会类似 UserController#get(id) 方法这个接口，返回一个 Model 即可。所以 @ApiResponse 注解，艿艿就简单讲解，嘿嘿。

@ApiResponse 注解，添加在 Controller 类的方法上，声明每个响应参数的信息。

@ApiResponse 注解的属性，基本已经被 @ApiOperation 注解所覆盖，如下：

message 属性：响应的提示内容。
code 属性：和 @ApiOperation 注解的 code 属性一致。
response 属性：和 @ApiOperation 注解的 response 属性一致。
reference 属性：和 @ApiOperation 注解的 responseReference 属性一致。
responseHeaders 属性：和 @ApiOperation 注解的 responseHeaders 属性一致。
responseContainer 属性：和 @ApiOperation 注解的 responseContainer 属性一致。
examples 属性：和 @ApiOperation 注解的 examples 属性一致。
当我们需要添加在方法上添加多个 @ApiResponse 注解时，可以使用 @ApiResponses 注解中添加多个。

至此，我们已经了解完 Swagger 项目中提供的主要注解。如果想要看到更多的 Swagger 的使用示例，可以看看艿艿开源的 onemall 项目。

咳咳咳，整理 Swagger 注解的每个属性，真的是花费时间。如果有哪个解释不到位，请一定给艿艿留言，我去优化和调整下，嘻嘻。
## 测试接口
在 Swagger 的 UI 界面上，提供了简单的测试接口的工具。我们仅仅需要点开某个接口，点击「Try it out」按钮。如下图：「Try it out」

然后，点击「Execute」按钮，即可执行一次 API 接口的调用。如下图：「Execute」

在三个红圈中，我们可以看到 Swagger 给我们提供了：

提供了 curl 命令，让我们可以直接在命令行执行。
提供了 Request URL 地址，方便我们在浏览器中访问。
提供了执行结果，我们可以人肉看看，是否符合我们希望的结果。
😈 当然，实际项目开发中，艿艿还是喜欢 Postman 来测试接口，嘿嘿。

# 更好看的 Swagger UI 界面
# Swagger Starter
## 快速体验
我们先来快速体验下 Swagger 官方 Starter，体验下其提供的自动配置的功能。

新建一个示例项目，最终代码会如下图：

## 引入依赖
在 pom.xml 文件中，引入 springfox-boot-starter 的依赖。

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.11.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>lab-24-apidoc-swagger-starter</artifactId>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 实现对 Swagger 的自动配置 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-boot-starter</artifactId>
            <version>3.0.0</version>
        </dependency>
    </dependencies>

</project>
注意，最低支持使用 SpringBoot 版本为 2.2+ 。

版本差异：是否使用 Starter 的依赖对比如下：

依赖对比

## 示例代码
下面，我们来编写用于展示 Swagger 功能的示例代码，和是否使用 Starter 并没有任何差别。

① 创建 UserController 类，添加相应的 Swagger 注解。代码如下：

@RestController
@RequestMapping("/users")
@Api(tags = "用户 API 接口")
public class UserController {

    @GetMapping("/list")
    @ApiOperation(value = "查询用户列表", notes = "目前仅仅是作为测试，所以返回用户全列表")
    public List<UserVO> list() {
        // 查询列表
        List<UserVO> result = new ArrayList<>();
        result.add(new UserVO().setId(1).setUsername("yudaoyuanma"));
        result.add(new UserVO().setId(2).setUsername("woshiyutou"));
        result.add(new UserVO().setId(3).setUsername("chifanshuijiao"));
        // 返回列表
        return result;
    }

    @GetMapping("/get")
    @ApiOperation("获得指定用户编号的用户")
    @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
    public UserVO get(@RequestParam("id") Integer id) {
        // 查询并返回用户
        return new UserVO().setId(id).setUsername(UUID.randomUUID().toString());
    }

    @PostMapping("add")
    @ApiOperation("添加用户")
    public Integer add(UserAddDTO addDTO) {
        // 插入用户记录，返回编号
        Integer returnId = UUID.randomUUID().hashCode();
        // 返回用户编号
        return returnId;
    }

    @PostMapping("/update")
    @ApiOperation("更新指定用户编号的用户")
    public Boolean update(UserUpdateDTO updateDTO) {
        // 更新用户记录
        Boolean success = true;
        // 返回更新是否成功
        return success;
    }

    @PostMapping("/delete")
    @ApiOperation(value = "删除指定用户编号的用户")
    @ApiImplicitParam(name = "id", value = "用户编号", paramType = "query", dataTypeClass = Integer.class, required = true, example = "1024")
    public Boolean delete(@RequestParam("id") Integer id) {
        // 删除用户记录
        Boolean success = false;
        // 返回是否更新成功
        return success;
    }

}
UserAddDTO
UserUpdateDTO
UserVO
② 创建 Application 类，用于 SpringBoot 应用的启动。代码如下：

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
## 简单测试
一切准备就绪，执行 Application 启动 SpringBoot 应用。

使用浏览器，访问 http://127.0.0.1:8080/swagger-ui/ 地址，进入 Swagger UI 界面。如下图所示：

版本差异：新版本的 Swagger UI 界面的地址，是 /swagger-ui/，而不是老版本的 /swagger-ui.html。

Swagger UI 界面

如此，我们已经完成了 Swagger 的快速集成与体验，还是非常方便。
## 自定义配置
当我们想进行 Swagger 接口文档的自定义时，例如说修改 title 标题、description 描述等等信息时，却发现官方 Starter 并未提供对应的配置项。如下图所示：

配置项

这就导致，我们不得不创建 Configuration 类，进行自定义配置。下面，我们来演示下。

3.1 SwaggerConfiguration
创建 SwaggerConfiguration 类，设置自定义的 title 标题、description 描述等等信息。代码如下：

@Configuration
// @EnableSwagger2 无需使用该注解
public class SwaggerConfiguration {

    @Bean
    public Docket createRestApi() {
        // 创建 Docket 对象
        return new Docket(DocumentationType.SWAGGER_2) // 文档类型，使用 Swagger2
                .apiInfo(this.apiInfo()) // 设置 API 信息
                // 扫描 Controller 包路径，获得 API 接口
                .select()
                .apis(RequestHandlerSelectors.basePackage("cn.iocoder.springboot.lab24.controller"))
                .paths(PathSelectors.any())
                // 构建出 Docket 对象
                .build();
    }

    /**
     * 创建 API 信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("测试接口文档示例")
                .description("我是一段描述")
                .version("1.0.0") // 版本号
                .contact(new Contact("芋艿", "http://www.iocoder.cn", "zhijiantianya@gmail.com")) // 联系人
                .build();
    }

}
版本差异：在使用官方 Starter 时，我们并不需要添加 @EnableSwagger2 注解，声明开启 Swagger 的功能。

3.2 简单测试
重启启动 SpringBoot 应用，访问 Swagger UI 界面，查看自定义配置是否生效。如下图所示：

Swagger UI 界面

成功~

4. 自定义 Starter
因为官方 Starter 提供的配置项较少，所以艿艿建议可以在其的基础之上，自定义一个公司的 Swagger Starter，提供更多自定义的配置项。

例如说，艿艿在自己的 onemall 开源项目中，自定义了 mall-spring-boot-starter-swagger 库。比较简单，胖友一看就明白，就不详细讲解代码。如下图所示：

自定义 Swagger Starter

这样，我们在 Web 项目中使用时，只需要引入 mall-spring-boot-starter-swagger 依赖，添加几行 Swagger 配置即可。如下图所示：

具体使用
