


# 快速入门
## 引入依赖
在`pom.xml`文件中，引入相关依赖。
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.3.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependencies>

    <!-- 实现对 Spring Data Redis 的自动化配置 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
        <exclusions>
            <!-- 去掉对 Lettuce 的依赖，因为 Spring Boot 优先使用 Lettuce 作为 Redis 客户端 -->
            <exclusion>
                <groupId>io.lettuce</groupId>
                <artifactId>lettuce-core</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <!-- 引入 Jedis 的依赖，这样 Spring Boot 实现对 Jedis 的自动化配置 -->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>

    <!-- 方便等会写单元测试 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- 等会示例会使用 fastjson 作为 JSON 序列化的工具 -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.61</version>
    </dependency>

    <!-- Spring Data Redis 默认使用 Jackson 作为 JSON 序列化的工具 -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>

</dependencies>
```
## 配置文件
在`application.yml`中，添加 Redis 配置：
```yaml
spring:
  # 对应 RedisProperties 类
  redis:
    host: 127.0.0.1
    port: 6379
    password: # Redis 服务器密码，默认为空。生产中，一定要设置 Redis 密码！
    database: 0 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
    # 对应 RedisProperties.Jedis 内部类
    jedis:
      pool:
        max-active: 8 # 连接池最大连接数，默认为 8 。使用负数表示没有限制。
        max-idle: 8 # 默认连接数最小空闲的连接数，默认为 8 。使用负数表示没有限制。
        min-idle: 0 # 默认连接池最小空闲的连接数，默认为 0 。允许设置 0 和 正数。
        max-wait: -1 # 连接池最大阻塞等待时间，单位：毫秒。默认为 -1 ，表示不限制。
```
## 简单测试
创建`Test01`测试类，我们来测试一下简单的`SET`指令。
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01 {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void testStringSetKey() {
        stringRedisTemplate.opsForValue().set("yunai", "shuai");
    }
}
```
通过`StringRedisTemplate`类，我们进行了一次 Redis SET 指令的执行。

我们先来执行下`testStringSetKey()`方法这个测试方法。执行完成后，我们在控制台查询，看看是否真的执行成功了。
```
$ redis-cli get yunai
"shuai"
```
## RedisTemplate
`org.springframework.data.redis.core.RedisTemplate<K, V>`类，从类名上，我们就明明白白知道，提供 Redis 操作模板 API。核心属性如下：
```java
// RedisTemplate.java

// <1> 序列化相关属性
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer keySerializer = null;
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer valueSerializer = null;
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashKeySerializer = null;
@SuppressWarnings("rawtypes") private @Nullable RedisSerializer hashValueSerializer = null;
private RedisSerializer<String> stringSerializer = RedisSerializer.string();

// <2> Lua 脚本执行器
private @Nullable ScriptExecutor<K> scriptExecutor;

// <3> 常见数据结构操作类
// cache singleton objects (where possible)
private @Nullable ValueOperations<K, V> valueOps;
private @Nullable ListOperations<K, V> listOps;
private @Nullable SetOperations<K, V> setOps;
private @Nullable ZSetOperations<K, V> zSetOps;
private @Nullable GeoOperations<K, V> geoOps;
private @Nullable HyperLogLogOperations<K, V> hllOps;
```
<1> 处，看到了四个序列化相关的属性，用于 KEY 和 VALUE 的序列化。
例如说，我们在使用 POJO 对象存储到 Redis 中，一般情况下，会使用 JSON 方式序列化成字符串，存储到 Redis 中。详细的，我们在 「3. 序列化」 小节中来说明。
在上文中，我们看到了 org.springframework.data.redis.core.StringRedisTemplate 类，它继承 RedisTemplate 类，使用 org.springframework.data.redis.serializer.StringRedisSerializer 字符串序列化方式。直接点开 StringRedisSerializer 源码，看下它的构造方法，瞬间明明白白。

`<2>`处，Lua 脚本执行器，提供`Redis scripting API`操作。

`<3>`处，Redis 常见数据结构操作类。
* `ValueOperations`类，提供`Redis String API`操作。
* `ListOperations`类，提供`Redis List API`操作。
* `SetOperations`类，提供`Redis Set API`操作。
* `ZSetOperations`类，提供`Redis ZSet(Sorted Set) API`操作。
* `GeoOperations`类，提供`Redis Geo API`操作。
* `HyperLogLogOperations`类，提供`Redis HyperLogLog API`操作。

那么`Pub/Sub、Transaction、Pipeline、Keys、Cluster、Connection`等相关的 API 操作呢？它在`RedisTemplate`自身提供，因为它们不属于具体每一种数据结构，所以没有封装在对应的`Operations`类中。
# 序列化
## RedisSerializer
`org.springframework.data.redis.serializer.RedisSerializer`接口，Redis 序列化接口，用于 Redis `KEY`和`VALUE`的序列化。简化代码如下：
```java
// RedisSerializer.java
public interface RedisSerializer<T> {

	@Nullable
	byte[] serialize(@Nullable T t) throws SerializationException;

	@Nullable
	T deserialize(@Nullable byte[] bytes) throws SerializationException;

}
```
定义了对象 <T> 和二进制数组的转换。
我们在 redis-cli 终端，看到的不都是字符串么，怎么这里是序列化成二进制数组呢？实际上，Redis Client 传递给 Redis Server 是传递的 KEY 和 VALUE 都是二进制值数组。好奇的胖友，可以打开 Jedis Connection#sendCommand(final Command cmd, final byte[]... args) 方法，传入的参数就是二进制数组，而 cmd 命令也会被序列化成二进制数组。
RedisSerializer 的实现类，如下图：

主要分成四类：
* JDK 序列化方式
* String 序列化方式
* JSON 序列化方式
* XML 序列化方式

### JDK 序列化方式
`org.springframework.data.redis.serializer.JdkSerializationRedisSerializer`，默认情况下，`RedisTemplate`使用该数据列化方式。具体的，可以看看 RedisTemplate#afterPropertiesSet() 方法，在 RedisTemplate 未设置序列化的情况下，使用 JdkSerializationRedisSerializer 作为序列化实现。在 Spring Boot 自动化配置 RedisTemplate Bean 对象时，就未设置。

绝大多数情况下，我们不会使用 JdkSerializationRedisSerializer 进行序列化。为什么呢？我们来看一个示例，代码如下：
```java
// Test01.java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Test01 {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void testStringSetKey02() {
        redisTemplate.opsForValue().set("yunai", "shuai");
    }

}
```
我们先来执行下 #testStringSetKey02() 方法这个测试方法。注意，此处我们使用的是 RedisTemplate 而不是 StringRedisTemplate 。执行完成后，我们在控制台查询，看看是否真的执行成功了。
```
# 在 `redis-cli` 终端中

127.0.0.1:6379> scan 0
1) "0"
2) 1) "\xac\xed\x00\x05t\x00\x05yunai"

127.0.0.1:6379> get "\xac\xed\x00\x05t\x00\x05yunai"
"\xac\xed\x00\x05t\x00\x05shuai"
```
通过 Redis SCAN 命令，我们扫描出了一个奇怪的 "yunai" KEY ，前面带着奇怪的 16 进制字符。而后，我们使用这个奇怪的 KEY 去获取对应的 VALUE ，结果前面也是一串奇怪的 16 进制字符。

具体为什么是这样一串奇怪的 16 进制，胖友可以看看 ObjectOutputStream#writeString(String str, boolean unshared) 的代码，实际就是标志位 + 字符串长度 + 字符串内容。

对于 KEY 被序列化成这样，我们线上通过 KEY 去查询对应的 VALUE 势必会非常不方便，所以 KEY 肯定是不能被这样序列化的。

对于 VALUE 被序列化成这样，除了阅读可能困难一点，不支持跨语言外，实际上也没啥问题。不过，实际线上场景，还是使用 JSON 序列化居多。

### String 序列化方式
① org.springframework.data.redis.serializer.StringRedisSerializer ，字符串和二进制数组的直接转换。代码如下：
```java
// StringRedisSerializer.java

private final Charset charset;

@Override
public String deserialize(@Nullable byte[] bytes) {
	return (bytes == null ? null : new String(bytes, charset));
}

@Override
public byte[] serialize(@Nullable String string) {
	return (string == null ? null : string.getBytes(charset));
}
```
绝大多数情况下，我们`KEY`和`VALUE`都会使用这种序列化方案。而`VALUE`的序列化和反序列化，自己在逻辑调用 JSON 方法去序列化。为什么呢？继续往下看。

② org.springframework.data.redis.serializer.GenericToStringSerializer<T> ，使用 Spring ConversionService 实现 <T> 对象和 String 的转换，从而 String 和二进制数组的转换。

例如说，序列化的过程，首先 <T> 对象通过 ConversionService 转换成 String ，然后 String 再序列化成二进制数组。反序列化的过程，胖友自己结合源码思考下 🤔 。

当然，GenericToStringSerializer 貌似基本不会去使用，所以不用去了解也问题不大，哈哈哈。

### JSON 序列化方式
① org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer ，使用 Jackson 实现 JSON 的序列化方式，并且从 Generic 单词可以看出，是支持所有类。怎么体现呢？参见构造方法的代码：
```java
// GenericJackson2JsonRedisSerializer.java

public GenericJackson2JsonRedisSerializer(@Nullable String classPropertyTypeName) {

	this(new ObjectMapper());

	// simply setting {@code mapper.disable(SerializationFeature.FAIL_ON_EMPTY_BEANS)} does not help here since we need
	// the type hint embedded for deserialization using the default typing feature.
	mapper.registerModule(new SimpleModule().addSerializer(new NullValueSerializer(classPropertyTypeName)));

	// <1>
	if (StringUtils.hasText(classPropertyTypeName)) {
		mapper.enableDefaultTypingAsProperty(DefaultTyping.NON_FINAL, classPropertyTypeName);
	// <2>
	} else {
		mapper.enableDefaultTyping(DefaultTyping.NON_FINAL, As.PROPERTY);
	}
}
```
<1> 处，如果传入了 classPropertyTypeName 属性，就是使用使用传入对象的 classPropertyTypeName 属性对应的值，作为默认类型（Default Typing）。
<2> 处，如果未传入 classPropertyTypeName 属性，则使用传入对象的类全名，作为默认类型（Default Typing）。
那么，胖友可能会问题，什么是**默认类型（Default Typing）**呢？我们来思考下，在将一个对象序列化成一个字符串，怎么保证字符串反序列化成对象的类型呢？Jackson 通过 Default Typing ，会在字符串多冗余一个类型，这样反序列化就知道具体的类型了。来举个例子，使用我们等会示例会用到的 UserCacheObject 类。

标准序列化的结果，如下：
```
{
    "id": 1,
    "name": "芋道源码",
    "gender": 1
}
```
使用 Jackson Default Typing 机制序列化的结果，如下：
```
{
       "@class": "cn.iocoder.springboot.labs.lab10.springdatarediswithjedis.cacheobject.UserCacheObject",
       "id": 1,
       "name": "芋道源码",
       "gender": 1
   }
```
看 @class 属性，反序列化的对象的类型不就有了么？
下面我们来看一个 GenericJackson2JsonRedisSerializer 的示例。在看之前，胖友先跳到 「3.2 配置序列化方式」 小节，来看看如何配置 GenericJackson2JsonRedisSerializer 作为 VALUE 的序列化方式。然后，马上调回到此处。

示例代码如下：
```java
// Test01.java

@Autowired
private RedisTemplate redisTemplate;

@Test
public void testStringSetKeyUserCache() {
    UserCacheObject object = new UserCacheObject()
            .setId(1)
            .setName("芋道源码")
            .setGender(1); // 男
    String key = String.format("user:%d", object.getId());
    redisTemplate.opsForValue().set(key, object);
}

@Test
public void testStringGetKeyUserCache() {
    String key = String.format("user:%d", 1);
    Object value = redisTemplate.opsForValue().get(key);
    System.out.println(value);
}
```
胖友分别执行 #testStringSetKeyUserCache() 和 #testStringGetKeyUserCache() 方法，然后对着 Redis 的结果看看，比较简单，就不多哔哔了。

我们在回过头来看看 @class 属性，它看似完美解决了反序列化后的对象类型，但是带来 JSON 字符串占用变大，所以实际项目中，我们也并不会采用 Jackson2JsonRedisSerializer 类。

② org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer<T> ，使用 Jackson 实现 JSON 的序列化方式，并且显示指定 <T> 类型。代码如下：
```java
// Jackson2JsonRedisSerializer.java
public class Jackson2JsonRedisSerializer<T> implements RedisSerializer<T> {
    // ... 省略不重要的代码

    public static final Charset DEFAULT_CHARSET = StandardCharsets.UTF_8;

    /**
     * 指定类型，和 <T> 要一致。
     */
    private final JavaType javaType;

    private ObjectMapper objectMapper = new ObjectMapper();

}
```
因为 Jackson2JsonRedisSerializer 序列化类里已经声明了类型，所以序列化的 JSON 字符串，无需在存储一个 @class 属性，用于存储类型。

但是，我们抠脚一想，如果使用 Jackson2JsonRedisSerializer 作为序列化实现类，那么如果我们类型比较多，岂不是每个类型都要定义一个 RedisTemplate Bean 了？！所以实际场景下，我们也并不会使用 Jackson2JsonRedisSerializer 类。😈

③ com.alibaba.fastjson.support.spring.GenericFastJsonRedisSerializer ，使用 FastJSON 实现 JSON 的序列化方式，和 GenericJackson2JsonRedisSerializer 一致，就不重复赘述。

注意，GenericFastJsonRedisSerializer 不是 Spring Data Redis 内置实现，而是由于 FastJSON 自己实现。

④ com.alibaba.fastjson.support.spring.FastJsonRedisSerializer<T> ，使用 FastJSON 实现 JSON 的序列化方式，和 Jackson2JsonRedisSerializer 一致，就不重复赘述。

注意，GenericFastJsonRedisSerializer 不是 Spring Data Redis 内置实现，而是由于 FastJSON 自己实现。

### XML 序列化方式
`org.springframework.data.redis.serializer.OxmSerializer`，使用 Spring OXM 实现将对象和 String 的转换，从而 String 和二进制数组的转换。

因为 XML 序列化方式，暂时没有这么干过，我自己也没有，所以就直接忽略它吧。
## 配置序列化方式
创建`RedisConfiguration`配置类，代码如下：
```java
@Configuration
public class RedisConfiguration {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 创建 RedisTemplate 对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置 RedisConnection 工厂。😈 它就是实现多种 Java Redis 客户端接入的秘密工厂。感兴趣的胖友，可以自己去撸下。
        template.setConnectionFactory(factory);

        // 使用 String 序列化方式，序列化 KEY 。
        template.setKeySerializer(RedisSerializer.string());

        // 使用 JSON 序列化方式（库是 Jackson ），序列化 VALUE 。
        template.setValueSerializer(RedisSerializer.json());
        return template;
    }

}
```
RedisSerializer#string() 静态方法，返回的就是使用 UTF-8 编码的 StringRedisSerializer 对象。代码如下：
```java
// RedisSerializer.java
static RedisSerializer<String> string() {
	return StringRedisSerializer.UTF_8;
}

// StringRedisSerializer.java
public static final StringRedisSerializer ISO_8859_1 = new StringRedisSerializer(StandardCharsets.ISO_8859_1);
```
RedisSerializer#json() 静态方法，返回 GenericJackson2JsonRedisSerializer 对象。代码如下：
```
// RedisSerializer.java

static RedisSerializer<Object> json() {
	return new GenericJackson2JsonRedisSerializer();
}
```
## 自定义 RedisSerializer 实现类
我们直接以 GenericFastJsonRedisSerializer 举例子，直接莽源码。代码如下：
```java
// GenericFastJsonRedisSerializer.java

public class GenericFastJsonRedisSerializer implements RedisSerializer<Object> {
    private final static ParserConfig defaultRedisConfig = new ParserConfig();
    static { defaultRedisConfig.setAutoTypeSupport(true);}

    public byte[] serialize(Object object) throws SerializationException {
        // 空，直接返回空数组
        if (object == null) {
            return new byte[0];
        }
        try {
            // 使用 JSON 进行序列化成二进制数组，同时通过 SerializerFeature.WriteClassName 参数，声明写入类全名。
            return JSON.toJSONBytes(object, SerializerFeature.WriteClassName);
        } catch (Exception ex) {
            throw new SerializationException("Could not serialize: " + ex.getMessage(), ex);
        }
    }

    public Object deserialize(byte[] bytes) throws SerializationException {
        // 如果为空，则返回空对象
        if (bytes == null || bytes.length == 0) {
            return null;
        }
        try {
            // 使用 JSON 解析成对象。
            return JSON.parseObject(new String(bytes, IOUtils.UTF8), Object.class, defaultRedisConfig);
        } catch (Exception ex) {
            throw new SerializationException("Could not deserialize: " + ex.getMessage(), ex);
        }
    }
}
```
完成自定义 RedisSerializer 配置类后，我们就可以参照 「3.2 配置序列化方式」 小节，将 VALUE 序列化的修改成我们的，哈哈哈。

# 项目实践
## Cache Object
在我们使用数据库时，我们会创建 dataobject 包，存放 DO（Data Object）数据库实体对象。

那么同理，我们缓存对象，怎么进行对应呢？对于复杂的缓存对象，我们创建了 cacheobject 包，和 dataobject 包同层。如：

service # 业务逻辑层
dao # 数据库访问层
dataobject # DO
cacheobject # 缓存对象
并且所有的 Cache Object 对象使用 CacheObject 结尾，例如说 UserCacheObject、ProductCacheObject 。

## 数据访问层
在我们访问数据库时，我们会创建 dao 包，存放每个 DO 对应的 Dao 对应。那么对于每一个 CacheObject 类，我们也会创建一个其对应的 Dao 类。例如说，UserCacheObject 对应 UserCacheObjectDao 类。示例代码如下：
```java
@Repository
public class UserCacheDao {

    private static final String KEY_PATTERN = "user:%d"; // user:用户编号 <1>

    @Resource(name = "redisTemplate")
    @SuppressWarnings("SpringJavaInjectionPointsAutowiringInspection")
    private ValueOperations<String, String> operations; // <2>

    private static String buildKey(Integer id) { // <3>
        return String.format(KEY_PATTERN, id);
    }

    public UserCacheObject get(Integer id) {
        String key = buildKey(id);
        String value = operations.get(key);
        return JSONUtil.parseObject(value, UserCacheObject.class);
    }

    public void set(Integer id, UserCacheObject object) {
        String key = buildKey(id);
        String value = JSONUtil.toJSONString(object);
        operations.set(key, value);
    }

}
```
<1> 处，通过静态变量，声明 KEY 的前缀，并且使用冒号作为间隔
<3> 处，声明 KEY_PATTERN 对应的 KEY 拼接方法，避免散落在每个方法中。
<2> 处，通过 @Resource 注入指定名字的 RedisTemplate 对应的 Operations 对象，这样明确每个 KEY 的类型。
剩余的，就是每个方法封装对应的操作。
可能会有胖友问，为什么不支持将 RedisTemplate 直接 Service 业务层调用呢？如果这样，我们业务代码里，就容易混杂着很多 Redis 访问代码的细节，导致很脏乱。我们试着把 RedisTemplate 想象成 Spring JDBCTemplate ，我们一定会声明对应的 Dao 类，访问数据库。所以，同理落。

那么还有一个问题，UserCacheDao 放在哪个包下？目前的想法是，将 dao 包下拆成 mysql、redis 包。这样，MySQL 相关的 Dao 放在 mysql 包下，Redis 相关的 Dao 放在 redis 。
## 序列化
在 「3. 序列化」 小节中，我们仔细翻看了每个序列化方式，暂时没有一个能够完美的契合我们的需求，所以我们直接使用最简单的 StringRedisSerializer 作为序列化实现类。而真正的序列化，我们在各个 Dao 类里，自己手动来调用。

例如说，在 UserCacheDao 示例中，已经看到了这么做了。这里还有一个细化点，虽然我们是自己手动序列化，可以自己简单封装一个 JSONUtil 类，未来如果我们想换 JSON 库，就比较方便了。其实，这个和 Spring Data Redis 所做的封装是一个思路。
# 示例补充
像 String、List、Set、ZSet、Geo、HyperLogLog 等等数据结构的操作，胖友自己去用用对应的 Operations 操作类的 API 方法，就非常容易懂了，我们更多的，补充 Pipeline、Transaction、Pub/Sub、Script 等等功能的示例。
## Pipeline
如果胖友没有了解过 Redis 的 Pipeline 机制，可以看看 《Redis 文档 —— Pipeline》 文章，批量操作，提升性能必备神器。

在 RedisTemplate 类中，提供了 2 组四个方法，用于执行 Redis Pipeline 操作。代码如下：
```java
// <1> 基于 Session 执行 Pipeline
@Override
public List<Object> executePipelined(SessionCallback<?> session) {
	return executePipelined(session, valueSerializer);
}
@Override
public List<Object> executePipelined(SessionCallback<?> session, @Nullable RedisSerializer<?> resultSerializer) {
    // ... 省略代码
}

// <2> 直接执行 Pipeline
@Override
public List<Object> executePipelined(RedisCallback<?> action) {
	return executePipelined(action, valueSerializer);
}
@Override
public List<Object> executePipelined(RedisCallback<?> action, @Nullable RedisSerializer<?> resultSerializer) {
    // ... 省略代码
}
```
两组方法的差异，在于是否是 Session 中执行。那么 Session 是什么呢？卖个关子，在 「5.3 Session」 中来详细解析。本小节，我们只讲 Pipeline + RedisCallback 的组合的方法。
每组方法里，差别在于是否传入 RedisSerializer 参数。如果不传，则使用 RedisTemplate 自己的序列化相关的属性。
5.1.1 源码解读
在看具体的 #executePipelined(RedisCallback<?> action, ...) 方法的示例之前，我们先来看一波源码，这样我们才能更好的理解具体的使用方法。代码如下：
```java
// RedisTemplate.java
@Override
public List<Object> executePipelined(RedisCallback<?> action, @Nullable RedisSerializer<?> resultSerializer) {
	// <1> 执行 Redis 方法
	return execute((RedisCallback<List<Object>>) connection -> {
		// <2> 打开 pipeline
		connection.openPipeline();
		boolean pipelinedClosed = false; // 标记 pipeline 是否关闭
		try {
			// <3> 执行
			Object result = action.doInRedis(connection);
			// <4> 不要返回结果
			if (result != null) {
				throw new InvalidDataAccessApiUsageException(
						"Callback cannot return a non-null value as it gets overwritten by the pipeline");
			}
			// <5> 提交 pipeline 执行
			List<Object> closePipeline = connection.closePipeline();
			pipelinedClosed = true;
			// <6> 反序列化结果，并返回
			return deserializeMixedResults(closePipeline, resultSerializer, hashKeySerializer, hashValueSerializer);
		} finally {
			if (!pipelinedClosed) {
				connection.closePipeline();
			}
		}
	});
}
```
<1> 处，调用 #execute(RedisCallback<T> action) 方法，执行 Redis 方法。注意，此处传入的 action 参数，不是我们传入的 RedisCallback 参数。我们的会在该 action 中被执行。
<2> 处，调用 RedisConnection#openPipeline() 方法，自动打开 Pipeline 模式。这样，我们就不需要手动去打开了。
<3> 处，调用我们传入的实现的 RedisCallback#doInRedis(RedisConnection connection) 方法，执行在 Pipeline 中，想要执行的 Redis 操作。
<4> 处，不要返回结果。因为 RedisCallback 是统一定义的接口，所以可以返回一个结果。但是在 Pipeline 中，未提交执行时，显然是没有结果，返回也没有意思。简单来说，就是我们在实现 RedisCallback#doInRedis(RedisConnection connection) 方法时，返回 null 即可。
<5> 处，调用 RedisConnection#closePipeline() 方法，自动提交 Pipeline 执行，并返回执行结果。
<6> 处，反序列化结果，并返回 Pipeline 结果。
至此，Spring Data Redis 对 Pipeline 的封装，我们已经做了一个简单的了解，实际就是经典的“模板方法”设计模式化的应用。下面，在让我们来看看 org.springframework.data.redis.core.RedisCallback<T> 接口，Redis 回调接口。代码如下：
```java
// RedisCallback.java
public interface RedisCallback<T> {

	/**
	 * Gets called by {@link RedisTemplate} with an active Redis connection. Does not need to care about activating or
	 * closing the connection or handling exceptions.
	 *
	 * @param connection active Redis connection
	 * @return a result object or {@code null} if none
	 * @throws DataAccessException
	 */
	@Nullable
	T doInRedis(RedisConnection connection) throws DataAccessException;
}
```
虽然接口名是以 Callback 结尾，但是通过 #doInRedis(RedisConnection connection) 方法可以很容易知道，实际可以理解是 Redis Action ，想要执行的 Redis 操作。

有一点要注意，传入的 connection 参数是 RedisConnection 对象，它提供的 'low level' 更底层的 Redis API 操作。例如说：
```
// RedisStringCommands.java
// RedisConnection 实现 RedisStringCommands 接口

byte[] get(byte[] key);

Boolean set(byte[] key, byte[] value);
```
传入和返回的是二进制数组，实际就是 RedisTemplate 已经序列化的入参和会被反序列化的出参。
5.1.2 具体示例
示例代码对应测试类：PipelineTest 。

创建 PipelineTest 单元测试类，编写代码如下：
```java
// PipelineTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class PipelineTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() {
        List<Object> results = stringRedisTemplate.executePipelined(new RedisCallback<Object>() {

            @Override
            public Object doInRedis(RedisConnection connection) throws DataAccessException {
                // set 写入
                for (int i = 0; i < 3; i++) {
                    connection.set(String.format("yunai:%d", i).getBytes(), "shuai".getBytes());
                }

                // get
                for (int i = 0; i < 3; i++) {
                    connection.get(String.format("yunai:%d", i).getBytes());
                }

                // 返回 null 即可
                return null;
            }
        });

        // 打印结果
        System.out.println(results);
    }
}
```
执行 #test01() 方法，结果如下：

[true, true, true, shuai, shuai, shuai]
因为我们使用 StringRedisTemplate 自己的序列化相关属性，所以 Redis GET 命令返回的二进制，被反序列化成了字符串。

## Transaction
基情提示：实际项目实战中，Redis Transaction 事务基本不用，至少问了一些胖友，包括自己，都没有再用。所以呢，本小节可以选择性看看。或者，就不看，哈哈哈哈。

在看 Redis Transaction 事务之前，我们先回想下 Spring 是如何管理数据库 Transaction 的。在应用程序中处理一个请求时，如果我们的方法开启Trasaction 功能，Spring 会把数据库的 Connection 连接和当前线程进行绑定，从而实现 Connection 打开一个 Transaction 后，所有当前线程的数据库操作都在该 Connection 上执行，达到所有操作在这个 Transaction 中，最终提交或回滚。

在 Spring Data Redis 中，实现 Redis Transaction 也是这个思路。通过 SessionCallback 操作 Redis 时，会从当前线程获得 Redis Connection ，如果获取不到，则会去“创建”一个 Redis Connection 并绑定到当前线程中。这样，我们在该 Redis Connection 开启 Redis Transaction 后，在该线程的所有操作，都可以在这个 Transaction 中，最后交由 Spring 事务管理器统一提供或回滚 Transaction 。

如果想要使用 Redis Transaction 功能，需要创建 RedisTemplate Bean 时，设置其 enableTransactionSupport 属性为 true ，默认为 false 不开启。示例如下：
```java
@Configuration
public class RedisConfiguration {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        // 创建 RedisTemplate 对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();

        // 【重要】设置开启事务支持
        template.setEnableTransactionSupport(true);

        // 设置 RedisConnection 工厂。😈 它就是实现多种 Java Redis 客户端接入的秘密工厂。感兴趣的胖友，可以自己去撸下。
        template.setConnectionFactory(factory);

        // 使用 String 序列化方式，序列化 KEY 。
        template.setKeySerializer(RedisSerializer.string());

        // 使用 JSON 序列化方式（库是 Jackson ），序列化 VALUE 。
        template.setValueSerializer(RedisSerializer.json());
        return template;
    }

}
```
5.2.1 源码解析
概念和原理层面的东西，一旦复杂，就会特别抽象，那么还是老规矩，让我们一起撸下源码，让原理具象化。很多时候，这就是为什么我们要去撸源码的意义。

我们先来看看，配置下 enableTransactionSupport 属性，Redis 在执行命令，是如何获得 Connection 连接的。代码如下：
```java
// RedisTemplate.java

public <T> T execute(RedisCallback<T> action, boolean exposeConnection, boolean pipeline) {

	Assert.isTrue(initialized, "template not initialized; call afterPropertiesSet() before using it");
	Assert.notNull(action, "Callback object must not be null");

	RedisConnectionFactory factory = getRequiredConnectionFactory();
	RedisConnection conn = null;
	try {
		// <1.1>
		if (enableTransactionSupport) {
			// only bind resources in case of potential transaction synchronization
			conn = RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
		} else {
			// <1.2>
			conn = RedisConnectionUtils.getConnection(factory);
		}

		// ... 省略中间，执行 Redis 命令的代码。
	} finally {
		// <2>
		RedisConnectionUtils.releaseConnection(conn, factory);
	}
}
```
<1.2> 处，当我们未开启`enableTransactionSupport`事务时，调用`RedisConnectionUtils#getConnection(factory)`方法，获得 Redis Connection 。如果获取不到，则进行创建。

<1.1> 处，当我们有开启`enableTransactionSupport`事务时，调用`RedisConnectionUtils#bindConnection(RedisConnectionFactory factory, boolean enableTranactionSupport)`方法，在 RedisConnectionUtils#getConnection(factory) 的基础上，如果是创建的 Redis Connection ，会绑定到当前啊线程中。因为 Transaction 是需要在 Connection 打开，然后后续的 Redis 的操作，都需要在其上。并且，还有一个非常重要的操作，打开 Redis Transaction ，会在该方法中，通过调用 RedisConnectionUtils#potentiallyRegisterTransactionSynchronisation(RedisConnectionHolder connHolder, final RedisConnectionFactory factory) 。

<2> 处，调用 RedisConnectionUtils#releaseConnection(RedisConnection conn, RedisConnectionFactory factory) 方法，释放 Redis Connection 。当然，这是有一个前提，整个 Transaction 已经完成。如果未完成，实际 Redis Connection 不会释放。

那么，此时会有胖友有疑问，Redis Transaction 的提交和回滚在哪呢？答案在 RedisConnectionUtils 的内部类 RedisTransactionSynchronizer 中。代码如下：
```java
// RedisConnectionUtils.java

private static class RedisTransactionSynchronizer extends TransactionSynchronizationAdapter {

	private final RedisConnectionHolder connHolder;
	private final RedisConnection connection;
	private final RedisConnectionFactory factory;

	@Override
	public void afterCompletion(int status) {

		try {
			switch (status) {
				// 提交
				case TransactionSynchronization.STATUS_COMMITTED:
					connection.exec();
					break;
				// 回滚
				case TransactionSynchronization.STATUS_ROLLED_BACK:
				case TransactionSynchronization.STATUS_UNKNOWN:
				default:
					connection.discard();
			}
		} finally {
			connHolder.setTransactionSyncronisationActive(false);
			connection.close();
			TransactionSynchronizationManager.unbindResource(factory);
		}
	}
}
```
根据事务结果的状态，进行 Redis Transaction 提交或回滚。
5.2.2 具体示例
示例代码对应测试类：TransactionTest 。

创建 TransactionTest 单元测试类，编写代码如下：
```java
// TransactionTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class TransactionTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
//    @Transactional
    public void test01() {
        // 这里是偷懒，没在 RedisConfiguration 配置类中，设置 stringRedisTemplate 开启事务。
        stringRedisTemplate.setEnableTransactionSupport(true);

        // 执行想要的操作
        stringRedisTemplate.opsForValue().set("yunai:1", "shuai");
        stringRedisTemplate.opsForValue().set("yudaoyuanma:1", "dai");
    }
}
```
目前这仅仅是一个示例。因为 Redis Transaction 实际创建事务的前提，是当前已经存在 Spring Transaction 。具体可以看看传送门处的判断的代码。
## Session
`Session`不是 Redis 的功能，而是 Spring Data Redis 封装的一个功能。一次`Session`，代表通过同一个 Redis Connection 执行一系列的 Redis 操作。

如果我们在一个 Redis Transaction 中的时候，所有 Redis 操作都使用通过同一个 Redis Connection ，因为我们会将获得到的 Connection 绑定到当前线程中。

但是，如果我们不在一个 Redis Transaction 中的时候，我们每一次使用 Redis Operations 执行 Redis 操作的时候，每一次都会获取一次 Redis Connection 的获取。实际项目中，我们必然会使用 Redis Connection 连接池，那么在获取的时候，会存在一定的竞争，会有资源上的消耗。那么，如果我们希望如果已知我们要执行一个系列的 Redis 操作，能不能使用同一个 Redis Connection ，避免重复获取它呢？答案是有，那就是 Session 。

当我们要执行在同一个 Session 里的操作时，我们通过实现`org.springframework.data.redis.core.SessionCallback<T>`接口，其代码如下：
```java
// SessionCallback.java

public interface SessionCallback<T> {

	@Nullable
	<K, V> T execute(RedisOperations<K, V> operations) throws DataAccessException;
}
```
相比 RedisCallback 来说，总比是比较相似的。但是比较友好的是，它的入参 operations 是 org.springframework.data.redis.core.RedisOperations 接口类型，而 RedisTemplate 的各种操作，实际就是在 RedisOperations 接口中定义，由 RedisTemplate 来实现。所以使用上也会更加便利。
实际上，我们在实现 RedisCallback 接口，也能实现在同一个 Connection 执行一系列的 Redis 操作，因为 RedisCallback 的入参本身就是一个 Redis Connection 。
5.3.1 源码解析
在生产中，Transaction 和 Pipeline 会经常一起时候用，从而提升性能。所以在 RedisTemplate#executePipelined(SessionCallback<?> session, ...) 方法中，提供了这种的功能。而在这个方法的实现上，本质和 RedisTemplate#executePipelined(RedisCallback<?> action, ...) 方法是基本一致的，差别在于这一行 ，替换成了调用 #executeSession(SessionCallback<?> session) 方法。所以，我们来直接来看被调用的这个方法的实现。代码如下：
```java
// RedisTemplate.java

@Override
public <T> T execute(SessionCallback<T> session) {

	Assert.isTrue(initialized, "template not initialized; call afterPropertiesSet() before using it");
	Assert.notNull(session, "Callback object must not be null");

	RedisConnectionFactory factory = getRequiredConnectionFactory();
	// bind connection
	// <1> 获得并绑定 Connection 。
	RedisConnectionUtils.bindConnection(factory, enableTransactionSupport);
	try {
	   // <2> 执行定义的一系列 Redis 操作
		return session.execute(this);
	} finally {
		// <3> 释放并解绑 Connection 。
		RedisConnectionUtils.unbindConnection(factory);
	}
}
```
<1> 处，调用 RedisConnectionUtils#bindConnection(RedisConnectionFactory factory, boolean enableTranactionSupport) 方法，实际和我们开启 enableTranactionSupport 事务时候，获取 Connection 和处理的方式，是一模一样的。也就是说：
如果当前线程已经有一个绑定的 Connection 则直接使用（例如说，当前正在 Redis Transaction 事务中）；
如果当前线程未绑定一个 Connection ，则进行创建并绑定到当前线程。甚至，如果此时是配置开启 enableTranactionSupport 事务的，那么此处就会触发 Redis Transaction 的开启。
<2> 处，调用 SessionCallback#execute(RedisOperations<K, V> operations) 方法，执行我们定义的一系列的 Redis 操作。看看此处传入的参数是 this ，是不是仿佛更加明白点什么了？
<3> 处，调用 RedisConnectionUtils#unbindConnection(RedisConnectionFactory factory) 方法，释放并解绑 Connection 。当前，前提是当前不存在激活的 Redis Transaction ，不然不就提早释放了嘛。
恩，现在胖友在回过头，好好在想一想 Pipeline、Transaction、Session 之间的关系，以及组合排列。之后，我们在使用上，会更加得心应手。

5.3.2 具体示例
示例代码对应测试类：SessionTest 。

创建 SessionTest 单元测试类，编写代码如下：
```java
// SessionTest.java

@RunWith(SpringRunner.class)
@SpringBootTest
public class SessionTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() {
        String result = stringRedisTemplate.execute(new SessionCallback<String>() {

            @Override
            public String execute(RedisOperations operations) throws DataAccessException {
                for (int i = 0; i < 100; i++) {
                    operations.opsForValue().set(String.format("yunai:%d", i), "shuai02");
                }
                return (String) operations.opsForValue().get(String.format("yunai:%d", 0));
            }

        });

        System.out.println("result:" + result);
    }

}
```
执行 #test01() 方法，结果如下：

result:shuai02
卧槽，一直被 Redis 夸奖，已经超级不好意思了。

## Pub/Sub
Redis 提供了`Pub/Sub`功能，实现简单的订阅功能。
### 具体示例
示例代码对应测试类：PubSubTest 。

Spring Data Redis 实现 Pub/Sub 的示例，主要分成两部分：

配置 RedisMessageListenerContainer Bean 对象，并添加我们自己实现的 MessageListener 对象，用于监听处理相应的消息。
使用 RedisTemplate 发布消息。
下面，我们通过四个小步骤，来实现一个简单的示例。

第一步，了解 Topic

`org.springframework.data.redis.listener.Topic`接口，表示 Redis 消息的 Topic 。它有两个子类实现：

ChannelTopic ：对应 SUBSCRIBE 订阅命令。
PatternTopic ：对应 PSUBSCRIBE 订阅命令。
第二步，实现 MessageListener 类

创建 TestChannelTopicMessageListener 类，编写代码如下：
```java
public class TestPatternTopicMessageListener implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] pattern) {
        System.out.println("收到 PatternTopic 消息：");
        System.out.println("线程编号：" + Thread.currentThread().getName());
        System.out.println("message：" + message);
        System.out.println("pattern：" + new String(pattern));
    }

}
```
message 参数，可获得到具体的消息内容，不过是二进制数组，需要我们自己序列化。具体可以看下 org.springframework.data.redis.connection.DefaultMessage 类。
pattern 参数，发布的 Topic 的内容。
有一点要注意，默认的 RedisMessageListenerContainer 情况下，MessageListener 是并发消费，在线程池中执行(具体见传送门代码)。所以如果想相同 MessageListener 串行消费，可以在方法上加 synchronized 修饰，来实现同步。

第三步，创建 RedisMessageListenerContainer Bean

在 RedisConfiguration 中，配置 RedisMessageListenerContainer Bean 。代码如下：
```java
// RedisConfiguration.java

@Bean
public RedisMessageListenerContainer listenerContainer(RedisConnectionFactory factory) {
    // 创建 RedisMessageListenerContainer 对象
    RedisMessageListenerContainer container = new RedisMessageListenerContainer();

    // 设置 RedisConnection 工厂。😈 它就是实现多种 Java Redis 客户端接入的秘密工厂。感兴趣的胖友，可以自己去撸下。
    container.setConnectionFactory(factory);

    // 添加监听器
    container.addMessageListener(new TestChannelTopicMessageListener(), new ChannelTopic("TEST"));
//        container.addMessageListener(new TestChannelTopicMessageListener(), new ChannelTopic("AOTEMAN"));
//        container.addMessageListener(new TestPatternTopicMessageListener(), new PatternTopic("TEST"));
    return container;
}
```
要注意，虽然 RedisConnectionFactory 可以多次调用 #addMessageListener(MessageListener listener, Topic topic) 方法，但是一定要都是相同的 Topic 类型。例如说，添加了 ChannelTopic 类型，就不能添加 PatternTopic 类型。为什么呢？因为 RedisMessageListenerContainer 是基于一次 SUBSCRIBE 或 PSUBSCRIBE 命令，所以不支持不同类型的 Topic 。当然，如果是相同类型的 Topic ，多个 MessageListener 是支持的。

那么，可能会有胖友会问，如果我添加了 "Test" 给 MessageListenerA ，"AOTEMAN" 给 MessageListenerB ，两个 Topic 是怎么分发（Dispatch）的呢？在 RedisMessageListenerContainer 中，有个 DispatchMessageListener 分发器，负责将不同的 Topic 分发到配置的 MessageListener 中。看到此处，有木有想到 Spring MVC 的 DispatcherServlet 分发不同的请求到对应的 @RequestMapping 方法。

第四步，使用 RedisTemplate 发布消息

创建 PubSubTest 测试类，编写代码如下：
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PubSubTest {

    public static final String TOPIC = "TEST";

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            stringRedisTemplate.convertAndSend(TOPIC, "yunai:" + i);
            Thread.sleep(1000L);
        }
    }

}
```
通过 RedisTemplate#convertAndSend(String channel, Object message) 方法，PUBLISH 消息。
执行 #test01() 方法，运行结果如下：
```
收到 ChannelTopic 消息：
线程编号：listenerContainer-2
message：yunai:0
pattern：TEST
收到 ChannelTopic 消息：
线程编号：listenerContainer-3
message：yunai:1
pattern：TEST
收到 ChannelTopic 消息：
线程编号：listenerContainer-4
message：yunai:2
pattern：TEST
```
整整齐齐，发送和订阅都成功了。注意，线程编号。
5.4.3 闲话两句
Redis 提供了 PUB/SUB 订阅功能，实际我们在使用时，一定要注意，它提供的不是一个可靠的订阅系统。例如说，有消息 PUBLISH 了，Redis Client 因为网络异常断开，无法订阅到这条消息。等到网络恢复后，Redis Client 重连上后，是无法获得到该消息的。相比来说，成熟的消息队列提供的订阅功能，因为消息会进行持久化（Redis 是不持久化 Publish 的消息的），并且有客户端的 ACK 机制做保障，所以即使网络断开重连，消息一样不会丢失。

Redis 5.0 版本后，正式发布 Stream 功能，相信是有可能可以替代掉 Redis Pub/Sub 功能，提供可靠的消息订阅功能。

上述的场景，艿艿自己在使用 PUB/SUB 功能的时候，确实被这么坑过。当时我们的管理后台的权限，是缓存在 Java 进程当中，通过 Redis Pub/Sub 实现缓存的刷新。结果，当时某个 Java 节点网络出问题，恰好那个时候，有一条刷新权限缓存的消息 PUBLISH 出来，结果没刷新到。结果呢，运营在访问某个功能的时候，一会有权限（因为其他 Java 节点缓存刷新了），一会没有权限。

最近，艿艿又去找了几个朋友请教了下，问问他们在生产环境下，是否使用 Redis Pub/Sub 功能，他们说使用 Kafka、或者 RocketMQ 的广播消费功能，更加可靠有保障。

对了，我们有个管理系统里面有 Websocket 需要实时推送管理员消息，因为不知道管理员当前连接的是哪个 Websocket 服务节点，所以我们是通过 Redis Pub/Sub 功能，广播给所有 Websocket 节点，然后每个 Websocket 节点判断当前管理员是否连接的是它，如果是，则进行 Websocket 推送。因为之前网络偶尔出故障，会存在消息丢失，所以近期我们替换成了 RocketMQ 的广播消费，替代 Redis Pub/Sub 功能。

当然，不能说 Redis Pub/Sub 毫无使用的场景，以下艿艿来列举几个：

1、在使用 Redis Sentinel 做高可用时，Jedis 通过 Redis Pub/Sub 功能，实现对 Redis 主节点的故障切换，刷新 Jedis 客户端的主节点的缓存。如果出现 Redis Connection 订阅的异常断开，会重新主动去 Redis Sentinel 的最新主节点信息，从而解决 Redis Pub/Sub 可能因为网络问题，丢失消息。
2、Redis Sentinel 节点之间的部分信息同步，通过 Redis Pub/Sub 订阅发布。
3、在我们实现 Redis 分布式锁时，如果获取不到锁，可以通过 Redis 的 Pub/Sub 订阅锁释放消息，从而实现其它获得不到锁的线程，快速抢占锁。当然，Redis Client 释放锁时，需要 PUBLISH 一条释放锁的消息。在 Redisson 实现分布式锁的源码中，我们可以看到。
4、Dubbo 使用 Redis 作为注册中心时，使用 Redis Pub/Sub 实现注册信息的同步。
也就是说，如果想要有保障的使用 Redis Pub/Sub 功能，需要处理下发起订阅的 Redis Connection 的异常，例如说网络异常。然后，重新主动去查询最新的数据的状态。😈

## Script
Redis 提供 Lua 脚本，满足我们希望组合排列使用 Redis 的命令，保证串行执行的过程中，不存在并发的问题。同时，通过将多个命令组合在同一个 Lua 脚本中，一次请求，直接处理，也是一个提升性能的手段。不了解的胖友，可以看看 「Redis 文档 —— Lua 脚本」 。

下面，我们来看看 Spring Data Redis 使用 Lua 脚本的示例。

示例代码对应测试类：ScriptTest 。

第一步，编写 Lua 脚本

创建 resources/compareAndSet.lua 脚本，实现 CAS 功能。代码如下：
```
if redis.call('GET', KEYS[1]) ~= ARGV[1] then
    return 0
end
redis.call('SET', KEYS[1], ARGV[2])
return 1
```
第 1 到 3 行：判断 KEYS[1] 对应的 VALUE 是否为 ARGV[1] 值。如果不是（Lua 中不等于使用 ~=），则直接返回 0 表示失败。
第 4 到 5 行：设置 KEYS[1] 对应的 VALUE 为新值 ARGV[2] ，并返回 1 表示成功。
第二步，调用 Lua 脚本

创建 ScriptTest 测试类，编写代码如下：
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ScriptTest {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test01() throws IOException {
        // <1.1> 读取 /resources/lua/compareAndSet.lua 脚本 。注意，需要引入下 commons-io 依赖。
        String  scriptContents = IOUtils.toString(getClass().getResourceAsStream("/lua/compareAndSet.lua"), "UTF-8");
        // <1.2> 创建 RedisScript 对象
        RedisScript<Long> script = new DefaultRedisScript<>(scriptContents, Long.class);
        // <2> 执行 LUA 脚本
        Long result = stringRedisTemplate.execute(script, Collections.singletonList("yunai:1"), "shuai02", "shuai");
        System.out.println(result);
    }
}
```
<1.1> 行，读取 /resources/lua/compareAndSet.lua 脚本。注意，需要引入下 commons-io 依赖。
<1.2> 行，创建 DefaultRedisScript 对象。第一个参数是脚本内容( scriptSource )，第二个是脚本执行返回值( resultType )。
<2> 处，调用 RedisTemplate#execute(RedisScript<T> script, List<K> keys, Object... args) 方法，发送 Redis 执行 LUA 脚本。
最后，我们打印下执行结果。胖友可以自己执行下试试。😈

