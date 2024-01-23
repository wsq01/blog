


在 MyBatis 初始化过程中，会加载`mybatis-config.xml`配置文件、映射配置文件以及`Mapper`接口中的注解信息，解析后的配置信息会形成相应的对象并保存到`Configuration`对象中。例如：
* `<resultMap>`节点(即`ResultSet`的映射规则) 会被解析成`ResultMap`对象。
* `<result>`节点(即属性映射)会被解析成`ResultMapping`对象。

之后，利用该`Configuration`对象创建`SqlSessionFactory`对象。待 MyBatis 初始化之后，开发人员可以通过初始化得到`SqlSessionFactory`创建`SqlSession`对象并完成数据库操作。

对应`builder`模块，为配置解析过程。
对应`mapping`模块，主要为 SQL 操作解析后的映射。

MyBatis 的初始化流程的入口是`SqlSessionFactoryBuilder`的`build(Reader reader, String environment, Properties properties)`方法，代码如下：

`SqlSessionFactoryBuilder`中，`build`方法有多种重载方式。这里就选取一个。
```java
// SqlSessionFactoryBuilder.java

/**
 * 构造 SqlSessionFactory 对象
 *
 * @param reader Reader 对象
 * @param environment 环境
 * @param properties Properties 变量
 * @return SqlSessionFactory 对象
 */
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
        // <1> 创建 XMLConfigBuilder 对象
        XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
        // <2> 执行 XML 解析
        // <3> 创建 DefaultSqlSessionFactory 对象
        return build(parser.parse());
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
        ErrorContext.instance().reset();
        try {
            reader.close();
        } catch (IOException e) {
            // Intentionally ignore. Prefer previous error.
        }
    }
}
```
* <2> 处，调用`XMLConfigBuilder#parse()`方法，执行 XML 解析，返回`Configuration`对象。
* <3> 处，创建`DefaultSqlSessionFactory`对象。

本文的重点是 <1> 和 <2> 处，即`XMLConfigBuilder`类。
# BaseBuilder
`org.apache.ibatis.builder.BaseBuilder`，基础构造器抽象类，为子类提供通用的工具类。

`BaseBuilder`是`XMLConfigBuilder`的父类，并且它还有其他的子类。如下图所示：

{% asset_img 1.png BaseBuilder 类图 %}

## 构造方法
```java
// BaseBuilder.java

/**
 * MyBatis Configuration 对象
 */
protected final Configuration configuration;
protected final TypeAliasRegistry typeAliasRegistry;
protected final TypeHandlerRegistry typeHandlerRegistry;

public BaseBuilder(Configuration configuration) {
    this.configuration = configuration;
    this.typeAliasRegistry = this.configuration.getTypeAliasRegistry();
    this.typeHandlerRegistry = this.configuration.getTypeHandlerRegistry();
}
```
`configuration`属性，MyBatis `Configuration`对象。XML 和注解中解析到的配置，最终都会设置到`org.apache.ibatis.session.Configuration`中。
## parseExpression
`parseExpression(String regex, String defaultValue)`方法，创建正则表达式。
```java
// BaseBuilder.java

/**
 * 创建正则表达式
 *
 * @param regex 指定表达式
 * @param defaultValue 默认表达式
 * @return 正则表达式
 */
@SuppressWarnings("SameParameterValue")
protected Pattern parseExpression(String regex, String defaultValue) {
    return Pattern.compile(regex == null ? defaultValue : regex);
}
```
## xxxValueOf
`xxxValueOf(...)`方法，将字符串转换成对应的数据类型的值。代码如下：
```java
// BaseBuilder.java

protected Boolean booleanValueOf(String value, Boolean defaultValue) {
    return value == null ? defaultValue : Boolean.valueOf(value);
}

protected Integer integerValueOf(String value, Integer defaultValue) {
    return value == null ? defaultValue : Integer.valueOf(value);
}

protected Set<String> stringSetValueOf(String value, String defaultValue) {
    value = (value == null ? defaultValue : value);
    return new HashSet<>(Arrays.asList(value.split(",")));
}
```
## resolveJdbcType
`resolveJdbcType(String alias)`方法，解析对应的`JdbcType`类型。
```java
// BaseBuilder.java

protected JdbcType resolveJdbcType(String alias) {
    if (alias == null) {
        return null;
    }
    try {
        return JdbcType.valueOf(alias);
    } catch (IllegalArgumentException e) {
        throw new BuilderException("Error resolving JdbcType. Cause: " + e, e);
    }
}
```
## resolveResultSetType
`resolveResultSetType(String alias)`方法，解析对应的`ResultSetType`类型。
```java
// BaseBuilder.java

protected ResultSetType resolveResultSetType(String alias) {
    if (alias == null) {
        return null;
    }
    try {
        return ResultSetType.valueOf(alias);
    } catch (IllegalArgumentException e) {
        throw new BuilderException("Error resolving ResultSetType. Cause: " + e, e);
    }
}
```
## resolveParameterMode
`resolveParameterMode(String alias)`方法，解析对应的`ParameterMode`类型。
```java
// BaseBuilder.java

protected ParameterMode resolveParameterMode(String alias) {
    if (alias == null) {
        return null;
    }
    try {
        return ParameterMode.valueOf(alias);
    } catch (IllegalArgumentException e) {
        throw new BuilderException("Error resolving ParameterMode. Cause: " + e, e);
    }
}
```
## createInstance
`createInstance(String alias)`方法，创建指定对象。
```java
// BaseBuilder.java

protected Object createInstance(String alias) {
    // <1> 获得对应的类型
    Class<?> clazz = resolveClass(alias);
    if (clazz == null) {
        return null;
    }
    try {
        // <2> 创建对象
        return resolveClass(alias).newInstance(); // 这里重复获得了一次
    } catch (Exception e) {
        throw new BuilderException("Error creating instance. Cause: " + e, e);
    }
}
```
<1> 处，调用`resolveClass(String alias)`方法，获得对应的类型。
```java
// BaseBuilder.java

protected <T> Class<? extends T> resolveClass(String alias) {
    if (alias == null) {
        return null;
    }
    try {
        return resolveAlias(alias);
    } catch (Exception e) {
        throw new BuilderException("Error resolving class. Cause: " + e, e);
    }
}

protected <T> Class<? extends T> resolveAlias(String alias) {
    return typeAliasRegistry.resolveAlias(alias);
}
```
从`typeAliasRegistry`中，通过别名或类全名，获得对应的类。

<2> 处，创建对象。
## resolveTypeHandler
`resolveTypeHandler(Class<?> javaType, String typeHandlerAlias)`方法，从`typeHandlerRegistry`中获得或创建对应的`TypeHandler`对象。
```java
// BaseBuilder.java

protected TypeHandler<?> resolveTypeHandler(Class<?> javaType, Class<? extends TypeHandler<?>> typeHandlerType) {
    if (typeHandlerType == null) {
        return null;
    }
    // javaType ignored for injected handlers see issue #746 for full detail
    // 先获得 TypeHandler 对象
    TypeHandler<?> handler = typeHandlerRegistry.getMappingTypeHandler(typeHandlerType);
    if (handler == null) { // 如果不存在，进行创建 TypeHandler 对象
        // not in registry, create a new one
        handler = typeHandlerRegistry.getInstance(javaType, typeHandlerType);
    }
    return handler;
}
```
# XMLConfigBuilder
`org.apache.ibatis.builder.xml.XMLConfigBuilder`，继承`BaseBuilder`抽象类，XML 配置构建器，主要负责解析`mybatis-config.xml`配置文件。
## 构造方法
```java
// XMLConfigBuilder.java

/**
 * 是否已解析
 */
private boolean parsed;
/**
 * 基于 Java XPath 解析器
 */
private final XPathParser parser;
/**
 * 环境
 */
private String environment;
/**
 * ReflectorFactory 对象
 */
private final ReflectorFactory localReflectorFactory = new DefaultReflectorFactory();

public XMLConfigBuilder(Reader reader) {
    this(reader, null, null);
}

public XMLConfigBuilder(Reader reader, String environment) {
    this(reader, environment, null);
}

public XMLConfigBuilder(Reader reader, String environment, Properties props) {
    this(new XPathParser(reader, true, props, new XMLMapperEntityResolver()), environment, props);
}

public XMLConfigBuilder(InputStream inputStream) {
    this(inputStream, null, null);
}

public XMLConfigBuilder(InputStream inputStream, String environment) {
    this(inputStream, environment, null);
}

public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
    this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
}

private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    // <1> 创建 Configuration 对象
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    // <2> 设置 Configuration 的 variables 属性
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
}
```
`parser`属性，`XPathParser`对象。

`localReflectorFactory`属性，`DefaultReflectorFactory`对象。

构造方法重载了比较多，只需要看最后一个。
* <1> 处，创建`Configuration`对象。
* <2> 处，设置`Configuration`对象的`variables`属性。代码如下：
```java
// Configuration.java

/**
 * 变量 Properties 对象。
 *
 * 参见 {@link org.apache.ibatis.builder.xml.XMLConfigBuilder#propertiesElement(XNode context)} 方法
 */
protected Properties variables = new Properties();

public void setVariables(Properties variables) {
    this.variables = variables;
}
```
## parse
`parse()`方法，解析 XML 成`Configuration`对象。
```java
// XMLConfigBuilder.java

public Configuration parse() {
    // <1.1> 若已解析，抛出 BuilderException 异常
    if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    // <1.2> 标记已解析
    parsed = true;
    // <2> 解析 XML configuration 节点
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
}
```
<1.1> 处，若已解析，抛出`BuilderException`异常。
<1.2> 处，标记已解析。
<2> 处，调用`XPathParser#evalNode(String expression)`方法，获得 XML `<configuration />`节点，后调用`parseConfiguration(XNode root)`方法，解析该节点。
## parseConfiguration
`parseConfiguration(XNode root)`方法，解析`<configuration />`节点。
```java
// XMLConfigBuilder.java

private void parseConfiguration(XNode root) {
    try {
        //issue #117 read properties first
        // 解析 <properties /> 标签
        propertiesElement(root.evalNode("properties"));
        // 解析 <settings /> 标签
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        // 加载自定义 VFS 实现类
        loadCustomVfs(settings);
        // 解析 <typeAliases /> 标签
        typeAliasesElement(root.evalNode("typeAliases"));
        // 解析 <plugins /> 标签
        pluginElement(root.evalNode("plugins"));
        // 解析 <objectFactory /> 标签
        objectFactoryElement(root.evalNode("objectFactory"));
        // 解析 <objectWrapperFactory /> 标签
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        // 解析 <reflectorFactory /> 标签
        reflectorFactoryElement(root.evalNode("reflectorFactory"));
        // 赋值 <settings /> 到 Configuration 属性
        settingsElement(settings);
        // read it after objectFactory and objectWrapperFactory issue #631
        // 解析 <environments /> 标签
        environmentsElement(root.evalNode("environments"));
        // 解析 <databaseIdProvider /> 标签
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        // 解析 <typeHandlers /> 标签
        typeHandlerElement(root.evalNode("typeHandlers"))
        // 解析 <mappers /> 标签
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```
### propertiesElement
`propertiesElement(XNode context)`方法，解析`<properties />`节点。大体逻辑如下：
* 解析`<properties />`标签，成`Properties`对象。
* 覆盖`configuration`中的`Properties`对象到上面的结果。
* 设置结果到`parser`和`configuration`中。

```java
// XMLConfigBuilder.java

private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
        // 读取子标签们，为 Properties 对象
        Properties defaults = context.getChildrenAsProperties();
        // 读取 resource 和 url 属性
        String resource = context.getStringAttribute("resource");
        String url = context.getStringAttribute("url");
        if (resource != null && url != null) { // resource 和 url 都存在的情况下，抛出 BuilderException 异常
            throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
        }
        // 读取本地 Properties 配置文件到 defaults 中。
        if (resource != null) {
            defaults.putAll(Resources.getResourceAsProperties(resource));
            // 读取远程 Properties 配置文件到 defaults 中。
        } else if (url != null) {
            defaults.putAll(Resources.getUrlAsProperties(url));
        }
        // 覆盖 configuration 中的 Properties 对象到 defaults 中。
        Properties vars = configuration.getVariables();
        if (vars != null) {
            defaults.putAll(vars);
        }
        // 设置 defaults 到 parser 和 configuration 中。
        parser.setVariables(defaults);
        configuration.setVariables(defaults);
    }
}
```
### settingsAsProperties
`settingsElement(Properties props)`方法，将`<setting />`标签解析为`Properties`对象。
```java
// XMLConfigBuilder.java

private Properties settingsAsProperties(XNode context) {
    // 将子标签，解析成 Properties 对象
    if (context == null) {
        return new Properties();
    }
    Properties props = context.getChildrenAsProperties();
    // Check that all settings are known to the configuration class
    // 校验每个属性，在 Configuration 中，有相应的 setting 方法，否则抛出 BuilderException 异常
    MetaClass metaConfig = MetaClass.forClass(Configuration.class, localReflectorFactory);
    for (Object key : props.keySet()) {
        if (!metaConfig.hasSetter(String.valueOf(key))) {
            throw new BuilderException("The setting " + key + " is not known.  Make sure you spelled it correctly (case sensitive).");
        }
    }
    return props;
}
```
### loadCustomVfs
`loadCustomVfs(Properties settings)`方法，加载自定义 VFS 实现类。
```java
// XMLConfigBuilder.java

private void loadCustomVfs(Properties props) throws ClassNotFoundException {
    // 获得 vfsImpl 属性
    String value = props.getProperty("vfsImpl");
    if (value != null) {
        // 使用 , 作为分隔符，拆成 VFS 类名的数组
        String[] clazzes = value.split(",");
        // 遍历 VFS 类名的数组
        for (String clazz : clazzes) {
            if (!clazz.isEmpty()) {
                // 获得 VFS 类
                @SuppressWarnings("unchecked")
                Class<? extends VFS> vfsImpl = (Class<? extends VFS>) Resources.classForName(clazz);
                // 设置到 Configuration 中
                configuration.setVfsImpl(vfsImpl);
            }
        }
    }
}
```
```java
// Configuration.java

/**
 * VFS 实现类
 */
protected Class<? extends VFS> vfsImpl;

public void setVfsImpl(Class<? extends VFS> vfsImpl) {
    if (vfsImpl != null) {
        // 设置 vfsImpl 属性
        this.vfsImpl = vfsImpl;
        // 添加到 VFS 中的自定义 VFS 类的集合
        VFS.addImplClass(this.vfsImpl);
    }
}
```
### typeAliasesElement
`typeAliasesElement(XNode parent)`方法，解析`<typeAliases />`标签，将配置类注册到`typeAliasRegistry`中。
```java
// XMLConfigBuilder.java

private void typeAliasesElement(XNode parent) {
    if (parent != null) {
        // 遍历子节点
        for (XNode child : parent.getChildren()) {
            // 指定为包的情况下，注册包下的每个类
            if ("package".equals(child.getName())) {
                String typeAliasPackage = child.getStringAttribute("name");
                configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
            // 指定为类的情况下，直接注册类和别名
            } else {
                String alias = child.getStringAttribute("alias");
                String type = child.getStringAttribute("type");
                try {
                    Class<?> clazz = Resources.classForName(type); // 获得类是否存在
                    // 注册到 typeAliasRegistry 中
                    if (alias == null) {
                        typeAliasRegistry.registerAlias(clazz);
                    } else {
                        typeAliasRegistry.registerAlias(alias, clazz);
                    }
                } catch (ClassNotFoundException e) { // 若类不存在，则抛出 BuilderException 异常
                    throw new BuilderException("Error registering typeAlias for '" + alias + "'. Cause: " + e, e);
                }
            }
        }
    }
}
```
### pluginElement
`pluginElement(XNode parent)`方法，解析`<plugins />`标签，添加到`Configuration#interceptorChain`中。
```java
// XMLConfigBuilder.java

private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
        // 遍历 <plugins /> 标签
        for (XNode child : parent.getChildren()) {
            String interceptor = child.getStringAttribute("interceptor");
            Properties properties = child.getChildrenAsProperties();
            // 创建 Interceptor 对象，并设置属性
            Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
            interceptorInstance.setProperties(properties);
            // <1> 添加到 configuration 中
            configuration.addInterceptor(interceptorInstance);
        }
    }
}
```
<1> 处，调用`Configuration#addInterceptor(Interceptor interceptor)`方法，添加到`configuration`中。代码如下：
```java
// Configuration.java

/**
 * 拦截器链
 */
protected final InterceptorChain interceptorChain = new InterceptorChain();

public void addInterceptor(Interceptor interceptor) {
    interceptorChain.addInterceptor(interceptor);
}
```
### objectFactoryElement
`objectFactoryElement(XNode parent)`方法，解析`<objectFactory />`节点。
```java
// XMLConfigBuilder.java

private void objectFactoryElement(XNode context) throws Exception {
    if (context != null) {
        // 获得 ObjectFactory 的实现类
        String type = context.getStringAttribute("type");
        // 获得 Properties 属性
        Properties properties = context.getChildrenAsProperties();
        // 创建 ObjectFactory 对象，并设置 Properties 属性
        ObjectFactory factory = (ObjectFactory) resolveClass(type).newInstance();
        factory.setProperties(properties);
        // <1> 设置 Configuration 的 objectFactory 属性
        configuration.setObjectFactory(factory);
    }
}
```
<1> 处，调用`Configuration#setObjectFactory(ObjectFactory objectFactory)`方法，设置`Configuration`的`objectFactory`属性。代码如下：
```java
// Configuration.java

/**
 * ObjectFactory 对象
 */
protected ObjectFactory objectFactory = new DefaultObjectFactory();

public void setObjectFactory(ObjectFactory objectFactory) {
    this.objectFactory = objectFactory;
}
```
### objectWrapperFactoryElement
`objectWrapperFactoryElement(XNode context)`方法，解析`<objectWrapperFactory />`节点。
```java
// XMLConfigBuilder.java

private void objectWrapperFactoryElement(XNode context) throws Exception {
    if (context != null) {
        // 获得 ObjectFactory 的实现类
        String type = context.getStringAttribute("type");
        // 创建 ObjectWrapperFactory 对象
        ObjectWrapperFactory factory = (ObjectWrapperFactory) resolveClass(type).newInstance();
        // 设置 Configuration 的 objectWrapperFactory 属性
        configuration.setObjectWrapperFactory(factory);
    }
}
```
<2> 处，调用`Configuration#setObjectWrapperFactory(ObjectWrapperFactory objectWrapperFactory)`方法，设置`Configuration`的`objectWrapperFactory`属性。代码如下：
```java
// Configuration.java

/**
 * ObjectWrapperFactory 对象
 */
protected ObjectWrapperFactory objectWrapperFactory = new DefaultObjectWrapperFactory();

public void setObjectWrapperFactory(ObjectWrapperFactory objectWrapperFactory) {
    this.objectWrapperFactory = objectWrapperFactory;
}
```
### reflectorFactoryElement
`reflectorFactoryElement(XNode parent)`方法，解析`<reflectorFactory />`节点。
```java
// XMLConfigBuilder.java

private void reflectorFactoryElement(XNode context) throws Exception {
    if (context != null) {
        // 获得 ReflectorFactory 的实现类
        String type = context.getStringAttribute("type");
        // 创建 ReflectorFactory 对象
        ReflectorFactory factory = (ReflectorFactory) resolveClass(type).newInstance();
        // 设置 Configuration 的 reflectorFactory 属性
        configuration.setReflectorFactory(factory);
    }
}
```
<2> 处，调用`Configuration#setReflectorFactory(ReflectorFactory reflectorFactory)`方法，设置`Configuration`的`reflectorFactory`属性。代码如下：
```java
// Configuration.java

/**
 * ReflectorFactory 对象
 */
protected ReflectorFactory reflectorFactory = new DefaultReflectorFactory();

public void setReflectorFactory(ReflectorFactory reflectorFactory) {
    this.reflectorFactory = reflectorFactory;
}
```
### settingsElement
`settingsElement(Properties props)`方法，赋值`<settings />`到`Configuration`属性。
```java
// XMLConfigBuilder.java

private void settingsElement(Properties props) throws Exception {
    configuration.setAutoMappingBehavior(AutoMappingBehavior.valueOf(props.getProperty("autoMappingBehavior", "PARTIAL")));
    configuration.setAutoMappingUnknownColumnBehavior(AutoMappingUnknownColumnBehavior.valueOf(props.getProperty("autoMappingUnknownColumnBehavior", "NONE")));
    configuration.setCacheEnabled(booleanValueOf(props.getProperty("cacheEnabled"), true));
    configuration.setProxyFactory((ProxyFactory) createInstance(props.getProperty("proxyFactory")));
    configuration.setLazyLoadingEnabled(booleanValueOf(props.getProperty("lazyLoadingEnabled"), false));
    configuration.setAggressiveLazyLoading(booleanValueOf(props.getProperty("aggressiveLazyLoading"), false));
    configuration.setMultipleResultSetsEnabled(booleanValueOf(props.getProperty("multipleResultSetsEnabled"), true));
    configuration.setUseColumnLabel(booleanValueOf(props.getProperty("useColumnLabel"), true));
    configuration.setUseGeneratedKeys(booleanValueOf(props.getProperty("useGeneratedKeys"), false));
    configuration.setDefaultExecutorType(ExecutorType.valueOf(props.getProperty("defaultExecutorType", "SIMPLE")));
    configuration.setDefaultStatementTimeout(integerValueOf(props.getProperty("defaultStatementTimeout"), null));
    configuration.setDefaultFetchSize(integerValueOf(props.getProperty("defaultFetchSize"), null));
    configuration.setMapUnderscoreToCamelCase(booleanValueOf(props.getProperty("mapUnderscoreToCamelCase"), false));
    configuration.setSafeRowBoundsEnabled(booleanValueOf(props.getProperty("safeRowBoundsEnabled"), false));
    configuration.setLocalCacheScope(LocalCacheScope.valueOf(props.getProperty("localCacheScope", "SESSION")));
    configuration.setJdbcTypeForNull(JdbcType.valueOf(props.getProperty("jdbcTypeForNull", "OTHER")));
    configuration.setLazyLoadTriggerMethods(stringSetValueOf(props.getProperty("lazyLoadTriggerMethods"), "equals,clone,hashCode,toString"));
    configuration.setSafeResultHandlerEnabled(booleanValueOf(props.getProperty("safeResultHandlerEnabled"), true));
    configuration.setDefaultScriptingLanguage(resolveClass(props.getProperty("defaultScriptingLanguage")));
    @SuppressWarnings("unchecked")
    Class<? extends TypeHandler> typeHandler = resolveClass(props.getProperty("defaultEnumTypeHandler"));
    configuration.setDefaultEnumTypeHandler(typeHandler);
    configuration.setCallSettersOnNulls(booleanValueOf(props.getProperty("callSettersOnNulls"), false));
    configuration.setUseActualParamName(booleanValueOf(props.getProperty("useActualParamName"), true));
    configuration.setReturnInstanceForEmptyRow(booleanValueOf(props.getProperty("returnInstanceForEmptyRow"), false));
    configuration.setLogPrefix(props.getProperty("logPrefix"));
    @SuppressWarnings("unchecked")
    Class<? extends Log> logImpl = resolveClass(props.getProperty("logImpl"));
    configuration.setLogImpl(logImpl);
    configuration.setConfigurationFactory(resolveClass(props.getProperty("configurationFactory")));
}
```
### environmentsElement
`environmentsElement(XNode context)`方法，解析`<environments />`标签。
```java
// XMLConfigBuilder.java

private void environmentsElement(XNode context) throws Exception {
    if (context != null) {
        // <1> environment 属性非空，从 default 属性获得
        if (environment == null) {
            environment = context.getStringAttribute("default");
        }
        // 遍历 XNode 节点
        for (XNode child : context.getChildren()) {
            // <2> 判断 environment 是否匹配
            String id = child.getStringAttribute("id");
            if (isSpecifiedEnvironment(id)) {
                // <3> 解析 `<transactionManager />` 标签，返回 TransactionFactory 对象
                TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
                // <4> 解析 `<dataSource />` 标签，返回 DataSourceFactory 对象
                DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
                DataSource dataSource = dsFactory.getDataSource();
                // <5> 创建 Environment.Builder 对象
                Environment.Builder environmentBuilder = new Environment.Builder(id)
                        .transactionFactory(txFactory)
                        .dataSource(dataSource);
                // <6> 构造 Environment 对象，并设置到 configuration 中
                configuration.setEnvironment(environmentBuilder.build());
            }
        }
    }
}
```
<1> 处，若`environment`属性非空，从`default`属性种获得`environment`属性。
<2> 处，遍历`XNode`节点，获得其`id`属性，后调用`isSpecifiedEnvironment(String id)`方法，判断`environment`和`id`是否匹配。代码如下：
```java
// XMLConfigBuilder.java

private boolean isSpecifiedEnvironment(String id) {
    if (environment == null) {
        throw new BuilderException("No environment specified.");
    } else if (id == null) {
        throw new BuilderException("Environment requires an id attribute.");
    } else if (environment.equals(id)) { // 相等
        return true;
    }
    return false;
}
```
<3> 处，调用`transactionManagerElement(XNode context)`方法，解析`<transactionManager />`标签，返回`TransactionFactory`对象。代码如下：
```java
// XMLConfigBuilder.java

private TransactionFactory transactionManagerElement(XNode context) throws Exception {
    if (context != null) {
        // 获得 TransactionFactory 的类
        String type = context.getStringAttribute("type");
        // 获得 Properties 属性
        Properties props = context.getChildrenAsProperties();
        // 创建 TransactionFactory 对象，并设置属性
        TransactionFactory factory = (TransactionFactory) resolveClass(type).newInstance();
        factory.setProperties(props);
        return factory;
    }
    throw new BuilderException("Environment declaration requires a TransactionFactory.");
}
```
<4> 处，调用`dataSourceElement(XNode context)`方法，解析`<dataSource />`标签，返回`DataSourceFactory`对象，而后返回`DataSource`对象。代码如下：
```java
// XMLConfigBuilder.java

private DataSourceFactory dataSourceElement(XNode context) throws Exception {
    if (context != null) {
        // 获得 DataSourceFactory 的类
        String type = context.getStringAttribute("type");
        // 获得 Properties 属性
        Properties props = context.getChildrenAsProperties();
        // 创建 DataSourceFactory 对象，并设置属性
        DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();
        factory.setProperties(props);
        return factory;
    }
    throw new BuilderException("Environment declaration requires a DataSourceFactory.");
}
```
<5> 处，创建`Environment.Builder`对象。

<6> 处，构造`Environment`对象，并设置到`configuration`中。代码如下：
```java
// Configuration.java

/**
 * DB Environment 对象
 */
protected Environment environment;

public void setEnvironment(Environment environment) {
    this.environment = environment;
}
```
#### Environment
`org.apache.ibatis.mapping.Environment`，DB 环境。代码如下：
```java
// Environment.java

public final class Environment {

    /**
     * 环境变好
     */
    private final String id;
    /**
     * TransactionFactory 对象
     */
    private final TransactionFactory transactionFactory;
    /**
     * DataSource 对象
     */
    private final DataSource dataSource;

    public Environment(String id, TransactionFactory transactionFactory, DataSource dataSource) {
        if (id == null) {
            throw new IllegalArgumentException("Parameter 'id' must not be null");
        }
        if (transactionFactory == null) {
            throw new IllegalArgumentException("Parameter 'transactionFactory' must not be null");
        }
        this.id = id;
        if (dataSource == null) {
            throw new IllegalArgumentException("Parameter 'dataSource' must not be null");
        }
        this.transactionFactory = transactionFactory;
        this.dataSource = dataSource;
    }

    /**
     * 构造器
     */
    public static class Builder {

        /**
         * 环境变好
         */
        private String id;
        /**
         * TransactionFactory 对象
         */
        private TransactionFactory transactionFactory;
        /**
         * DataSource 对象
         */
        private DataSource dataSource;

        public Builder(String id) {
            this.id = id;
        }

        public Builder transactionFactory(TransactionFactory transactionFactory) {
            this.transactionFactory = transactionFactory;
            return this;
        }

        public Builder dataSource(DataSource dataSource) {
            this.dataSource = dataSource;
            return this;
        }

        public String id() {
            return this.id;
        }

        public Environment build() {
            return new Environment(this.id, this.transactionFactory, this.dataSource);
        }

    }

    public String getId() {
        return this.id;
    }

    public TransactionFactory getTransactionFactory() {
        return this.transactionFactory;
    }

    public DataSource getDataSource() {
        return this.dataSource;
    }

}
```
### databaseIdProviderElement
`databaseIdProviderElement(XNode context)`方法，解析`<databaseIdProvider />`标签。
```java
// XMLConfigBuilder.java

private void databaseIdProviderElement(XNode context) throws Exception {
    DatabaseIdProvider databaseIdProvider = null;
    if (context != null) {
        // 获得 DatabaseIdProvider 的类
        String type = context.getStringAttribute("type");
        // awful patch to keep backward compatibility 保持兼容
        if ("VENDOR".equals(type)) {
            type = "DB_VENDOR";
        }
        // 获得 Properties 对象
        Properties properties = context.getChildrenAsProperties();
        // 创建 DatabaseIdProvider 对象，并设置对应的属性
        databaseIdProvider = (DatabaseIdProvider) resolveClass(type).newInstance();
        databaseIdProvider.setProperties(properties);
    }
    Environment environment = configuration.getEnvironment();
    if (environment != null && databaseIdProvider != null) {
        // <4> 获得对应的 databaseId 编号
        String databaseId = databaseIdProvider.getDatabaseId(environment.getDataSource());
        // <5> 设置到 configuration 中
        configuration.setDatabaseId(databaseId);
    }
}
```
<4> 处，调用`DatabaseIdProvider#getDatabaseId(DataSource dataSource)`方法，获得对应的`databaseId`标识。

<5> 处，设置到`configuration`中。代码如下：
```java
// Configuration.java

/**
 * 数据库标识
 */
protected String databaseId;

public void setDatabaseId(String databaseId) {
    this.databaseId = databaseId;
}
```
#### DatabaseIdProvider
`org.apache.ibatis.mapping.DatabaseIdProvider`，数据库标识提供器接口。
```java
public interface DatabaseIdProvider {

    /**
     * 设置属性
     *
     * @param p Properties 对象
     */
    void setProperties(Properties p);

    /**
     * 获得数据库标识
     *
     * @param dataSource 数据源
     * @return 数据库标识
     * @throws SQLException 当 DB 发生异常时
     */
    String getDatabaseId(DataSource dataSource) throws SQLException;
}
```
#### VendorDatabaseIdProvider
`org.apache.ibatis.mapping.VendorDatabaseIdProvider`，实现`DatabaseIdProvider`接口，供应商数据库标识提供器实现类。

① 构造方法
```java
// VendorDatabaseIdProvider.java

/**
 * Properties 对象
 */
private Properties properties;

@Override
public String getDatabaseId(DataSource dataSource) {
    if (dataSource == null) {
        throw new NullPointerException("dataSource cannot be null");
    }
    try {
        return getDatabaseName(dataSource);
    } catch (Exception e) {
        log.error("Could not get a databaseId from dataSource", e);
    }
    return null;
}

@Override
public void setProperties(Properties p) {
    this.properties = p;
}
```
② 获得数据库标识

`getDatabaseId(DataSource dataSource)`方法，代码如下：
```java
// VendorDatabaseIdProvider.java

@Override
public String getDatabaseId(DataSource dataSource) {
    if (dataSource == null) {
        throw new NullPointerException("dataSource cannot be null");
    }
    try {
        // 获得数据库标识
        return getDatabaseName(dataSource);
    } catch (Exception e) {
        log.error("Could not get a databaseId from dataSource", e);
    }
    return null;
}

private String getDatabaseName(DataSource dataSource) throws SQLException {
    // <1> 获得数据库产品名
    String productName = getDatabaseProductName(dataSource);
    if (this.properties != null) {
        for (Map.Entry<Object, Object> property : properties.entrySet()) {
            // 如果产品名包含 KEY ，则返回对应的  VALUE
            if (productName.contains((String) property.getKey())) {
                return (String) property.getValue();
            }
        }
        // no match, return null
        return null;
    }
    // <3> 不存在 properties ，则直接返回 productName
    return productName;
}
```
<1> 处，调用`getDatabaseProductName(DataSource dataSource)`方法，获得数据库产品名。代码如下：
```java
// VendorDatabaseIdProvider.java

private String getDatabaseProductName(DataSource dataSource) throws SQLException {
    try (Connection con = dataSource.getConnection()) {
        // 获得数据库连接
        DatabaseMetaData metaData = con.getMetaData();
        // 获得数据库产品名
        return metaData.getDatabaseProductName();
    }
}
```
通过从`Connection`获得数据库产品名。
<2> 处，如果`properties`非空，则从`properties`中匹配 KEY ？若成功，则返回`VALUE`，否则，返回`null`。
<3> 处，如果`properties`为空，则直接返回`productName`。
### typeHandlerElement
`typeHandlerElement(XNode parent)`方法，解析`<typeHandlers />`标签。
```java
// XMLConfigBuilder.java

private void typeHandlerElement(XNode parent) throws Exception {
    if (parent != null) {
        // 遍历子节点
        for (XNode child : parent.getChildren()) {
            // <1> 如果是 package 标签，则扫描该包
            if ("package".equals(child.getName())) {
                String typeHandlerPackage = child.getStringAttribute("name");
                typeHandlerRegistry.register(typeHandlerPackage);
            // <2> 如果是 typeHandler 标签，则注册该 typeHandler 信息
            } else {
                // 获得 javaType、jdbcType、handler
                String javaTypeName = child.getStringAttribute("javaType");
                String jdbcTypeName = child.getStringAttribute("jdbcType");
                String handlerTypeName = child.getStringAttribute("handler");
                Class<?> javaTypeClass = resolveClass(javaTypeName);
                JdbcType jdbcType = resolveJdbcType(jdbcTypeName);
                Class<?> typeHandlerClass = resolveClass(handlerTypeName); // 非空
                // 注册 typeHandler
                if (javaTypeClass != null) {
                    if (jdbcType == null) {
                        typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);
                    } else {
                        typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);
                    }
                } else {
                    typeHandlerRegistry.register(typeHandlerClass);
                }
            }
        }
    }
}
```
遍历子节点，分别处理 <1> 是`<package />`和 <2> 是`<typeHandler />`两种标签的情况。逻辑比较简单，最终都是注册到`typeHandlerRegistry`中。
### mapperElement
`mapperElement(XNode context)`方法，解析`<mappers />`标签。
```java
// XMLConfigBuilder.java

private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
        // <0> 遍历子节点
        for (XNode child : parent.getChildren()) {
            // <1> 如果是 package 标签，则扫描该包
            if ("package".equals(child.getName())) {
                // 获得包名
                String mapperPackage = child.getStringAttribute("name");
                // 添加到 configuration 中
                configuration.addMappers(mapperPackage);
            // 如果是 mapper 标签，
            } else {
                // 获得 resource、url、class 属性
                String resource = child.getStringAttribute("resource");
                String url = child.getStringAttribute("url");
                String mapperClass = child.getStringAttribute("class");
                // <2> 使用相对于类路径的资源引用
                if (resource != null && url == null && mapperClass == null) {
                    ErrorContext.instance().resource(resource);
                    // 获得 resource 的 InputStream 对象
                    InputStream inputStream = Resources.getResourceAsStream(resource);
                    // 创建 XMLMapperBuilder 对象
                    XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
                    // 执行解析
                    mapperParser.parse();
                // <3> 使用完全限定资源定位符（URL）
                } else if (resource == null && url != null && mapperClass == null) {
                    ErrorContext.instance().resource(url);
                    // 获得 url 的 InputStream 对象
                    InputStream inputStream = Resources.getUrlAsStream(url);
                    // 创建 XMLMapperBuilder 对象
                    XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
                    // 执行解析
                    mapperParser.parse();
                // <4> 使用映射器接口实现类的完全限定类名
                } else if (resource == null && url == null && mapperClass != null) {
                    // 获得 Mapper 接口
                    Class<?> mapperInterface = Resources.classForName(mapperClass);
                    // 添加到 configuration 中
                    configuration.addMapper(mapperInterface);
                } else {
                    throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
                }
            }
        }
    }
}
```
`<0>`处，遍历子节点，处理每一个节点。根据节点情况，会分成`<1>、<2>、<3>、<4>`种情况，并且第一个是处理`<package />`标签，后三个是处理`<mapper />`标签。

`<1>`处，如果是`<package />`标签，则获得`name`报名，并调用`Configuration#addMappers(String packageName)`方法，扫描该包下的所有`Mapper`接口。代码如下：
```java
// Configuration.java

/**
 * MapperRegistry 对象
 */
protected final MapperRegistry mapperRegistry = new MapperRegistry(this);

public void addMappers(String packageName) {
    // 扫描该包下所有的 Mapper 接口，并添加到 mapperRegistry 中
    mapperRegistry.addMappers(packageName);
}
```
`<4>`处，如果是`mapperClass`非空，则是使用映射器接口实现类的完全限定类名，则获得`Mapper`接口，并调用`Configuration#addMapper(Class<T> type)`方法，直接添加到`configuration`中。
```java
// Configuration.java

public <T> void addMapper(Class<T> type) {
    mapperRegistry.addMapper(type);
}
```
实际上，<1> 和 <4> 是相似情况，差别在于前者需要扫描，才能获取到所有的`Mapper`接口，而后者明确知道是哪个`Mapper`接口。
<2> 处，如果是`resource`非空，则是使用相对于类路径的资源引用，则需要创建`XMLMapperBuilder`对象，并调用`XMLMapperBuilder#parse()`方法，执行解析 Mapper XML 配置。执行之后，我们就能知道这个 Mapper XML 配置对应的`Mapper`接口。
<3> 处，如果是 url 非空，则是使用完全限定资源定位符（URL），情况和 <2> 是类似的。