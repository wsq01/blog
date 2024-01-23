


# 概述
`org.springframework.web.servlet.HandlerAdapter`，处理器适配器接口。
```java
// HandlerAdapter.java

public interface HandlerAdapter {

    /**
     * 是否支持该处理器
     */
    boolean supports(Object handler);

    /**
     * 执行处理器，返回 ModelAndView 结果
     */
    @Nullable
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

    /**
     * 返回请求的最新更新时间。
     *
     * 如果不支持该操作，则返回 -1 即可
     */
    long getLastModified(HttpServletRequest request, Object handler);

}
```
因为，处理器`handler`的类型是`Object`类型，需要有一个调用者来实现`handler`是怎么被使用，怎么被执行。而`HandlerAdapter`的用途就在于此。
# HandlerAdapter
HandlerAdapter 的子类比较多，整体类图如下：

{% asset_img 1.png %}

# SimpleControllerHandlerAdapter
`org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter`，实现`HandlerAdapter`接口，基于`org.springframework.web.servlet.mvc.Controller`的`HandlerAdapter`实现类。
```java
// SimpleControllerHandlerAdapter.java

public class SimpleControllerHandlerAdapter implements HandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        // <1> 判断是 Controller 类型
        return (handler instanceof Controller);
    }

    @Override
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        // <2> Controller 类型的调用
        return ((Controller) handler).handleRequest(request, response);
    }

    @Override
    public long getLastModified(HttpServletRequest request, Object handler) {
        // 处理器实现了 LastModified 接口的情况下
        if (handler instanceof LastModified) {
            return ((LastModified) handler).getLastModified(request);
        }
        return -1L;
    }

}
```
`<1>`处，判断处理器`handler`是`Controller`类型。注意，不是`@Controller`注解。

`<2>`处，调用`Controller#handleRequest(HttpServletRequest request, HttpServletResponse response)`方法，`Controller`类型的调用。
# HttpRequestHandlerAdapter
`org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter`，实现`HandlerAdapter`接口，基于`org.springframework.web.HttpRequestHandler`的`HandlerAdapter`实现类。
```java
// HttpRequestHandlerAdapter.java

public class HttpRequestHandlerAdapter implements HandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        // 判断是 HttpRequestHandler 类型
        return (handler instanceof HttpRequestHandler);
    }

    @Override
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        // HttpRequestHandler 类型的调用
        ((HttpRequestHandler) handler).handleRequest(request, response);
        return null;
    }

    @Override
    public long getLastModified(HttpServletRequest request, Object handler) {
        // 处理器实现了 LastModified 接口的情况下
        if (handler instanceof LastModified) {
            return ((LastModified) handler).getLastModified(request);
        }
        return -1L;
    }

}
```
# SimpleServletHandlerAdapter
`org.springframework.web.servlet.handler.SimpleServletHandlerAdapter`，实现`HandlerAdapter`接口，基于`javax.servlet.Servlet`的`HandlerAdapter`实现类。
```java
// SimpleServletHandlerAdapter.java

public class SimpleServletHandlerAdapter implements HandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        // 判断是 Servlet 类型
        return (handler instanceof Servlet);
    }

    @Override
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        // Servlet 类型的调用
        ((Servlet) handler).service(request, response);
        return null;
    }

    @Override
    public long getLastModified(HttpServletRequest request, Object handler) {
        return -1;
    }

}
```
# AbstractHandlerMethodAdapter
`org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter`，实现`HandlerAdapter、Ordered`接口，继承`WebContentGenerator`抽象类，基于`org.springframework.web.method.HandlerMethod`的`HandlerMethodAdapter`抽象类。

## 构造方法
```java
// AbstractHandlerMethodAdapter.java

private int order = Ordered.LOWEST_PRECEDENCE;

public AbstractHandlerMethodAdapter() {
    // no restriction of HTTP methods by default
    // 调用 WebContentGenerator 类的构造方法
    // 参数 restrictDefaultSupportedMethods 参数为 false ，表示不需要严格校验 HttpMethod
    super(false);
}
```
## supports
实现`supports(Object handler)`方法，支持`HandlerMethod`类型的处理器。
```
// AbstractHandlerMethodAdapter.java

@Override
public final boolean supports(Object handler) {
	return (handler instanceof HandlerMethod && supportsInternal((HandlerMethod) handler));
}
```
其中，#supportsInternal(HandlerMethod handlerMethod) 方法，由子类实现。
```
// AbstractHandlerMethodAdapter.java

/**
 * Given a handler method, return whether or not this adapter can support it.
 * @param handlerMethod the handler method to check
 * @return whether or not this adapter can adapt the given method
 */
protected abstract boolean supportsInternal(HandlerMethod handlerMethod);
```
## handle
实现`handle(HttpServletRequest request, HttpServletResponse response, Object handler)`方法，处理器请求。
```java
// AbstractHandlerMethodAdapter.java

@Override
@Nullable
public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
		throws Exception {
	return handleInternal(request, response, (HandlerMethod) handler);
}

/**
 * Use the given handler method to handle the request.
 * @param request current HTTP request
 * @param response current HTTP response
 * @param handlerMethod handler method to use. This object must have previously been passed to the
 * {@link #supportsInternal(HandlerMethod)} this interface, which must have returned {@code true}.
 * @return a ModelAndView object with the name of the view and the required model data,
 * or {@code null} if the request has been handled directly
 * @throws Exception in case of errors
 */
@Nullable
protected abstract ModelAndView handleInternal(HttpServletRequest request,
		HttpServletResponse response, HandlerMethod handlerMethod) throws Exception;
```
其中`handleInternal(...)`抽象方法，将`handler`参数是`HandlerMethod`类型。

## getLastModified
`getLastModified(HttpServletRequest request, Object handler)`方法，获得最后更新时间。
```java
// AbstractHandlerMethodAdapter.java

@Override
public final long getLastModified(HttpServletRequest request, Object handler) {
	return getLastModifiedInternal(request, (HandlerMethod) handler);
}

/**
 * Same contract as for {@link javax.servlet.http.HttpServlet#getLastModified(HttpServletRequest)}.
 * @param request current HTTP request
 * @param handlerMethod handler method to use
 * @return the lastModified value for the given handler
 */
protected abstract long getLastModifiedInternal(HttpServletRequest request, HandlerMethod handlerMethod);
```
# RequestMappingHandlerAdapter
org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter ，实现 BeanFactoryAware、InitializingBean 接口，继承 AbstractHandlerMethodAdapter 抽象类，基于 @RequestMapping 注解的 HandlerMethod 的 HandlerMethodAdapter 实现类。

## 构造方法
```java
// RequestMappingHandlerAdapter.java

/**
 * MethodFilter that matches {@link InitBinder @InitBinder} methods.
 */
public static final MethodFilter INIT_BINDER_METHODS = method ->
		AnnotatedElementUtils.hasAnnotation(method, InitBinder.class);

/**
 * MethodFilter that matches {@link ModelAttribute @ModelAttribute} methods.
 */
public static final MethodFilter MODEL_ATTRIBUTE_METHODS = method ->
		(!AnnotatedElementUtils.hasAnnotation(method, RequestMapping.class) &&
				AnnotatedElementUtils.hasAnnotation(method, ModelAttribute.class));

@Nullable
private List<HandlerMethodArgumentResolver> customArgumentResolvers;

@Nullable
private HandlerMethodArgumentResolverComposite argumentResolvers;

@Nullable
private HandlerMethodArgumentResolverComposite initBinderArgumentResolvers;

@Nullable
private List<HandlerMethodReturnValueHandler> customReturnValueHandlers;

@Nullable
private HandlerMethodReturnValueHandlerComposite returnValueHandlers;

@Nullable
private List<ModelAndViewResolver> modelAndViewResolvers;

private ContentNegotiationManager contentNegotiationManager = new ContentNegotiationManager();

private List<HttpMessageConverter<?>> messageConverters;

private List<Object> requestResponseBodyAdvice = new ArrayList<>();

@Nullable
private WebBindingInitializer webBindingInitializer;

private AsyncTaskExecutor taskExecutor = new SimpleAsyncTaskExecutor("MvcAsync");

@Nullable
private Long asyncRequestTimeout;

private CallableProcessingInterceptor[] callableInterceptors = new CallableProcessingInterceptor[0];

private DeferredResultProcessingInterceptor[] deferredResultInterceptors = new DeferredResultProcessingInterceptor[0];

private ReactiveAdapterRegistry reactiveAdapterRegistry = ReactiveAdapterRegistry.getSharedInstance();

private boolean ignoreDefaultModelOnRedirect = false;

private int cacheSecondsForSessionAttributeHandlers = 0;

/**
 * 是否对相同 Session 加锁
 */
private boolean synchronizeOnSession = false;

private SessionAttributeStore sessionAttributeStore = new DefaultSessionAttributeStore();

private ParameterNameDiscoverer parameterNameDiscoverer = new DefaultParameterNameDiscoverer();

@Nullable
private ConfigurableBeanFactory beanFactory;

// ========== 缓存 ==========

private final Map<Class<?>, SessionAttributesHandler> sessionAttributesHandlerCache = new ConcurrentHashMap<>(64);

private final Map<Class<?>, Set<Method>> initBinderCache = new ConcurrentHashMap<>(64);

private final Map<ControllerAdviceBean, Set<Method>> initBinderAdviceCache = new LinkedHashMap<>();

private final Map<Class<?>, Set<Method>> modelAttributeCache = new ConcurrentHashMap<>(64);

private final Map<ControllerAdviceBean, Set<Method>> modelAttributeAdviceCache = new LinkedHashMap<>();
	
public RequestMappingHandlerAdapter() {
	// 初始化 messageConverters
	StringHttpMessageConverter stringHttpMessageConverter = new StringHttpMessageConverter();
	stringHttpMessageConverter.setWriteAcceptCharset(false);  // see SPR-7316

	this.messageConverters = new ArrayList<>(4);
	this.messageConverters.add(new ByteArrayHttpMessageConverter());
	this.messageConverters.add(stringHttpMessageConverter);
	try {
		this.messageConverters.add(new SourceHttpMessageConverter<>());
	} catch (Error err) {
		// Ignore when no TransformerFactory implementation is available
	}
	this.messageConverters.add(new AllEncompassingFormHttpMessageConverter());
}
```