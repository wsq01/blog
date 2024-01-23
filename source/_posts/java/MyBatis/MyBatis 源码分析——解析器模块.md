


MyBatis 的解析器模块，对应`parsing`包。

{% asset_img 1.png %}

解析器模块，主要提供了两个功能:
* 对`XPath`进行封装，为 MyBatis 初始化时解析`mybatis-config.xml`配置文件以及映射配置文件提供支持。
* 另一个功能，是为处理动态 SQL 语句中的占位符提供支持。

# XPathParser
`org.apache.ibatis.parsing.XPathParser`，基于 Java XPath 解析器，用于解析 MyBatis `mybatis-config.xml`和`**Mapper.xml`等 XML 配置文件。属性如下：
```java
// XPathParser.java

/**
 * XML Document 对象
 */
private final Document document;
/**
 * 是否校验
 */
private boolean validation;
/**
 * XML 实体解析器
 */
private EntityResolver entityResolver;
/**
 * 变量 Properties 对象
 */
private Properties variables;
/**
 * Java XPath 对象
 */
private XPath xpath;
```
* `document`属性，XML 被解析后，生成的`org.w3c.dom.Document`对象。
* `validation`属性，是否校验 XML。一般情况下，值为`true`。
* `entityResolver`属性，`org.xml.sax.EntityResolver`对象，XML 实体解析器。默认情况下，对 XML 进行校验时，会基于 XML 文档开始位置指定的 DTD 文件或 XSD 文件。例如说，解析`mybatis-config.xml`配置文件时，会加载`http://mybatis.org/dtd/mybatis-3-config.dtd`这个 DTD 文件。但是，如果每个应用启动都从网络加载该 DTD 文件，势必在弱网络下体验非常下，甚至说应用部署在无网络的环境下，还会导致下载不下来，那么就会出现 XML 校验失败的情况。所以，在实际场景下，MyBatis 自定义了`EntityResolver`的实现，达到使用本地 DTD 文件，从而避免下载网络 DTD 文件的效果。另外，Spring 也自定义了`EntityResolver`的实现 。
* `xpath`属性，`javax.xml.xpath.XPath`对象，用于查询 XML 中的节点和元素。
* `variables`属性，变量`Properties`对象，用来替换需要动态配置的属性值。例如：
```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```
`variables`的来源，即可以在常用的 Java `Properties`文件中配置，也可以使用 MyBatis `<property />`标签中配置。例如：
```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```
这里配置的`username`和`password`属性，就可以替换上面的`${username}`和`${password}`这两个动态属性。

具体如何实现的，看下面的`PropertyParser#parse(String string, Properties variables)`方法。
## 构造方法
`XPathParser`的构造方法有 16 个之多，当然基本都非常相似，我们来挑选其中一个。代码如下：
```java
// XPathParser.java

/**
 * 构造 XPathParser 对象
 *
 * @param xml XML 文件地址
 * @param validation 是否校验 XML
 * @param variables 变量 Properties 对象
 * @param entityResolver XML 实体解析器
 */
public XPathParser(String xml, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    this.document = createDocument(new InputSource(new StringReader(xml)));
}
```
调用`commonConstructor(boolean validation, Properties variables, EntityResolver entityResolver)`方法，公用的构造方法逻辑。代码如下：
```java
// XPathParser.java

private void commonConstructor(boolean validation, Properties variables, EntityResolver entityResolver) {
    this.validation = validation;
    this.entityResolver = entityResolver;
    this.variables = variables;
    // 创建 XPathFactory 对象
    XPathFactory factory = XPathFactory.newInstance();
    this.xpath = factory.newXPath();
}
```
调用`createDocument(InputSource inputSource)`方法，将 XML 文件解析成`Document`对象。代码如下：
```java
// XPathParser.java

/**
 * 创建 Document 对象
 *
 * @param inputSource XML 的 InputSource 对象
 * @return Document 对象
 */
private Document createDocument(InputSource inputSource) {
    // important: this must only be called AFTER common constructor
    try {
        // 1> 创建 DocumentBuilderFactory 对象
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        factory.setValidating(validation); // 设置是否验证 XML

        factory.setNamespaceAware(false);
        factory.setIgnoringComments(true);
        factory.setIgnoringElementContentWhitespace(false);
        factory.setCoalescing(false);
        factory.setExpandEntityReferences(true);

        // 2> 创建 DocumentBuilder 对象
        DocumentBuilder builder = factory.newDocumentBuilder();
        builder.setEntityResolver(entityResolver); // 设置实体解析器
        builder.setErrorHandler(new ErrorHandler() { // 实现都空的

            @Override
            public void error(SAXParseException exception) throws SAXException {
                throw exception;
            }

            @Override
            public void fatalError(SAXParseException exception) throws SAXException {
                throw exception;
            }

            @Override
            public void warning(SAXParseException exception) throws SAXException {
            }

        });
        // 3> 解析 XML 文件
        return builder.parse(inputSource);
    } catch (Exception e) {
        throw new BuilderException("Error creating document instance.  Cause: " + e, e);
    }
}
```
## eval 方法族
`XPathParser`提供了一系列的`eval*`方法，用于获得`Boolean、Short、Integer、Long、Float、Double、String、Node`类型的元素或节点的“值”。当然，虽然方法很多，但是都是基于`evaluate(String expression, Object root, QName returnType)`方法，代码如下：
```java
// XPathParser.java

/**
 * 获得指定元素或节点的值
 *
 * @param expression 表达式
 * @param root 指定节点
 * @param returnType 返回类型
 * @return 值
 */
private Object evaluate(String expression, Object root, QName returnType) {
    try {
        return xpath.evaluate(expression, root, returnType);
    } catch (Exception e) {
        throw new BuilderException("Error evaluating XPath.  Cause: " + e, e);
    }
}
```
调用`xpath`的`evaluate(String expression, Object root, QName returnType)`方法，获得指定元素或节点的值。
### eval 元素
`eval`元素的方法，用于获得`Boolean、Short、Integer、Long、Float、Double、String`类型的元素的值。我们以`evalString(Object root, String expression)`方法为例子，代码如下：
```java
// XPathParser.java

public String evalString(Object root, String expression) {
    // <1> 获得值
    String result = (String) evaluate(expression, root, XPathConstants.STRING);
    // <2> 基于 variables 替换动态值，如果 result 为动态值
    result = PropertyParser.parse(result, variables);
    return result;
}
```
<1> 处，调用`evaluate(String expression, Object root, QName returnType)`方法，获得值。其中，`returnType`方法传入的是`XPathConstants.STRING`，表示返回的值是`String`类型。
<2> 处，调用`PropertyParser#parse(String string, Properties variables)`方法，基于`variables`替换动态值，如果`result`为动态值。这就是 MyBatis 如何替换掉 XML 中的动态值实现的方式。
### eval 节点
`eval`元素的方法，用于获得`Node`类型的节点的值。代码如下：
```java
// XPathParser.java

public List<XNode> evalNodes(String expression) { // Node 数组
    return evalNodes(document, expression);
}

public List<XNode> evalNodes(Object root, String expression) { // Node 数组
    // <1> 获得 Node 数组
    NodeList nodes = (NodeList) evaluate(expression, root, XPathConstants.NODESET);
    // <2> 封装成 XNode 数组
    List<XNode> xnodes = new ArrayList<>();
    for (int i = 0; i < nodes.getLength(); i++) {
        xnodes.add(new XNode(this, nodes.item(i), variables));
    }
    return xnodes;
}

public XNode evalNode(String expression) { // Node 对象
    return evalNode(document, expression);
}

public XNode evalNode(Object root, String expression) { // Node 对象
    // <1> 获得 Node 对象
    Node node = (Node) evaluate(expression, root, XPathConstants.NODE);
    if (node == null) {
        return null;
    }
    // <2> 封装成 XNode 对象
    return new XNode(this, node, variables);
}
```
<1> 处，返回结果有`Node`对象和数组两种情况，根据方法参数`expression`需要获取的节点不同。
<2> 处， 最终结果会将 Node 封装成`org.apache.ibatis.parsing.XNode`对象，主要为了动态值的替换。例如：
```java
// XNode.java

public String evalString(String expression) {
    return xpathParser.evalString(node, expression);
}
```
# XMLMapperEntityResolver
`org.apache.ibatis.builder.xml.XMLMapperEntityResolver`，实现`EntityResolver`接口，MyBatis 自定义`EntityResolver`实现类，用于加载本地的`mybatis-3-config.dtd`和`mybatis-3-mapper.dtd`这两个 DTD 文件。代码如下：
```java
// XMLMapperEntityResolver.java

public class XMLMapperEntityResolver implements EntityResolver {

    private static final String IBATIS_CONFIG_SYSTEM = "ibatis-3-config.dtd";
    private static final String IBATIS_MAPPER_SYSTEM = "ibatis-3-mapper.dtd";
    private static final String MYBATIS_CONFIG_SYSTEM = "mybatis-3-config.dtd";
    private static final String MYBATIS_MAPPER_SYSTEM = "mybatis-3-mapper.dtd";

    /**
     * 本地 mybatis-config.dtd 文件
     */
    private static final String MYBATIS_CONFIG_DTD = "org/apache/ibatis/builder/xml/mybatis-3-config.dtd";
    /**
     * 本地 mybatis-mapper.dtd 文件
     */
    private static final String MYBATIS_MAPPER_DTD = "org/apache/ibatis/builder/xml/mybatis-3-mapper.dtd";

    /**
     * Converts a public DTD into a local one
     *
     * @param publicId The public id that is what comes after "PUBLIC"
     * @param systemId The system id that is what comes after the public id.
     * @return The InputSource for the DTD
     *
     * @throws org.xml.sax.SAXException If anything goes wrong
     */
    @Override
    public InputSource resolveEntity(String publicId, String systemId) throws SAXException {
        try {
            if (systemId != null) {
                String lowerCaseSystemId = systemId.toLowerCase(Locale.ENGLISH);
                // 本地 mybatis-config.dtd 文件
                if (lowerCaseSystemId.contains(MYBATIS_CONFIG_SYSTEM) || lowerCaseSystemId.contains(IBATIS_CONFIG_SYSTEM)) {
                    return getInputSource(MYBATIS_CONFIG_DTD, publicId, systemId);
                // 本地 mybatis-mapper.dtd 文件
                } else if (lowerCaseSystemId.contains(MYBATIS_MAPPER_SYSTEM) || lowerCaseSystemId.contains(IBATIS_MAPPER_SYSTEM)) {
                    return getInputSource(MYBATIS_MAPPER_DTD, publicId, systemId);
                }
            }
            return null;
        } catch (Exception e) {
            throw new SAXException(e.toString());
        }
    }

    private InputSource getInputSource(String path, String publicId, String systemId) {
        InputSource source = null;
        if (path != null) {
            try {
                // 创建 InputSource 对象
                InputStream in = Resources.getResourceAsStream(path);
                source = new InputSource(in);
                // 设置  publicId、systemId 属性
                source.setPublicId(publicId);
                source.setSystemId(systemId);
            } catch (IOException e) {
                // ignore, null is ok
            }
        }
        return source;
    }

}
```
# GenericTokenParser
`org.apache.ibatis.parsing.GenericTokenParser`，通用的`Token`解析器。代码如下：
```java
// GenericTokenParser.java

public class GenericTokenParser {

    /**
     * 开始的 Token 字符串
     */
    private final String openToken;
    /**
     * 结束的 Token 字符串
     */
    private final String closeToken;
    private final TokenHandler handler;

    public GenericTokenParser(String openToken, String closeToken, TokenHandler handler) {
        this.openToken = openToken;
        this.closeToken = closeToken;
        this.handler = handler;
    }

    public String parse(String text) {
        if (text == null || text.isEmpty()) {
            return "";
        }
        // search open token
        // 寻找开始的 openToken 的位置
        int start = text.indexOf(openToken, 0);
        if (start == -1) { // 找不到，直接返回
            return text;
        }
        char[] src = text.toCharArray();
        int offset = 0; // 起始查找位置
        // 结果
        final StringBuilder builder = new StringBuilder();
        StringBuilder expression = null; // 匹配到 openToken 和 closeToken 之间的表达式
        // 循环匹配
        while (start > -1) {
            // 转义字符
            if (start > 0 && src[start - 1] == '\\') {
                // this open token is escaped. remove the backslash and continue.
                // 因为 openToken 前面一个位置是 \ 转义字符，所以忽略 \
                // 添加 [offset, start - offset - 1] 和 openToken 的内容，添加到 builder 中
                builder.append(src, offset, start - offset - 1).append(openToken);
                // 修改 offset
                offset = start + openToken.length();
            // 非转义字符
            } else {
                // found open token. let's search close token.
                // 创建/重置 expression 对象
                if (expression == null) {
                    expression = new StringBuilder();
                } else {
                    expression.setLength(0);
                }
                // 添加 offset 和 openToken 之间的内容，添加到 builder 中
                builder.append(src, offset, start - offset);
                // 修改 offset
                offset = start + openToken.length();
                // 寻找结束的 closeToken 的位置
                int end = text.indexOf(closeToken, offset);
                while (end > -1) {
                    // 转义
                    if (end > offset && src[end - 1] == '\\') {
                        // this close token is escaped. remove the backslash and continue.
                        // 因为 endToken 前面一个位置是 \ 转义字符，所以忽略 \
                        // 添加 [offset, end - offset - 1] 和 endToken 的内容，添加到 builder 中
                        expression.append(src, offset, end - offset - 1).append(closeToken);
                        // 修改 offset
                        offset = end + closeToken.length();
                        // 继续，寻找结束的 closeToken 的位置
                        end = text.indexOf(closeToken, offset);
                    // 非转义
                    } else {
                        // 添加 [offset, end - offset] 的内容，添加到 builder 中
                        expression.append(src, offset, end - offset);
                        break;
                    }
                }
                // 拼接内容
                if (end == -1) {
                    // close token was not found.
                    // closeToken 未找到，直接拼接
                    builder.append(src, start, src.length - start);
                    // 修改 offset
                    offset = src.length;
                } else {
                    // <x> closeToken 找到，将 expression 提交给 handler 处理 ，并将处理结果添加到 builder 中
                    builder.append(handler.handleToken(expression.toString()));
                    // 修改 offset
                    offset = end + closeToken.length();
                }
            }
            // 继续，寻找开始的 openToken 的位置
            start = text.indexOf(openToken, offset);
        }
        // 拼接剩余的部分
        if (offset < src.length) {
            builder.append(src, offset, src.length - offset);
        }
        return builder.toString();
    }

}
```
就一个`parse(String text)`方法，循环( 因为可能不只一个 )，解析以`openToken`开始，以`closeToken`结束的`Token`，并提交给`handler`进行处理，即`<x>`处。

这也是为什么`GenericTokenParser`叫做通用的原因，而`TokenHandler`处理特定的逻辑。
# PropertyParser
`org.apache.ibatis.parsing.PropertyParser`，动态属性解析器。代码如下：
```java
// PropertyParser.java

public class PropertyParser {

    // ... 省略部分无关的

    private PropertyParser() { // <1>
        // Prevent Instantiation
    }

    public static String parse(String string, Properties variables) { // <2>
        // <2.1> 创建 VariableTokenHandler 对象
        VariableTokenHandler handler = new VariableTokenHandler(variables);
        // <2.2> 创建 GenericTokenParser 对象
        GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
        // <2.3> 执行解析
        return parser.parse(string);
    }
    
}
```
* <1>，构造方法，修饰符为`private`，禁止构造`PropertyParser`对象，因为它是一个静态方法的工具类。
* <2>，基于 variables 变量，替换`string`字符串中的动态属性，并返回结果。
* <2.1>，创建`VariableTokenHandler`对象。
* <2.2>，创建`GenericTokenParser`对象。
我们可以看到，`openToken = { ，closeToken = }`，这不就是上面看到的`${username}`和`{password}`的么。
同时，我们也可以看到，`handler`类型为`VariableTokenHandler`，也就是说，通过它实现自定义的处理逻辑。
* <2.3> ，调用`GenericTokenParser#parse(String text)`方法，执行解析。

# TokenHandler
`org.apache.ibatis.parsing.TokenHandler`，`Token`处理器接口。代码如下：
```java
// TokenHandler.java

public interface TokenHandler {

    /**
     * 处理 Token
     *
     * @param content Token 字符串
     * @return 处理后的结果
     */
    String handleToken(String content);

}
```
`handleToken(String content)`方法，处理`Token`，在`GenericTokenParser`中，我们已经看到它的调用了。

`TokenHandler`有四个子类实现，如下图所示：

{% asset_img 2.png %}

本文暂时只解析`VariableTokenHandler`类，因为只有它在`parsing`包中，和解析器模块相关。
## VariableTokenHandler
`VariableTokenHandler`，是`PropertyParser`的内部静态类，变量`Token`处理器。
### 构造方法
```java
// PropertyParser.java

private static final String KEY_PREFIX = "org.apache.ibatis.parsing.PropertyParser.";
/**
 * The special property key that indicate whether enable a default value on placeholder.
 * <p>
 *   The default value is {@code false} (indicate disable a default value on placeholder)
 *   If you specify the {@code true}, you can specify key and default value on placeholder (e.g. {@code ${db.username:postgres}}).
 * </p>
 * @since 3.4.2
 */
public static final String KEY_ENABLE_DEFAULT_VALUE = KEY_PREFIX + "enable-default-value";

/**
 * The special property key that specify a separator for key and default value on placeholder.
 * <p>
 *   The default separator is {@code ":"}.
 * </p>
 * @since 3.4.2
 */
public static final String KEY_DEFAULT_VALUE_SEPARATOR = KEY_PREFIX + "default-value-separator";

private static final String ENABLE_DEFAULT_VALUE = "false";
private static final String DEFAULT_VALUE_SEPARATOR = ":";

// VariableTokenHandler 类里

/**
 * 变量 Properties 对象
 */
private final Properties variables;
/**
 * 是否开启默认值功能。默认为 {@link #ENABLE_DEFAULT_VALUE}
 */
private final boolean enableDefaultValue;
/**
 * 默认值的分隔符。默认为 {@link #KEY_DEFAULT_VALUE_SEPARATOR} ，即 ":" 。
 */
private final String defaultValueSeparator;

private VariableTokenHandler(Properties variables) {
    this.variables = variables;
    this.enableDefaultValue = Boolean.parseBoolean(getPropertyValue(KEY_ENABLE_DEFAULT_VALUE, ENABLE_DEFAULT_VALUE));
    this.defaultValueSeparator = getPropertyValue(KEY_DEFAULT_VALUE_SEPARATOR, DEFAULT_VALUE_SEPARATOR);
}

private String getPropertyValue(String key, String defaultValue) {
    return (variables == null) ? defaultValue : variables.getProperty(key, defaultValue);
}
```
`variables`属性，变量`Properties`对象。

`enableDefaultValue`属性，是否开启默认值功能。默认为`ENABLE_DEFAULT_VALUE`，即不开启。想要开启，可以配置如下：
```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- Enable this feature -->
</properties>
```
`defaultValueSeparator`属性，默认值的分隔符。默认为`KEY_DEFAULT_VALUE_SEPARATOR`，即`:`。想要修改，可以配置如下：
```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- Change default value of separator -->
</properties>
```
分隔符被修改成了`?:`。
### handleToken
```java
// VariableTokenHandler 类里

@Override
public String handleToken(String content) {
    if (variables != null) {
        String key = content;
        // 开启默认值功能
        if (enableDefaultValue) {
            // 查找默认值
            final int separatorIndex = content.indexOf(defaultValueSeparator);
            String defaultValue = null;
            if (separatorIndex >= 0) {
                key = content.substring(0, separatorIndex);
                defaultValue = content.substring(separatorIndex + defaultValueSeparator.length());
            }
            // 有默认值，优先替换，不存在则返回默认值
            if (defaultValue != null) {
                return variables.getProperty(key, defaultValue);
            }
        }
        // 未开启默认值功能，直接替换
        if (variables.containsKey(key)) {
            return variables.getProperty(key);
        }
    }
    // 无 variables ，直接返回
    return "${" + content + "}";
}
```