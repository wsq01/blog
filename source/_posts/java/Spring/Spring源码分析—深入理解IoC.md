

IoC 全称为`Inversion of Control`，翻译为 “控制反转”，它还有一个别名为 DI（`Dependency Injection`），即依赖注入。

所谓 IoC，就是由 Spring IoC 容器来负责对象的生命周期和对象之间的关系。

# 各个组件
{% asset_img 1.png %}

该图为 ClassPathXmlApplicationContext 的类继承体系结构，虽然只有一部分，但是它基本上包含了 IoC 体系中大部分的核心类和接口。

下面我们就针对这个图进行简单的拆分和补充说明。
## Resource 体系
`org.springframework.core.io.Resource`，对资源的抽象。它的每一个实现类都代表了一种资源的访问策略，如`ClassPathResource、RLResource、FileSystemResource`等。

{% asset_img 2.png %}

### ResourceLoader 体系
有了资源，就应该有资源加载，Spring 利用`org.springframework.core.io.ResourceLoader`来进行统一资源加载，类图如下：

{% asset_img 3.png %}
## BeanFactory 体系
`org.springframework.beans.factory.BeanFactory`，是一个非常纯粹的`bean`容器，它是 IoC 必备的数据结构，其中`BeanDefinition`是它的基本结构。`BeanFactory`内部维护着一个`BeanDefinition map`，并可根据`BeanDefinition`的描述进行`bean`的创建和管理。

{% asset_img 4.png %}

`BeanFactory`有三个直接子类`ListableBeanFactory、HierarchicalBeanFactory`和`AutowireCapableBeanFactory`。

`DefaultListableBeanFactory`为最终默认实现，它实现了所有接口。
## BeanDefinition 体系
`org.springframework.beans.factory.config.BeanDefinition`，用来描述 Spring 中的`Bean`对象。

{% asset_img 5.png %}

## BeanDefinitionReader 体系
`org.springframework.beans.factory.support.BeanDefinitionReader`的作用是读取 Spring 的配置文件的内容，并将其转换成 Ioc 容器内部的数据结构：`BeanDefinition`。

{% asset_img 6.png %}

## ApplicationContext 体系
`org.springframework.context.ApplicationContext`，这个就是大名鼎鼎的 Spring 容器，它叫做应用上下文，与我们应用息息相关。它继承 `BeanFactory`，所以它是`BeanFactory`的扩展升级版，如果`BeanFactory`是屌丝的话，那么`ApplicationContext`则是名副其实的高富帅。由于`ApplicationContext`的结构就决定了它与`BeanFactory`的不同，其主要区别有：
* 继承`org.springframework.context.MessageSource`接口，提供国际化的标准访问策略。
* 继承`org.springframework.context.ApplicationEventPublisher`接口，提供强大的事件机制。
* 扩展`ResourceLoader`，可以用来加载多种`Resource`，可以灵活访问不同的资源。
* 对 Web 应用的支持。

{% asset_img 7.png %}

上面五个体系可以说是 Spring IoC 中最核心的部分，后面也是针对这五个部分进行源码分析。其实 IoC 咋一看还是挺简单的，无非就是将配置文件（暂且认为是 xml 文件）进行解析，然后放到一个`Map`里面就差不多了。

另外，通过上面五个体系，我们可以看出，IoC 主要由`spring-beans`和`spring-context`项目，进行实现。
