
# 快速入门
使用`spring-boot-starter-web`实现 SpringMVC 的自动化配置。然后实现用户的增删改查接口。接口列表如下：

| 请求方法|  URL         | 功能 |
| :--: | :--: | :--: |
| GET     | /users      | 查询用户列表 |
| GET     | /users/{id} | 获得指定用户编号的用户 |
| POST    | /users      | 添加用户 |
| PUT     | /users/{id} | 更新指定用户编号的用户 |
| DELETE  | /users/{id} | 删除指定用户编号的用户 |

## 注解
SpringMVC 提供的注解：
* `@Controller`
* `@RestController`
* `@RequestMapping`
* `@GetMapping`
* `@PostMapping`
* `@PutMapping`
* `@RequestParam`
* `@PathVariable`

### @Controller
`@Controller`注解，添加在类上，表示这是控制器`Controller`对象。属性如下：
* name 属性：该 Controller 对象的 Bean 名字。允许空。

`@RestController`注解，添加在类上，是`@Controller`和`@ResponseBody`的组合注解，直接使用接口方法的返回结果，经过 JSON/XML 等序列化方式，最终返回。也就是说，无需使用`InternalResourceViewResolver`解析视图，返回 HTML 结果。

### @RequestMapping
`@RequestMapping`注解，添加在类或方法上，标记该类/方法对应接口的配置信息。

`@RequestMapping`注解的常用属性，如下：
`path`属性：接口路径。`[]`数组，可以填写多个接口路径。
`values`属性：和`path`属性相同，是它的别名。
`method`属性：请求方法`RequestMethod`，可以填写`GET、POST、POST、DELETE`等等。`[]`数组，可以填写多个请求方法。如果为空，表示匹配所有请求方法。

`@RequestMapping`注解的不常用属性，如下：
* `name`属性：接口名。一般情况下，我们不填写。
* `params`属性：请求参数需要包含值的参数名。可以填写多个参数名。如果为空，表示匹配所有请你求方法。
* `headers`属性：和`params`类似，只是从参数名变成请求头。
* `consumes`属性：和`params`类似，只是从参数名变成请求头的提交内容类型(`Content-Type`)
* `produces`属性：和`params`类似，只是从参数名变成请求头的(`Accept`)可接受类型。

考虑到让开发更加方便，Spring 给每种请求方法提供了对应的注解：
* `@GetMapping`注解：对应`@GET`请求方法的`@RequestMapping`注解。
* `@PostMapping`注解：对应`@POST`请求方法的`@RequestMapping`注解。
* `@PutMapping`注解：对应`@PUT`请求方法的`@RequestMapping`注解。
* `@DeleteMapping`注解：对应`@DELETE`请求方法的`@RequestMapping`注解。

### @RequestParam
`@RequestParam`注解，添加在方法参数上，标记该方法参数对应的请求参数的信息。属性如下：
* `name`属性：对应的请求参数名。如果为空，则直接使用方法上的参数变量名。
* `value`属性：和`name`属性相同，是它的别名。
* `required`属性：参数是否必须传。默认为`true`，表示必传。
* `defaultValue`属性：参数默认值。

`@PathVariable`注解，添加在方法参数上，标记接口路径和方法参数的映射关系。相比`@RequestParam`注解，少一个`defaultValue`属性。
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

    <artifactId>lab-springmvc-23-01</artifactId>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
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
## Application
创建`Application.java`类，配置`@SpringBootApplication`注解即可。
```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
## UserController
创建`UserController`类。
```java
// UserController.java

@RestController
@RequestMapping("/users")
public class UserController {

    /**
     * 查询用户列表
     *
     * @return 用户列表
     */
    @GetMapping("")
    public List<UserVO> list() {
        // 查询列表
        List<UserVO> result = new ArrayList<>();
        result.add(new UserVO().setId(1).setUsername("yudaoyuanma"));
        result.add(new UserVO().setId(2).setUsername("woshiyutou"));
        result.add(new UserVO().setId(3).setUsername("chifanshuijiao"));
        // 返回列表
        return result;
    }

    /**
     * 获得指定用户编号的用户
     *
     * @param id 用户编号
     * @return 用户
     */
    @GetMapping("/{id}")
    public UserVO get(@PathVariable("id") Integer id) {
        // 查询并返回用户
        return new UserVO().setId(id).setUsername("username:" + id);
    }

    /**
     * 添加用户
     *
     * @param addDTO 添加用户信息 DTO
     * @return 添加成功的用户编号
     */
    @PostMapping("")
    public Integer add(UserAddDTO addDTO) {
        // 插入用户记录，返回编号
        Integer returnId = 1;
        // 返回用户编号
        return returnId;
    }

    /**
     * 更新指定用户编号的用户
     *
     * @param id 用户编号
     * @param updateDTO 更新用户信息 DTO
     * @return 是否修改成功
     */
    @PutMapping("/{id}")
    public Boolean update(@PathVariable("id") Integer id, UserUpdateDTO updateDTO) {
        // 将 id 设置到 updateDTO 中
        updateDTO.setId(id);
        // 更新用户记录
        Boolean success = true;
        // 返回更新是否成功
        return success;
    }

    /**
     * 删除指定用户编号的用户
     *
     * @param id 用户编号
     * @return 是否删除成功
     */
    @DeleteMapping("/{id}")
    public Boolean delete(@PathVariable("id") Integer id) {
        // 删除用户记录
        Boolean success = false;
        // 返回是否更新成功
        return success;
    }

}
```
在类上，添加`@RestController`注解，表示直接返回接口结果。默认情况下，使用 JSON 作为序列化方式。

在类上，添加`@RequestMapping("/users")`注解，表示`UserController`所有接口路径，以`/users`开头。

`list()`方法，查询用户列表。请求对应`GET /users`，请求结果为：
```json
[
    {
        "id": 1,
        "username": "yudaoyuanma"
    },
    {
        "id": 2,
        "username": "woshiyutou"
    },
    {
        "id": 3,
        "username": "chifanshuijiao"
    }
]
```
其中，`UserVO`为用户返回`VO`类。

`get(Integer id)`方法，获得指定用户编号的用户。请求对应`GET /users/{id}`，请求结果为:
```json
{
    "id": 1,
    "username": "username:1"
}
```
`add(UserAddDTO addDTO)`方法，添加用户。请求对应`POST /users`，请求结果为：
```
1
```
因为我们这里返回的是`Integer`类型，对于非`POJO`对象，所以无需使用 JSON 序列化返回。

其中，`UserAddDTO`为用户添加`DTO`类。

`update(Integer id, UserUpdateDTO updateDTO)`方法，更新指定用户编号的用户。请求对应`PUT /users/{id}`，请求结果为：
```
true
```
其中，`UserUpdateDTO`为用户更新`DTO`类。

`delete(Integer id)`方法，删除指定用户编号的用户。请求对应`DELETE /users/{id}`，请求结果为：
```
false
```
以上的测试，肯定需要通过运行`Application`，启动项目。这里，补充下它的启动日志如下：
```bash
2019-11-15 18:46:00.671  INFO 99493 --- [           main] c.i.s.lab23.springmvc.Application        : Starting Application on MacBook-Pro-8 with PID 99493 (/Users/yunai/Java/SpringBoot-Labs/lab-23/lab-springmvc-23-01/target/classes started by yunai in /Users/yunai/Java/SpringBoot-Labs)
2019-11-15 18:46:00.673  INFO 99493 --- [           main] c.i.s.lab23.springmvc.Application        : No active profile set, falling back to default profiles: default
2019-11-15 18:46:01.593  INFO 99493 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-11-15 18:46:01.613  INFO 99493 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-11-15 18:46:01.613  INFO 99493 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.16]
2019-11-15 18:46:01.619  INFO 99493 --- [           main] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/Users/yunai/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.]
2019-11-15 18:46:01.684  INFO 99493 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-11-15 18:46:01.684  INFO 99493 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 976 ms
2019-11-15 18:46:01.844  INFO 99493 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-11-15 18:46:01.987  INFO 99493 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-11-15 18:46:01.990  INFO 99493 --- [           main] c.i.s.lab23.springmvc.Application        : Started Application in 1.559 seconds (JVM running for 2.146)
```
我们可以看到，Spring Boot 在启动 SpringMVC 时，会默认初始化一个内嵌的 Tomcat ，监听 8080 端口的请求。
# 测试接口
在开发完接口，我们会进行接口的自测。一般情况下，我们先启动项目，然后使用 Postman、curl、浏览器，手工模拟请求后端 API 接口。

实际上，SpringMVC 提供了测试框架 MockMvc，方便我们快速测试接口。下面，我们对`UserController`提供的接口，进行下单元测试。

MockMvc 提供了集成测试和单元测试的能力。
## 集成测试
创建`UserControllerTest`测试类，我们来测试一下简单的`UserController`的每个操作。核心代码如下：
```java
// UserControllerTest.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
@AutoConfigureMockMvc
public class UserControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void testList() throws Exception {
        // 查询用户列表
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/users"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().json("[\n" +
                "    {\n" +
                "        \"id\": 1,\n" +
                "        \"username\": \"yudaoyuanma\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 2,\n" +
                "        \"username\": \"woshiyutou\"\n" +
                "    },\n" +
                "    {\n" +
                "        \"id\": 3,\n" +
                "        \"username\": \"chifanshuijiao\"\n" +
                "    }\n" +
                "]")); // 响应结果
    }

    @Test
    public void testGet() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/users/1"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().json("{\n" +
                "\"id\": 1,\n" +
                "\"username\": \"username:1\"\n" +
                "}")); // 响应结果
    }

    @Test
    public void testAdd() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.post("/users")
            .param("username", "yudaoyuanma")
            .param("passowrd", "nicai"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().string("1")); // 响应结果
    }

    @Test
    public void testUpdate() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.put("/users/1")
                .param("username", "yudaoyuanma")
                .param("passowrd", "nicai"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().string("true")); // 响应结果
    }

    @Test
    public void testDelete() throws Exception {
        // 获得指定用户编号的用户
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.delete("/users/1"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().string("false")); // 响应结果
    }

}
```
在类上，我们添加了`@AutoConfigureMockMvc`注解，用于自动化配置我们稍后注入的`MockMvc Bean`对象`mvc`。在后续的测试中，我们会看到都是通过`mvc`调用后端 API 接口。而每一次调用后端 API 接口，都会执行真正的后端逻辑。因此，整个逻辑，走的是集成测试，会启动一个真实的 Spring 环境。

每次 API 接口的请求，都通过`MockMvcRequestBuilders`来构建。构建完成后，通过`mvc`执行请求，返回`ResultActions`结果。

执行完请求后，通过调用`ResultActions`的`andExpect(ResultMatcher matcher)`方法，添加对结果的预期，相当于做断言。如果不符合预期，则会抛出异常，测试不通过。
另外，`ResultActions`还有两个方法：

`andDo(ResultHandler handler)`方法，添加`ResultHandler`结果处理器，例如说调试时打印结果到控制台，人肉看看结果是否正确。
`andReturn()`方法，最后返回相应的 MvcResult 结果。后续，自己针对 MvcResult 写一些自定义的逻辑。
例如说，我们将 #testGet() 方法，使用上述介绍的两个，进行补充改写如下：
```java
// UserControllerTest.java

@Test
public void testGet2() throws Exception {
    // 获得指定用户编号的用户
    ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/users/1"));
    // 校验结果
    resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
    resultActions.andExpect(MockMvcResultMatchers.content().json("{\n" +
            "\"id\": 1,\n" +
            "\"username\": \"username:1\"\n" +
            "}")); // 响应结果

    // <1> 打印结果
    resultActions.andDo(MockMvcResultHandlers.print());

    // <2> 获得 MvcResult ，后续执行各种自定义逻辑
    MvcResult mvcResult = resultActions.andReturn();
    System.out.println("拦截器数量：" + mvcResult.getInterceptors().length);
}
```
<1> 处，打印请求和响应信息。输出如下：
```
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /users/1
       Parameters = {}
          Headers = []
             Body = null
    Session Attrs = {}

Handler:
             Type = cn.iocoder.springboot.lab23.springmvc.controller.UserController
           Method = public cn.iocoder.springboot.lab23.springmvc.vo.UserVO cn.iocoder.springboot.lab23.springmvc.controller.UserController.get(java.lang.Integer)

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json;charset=UTF-8"]
     Content type = application/json;charset=UTF-8
             Body = {"id":1,"username":"username:1"}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```
<2> 处，获得`MvcResult`后，打印下拦截器的数量。输出如下：
```
拦截器数量：2
```
## 单元测试
为了更好的展示 SpringMVC 单元测试的示例，我们需要改写 UserController 的代码，让其会依赖 UserService 。修改点如下：

在 cn.iocoder.springboot.lab23.springmvc.service 包路径下，创建 UserService 类。代码如下：

// UserService.java

@Service
public class UserService {

    public UserVO get(Integer id) {
        return new UserVO().setId(id).setUsername("test");
    }

}
在 UserController 类中，增加 GET /users/v2/{id} 接口，获得指定用户编号的用户。代码如下：

// UserController.java

@Autowired
private UserService userService;

/**
 * 获得指定用户编号的用户
 *
 * @param id 用户编号
 * @return 用户
 */
@GetMapping("/v2/{id}")
public UserVO get2(@PathVariable("id") Integer id) {
    return userService.get(id);
}
在代码中，我们注入了 UserService Bean 对象 userService ，然后在新增的接口方法中，会调用 UserService#get(Integer id) 方法，获得指定用户编号的用户。
创建 UserControllerTest2 测试类，我们来测试一下简单的 UserController 的新增的这个 API 操作。代码如下：

// UserControllerTest2.java

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
public class UserControllerTest2 {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private UserService userService;

    @Test
    public void testGet2() throws Exception {
        // Mock UserService 的 get 方法
        System.out.println("before mock:" + userService.get(1)); // <1.1>
        Mockito.when(userService.get(1)).thenReturn(
                new UserVO().setId(1).setUsername("username:1")); // <1.2>
        System.out.println("after mock:" + userService.get(1)); // <1.3>

        // 查询用户列表
        ResultActions resultActions = mvc.perform(MockMvcRequestBuilders.get("/users/v2/1"));
        // 校验结果
        resultActions.andExpect(MockMvcResultMatchers.status().isOk()); // 响应状态码 200
        resultActions.andExpect(MockMvcResultMatchers.content().json("{\n" +
                "    \"id\": 1,\n" +
                "    \"username\": \"username:1\"\n" +
                "}")); // 响应结果
    }

}
在类上添加 @WebMvcTest 注解，并且传入的是 UserController 类，表示我们要对 UserController 进行单元测试。

同时，@WebMvcTest 注解，是包含了 @AutoConfigureMockMvc 的组合注解，所以它会自动化配置我们稍后注入的 MockMvc Bean 对象 mvc 。在后续的测试中，我们会看到都是通过 mvc 调用后端 API 接口。但是！每一次调用后端 API 接口，并不会执行真正的后端逻辑，而是走的 Mock 逻辑。也就是说，整个逻辑，走的是单元测试，只会启动一个 Mock 的 Spring 环境。

艿艿：注意上面每个加粗的地方！

userService 属性，我们添加了 @MockBean 注解，实际这里注入的是一个使用 Mockito 创建的 UserService Mock 代理对象。如下图所示：UserService Mock 代理对象

UserController 中，也会注入一个 UserService 属性，此时注入的就是该 Mock 出来的 UserService Bean 对象。

默认情况下，

<1.1> 处，我们调用 UserService#get(Integer id) 方法，然后打印返回结果。执行结果如下：

before mock:null
结果竟然返回的是 null 空。理论来说，此时应该返回一个 id = 1 的 UserVO 对象。实际上，因为此时的 userService 是通过 Mockito 来 Mock 出来的对象，其所有调用它的方法，返回的都是空。
<1.2> 处，通过 Mockito 进行 Mock userService 的 #get(Integer id) 方法，当传入的 id = 1 方法参数时，返回 id = 1 并且 username = "username:1" 的 UserVO 对象。

<1.3> 处，再次调用 UserService#get(Integer id) 方法，然后打印返回结果。执行结果如下：

after mock:cn.iocoder.springboot.lab23.springmvc.vo.UserVO@23202c31
打印的就是我们 Mock 返回的 UserVO 对象。
后续，使用 mvc 完成一次后端 API 调用，并进行断言结果是否正确。执行成功，单元测试通过。
# 全局统一返回
示例代码对应仓库：lab-springmvc-23-02 。

在我们提供后端 API 给前端时，我们需要告前端，这个 API 调用结果是否成功：

如果成功，成功的数据是什么。后续，前端会取数据渲染到页面上。
如果失败，失败的原因是什么。一般，前端会将原因弹出提示给用户。
这样，我们就需要有统一的返回结果，而不能是每个接口自己定义自己的风格。一般来说，统一的全局返回信息如下：

成功时，返回成功的状态码 + 数据。
失败时，返回失败的状态码 + 错误提示。
在标准的 RESTful API 的定义，是推荐使用 HTTP 响应状态码 返回状态码。一般来说，我们实践很少这么去做，主要有如下原因：

业务返回的错误状态码很多，HTTP 响应状态码无法很好的映射。例如说，活动还未开始、订单已取消等等。
国内开发者对 HTTP 响应状态码不是很了解，可能只知道 200、403、404、500 几种常见的。这样，反倒增加学习成本。
所以，实际项目在实践时，我们会将状态码放在 Response Body 响应内容中返回。

在全局统一返回里，我们至少需要定义三个字段：

code：状态码。无论是否成功，必须返回。

成功时，状态码为 0 。
失败时，对应业务的错误码。
关于这一块，也有团队实践时，增加了 success 字段，通过 true 和 false 表示成功还是失败。这个看每个团队的习惯吧。艿艿的话，还是偏好基于约定，返回 0 时表示成功。

data：数据。成功时，返回该字段。

message：错误提示。失败时，返回该字段。

那么，让我们来看两个示例：
```
// 成功响应
{
    code: 0,
    data: {
        id: 1,
        username: "yudaoyuanma"
    }
}

// 失败响应
{
    code: 233666,
    message: "徐妈太丑了"
}
```
## CommonResult
在 cn.iocoder.springboot.lab23.springmvc.core.vo 包路径，创建 CommonResult 类，用于全局统一返回。代码如下：
```java
// CommonResult.java

public class CommonResult<T> implements Serializable {

    public static Integer CODE_SUCCESS = 0;

    /**
     * 错误码
     */
    private Integer code;
    /**
     * 错误提示
     */
    private String message;
    /**
     * 返回数据
     */
    private T data;

    /**
     * 将传入的 result 对象，转换成另外一个泛型结果的对象
     *
     * 因为 A 方法返回的 CommonResult 对象，不满足调用其的 B 方法的返回，所以需要进行转换。
     *
     * @param result 传入的 result 对象
     * @param <T> 返回的泛型
     * @return 新的 CommonResult 对象
     */
    public static <T> CommonResult<T> error(CommonResult<?> result) {
        return error(result.getCode(), result.getMessage());
    }

    public static <T> CommonResult<T> error(Integer code, String message) {
        Assert.isTrue(!CODE_SUCCESS.equals(code), "code 必须是错误的！");
        CommonResult<T> result = new CommonResult<>();
        result.code = code;
        result.message = message;
        return result;
    }

    public static <T> CommonResult<T> success(T data) {
        CommonResult<T> result = new CommonResult<>();
        result.code = CODE_SUCCESS;
        result.data = data;
        result.message = "";
        return result;
    }

    @JsonIgnore // 忽略，避免 jackson 序列化给前端
    public boolean isSuccess() { // 方便判断是否成功
        return CODE_SUCCESS.equals(code);
    }

    @JsonIgnore // 忽略，避免 jackson 序列化给前端
    public boolean isError() { // 方便判断是否失败
        return !isSuccess();
    }

    // ... 省略 setting/getting/toString 方法

}
```
## GlobalResponseBodyHandler
在 cn.iocoder.springboot.lab23.springmvc.core.web 包路径，创建 GlobalResponseBodyHandler 类，全局统一返回的处理器。代码如下：
```
// GlobalResponseBodyHandler.java

@ControllerAdvice(basePackages = "cn.iocoder.springboot.lab23.springmvc.controller")
public class GlobalResponseBodyHandler implements ResponseBodyAdvice {

    @Override
    public boolean supports(MethodParameter returnType, Class converterType) {
        return true;
    }

    @Override
    public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class selectedConverterType,
                                  ServerHttpRequest request, ServerHttpResponse response) {
        // 如果已经是 CommonResult 类型，则直接返回
        if (body instanceof CommonResult) {
            return body;
        }
        // 如果不是，则包装成 CommonResult 类型
        return CommonResult.success(body);
    }

}
```
在 SpringMVC 中，可以使用通过实现 ResponseBodyAdvice 接口，并添加 @ControllerAdvice 接口，拦截 Controller 的返回结果。注意，我们这里 @ControllerAdvice 注解，设置了 basePackages 属性，只拦截 "cn.iocoder.springboot.lab23.springmvc.controller" 包，也就是我们定义的 Controller 。为什么呢？因为在项目中，我们可能会引入 Swagger 等库，也使用 Controller 提供 API 接口，那么我们显然不应该让 GlobalResponseBodyHandler 去拦截这些接口，毕竟它们并不需要我们去替它们做全局统一的返回。
实现 #supports(MethodParameter returnType, Class converterType) 方法，返回 true 。表示拦截 Controller 所有 API 接口的返回结果。
实现 #beforeBodyWrite(...) 方法，当返回的结果不是 CommonResult 类型时，则包装成 CommonResult 类型。这里有两点要注意：
第一点，可能 API 接口的返回结果已经是 CommonResult 类型，就无需做二次包装了。
第二点，API 接口既然返回结果，被 GlobalResponseBodyHandler 拦截到，约定就是成功返回，所以使用 CommonResult#success(T data) 方法，进行包装成成功的 CommonResult 返回。那么，如果我们希望 API 接口是失败的返回呢？我们约定在 Controller 抛出异常，这点我们会在 「5. 全局异常处理」 看到。

## UserController
在 cn.iocoder.springboot.lab23.springmvc.controller 包路径下，创建 UserController 类。代码如下：

// UserController.java

@RestController
@RequestMapping("/users")
public class UserController {

    /**
     * 获得指定用户编号的用户
     *
     * 提供不使用 CommonResult 包装
     *
     * @param id 用户编号
     * @return 用户
     */
    @GetMapping("/get")
    public UserVO get(@RequestParam("id") Integer id) {
        // 查询并返回用户
        return new UserVO().setId(id).setUsername(UUID.randomUUID().toString());
    }

    /**
     * 获得指定用户编号的用户
     *
     * 提供使用 CommonResult 包装
     *
     * @param id 用户编号
     * @return 用户
     */
    @GetMapping("/get2")
    public CommonResult<UserVO> get2(@RequestParam("id") Integer id) {
        // 查询用户
        UserVO user = new UserVO().setId(id).setUsername(UUID.randomUUID().toString());
        // 返回结果
        return CommonResult.success(user);
    }

}
在 #get(Integer id) 方法，返回的结果是 UserVO 类型。这样，结果会被 GlobalResponseBodyHandler 拦截，包装成 CommonResult 类型返回。请求结果如下：

{
    "code": 0,
    "message": "",
    "data": {
        "id": 10,
        "username": "f0ab9401-062f-4697-bcc9-1dc70c1c1310"
    }
}
会有 "message": "" 的返回的原因是，我们使用 SpringMVC 提供的 Jackson 序列化，对于 CommonResult 此时的 message = null 的情况下，会序列化它成 "message": "" 返回。实际情况下，不会影响前端处理。
在 #get2(Integer id) 方法，返回的结果是 Common<UserVO> 类型。结果虽然也会被 GlobalResponseBodyHandler 拦截，但是不会二次再重复包装成 CommonResult 类型返回。
# 全局异常处理
示例代码对应仓库：lab-springmvc-23-02 。

在 「4. 全局统一返回」 中，我们已经定义了使用 CommonResult 全局统一返回，并且看到了成功返回的示例与代码。这一小节，我们主要是来全局异常处理，最终能也是通过 CommonResult 返回。
## ServiceExceptionEnum
在 cn.iocoder.springboot.lab23.springmvc.constants 包路径，创建 ServiceExceptionEnum 枚举类，枚举项目中的错误码。代码如下：

// ServiceExceptionEnum.java

public enum ServiceExceptionEnum {

    // ========== 系统级别 ==========
    SUCCESS(0, "成功"),
    SYS_ERROR(2001001000, "服务端发生异常"),
    MISSING_REQUEST_PARAM_ERROR(2001001001, "参数缺失"),

    // ========== 用户模块 ==========
    USER_NOT_FOUND(1001002000, "用户不存在"),

    // ========== 订单模块 ==========

    // ========== 商品模块 ==========
    ;

    /**
     * 错误码
     */
    private int code;
    /**
     * 错误提示
     */
    private String message;

    ServiceExceptionEnum(int code, String message) {
        this.code = code;
        this.message = message;
    }

    // ... 省略 getting 方法

}
因为错误码是全局的，最好按照模块来拆分。如下是艿艿在 onemall 项目的实践：

/**
 * 服务异常
 *
 * 参考 https://www.kancloud.cn/onebase/ob/484204 文章
 *
 * 一共 10 位，分成四段
 *
 * 第一段，1 位，类型
 *      1 - 业务级别异常
 *      2 - 系统级别异常
 * 第二段，3 位，系统类型
 *      001 - 用户系统
 *      002 - 商品系统
 *      003 - 订单系统
 *      004 - 支付系统
 *      005 - 优惠劵系统
 *      ... - ...
 * 第三段，3 位，模块
 *      不限制规则。
 *      一般建议，每个系统里面，可能有多个模块，可以再去做分段。以用户系统为例子：
 *          001 - OAuth2 模块
 *          002 - User 模块
 *          003 - MobileCode 模块
 * 第四段，3 位，错误码
 *       不限制规则。
 *       一般建议，每个模块自增。
 */

## ServiceException
我们在一起讨论下 Service 逻辑异常的时候，如何进行返回。这里的逻辑异常，我们指的是，例如说用户名已经存在，商品库存不足等。一般来说，常用的方案选择，有两种：

封装统一的业务异常类 ServiceException ，里面有错误码和错误提示，然后进行 throws 抛出。
封装通用的返回类 CommonResult ，里面有错误码和错误提示，然后进行 return 返回。
一开始，我们选择了 CommonResult ，结果发现如下情况：

因为 Spring @Transactional 声明式事务，是基于异常进行回滚的，如果使用 CommonResult 返回，则事务回滚会非常麻烦。
当调用别的方法时，如果别人返回的是 CommonResult 对象，还需要不断的进行判断，写起来挺麻烦的。
所以，后来我们采用了抛出业务异常 ServiceException 的方式。

在 cn.iocoder.springboot.lab23.springmvc.core.exception 包路径，创建 ServiceException 异常类，继承 RuntimeException 异常类，用于定义业务异常。代码如下：

// ServiceException.java

public final class ServiceException extends RuntimeException {

    /**
     * 错误码
     */
    private final Integer code;

    public ServiceException(ServiceExceptionEnum serviceExceptionEnum) {
        // 使用父类的 message 字段
        super(serviceExceptionEnum.getMessage());
        // 设置错误码
        this.code = serviceExceptionEnum.getCode();
    }

    // ... 省略 getting 方法

}
提供传入 serviceExceptionEnum 参数的构造方法。具体的处理，看下代码和注释。
## GlobalExceptionHandler
在 cn.iocoder.springboot.lab23.springmvc.core.web 包路径，创建 GlobalExceptionHandler 类，全局统一返回的处理器。代码如下：

// GlobalExceptionHandler.java

@ControllerAdvice(basePackages = "cn.iocoder.springboot.lab23.springmvc.controller")
public class GlobalExceptionHandler {

    private Logger logger = LoggerFactory.getLogger(getClass());

    /**
     * 处理 ServiceException 异常
     */
    @ResponseBody
    @ExceptionHandler(value = ServiceException.class)
    public CommonResult serviceExceptionHandler(HttpServletRequest req, ServiceException ex) {
        logger.debug("[serviceExceptionHandler]", ex);
        // 包装 CommonResult 结果
        return CommonResult.error(ex.getCode(), ex.getMessage());
    }

    /**
     * 处理 MissingServletRequestParameterException 异常
     *
     * SpringMVC 参数不正确
     */
    @ResponseBody
    @ExceptionHandler(value = MissingServletRequestParameterException.class)
    public CommonResult missingServletRequestParameterExceptionHandler(HttpServletRequest req, MissingServletRequestParameterException ex) {
        logger.debug("[missingServletRequestParameterExceptionHandler]", ex);
        // 包装 CommonResult 结果
        return CommonResult.error(ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getCode(),
                ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR.getMessage());
    }

    /**
     * 处理其它 Exception 异常
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public CommonResult exceptionHandler(HttpServletRequest req, Exception e) {
        // 记录异常日志
        logger.error("[exceptionHandler]", e);
        // 返回 ERROR CommonResult
        return CommonResult.error(ServiceExceptionEnum.SYS_ERROR.getCode(),
                ServiceExceptionEnum.SYS_ERROR.getMessage());
    }

}
在类上，添加 @ControllerAdvice 注解。这一点，和 「4.4 GlobalResponseBodyHandler」 是一样的。不过，不会实现 ResponseBodyAdvice 接口，因为咱不需要拦截接口返回结果，进行修改。
我们定义了三个方法，通过添加 @ExceptionHandler 注解，定义每个方法对应处理的异常。并且，也添加了 @ResponseBody 注解，标记直接使用返回结果作为 API 的响应。
#serviceExceptionHandler(...) 方法，拦截处理 ServiceException 业务异常，直接使用该异常的 code + message 属性，构建出 CommonResult 对象返回。
#missingServletRequestParameterExceptionHandler(...) 方法，拦截处理 MissingServletRequestParameterException 请求参数异常，构建出错误码为 ServiceExceptionEnum.MISSING_REQUEST_PARAM_ERROR 的 CommonResult 对象返回。
#exceptionHandler(...) 方法，拦截处理 Exception 异常，构建出错误码为 ServiceExceptionEnum.SYS_ERROR 的 CommonResult 对象返回。这是一个兜底的异常处理，避免有一些其它异常，我们没有在 GlobalExceptionHandler 中，提供自定义的处理方式。
注意，在 #exceptionHandler(...) 方法中，我们还多使用 logger 打印了错误日志，方便我们接入 ELK 等日志服务，发起告警，通知我们去排查解决。如果胖友的系统里暂时没有日志服务，可以记录错误日志到数据库中，也是不错的选择。而其它两个方法，因为是更偏业务的，相对正常的异常，所以无需记录错误日志。
## UserController
在 UserController 类中，我们添加两个 API 接口，抛出异常，方便我们测试全局异常处理的效果。代码如下：

// UserController.java

/**
 * 测试抛出 NullPointerException 异常
 */
@GetMapping("/exception-01")
public UserVO exception01() {
    throw new NullPointerException("没有粗面鱼丸");
}

/**
 * 测试抛出 ServiceException 异常
 */
@GetMapping("/exception-02")
public UserVO exception02() {
    throw new ServiceException(ServiceExceptionEnum.USER_NOT_FOUND);
}
在 #exception01() 方法，抛出 NullPointerException 异常。这样，异常会被 GlobalExceptionHandler#exceptionHandler(...) 方法来拦截，包装成 CommonResult 类型返回。请求结果如下：

{
    "code": 2001001000,
    "message": "服务端发生异常",
    "data": null
}
在 #exception02() 方法，抛出 ServiceException 异常。这样，异常会被 GlobalExceptionHandler#serviceExceptionHandler(...) 方法来拦截，包装成 CommonResult 类型返回。请求结果如下：

{
    "code": 1001002000,
    "message": "用户不存在",
    "data": null
}

# HandlerInterceptor 拦截器
示例代码对应仓库：lab-springmvc-23-02 。

在使用 SpringMVC 的时候，我们可以使用 HandlerInterceptor ，拦截 SpringMVC 处理请求的过程，自定义前置和处理的逻辑。例如说：

日志拦截器，记录请求与响应。这样，我们可以知道每一次请求的参数，响应的结果，执行的时长等等信息。
认证拦截器，我们可以解析前端传入的用户标识，例如说 access_token 访问令牌，获得当前用户的信息，记录到 ThreadLocal 中。这样，后续的逻辑，只需要通过 ThreadLocal 就可以获取到用户信息。
授权拦截器，我们可以通过每个 API 接口需要的授权信息，进行判断，当前请求是否允许访问。例如说，用户是否登录，是否有该 API 操作的权限等等。
限流拦截器，我们可以通过每个 API 接口的限流配置，进行判断，当前请求是否超过允许的请求频率，避免恶意的请求，打爆整个系统。
HandlerInterceptor 接口，定义了三个拦截点。代码如下：

// HandlerInterceptor.java

public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}

}
首先，我们要普及一个概念。我们的每一个 API 请求，会对应到一个 handler 处理器。如下图所示：

我们可以看到，这一次 users/exception_03 请求，handler 对应上了 UserController 的 #exception03() 方法。
也因此，HandlerInterceptor 的接口名上，是以 Handler 开头，基于 Handler 的拦截器。
然后，我们先来看一段伪代码，看看这三个拦截点和 handler 的执行过程，是怎么结合的。代码如下：

// 伪代码
Exception ex = null;
try {
    // 前置处理
    if (!preHandle(request, response, handler)) {
        return;
    }

    // 执行处理器，即执行 API 的逻辑
    handler.execute();

    // 后置处理
    postHandle(request, response, handler);
} catch(Exception exception) {
    // 如果发生了异常，记录到 ex 中
    ex = exception;
} finally {
    afterCompletion(request, response, handler);
}
结合艿艿的代码注释，胖友理解下整个处理的过程。
当然，这里仅仅是伪代码，并没有考虑多个拦截器的情况下。我们会在下文中，提供具体的代码示例，方便胖友更加深入的理解整个执行过程。
#preHandle(...) 方法，实现 handler 的前置处理逻辑。当返回 true 时，继续后续 handler 的执行；当返回 false 时，不进行后续 handler 的执行。

例如说，判断用户是否已经登录，如果未登录，返回 false ，不进行后续 handler 的执行。

#postHandle(...) 方法，实现 handler 的后置处理逻辑。

例如说，在视图 View 在渲染之前，做一些处理。不过因为目前都前后端分离，所以这个后置拦截点，使用的就已经比较少了。

#afterCompletion(...) 方法，整个 handler 执行完成，并且拦截器链都执行完前置和后置的拦截逻辑，实现请求完成后的处理逻辑。注意，只有 #preHandle(...) 方法返回 true 的 HandlerInterceptor 拦截器，才能执行 #afterCompletion(...) 方法，因为这样要算 HandlerInterceptor 执行完成才有效。

例如说，释放资源。比如，清理认证拦截器产生的 ThreadLocal 线程变量，避免“污染”下一个使用到该线程的请求。

又例如说，处理 handler 执行过程中发生的异常，并记录异常日志。不过因为现在一般通过 「5. 全局异常处理」 来处理，所以很少这么做了。

再例如说，记录请求结束时间，这样我们就可以计算出整个请求的耗时。

在上文中，考虑到让 HandlerInterceptor 的理解更加简洁，一直漏说了一个概念，多个 HandlerInterceptor 们，可以组成一个 Chain 拦截器链。那么，整个执行的过程，就变成：

首先，按照 HandlerInterceptor 链的正序，执行 #preHandle(...) 方法。
然后，执行 handler 的逻辑处理。
之后，按照 HandlerInterceptor 链的倒序，执行 #postHandle(...) 方法。
最后，按照 HandlerInterceptor 链的倒序，执行 #afterCompletion(...) 方法。
这里，我们先只说了正常执行的情况。那么异常执行的情况呢？例如说：

某个 HandlerInterceptor 执行 #preHandle(...) 方法，返回 false 的情况。
handler 的逻辑处理，抛出 Exception 异常的情况。
某个 HandlerInterceptor 执行 #afterCompletion(...) 方法，抛出 Exception 异常的情况。
... 等等

## 自定义 HandlerInterceptor
在 cn.iocoder.springboot.lab23.springmvc.core.interceptor 包路径，我们创建三个自定义 HandlerInterceptor 拦截器。

FirstInterceptor 代码如下：

// FirstInterceptor.java

public class FirstInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        logger.info("[preHandle][handler({})]", handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("[postHandle][handler({})]", handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("[afterCompletion][handler({})]", handler, ex);
    }

}
每个方法中，打印日志。
SecondInterceptor 代码如下：

// SecondInterceptor.java

public class SecondInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        logger.info("[preHandle][handler({})]", handler);
        return false; // 故意返回 false
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("[postHandle][handler({})]", handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("[afterCompletion][handler({})]", handler, ex);
    }

}
和 「FirstInterceptor」 基本一致，差别在于 #preHandle(...) 方法，返回 false 。
ThirdInterceptor 代码如下：

// ThirdInterceptor.java

public class ThirdInterceptor implements HandlerInterceptor {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        logger.info("[preHandle][handler({})]", handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("[postHandle][handler({})]", handler);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("[afterCompletion][handler({})]", handler, ex);
        throw new RuntimeException("故意抛个错误"); // 故意抛出异常
    }

}
和 「FirstInterceptor」 基本一致，差别在于 #afterCompletion(...) 方法，抛出 RuntimeException 异常。
## SpringMVCConfiguration
在 cn.iocoder.springboot.lab23.springmvc.config 包路径下，创建 SpringMVCConfiguration 配置类。代码如下：

// SpringMVCConfiguration.java

@Configuration
public class SpringMVCConfiguration implements WebMvcConfigurer {

    @Bean
    public FirstInterceptor firstInterceptor() {
        return new FirstInterceptor();
    }

    @Bean
    public SecondInterceptor secondInterceptor() {
        return new SecondInterceptor();
    }

    @Bean
    public ThirdInterceptor thirdInterceptor() {
        return new ThirdInterceptor();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 拦截器一
        registry.addInterceptor(this.firstInterceptor()).addPathPatterns("/**");
        // 拦截器二
        registry.addInterceptor(this.secondInterceptor()).addPathPatterns("/users/current_user");
        // 拦截器三
        registry.addInterceptor(this.thirdInterceptor()).addPathPatterns("/**");
    }

}
该配置类，实现 WebMvcConfigurer 接口，实现 SpringMVC 的自定义配置。当然，类上还是要加上 @Configuration 注解，表明 SpringMVCConfiguration 是个配置类。
#addInterceptors(InterceptorRegistry registry) 方法，添加自定义的 HandlerInterceptor 拦截器，到 InterceptorRegistry 拦截器注册表中。其中，SecondInterceptor 拦截器，我们配置拦截的是 /users/current_user 路径，用于测试 SecondInterceptor#preHandle(...) 前置拦截返回 false 的情况。

## UserController
本小节，我们会在 UserController 接口，测试拦截器链的表现。

① /users/do_something 接口

// UserController.java

@GetMapping("/do_something")
public void doSomething() {
    logger.info("[doSomething]");
}
调用该接口，执行日志如下：

// 首先，按照 HandlerInterceptor 链的**正序**，执行 `#preHandle(...)` 方法。
2019-11-17 12:31:38.049  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.c.i.FirstInterceptor           : [preHandle][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.doSomething())]
2019-11-17 12:31:38.050  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.c.i.ThirdInterceptor           : [preHandle][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.doSomething())]
2019-11-17 12:31:38.055  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.controller.UserController      :

// 然后，执行 `handler` 的逻辑处理。
[doSomething]

// 之后，按照 HandlerInterceptor 链的**倒序**，执行 `#postHandle(...)` 方法。
2019-11-17 12:31:38.109  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.c.i.ThirdInterceptor           : [postHandle][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.doSomething())]
2019-11-17 12:31:38.109  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.c.i.FirstInterceptor           : [postHandle][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.doSomething())]

// 最后，按照 HandlerInterceptor 链的**倒序**，执行 `#afterCompletion(...)` 方法。
2019-11-17 12:31:38.109  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.c.i.ThirdInterceptor           : [afterCompletion][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.doSomething())]
java.lang.RuntimeException: 故意抛个错误 // ... 省略异常堆栈

2019-11-17 12:31:38.116  INFO 28157 --- [nio-8080-exec-1] c.i.s.l.s.c.i.FirstInterceptor           : [afterCompletion][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.doSomething())]
整个执行日志，看下艿艿添加在其上的注释。
因为 SecondInterceptor 拦截的是 /users/current_user 路径，所以它没有拦截本次 API 调用。
注意，虽然说，ThirdInterceptor 在 #afterCompletion(...) 方法中，抛出异常，但是不影响 FirstInterceptor 的 #afterCompletion(...) 的后续执行。
② /users/current_user 接口

// UserController.java

@GetMapping("/current_user")
public UserVO currentUser() {
    logger.info("[currentUser]");
    return new UserVO().setId(10).setUsername(UUID.randomUUID().toString());
}
调用该接口，执行日志如下：

// 首先，按照 HandlerInterceptor 链的**正序**，执行 `#preHandle(...)` 方法。
2019-11-17 12:48:37.357  INFO 28157 --- [nio-8080-exec-5] c.i.s.l.s.c.i.FirstInterceptor           : [preHandle][handler(public cn.iocoder.springboot.lab23.springmvc.vo.UserVO cn.iocoder.springboot.lab23.springmvc.controller.UserController.currentUser())]
2019-11-17 12:48:37.357  INFO 28157 --- [nio-8080-exec-5] c.i.s.l.s.c.i.SecondInterceptor          : [preHandle][handler(public cn.iocoder.springboot.lab23.springmvc.vo.UserVO cn.iocoder.springboot.lab23.springmvc.controller.UserController.currentUser())]

//【不存在】然后，执行 `handler` 的逻辑处理。

//【不存在】之后，按照 HandlerInterceptor 链的**倒序**，执行 `#postHandle(...)` 方法。

// 最后，按照 HandlerInterceptor 链的**倒序**，执行 `#afterCompletion(...)` 方法。
2019-11-17 12:48:37.358  INFO 28157 --- [nio-8080-exec-5] c.i.s.l.s.c.i.FirstInterceptor           : [afterCompletion][handler(public cn.iocoder.springboot.lab23.springmvc.vo.UserVO cn.iocoder.springboot.lab23.springmvc.controller.UserController.currentUser())]
因为只有 FirstInterceptor 完成了 #preHandle(...) 方法的执行，所以也只有 FirstInterceptor 的 #afterCompletion(...) 方法被执行。
在 handler 未执行逻辑处理的情况下，HandlerInterceptor 的 #postHandle(...) 方法不会执行。答案也很显而易见，handler 丫根就没完成执行，何来的后置处理，嘿嘿。
③ /users/exception-03 接口

// UserController.java

@GetMapping("/exception-03")
public void exception03() {
    logger.info("[exception03]");
    throw new ServiceException(ServiceExceptionEnum.USER_NOT_FOUND);
}
调用该接口，执行日志如下：

// 首先，按照 HandlerInterceptor 链的**正序**，执行 `#preHandle(...)` 方法。
2019-11-17 12:54:45.029  INFO 28157 --- [nio-8080-exec-7] c.i.s.l.s.c.i.FirstInterceptor           : [preHandle][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.exception03())]
2019-11-17 12:54:45.029  INFO 28157 --- [nio-8080-exec-7] c.i.s.l.s.c.i.ThirdInterceptor           : [preHandle][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.exception03())]

// 然后，执行 `handler` 的逻辑处理。
2019-11-17 12:54:45.029  INFO 28157 --- [nio-8080-exec-7] c.i.s.l.s.controller.UserController      : [exception03]

//【不存在】之后，按照 HandlerInterceptor 链的**倒序**，执行 `#postHandle(...)` 方法。

// 最后，按照 HandlerInterceptor 链的**倒序**，执行 `#afterCompletion(...)` 方法。
2019-11-17 12:54:45.036  INFO 28157 --- [nio-8080-exec-7] c.i.s.l.s.c.i.ThirdInterceptor           : [afterCompletion][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.exception03())]
2019-11-17 12:54:45.037 ERROR 28157 --- [nio-8080-exec-7] o.s.web.servlet.HandlerExecutionChain    : HandlerInterceptor.afterCompletion threw exception
java.lang.RuntimeException: 故意抛个错误 // ... 省略异常堆栈

2019-11-17 12:54:45.037  INFO 28157 --- [nio-8080-exec-7] c.i.s.l.s.c.i.FirstInterceptor           : [afterCompletion][handler(public void cn.iocoder.springboot.lab23.springmvc.controller.UserController.exception03())]
handler 的逻辑处理，抛出 Exception 异常的情况，HandlerInterceptor 的 #postHandle(...) 方法不会执行。为什么呢？答案我们已经在 /users/current_user 接口中，进行了说明。
# Servlet、Filter、Listener
虽然绝大多数情况下，我们无需在 SpringMVC 中，直接使用 java.servlet 提供的 Servlet、Filter、Listener 了，但是在使用一些三方类库时，它们更多的提供的是 java.servlet 中的组件，毕竟它们需要提供更加公用的组件，而不能和 SpringMVC 耦合在一起。

例如说，在使用 Shiro 做权限认证相关方面的功能时，我们就需要配置 Shiro 提供的 ShiroFilterFactoryBean 。

我们有两种方式，使用 Java 代码的方式，配置 Servlet、Filter、Listener 。分别是：

通过 Bean 的方式
通过注解的方式
## 通过 Bean 的方式
在 SpringMVCConfiguration 配置类中，我们来添加 Servlet、Filter、Listener 三个 Bean 的配置。代码如下：

// SpringMVCConfiguration.java

@Bean
public ServletRegistrationBean testServlet01() {
    ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean<>(new HttpServlet() {

        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            logger.info("[doGet][uri: {}]", req.getRequestURI());
        }

    });
    servletRegistrationBean.setUrlMappings(Collections.singleton("/test/01"));
    return servletRegistrationBean;
}

@Bean
public FilterRegistrationBean testFilter01() {
    FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean<>(new Filter() {

        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            logger.info("[doFilter]");
            filterChain.doFilter(servletRequest, servletResponse);
        }

    });
    filterRegistrationBean.setUrlPatterns(Collections.singleton("/test/*"));
    return filterRegistrationBean;
}

@Bean
public ServletListenerRegistrationBean<?> testListener01() {
    return new ServletListenerRegistrationBean<>(new ServletContextListener() {

        @Override
        public void contextInitialized(ServletContextEvent sce) {
            logger.info("[contextInitialized]");
        }

        @Override
        public void contextDestroyed(ServletContextEvent sce) {
        }

    });
}
在 Spring Boot 中，提供了 ServletRegistrationBean 来配置 Servlet Bean、FilterRegistrationBean 来配置 Filter Bean、ServletListenerRegistrationBean 来配置 Listener Bean 。
这里，我们为了让示例比较简洁，所以采用了内部类。实际在使用时，还是正常去定义类，嘿嘿。

## 通过注解的方式
在 Servlet3.0 的新特性里，提供了 @WebServlet、@WebFilter、@WebListener 三个注解，方便配置 Servlet、Filter、Listener 。

而在 SpringBoot 中，我们仅需要在 Application 类上，添加 @ServletComponentScan 注解，开启对 @WebServlet、@WebFilter、@WebListener 注解的扫描。不过要注意，当且仅当使用内嵌的 Web Server 才会生效。

在 cn.iocoder.springboot.lab23.springmvc.core.servlet 包路径，我们创建了三个示例。代码如下：

// TestServlet02.java
@WebServlet(urlPatterns = "/test/02")
public class TestServlet02 extends HttpServlet {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        logger.info("[doGet][uri: {}]", req.getRequestURI());
    }

}

// TestFilter02.java
@WebFilter("/test/*")
public class TestFilter02 implements Filter {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        logger.info("[doFilter]");
        filterChain.doFilter(servletRequest, servletResponse);
    }

}

// TestServletContextListener02.java
@WebListener
public class TestServletContextListener02 implements ServletContextListener {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    public void contextInitialized(ServletContextEvent sce) {
        logger.info("[contextInitialized]");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
    }

}
# Cors 跨域
示例代码对应仓库：lab-springmvc-23-02 。

在前后端分离之后，我们会碰到跨域的问题。例如说，前端在 http://www.iocoder.cn 域名下，而后端 API 在 http://api.iocoder.cn 域名下。

对跨域不是很了解的胖友，可以看看阮大的 《跨域资源共享 CORS 详解》 文章。😈 当然，也可以先继续本文的阅读。

解决跨域的方式有很多，例如说，在 Nginx 上配置处理跨域请求的参数。又例如说，项目中有网关服务，统一配置处理。当然，本文既然是 Spring Boot SpringMVC 入门，那么必然只使用 SpringMVC 来解决跨域。目前一共有三种方案：

方式一，使用 @CrossCors 注解，配置每个 API 接口。
方式二，使用 CorsRegistry.java 注册表，配置每个 API 接口。
方案三，使用 CorsFilter.java 过滤器，处理跨域请求。
其中，方案一和方案二，本质是相同的方案，只是配置方式不同。想要理解底层原理的胖友，可以看看 CorsInterceptor 拦截器。
##  @CrossCors
@CrossCors 注解，添加在类或方法上，标记该类/方法对应接口的 Cors 信息。

@CrossCors 注解的常用属性，如下：

origins 属性，设置允许的请求来源。[] 数组，可以填写多个请求来源。默认值为 * 。
value 属性，和 origins 属性相同，是它的别名。
allowCredentials 属性，是否允许客户端请求发送 Cookie 。默认为 false ，不允许请求发送 Cookie 。
maxAge 属性，本次预检请求的有效期，单位为秒。默认值为 1800 秒。
@CrossCors 注解的不常用属性，如下：

methods 属性，设置允许的请求方法。[] 数组，可以填写多个请求方法。默认值为 GET + POST 。
allowedHeaders 属性，允许的请求头 Header 。[] 数组，可以填写多个请求来源。默认值为 * 。
exposedHeaders 属性，允许的响应头 Header 。[] 数组，可以填写多个请求来源。默认值为 * 。
一般情况下，我们在每个 Controller 上，添加 @CrossCors 注解即可。当然，如果某个 API 接口希望做自定义的配置，可以在 Method 方法上添加。示例如下：

// TestController.java

@RestController
@RequestMapping("/test")
@CrossOrigin(origins = "*", allowCredentials = "true") // 允许所有来源，允许发送 Cookie
public class TestController {

    /**
     * 获得指定用户编号的用户
     *
     * @return 用户
     */
    @GetMapping("/get")
    @CrossOrigin(allowCredentials = "false") // 允许所有来源，不允许发送 Cookie
    public UserVO get() {
        return new UserVO().setId(1).setUsername(UUID.randomUUID().toString());
    }

}
在绝大数场合下，肯定是在 Controller 上，添加 @CrossOrigin(allowCredentials = "true") 即可。

坑坑坑坑坑

在前端使用符合 CORS 规范的网络库时，例如说 Vue 常用的网络库 axios ，在发起非简单请求时，会自动先先发起 OPTIONS “预检”请求，要求服务器确认是否能够这样请求。这样，这个请求就会被 SpringMVC 的拦截器所处理。

此时，如果我们的拦截器认为 handler 一定是 HandlerMethod 类型时，就会导致报错。例如说，艿艿在 UserSecurityInterceptor 拦截器中，会认为 handler 是 HandlerMethod 类型，然后通过它获得 @RequiresLogin 注解信息，判断是否需要登录。然后，实际上，此时 handler 是 PreFlightHandler 类型，则会导致抛出异常。如下图所示：getCorsHandlerExecutionChain

此时，有两种解决方案：

1）检查每个拦截器的实现，是不是依赖于 handler 是 HandlerMethod 的逻辑，进行修复。
2）不使用该方案，而是采用 「8.3 CorsFilter」 过滤器，避免 OPTIONS 预检查走到拦截器里。
显然，1） 的成本略微有点高，所以一般情况下，推荐 2） 。目前艿艿的项目，也是采用 「8.3 CorsFilter」 过滤器的方式。

😈 在 《【SpringMVC】与权限拦截器冲突导致的 Cors 跨域设置失效问题》 文章中，我们也可以看到有开发者，碰到一样的问题。
## CorsRegistry
显然，在每个 Controller 上配置 @CrossOrigin 注解，是挺麻烦一事。所以更多的情况下，我们会选择配置 CorsRegistry 注册表。

修改 SpringMVCConfiguration 配置类，增加 CorsRegistry 相关的配置。代码如下：

// SpringMVCConfiguration.java

@Override
public void addCorsMappings(CorsRegistry registry) {
    // 添加全局的 CORS 配置
    registry.addMapping("/**") // 匹配所有 URL ，相当于全局配置
            .allowedOrigins("*") // 允许所有请求来源
            .allowCredentials(true) // 允许发送 Cookie
            .allowedMethods("*") // 允许所有请求 Method
            .allowedHeaders("*") // 允许所有请求 Header
//                .exposedHeaders("*") // 允许所有响应 Header
            .maxAge(1800L); // 有效期 1800 秒，2 小时
}
这里配置匹配的路径是 /** ，从而实现全局 CORS 配置。
如果想要配置单个路径的 CORS 配置，可以通过 CorsRegistry#addMapping(String pathPattern) 方法，继续往其中添加 CORS 配置。
如果胖友想要更安全，可以 originns 属性，只填写允许的前端域名地址。
这种方式，一样会存在 「8.1 @CrossCors」 提供到的坑坑坑坑坑，因为这两者的实现方式是一致的。所以，继续看 「8.3 CorsFilter」 方案。
## CorsFilter
在 Spring Web 中，内置提供 CorsFilter 过滤器，实现对 CORS 的处理。

配置方式很简单，既然是 Filter 过滤器，就可以采用 「7.1 通过 Bean 的方式」 ，进行配置。所以修改 SpringMVCConfiguration 配置类，增加 CorsFilter 相关的配置。代码如下：

// SpringMVCConfiguration.java

@Bean
public FilterRegistrationBean<CorsFilter> corsFilter() {
    // 创建 UrlBasedCorsConfigurationSource 配置源，类似 CorsRegistry 注册表
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    // 创建 CorsConfiguration 配置，相当于 CorsRegistration 注册信息
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(Collections.singletonList("*")); // 允许所有请求来源
    config.setAllowCredentials(true); // 允许发送 Cookie
    config.addAllowedMethod("*"); // 允许所有请求 Method
    config.setAllowedHeaders(Collections.singletonList("*")); // 允许所有请求 Header
    // config.setExposedHeaders(Collections.singletonList("*")); // 允许所有响应 Header
    config.setMaxAge(1800L); // 有效期 1800 秒，2 小时
    source.registerCorsConfiguration("/**", config);
    // 创建 FilterRegistrationBean 对象
    FilterRegistrationBean<CorsFilter> bean = new FilterRegistrationBean<>(
            new CorsFilter(source)); // 创建 CorsFilter 过滤器
    bean.setOrder(0); // 设置 order 排序。这个顺序很重要哦，为避免麻烦请设置在最前
    return bean;
}
艿艿已经添加了详细的注释，胖友自己看下噢。效果上，和 「8.2 CorsRegistry」 是一致的。
至此，我们已经学习完三种 SpringMVC 配置 CORS 的方式。**结论就是，使用 「8.3 CorsFilter」 方式。**😈 感觉自己越来越啰嗦了。最后在推荐一篇文章，感觉写的不错 《Spring 里那么多种 CORS 的配置方式，到底有什么区》 。
# HttpMessageConverter 消息转换器
示例代码对应仓库：lab-springmvc-23-02 。

在 Spring MVC 中，可以使用 @RequestBody 和 @ResponseBody 两个注解，分别完成请求报（内容）到对象和**对象到响应报文（内容）**的转换，底层这种灵活的消息转换机制，就是 Spring 3.x 中新引入的 HttpMessageConverter ，即消息转换器机制。

这么解释，胖友肯定是不会买单的。😜 在上面的示例里，我们已经看到，我们明明返回的是 UserVO 对象，最后输出给前端时，变成了 JSON 字符串，这就是使用了 MappingJackson2HttpMessageConverter 消息转换器，将 UserVO 对象转换成 JSON 字符串，写回给前端。😈 现在是不是有点理解了？！

在一些业务场景下，前端提交给后端 API 参数，比较复杂，那么可能我们希望能够使用 JSON 的格式，提交给后端 API 接口。此时，我们又可以使用 MappingJackson2HttpMessageConverter 消息转换器，将 JSON 字符串，转换成对应的对象。😈 现在是不是更加清晰了!？

让我们再来一起看看 HttpMessageConverter 消息转换器接口，代码如下：

// HttpMessageConverter.java

// 是否能够读取指定的 mediaType 内容类型，转换成对应的 clazz 对象
boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
// 读取请求内容，转换成 clazz 对象
T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

// 是否能够将 clazz 对象，序列化成 mediaType 内容类型
boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
// 将 clazz 对象，序列化成 contentType 内容类型，写入到响应。
void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

// 获得 HttpMessageConverter 能够支持的内容类型。
List<MediaType> getSupportedMediaTypes();
在请求时，我们在请求头 Content-Type 上，表示请求内容（Request Body）的内容类型。这样，SpringMVC 会从自己的 HttpMessageConverter 数组中，通过 #canRead(clazz, mediaType) 方法，判断是否够读取指定的 mediaType 内容类型，转换成对应的 clazz 对象。如果可以，则调用 #read(Class<? extends T> clazz, HttpInputMessage inputMessage) 方法，读取请求内容，转换成 clazz 对象。
在响应时，我们在请求头 Accept 上，表示请求内容（Response Body）的内容类型。这样，SpringMVC 会从自己的 HttpMessageConverter 数组中，通过 #canWrite(clazz, mediaType) 方法，判断是否能够将 clazz 对象，序列化成 mediaType 内容类型。如果可以，则调用 #write(contentType, outputMessage) 方法， 将 clazz 对象，序列化成 contentType 内容类型，写入到响应。
如果还是不理解，我们来结合示例，一起来理解下。我们会实现一个添加用户接口，同时支持 JSON/XML 格式提交数据，也同时支持 JSON/XML 格式响应数据。
引入依赖
在 pom.xml 文件中，额外引入 jackson-dataformat-xml 依赖。如下：

<!-- 引入 jackson 对 xml 的转换器，实现对 XML 的序列化 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
9.2 SpringMVCConfiguration
修改 SpringMVCConfiguration 配置类，增加 MappingJackson2XmlHttpMessageConverter 相关的配置，用于 XML 格式的 HttpMessageConverter 消息转换器。代码如下：

// SpringMVCConfiguration.java

@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    // 增加 XML 消息转换器
    Jackson2ObjectMapperBuilder xmlBuilder = Jackson2ObjectMapperBuilder.xml();
    xmlBuilder.indentOutput(true);
    converters.add(new MappingJackson2XmlHttpMessageConverter(xmlBuilder.build()));
}
9.3 UserController
在 UserController 类中，我们添加一个 API 接口，新增用户，方便我们 XML/JSON 格式的请求和响应。代码如下：

// UserController.java

@PostMapping(value = "/add",
        // ↓ 增加 "application/xml"、"application/json" ，针对 Content-Type 请求头
        consumes = {MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_VALUE},
        // ↓ 增加 "application/xml"、"application/json" ，针对 Accept 请求头
        produces = {MediaType.APPLICATION_XML_VALUE, MediaType.APPLICATION_JSON_VALUE}
)
public UserVO add(@RequestBody UserVO user) {
    return user;
}
请求
在接口参数上，我们添加了 @RequestBody 注解。
添加 consumes 属性，增加 "application/xml"、"application/json" ，针对 Content-Type 请求头。
响应
因为 UserController 已经添加了 @RestController 注解，我们无需重复给改 API 接口，添加 @ReponseBody 注解。
添加 produces 属性，增加 "application/xml"、"application/json" ，针对 Accept 请求头。
下面，我们使用 Postman ，模拟请求。

① JSON 格式请求，JSON 格式响应

JSON + JSON

② XML 格式请求，XML 格式响应

XML + XML

③ JSON 格式请求，XML 格式响应

JSON + XML

# 整合 Fastjson
在国内，我们可能希望使用 Fastjson 作为 JSON 默认的工具类，以提升 JSON 的序列化和反序列化性能。在 Fastjson 中，已经内置提供 FastJsonHttpMessageConverter 消息转换器，方便我们替换 SpringMVC 默认的 HttpMessageConverter 。

友情提示：该示例，基于 「8. Cors 跨域」 的 lab-springmvc-23-02 的基础上，继续改造。

10.1 引入依赖
在 pom.xml 文件中，额外引入 fastjson 依赖。如下：

<!-- 引入 Fastjson ，实现对 JSON 的序列化 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.62</version>
</dependency>
10.2 SpringMVCConfiguration
修改 SpringMVCConfiguration 配置类，增加 FastJsonHttpMessageConverter 相关的配置。代码如下：

// SpringMVCConfiguration.java

@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    // 创建 FastJsonHttpMessageConverter 对象
    FastJsonHttpMessageConverter fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter();
    // 自定义 FastJson 配置
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    fastJsonConfig.setCharset(Charset.defaultCharset()); // 设置字符集
    fastJsonConfig.setSerializerFeatures(SerializerFeature.DisableCircularReferenceDetect); // 剔除循环引用
    fastJsonHttpMessageConverter.setFastJsonConfig(fastJsonConfig);
    // 设置支持的 MediaType
    fastJsonHttpMessageConverter.setSupportedMediaTypes(Arrays.asList(MediaType.APPLICATION_JSON,
            MediaType.APPLICATION_JSON_UTF8));
    // 添加到 converters 中
    converters.add(0, fastJsonHttpMessageConverter); // 注意，添加到最开头，放在 MappingJackson2XmlHttpMessageConverter 前面
}
至此，SpringMVC 整合 Fastjson 已经完成，还是蛮方便的。
