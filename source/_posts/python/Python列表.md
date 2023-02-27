---
title: Python列表
date: 2022-12-03 15:18:21
tags: [python]
categories: python
---

# 序列
序列指的是一块可存放多个值的连续内存空间，这些值按一定顺序排列，可通过每个值所在位置的编号（称为索引）访问它们。

在 Python 中，序列类型包括字符串、列表、元组、集合和字典，这些序列支持以下几种通用的操作，但比较特殊的是，集合和字典不支持索引、切片、相加和相乘操作。

字符串也是一种常见的序列，它也可以直接通过索引访问字符串内的字符。
## 序列索引
序列中，每个元素都有属于自己的编号（索引）。从起始元素开始，索引值从 0 开始递增。

{% asset_img 1.gif 序列索引值示意图 %}

除此之外，Python 还支持索引值是负数，此类索引是从右向左计数，换句话说，从最后一个元素开始计数，从索引值 -1 开始。

{% asset_img 2.gif 负值索引示意图 %}

> 注意，在使用负值作为列序中各元素的索引值时，是从 -1 开始，而不是从 0 开始。

无论是采用正索引值，还是负索引值，都可以访问序列中的任何元素。
```python
str="hello!"
print(str[0],"==",str[-6])
print(str[5],"==",str[-1])
```
## 序列切片
切片操作是访问序列中元素的另一种方法，它可以访问一定范围内的元素，通过切片操作，可以生成一个新的序列。
```py
sname[start : end : step]
```
各个参数的含义：
* `sname`：表示序列的名称；
* `start`：表示切片的开始索引位置（包括该位置），此参数也可以不指定，会默认为 0，也就是从序列的开头进行切片；
* `end`：表示切片的结束索引位置（不包括该位置），如果不指定，则默认为序列的长度；
* `step`：表示在切片过程中，隔几个存储位置（包含当前位置）取一次元素，也就是说，如果`step`的值大于 1，则在进行切片去序列元素时，会“跳跃式”的取元素。如果省略设置`step`的值，则最后一个冒号就可以省略。

```py
str="今天天气真好"
#取索引区间为[0,2]之间（不包括索引2处的字符）的字符串
print(str[:2]) # 今天
#隔 1 个字符取一个字符，区间是整个字符串
print(str[::2]) # 今天真
#取整个字符串，此时 [] 中只需一个冒号即可
print(str[:]) # 今天天气真好
```
## 序列相加
Python 中，支持两种类型相同的序列使用`+`运算符做相加操作，它会将两个序列进行连接，但不会去除重复的元素。

这里所说的“类型相同”，指的是`+`运算符的两侧序列要么都是列表类型，要么都是元组类型，要么都是字符串。
```py
str="!"
print("hello " + "world" + str) # hello world!
```
## 序列相乘
使用数字`n`乘以一个序列会生成新的序列，其内容为原来序列被重复`n`次的结果。
```py
str="hello"
print(str * 3) # hellohellohello
```
比较特殊的是，列表类型在进行乘法运算时，还可以实现初始化指定长度列表的功能。
```py
list = [None] * 5
print(list) # [None, None, None, None, None]
```
## 检查元素是否包含在序列中
可以使用`in`关键字检查某元素是否为序列的成员：
```py
value in sequence
```
其中，`value`表示要检查的元素，`sequence`表示指定的序列。
```py
str="www.baidu.com"
print('c' in str) # True
```
和`in`关键字用法相同，但功能恰好相反的，还有`not in`关键字，它用来检查某个元素是否不包含在指定的序列中：
```py
str="www.baidu.com"
print('c' not in str) # False
```
## 和序列相关的内置函数
Python 提供了几个内置函数，可用于实现与序列相关的一些常用操作。

| 函数        | 功能 |
| :--:        | :--: |
| len()	      |    计算序列的长度，即返回序列中包含多少个元素。 |
| max()	      |    找出序列中的最大元素。注意，对序列使用 sum() 函数时，做加和操作的必须都是数字，不能是字符或字符串，否则该函数将抛出异常，因为解释器无法判定是要做连接操作（+ 运算符可以连接两个序列），还是做加和操作。 |
| min()	      |   找出序列中的最小元素。 |
| list()	  |  将序列转换为列表。 |
| str()	      |   将序列转换为字符串。 |
| sum()	      |   计算元素和。 |
| sorted()	  |  对元素进行排序。 |
| reversed()  |  反向序列中的元素。 |
| enumerate() |	  将序列组合为一个索引序列，多用在 for 循环中。 |

# list列表
Python 中没有数组，但是加入了列表。列表会将所有元素都放在一对中括号`[ ]`里面，相邻元素之间用逗号`,`分隔：
```py
[element1, element2, element3, ..., elementn]
```
格式中，`element1 ~ elementn`表示列表中的元素，个数没有限制，只要是 Python 支持的数据类型就可以。

列表可以存储整数、小数、字符串、列表、元组等任何类型的数据，并且同一个列表中元素的类型也可以不同。

> 注意，在使用列表时，虽然可以将不同类型的数据放入到同一个列表中，但通常情况下不这么做，同一列表中只放入同一类型的数据，这样可以提高程序的可读性。

另外，经常用`list`代指列表，这是因为列表的数据类型就是`list`，通过`type()`函数就可以知道：
```py
type( ["python", 1, [2,3,4] , 3.0] )
<class 'list'>
```
## 创建列表
创建列表的方法可分为两种。
### 1.使用 [] 直接创建列表
使用`[]`创建列表后，一般使用`=`将它赋值给某个变量：
```python
listname = [element1 , element2 , element3 , ... , elementn]
```
其中，`listname`表示变量名，`element1 ~ elementn`表示列表元素。
```py
num = [1, 2, 3, 4, 5, 6, 7]
program = ["JS", "Python", "Java"]
```
另外，使用此方式创建列表时，列表中元素可以有多个，也可以一个都没有：
```
emptylist = []
```
这表明，`emptylist`是一个空列表。
### 2.使用 list() 函数创建列表
Python 还提供了一个内置的函数`list()`，使用它可以将其它数据类型转换为列表类型。
```py
#将字符串转换成列表
list1 = list("hello")
print(list1) # ['h', 'e', 'l', 'l', 'o']
#将元组转换成列表
tuple1 = ('Python', 'Java', 'C++', 'JavaScript')
list2 = list(tuple1)
print(list2) # ['Python', 'Java', 'C++', 'JavaScript']
#将字典转换成列表
dict1 = {'a':100, 'b':42, 'c':9}
list3 = list(dict1)
print(list3) # ['a', 'b', 'c']
#将区间转换成列表
range1 = range(1, 6)
list4 = list(range1)
print(list4) # [1, 2, 3, 4, 5]
#创建空列表
print(list()) # []
```
## 访问列表元素
列表是 Python 序列的一种，我们可以使用索引访问列表中的某个元素（得到的是一个元素的值），也可以使用切片访问列表中的一组元素（得到的是一个新的子列表）。
```
listname[i]
```
其中，`listname`表示列表名字，`i`表示索引值。列表的索引可以是正数，也可以是负数。
```
listname[start : end : step]
```
其中，`listname`表示列表名字，`start`表示起始索引，`end`表示结束索引，`step`表示步长。
```py
url = list("http://www.baidu.com/python")
#使用索引访问列表中的某个元素
print(url[3])  #使用正数索引
print(url[-4])  #使用负数索引
#使用切片访问列表中的一组元素
print(url[9: 18])  #使用正数切片
print(url[9: 18: 3])  #指定步长
print(url[-6: -1])  #使用负数切片
```
运行结果：
```
p
t
['w', '.', 'b', 'a', 'i', 'd', 'u', '.', 'c']
['w', 'a', 'u']
['p', 'y', 't', 'h', 'o']
```
## 删除列表
可以使用`del`关键字将其删除。

实际开发中并不经常使用`del`来删除列表，因为 Python 自带的垃圾回收机制会自动销毁无用的列表，即使开发者不手动删除，Python 也会自动将其回收。
```py
del listname
```
其中，`listname`表示要删除列表的名称。
```py
intlist = [1, 45, 8, 34]
del intlist
print(intlist)
```
运行结果：
```
Traceback (most recent call last):
    File "C:\Users\mozhiyan\Desktop\demo.py", line 4, in <module>
        print(intlist)
NameError: name 'intlist' is not defined
```
# list列表添加元素
使用`+`运算符可以将多个序列连接起来；列表是序列的一种，所以也可以使用`+`进行连接，这样就相当于在第一个列表的末尾添加了另一个列表。
```py
language = ["Python", "C++", "Java"]
birthday = [1991, 1998, 1995]
info = language + birthday
print(language) # ['Python', 'C++', 'Java']
print(birthday) # [1991, 1998, 1995]
print(info) # ['Python', 'C++', 'Java', 1991, 1998, 1995]
```
从运行结果可以发现，使用`+`会生成一个新的列表，原有的列表不会被改变。

`+`更多的是用来拼接列表，而且执行效率并不高，如果想在列表中插入元素，应该使用下面几个专门的方法。
## append()方法添加元素
`append()`方法用于在列表的末尾追加元素：
```py
listname.append(obj)
```
其中，`listname`表示要添加元素的列表；`obj`表示到添加到列表末尾的数据，它可以是单个元素，也可以是列表、元组等。
```py
l = ['Python', 'C++', 'Java']
#追加元素
l.append('PHP')
print(l)
#追加元组，整个元组被当成一个元素
t = ('JavaScript', 'C#', 'Go')
l.append(t)
print(l)
#追加列表，整个列表也被当成一个元素
l.append(['Ruby', 'SQL'])
print(l)
```
运行结果为：
```
['Python', 'C++', 'Java', 'PHP']
['Python', 'C++', 'Java', 'PHP', ('JavaScript', 'C#', 'Go')]
['Python', 'C++', 'Java', 'PHP', ('JavaScript', 'C#', 'Go'), ['Ruby', 'SQL']]
```
可以看到，当给`append()`方法传递列表或者元组时，此方法会将它们视为一个整体，作为一个元素添加到列表中，从而形成包含列表和元组的新列表。
## extend()方法添加元素
`extend()`和`append()`的不同之处在于：`extend()`不会把列表或者元祖视为一个整体，而是把它们包含的元素逐个添加到列表中。
```py
listname.extend(obj)
```
其中，`listname`指的是要添加元素的列表；`obj`表示到添加到列表末尾的数据，它可以是单个元素，也可以是列表、元组等，但不能是单个的数字。
```py
l = ['Python', 'C++', 'Java']
#追加元素
l.extend('C')
print(l)
#追加元组，元祖被拆分成多个元素
t = ('JavaScript', 'C#', 'Go')
l.extend(t)
print(l)
#追加列表，列表也被拆分成多个元素
l.extend(['Ruby', 'SQL'])
print(l)
```
运行结果：
```
['Python', 'C++', 'Java', 'C']
['Python', 'C++', 'Java', 'C', 'JavaScript', 'C#', 'Go']
['Python', 'C++', 'Java', 'C', 'JavaScript', 'C#', 'Go', 'Ruby', 'SQL']
```
## insert()方法插入元素
如果希望在列表中间某个位置插入元素，可以使用`insert()`方法。
```py
listname.insert(index , obj)
```
其中，`index`表示指定位置的索引值。`insert()`会将`obj`插入到`listname`列表第`index`个元素的位置。如果`index`位置在列表中不存在，则将新元素添加至列表结尾。

当插入列表或者元组时，`insert()`也会将它们视为一个整体，作为一个元素插入到列表中，这一点和`append()`是一样的。
```py
l = ['Python', 'C++', 'Java']
#插入元素
l.insert(1, 'C')
print(l)
#插入元组，整个元祖被当成一个元素
t = ('C#', 'Go')
l.insert(2, t)
print(l)
#插入列表，整个列表被当成一个元素
l.insert(3, ['Ruby', 'SQL'])
print(l)
#插入字符串，整个字符串被当成一个元素
l.insert(0, "JS")
print(l)
```
输出结果为：
```
['Python', 'C', 'C++', 'Java']
['Python', 'C', ('C#', 'Go'), 'C++', 'Java']
['Python', 'C', ('C#', 'Go'), ['Ruby', 'SQL'], 'C++', 'Java']
['JS', 'Python', 'C', ('C#', 'Go'), ['Ruby', 'SQL'], 'C++', 'Java']
```
提示，`insert()`主要用来在列表的中间位置插入元素，如果你仅仅希望在列表的末尾追加元素，那更建议使用`append()`和`extend()`。
# list列表删除元素
列表中删除元素主要分为以下 3 种场景：
* 根据目标元素所在位置的索引进行删除，可以使用`del`关键字或者`pop()`方法；
* 根据元素本身的值进行删除，可使用列表（`list`类型）提供的`remove()`方法；
* 将列表中所有元素全部删除，可使用列表（`list`类型）提供的`clear()`方法。

# del：根据索引值删除元素
`del`不仅可以删除整个列表，还可以删除列表中的某些元素。

`del`可以删除列表中的单个元素：
```
del listname[index]
```
其中，`listname`表示列表名称，`index`表示元素的索引值。

`del`也可以删除中间一段连续的元素：
```
del listname[start : end]
```
其中，`start`表示起始索引，`end`表示结束索引。`del`会删除从索引`start`到`end`之间的元素，不包括`end`位置的元素。
```py
lang = ["Python", "C++", "Java", "PHP", "Ruby", "MATLAB"]
#使用正数索引
del lang[2]
print(lang) # ['Python', 'C++', 'PHP', 'Ruby', 'MATLAB']
#使用负数索引
del lang[-2]
print(lang) # ['Python', 'C++', 'PHP', 'MATLAB']
```
```py
lang = ["Python", "C++", "Java", "PHP", "Ruby", "MATLAB"]
del lang[1: 4]
print(lang) # ['Python', 'Ruby', 'MATLAB']
lang.extend(["SQL", "C#", "Go"])
del lang[-5: -2]
print(lang) # ['Python', 'C#', 'Go']
```
# pop()：根据索引值删除元素
`pop()`方法用来删除列表中指定索引处的元素：
```
listname.pop(index)
```
其中，`listname`表示列表名称，`index`表示索引值。如果不写`index`参数，默认会删除列表中的最后一个元素，类似于数据结构中的“出栈”操作。
```py
nums = [40, 36, 89, 2, 36, 100, 7]
nums.pop(3)
print(nums) # [40, 36, 89, 36, 100, 7]
nums.pop()
print(nums) # [40, 36, 89, 36, 100]
```
大部分编程语言都会提供和`pop()`相对应的方法，就是`push()`，该方法用来将元素添加到列表的尾部，类似于数据结构中的“入栈”操作。但是 Python 是个例外，Python 并没有提供`push()`方法，因为完全可以使用`append()`来代替`push()`的功能。
# remove()：根据元素值进行删除
除了`del`关键字，Python 还提供了`remove()`方法，该方法会根据元素本身的值来进行删除操作。

需要注意的是，`remove()`方法只会删除第一个和指定值相同的元素，而且必须保证该元素是存在的，否则会引发`ValueError`错误。
```py
nums = [40, 36, 89, 2, 36, 100, 7]
#第一次删除36
nums.remove(36)
print(nums)
#第二次删除36
nums.remove(36)
print(nums)
#删除78
nums.remove(78)
print(nums)
```
运行结果：
```
[40, 89, 2, 36, 100, 7]
[40, 89, 2, 100, 7]
Traceback (most recent call last):
    File "C:\Users\mozhiyan\Desktop\demo.py", line 9, in <module>
        nums.remove(78)
ValueError: list.remove(x): x not in list
```
## clear()：删除列表所有元素
`clear()`用来删除列表的所有元素，也即清空列表：
```py
url = list("test")
url.clear()
print(url) # []
```
# list列表修改元素
Python 提供了两种修改列表元素的方法，你可以每次修改单个元素，也可以每次修改一组元素（多个）。
## 修改单个元素
修改单个元素非常简单，直接对元素赋值即可。
```py
nums = [40, 36, 89, 2, 36, 100, 7]
nums[2] = -26  #使用正数索引
nums[-3] = -66.2  #使用负数索引
print(nums) # [40, 36, -26, 2, -66.2, 100, 7]
```
使用索引得到列表元素后，通过`=`赋值就改变了元素的值。
## 修改一组元素
Python 支持通过切片语法给一组元素赋值。在进行这种操作时，如果不指定步长（`step`参数），Python 就不要求新赋值的元素个数与原来的元素个数相同；这意味，该操作既可以为列表添加元素，也可以为列表删除元素。
```py
nums = [40, 36, 89, 2, 36, 100, 7]
#修改第 1~4 个元素的值（不包括第4个元素）
nums[1: 4] = [45.25, -77, -52.5]
print(nums) # [40, 45.25, -77, -52.5, 36, 100, 7]
```
如果对空切片（`slice`）赋值，就相当于插入一组新的元素：
```py
nums = [40, 36, 89, 2, 36, 100, 7]
#在4个位置插入元素
nums[4: 4] = [-77, -52.5, 999]
print(nums) # [40, 36, 89, 2, -77, -52.5, 999, 36, 100, 7]
```
使用切片语法赋值时，Python 不支持单个值，下面的写法就是错误的：
```py
nums[4: 4] = -77
```
但是如果使用字符串赋值，Python 会自动把字符串转换成序列，其中的每个字符都是一个元素：
```py
s = list("Hello")
s[2:4] = "XYZ"
print(s) # ['H', 'e', 'X', 'Y', 'Z', 'o']
```
使用切片语法时也可以指定步长（`step`参数），但这个时候就要求所赋值的新元素的个数与原有元素的个数相同：
```py
nums = [40, 36, 89, 2, 36, 100, 7]
#步长为2，为第1、3、5个元素赋值
nums[1: 6: 2] = [0.025, -99, 20.5]
print(nums) # [40, 0.025, 89, -99, 36, 20.5, 7]
```
# list列表查找元素
列表提供了`index()`和`count()`方法，它们都可以用来查找元素。
## index() 方法
`index()`方法用来查找某个元素在列表中出现的位置（也就是索引），如果该元素不存在，则会导致`ValueError`错误，所以在查找之前最好使用`count()`方法判断一下。
```py
listname.index(obj, start, end)
```
其中，`listname`表示列表名称，`obj`表示要查找的元素，`start`表示起始位置，`end`表示结束位置。

`start`和`end`参数用来指定检索范围：
* `start`和`end`可以都不写，此时会检索整个列表；
* 如果只写`start`不写`end`，那么表示检索从`start`到末尾的元素；
* 如果`start`和`end`都写，那么表示检索`start`和`end`之间的元素。

`index()`方法会返回元素所在列表中的索引值。
```py
nums = [40, 36, 89, 2, 36, 100, 7, -20.5, -999]
#检索列表中的所有元素
print( nums.index(2) )
#检索3~7之间的元素
print( nums.index(100, 3, 7) )
#检索4之后的元素
print( nums.index(7, 4) )
#检索一个不存在的元素
print( nums.index(55) )
```
运行结果：
```
3
5
6
Traceback (most recent call last):
    File "C:\Users\mozhiyan\Desktop\demo.py", line 9, in <module>
        print( nums.index(55) )
ValueError: 55 is not in list
```
## count()方法
`count()`方法用来统计某个元素在列表中出现的次数：
```py
listname.count(obj)
```
其中，`listname`代表列表名，`obj`表示要统计的元素。

如果`count()`返回 0，就表示列表中不存在该元素，所以`count()`也可以用来判断列表中的某个元素是否存在。
```py
nums = [40, 36, 89, 2, 36, 100, 7, -20.5, 36]
#统计元素出现的次数
print("36出现了%d次" % nums.count(36))
#判断一个元素是否存在
if nums.count(100):
  print("列表中存在100这个元素")
else:
  print("列表中不存在100这个元素")
```
运行结果：
```
36出现了3次
列表中存在100这个元素
```