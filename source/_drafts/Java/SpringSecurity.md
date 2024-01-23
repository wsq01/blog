

SpringSecurity 的核心功能是认证和授权：
 * 认证：验证当前访问系统的是不是本系统的用户，并且要确认具体是哪个用户
 * 授权：经过认证后判断当前用户是否有权限进行某个操作

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
        <version>2.5.15</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <!-- ... -->

    <dependencies>
        <!-- 实现对 Spring MVC 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- 实现对 Spring Security 的自动化配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    </dependencies>

</project>
```
## 创建启动类
创建`Application.java`类，配置`@SpringBootApplication`注解。
```java
// Application.java

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
## 配置文件
在`application.yml`中，添加 Spring Security 配置：
```yaml
spring:
  # Spring Security 配置项，对应 SecurityProperties 配置类
  security:
    # 配置默认的 InMemoryUserDetailsManager 的用户账号与密码。
    user:
      name: user # 账号
      password: user # 密码
      roles: ADMIN # 拥有角色
```
说明：
* 在`spring.security`配置项，设置 Spring Security 的配置，对应`SecurityProperties`配置类。
* 默认情况下，Spring Boot `UserDetailsServiceAutoConfiguration`自动化配置类，会创建一个内存级别的`InMemoryUserDetailsManager Bean`对象，提供认证的用户信息。
* 这里，我们添加了`spring.security.user`配置项，`UserDetailsServiceAutoConfiguration`会基于配置的信息创建一个用户`User`在内存中。
* 如果，我们未添加`spring.security.user`配置项，`UserDetailsServiceAutoConfiguration`会自动创建一个用户名为`user`，密码为`UUID`随机的用户`User`在内存中。

## 创建Controller
创建`AdminController`类，提供管理员 API 接口：
```java
// AdminController.java

@RestController
@RequestMapping("/admin")
public class AdminController {

    @GetMapping("/demo")
    public String demo() {
        return "示例返回";
    }
}
```
这里，我们先提供一个`"/admin/demo"`接口，用于测试未登录时，会被拦截到登录界面。
## 简单测试
执行`Application#main(String[] args)`方法，运行项目。

项目启动成功后，浏览器访问`http://127.0.0.1:8080/admin/demo`接口。因为未登录，所以被 Spring Security 拦截到登录界面。如下图所示：

{% asset_img 1.png %}

因为没有自定义登录界面，所以默认会使用`DefaultLoginPageGeneratingFilter`类，生成上述界面。

输入我们在配置文件中配置的`user/user`账号，进行登录。登录完成后，因为 Spring Security 会记录被拦截的访问地址，所以浏览器自动动跳转`http://127.0.0.1:8080/admin/demo`接口。访问结果如下图所示：

{% asset_img 2.png %}

# 进阶使用
## 权限控制
自定义 Spring Security 的配置，实现权限控制。
### SecurityConfig
创建 SecurityConfig 配置类，继承 WebSecurityConfigurerAdapter 抽象类，实现 Spring Security 在 Web 场景下的自定义配置。
```java
// SecurityConfig.java

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // ...
}
```
我们可以通过重写`WebSecurityConfigurerAdapter`的方法，实现自定义的 Spring Security 的配置。

首先，我们重写`configure(AuthenticationManagerBuilder auth)`方法，实现`AuthenticationManager`认证管理器。
```java
// SecurityConfig.java

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.
        // 1. 使用内存中的 InMemoryUserDetailsManager
        inMemoryAuthentication()
        // 2. 不使用 PasswordEncoder 密码编码器
        .passwordEncoder(NoOpPasswordEncoder.getInstance())
        // 3. 配置 admin 用户
        .withUser("admin").password("admin").roles("ADMIN")
        // 3. 配置 normal 用户
        .and().withUser("normal").password("normal").roles("NORMAL");
}
```
1 处调用`AuthenticationManagerBuilder#inMemoryAuthentication()`方法，使用内存级别的`InMemoryUserDetailsManager Bean`对象，提供认证的用户信息。

Spring 内置了两种`UserDetailsManager`实现：
* `InMemoryUserDetailsManager`。
* `JdbcUserDetailsManager`，基于 JDBC 的`JdbcUserDetailsManager`。

实际项目中，我们更多采用调用`AuthenticationManagerBuilder#userDetailsService(userDetailsService)`方法，使用自定义实现的`UserDetailsService`实现类，更加灵活且自由的实现认证的用户信息的读取。
```java
@Autowired
private UserDetailsService userDetailsService;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
}

@Bean
public BCryptPasswordEncoder bCryptPasswordEncoder() {
    return new BCryptPasswordEncoder();
}
```
2 处调用`AbstractDaoAuthenticationConfigurer#passwordEncoder(passwordEncoder)`方法，设置`PasswordEncoder`密码编码器。

在这里，为了方便，我们使用`NoOpPasswordEncoder`。实际上，等于不使用`PasswordEncoder`，不配置的话会报错。生产环境下，推荐使用`BCryptPasswordEncoder`。

3 处，配置`admin/admin`和`normal/normal`两个用户，分别对应`ADMIN`和`NORMAL`角色。

然后，我们重写`configure(HttpSecurity http)`方法，主要配置 URL 的权限控制。
```java
// SecurityConfig.java

@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        // 1 配置请求地址的权限
        .authorizeRequests()
            .antMatchers("/test/echo").permitAll() // 所有用户可访问
            .antMatchers("/test/admin").hasRole("ADMIN") // 需要 ADMIN 角色
            .antMatchers("/test/normal").access("hasRole('ROLE_NORMAL')") // 需要 NORMAL 角色。
            // 任何请求，访问的用户都需要经过认证
            .anyRequest().authenticated()
        .and()
        // 2 设置 Form 表单登录
        .formLogin()
            //   .loginPage("/login") // 登录 URL 地址
            .permitAll() // 所有用户可访问
        .and()
        // 3 配置退出相关
        .logout()
            //  .logoutUrl("/logout") // 退出 URL 地址
            .permitAll(); // 所有用户可访问
}
```
1 处，调用`HttpSecurity#authorizeRequests()`方法，开始配置 URL 的权限控制。配置权限控制会使用到的方法：
* `(String... antPatterns)`方法，配置匹配的 URL 地址，基于 Ant 风格路径表达式，可传入多个。
* `permitAll()`方法，所有用户可访问。
* `denyAll()`方法，所有用户不可访问。
* `authenticated()`方法，登录用户可访问。
* `anonymous()`方法，无需登录，即匿名用户可访问。
* `rememberMe()`方法，通过`remember me`登录的用户可访问。
* `fullyAuthenticated()`方法，非`remember me`登录的用户可访问。
* `hasIpAddress(String ipaddressExpression)`方法，来自指定 IP 表达式的用户可访问。
* `hasRole(String role)`方法，拥有指定角色的用户可访问。
* `hasAnyRole(String... roles)`方法，拥有指定任一角色的用户可访问。
* `hasAuthority(String authority)`方法，拥有指定权限(`authority`)的用户可访问。
* `hasAuthority(String... authorities)`方法，拥有指定任一权限(`authority`)的用户可访问。
* `access(String attribute)`方法，当 Spring EL 表达式的执行结果为`true`时，可以访问。

2 处，调用`HttpSecurity#formLogin()`方法，设置`Form`表单登录。

如果想要自定义登录页面，可以通过`loginPage(String loginPage)`方法，来进行设置。这里我们使用默认的登录界面，所以不进行设置。

3 处，调用`HttpSecurity#logout()`方法，配置退出相关。

如果想要自定义退出页面，可以通过`logoutUrl(String logoutUrl)`方法，来进行设置。这里我们使用默认的退出界面，所以不进行设置。
### 测试
创建`TestController`类，提供测试 API 接口。
```java
// TestController.java

@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/echo")
    public String demo() {
        return "示例返回";
    }

    @GetMapping("/home")
    public String home() {
        return "我是首页";
    }

    @GetMapping("/admin")
    public String admin() {
        return "我是管理员";
    }

    @GetMapping("/normal")
    public String normal() {
        return "我是普通用户";
    }

}
```
* 对于`/test/echo`接口，直接访问，无需登录。
* 对于`/test/home`接口，无法直接访问，需要进行登录。
* 对于`/test/admin`接口，需要登录`admin/admin`用户，因为需要`ADMIN`角色。
* 对于`/test/normal`接口，需要登录`normal/normal`用户，因为需要`NORMAL`角色。

## 注解实现权限控制
### SecurityConfig
修改`SecurityConfig`配置类，增加`@EnableGlobalMethodSecurity`注解，开启对 Spring Security 注解的方法，进行权限验证。\
```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter
```
### DemoController
创建`DemoController`类，提供测试 API 接口。
```java
// DemoController.java

@RestController
@RequestMapping("/demo")
public class DemoController {

    @PermitAll
    @GetMapping("/echo")
    public String demo() {
        return "示例返回";
    }

    @GetMapping("/home")
    public String home() {
        return "我是首页";
    }

    @PreAuthorize("hasRole('ROLE_ADMIN')")
    @GetMapping("/admin")
    public String admin() {
        return "我是管理员";
    }

    @PreAuthorize("hasRole('ROLE_NORMAL')")
    @GetMapping("/normal")
    public String normal() {
        return "我是普通用户";
    }

}

```

`@PermitAll`注解，等价于`.permitAll()`方法，所有用户可访问。

重要！！！因为在`SecurityConfig`中，配置了`.anyRequest().authenticated()`，任何请求，访问的用户都需要经过认证。所以这里`@PermitAll`注解实际是不生效的。

也就是说，Java Config 配置的权限，和注解配置的权限，两者是叠加的。

`@PreAuthorize`注解，等价于`access(String attribute)`方法，，当 Spring EL 表达式的执行结果为`true`时，可以访问。

### 区别：@Secured(), @PreAuthorize() 及 @RolesAllowed()
现在举例，比如修改用户密码，必须是ADMIN的权限才可以。则可以用下面三种方法：
```
@Secured({"ROLE_ADMIN"})public void changePassword(String username, String password);

@RolesAllowed({"ROLE_ADMIN"})public void changePassword(String username, String password);

@PreAuthorize("hasRole('ROLE_ADMIN')")public void changePassword(String username, String password);
```
然而，这三个的区别，其实很容易被大家忽视，虽然不是太大的区别。

1、@Secured(): secured_annotation

使用时，需要如下配置Spring Security (无论是通过xml配置，还是在Spring boot下，直接注解配置，都需要指明secured-annotations)
```
XML: <global-method-security secured-annotations="enabled"/>

Spring boot: @EnableGlobalMethodSecurity(securedEnabled = true)
```
2、@RolesAllowed(): jsr250-annotations

使用时，需要如下配置Spring Security (无论是通过xml配置，还是在Spring boot下，直接注解配置，都需要指明jsr250-annotations)
```
XML: <global-method-security jsr250-annotations="enabled"/>

Spring boot: @EnableGlobalMethodSecurity(jsr250Enabled = true)
```
3、@PreAuthorize(): pre-post-annotations

使用时，需要如下配置Spring Security (无论是通过xml配置，还是在Spring boot下，直接注解配置，都需要指明pre-post-annotations)
```
XML: <global-method-security pre-post-annotations="enabled"/>

Spring boot: @EnableGlobalMethodSecurity(prePostEnabled = true)
```

| 方法授权类型	                     | 声明方式	 | JSR标准 |  允许SpEL表达式|
| :--: | :--: | :--: | :--: |
| @PreAuthorize @PostAuthorize      | 注解	    | No	     | Yes |
| @RolesAllowed @PermitAll @DenyAll	| 注解	    | Yes	     | NO |
| @Secure	                          | 注解	    | No	     | No |
| protect-pointcut                  | XML	      | No	     | No |