

# 注解
## Bean Validation 定义的约束注解
`javax.validation.constraints`包下，定义了一系列的约束(`constraint`)注解。如下：

空和非空检查
`@NotBlank`：只能用于字符串不为`null`，并且字符串`trim()`以后`length`要大于 0。
`@NotEmpty`：集合对象的元素不为 0，即集合不为空，也可以用于字符串不为`null`。
`@NotNull`：不能为`null`。
`@Null`：必须为`null`。

数值检查
* `@DecimalMax(value)`：被注释的元素必须是一个数字，其值必须小于等于指定的最大值。
* `@DecimalMin(value)`：被注释的元素必须是一个数字，其值必须大于等于指定的最小值。
* `@Digits(integer, fraction)`：被注释的元素必须是一个数字，其值必须在可接受的范围内。
* `@Positive`：判断正数。
* `@PositiveOrZero`：判断正数或 0 。
* `@Max(value)`：该字段的值只能小于或等于该值。
* `@Min(value)`：该字段的值只能大于或等于该值。
* `@Negative`：判断负数。
* `@NegativeOrZero`：判断负数或 0 。

Boolean 值检查
* `@AssertFalse`：被注释的元素必须为 true 。
* `@AssertTrue`：被注释的元素必须为 false 。

长度检查
* `@Size(max, min)`：检查该字段的 size 是否在 min 和 max 之间，可以是字符串、数组、集合、Map 等。

日期检查
* `@Future`：被注释的元素必须是一个将来的日期。
* `@FutureOrPresent`：判断日期是否是将来或现在日期。
* `@Past`：检查该字段的日期是在过去。
* `@PastOrPresent`：判断日期是否是过去或现在日期。

其它检查
* `@Email`：被注释的元素必须是电子邮箱地址。
* `@Pattern(value)`：被注释的元素必须符合指定的正则表达式。

## Hibernate Validator 附加的约束注解
`org.hibernate.validator.constraints`包下，定义了一系列的约束(`constraint`)注解。如下：
`@Range(min=, max=)`：被注释的元素必须在合适的范围内。
`@Length(min=, max=)`：被注释的字符串的大小必须在指定的范围内。
`@URL(protocol=,host=,port=,regexp=,flags=)`：被注释的字符串必须是一个有效的 URL 。
`@SafeHtml`：判断提交的 HTML 是否安全。例如说，不能包含 javascript 脚本等等。
... 等等，就不一一列举了。

## @Valid 和 @Validated
`@Valid`注解，是 Bean Validation 所定义，可以添加在普通方法、构造方法、方法参数、方法返回、成员变量上，表示它们需要进行约束校验。

`@Validated`注解，是 Spring Validation 锁定义，可以添加在类、方法参数、普通方法上，表示它们需要进行约束校验。同时，`@Validated`有`value`属性，支持分组校验。属性如下：
```
// Validated.java

Class<?>[] value() default {};
```

① 声明式校验

Spring Validation 仅对 @Validated 注解，实现声明式校验。

② 分组校验

Bean Validation 提供的 @Valid 注解，因为没有分组校验的属性，所以无法提供分组校验。此时，我们只能使用 ``@Validated` 注解。

③ 嵌套校验

相比来说，@Valid 注解的地方，多了【成员变量】。这就导致，如果有嵌套对象的时候，只能使用 @Valid 注解。例如说：
```java
// User.java
public class User {
    
    private String id;

    @Valid
    private UserProfile profile;

}

// UserProfile.java
public class UserProfile {

    @NotBlank
    private String nickname;

}
```
如果不在`User.profile`属性上，添加 @Valid 注解，就会导致 UserProfile.nickname 属性，不会进行校验。

当然，@Valid 注解的地方，也多了【构造方法】和【方法返回】，所以在有这方面的诉求的时候，也只能使用 @Valid 注解。

总的来说，绝大多数场景下，我们使用`@Validated`注解即可。

而在有嵌套校验的场景，我们使用`@Valid`注解添加到成员属性上。
# 快速入门
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

    <artifactId>lab-22-validation-01</artifactId>

    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 保证 Spring AOP 相关的依赖包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
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
具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。
spring-boot-starter-web 依赖里，已经默认引入 hibernate-validator 依赖，所以本示例使用的是 Hibernate Validator 作为 Bean Validation 的实现框架。
在 Spring Boot 体系中，也提供了 spring-boot-starter-validation 依赖。在这里，我们并没有引入。为什么呢？该依赖的目的，重点也是引入 hibernate-validator 依赖，这在 spring-boot-starter-web 已经引入，所以无需重复引入。
## Application
创建`Application.java`类，配置`@SpringBootApplication`注解即可。代码如下：
```java
@SpringBootApplication
@EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```
添加 @EnableAspectJAutoProxy 注解，重点是配置 exposeProxy = true ，因为我们希望 Spring AOP 能将当前代理对象设置到 AopContext 中。具体用途，我们会在下文看到。想要提前看的胖友，可以看看 《Spring AOP 通过获取代理对象实现事务切换》 文章。
## UserAddDTO
在 cn.iocoder.springboot.lab22.validation.dto 包路径下，创建 UserAddDTO 类，为用户添加 DTO 类。代码如下：
```java
// UserAddDTO.java

public class UserAddDTO {

    /**
     * 账号
     */
    @NotEmpty(message = "登录账号不能为空")
    @Length(min = 5, max = 16, message = "账号长度为 5-16 位")
    @Pattern(regexp = "^[A-Za-z0-9]+$", message = "账号格式为数字以及字母")
    private String username;
    /**
     * 密码
     */
    @NotEmpty(message = "密码不能为空")
    @Length(min = 4, max = 16, message = "密码长度为 4-16 位")
    private String password;
    
    // ... 省略 setting/getting 方法
}
```
每个字段上的约束注解，胖友仔细瞅瞅。
## UserController
创建 UserController 类，提供用户 API 接口。代码如下：
```java
// UserController.java

@RestController
@RequestMapping("/users")
@Validated
public class UserController {

    private Logger logger = LoggerFactory.getLogger(getClass());

    @GetMapping("/get")
    public void get(@RequestParam("id") @Min(value = 1L, message = "编号必须大于 0") Integer id) {
        logger.info("[get][id: {}]", id);
    }

    @PostMapping("/add")
    public void add(@Valid UserAddDTO addDTO) {
        logger.info("[add][addDTO: {}]", addDTO);
    }

}
```
在类上，添加 @Validated 注解，表示 UserController 是所有接口都需要进行参数校验。

对于 #get(id) 方法，我们在 id 参数上，添加了 @Min 注解，校验 id 必须大于 0 。校验不通过示例如下图：


对于 #add(addDTO) 方法，我们在 addDTO 参数上，添加了 @Valid 注解，实现对该参数的校验。校验不通过示例如下图：



errors 字段，参数错误明细数组。每一个数组元素，对应一个参数错误明细。这里，username 违背了长度不满足 [5, 16] 。
示例我们是已经成功跑通了，但是呢，这里有几点差异性，我们要来理解下。

也可以不理解，就按照这么使用即可。

第一点，#get(id) 方法上，我们并没有给 id 添加 @Valid 注解，而 #add(addDTO) 方法上，我们给 addDTO 添加 @Valid 注解。这个差异，是为什么呢？

因为 UserController 使用了 @Validated 注解，那么 Spring Validation 就会使用 AOP 进行切面，进行参数校验。而该切面的拦截器，使用的是 MethodValidationInterceptor 。

对于 #get(id) 方法，需要校验的参数 id ，是平铺开的，所以无需添加 @Valid 注解。
对于 #add(addDTO) 方法，需要校验的参数 addDTO ，实际相当于嵌套校验，要校验的参数的都在 addDTO 里面，所以需要添加 @Valid 注解。
第二点，#get(id) 方法的返回的结果是 status = 500 ，而 #add(addDTO) 方法的返回的结果是 status = 400 。

对于 #get(id) 方法，在 MethodValidationInterceptor 拦截器中，校验到参数不正确，会抛出 ConstraintViolationException 异常。
对于 #add(addDTO) 方法，因为 addDTO 是个 POJO 对象，所以会走 SpringMVC 的 DataBinder 机制，它会调用 DataBinder#validate(Object... validationHints) 方法，进行校验。在校验不通过时，会抛出 BindException 。
在 SpringMVC 中，默认使用 DefaultHandlerExceptionResolver 处理异常。

对于 BindException 异常，处理成 400 的状态码。
对于 ConstraintViolationException 异常，没有特殊处理，所以处理成 500 的状态码。
这里，我们在抛个问题，如果 #add(addDTO 方法，如果参数正确，在走完 DataBinder 中的参数校验后，会不会在走一遍 MethodValidationInterceptor 的拦截器呢？思考 100 毫秒...

答案是会。这样，就会导致浪费。所以 Controller 类里，如果只有类似的 #add(addDTO) 方法的嵌套校验，那么我可以不在 Controller 类上添加 @Validated 注解。从而实现，仅使用 DataBinder 中来做参数校验。

第三点，无论是 #get(id) 方法，还是 #add(addDTO) 方法，它们的返回提示都非常不友好，那么该怎么办呢？

参考 《芋道 Spring Boot SpringMVC 入门》 的 「5. 全局异常处理」 ，使用 @ExceptionHandler 注解，实现自定义的异常处理。这个，我们在本文的 4. 处理校验异常 小节中，来提供具体示例。

## UserService
相比在 Controller 添加参数校验来说，在 Service 进行参数校验，会更加安全可靠。艿艿个人建议的话，Controller 的参数校验可以不做，Service 的参数校验一定要做。

创建 UserService 类，提供用户 Service 逻辑。代码如下：
```java
// UserService.java

@Service
@Validated
public class UserService {

    private Logger logger = LoggerFactory.getLogger(getClass());

    public void get(@Min(value = 1L, message = "编号必须大于 0") Integer id) {
        logger.info("[get][id: {}]", id);
    }

    public void add(@Valid UserAddDTO addDTO) {
        logger.info("[add][addDTO: {}]", addDTO);
    }

    public void add01(UserAddDTO addDTO) {
        this.add(addDTO);
    }

    public void add02(UserAddDTO addDTO) {
        self().add(addDTO);
    }

    private UserService self() {
        return (UserService) AopContext.currentProxy();
    }

}
```
和 UserController 的方法是一致的，包括注解。
额外添加了 #add01(addDTO) 和 #add02(addDTO) 方法，用于演示方法内部调用。
创建 UserServiceTest 测试类，我们来测试一下简单的 UserService 的每个操作。代码如下：
```java
// UserService.java

@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void testGet() {
        userService.get(-1);
    }

    @Test
    public void testAdd() {
        UserAddDTO addDTO = new UserAddDTO();
        userService.add(addDTO);
    }

    @Test
    public void testAdd01() {
        UserAddDTO addDTO = new UserAddDTO();
        userService.add01(addDTO);
    }

    @Test
    public void testAdd02() {
        UserAddDTO addDTO = new UserAddDTO();
        userService.add02(addDTO);
    }

}
```
① #testGet() 测试方法

执行，抛出 ConstraintViolationException 异常。日志如下：

javax.validation.ConstraintViolationException: get.id: 编号必须大于 0

	at org.springframework.validation.beanvalidation.MethodValidationInterceptor.invoke(MethodValidationInterceptor.java:116)
符合期望。
② #testAdd() 测试方法

执行，抛出 ConstraintViolationException 异常。日志如下：

javax.validation.ConstraintViolationException: add.addDTO.username: 登录账号不能为空, add.addDTO.password: 密码不能为空

	at org.springframework.validation.beanvalidation.MethodValidationInterceptor.invoke(MethodValidationInterceptor.java:116)
符合期望。不同于我们在调用 UserController#add(addDTO) 方法，这里被 MethodValidationInterceptor 拦截，进行参数校验，而不是 DataBinder 当中。
③ #testAdd01() 测试方法

执行，正常结束。因为进行 this.add(addDTO) 调用时，this 并不是 Spring AOP 代理对象，所以并不会被 MethodValidationInterceptor 所拦截。

④ #testAdd02() 测试方法

执行，抛出 IllegalStateException 异常。日志如下：

java.lang.IllegalStateException: Cannot find current proxy: Set 'exposeProxy' property on Advised to 'true' to make it available.

	at org.springframework.aop.framework.AopContext.currentProxy(AopContext.java:69)
理论来说，因为我们配置了 @EnableAspectJAutoProxy(exposeProxy = true) 注解，在 Spring AOP 拦截时，通过调用 AopContext.currentProxy() 方法，是可以获取到当前的代理对象。结果，此处抛出 IllegalStateException 异常。
显然，这里并没有将当前的代理对象，设置到 AopContext 中，所以抛出 IllegalStateException 异常。目前猜测，可能是 BUG 。😈 暂时木有心情去调试，嘿嘿。

# 处理校验异常
