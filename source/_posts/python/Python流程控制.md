


# if else条件语句
```
if 表达式：
    代码块

if 表达式：
    代码块 1
else：
    代码块 2

if 表达式 1：
    代码块 1
elif 表达式 2：
    代码块 2
elif 表达式 3：
    代码块 3
...//其它elif语句
else：
    代码块 n
```
# if else 如何判断表达式是否成立
`if`和`elif`后面的“表达式”的形式是很自由的，只要表达式有一个结果，不管这个结果是什么类型，Python 都能判断它是“真”还是“假”。

布尔类型（`bool`）只有两个值，分别是`True`和`False`，Python 会把`True`当做“真”，把`False`当做“假”。

对于数字，Python 会把 0 和 0.0 当做“假”，把其它值当做“真”。

对于其它类型，当对象为空或者为`None`时，Python 会把它们当做“假”，其它情况当做真。比如，下面的表达式都是不成立的：
```py
""  #空字符串
[ ]  #空列表
( )  #空元组
{ }  #空字典
None  #空值
```
```py
b = False
if b:
  print('b是True')
else:
  print('b是False')

n = 0
if n:
  print('n不是零值')
else:
  print('n是零值')

s = ""
if s:
  print('s不是空字符串')
else:
  print('s是空字符串')

l = []
if l:
  print('l不是空列表')
else:
  print('l是空列表')

d = {}
if d:
  print('d不是空字典')
else:
  print('d是空字典')

def func():
  print("函数被调用")
if func():
  print('func()返回值不是空')
else:
  print('func()返回值为空')
```
运行结果：
```
b是False
n是零值
s是空字符串
l是空列表
d是空字典
函数被调用
func()返回值为空
```
说明：对于没有`return`语句的函数，返回值为空，也即`None`。
# pass语句
```py
age = int( input("请输入你的年龄：") )
if age < 12 :
    print("婴幼儿")
elif age >= 12 and age < 18:
    print("青少年")
elif age >= 18 and age < 30:
    print("成年人")
elif age >= 30 and age < 50:
    #TODO: 成年人
else:
    print("老年人")
```
当年龄大于等于 30 并且小于 50 时，我们没有使用`print()`语句，而是使用了一个注释，希望以后再处理成年人的情况。当 Python 执行到该`elif`分支时，会跳过注释，什么都不执行。

但是 Python 提供了一种更加专业的做法，就是空语句`pass`。`pass`是 Python 中的关键字，用来让解释器跳过此处，什么都不做。

就像上面的情况，有时候程序需要占一个位置，或者放一条语句，但又不希望这条语句做任何事情，此时就可以通过`pass`语句来实现。使用`pass`语句比使用注释更加优雅。

使用`pass`语句更改上面的代码：
```py
age = int( input("请输入你的年龄：") )
if age < 12 :
  print("婴幼儿")
elif age >= 12 and age < 18:
  print("青少年")
elif age >= 18 and age < 30:
  print("成年人")
elif age >= 30 and age < 50:
  pass
else:
  print("老年人")
```
# assert断言函数
`assert`语句，又称断言语句，可以看做是功能缩小版的`if`语句，它用于判断某个表达式的值，如果值为真，则程序可以继续往下执行；反之，Python 解释器会报`AssertionError`错误。
```
assert 表达式
```
`assert`语句的执行流程可以用`if`判断语句表示：
```
if 表达式==True:
  程序继续执行
else:
  程序报 AssertionError 错误
```
明明 assert 会令程序崩溃，为什么还要使用它呢？这是因为，与其让程序在晚些时候崩溃，不如在错误条件出现时，就直接让程序崩溃，这有利于我们对程序排错，提高程序的健壮性。

因此，`assert`语句通常用于检查用户的输入是否符合规定，还经常用作程序初期测试和调试过程中的辅助工具。
```py
mathmark = int(input())
#断言数学考试分数是否位于正常范围内
assert 0 <= mathmark <= 100
#只有当 mathmark 位于 [0,100]范围内，程序才会继续执行
print("数学考试分数为：", mathmark)
```
运行该程序，测试数据如下：
```
90
数学考试分数为： 90
```
再次执行该程序，测试数据为：
```
159
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\file.py", line 3, in <module>
    assert 0 <= mathmark <= 100
AssertionError
```
可以看到，当`assert`语句后的表达式值为真时，程序继续执行；反之，程序停止执行，并报`AssertionError`错误。
# while循环语句
```
while 条件表达式：
  代码块
```
# for循环
```
for 迭代变量 in 字符串|列表|元组|字典|集合：
  代码块
```
```py
print("计算 1+2+...+100 的结果为：")
#保存累加结果的变量
result = 0
#逐个获取从 1 到 100 这些值，并做累加操作
for i in range(101):
    result += i
print(result)
```
```py
my_list = [1,2,3,4,5]
for ele in my_list:
  print('ele =', ele)
```
程序执行结果为：
```
ele = 1
ele = 2
ele = 3
ele = 4
ele = 5
```
# 循环结构中else用法
Python 中，无论是`while`循环还是`for`循环，其后都可以紧跟着一个`else`代码块，它的作用是当循环条件为`False`跳出循环时，程序会最先执行`else`代码块中的代码。
```py
add = "python"
i = 0
while i < len(add):
  print(add[i],end="")
  i = i + 1
else:
  print("\n执行 else 代码块")
```
程序执行结果为：
```
python
执行 else 代码块
```
上面程序中，当`i==len(add)`结束循环时（确切的说，是在结束循环之前），Python 解释器会执行`while`循环后的`else`代码块。

修改上面程序，去掉`else`代码块：
```py
add = "python"
i = 0
while i < len(add):
  print(add[i],end="")
  i = i + 1
#原本位于 else 代码块中的代码
print("\n执行 else 代码块")
```
程序执行结果为：
```
python
执行 else 代码块
```
那么，`else`代码块真的没有用吗？当然不是。

当然，我们也可以为`for`循环添加一个`else`代码块：
```py
add = "python"
for i in  add:
  print(i,end="")
else:
  print("\n执行 else 代码块")
```
程序执行结果为：
```
python
执行 else 代码块
```
# break
我们知道，在执行`while`循环或者`for`循环时，只要循环条件满足，程序将会一直执行循环体，不停地转圈。但在某些场景，我们可能希望在循环结束前就强制结束循环，Python 提供了 2 种强制离开当前循环体的办法：
* 使用`continue`语句，可以跳过执行本次循环体中剩余的代码，转而执行下一次的循环。
* 使用`break`语句，可以完全终止当前循环。

`break`语句可以立即终止当前循环的执行，跳出当前所在的循环结构。无论是`while`循环还是`for`循环，只要执行`break`语句，就会直接结束当前正在执行的循环体。

`break`语句的语法非常简单，只需要在相应`while`或`for`语句中直接加入即可。
```py
add = "python,shell"
# 一个简单的for循环
for i in add:
  if i == ',' :
    #终止循环
    break
  print(i,end="")
print("\n执行循环体外的代码")
```
运行结果为：
```
python
执行循环体外的代码
```
分析上面程序不难看出，当循环至`add`字符串中的逗号（`,`）时，程序执行`break`语句，其会直接终止当前的循环，跳出循环体。

`break`语句一般会结合`if`语句进行搭配使用，表示在某种条件下跳出循环体。

注意，`for`循环后也可以配备一个`else`语句。这种情况下，如果使用`break`语句跳出循环体，不会执行`else`中包含的代码。
```py
add = "python,shell"
for i in add:
  if i == ',' :
    #终止循环
    break
  print(i,end="")
else:
    print("执行 else 语句中的代码")
print("\n执行循环体外的代码")
```
程序执行结果为：
```
python
执行循环体外的代码
```
从输出结果可以看出，使用`break`跳出当前循环体之后，该循环后的`else`代码块也不会被执行。但是，如果将`else`代码块中的代码直接放在循环体的后面，则该部分代码将会被执行。

另外，对于嵌套的循环结构来说，`break`语句只会终止所在循环体的执行，而不会作用于所有的循环体。
```py
add = "python,shell"
for i in range(3):
  for j in add:
    if j == ',':
      break   
    print(j,end="")
  print("\n跳出内循环")
```
程序执行结果为：
```
python
跳出内循环
python
跳出内循环
python
跳出内循环
```
分析上面程序，每当执行内层循环时，只要循环至`add`字符串中的逗号（`,`）就会执行`break`语句，它会立即停止执行当前所在的内存循环体，转而继续执行外层循环。

在嵌套循环结构中，如何同时跳出内层循环和外层循环呢？最简单的方法就是借用一个`bool`类型的变量。
```py
add = "python,shell"
#提前定义一个 bool 变量，并为其赋初值
flag = False
for i in range(3):
  for j in add:
    if j == ',':
      #在 break 前，修改 flag 的值
      flag = True
      break   
    print(j,end="")
  print("\n跳出内循环")
  #在外层循环体中再次使用 break
  if flag == True:
    print("跳出外层循环")
    break
```
可以看到，通过借助一个`bool`类型的变量`flag`，在跳出内循环时更改`flag`的值，同时在外层循环体中，判断`flag`的值是否发生改动，如有改动，则再次执行`break`跳出外层循环；反之，则继续执行外层循环。

因此，上面程序的执行结果为：
```
python
跳出内循环
跳出外层循环
```
当然，这里仅跳出了 2 层嵌套循环，此方法支持跳出多层嵌套循环。
# continue
和`break`语句相比，`continue`语句的作用则没有那么强大，它只会终止执行本次循环中剩下的代码，直接从下一次循环继续执行。

`continue`语句的用法和`break`语句一样，只要`while`或`for`语句中的相应位置加入即可。
```py
add = "python,shell"
# 一个简单的for循环
for i in add:
  if i == ',' :
    # 忽略本次循环的剩下语句
    print('\n')
    continue
  print(i,end="")
```
运行结果：
```
python
shell
```
可以看到，当遍历 add 字符串至逗号（ , ）时，会进入 if 判断语句执行 print() 语句和 continue 语句。其中，print() 语句起到换行的作用，而 continue 语句会使 Python 解释器忽略执行第 8 行代码，直接从下一次循环开始执行。
# zip函数
`zip()`函数是 Python 内置函数之一，它可以将多个序列（列表、元组、字典、集合、字符串以及`range()`区间构成的列表）“压缩”成一个`zip`对象。所谓“压缩”，其实就是将这些序列中对应位置的元素重新组合，生成一个个新的元组。
```
zip(iterable, ...)
```
其中`iterable,...`表示多个列表、元组、字典、集合、字符串，甚至还可以为`range()`区间。
```py
my_list = [11,12,13]
my_tuple = (21,22,23)
print([x for x in zip(my_list,my_tuple)])
my_dic = {31:2,32:4,33:5}
my_set = {41,42,43,44}
print([x for x in zip(my_dic)])
my_pychar = "python"
my_shechar = "shell"
print([x for x in zip(my_pychar,my_shechar)])
```
程序执行结果为：
```
[(11, 21), (12, 22), (13, 23)]
[(31,), (32,), (33,)]
[('p', 's'), ('y', 'h'), ('t', 'e'), ('h', 'l'), ('o', 'l')]
```
在使用`zip()`函数“压缩”多个序列时，它会分别取各序列中第 1 个元素、第 2 个元素、... 第`n`个元素，各自组成新的元组。需要注意的是，当多个序列中元素个数不一致时，会以最短的序列为准进行压缩。

另外，对于`zip()`函数返回的`zip`对象，既可以像上面程序那样，通过遍历提取其存储的元组，也可以向下面程序这样，通过调用`list()`函数将`zip()`对象强制转换成列表：
```py
my_list = [11,12,13]
my_tuple = (21,22,23)
print(list(zip(my_list,my_tuple))) # [(11, 21), (12, 22), (13, 23)]
```
# reversed函数
`reserved()`是 Pyton 内置函数之一，其功能是对于给定的序列（包括列表、元组、字符串以及`range(n)`区间），该函数可以返回一个逆序序列的迭代器（用于遍历该逆序序列）。
```
reversed(seq)
```
其中，`seq`可以是列表，元素，字符串以及`range()`生成的区间列表。
```py
#将列表进行逆序
print([x for x in reversed([1,2,3,4,5])]) # [5, 4, 3, 2, 1]
#将元组进行逆序
print([x for x in reversed((1,2,3,4,5))]) # [5, 4, 3, 2, 1]
#将字符串进行逆序
print([x for x in reversed("abcdefg")]) # ['g', 'f', 'e', 'd', 'c', 'b', 'a']
#将 range() 生成的区间列表进行逆序
print([x for x in reversed(range(10))]) # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```
除了使用列表推导式的方式，还可以使用`list()`函数，将`reversed()`函数逆序返回的迭代器，直接转换成列表。
```py
#将列表进行逆序
print(list(reversed([1,2,3,4,5]))) # [5, 4, 3, 2, 1]
```

再次强调，使用`reversed()`函数进行逆序操作，并不会修改原来序列中元素的顺序：
```py
a = [1,2,3,4,5]
#将列表进行逆序
print(list(reversed(a))) # [5, 4, 3, 2, 1]
print("a=",a) # a=[1, 2, 3, 4, 5]
```
# sorted函数
`sorted()`作为 Python 内置函数之一，其功能是对序列（列表、元组、字典、集合、还包括字符串）进行排序。
```py
list = sorted(iterable, key=None, reverse=False)  
```
其中，`iterable`表示指定的序列，`key`参数可以自定义排序规则；`reverse`参数指定以升序（`False`，默认）还是降序（`True`）进行排序。`sorted()`函数会返回一个排好序的列表。

注意，`key`参数和`reverse`参数是可选参数，即可以使用，也可以忽略。
```py
#对列表进行排序
a = [5,3,4,2,1]
print(sorted(a)) # [1, 2, 3, 4, 5]
#对元组进行排序
a = (5,4,3,1,2)
print(sorted(a)) # [1, 2, 3, 4, 5]
#字典默认按照key进行排序
a = {4:1, 5:2, 3:3, 2:6, 1:8}
print(sorted(a.items())) # [(1, 8), (2, 6), (3, 3), (4, 1), (5, 2)]
#对集合进行排序
a = {1,5,3,2,4}
print(sorted(a)) # [1, 2, 3, 4, 5]
#对字符串进行排序
a = "51423"
print(sorted(a)) # ['1', '2', '3', '4', '5']
```
再次强调，使用`sorted()`函数对序列进行排序，并不会在原序列的基础进行修改，而是会重新生成一个排好序的列表。
```py
#对列表进行排序
a = [5,3,4,2,1]
print(sorted(a)) # [1, 2, 3, 4, 5]
#再次输出原来的列表 a
print(a) # [5, 3, 4, 2, 1]
```
显然，`sorted()`函数不会改变所传入的序列，而是返回一个新的、排序好的列表。

除此之外，`sorted()`函数默认对序列中元素进行升序排序，通过手动将其`reverse`参数值改为`True`，可实现降序排序。
```py
#对列表进行排序
a = [5,3,4,2,1]
print(sorted(a, reverse=True)) # [5, 4, 3, 2, 1]
```
另外在调用`sorted()`函数时，还可传入一个`key`参数，它可以接受一个函数，该函数的功能是指定`sorted()`函数按照什么标准进行排序。
```py
chars=['python', 'shell', 'java', 'golang']
#默认排序
print(sorted(chars)) # ['golang', 'java', 'python', 'shell']
#自定义按照字符串长度排序
print(sorted(chars,key=lambda x:len(x))) # ['java', 'shell', 'python', 'golang']
```
