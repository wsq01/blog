---
title: Python集合
date: 2022-12-07 16:41:21
tags: [python]
categories: python
---


Python 中的集合，和数学中的集合概念一样，用来保存不重复的元素，即集合中的元素都是唯一的，互不相同。

从形式上看，和字典类似，Python 集合会将所有元素放在一对大括号`{}`中，相邻元素之间用`,`分隔：
```
{element1,element2,...,elementn}
```
其中，`elementn`表示集合中的元素，个数没有限制。

从内容上看，同一集合中，只能存储不可变的数据类型，包括整形、浮点型、字符串、元组，无法存储列表、字典、集合这些可变的数据类型，否则 Python 解释器会抛出`TypeError`错误。
```
>>> {{'a':1}}
Traceback (most recent call last):
  File "<pyshell#8>", line 1, in <module>
    {{'a':1}}
TypeError: unhashable type: 'dict'
>>> {[1,2,3]}
Traceback (most recent call last):
  File "<pyshell#9>", line 1, in <module>
    {[1,2,3]}
TypeError: unhashable type: 'list'
>>> {{1,2,3}}
Traceback (most recent call last):
  File "<pyshell#10>", line 1, in <module>
    {{1,2,3}}
TypeError: unhashable type: 'set'
```
并且需要注意的是，数据必须保证是唯一的，因为集合对于每种数据元素，只会保留一份。

由于 Python 中的集合是无序的，所以每次输出时元素的排序顺序可能都不相同。

Python 中有两种集合类型，一种是`set`类型的集合，另一种是`frozenset`类型的集合，它们唯一的区别是，`set`类型集合可以做添加、删除元素的操作，而`forzenset`类型集合不行。
# 创建集合
Python 提供了 2 种创建集合的方法，分别是使用`{}`创建和使用`set()`函数将列表、元组等类型数据转换为集合。
## 使用 {} 创建
创建`set`集合可以像列表、元素和字典一样，直接将集合赋值给变量：
```
setname = {element1,element2,...,elementn}
```
其中，`setname`表示集合的名称。
```py
a = {1,'c',1,(1,2,3),'c'}
print(a) # {1, 'c', (1, 2, 3)}
```
## set()函数创建集合
`set()`函数为内置函数，其功能是将字符串、列表、元组、`range`对象等可迭代对象转换成集合。
```
setname = set(iteration)
```
其中，`iteration`就表示字符串、列表、元组、`range`对象等数据。
```py
set1 = set("hello")
set2 = set([1,2,3,4,5])
set3 = set((1,2,3,4,5))
print("set1:", set1) # set1: {'h', 'e', 'l', 'l', 'o'}
print("set2:", set2) # set2: {1, 2, 3, 4, 5}
print("set3:", set3) # set3: {1, 2, 3, 4, 5}
```
> 注意，如果要创建空集合，只能使用`set()`函数实现。因为直接使用一对`{}`，Python 解释器会将其视为一个空字典。

## 访问集合元素
由于集合中的元素是无序的，因此无法向列表那样使用下标访问元素。Python 中，访问集合元素最常用的方法是使用循环结构，将集合中的数据逐一读取出来。
```py
a = {1, 'c', 1, (1,2,3), 'c'}
for ele in a:
  print(ele,end=' ')
```
运行结果为：
```
1 c (1, 2, 3)
```
## 删除集合
删除集合类型可以使用`del()`语句：
```py
a = {1, 'c', 1, (1,2,3), 'c'}
print(a)
del(a)
print(a)
```
运行结果为：
```
{1, 'c', (1, 2, 3)}
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 4, in <module>
    print(a)
NameError: name 'a' is not defined
```
# 向集合中添加元素
`set`集合中添加元素，可以使用`set`类型提供的`add()`方法实现：
```
setname.add(element)
```
其中，`setname`表示要添加元素的集合，`element`表示要添加的元素内容。

需要注意的是，使用`add()`方法添加的元素，只能是数字、字符串、元组或者布尔类型（`True`和`False`）值，不能添加列表、字典、集合这类可变的数据，否则 Python 解释器会报`TypeError`错误。
```py
a = {1, 2, 3}
a.add((1,2))
print(a)
a.add([1,2])
print(a)
```
运行结果为：
```
{(1, 2), 1, 2, 3}
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 4, in <module>
    a.add([1,2])
TypeError: unhashable type: 'list'
```
# 从集合中删除元素
删除现有`set`集合中的指定元素，可以使用`remove()`方法：
```
setname.remove(element)
```
使用此方法删除集合中元素，需要注意的是，如果被删除元素本就不包含在集合中，则此方法会抛出`KeyError`错误：
```py
a = {1, 2, 3}
a.remove(1)
print(a)
a.remove(1)
print(a)
```
运行结果为：
```
{2, 3}
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 4, in <module>
    a.remove(1)
KeyError: 1
```
上面程序中，由于集合中的元素 1 已被删除，因此当再次尝试使用`remove()`方法删除时，会引发`KeyError`错误。

如果我们不想在删除失败时令解释器提示`KeyError`错误，还可以使用`discard()`方法，此方法和`remove()`方法的用法完全相同，唯一的区别就是，当删除集合中元素失败时，此方法不会抛出任何错误。
```py
a = {1, 2, 3}
a.remove(1)
print(a)
a.discard(1)
print(a)
```
运行结果为：
```
{2, 3}
{2, 3}
```
# 集合做交集、并集、差集运算
有 2 个集合，分别为`set1={1, 2, 3}`和`set2={3, 4, 5}`。以这两个集合为例，分别做不同运算的结果如表。

| 运算操作  | Python运算符 | 含义	                             | 例子 |
| :--: | :--: | :--: | :--: |
| 交集      | &	           | 取两集合公共的元素                | set1 & set2 <br> {3} |
| 并集      | `|`	         | 取两集合全部的元素                | `set1 | set2` <br> {1,2,3,4,5} |
| 差集      | -	           | 取一个集合中另一集合没有的元素    |  set1 - set2 <br> {1,2} <br> set2 - set1 <br> {4,5} |
| 对称差集  | ^	           | 取集合 A 和 B 中不属于 A&B 的元素 |  set1 ^ set2 <br> {1,2,4,5} |

# 集合方法
| 方法名 | 语法格式 | 功能 | 实例 |
| :--: | :--: | :--: | :--: |
| add() | set1.add() | 向 set1 集合中添加数字、字符串、元组或者布尔类型 | set1 = {1,2,3} <br> set1.add((1,2)) <br> set1 <br> {(1, 2), 1, 2, 3} |
| clear() | set1.clear() | 清空 set1 集合中所有元素 | set1 = {1,2,3} <br> set1.clear() <br> set1 <br> set() <br> set()才表示空集合，{}表示的是空字典 |
| copy() | set2 = set1.copy() | 拷贝 set1 集合给 set2 | set1 = {1,2,3} <br> set2 = set1.copy() <br> set1.add(4) <br> set1 {1, 2, 3, 4} <br> set1 {1, 2, 3} |
| difference() | set3 = set1.difference(set2) | 将 set1 中有而 set2 没有的元素给 set3 | set1 = {1,2,3} set2 = {3,4} set3 = set1.difference(set2) set3 {1, 2} |
| difference_update() | set1.difference_update(set2) | 从 set1 中删除与 set2 相同的元素 | set1 = {1,2,3} <br> set2 = {3,4} <br> set1.difference_update(set2) <br> set1 {1, 2} |
| discard() | set1.discard(elem) | 删除 set1 中的 elem 元素 | set1 = {1,2,3} <br> set1.discard(2) <br> set1 {1, 3} <br> set1.discard(4) {1, 3} |
| intersection() | set3 = set1.intersection(set2) | 取 set1 和 set2 的交集给 set3 | set1 = {1,2,3} <br> set2 = {3,4} <br> set3 = set1.intersection(set2) <br> set3 {3} |
| intersection_update() | set1.intersection_update(set2) | 取 set1和 set2 的交集，并更新给 set1 | set1 = {1,2,3} set2 = {3,4} <br> set1.intersection_update(set2) <br> set1 {3} |
| isdisjoint() | set1.isdisjoint(set2) | 判断 set1 和 set2 是否没有交集，有交集返回 False；没有交集返回 True | set1 = {1,2,3} set2 = {3,4} <br> set1.isdisjoint(set2) <br> False |
| issubset() | set1.issubset(set2) | 判断 set1 是否是 set2 的子集 | set1 = {1,2,3} set2 = {1,2} <br> set1.issubset(set2) <br> False |
| issuperset() | set1.issuperset(set2) | 判断 set2 是否是 set1 的子集 | set1 = {1,2,3} set2 = {1,2} <br> set1.issuperset(set2) <br> True |
| pop() | a = set1.pop() | 取 set1 中一个元素，并赋值给 a | set1 = {1,2,3} <br> a = set1.pop() <br> set1 {2,3} a 1 |
| remove() | set1.remove(elem) | 移除 set1 中的 elem 元素 | set1 = {1,2,3} set1.remove(2) <br> set1 {1, 3} <br> set1.remove(4) |
| symmetric_difference() | set3 = set1.symmetric_difference(set2) | 取 set1 和 set2 中互不相同的元素，给 set3 | set1 = {1,2,3} set2 = {3,4} <br> set3 = set1.symmetric_difference(set2) <br> set3 {1, 2, 4} |
| symmetric_difference_update() | set1.symmetric_difference_update(set2) | 取 set1 和 set2 中互不相同的元素，并更新给 set1 | set1 = {1,2,3} set2 = {3,4} <br> set1.symmetric_difference_update(set2) <br> set1 {1, 2, 4} |
| union() | set3 = set1.union(set2) | 取 set1 和 set2 的并集，赋给 set3 | set1 = {1,2,3} set2 = {3,4} <br> set3=set1.union(set2) <br> set3 {1, 2, 3, 4} |
| update() | set1.update(elem) | 添加列表或集合中的元素到 set1 | set1 = {1,2,3} <br> set1.update([3,4]) <br> set1 {1,2,3,4} |

# frozenset集合
`set`集合是可变序列，程序可以改变序列中的元素；`frozenset`集合是不可变序列，程序不能改变序列中的元素。`set`集合中所有能改变集合本身的方法，比如`remove()、discard()、add()`等，`frozenset`都不支持；`set`集合中不改变集合本身的方法，`fronzenset`都支持。

我们可以在交互式编程环境中输入`dir(frozenset)`来查看`frozenset`集合支持的方法：
```
>>> dir(frozenset)
['copy', 'difference', 'intersection', 'isdisjoint', 'issubset', 'issuperset', 'symmetric_difference', 'union']
```
`frozenset`集合的这些方法和`set`集合中同名方法的功能是一样的。

两种情况下可以使用`fronzenset`：
* 当集合的元素不需要改变时，我们可以使用`fronzenset`替代`set`，这样更加安全。
* 有时候程序要求必须是不可变对象，这个时候也要使用`fronzenset`替代`set`。比如，字典（`dict`）的键（`key`）就要求是不可变对象。

```py
s = {'Python', 'C', 'C++'}
fs = frozenset(['Java', 'Shell'])
s_sub = {'PHP', 'C#'}
#向set集合中添加frozenset
s.add(fs)
print('s =', s)
#向为set集合添加子set集合
s.add(s_sub)
print('s =', s)
```
运行结果：
```
s = {'Python', frozenset({'Java', 'Shell'}), 'C', 'C++'}
Traceback (most recent call last):
    File "C:\Users\mozhiyan\Desktop\demo.py", line 11, in <module>
        s.add(s_sub)
TypeError: unhashable type: 'set'
```
需要注意的是，`set`集合本身的元素必须是不可变的，所以`set`的元素不能是`set`，只能是`frozenset`。第 5 行代码向`set`中添加`frozenset`是没问题的，因为`frozenset`是不可变的；但是，第 8 行代码中尝试向`set`中添加子`set`，这是不允许的，因为`set`是可变的。