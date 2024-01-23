

Lombok 是一个 Java 工具，通过使用其定义的注解，自动生成常见的冗余代码，提升开发效率。

举个例子，在 Java POJO 类上，添加 @Setter 和 @Getter 注解，自动生成 set、get 方法的代码。示例如下：
```java
// 我们编写的 UserDO.java 代码
@Setter
@Getter
public class UserDO {

    private String username;
    private String password;

}

// 实际生成的代码（通过 UserDO.class 反编译）
public class UserDO {
    private String username;
    private String password;

    public UserDO() {
    }

    public void setUsername(final String username) {
        this.username = username;
    }

    public void setPassword(final String password) {
        this.password = password;
    }

    public String getUsername() {
        return this.username;
    }

    public String getPassword() {
        return this.password;
    }
}
```
# 安装 Lombok
在 IDEA 中，已经提供了 IntelliJ Lombok plugin 插件，方便我们使用 Lombok。安装方式很简单，只需要在 IDEA Plugins 功能中，搜索 Lombok 关键字即可。如下图所示：

{% asset_img 1.png %}

安装完成，需要重启 IDEA 来让该插件生效。生效完成后，我们可以在 IDEA 的设置中，找到 IDEA Lombok 功能。如下图所示：

{% asset_img 2.png %}

# 搭建示例项目
在 Maven 项目的`pom`文件，我们只需要引入`lombok`依赖，就可以使用 Lombok 啦。
```xml
<!-- 引入 Lombok 依赖 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```
具体的示例项目搭建，就是一个普通的项目，就不重复赘述啦。
# Lombok 注解一览
Lombok 的注解非常多，我们逐个来看看。
* `@Getter`注解，添加在类或属性上，生成对应的`get`方法。
* `@Setter`注解，添加在类或属性上，生成对应的`set`方法。
* `@ToString`注解，添加在类上，生成`toString`方法。
* `@EqualsAndHashCode`注解，添加在类上，生成`equals`和`hashCode`方法。
* `@AllArgsConstructor、@RequiredArgsConstructor、@NoArgsConstructor`注解，添加在类上，为类自动生成对应参数的构造方法。
* `@Data`注解，添加在类上，是 5 个 Lombok 注解的组合。
 * 为所有属性，添加`@Getter、@ToString、@EqualsAndHashCode`注解的效果
 * 为非`final`修饰的属性，添加`@Setter`注解的效果
 * 为`final`修改的属性，添加`@RequiredArgsConstructor`注解的效果
* `@Value`注解，添加在类上，和`@Data`注解类似，区别在于它会把所有属性默认定义为`private final`修饰，所以不会生成`set`方法。
* `@CommonsLog、@Flogger、@Log、@JBossLog、@Log4j、@Log4j2、@Slf4j、@Slf4jX`注解，添加在类上，自动为类添加对应的日志支持。
* `@NonNull`注解，添加在方法参数、类属性上，用于自动生成`null`参数检查。若确实是`null`时，抛出`NullPointerException`异常。
* `@Cleanup`注解，添加在方法中的局部变量上，在作用域结束时会自动调用`close()`方法，来释放资源。例如说，使用在 Java IO 流操作的时候。
* `@Builder`注解，添加在类上，给该类加个构造者模式`Builder`内部类。
* `@Synchronized`注解，添加在方法上，添加同步锁。
* `@SneakyThrows`注解，添加在方法上，给该方法添加`try catch`代码块。
* `@Accessors`注解，添加在方法或属性上，并设置`chain = true`，实现链式编程。

下面，我们在 Spring Boot 示例项目中，使用下 @Data 和 @Slf4j、@NonNull 这三个 Lombok 常用注解。
# @Data 注解
@Data 注解，添加在类上，是 5 个 Lombok 注解的组合。
为所有属性，添加 @Getter、@ToString、@EqualsAndHashCode 注解的效果
为非 final 修饰的属性，添加 @Setter 注解的效果
为 final 修改的属性，添加 @RequiredArgsConstructor 注解的效果

创建示例类 UserDO01，演示 @Data 注解的使用。

{% asset_img 3.png %}

友情提示：生成的`class`类，需要手动编译下项目，才能看到对应的类，之后进行反编译。

不过要注意，如果使用`@Data`注解的类，继承成了其它父类的属性，最好额外添加`@ToString(callSuper = true)`和`@EqualsAndHashCode(callSuper = true)`注解。

因为默认情况下，`@Data`注解不会处理父类的属性。所以需要我们通过`callSuper = true`属性，声明需要调用父类对应的方法。
这个情况非常常见，例如说在实体类中，我们可能会声明一个抽象父类 AbstractEntity，而该类里有一个 id 编号属性。
# @Slf4j 注解
`@Slf4j`注解，添加在类上，给该类创建`Slf4j Logger`静态属性。

创建示例类`UserService`，演示`@Slf4`注解的使用。

{% asset_img 4.png %}

Lombok 还提供了`@CommonsLog、@Flogger、@Log、@JBossLog、@Log4j、@Log4j2、@Slf4jX`注解，支持持不同的`Logger`组件。因为 Spring Boot 使用`Slf4j`日志门面框架，所以绝大多数情况下，我们都是使用`@Slf4j`注解。
# @NonNull 注解
`@NonNull`注解，添加在方法参数、类属性上，用于自动生成`null`参数检查。若确实是`null`时，抛出`NullPointerException`异常。

创建示例类`UserService01`，演示`@NonNull`注解的使用。

{% asset_img 5.png %}
