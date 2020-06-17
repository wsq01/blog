

Redis 属于键值（key-value）数据库，键值数据库会使用哈希表存储键值和数据，其中 key 作为唯一的标识，而且 key 和 value 可以是任何的内容，不论是简单的对象还是复杂的对象都可以存储。键值数据库的查询性能高，易于扩展。
# Redis 是什么，为什么这么快
Redis 全称是 REmote DIctionary Server，从名字中你也能看出来它用字典结构存储数据，也就是`key-value`类型的数据。

Redis 的查询效率非常高，根据官方提供的数据，Redis 每秒最多处理的请求可以达到 10 万次。

为什么这么快呢？

Redis 采用 ANSI C 语言编写，它和 SQLite 一样。采用 C 语言进行编写的好处是底层代码执行效率高，依赖性低，因为使用 C 语言开发的库没有太多运行时（Runtime）依赖，而且系统的兼容性好，稳定性高。

此外，Redis 是基于内存的数据库，我们之前讲到过，这样可以避免磁盘 I/O，因此 Redis 也被称为缓存工具。

其次，数据结构结构简单，Redis 采用`Key-Value`方式进行存储，也就是使用 Hash 结构进行操作，数据的操作复杂度为`O(1)`。

但 Redis 快的原因还不止这些，它采用单进程单线程模型，这样做的好处就是避免了上下文切换和不必要的线程之间引起的资源竞争。

在技术上 Redis 还采用了多路 I/O 复用技术。这里的多路指的是多个`socket`网络连接，复用指的是复用同一个线程。采用多路 I/O 复用技术的好处是可以在同一个线程中处理多个 I/O 请求，尽量减少网络 I/O 的消耗，提升使用效率。
# Redis 的数据类型
相比 Memcached，Redis 有一个非常大的优势，就是支持多种数据类型。Redis 支持的数据类型包括字符串、哈希、列表、集合、有序集合等。

字符串类型是 Redis 提供的最基本的数据类型，对应的结构是`key-value`。

如果我们想要设置某个键的值，使用方法为`set key value`，比如我们想要给`name`这个键设置值为`zhangfei`，可以写成`set name zhangfei`。如果想要取某个键的值，可以使用`get key`，比如想取`name`的值，写成`get name`即可。

哈希（hash）提供了字段和字段值的映射，对应的结构是`key-field-value`。

如果我们想要设置某个键的哈希值，可以使用`hset key field value`，如果想要给`user1`设置`username`为`zhangfei`，`age`为 28，可以写成下面这样：
```
hset user1 username zhangfei
hset user1 age 28
```
如果我们想要同时将多个`field-value`设置给某个键`key`的时候，可以使用`hmset key field value [field value...]`，比如上面这个可以写成：
```
Hmset user1 username zhangfei age 28
```
如果想要取某个键的某个`field`字段值，可以使用`hget key field`，比如想要取`user1`的`username`，那么写成`hget user1 username`即可。

如果想要一次获取某个键的多个`field`字段值，可以使用`hmget key field[field...]`，比如想要取`user1`的`username`和`age`，可以写成`hmget user1 username age`。

字符串列表（list）的底层是一个双向链表结构，所以我们可以向列表的两端添加元素，时间复杂度都为 O(1)，同时我们也可以获取列表中的某个片段。

如果想要向列表左侧增加元素可以使用：LPUSH key value [...]，比如我们给 heroList 列表向左侧添加 zhangfei、guanyu 和 liubei 这三个元素，可以写成：

LPUSH heroList zhangfei guanyu liubei
同样，我们也可以使用RPUSH key value [...]向列表右侧添加元素，比如我们给 heroList 列表向右侧添加 dianwei、lvbu 这两个元素，可以写成下面这样：

RPUSH heroList dianwei lvbu
 
如果我们想要获取列表中某一片段的内容，使用LRANGE key start stop即可，比如我们想要获取 heroList 从 0 到 4 位置的数据，写成LRANGE heroList 0 4即可。