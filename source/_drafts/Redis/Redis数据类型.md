


Redis 的数据类型有：`String`（字符串）、`Hash`（哈希）、`List`（列表）、`Set`（集合）、`Zset`（有序集合）、`BitMap`（2.2 版新增）、`HyperLogLog`（2.8 版新增）、`GEO`（3.2 版新增）、`Stream`（5.0 版新增）。

# String
## 介绍
String 是最基本的 key-value 结构，key 是唯一标识，value 是具体的值，value其实不仅是字符串， 也可以是数字（整数或浮点数），value 最多可以容纳的数据长度是 512M。

{% asset_img 1.png %}

## 内部实现
`String`类型的底层的数据结构实现主要是`int`和 SDS（简单动态字符串）。

SDS 和我们认识的 C 字符串不太一样，之所以没有使用 C 语言的字符串表示，因为 SDS 相比于 C 的原生字符串：
* SDS 不仅可以保存文本数据，还可以保存二进制数据。因为 SDS 使用`len`属性的值而不是空字符来判断字符串是否结束，并且 SDS 的所有 API 都会以处理二进制的方式来处理 SDS 存放在 buf[] 数组里的数据。所以 SDS 不光能存放文本数据，而且能保存图片、音频、视频、压缩文件这样的二进制数据。
* SDS 获取字符串长度的时间复杂度是 O(1)。因为 C 语言的字符串并不记录自身长度，所以获取长度的复杂度为`O(n)`；而 SDS 结构里用`len`属性记录了字符串长度，所以复杂度为 O(1)。
* Redis 的 SDS API 是安全的，拼接字符串不会造成缓冲区溢出。因为 SDS 在拼接字符串之前会检查 SDS 空间是否满足要求，如果空间不够会自动扩容，所以不会导致缓冲区溢出的问题。
字符串对象的内部编码（`encoding`）有 3 种：`int、raw`和`embstr`。

{% asset_img 2.png %}

如果一个字符串对象保存的是整数值，并且这个整数值可以用`long`类型来表示，那么字符串对象会将整数值保存在字符串对象结构的`ptr`属性里面（将`void*`转换成`long`），并将字符串对象的编码设置为`int`。

{% asset_img 3.png %}

如果字符串对象保存的是一个字符串，并且这个字符申的长度小于等于 32 字节（redis2.+版本），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串，并将对象的编码设置为`embstr`，`embstr`编码是专门用于保存短字符串的一种优化编码方式：

{% asset_img 4.png %}

如果字符串对象保存的是一个字符串，并且这个字符串的长度大于 32 字节（redis 2.+版本），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串，并将对象的编码设置为`raw`：

{% asset_img 5.png %}

注意，`embstr`编码和`raw`编码的边界在 redis 不同版本中是不一样的：
* redis 2.+ 是 32 字节
* redis 3.0-4.0 是 39 字节
* redis 5.0 是 44 字节

可以看到`embstr`和`raw`编码都会使用 SDS 来保存值，但不同之处在于`embstr`会通过一次内存分配函数来分配一块连续的内存空间来保存`redisObject`和 SDS，而`raw`编码会通过调用两次内存分配函数来分别分配两块空间来保存`redisObject`和 SDS。Redis 这样做会有很多好处：
* `embstr`编码将创建字符串对象所需的内存分配次数从`raw`编码的两次降低为一次；
* 释放`embstr`编码的字符串对象同样只需要调用一次内存释放函数；
* 因为`embstr`编码的字符串对象的所有数据都保存在一块连续的内存里面可以更好的利用 CPU 缓存提升性能。

但是`embstr`也有缺点的：
如果字符串的长度增加需要重新分配内存时，整个`redisObject`和 SDS 都需要重新分配空间，所以`embstr`编码的字符串对象实际上是只读的，redis没有为`embstr`编码的字符串对象编写任何相应的修改程序。当我们对`embstr`编码的字符串对象执行任何修改命令（例如`append`）时，程序会先将对象的编码从`embstr`转换成`raw`，然后再执行修改命令。

## 常用指令
普通字符串的基本操作：
```shell
# 设置 key-value 类型的值
> SET name lin
OK
# 根据 key 获得对应的 value
> GET name
"lin"
# 判断某个 key 是否存在
> EXISTS name
(integer) 1
# 返回 key 所储存的字符串值的长度
> STRLEN name
(integer) 3
# 删除某个 key 对应的值
> DEL name
(integer) 1
```
批量设置:
```shell
# 批量设置 key-value 类型的值
> MSET key1 value1 key2 value2 
OK
# 批量获取多个 key 对应的 value
> MGET key1 key2 
1) "value1"
2) "value2"
```
计数器（字符串的内容为整数的时候可以使用）：
```shell
# 设置 key-value 类型的值
> SET number 0
OK
# 将 key 中储存的数字值增一
> INCR number
(integer) 1
# 将key中存储的数字值加 10
> INCRBY number 10
(integer) 11
# 将 key 中储存的数字值减一
> DECR number
(integer) 10
# 将key中存储的数字值键 10
> DECRBY number 10
(integer) 0
```
过期（默认为永不过期）：
```shell
# 设置 key 在 60 秒后过期（该方法是针对已经存在的key设置过期时间）
> EXPIRE name  60 
(integer) 1
# 查看数据还有多久过期
> TTL name 
(integer) 51

#设置 key-value 类型的值，并设置该key的过期时间为 60 秒
> SET key  value EX 60
OK
> SETEX key  60 value
OK
```
不存在就插入：
```shell
# 不存在就插入（not exists）
>SETNX key value
(integer) 1
```
