



元组（tuple）是 Python 中另一个重要的序列结构，和列表类似，元组也是由一系列按特定顺序排序的元素组成。

元组和列表（list）的不同之处在于：
列表的元素是可以更改的，包括修改元素值，删除和插入元素，所以列表是可变序列；
而元组一旦被创建，它的元素就不可更改了，所以元组是不可变序列。

元组也可以看做是不可变的列表，通常情况下，元组用于保存无需修改的内容。

从形式上看，元组的所有元素都放在一对小括号( )中，相邻元素之间用逗号,分隔，如下所示：
```
(element1, element2, ... , elementn)
```
其中 element1~elementn 表示元组中的各个元素，个数没有限制，只要是 Python 支持的数据类型就可以。

从存储内容上看，元组可以存储整数、实数、字符串、列表、元组等任何类型的数据，并且在同一个元组中，元素的类型可以不同，例如：
("c.biancheng.net", 1, [2,'a'], ("abc",3.0))

在这个元组中，有多种类型的数据，包括整形、字符串、列表、元组。

另外，我们都知道，列表的数据类型是 list，那么元组的数据类型是什么呢：
```
>>> type( ("c.biancheng.net",1,[2,'a'],("abc",3.0)) )
<class 'tuple'>
```
可以看到，元组是 tuple 类型，这也是很多教程中用 tuple 指代元组的原因。
# Python创建元组
Python 提供了两种创建元组的方法，下面一一进行介绍。
1) 使用 ( ) 直接创建
通过( )创建元组后，一般使用=将它赋值给某个变量，具体格式为：
```
tuplename = (element1, element2, ..., elementn)
```
其中，tuplename 表示变量名，element1 ~ elementn 表示元组的元素。

例如，下面的元组都是合法的：
num = (7, 14, 21, 28, 35)
course = ("Python教程", "http://c.biancheng.net/python/")
abc = ( "Python", 19, [1,2], ('c',2.0) )

在 Python 中，元组通常都是使用一对小括号将所有元素包围起来的，但小括号不是必须的，只要将各元素用逗号隔开，Python 就会将其视为元组，请看下面的例子：
course = "Python教程", "http://c.biancheng.net/python/"
print(course)
运行结果为：
('Python教程', 'http://c.biancheng.net/python/')


需要注意的一点是，当创建的元组中只有一个字符串类型的元素时，该元素后面必须要加一个逗号,，否则 Python 解释器会将它视为字符串。
```
#最后加上逗号
a =("http://c.biancheng.net/cplus/",)
print(type(a))
print(a)
#最后不加逗号
b = ("http://c.biancheng.net/socket/")
print(type(b))
print(b)
```
运行结果为：
```
<class 'tuple'>
('http://c.biancheng.net/cplus/',)
<class 'str'>
http://c.biancheng.net/socket/
```
你看，只有变量 a 才是元组，后面的变量 b 是一个字符串。
2) 使用tuple()函数创建元组
除了使用( )创建元组外，Python 还提供了一个内置的函数 tuple()，用来将其它数据类型转换为元组类型。
```
tuple(data)
```
其中，data 表示可以转化为元组的数据，包括字符串、元组、range 对象等。

tuple() 使用示例：
```
#将字符串转换成元组
tup1 = tuple("hello")
print(tup1)
#将列表转换成元组
list1 = ['Python', 'Java', 'C++', 'JavaScript']
tup2 = tuple(list1)
print(tup2)
#将字典转换成元组
dict1 = {'a':100, 'b':42, 'c':9}
tup3 = tuple(dict1)
print(tup3)
#将区间转换成元组
range1 = range(1, 6)
tup4 = tuple(range1)
print(tup4)
#创建空元组
print(tuple())
```
运行结果为：
```
('h', 'e', 'l', 'l', 'o')
('Python', 'Java', 'C++', 'JavaScript')
('a', 'b', 'c')
(1, 2, 3, 4, 5)
()
```
Python访问元组元素
和列表一样，我们可以使用索引（Index）访问元组中的某个元素（得到的是一个元素的值），也可以使用切片访问元组中的一组元素（得到的是一个新的子元组）。

使用索引访问元组元素的格式为：
tuplename[i]

其中，tuplename 表示元组名字，i 表示索引值。元组的索引可以是正数，也可以是负数。

使用切片访问元组元素的格式为：
tuplename[start : end : step]

其中，start 表示起始索引，end 表示结束索引，step 表示步长。

以上两种方式我们已在《Python序列》中进行了讲解，这里就不再赘述了，仅作示例演示，请看下面代码：
url = tuple("http://c.biancheng.net/shell/")
#使用索引访问元组中的某个元素
print(url[3])  #使用正数索引
print(url[-4])  #使用负数索引
#使用切片访问元组中的一组元素
print(url[9: 18])  #使用正数切片
print(url[9: 18: 3])  #指定步长
print(url[-6: -1])  #使用负数切片
运行结果：
p
e
('b', 'i', 'a', 'n', 'c', 'h', 'e', 'n', 'g')
('b', 'n', 'e')
('s', 'h', 'e', 'l', 'l')

# 修改元组
元组是不可变序列，元组中的元素不能被修改，所以我们只能创建一个新的元组去替代旧的元组。

例如，对元组变量进行重新赋值：
```
tup = (100, 0.5, -36, 73)
print(tup)
#对元组进行重新赋值
tup = ('Shell脚本',"http://c.biancheng.net/shell/")
print(tup)
```
运行结果为：
```
(100, 0.5, -36, 73)
('Shell脚本', 'http://c.biancheng.net/shell/')
```
另外，还可以通过连接多个元组（使用+可以拼接元组）的方式向元组中添加新元素，例如：
```
tup1 = (100, 0.5, -36, 73)
tup2 = (3+12j, -54.6, 99)
print(tup1+tup2)
print(tup1)
print(tup2)
```
运行结果为：
```
(100, 0.5, -36, 73, (3+12j), -54.6, 99)
(100, 0.5, -36, 73)
((3+12j), -54.6, 99)
```
你看，使用+拼接元组以后，tup1 和 tup2 的内容没法发生改变，这说明生成的是一个新的元组。
# 删除元组
可以通过`del`关键字将其删除：
```py
tup = ('百度', "http://www.baidu.com/")
print(tup)
del tup
print(tup)
```
运行结果为：
```
('百度', "http://www.baidu.com/")
Traceback (most recent call last):
    File "C:\Users\mozhiyan\Desktop\demo.py", line 4, in <module>
        print(tup)
NameError: name 'tup' is not defined
```
Python 自带垃圾回收功能，会自动销毁不用的元组，所以一般不需要通过`del`来手动删除。