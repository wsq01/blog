

加载`Statement`配置。而这个步骤的入口是`XMLStatementBuilder`。
```java
// XMLMapperBuilder.java

private void buildStatementFromContext(List<XNode> list) {
    if (configuration.getDatabaseId() != null) {
        buildStatementFromContext(list, configuration.getDatabaseId());
    }
    buildStatementFromContext(list, null);
    // 上面两块代码，可以简写成 buildStatementFromContext(list, configuration.getDatabaseId());
}

private void buildStatementFromContext(List<XNode> list, String requiredDatabaseId) {
    // <1> 遍历 <select /> <insert /> <update /> <delete /> 节点们
    for (XNode context : list) {
        // <1> 创建 XMLStatementBuilder 对象，执行解析
        final XMLStatementBuilder statementParser = new XMLStatementBuilder(configuration, builderAssistant, context, requiredDatabaseId);
        try {
            statementParser.parseStatementNode();
        } catch (IncompleteElementException e) {
            // <2> 解析失败，添加到 configuration 中
            configuration.addIncompleteStatement(statementParser);
        }
    }
}
```
# XMLStatementBuilder
`org.apache.ibatis.builder.xml.XMLStatementBuilder`，继承`BaseBuilder`抽象类，Statement XML 配置构建器，主要负责解析 Statement 配置，即`<select />、<insert />、<update />、<delete />`标签。

## 构造方法
```java
// XMLStatementBuilder.java

private final MapperBuilderAssistant builderAssistant;
/**
 * 当前 XML 节点，例如：<select />、<insert />、<update />、<delete /> 标签
 */
private final XNode context;
/**
 * 要求的 databaseId
 */
private final String requiredDatabaseId;

public XMLStatementBuilder(Configuration configuration, MapperBuilderAssistant builderAssistant, XNode context, String databaseId) {
    super(configuration);
    this.builderAssistant = builderAssistant;
    this.context = context;
    this.requiredDatabaseId = databaseId;
}
```
## parseStatementNode
#parseStatementNode() 方法，执行 Statement 解析。代码如下：
```java
// XMLStatementBuilder.java

public void parseStatementNode() {
    // <1> 获得 id 属性，编号。
    String id = context.getStringAttribute("id");
    // <2> 获得 databaseId ， 判断 databaseId 是否匹配
    String databaseId = context.getStringAttribute("databaseId");
    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
        return;
    }

    // <3> 获得各种属性
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultType = context.getStringAttribute("resultType");
    String lang = context.getStringAttribute("lang");

    // <4> 获得 lang 对应的 LanguageDriver 对象
    LanguageDriver langDriver = getLanguageDriver(lang);

    // <5> 获得 resultType 对应的类
    Class<?> resultTypeClass = resolveClass(resultType);
    // <6> 获得 resultSet 对应的枚举值
    String resultSetType = context.getStringAttribute("resultSetType");
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
    // <7> 获得 statementType 对应的枚举值
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));

    // <8> 获得 SQL 对应的 SqlCommandType 枚举值
    String nodeName = context.getNode().getNodeName();
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    // <9> 获得各种属性
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    // <10> 创建 XMLIncludeTransformer 对象，并替换 <include /> 标签相关的内容
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    // Parse selectKey after includes and remove them.
    // <11> 解析 <selectKey /> 标签
    processSelectKeyNodes(id, parameterTypeClass, langDriver);

    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
    // <12> 创建 SqlSource
    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
    // <13> 获得 KeyGenerator 对象
    String resultSets = context.getStringAttribute("resultSets");
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    KeyGenerator keyGenerator;
    // <13.1> 优先，从 configuration 中获得 KeyGenerator 对象。如果存在，意味着是 <selectKey /> 标签配置的
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
        keyGenerator = configuration.getKeyGenerator(keyStatementId);
    // <13.2> 其次，根据标签属性的情况，判断是否使用对应的 Jdbc3KeyGenerator 或者 NoKeyGenerator 对象
    } else {
        keyGenerator = context.getBooleanAttribute("useGeneratedKeys", // 优先，基于 useGeneratedKeys 属性判断
                configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType)) // 其次，基于全局的 useGeneratedKeys 配置 + 是否为插入语句类型
                ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

    // 创建 MappedStatement 对象
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
            fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
            resultSetTypeEnum, flushCache, useCache, resultOrdered,
            keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
```
<1> 处，获得 id 属性，编号。
<2> 处，获得 databaseId 属性，并调用 #databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) 方法，判断 databaseId 是否匹配。详细解析，见 「2.3 databaseIdMatchesCurrent」 。
<3> 处，获得各种属性。
<4> 处，调用 #getLanguageDriver(String lang) 方法，获得 lang 对应的 LanguageDriver 对象。详细解析，见 「2.4 getLanguageDriver」 。
<5> 处，获得 resultType 对应的类。
<6> 处，获得 resultSet 对应的枚举值。关于 org.apache.ibatis.mapping.ResultSetType 枚举类，点击查看。一般情况下，不会设置该值。它是基于 java.sql.ResultSet 结果集的几种模式，感兴趣的话，可以看看 《ResultSet 的 Type 属性》 。
<7> 处，获得 statementType 对应的枚举值。关于 org.apache.ibatis.mapping.StatementType 枚举类，点击查看。
<8> 处，获得 SQL 对应的 SqlCommandType 枚举值。
<9> 处，获得各种属性。
<10> 处，创建 XMLIncludeTransformer 对象，并调用 XMLIncludeTransformer#applyIncludes(Node source) 方法，替换 <include /> 标签相关的内容。详细解析，见 「3. XMLIncludeTransformer」 。
<11> 处，调用 #processSelectKeyNodes(String id, Class<?> parameterTypeClass, LanguageDriver langDriver) 方法，解析 <selectKey /> 标签。详细解析，见 「2.5 processSelectKeyNodes」 。
<12> 处，调用 LanguageDriver#createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) 方法，创建 SqlSource 对象。详细解析，见后续文章。
<13> 处，获得 KeyGenerator 对象。分成 <13.1> 和 <13.2> 两种情况。具体的，胖友耐心看下代码注释。
<14> 处，调用 MapperBuilderAssistant#addMappedStatement(...) 方法，创建 MappedStatement 对象。详细解析，见 「4.1 addMappedStatement」 中。

## databaseIdMatchesCurrent
#databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) 方法，判断 databaseId 是否匹配。代码如下：
```java
// XMLStatementBuilder.java

private boolean databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) {
    // 如果不匹配，则返回 false
    if (requiredDatabaseId != null) {
        return requiredDatabaseId.equals(databaseId);
    } else {
        // 如果未设置 requiredDatabaseId ，但是 databaseId 存在，说明还是不匹配，则返回 false
        // mmp ，写的好绕
        if (databaseId != null) {
            return false;
        }
        // skip this statement if there is a previous one with a not null databaseId
        // 判断是否已经存在
        id = builderAssistant.applyCurrentNamespace(id, false);
        if (this.configuration.hasStatement(id, false)) {
            MappedStatement previous = this.configuration.getMappedStatement(id, false); // issue #2
            // 若存在，则判断原有的 sqlFragment 是否 databaseId 为空。因为，当前 databaseId 为空，这样两者才能匹配。
            return previous.getDatabaseId() == null;
        }
    }
    return true;
}
```
从逻辑上，和我们在 XMLMapperBuilder 看到的同名方法 #databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) 方法是一致的。

## getLanguageDriver
#getLanguageDriver(String lang) 方法，获得 lang 对应的 LanguageDriver 对象。代码如下：
```java
// XMLStatementBuilder.java

private LanguageDriver getLanguageDriver(String lang) {
    // 解析 lang 对应的类
    Class<? extends LanguageDriver> langClass = null;
    if (lang != null) {
        langClass = resolveClass(lang);
    }
    // 获得 LanguageDriver 对象
    return builderAssistant.getLanguageDriver(langClass);
}
```
调用 MapperBuilderAssistant#getLanguageDriver(lass<? extends LanguageDriver> langClass) 方法，获得 LanguageDriver 对象。代码如下：
```java
// MapperBuilderAssistant.java

public LanguageDriver getLanguageDriver(Class<? extends LanguageDriver> langClass) {
    // 获得 langClass 类
    if (langClass != null) {
        configuration.getLanguageRegistry().register(langClass);
    } else { // 如果为空，则使用默认类
        langClass = configuration.getLanguageRegistry().getDefaultDriverClass();
    }
    // 获得 LanguageDriver 对象
    return configuration.getLanguageRegistry().getDriver(langClass);
}
```

## processSelectKeyNodes
#processSelectKeyNodes(String id, Class<?> parameterTypeClass, LanguageDriver langDriver) 方法，解析 <selectKey /> 标签。代码如下：
```
// XMLStatementBuilder.java

private void processSelectKeyNodes(String id, Class<?> parameterTypeClass, LanguageDriver langDriver) {
    // <1> 获得 <selectKey /> 节点们
    List<XNode> selectKeyNodes = context.evalNodes("selectKey");
    // <2> 执行解析 <selectKey /> 节点们
    if (configuration.getDatabaseId() != null) {
        parseSelectKeyNodes(id, selectKeyNodes, parameterTypeClass, langDriver, configuration.getDatabaseId());
    }
    parseSelectKeyNodes(id, selectKeyNodes, parameterTypeClass, langDriver, null);
    // <3> 移除 <selectKey /> 节点们
    removeSelectKeyNodes(selectKeyNodes);
}
```
<1> 处，获得 <selectKey /> 节点们。
<2> 处，调用 #parseSelectKeyNodes(...) 方法，执行解析 <selectKey /> 节点们。详细解析，见 「2.5.1 parseSelectKeyNodes」 。
<3> 处，调用 #removeSelectKeyNodes(List<XNode> selectKeyNodes) 方法，移除 <selectKey /> 节点们。代码如下：
```
// XMLStatementBuilder.java

private void removeSelectKeyNodes(List<XNode> selectKeyNodes) {
    for (XNode nodeToHandle : selectKeyNodes) {
        nodeToHandle.getParent().getNode().removeChild(nodeToHandle.getNode());
    }
}
```
### parseSelectKeyNodes
#parseSelectKeyNodes(String parentId, List<XNode> list, Class<?> parameterTypeClass, LanguageDriver langDriver, String skRequiredDatabaseId) 方法，执行解析 <selectKey /> 子节点们。代码如下：
```java
// XMLStatementBuilder.java

private void parseSelectKeyNodes(String parentId, List<XNode> list, Class<?> parameterTypeClass, LanguageDriver langDriver, String skRequiredDatabaseId) {
    // <1> 遍历 <selectKey /> 节点们
    for (XNode nodeToHandle : list) {
        // <2> 获得完整 id ，格式为 `${id}!selectKey`
        String id = parentId + SelectKeyGenerator.SELECT_KEY_SUFFIX;
        // <3> 获得 databaseId ， 判断 databaseId 是否匹配
        String databaseId = nodeToHandle.getStringAttribute("databaseId");
        if (databaseIdMatchesCurrent(id, databaseId, skRequiredDatabaseId)) {
            // <4> 执行解析单个 <selectKey /> 节点
            parseSelectKeyNode(id, nodeToHandle, parameterTypeClass, langDriver, databaseId);
        }
    }
}
```
<1> 处，遍历 <selectKey /> 节点们，逐个处理。
<2> 处，获得完整 id 编号，格式为 ${id}!selectKey 。这里很重要，最终解析的 <selectKey /> 节点，会创建成一个 MappedStatement 对象。而该对象的编号，就是 id 。
<3> 处，获得 databaseId ，并调用 #databaseIdMatchesCurrent(String id, String databaseId, String requiredDatabaseId) 方法，判断 databaseId 是否匹配。😈 通过此处，我们可以看到，即使有多个 <selectionKey /> 节点，但是最终只会有一个节点被解析，就是符合的 databaseId 对应的。因为不同的数据库实现不同，对于获取主键的方式也会不同。
<4> 处，调用 #parseSelectKeyNode(...) 方法，执行解析单个 <selectKey /> 节点。详细解析，见 「2.5.2 parseSelectKeyNode」 。


### parseSelectKeyNode
#parseSelectKeyNode(...) 方法，执行解析单个 <selectKey /> 节点。代码如下：
```java
// XMLStatementBuilder.java

private void parseSelectKeyNode(String id, XNode nodeToHandle, Class<?> parameterTypeClass, LanguageDriver langDriver, String databaseId) {
    // <1.1> 获得各种属性和对应的类
    String resultType = nodeToHandle.getStringAttribute("resultType");
    Class<?> resultTypeClass = resolveClass(resultType);
    StatementType statementType = StatementType.valueOf(nodeToHandle.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    String keyProperty = nodeToHandle.getStringAttribute("keyProperty");
    String keyColumn = nodeToHandle.getStringAttribute("keyColumn");
    boolean executeBefore = "BEFORE".equals(nodeToHandle.getStringAttribute("order", "AFTER"));

    // defaults
    // <1.2> 创建 MappedStatement 需要用到的默认值
    boolean useCache = false;
    boolean resultOrdered = false;
    KeyGenerator keyGenerator = NoKeyGenerator.INSTANCE;
    Integer fetchSize = null;
    Integer timeout = null;
    boolean flushCache = false;
    String parameterMap = null;
    String resultMap = null;
    ResultSetType resultSetTypeEnum = null;

    // <1.3> 创建 SqlSource 对象
    SqlSource sqlSource = langDriver.createSqlSource(configuration, nodeToHandle, parameterTypeClass);
    SqlCommandType sqlCommandType = SqlCommandType.SELECT;

    // <1.4> 创建 MappedStatement 对象
    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
            fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
            resultSetTypeEnum, flushCache, useCache, resultOrdered,
            keyGenerator, keyProperty, keyColumn, databaseId, langDriver, null);

    // <2.1> 获得 SelectKeyGenerator 的编号，格式为 `${namespace}.${id}`
    id = builderAssistant.applyCurrentNamespace(id, false);
    // <2.2> 获得 MappedStatement 对象
    MappedStatement keyStatement = configuration.getMappedStatement(id, false);
    // <2.3> 创建 SelectKeyGenerator 对象，并添加到 configuration 中
    configuration.addKeyGenerator(id, new SelectKeyGenerator(keyStatement, executeBefore));
}
```
<1.1> 处理，获得各种属性和对应的类。
<1.2> 处理，创建 MappedStatement 需要用到的默认值。
<1.3> 处理，调用 LanguageDriver#createSqlSource(Configuration configuration, XNode script, Class<?> parameterType) 方法，创建 SqlSource 对象。详细解析，见后续文章。
<1.4> 处理，调用 MapperBuilderAssistant#addMappedStatement(...) 方法，创建 MappedStatement 对象。
<2.1> 处理，获得 SelectKeyGenerator 的编号，格式为 ${namespace}.${id} 。
<2.2> 处理，获得 MappedStatement 对象。该对象，实际就是 <1.4> 处创建的 MappedStatement 对象。
<2.3> 处理，调用 Configuration#addKeyGenerator(String id, KeyGenerator keyGenerator) 方法，创建 SelectKeyGenerator 对象，并添加到 configuration 中。代码如下：
```java
// Configuration.java

/**
 * KeyGenerator 的映射
 *
 * KEY：在 {@link #mappedStatements} 的 KEY 的基础上，跟上 {@link SelectKeyGenerator#SELECT_KEY_SUFFIX}
 */
protected final Map<String, KeyGenerator> keyGenerators = new StrictMap<>("Key Generators collection");

public void addKeyGenerator(String id, KeyGenerator keyGenerator) {
    keyGenerators.put(id, keyGenerator);
}
```
# XMLIncludeTransformer
org.apache.ibatis.builder.xml.XMLIncludeTransformer ，XML <include /> 标签的转换器，负责将 SQL 中的 <include /> 标签转换成对应的 <sql /> 的内容。
## 构造方法
```
// XMLIncludeTransformer.java

private final Configuration configuration;
private final MapperBuilderAssistant builderAssistant;

public XMLIncludeTransformer(Configuration configuration, MapperBuilderAssistant builderAssistant) {
    this.configuration = configuration;
    this.builderAssistant = builderAssistant;
}
```
## applyIncludes
`applyIncludes(Node source)`方法，将`<include />`标签，替换成引用的`<sql />`。代码如下：
```java
// XMLIncludeTransformer.java

public void applyIncludes(Node source) {
    // <1> 创建 variablesContext ，并将 configurationVariables 添加到其中
    Properties variablesContext = new Properties();
    Properties configurationVariables = configuration.getVariables();
    if (configurationVariables != null) {
        variablesContext.putAll(configurationVariables);
    }
    // <2> 处理 <include />
    applyIncludes(source, variablesContext, false);
}
```
<1> 处，创建 variablesContext ，并将 configurationVariables 添加到其中。这里的目的是，避免 configurationVariables 被下面使用时候，可能被修改。实际上，从下面的实现上，不存在这个情况。
<2> 处，调用 #applyIncludes(Node source, final Properties variablesContext, boolean included) 方法，处理 <include /> 。
#applyIncludes(Node source, final Properties variablesContext, boolean included) 方法，使用递归的方式，将 <include /> 标签，替换成引用的 <sql /> 。代码如下：
```
// XMLIncludeTransformer.java

private void applyIncludes(Node source, final Properties variablesContext, boolean included) {
    // <1> 如果是 <include /> 标签
    if (source.getNodeName().equals("include")) {
        // <1.1> 获得 <sql /> 对应的节点
        Node toInclude = findSqlFragment(getStringAttribute(source, "refid"), variablesContext);
        // <1.2> 获得包含 <include /> 标签内的属性
        Properties toIncludeContext = getVariablesContext(source, variablesContext);
        // <1.3> 递归调用 #applyIncludes(...) 方法，继续替换。注意，此处是 <sql /> 对应的节点
        applyIncludes(toInclude, toIncludeContext, true);
        if (toInclude.getOwnerDocument() != source.getOwnerDocument()) { // 这个情况，艿艿暂时没调试出来
            toInclude = source.getOwnerDocument().importNode(toInclude, true);
        }
        // <1.4> 将 <include /> 节点替换成 <sql /> 节点
        source.getParentNode().replaceChild(toInclude, source); // 注意，这是一个奇葩的 API ，前者为 newNode ，后者为 oldNode
        // <1.4> 将 <sql /> 子节点添加到 <sql /> 节点前面
        while (toInclude.hasChildNodes()) {
            toInclude.getParentNode().insertBefore(toInclude.getFirstChild(), toInclude); // 这里有个点，一定要注意，卡了艿艿很久。当子节点添加到其它节点下面后，这个子节点会不见了，相当于是“移动操作”
        }
        // <1.4> 移除 <include /> 标签自身
        toInclude.getParentNode().removeChild(toInclude);
    // <2> 如果节点类型为 Node.ELEMENT_NODE
    } else if (source.getNodeType() == Node.ELEMENT_NODE) {
        // <2.1> 如果在处理 <include /> 标签中，则替换其上的属性，例如 <sql id="123" lang="${cpu}"> 的情况，lang 属性是可以被替换的
        if (included && !variablesContext.isEmpty()) {
            // replace variables in attribute values
            NamedNodeMap attributes = source.getAttributes();
            for (int i = 0; i < attributes.getLength(); i++) {
                Node attr = attributes.item(i);
                attr.setNodeValue(PropertyParser.parse(attr.getNodeValue(), variablesContext));
            }
        }
        // <2.2> 遍历子节点，递归调用 #applyIncludes(...) 方法，继续替换
        NodeList children = source.getChildNodes();
        for (int i = 0; i < children.getLength(); i++) {
            applyIncludes(children.item(i), variablesContext, included);
        }
    // <3> 如果在处理 <include /> 标签中，并且节点类型为 Node.TEXT_NODE ，并且变量非空
    // 则进行变量的替换，并修改原节点 source
    } else if (included && source.getNodeType() == Node.TEXT_NODE
            && !variablesContext.isEmpty()) {
        // replace variables in text node
        source.setNodeValue(PropertyParser.parse(source.getNodeValue(), variablesContext));
    }
}
```
这是个有自递归逻辑的方法，所以理解起来会有点绕，实际上还是蛮简单的。为了更好的解释，我们假设示例如下：
```
// mybatis-config.xml

<properties>
    <property name="cpu" value="16c" />
    <property name="target_sql" value="123" />
</properties>

// Mapper.xml

<sql id="123" lang="${cpu}">
    ${cpu}
    aoteman
    qqqq
</sql>

<select id="testForInclude">
    SELECT * FROM subject
    <include refid="${target_sql}" />
</select>
```
方法参数 included ，是否正在处理 <include /> 标签中。😈 一脸懵逼？不要方，继续往下看。
在上述示例的 <select /> 节点进入这个方法时，会首先进入 <2> 这块逻辑。
<2.1> 处，因为 不满足 included 条件，初始传入是 false ，所以跳过。
<2.2> 处，遍历子节点，递归调用 #applyIncludes(...) 方法，继续替换。如图所示：子节点
子节点
子节点 [0] 和 [2] ，执行该方法时，不满足 <1>、<2>、<3> 任一一种情况，所以可以忽略。虽然说，满足 <3> 的节点类型为 Node.TEXT_NODE ，但是 included 此时为 false ，所以不满足。
子节点 [1] ，执行该方法时，满足 <1> 的情况，所以走起。
在子节点 [1] ，即 <include /> 节点进入 <1> 这块逻辑：
<1.1> 处，调用 #findSqlFragment(String refid, Properties variables) 方法，获得 <sql /> 对应的节点，即上述示例看到的，<sql id="123" lang="${cpu}"> ... </> 。详细解析，见 「3.3 findSqlFragment」 。
<1.2> 处，调用 #getVariablesContext(Node node, Properties inheritedVariablesContext) 方法，获得包含 <include /> 标签内的属性 Properties 对象。详细解析，见 「3.4 getVariablesContext」 。
<1.3> 处，递归调用 #applyIncludes(...) 方法，继续替换。注意，此处是 <sql /> 对应的节点，并且 included 参数为 true 。详细的结果，见 😈😈😈 处。
<1.4> 处，将处理好的 <sql /> 节点，替换掉 <include /> 节点。逻辑有丢丢绕，胖友耐心看下注释，好好思考。
😈😈😈 在 <sql /> 节点，会进入 <2> 这块逻辑：
<2.1> 处，因为 included 为 true ，所以能满足这块逻辑，会进行执行。如 <sql id="123" lang="${cpu}"> 的情况，lang 属性是可以被替换的。
<2.2> 处，遍历子节点，递归调用 #applyIncludes(...) 方法，继续替换。如图所示：子节点
子节点
子节点 [0] ，执行该方法时，满足 <3> 的情况，所以可以使用变量 Properteis 对象，进行替换，并修改原节点。
其实，整理一下，逻辑也不会很绕。耐心耐心耐心。


## findSqlFragment
#findSqlFragment(String refid, Properties variables) 方法，获得对应的 <sql /> 节点。代码如下：
```java
// XMLIncludeTransformer.java

private Node findSqlFragment(String refid, Properties variables) {
    // 因为 refid 可能是动态变量，所以进行替换
    refid = PropertyParser.parse(refid, variables);
    // 获得完整的 refid ，格式为 "${namespace}.${refid}"
    refid = builderAssistant.applyCurrentNamespace(refid, true);
    try {
        // 获得对应的 <sql /> 节点
        XNode nodeToInclude = configuration.getSqlFragments().get(refid);
        // 获得 Node 节点，进行克隆
        return nodeToInclude.getNode().cloneNode(true);
    } catch (IllegalArgumentException e) {
        throw new IncompleteElementException("Could not find SQL statement to include with refid '" + refid + "'", e);
    }
}

private String getStringAttribute(Node node, String name) {
    return node.getAttributes().getNamedItem(name).getNodeValue();
}
```
## getVariablesContext
#getVariablesContext(Node node, Properties inheritedVariablesContext) 方法，获得包含 <include /> 标签内的属性 Properties 对象。代码如下：
```java
// XMLIncludeTransformer.java

private Properties getVariablesContext(Node node, Properties inheritedVariablesContext) {
    // 获得 <include /> 标签的属性集合
    Map<String, String> declaredProperties = null;
    NodeList children = node.getChildNodes();
    for (int i = 0; i < children.getLength(); i++) {
        Node n = children.item(i);
        if (n.getNodeType() == Node.ELEMENT_NODE) {
            String name = getStringAttribute(n, "name");
            // Replace variables inside
            String value = PropertyParser.parse(getStringAttribute(n, "value"), inheritedVariablesContext);
            if (declaredProperties == null) {
                declaredProperties = new HashMap<>();
            }
            if (declaredProperties.put(name, value) != null) { // 如果重复定义，抛出异常
                throw new BuilderException("Variable " + name + " defined twice in the same include definition");
            }
        }
    }
    // 如果 <include /> 标签内没有属性，直接使用 inheritedVariablesContext 即可
    if (declaredProperties == null) {
        return inheritedVariablesContext;
    // 如果 <include /> 标签内有属性，则创建新的 newProperties 集合，将 inheritedVariablesContext + declaredProperties 合并
    } else {
        Properties newProperties = new Properties();
        newProperties.putAll(inheritedVariablesContext);
        newProperties.putAll(declaredProperties);
        return newProperties;
    }
}
```
如下是 <include /> 标签内有属性的示例：
```
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>

<select id="selectUsers" resultType="map">
  select
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```
# MapperBuilderAssistant
4.1 addMappedStatement
#addMappedStatement(String id, SqlSource sqlSource, StatementType statementType, SqlCommandType sqlCommandType, Integer fetchSize, Integer timeout, String parameterMap, Class<?> parameterType, String resultMap, Class<?> resultType, ResultSetType resultSetType, boolean flushCache, boolean useCache, boolean resultOrdered, KeyGenerator keyGenerator, String keyProperty, String keyColumn, String databaseId, LanguageDriver lang, String resultSets) 方法，构建 MappedStatement 对象。代码如下：
```java
// MapperBuilderAssistant.java

public MappedStatement addMappedStatement(
        String id,
        SqlSource sqlSource,
        StatementType statementType,
        SqlCommandType sqlCommandType,
        Integer fetchSize,
        Integer timeout,
        String parameterMap,
        Class<?> parameterType,
        String resultMap,
        Class<?> resultType,
        ResultSetType resultSetType,
        boolean flushCache,
        boolean useCache,
        boolean resultOrdered,
        KeyGenerator keyGenerator,
        String keyProperty,
        String keyColumn,
        String databaseId,
        LanguageDriver lang,
        String resultSets) {

    // <1> 如果只想的 Cache 未解析，抛出 IncompleteElementException 异常
    if (unresolvedCacheRef) {
        throw new IncompleteElementException("Cache-ref not yet resolved");
    }

    // <2> 获得 id 编号，格式为 `${namespace}.${id}`
    id = applyCurrentNamespace(id, false);
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;

    // <3> 创建 MappedStatement.Builder 对象
    MappedStatement.Builder statementBuilder = new MappedStatement.Builder(configuration, id, sqlSource, sqlCommandType)
            .resource(resource)
            .fetchSize(fetchSize)
            .timeout(timeout)
            .statementType(statementType)
            .keyGenerator(keyGenerator)
            .keyProperty(keyProperty)
            .keyColumn(keyColumn)
            .databaseId(databaseId)
            .lang(lang)
            .resultOrdered(resultOrdered)
            .resultSets(resultSets)
            .resultMaps(getStatementResultMaps(resultMap, resultType, id)) // <3.1> 获得 ResultMap 集合
            .resultSetType(resultSetType)
            .flushCacheRequired(valueOrDefault(flushCache, !isSelect))
            .useCache(valueOrDefault(useCache, isSelect))
            .cache(currentCache);

    // <3.2> 获得 ParameterMap ，并设置到 MappedStatement.Builder 中
    ParameterMap statementParameterMap = getStatementParameterMap(parameterMap, parameterType, id);
    if (statementParameterMap != null) {
        statementBuilder.parameterMap(statementParameterMap);
    }

    // <4> 创建 MappedStatement 对象
    MappedStatement statement = statementBuilder.build();
    // <5> 添加到 configuration 中
    configuration.addMappedStatement(statement);
    return statement;
}
```
<1> 处，如果只想的 Cache 未解析，抛出 IncompleteElementException 异常。
<2> 处，获得 id 编号，格式为 ${namespace}.${id} 。
<3> 处，创建 MappedStatement.Builder 对象。详细解析，见 「4.1.3 MappedStatement」 。
<3.1> 处，调用 #getStatementResultMaps(...) 方法，获得 ResultMap 集合。详细解析，见 「4.1.3 getStatementResultMaps」 。
<3.2> 处，调用 #getStatementParameterMap(...) 方法，获得 ParameterMap ，并设置到 MappedStatement.Builder 中。详细解析，见 4.1.4 getStatementResultMaps」 。
<4> 处，创建 MappedStatement 对象。详细解析，见 「4.1.1 MappedStatement」 。
<5> 处，调用 Configuration#addMappedStatement(statement) 方法，添加到 configuration 中。代码如下：
```java
// Configuration.java

/**
 * MappedStatement 映射
 *
 * KEY：`${namespace}.${id}`
 */
protected final Map<String, MappedStatement> mappedStatements = new StrictMap<>("Mapped Statements collection");

public void addMappedStatement(MappedStatement ms) {
    mappedStatements.put(ms.getId(), ms);
}
```
### MappedStatement
`org.apache.ibatis.mapping.MappedStatement`，映射的语句，每个`<select />、<insert />、<update />、<delete />`对应一个`MappedStatement`对象。

另外，比较特殊的是，`<selectKey />`解析后，也会对应一个`MappedStatement`对象。

在另外，MappedStatement 有一个非常重要的方法`getBoundSql(Object parameterObject)`方法，获得 BoundSql 对象。代码如下：
```java
// MappedStatement.java

public BoundSql getBoundSql(Object parameterObject) {
    // 获得 BoundSql 对象
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);
    // 忽略，因为 <parameterMap /> 已经废弃，参见 http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html 文档
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings == null || parameterMappings.isEmpty()) {
        boundSql = new BoundSql(configuration, boundSql.getSql(), parameterMap.getParameterMappings(), parameterObject);
    }

    // check for nested result maps in parameter mappings (issue #30)
    // 判断传入的参数中，是否有内嵌的结果 ResultMap 。如果有，则修改 hasNestedResultMaps 为 true
    // 存储过程相关，暂时无视
    for (ParameterMapping pm : boundSql.getParameterMappings()) {
        String rmId = pm.getResultMapId();
        if (rmId != null) {
            ResultMap rm = configuration.getResultMap(rmId);
            if (rm != null) {
                hasNestedResultMaps |= rm.hasNestedResultMaps();
            }
        }
    }

    return boundSql;
}
```
### getStatementResultMaps
`getStatementResultMaps(...)`方法，获得`ResultMap`集合。
```java
// MapperBuilderAssistant.java

private List<ResultMap> getStatementResultMaps(
        String resultMap,
        Class<?> resultType,
        String statementId) {
    // 获得 resultMap 的编号
    resultMap = applyCurrentNamespace(resultMap, true);

    // 创建 ResultMap 集合
    List<ResultMap> resultMaps = new ArrayList<>();
    // 如果 resultMap 非空，则获得 resultMap 对应的 ResultMap 对象(们）
    if (resultMap != null) {
        String[] resultMapNames = resultMap.split(",");
        for (String resultMapName : resultMapNames) {
            try {
                resultMaps.add(configuration.getResultMap(resultMapName.trim())); // 从 configuration 中获得
            } catch (IllegalArgumentException e) {
                throw new IncompleteElementException("Could not find result map " + resultMapName, e);
            }
        }
    // 如果 resultType 非空，则创建 ResultMap 对象
    } else if (resultType != null) {
        ResultMap inlineResultMap = new ResultMap.Builder(
                configuration,
                statementId + "-Inline",
                resultType,
                new ArrayList<>(),
                null).build();
        resultMaps.add(inlineResultMap);
    }
    return resultMaps;
}
```
### getStatementResultMaps
`getStatementParameterMap(...)`方法，获得`ParameterMap`对象。
```java
// MapperBuilderAssistant.java

private ParameterMap getStatementParameterMap(
        String parameterMapName,
        Class<?> parameterTypeClass,
        String statementId) {
    // 获得 ParameterMap 的编号，格式为 `${namespace}.${parameterMapName}`
    parameterMapName = applyCurrentNamespace(parameterMapName, true);
    ParameterMap parameterMap = null;
    // 如果 parameterMapName 非空，则获得 parameterMapName 对应的 ParameterMap 对象
    if (parameterMapName != null) {
        try {
            parameterMap = configuration.getParameterMap(parameterMapName);
        } catch (IllegalArgumentException e) {
            throw new IncompleteElementException("Could not find parameter map " + parameterMapName, e);
        }
    // 如果 parameterTypeClass 非空，则创建 ParameterMap 对象
    } else if (parameterTypeClass != null) {
        List<ParameterMapping> parameterMappings = new ArrayList<>();
        parameterMap = new ParameterMap.Builder(
                configuration,
                statementId + "-Inline",
                parameterTypeClass,
                parameterMappings).build();
    }
    return parameterMap;
}
```
