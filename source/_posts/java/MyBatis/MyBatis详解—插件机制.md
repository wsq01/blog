


# 概述
MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis允许使用插件来拦截的方法调用包括：
* `Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)`拦截执行器的方法
* `ParameterHandler (getParameterObject, setParameters)`拦截参数的处理
* `ResultSetHandler (handleResultSets, handleOutputParameters)`拦截结果集的处理
* `StatementHandler (prepare, parameterize, batch, update, query)`拦截`Sql`语法构建的处理

Mybatis 采用责任链模式，通过动态代理组织多个拦截器（插件），通过这些拦截器可以改变 Mybatis 的默认行为（诸如 SQL 重写之类的），由于插件会深入到 Mybatis 的核心，因此在编写自己的插件前最好了解下它的原理，以便写出安全高效的插件。
# 拦截器的使用
## 拦截器介绍及配置
首先我们看下 MyBatis 拦截器的接口定义：
```java
public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  Object plugin(Object target);

  void setProperties(Properties properties);
}
```
比较简单，只有 3 个方法。 MyBatis 默认没有一个拦截器接口的实现类，开发者们可以实现符合自己需求的拦截器。下面的 MyBatis 官网的一个拦截器实例：
```java
@Intercepts({@Signature(type= Executor.class, method = "update", args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  public Object intercept(Invocation invocation) throws Throwable {
    return invocation.proceed();
  }
  public Object plugin(Object target) {
    return Plugin.wrap(target, this);
  }
  public void setProperties(Properties properties) {
  }
}
```
全局xml配置：
```xml
<plugins>
    <plugin interceptor="org.format.mybatis.cache.interceptor.ExamplePlugin"></plugin>
</plugins>
```
这个拦截器拦截`Executor`接口的`update`方法（其实也就是`SqlSession`的新增，删除，修改操作），所有执行`executor`的`update`方法都会被该拦截器拦截到。
## 源码分析
首先从源头->配置文件开始分析：

`XMLConfigBuilder`解析 MyBatis 全局配置文件的`pluginElement`私有方法：
```java
private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
        for (XNode child : parent.getChildren()) {
            String interceptor = child.getStringAttribute("interceptor");
            Properties properties = child.getChildrenAsProperties();
            Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
            interceptorInstance.setProperties(properties);
            configuration.addInterceptor(interceptorInstance);
        }
    }
}
```
具体的解析代码其实比较简单，就不贴了，主要就是通过反射实例化`plugin`节点中的`interceptor`属性表示的类。然后调用全局配置类`Configuration`的`addInterceptor`方法。
```java
public void addInterceptor(Interceptor interceptor) {
    interceptorChain.addInterceptor(interceptor);
}
```
这个`interceptorChain`是`Configuration`的内部属性，类型为`InterceptorChain`，也就是一个拦截器链，我们来看下它的定义：
```java
public class InterceptorChain {

    private final List<Interceptor> interceptors = new ArrayList<Interceptor>();

    public Object pluginAll(Object target) {
        for (Interceptor interceptor : interceptors) {
            target = interceptor.plugin(target);
        }
        return target;
    }

    public void addInterceptor(Interceptor interceptor) {
        interceptors.add(interceptor);
    }

    public List<Interceptor> getInterceptors() {
        return Collections.unmodifiableList(interceptors);
    }

}
```
现在我们理解了拦截器配置的解析以及拦截器的归属，现在我们回过头看下为何拦截器会拦截这些方法（`Executor，ParameterHandler，ResultSetHandler，StatementHandler`的部分方法）：
```java
public ParameterHandler newParameterHandler(MappedStatement mappedStatement, Object parameterObject, BoundSql boundSql) {
    ParameterHandler parameterHandler = mappedStatement.getLang().createParameterHandler(mappedStatement, parameterObject, boundSql);
    parameterHandler = (ParameterHandler) interceptorChain.pluginAll(parameterHandler);
    return parameterHandler;
}

public ResultSetHandler newResultSetHandler(Executor executor, MappedStatement mappedStatement, RowBounds rowBounds, ParameterHandler parameterHandler, ResultHandler resultHandler, BoundSql boundSql) {
    ResultSetHandler resultSetHandler = new DefaultResultSetHandler(executor, mappedStatement, parameterHandler, resultHandler, boundSql, rowBounds);
    resultSetHandler = (ResultSetHandler) interceptorChain.pluginAll(resultSetHandler);
    return resultSetHandler;
}

public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);
    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);
    return statementHandler;
}

public Executor newExecutor(Transaction transaction, ExecutorType executorType, boolean autoCommit) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
        executor = new CachingExecutor(executor, autoCommit);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
}
```
以上4个方法都是`Configuration`的方法。这些方法在 MyBatis 的一个操作(新增，删除，修改，查询)中都会被执行到，执行的先后顺序是`Executor，ParameterHandler，ResultSetHandler，StatementHandler`(其中`ParameterHandler`和`ResultSetHandler`的创建是在创建`StatementHandler` [ 3个可用的实现类`CallableStatementHandler,PreparedStatementHandler,SimpleStatementHandler` ] 的时候，其构造函数调用的 [ 这3个实现类的构造函数其实都调用了父类`BaseStatementHandler`的构造函数 ])。

这 4 个方法实例化了对应的对象之后，都会调用`interceptorChain`的`pluginAll`方法，`InterceptorChain`的`pluginAll`刚才已经介绍过了，就是遍历所有的拦截器，然后调用各个拦截器的`plugin`方法。注意：拦截器的`plugin`方法的返回值会直接被赋值给原先的对象。

由于可以拦截`StatementHandler`，这个接口主要处理`sql`语法的构建，因此比如分页的功能，可以用拦截器实现，只需要在拦截器的`plugin`方法中处理`StatementHandler`接口实现类中的`sql`即可，可使用反射实现。

MyBatis 还提供了`@Intercepts`和`@Signature`关于拦截器的注解。官网的例子就是使用了这 2 个注解，还包括了`Plugin`类的使用：
```java
@Override
public Object plugin(Object target) {
    return Plugin.wrap(target, this);
}
```
# 代理链的生成
Mybatis 支持对`Executor、StatementHandler、ParameterHandler`和`ResultSetHandler`进行拦截，也就是说会对这 4 种对象进行代理。通过查看`Configuration`类的源代码我们可以看到，每次都对目标对象进行代理链的生成。下面以`Executor`为例。Mybatis 在创建`Executor`对象时会执行下面一行代码：
```
executor =(Executor) interceptorChain.pluginAll(executor);
```
`InterceptorChain`里保存了所有的拦截器，它在 mybatis 初始化的时候创建。上面这句代码的含义是调用拦截器链里的每个拦截器依次对`executor`进行`plugin`（插入？）代码如下： 
```java
/** 
  * 每一个拦截器对目标类都进行一次代理 
  * @param target 
  * @return 层层代理后的对象 
  */  
 public Object pluginAll(Object target) {  
     for(Interceptor interceptor : interceptors) {  
         target= interceptor.plugin(target);  
     }  
     return target;  
 }
```
下面以一个简单的例子来看看这个`plugin`方法里到底发生了什么：
```java
@Intercepts({@Signature(type = Executor.class, method ="update", args = {MappedStatement.class, Object.class})})  
public class ExamplePlugin implements Interceptor {  
    @Override  
    public Object intercept(Invocation invocation) throws Throwable {  
        return invocation.proceed();  
    }  
  
    @Override  
    public Object plugin(Object target) {  
        return Plugin.wrap(target, this);  
    }  
  
    @Override  
    public void setProperties(Properties properties) {  
    }
}
```
每一个拦截器都必须实现上面的三个方法，其中：
* `Object intercept(Invocation invocation)`是实现拦截逻辑的地方，内部要通过`invocation.proceed()`显式地推进责任链前进，也就是调用下一个拦截器拦截目标方法。
* `Object plugin(Object target)`就是用当前这个拦截器生成对目标`target`的代理，实际是通过`Plugin.wrap(target,this)`来完成的，把目标`target`和拦截器`this`传给了包装函数。
* `setProperties(Properties properties)`用于设置额外的参数，参数配置在拦截器的`Properties`节点里。

注解里描述的是指定拦截方法的签名`[type,method,args]`（即对哪种对象的哪种方法进行拦截），它在拦截前用于决断。

定义自己的`Interceptor`最重要的是要实现`plugin`方法和`intercept`方法，在`plugin`方法中我们可以决定是否要进行拦截进而决定要返回一个什么样的目标对象。而`intercept`方法就是要进行拦截的时候要执行的方法。

对于`plugin`方法而言，其实 Mybatis 已经为我们提供了一个实现。Mybatis 中有一个叫做`Plugin`的类，里面有一个静态方法`wrap(Object target,Interceptor interceptor)`，通过该方法可以决定要返回的对象是目标对象还是对应的代理。这里我们先来看一下`Plugin`的源码：
```java
package org.apache.ibatis.plugin;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Set;

import org.apache.ibatis.reflection.ExceptionUtil;

//这个类是Mybatis拦截器的核心,大家可以看到该类继承了InvocationHandler
//又是JDK动态代理机制
public class Plugin implements InvocationHandler {

  //目标对象
  private Object target;
  //拦截器
  private Interceptor interceptor;
  //记录需要被拦截的类与方法
  private Map<Class<?>, Set<Method>> signatureMap;

  private Plugin(Object target, Interceptor interceptor, Map<Class<?>, Set<Method>> signatureMap) {
    this.target = target;
    this.interceptor = interceptor;
    this.signatureMap = signatureMap;
  }

  //一个静态方法,对一个目标对象进行包装，生成代理类。
  public static Object wrap(Object target, Interceptor interceptor) {
    //首先根据interceptor上面定义的注解 获取需要拦截的信息
    Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
    //目标对象的Class
    Class<?> type = target.getClass();
    //返回需要拦截的接口信息
    Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
    //如果长度为>0 则返回代理类 否则不做处理
    if (interfaces.length > 0) {
      return Proxy.newProxyInstance(
          type.getClassLoader(),
          interfaces,
          new Plugin(target, interceptor, signatureMap));
    }
    return target;
  }

  //代理对象每次调用的方法
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      //通过method参数定义的类 去signatureMap当中查询需要拦截的方法集合
      Set<Method> methods = signatureMap.get(method.getDeclaringClass());
      //判断是否需要拦截
      if (methods != null && methods.contains(method)) {
        return interceptor.intercept(new Invocation(target, method, args));
      }
      //不拦截 直接通过目标对象调用方法
      return method.invoke(target, args);
    } catch (Exception e) {
      throw ExceptionUtil.unwrapThrowable(e);
    }
  }

  //根据拦截器接口（Interceptor）实现类上面的注解获取相关信息
  private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
    //获取注解信息
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    //为空则抛出异常
    if (interceptsAnnotation == null) { // issue #251
      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());      
    }
    //获得Signature注解信息
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<Class<?>, Set<Method>>();
    //循环注解信息
    for (Signature sig : sigs) {
      //根据Signature注解定义的type信息去signatureMap当中查询需要拦截方法的集合
      Set<Method> methods = signatureMap.get(sig.type());
      //第一次肯定为null 就创建一个并放入signatureMap
      if (methods == null) {
        methods = new HashSet<Method>();
        signatureMap.put(sig.type(), methods);
      }
      try {
        //找到sig.type当中定义的方法 并加入到集合
        Method method = sig.type().getMethod(sig.method(), sig.args());
        methods.add(method);
      } catch (NoSuchMethodException e) {
        throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
      }
    }
    return signatureMap;
  }

  //根据对象类型与signatureMap获取接口信息
  private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
    Set<Class<?>> interfaces = new HashSet<Class<?>>();
    //循环type类型的接口信息 如果该类型存在与signatureMap当中则加入到set当中去
    while (type != null) {
      for (Class<?> c : type.getInterfaces()) {
        if (signatureMap.containsKey(c)) {
          interfaces.add(c);
        }
      }
      type = type.getSuperclass();
    }
    //转换为数组返回
    return interfaces.toArray(new Class<?>[interfaces.size()]);
  }

}
```
下面是俩个注解类的定义源码：
```java
package org.apache.ibatis.plugin;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Intercepts {
  Signature[] value();
}
package org.apache.ibatis.plugin;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Signature {
  Class<?> type();

  String method();

  Class<?>[] args();
}
```
# Plugin.wrap方法
从前面可以看出，每个拦截器的`plugin`方法是通过调用`Plugin.wrap`方法来实现的。代码如下：
```java
public static Object wrap(Object target, Interceptor interceptor) {  
   // 从拦截器的注解中获取拦截的类名和方法信息  
   Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);  
   Class<?> type = target.getClass();  
   // 解析被拦截对象的所有接口（注意是接口）  
   Class<?>[] interfaces = getAllInterfaces(type, signatureMap);  
   if(interfaces.length > 0) {  
        // 生成代理对象， Plugin对象为该代理对象的InvocationHandler  （InvocationHandler属于java代理的一个重要概念，不熟悉的请参考相关概念）  
        return Proxy.newProxyInstance(type.getClassLoader(), interfaces, new Plugin(target,interceptor,signatureMap));  
    }  
    return target;  
}
```
这个`Plugin`类有三个属性：
```java
private Object target;// 被代理的目标类

private Interceptor interceptor;// 对应的拦截器

private Map<Class<?>, Set<Method>> signatureMap;// 拦截器拦截的方法缓存
```
`getSignatureMap`方法：
```java
private static Map<Class<?>, Set<Method>> getSignatureMap(Interceptor interceptor) {
    Intercepts interceptsAnnotation = interceptor.getClass().getAnnotation(Intercepts.class);
    if (interceptsAnnotation == null) { // issue #251
      throw new PluginException("No @Intercepts annotation was found in interceptor " + interceptor.getClass().getName());      
    }
    Signature[] sigs = interceptsAnnotation.value();
    Map<Class<?>, Set<Method>> signatureMap = new HashMap<Class<?>, Set<Method>>();
    for (Signature sig : sigs) {
      Set<Method> methods = signatureMap.get(sig.type());
      if (methods == null) {
        methods = new HashSet<Method>();
        signatureMap.put(sig.type(), methods);
      }
      try {
        Method method = sig.type().getMethod(sig.method(), sig.args());
        methods.add(method);
      } catch (NoSuchMethodException e) {
        throw new PluginException("Could not find method on " + sig.type() + " named " + sig.method() + ". Cause: " + e, e);
      }
    }
    return signatureMap;
}
```
`getSignatureMap`方法解释：首先会拿到拦截器这个类的`@Interceptors`注解，然后拿到这个注解的属性`@Signature`注解集合，然后遍历这个集合，遍历的时候拿出`@Signature`注解的`type`属性(`Class`类型)，然后根据这个`type`得到带有`method`属性和`args`属性的`Method`。由于`@Interceptors`注解的`@Signature`属性是一个属性，所以最终会返回一个以`type`为`key，value`为`Set<Method>的Map`。
```
@Intercepts({@Signature(type= Executor.class, method = "update", args = {MappedStatement.class,Object.class})})
```
比如这个`@Interceptors`注解会返回一个`key`为`Executor`，`value`为集合(这个集合只有一个元素，也就是`Method`实例，这个`Method`实例就是`Executor`接口的`update`方法，且这个方法带有`MappedStatement`和`Object`类型的参数)。这个`Method`实例是根据`@Signature`的`method`和`args`属性得到的。如果`args`参数跟`type`类型的`method`方法对应不上，那么将会抛出异常。

`getAllInterfaces`方法：
```java
private static Class<?>[] getAllInterfaces(Class<?> type, Map<Class<?>, Set<Method>> signatureMap) {
    Set<Class<?>> interfaces = new HashSet<Class<?>>();
    while (type != null) {
      for (Class<?> c : type.getInterfaces()) {
        if (signatureMap.containsKey(c)) {
          interfaces.add(c);
        }
      }
      type = type.getSuperclass();
    }
    return interfaces.toArray(new Class<?>[interfaces.size()]);
}
```
`getAllInterfaces`方法解释：根据目标实例`target`(这个`target`就是之前所说的 MyBatis 拦截器可以拦截的类，`Executor,ParameterHandler,ResultSetHandler,StatementHandler`)和它的父类们，返回`signatureMap`中含有`target`实现的接口数组。

所以`Plugin`这个类的作用就是根据`@Interceptors`注解，得到这个注解的属性`@Signature`数组，然后根据每个`@Signature`注解的`type，method，args`属性使用反射找到对应的`Method`。最终根据调用的`target`对象实现的接口决定是否返回一个代理对象替代原先的`target`对象。

我们再次结合`(Executor)interceptorChain.pluginAll(executor)`这个语句来看，这个语句内部对`executor`执行了多次`plugin`，第一次`plugin`后通过`Plugin.wrap`方法生成了第一个代理类，姑且就叫`executorProxy1`，这个代理类的`target`属性是该`executor`对象。第二次`plugin`后通过`Plugin.wrap`方法生成了第二个代理类，姑且叫`executorProxy2`，这个代理类的`target`属性是`executorProxy1...`这样通过每个代理类的`target`属性就构成了一个代理链（从最后一个`executorProxyN`往前查找，通过`target`属性可以找到最原始的`executor`类）。
# 代理链上的拦截代理
链生成后，对原始目标的方法调用都转移到代理者的`invoke`方法上来了。`Plugin`作为`InvocationHandler`的实现类，他的`invoke`方法是怎么样的呢？

比如 MyBatis 官网的例子，当`Configuration`调用`newExecutor`方法的时候，由于`Executor`接口的`update(MappedStatement ms, Object parameter)`方法被拦截器被截获。因此最终返回的是一个代理类`Plugin`，而不是`Executor`。这样调用方法的时候，如果是个代理类，那么会执行：
```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
    try {  
       Set<Method> methods = signatureMap.get(method.getDeclaringClass());  
        if(methods != null && methods.contains(method)) {  
           // 调用代理类所属拦截器的intercept方法，  
           return interceptor.intercept(new Invocation(target, method, args));  
        }  
        return method.invoke(target, args);  
    } catch(Exception e) {  
        throw ExceptionUtil.unwrapThrowable(e);  
    }  
}
```
没错，如果找到对应的方法被代理之后，那么会执行`Interceptor`接口的`interceptor`方法。

在`invoke`里，如果方法签名和拦截中的签名一致，就调用拦截器的拦截方法。我们看到传递给拦截器的是一个`Invocation`对象，这个对象是什么样子的，他的功能又是什么呢？
```java
public class Invocation {  
  
    private Object target;  
    private Method method;  
    private Object[] args;  
   
    public Invocation(Object target, Method method, Object[] args) {  
        this.target =target;  
        this.method =method;  
        this.args =args;  
    }  
    ...  
  
    public Object proceed() throws InvocationTargetException, IllegalAccessException {  
        return method.invoke(target, args);  
    }  
}
```
可以看到，`Invocation`类保存了代理对象的目标类，执行的目标类方法以及传递给它的参数。

在每个拦截器的`intercept`方法内，最后一个语句一定是`return invocation.proceed()`（不这么做的话拦截器链就断了，你的 mybatis 基本上就不能正常工作了）。`invocation.proceed()`只是简单的调用了下`target`的对应方法，如果`target`还是个代理，就又回到了上面的`Plugin.invoke`方法了。这样就形成了拦截器的调用链推进。
```java
public Object intercept(Invocation invocation) throws Throwable {  
    //完成代理类本身的逻辑  
    ...
    //通过invocation.proceed()方法完成调用链的推进
    return invocation.proceed();
}
```
# 总结
MyBatis拦截器接口提供的 3 个方法中，`plugin`方法用于某些处理器(`Handler`)的构建过程。`interceptor`方法用于处理代理类的执行。`setProperties`方法用于拦截器属性的设置。

其实 MyBatis 官网提供的使用`@Interceptors`和`@Signature`注解以及`Plugin`类这样处理拦截器的方法，我们不一定要直接这样使用。我们也可以抛弃这 3 个类，直接在`plugin`方法内部根据`target`实例的类型做相应的操作。

总体来说 MyBatis 拦截器还是很简单的，拦截器本身不需要太多的知识点，但是学习拦截器需要对 MyBatis 中的各个接口很熟悉，因为拦截器涉及到了各个接口的知识点。

我们假设在 MyBatis 配置了一个插件，在运行时会发生什么？
* 所有可能被拦截的处理类都会生成一个代理
* 处理类代理在执行对应方法时，判断要不要执行插件中的拦截方法
* 执行插接中的拦截方法后，推进目标的执行
* 如果有`N`个插件，就有`N`个代理，每个代理都要执行上面的逻辑。这里面的层层代理要多次生成动态代理，是比较影响性能的。虽然能指定插件拦截的位置，但这个是在执行方法时动态判断，初始化的时候就是简单的把插件包装到了所有可以拦截的地方。

因此，在编写插件时需注意以下几个原则：
* 不编写不必要的插件；
* 实现`plugin`方法时判断一下目标类型，是本插件要拦截的对象才执行`Plugin.wrap`方法，否者直接返回目标本省，这样可以减少目标被代理的次数。
