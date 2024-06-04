


# MyBatis二级缓存实现
MyBatis 的二级缓存是`Application`级别的缓存，它可以提高对数据库查询的效率，以提高应用的性能。
## MyBatis的缓存机制整体设计以及二级缓存的工作模式

{% asset_img mybatis-y-cache-7.png %}

如图所示，当开一个会话时，一个`SqlSession`对象会使用一个`Executor`对象来完成会话操作，MyBatis 的二级缓存机制的关键就是对这个`Executor`对象做文章。如果用户配置了`cacheEnabled=true`，那么 MyBatis 在为`SqlSession`对象创建`Executor`对象时，会对`Executor`对象加上一个装饰者：`CachingExecutor`，这时`SqlSession`使用`CachingExecutor`对象来完成操作请求。`CachingExecutor`对于查询请求，会先判断该查询请求在`Application`级别的二级缓存中是否有缓存结果，如果有查询结果，则直接返回缓存结果；如果缓存中没有，再交给真正的`Executor`对象来完成查询操作，之后`CachingExecutor`会将真正`Executor`返回的查询结果放置到缓存中，然后在返回给用户。

{% asset_img mybatis-y-cache-8.png %}

`CachingExecutor`是`Executor`的装饰者，以增强`Executor`的功能，使其具有缓存查询的功能，这里用到了设计模式中的装饰者模式，`CachingExecutor`和`Executor`的接口的关系如下类图所示：

{% asset_img mybatis-y-cache-9.png %}

## MyBatis二级缓存的划分
MyBatis 并不是简单地对整个`Application`就只有一个`Cache`缓存对象，它将缓存划分的更细，即是`Mapper`级别的，即每一个`Mapper`都可以拥有一个`Cache`对象，具体如下：

**为每一个`Mapper`分配一个`Cache`缓存对象（使用`<cache>`节点配置）**

MyBatis 将`Application`级别的二级缓存细分到`Mapper`级别，即对于每一个`Mapper.xml`，如果在其中使用了`<cache>`节点，则 MyBatis 会为这个`Mapper`创建一个`Cache`缓存对象，如下图所示：

{% asset_img mybatis-y-cache-10.png %}

注：上述的每一个`Cache`对象，都会有一个自己所属的`namespace`命名空间，并且会将`Mapper`的`namespace`作为它们的`ID`；

**多个`Mapper`共用一个`Cache`缓存对象（使用`<cache-ref>`节点配置）**

如果你想让多个`Mapper`公用一个`Cache`的话，你可以使用`<cache-ref namespace="">`节点，来指定你的这个`Mapper`使用到了哪一个`Mapper`的`Cache`缓存。

{% asset_img mybatis-y-cache-11.png %}

## 使用二级缓存，必须要具备的条件
MyBatis 对二级缓存的支持粒度很细，它会指定某一条查询语句是否使用二级缓存。

虽然在`Mapper`中配置了`<cache>`,并且为此`Mapper`分配了`Cache`对象，这并不表示我们使用`Mapper`中定义的查询语句查到的结果都会放置到`Cache`对象之中，我们必须指定`Mapper`中的某条选择语句是否支持缓存，即如下所示，在`<select>`节点中配置`useCache="true"`，`Mapper`才会对此`Select`的查询支持缓存特性，否则，不会对此`Select`查询，不会经过`Cache`缓存。如下所示，`Select`语句配置了`useCache="true"`，则表明这条`Select`语句的查询会使用二级缓存。
```xml
<select id="selectByMinSalary" resultMap="BaseResultMap" parameterType="java.util.Map" useCache="true">
```
总之，要想使某条`Select`查询支持二级缓存，你需要保证：
* MyBatis 支持二级缓存的总开关：全局配置变量参数`cacheEnabled=true`
* 该`select`语句所在的`Mapper`，配置了`<cache>`或`<cached-ref>`节点，并且有效
* 该select语句的参数`useCache=true`

## 一级缓存和二级缓存的使用顺序
请注意，如果你的 MyBatis 使用了二级缓存，并且你的`Mapper`和`select`语句也配置使用了二级缓存，那么在执行`select`查询的时候，MyBatis 会先从二级缓存中取输入，其次才是一级缓存，即 MyBatis 查询数据的顺序是：二级缓存 ———> 一级缓存 ——> 数据库。

## 二级缓存实现的选择
MyBatis 对二级缓存的设计非常灵活，它自己内部实现了一系列的`Cache`缓存实现类，并提供了各种缓存刷新策略如 LRU，FIFO 等等；另外，MyBatis 还允许用户自定义`Cache`接口实现，用户是需要实现`org.apache.ibatis.cache.Cache`接口，然后将`Cache`实现类配置在`<cache type="">`节点的`type`属性上即可；除此之外，MyBatis 还支持跟第三方内存缓存库如`Memecached`的集成，总之，使用 MyBatis 的二级缓存有三个选择：
* MyBatis 自身提供的缓存实现；
* 用户自定义的`Cache`接口实现；
* 跟第三方内存缓存库的集成；

## MyBatis自身提供的二级缓存的实现
MyBatis 自身提供了丰富的，并且功能强大的二级缓存的实现，它拥有一系列的`Cache`接口装饰者，可以满足各种对缓存操作和更新的策略。

MyBatis 定义了大量的`Cache`的装饰器来增强`Cache`缓存的功能，如下类图所示。

{% asset_img mybatis-y-cache-12.png %}

对于每个`Cache`而言，都有一个容量限制，MyBatis 各供了各种策略来对`Cache`缓存的容量进行控制，以及对`Cache`中的数据进行刷新和置换。MyBatis 主要提供了以下几个刷新和置换策略：
* `LRU(Least Recently Used)`：最近最少使用算法，即如果缓存中容量已经满了，会将缓存中最近最少被使用的缓存记录清除掉，然后添加新的记录；
* `FIFO(First in first out)`：先进先出算法，如果缓存中的容量已经满了，那么会将最先进入缓存中的数据清除掉；
* `Scheduled`：指定时间间隔清空算法，该算法会以指定的某一个时间间隔将`Cache`缓存中的数据清空；

# 如何细粒度地控制你的MyBatis二级缓存
## 一个关于MyBatis的二级缓存的实际问题
现有`AMapper.xml`中定义了对数据库表`ATable`的 CRUD 操作，`BMapper`定义了对数据库表`BTable`的 CRUD 操作；

假设 MyBatis 的二级缓存开启，并且`AMapper`中使用了二级缓存，`AMapper`对应的二级缓存为`ACache`；

除此之外，`AMapper`中还定义了一个跟`BTable`有关的查询语句，类似如下所述：
```xml
<select id="selectATableWithJoin" resultMap="BaseResultMap" useCache="true">  
  select * from ATable left join BTable on ....  
</select>
```
执行以下操作：
执行`AMapper`中的`"selectATableWithJoin"`操作，此时会将查询到的结果放置到`AMapper`对应的二级缓存`ACache`中；
执行`BMapper`中对`BTable`的更新操作`(update、delete、insert)`后，`BTable`的数据更新；
再执行 1 完全相同的查询，这时候会直接从`AMapper`二级缓存`ACache`中取值，将`ACache`中的值直接返回；

好，问题就出现在第 3 步上：

由于`AMapper`的`“selectATableWithJoin”`对应的 SQL 语句需要和`BTable`进行`join`查找，而在第 2 步`BTable`的数据已经更新了，但是第 3 步查询的值是第 1 步的缓存值，已经极有可能跟真实数据库结果不一样，即`ACache`中缓存数据过期了！

总结来看，就是：对于某些使用了`join`连接的查询，如果其关联的表数据发生了更新，`join`连接的查询由于先前缓存的原因，导致查询结果和真实数据不同步；

从 MyBatis 的角度来看，这个问题可以这样表述：对于某些表执行了更新`(update、delete、insert)`操作后，如何去清空跟这些表有关联的查询语句所造成的缓存
## 当前MyBatis二级缓存的工作机制
MyBatis 二级缓存的一个重要特点：即松散的`Cache`缓存管理和维护

{% asset_img mybatis-y-cache-21.png %}

一个`Mapper`中定义的增删改查操作只能影响到自己关联的`Cache`对象。如上图所示的`Mapper namespace1`中定义的若干 CRUD 语句，产生的缓存只会被放置到相应关联的`Cache1`中，即`Mapper namespace2,namespace3,namespace4`中的 CRUD 的语句不会影响到`Cache1`。

可以看出，Mapper之间的缓存关系比较松散，相互关联的程度比较弱。

现在再回到上面描述的问题，如果我们将`AMapper`和`BMapper`共用一个`Cache`对象，那么，当`BMapper`执行更新操作时，可以清空对应`Cache`中的所有的缓存数据，这样的话，数据不是也可以保持最新吗？

确实这个也是一种解决方案，不过，它会使缓存的使用效率变的很低！`AMapper`和`BMapper`的任意的更新操作都会将共用的`Cache`清空，会频繁地清空`Cache`，导致`Cache`实际的命中率和使用率就变得很低了，所以这种策略实际情况下是不可取的。

最理想的解决方案就是：对于某些表执行了更新(`update、delete、insert`)操作后，去清空跟这些指定的表有关联的查询语句所造成的缓存; 这样，就是以很细的粒度管理 MyBatis 内部的缓存，使得缓存的使用率和准确率都能大大地提升。
