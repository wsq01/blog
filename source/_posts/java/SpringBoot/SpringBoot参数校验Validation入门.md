

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

`spring-boot-starter-web`依赖里，已经默认引入`hibernate-validator`依赖，所以本示例使用的是 Hibernate Validator 作为 Bean Validation 的实现框架。
在 Spring Boot 体系中，也提供了 spring-boot-starter-validation 依赖。该依赖的目的，重点也是引入 hibernate-validator 依赖，这在 spring-boot-starter-web 已经引入，所以无需重复引入。
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
添加`@EnableAspectJAutoProxy`注解，重点是配置`exposeProxy = true`，因为我们希望 Spring AOP 能将当前代理对象设置到`AopContext`中。具体用途，我们会在下文看到。
## UserAddDTO
在`cn.iocoder.springboot.lab22.validation.dto`包路径下，创建`UserAddDTO`类，为用户添加 DTO 类。代码如下：
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
## UserController
创建`UserController`类，提供用户 API 接口。代码如下：
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
在类上，添加`@Validated`注解，表示`UserController`是所有接口都需要进行参数校验。

对于 #get(id) 方法，我们在 id 参数上，添加了 @Min 注解，校验 id 必须大于 0。校验不通过示例如下图：


对于 #add(addDTO) 方法，我们在 addDTO 参数上，添加了 @Valid 注解，实现对该参数的校验。校验不通过示例如下图：



errors 字段，参数错误明细数组。每一个数组元素，对应一个参数错误明细。这里，username 违背了长度不满足 [5, 16] 。

示例我们是已经成功跑通了，但是呢，这里有几点差异性，我们要来理解下。

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
# 自定义约束
开发自定义约束一共只要两步：1）编写自定义约束的注解；2）编写自定义的校验器 ConstraintValidator 。

下面，就让我们一起来实现一个自定义约束，用于校验参数必须在枚举值的范围内。

5.1 IntArrayValuable
在 cn.iocoder.springboot.lab22.validation.core.validator 包路径下，创建 IntArrayValuable 接口，用于返回值数组。代码如下：

// IntArrayValuable.java

public interface IntArrayValuable {

    /**
     * @return int 数组
     */
    int[] array();

}
因为对于一个枚举类来说，我们无法获得它具体有那些值。所以，我们会要求这个枚举类实现该接口，返回它拥有的所有枚举值。

5.2 GenderEnum
在 cn.iocoder.springboot.lab22.validation.constants 包路径下，创建 GenderEnum 枚举类，枚举性别。代码如下：

// GenderEnum.java

public enum GenderEnum implements IntArrayValuable {

    MALE(1, "男"),
    FEMALE(2, "女");

    /**
     * 值数组
     */
    public static final int[] ARRAYS = Arrays.stream(values()).mapToInt(GenderEnum::getValue).toArray();

    /**
     * 性别值
     */
    private final Integer value;
    /**
     * 性别名
     */
    private final String name;

    GenderEnum(Integer value, String name) {
        this.value = value;
        this.name = name;
    }

    public Integer getValue() {
        return value;
    }

    public String getName() {
        return name;
    }

    @Override
    public int[] array() {
        return ARRAYS;
    }

}
实现 IntArrayValuable 接口，返回值数组 ARRAYS 。
5.3 @InEnum
在 cn.iocoder.springboot.lab22.validation.core.validator 包路径下，创建 @InEnum 自定义约束的注解。代码如下：

// InEnum.java

@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = InEnumValidator.class)
public @interface InEnum {

    /**
     * @return 实现 IntArrayValuable 接口的
     */
    Class<? extends IntArrayValuable> value();

    /**
     * @return 提示内容
     */
    String message() default "必须在指定范围 {value}";

    /**
     * @return 分组
     */
    Class<?>[] groups() default {};

    /**
     * @return Payload 数组
     */
    Class<? extends Payload>[] payload() default {};

    /**
     *  Defines several {@code @InEnum} constraints on the same element.
     */
    @Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @interface List {

        InEnum[] value();

    }

}
在类上，添加 @@Constraint(validatedBy = InEnumValidator.class) 注解，设置使用的自定义约束的校验器。
value() 属性，设置实现 IntArrayValuable 接口的类。这样，我们就能获得参数需要校验的值数组。
message() 属性，设置提示内容。默认为 "必须在指定范围 {value}" 。
其它属性，复制粘贴即可，都可以忽略不用理解。
5.4 InEnumValidator
在 cn.iocoder.springboot.lab22.validation.core.validator 包路径下，创建 InEnumValidator 自定义约束的校验器。代码如下：

// InEnumValidator.java

public class InEnumValidator implements ConstraintValidator<InEnum, Integer> {

    /**
     * 值数组
     */
    private Set<Integer> values;

    @Override
    public void initialize(InEnum annotation) {
        IntArrayValuable[] values = annotation.value().getEnumConstants();
        if (values.length == 0) {
            this.values = Collections.emptySet();
        } else {
            this.values = Arrays.stream(values[0].array()).boxed().collect(Collectors.toSet());
        }
    }

    @Override
    public boolean isValid(Integer value, ConstraintValidatorContext context) {
        // <2.1> 校验通过
        if (values.contains(value)) {
            return true;
        }
        // <2.2.1>校验不通过，自定义提示语句（因为，注解上的 value 是枚举类，无法获得枚举类的实际值）
        context.disableDefaultConstraintViolation(); // 禁用默认的 message 的值
        context.buildConstraintViolationWithTemplate(context.getDefaultConstraintMessageTemplate()
                .replaceAll("\\{value}", values.toString())).addConstraintViolation(); // 重新添加错误提示语句      
        return false; // <2.2.2.>
    }

}
实现 ConstraintValidator 接口。
第一个泛型为 A extends Annotation ，设置对应的自定义约束的注解。例如说，这里我们设置了 @InEnum 注解。
第二个泛型为 T ，设置对应的参数值的类型。例如说，这里我们设置了 Integer 类型。
实现 #initialize(annotation) 方法，获得 @InEnum 注解的 values() 属性，获得值数组，设置到 values 属性种。
实现 #isValid(value, context) 方法，实现校验参数值，是否在 values 范围内。
<2.1> 处，校验参数值在范围内，直接返回 true ，校验通过。
<2.2.1> 处，校验不通过，自定义提示语句。
<2.2.2> 处，校验不通过，所以返回 false 。
至此，我们已经完成了自定义约束的实现。下面，我们来进行下测试。

5.5 UserUpdateGenderDTO
在 cn.iocoder.springboot.lab22.validation.dto 包路径下，创建 UserUpdateGenderDTO 类，为用户更新性别 DTO。代码如下：

// UserUpdateGenderDTO.java

public class UserUpdateGenderDTO {

    /**
     * 用户编号
     */
    @NotNull(message = "用户编号不能为空")
    private Integer id;

    /**
     * 性别
     */
    @NotNull(message = "性别不能为空")
    @InEnum(value = GenderEnum.class, message = "性别必须是 {value}")
    private Integer gender;
    
    // ... 省略 set/get 方法
}
在 gender 字段上，添加 @InEnum(value = GenderEnum.class, message = "性别必须是 {value}") 注解，限制传入的参数值，必须在 GenderEnum 枚举范围内。
5.6 UserController
修改 UserController 类，增加修改性别 API 接口。代码如下：

// UserController.java

@PostMapping("/update_gender")
public void updateGender(@Valid UserUpdateGenderDTO updateGenderDTO) {
    logger.info("[updateGender][updateGenderDTO: {}]", updateGenderDTO);
}
模拟请求该 API 接口，响应结果如下：响应结果

因为我们传入的请求参数 gender 的值为 null ，显然不在 GenderEnum 范围内，所以校验不通过，输出 "性别必须是 [1, 2]" 。

6. 分组校验
示例代码对应仓库：lab-22-validation-01 。

在一些业务场景下，我们需要使用分组校验，即相同的 Bean 对象，根据校验分组，使用不同的校验规则。咳咳咳，貌似我们暂时没有这方面的诉求。即使有，也是拆分不同的 Bean 类。当然，作为一篇入门的文章，艿艿还是提供下分组校验的示例。

6.1 UserUpdateStatusDTO
在 cn.iocoder.springboot.lab22.validation.dto 包路径下，创建 UserUpdateStatusDTO 类，为用户更新状态 DTO 。代码如下：

// UserUpdateStatusDTO.java

public class UserUpdateStatusDTO {

    /**
     * 分组 01 ，要求状态必须为 true
     */
    public interface Group01 {}

    /**
     * 状态 02 ，要求状态必须为 false
     */
    public interface Group02 {}
    
    /**
     * 状态
     */
    @AssertTrue(message = "状态必须为 true", groups = Group01.class)
    @AssertFalse(message = "状态必须为 false", groups = Group02.class)
    private Boolean status;

    // ... 省略 set/get 方法
}
创建了 Group01 和 Group02 接口，作为两个校验分组。不一定要定义在 UserUpdateStatusDTO 类中，这里仅仅是为了方便。
status 字段，在 Group01 校验分组时，必须为 true ；在 Group02 校验分组时，必须为 false 。
6.2 UserController
修改 UserController 类，增加两个修改状态的 API 接口。代码如下：

// UserController.java

@PostMapping("/update_status_true")
public void updateStatusTrue(@Validated(UserUpdateStatusDTO.Group01.class) UserUpdateStatusDTO updateStatusDTO) {
    logger.info("[updateStatusTrue][updateStatusDTO: {}]", updateStatusDTO);
}

@PostMapping("/update_status_false")
public void updateStatusFalse(@Validated(UserUpdateStatusDTO.Group02.class) UserUpdateStatusDTO updateStatusDTO) {
    logger.info("[updateStatusFalse][updateStatusDTO: {}]", updateStatusDTO);
}
对于 #updateStatusTrue(updateStatusDTO) 方法，我们在 updateStatusDTO 参数上，添加了 @Validated 注解，并且设置校验分组为 Group01 。校验不通过示例如下图：不通过示例 1
对于 #updateStatusFalse(updateStatusDTO) 方法，我们在 updateStatusDTO 参数上，添加了 @Validated 注解，并且设置校验分组为 Group02 。校验不通过示例如下图：不通过示例 2
所以，使用分组校验，核心在于添加上 @Validated 注解，并设置对应的校验分组。

7. 手动校验
示例代码对应仓库：lab-22-validation-01 。

在上面的示例中，我们使用的主要是 Spring Validation 的声明式注解。然而在少数业务场景下，我们可能需要手动使用 Bean Validation API ，进行参数校验。

修改 UserServiceTest 测试类，增加手动参数校验的示例。代码如下：

// UserServiceTest.java

@Autowired // <1.1>
private Validator validator;

@Test
public void testValidator() {
    // 打印，查看 validator 的类型 // <1.2>
    System.out.println(validator);

    // 创建 UserAddDTO 对象 // <2>
    UserAddDTO addDTO = new UserAddDTO();
    // 校验 // <3>
    Set<ConstraintViolation<UserAddDTO>> result = validator.validate(addDTO);
    // 打印校验结果 // <4>
    for (ConstraintViolation<UserAddDTO> constraintViolation : result) {
        // 属性:消息
        System.out.println(constraintViolation.getPropertyPath() + ":" + constraintViolation.getMessage());
    }
}
<1.1> 处，注入 Validator Bean 对象。

<1.2> 处，打印 validator 的类型。输出如下：

org.springframework.validation.beanvalidation.LocalValidatorFactoryBean@48c3205a
validator 的类型为 LocalValidatorFactoryBean 。LocalValidatorFactoryBean 提供 JSR-303、JSR-349 的支持，同时兼容 Hibernate Validator 。
在 Spring Boot 体系中，使用 ValidationAutoConfiguration 自动化配置类，默认创建 LocalValidatorFactoryBean 作为 Validator Bean 。
<2> 处，创建 UserAddDTO 对象，即 「3.3 UserAddDTO」 ，已经添加相应的约束注解。

<3> 处，调用 Validator#validate(T object, Class<?>... groups) 方法，进行参数校验。

<4> 处，打印校验结果。输出如下：

username:登录账号不能为空
password:密码不能为空
如果校验通过，则返回的 Set<ConstraintViolation<?>> 集合为空。
8. 国际化 i18n
示例代码对应仓库：lab-22-validation-01 。

在一些项目中，我们会有国际化的需求，特别是我们在做 TOB 的 SASS 化服务的时候。那么，显然我们在使用 Bean Validator 做参数校验的时候，也需要提供国际化的错误提示。

给力的是，Hibernate Validator 已经内置了国际化的支持，所以我们只需要简单的配置，就可以实现国际化的错误提示。

8.1 应用配置文件
在 resources 目录下，创建 application.yaml 配置文件。配置如下：

spring:
  # i18 message 配置，对应 MessageSourceProperties 配置类
  messages:
    basename: i18n/messages # 文件路径基础名
    encoding: UTF-8 # 使用 UTF-8 编码
然后，我们在 resources/i18 目录下，创建不同语言的 messages 文件。如下：

messages.properties ：默认的 i18 配置文件。

UserUpdateDTO.id.NotNull=用户编号不能为空
messages_en.properties ：英文的 i18 配置文件。

UserUpdateDTO.id.NotNull=userId cannot be empty
messages_ja.properties ：日文的 i18 配置文件。

UserUpdateDTO.id.NotNull=ユーザー番号は空にできません
8.2 ValidationConfiguration
在 cn.iocoder.springboot.lab22.validation.config 包路径下，创建 ValidationConfiguration 配置类，用于创建一个支持 i18 国际化的 Validator Bean 对象。代码如下：

// ValidationConfiguration.java

@Configuration
public class ValidationConfiguration {

    /**
     * 参考 {@link ValidationAutoConfiguration#defaultValidator()} 方法，构建 Validator Bean
     *
     * @return Validator 对象
     */
    @Bean
    public Validator validator(MessageSource messageSource)  {
        // 创建 LocalValidatorFactoryBean 对象
        LocalValidatorFactoryBean validator = ValidationAutoConfiguration.defaultValidator();
        // 设置 messageSource 属性，实现 i18 国际化
        validator.setValidationMessageSource(messageSource);
        // 返回
        return validator;
    }

}
8.3 UserUpdateDTO
在 cn.iocoder.springboot.lab22.validation.dto 包路径下，创建 UserUpdateDTO 类，为用户更新 DTO 。代码如下：

// UserUpdateDTO.java

public class UserUpdateDTO {

    /**
     * 用户编号
     */
    @NotNull(message = "{UserUpdateDTO.id.NotNull}")
    private Integer id;

    // ... 省略 get/set 方法
    
}
不同于我们上面看到的约束注解的 message 属性的设置，这里我们使用了 {} 占位符。
8.4 UserController
修改 UserController 类，增加用户更新的 API 接口。代码如下：

// UserController.java

@PostMapping("/update")
public void update(@Valid UserUpdateDTO updateDTO) {
    logger.info("[update][updateDTO: {}]", updateDTO);
}
下面，我们来进行下 API 接口测试。有一点要注意，SpringMVC 通过 Accept-Language 请求头，实现 i18n 国际化。

Accept-Language = zh 的情况，响应结果如下：
Accept-Language = en 的情况，响应结果如下：
Accept-Language = ja 的情况，响应结果如下：
至此，我们的 Validator 的 i18n 国际化已经完成了。
