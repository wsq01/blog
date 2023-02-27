---
title: Python函数和lambda表达式
date: 2022-12-13 12:08:52
tags: [python]
categories: python
---


函数的本质就是一段有特定功能、可以重复使用的代码，这段代码已经被提前编写好了，并且为其起一个“好听”的名字。在后续编写程序过程中，如果需要同样的功能，直接通过起好的名字就可以调用这段代码。
# 函数的定义
定义函数需要用`def`关键字实现：
```py
def 函数名(参数列表):
  # 实现特定功能的多行代码
  [return [返回值]]
```
其中，用`[]`括起来的为可选择部分，即可以使用，也可以省略。

各部分参数的含义：
* 函数名：一个符合 Python 语法的标识符。
* 形参列表：设置该函数可以接收多少个参数，多个参数之间用逗号（`,`）分隔。
* `[return [返回值] ]`：整体作为函数的可选参参数，用于设置该函数的返回值。也就是说，一个函数，可以用返回值，也可以没有返回值，是否需要根据实际情况而定。

注意，在创建函数时，即使函数不需要参数，也必须保留一对空的`()`，否则 Python 解释器将提示`invaild syntax`错误。另外，如果想定义一个没有任何功能的空函数，可以使用`pass`语句作为占位符。
```py
#定义个空函数，没有实际意义
def pass_dis():
  pass
#定义一个比较字符串大小的函数
def str_max(str1,str2):
  str = str1 if str1 > str2 else str2
  return str
```
函数中的`return`语句可以直接返回一个表达式的值，例如修改上面的`str_max()`函数：
```py
def str_max(str1,str2):
  return str1 if str1 > str2 else str2
```
该函数的功能，和上面的`str_max()`函数是完全一样的，只是省略了创建`str`变量，因此函数代码更加简洁。
# 函数的调用
调用函数也就是执行函数。
```
[返回值] = 函数名([形参值])
```
其中，函数名即指的是要调用的函数的名称；形参值指的是当初创建函数时要求传入的各个形参的值。如果该函数有返回值，我们可以通过一个变量来接收该值，当然也可以不接受。

需要注意的是，创建函数有多少个形参，那么调用时就需要传入多少个值，且顺序必须和创建函数时一致。即便该函数没有参数，函数名后的小括号也不能省略。
```py
pass_dis()
strmax = str_max("python","shell")
print(strmax) # shell
```
首先，对于调用空函数来说，由于函数本身并不包含任何有价值的执行代码，也没有返回值，所以调用空函数不会有任何效果。

其次，对于上面程序中调用`str_max()`函数，由于当初定义该函数为其设置了 2 个参数，因此这里在调用该参数，就必须传入 2 个参数。同时，由于该函数内部还使用了`return`语句，因此我们可以使用`strmax`变量来接收该函数的返回值。
# 为函数提供说明文档
通过调用 Python 的`help()`内置函数或者`__doc__`属性，我们可以查看某个函数的使用说明文档。事实上，无论是 Python 提供给我们的函数，还是自定义的函数，其说明文档都需要设计该函数的程序员自己编写。

其实，函数的说明文档，本质就是一段字符串，只不过作为说明文档，字符串的放置位置是有讲究的，函数的说明文档通常位于函数内部、所有代码的最前面。
```py
#定义一个比较字符串大小的函数
def str_max(str1,str2):
  '''
  比较 2 个字符串的大小
  '''
  str = str1 if str1 > str2 else str2
  return str
help(str_max)
#print(str_max.__doc__)
```
程序执行结果为：
```
Help on function str_max in module __main__:

str_max(str1, str2)
  比较 2 个字符串的大小
```
上面程序中，还可以使用`__doc__`属性来获取`str_max()`函数的说明文档，即使用最后一行的输出语句，其输出结果为：
``` 
  比较 2 个字符串的大小
```
# 函数值传递和引用传递
通常情况下，定义函数时都会选择有参数的函数形式，函数参数的作用是传递数据给函数，令其对接收的数据做具体的操作处理。

在使用函数时，经常会用到形式参数（简称“形参”）和实际参数（简称“实参”），二者都叫参数，之间的区别是：
* 形式参数：在定义函数时，函数名后面括号中的参数就是形式参数:
```py
#定义函数时，这里的函数参数 obj 就是形式参数
def demo(obj):
  print(obj)
```
* 实际参数：在调用函数时，函数名后面括号中的参数称为实际参数，也就是函数的调用者给函数的参数。
```py
a = "测试"
#调用已经定义好的 demo 函数，此时传入的函数参数 a 就是实际参数
demo(a)
```
根据实际参数的类型不同，函数参数的传递方式可分为 2 种，分别为值传递和引用（地址）传递：
* 值传递：适用于实参类型为不可变类型（字符串、数字、元组）；
* 引用（地址）传递：适用于实参类型为可变类型（列表，字典）；

值传递和引用传递的区别是，函数参数进行值传递后，若形参的值发生改变，不会影响实参的值；而函数参数继续引用传递后，改变形参的值，实参的值也会一同改变。

例如，定义一个名为`demo`的函数，分别为传入一个字符串类型的变量（代表值传递）和列表类型的变量（代表引用传递）：
```py
def demo(obj) :
  obj += obj
  print("形参值为：",obj)
print("-------值传递-----")
a = "测试"
print("a的值为：",a)
demo(a)
print("实参值为：",a)
print("-----引用传递-----")
a = [1,2,3]
print("a的值为：",a)
demo(a)
print("实参值为：",a)
```
运行结果为：
```
-------值传递-----
a的值为： 测试
形参值为： 测试测试
实参值为： 测试
-----引用传递-----
a的值为： [1, 2, 3]
形参值为： [1, 2, 3, 1, 2, 3]
实参值为： [1, 2, 3, 1, 2, 3]
```
分析运行结果不难看出，在执行值传递时，改变形式参数的值，实际参数并不会发生改变；而在进行引用传递时，改变形式参数的值，实际参数也会发生同样的改变。
# 位置参数
位置参数，有时也称必备参数，指的是必须按照正确的顺序将实际参数传到函数中，换句话说，调用函数时传入实际参数的数量和位置都必须和定义函数时保持一致。
## 实参和形参数量必须一致
在调用函数，指定的实际参数的数量，必须和形式参数的数量一致（传多传少都不行），否则 Python 解释器会抛出`TypeError`异常，并提示缺少必要的位置参数。
```py
def girth(width , height):
  return 2 * (width + height)
#调用函数时，必须传递 2 个参数，否则会引发错误
print(girth(3))
'''
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 4, in <module>
    print(girth(3))
TypeError: girth() missing 1 required positional argument: 'height'
'''
```
同样，多传参数也会抛出异常：
```python
def girth(width , height):
  return 2 * (width + height)
#调用函数时，必须传递 2 个参数，否则会引发错误
print(girth(3,2,4))
'''
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 4, in <module>
    print(girth(3,2,4))
TypeError: girth() takes 2 positional arguments but 3 were given
'''
```
## 实参和形参位置必须一致
在调用函数时，传入实际参数的位置必须和形式参数位置一一对应，否则会产生以下 2 种结果：
### 1. 抛出`TypeError`异常
当实际参数类型和形式参数类型不一致，并且在函数中，这两种类型之间不能正常转换，此时就会抛出`TypeError`异常。
```py
def area(height,width):
  return height * width / 2
print(area("测试", 3))
'''
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 3, in <module>
    print(area("测试",3))
  File "C:\Users\mengma\Desktop\1.py", line 2, in area
    return height*width/2
TypeError: unsupported operand type(s) for /: 'str' and 'int'
'''
```
以上显示的异常信息，就是因为字符串类型和整形数值做除法运算。
### 2. 产生的结果和预期不符
调用函数时，如果指定的实际参数和形式参数的位置不一致，但它们的数据类型相同，那么程序将不会抛出异常，只不过导致运行结果和预期不符。

例如，设计一个求梯形面积的函数，并利用此函数求上底为 4cm，下底为 3cm，高为 5cm 的梯形的面积。但如果交互高和下低参数的传入位置，计算结果将导致错误：
```py
def area(upper_base, lower_bottom, height):
  return (upper_base + lower_bottom) * height / 2
print("正确结果为：",area(4, 3, 5)) # 正确结果为： 17.5
print("错误结果为：",area(4, 5, 3)) # 错误结果为： 13.5
```
# 关键字参数
关键字参数是指使用形式参数的名字来确定输入的参数值。通过此方式指定函数实参时，不再需要与形参的位置完全一致，只要将参数名写正确即可。
```py
def dis_str(str1, str2):
  print("str1:", str1)
  print("str2:", str2)
#位置参数
dis_str("python", "shell")
#关键字参数
dis_str("python", str2="shell")
dis_str(str2="python", str1="shell")
```
程序执行结果为：
```
str1: python
str2: shell
str1: python
str2: shell
str1: shell
str2: python
```
可以看到，在调用有参函数时，既可以根据位置参数来调用，也可以使用关键字参数（程序中第 8 行）来调用。在使用关键字参数调用时，可以任意调换参数传参的位置。

当然，还可以像第 7 行代码这样，使用位置参数和关键字参数混合传参的方式。但需要注意，混合传参时关键字参数必须位于所有的位置参数之后。
```py
# 位置参数必须放在关键字参数之前，下面代码错误
dis_str(str1="python", "shell")
```
Python 解释器会报如下错误：
```
SyntaxError: positional argument follows keyword argument
```
# 默认参数
在调用函数时如果不指定某个参数，Python 解释器会抛出异常。为了解决这个问题，Python 允许为参数设置默认值，即在定义函数时，直接给形式参数指定一个默认值。这样的话，即便调用函数时没有给拥有默认值的形参传递参数，该参数可以直接使用定义函数时设置的默认值。
```
def 函数名(..., 形参名，形参名=默认值)：
  代码块
```
注意，在使用此格式定义函数时，指定有默认值的形式参数必须在所有没默认值参数的最后，否则会产生语法错误。
```py
#str1没有默认参数，str2有默认参数
def dis_str(str1, str2 = "python"):
  print("str1:",str1)
  print("str2:",str2)
dis_str("shell")
dis_str("java","golang")
```
运行结果为：
```
str1: shell
str2: python
str1: java
str2: golang
```
当然在调用`dis_str()`函数时，也可以给所有的参数传值，这时即便`str2`有默认值，它也会优先使用传递给它的新值。

同时，结合关键字参数，以下 3 种调用`dis_str()`函数的方式也是可以的：
```py
dis_str(str1 = "shell")
dis_str("java", str2 = "golang")
dis_str(str1 = "java", str2 = "golang")
```
再次强调，当定义一个有默认值参数的函数时，有默认值的参数必须位于所有没默认值参数的后面。因此，下面例子中定义的函数是不正确的：
```py
#语法错误
def dis_str(str1="python", str2, str3):
  pass
```
显然，`str1`设有默认值，而`str2`和`str3`没有默认值，因此`str1`必须位于`str2`和`str3`之后。

对于自己自定义的函数，可以轻易知道哪个参数有默认值，但如果使用 Python 提供的内置函数，又或者其它第三方提供的函数，怎么知道哪些参数有默认值呢？

Pyhton 中，可以使用`函数名.__defaults__`查看函数的默认值参数的当前值，其返回值是一个元组。
```
print(dis_str.__defaults__)
```
程序执行结果为：
```
('python',)
```
# None（空值）
在 Python 中，有一个特殊的常量`None`。和`False`不同，它不表示 0，也不表示空字符串，而表示没有值，也就是空值。

这里的空值并不代表空对象，即`None`和`[]、''`不同：
```py
>>> None is []
False
>>> None is ""
False
```
`None`有自己的数据类型，我们可以在 IDLE 中使用`type()`函数查看它的类型：
```py
type(None)
<class 'NoneType'>
```
可以看到，它属于`NoneType`类型。

需要注意的是，`None`是`NoneType`数据类型的唯一值，也就是说，我们不能再创建其它`NoneType`类型的变量，但是可以将`None`赋值给任何变量。如果希望变量中存储的东西不与任何其它值混淆，就可以使用`None`。

除此之外，`None`常用于`assert`、判断以及函数无返回值的情况。`print()`函数返回值就是`None`。因为它的功能是在屏幕上显示文本，根本不需要返回任何值，所以`print()`就返回`None`。
```py
>>> spam = print('Hello!')
Hello!
>>> None == spam
True
```
另外，对于所有没有`return`语句的函数定义，Python 都会在末尾加上`return None`，使用不带值的`return`语句（也就是只有`return`关键字本身），那么就返回`None`。
# return函数返回值
用`def`语句创建函数时，可以用`return`语句指定应该返回的值，该返回值可以是任意类型。需要注意的是，`return`语句在同一函数中可以出现多次，但只要有一个得到执行，就会直接结束函数的执行。
```
return [返回值]
```
其中，返回值参数可以指定，也可以省略不写（将返回空值`None`）。
```py
def add(a, b):
  c = a + b
  return c
#函数赋值给变量
c = add(3, 4)
print(c) # 7
#函数返回值作为其他函数的实际参数
print(add(3, 4)) # 7
```
本例中，`add()`函数既可以用来计算两个数的和，也可以连接两个字符串，它会返回计算的结果。

通过`return`语句指定返回值后，我们在调用函数时，既可以将该函数赋值给一个变量，用变量保存函数的返回值，也可以将函数再作为某个函数的实际参数。
```py
def isGreater0(x):
  if x > 0:
    return True
  else:
    return False
print(isGreater0(5)) # True
print(isGreater0(0)) # False
```
可以看到，函数中可以同时包含多个`return`语句，但需要注意的是，最终真正执行的做多只有 1 个，且一旦执行，函数运行会立即结束。

以上实例中，我们通过`return`语句，都仅返回了一个值，但其实通过`return`语句，可以返回多个值。
# 变量作用域
所谓作用域，就是变量的有效范围，就是变量可以在哪个范围以内使用。有些变量可以在整段代码的任意位置使用，有些变量只能在函数内部使用，有些变量只能在`for`循环内部使用。

变量的作用域由变量的定义位置决定，在不同位置定义的变量，它的作用域是不一样的。
## 局部变量
在函数内部定义的变量，它的作用域也仅限于函数内部，出了函数就不能使用了，这样的变量称为局部变量。

当函数被执行时，Python 会为其分配一块临时的存储空间，所有在函数内部定义的变量，都会存储在这块空间中。而在函数执行完毕后，这块临时存储空间随即会被释放并回收，该空间中存储的变量自然也就无法再被使用。
```py
def demo():
  add = "python"
  print("函数内部 add =",add)
demo()
print("函数外部 add =",add)
```
程序执行结果为：
```
函数内部 add = python
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\file.py", line 6, in <module>
    print("函数外部 add =",add)
NameError: name 'add' is not defined
```
可以看到，如果试图在函数外部访问其内部定义的变量，Python 解释器会报`NameError`错误，并提示我们没有定义要访问的变量，这也证实了当函数执行完毕后，其内部定义的变量会被销毁并回收。

值得一提的是，函数的参数也属于局部变量，只能在函数内部使用。
```py
def demo(name,add):
  print("函数内部 name =",name)
  print("函数内部 add =",add)
demo("Python教程","python")
print("函数外部 name =",name)
print("函数外部 add =",add)
```
程序执行结果为：
```
函数内部 name = Python教程
函数内部 add = python
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\file.py", line 7, in <module>
    print("函数外部 name =",name)
NameError: name 'name' is not defined
```
由于 Python 解释器是逐行运行程序代码，由此这里仅提示给我“name 没有定义”，实际上在函数外部访问`add`变量也会报同样的错误。
## 全局变量
除了在函数内部定义变量，Python 还允许在所有函数的外部定义变量，这样的变量称为全局变量。

和局部变量不同，全局变量的默认作用域是整个程序，即全局变量既可以在各个函数的外部使用，也可以在各函数内部使用。

定义全局变量的方式有以下 2 种：
* 在函数体外定义的变量，一定是全局变量：
```py
add = "shell"
def text():
  print("函数体内访问：",add)
text()
print('函数体外访问：',add)
```
运行结果为：
```
函数体内访问： shell
函数体外访问： shell
```
* 在函数体内定义全局变量。即使用`global`关键字对变量进行修饰后，该变量就会变为全局变量。
```py
def text():
  global add
  add= "java"
  print("函数体内访问：", add)
text()
print('函数体外访问：', add)
```
运行结果为：
```
函数体内访问： java
函数体外访问： java
```

注意，在使用`global`关键字修饰变量名时，不能直接给变量赋初值，否则会引发语法错误。
## 获取指定作用域范围中的变量
在一些特定场景中，我们可能需要获取某个作用域内（全局范围内或者局部范围内）所有的变量，Python 提供了以下 3 种方式：
#### 1) globals()函数
`globals()`函数为 Python 的内置函数，它可以返回一个包含全局范围内所有变量的字典，该字典中的每个键值对，键为变量名，值为该变量的值。

举个例子：
```py
#全局变量
Pyname = "Python教程"
Pyadd = "python"
def text():
    #局部变量
    Shename = "shell教程"
    Sheadd= "shell"
print(globals())
```
程序执行结果为：
```
{ ...... , 'Pyname': 'Python教程', 'Pyadd': 'python', ......}
```
注意，`globals()`函数返回的字典中，会默认包含有很多变量，这些都是 Python 主程序内置的。

可以看到，通过调用`globals()`函数，我们可以得到一个包含所有全局变量的字典。并且，通过该字典，我们还可以访问指定变量，甚至如果需要，还可以修改它的值。例如，在上面程序的基础上，添加如下语句：
```python
print(globals()['Pyname'])
globals()['Pyname'] = "Python入门教程"
print(Pyname)
```
程序执行结果为：
```
Python教程
Python入门教程
```
#### 2) locals()函数
`locals()`函数也是 Python 内置函数之一，通过调用该函数，我们可以得到一个包含当前作用域内所有变量的字典。这里所谓的“当前作用域”指的是，在函数内部调用`locals()`函数，会获得包含所有局部变量的字典；而在全局范文内调用`locals()`函数，其功能和`globals()`函数相同。

举个例子：
```py
#全局变量
Pyname = "Python教程"
Pyadd = "python"
def text():
  #局部变量
  Shename = "shell教程"
  Sheadd= "shell"
  print("函数内部的 locals:")
  print(locals())
text()
print("函数外部的 locals:")
print(locals())
程序执行结果为：
函数内部的 locals:
{'Sheadd': 'shell', 'Shename': 'shell教程'}
函数外部的 locals:
{...... , 'Pyname': 'Python教程', 'Pyadd': 'python', ...... }
```
当使用`locals()`函数获取所有全局变量时，和`globals()`函数一样，其返回的字典中会默认包含有很多变量，这些都是 Python 主程序内置的，读者暂时不用理会它们。

注意，当使用`locals()`函数获得所有局部变量组成的字典时，可以向`globals()`函数那样，通过指定键访问对应的变量值，但无法对变量值做修改。例如：
```py
#全局变量
Pyname = "Python教程"
Pyadd = "http://c.biancheng.net/python/"
def text():
  #局部变量
  Shename = "shell教程"
  Sheadd= "http://c.biancheng.net/shell/"
  print(locals()['Shename'])
  locals()['Shename'] = "shell入门教程"
  print(Shename)
text()
```
程序执行结果为：
```
shell教程
shell教程
```
显然，`locals()`返回的局部变量组成的字典，可以用来访问变量，但无法修改变量的值。
#### 3) vars(object)
`vars()`函数也是 Python 内置函数，其功能是返回一个指定`object`对象范围内所有变量组成的字典。如果不传入`object`参数，`vars()`和`locals()`的作用完全相同。
```py
 #全局变量
Pyname = "Python教程"
Pyadd = "python"
class Demo:
    name = "Python 教程"
    add = "python"
print("有 object：")
print(vars(Demo))
print("无 object：")
print(vars())
程序执行结果为：
有 object：
{...... , 'name': 'Python 教程', 'add': 'python', ......}
无 object：
{...... , 'Pyname': 'Python教程', 'Pyadd': 'python', ...... }
```
# 局部函数
Python 支持在函数内部定义函数，此类函数又称为局部函数。

和局部变量一样，默认情况下局部函数只能在其所在函数的作用域内使用。
```py
#全局函数
def outdef ():
  #局部函数
  def indef():
    print("test")
  #调用局部函数
  indef()
#调用全局函数
outdef()
```
就如同全局函数返回其局部变量，就可以扩大该变量的作用域一样，通过将局部函数作为所在函数的返回值，也可以扩大局部函数的使用范围。例如，修改上面程序为：
```py
#全局函数
def outdef ():
  #局部函数
  def indef():
    print("调用局部函数")
  #调用局部函数
  return indef
#调用全局函数
new_indef = outdef()
# 调用全局函数中的局部函数
new_indef()
```
因此，对于局部函数的作用域，可以总结为：如果所在函数没有返回局部函数，则局部函数的可用范围仅限于所在函数内部；反之，如果所在函数将局部函数作为返回值，则局部函数的作用域就会扩大，既可以在所在函数内部使用，也可以在所在函数的作用域中使用。

以上面程序中的`outdef()`和`indef()`为例，如果`outdef()`不将`indef`作为返回值，则`indef()`只能在`outdef()`函数内部使用；反之，则`indef()`函数既可以在`outdef()`函数内部使用，也可以在`outdef()`函数的作用域，也就是全局范围内使用。

另外值得一提的是，如果局部函数中定义有和所在函数中变量同名的变量，也会发生“遮蔽”的问题。
```py
#全局函数
def outdef ():
  name = "所在函数中定义的 name 变量"
  #局部函数
  def indef():
    print(name)
    name = "局部函数中定义的 name 变量"
  indef()
#调用全局函数
outdef()
```
执行此程序，Python 解释器会报如下错误：
```
UnboundLocalError: local variable 'name' referenced before assignment
```
此错误直译过来的意思是“局部变量 name 还没定义就使用”。导致该错误的原因就在于，局部函数`indef()`中定义的`name`变量遮蔽了所在函数`outdef()`中定义的`name`变量。再加上，`indef()`函数中`name`变量的定义位于`print()`输出语句之后，导致`print(name)`语句在执行时找不到定义的`name`变量，因此程序报错。

由于这里的`name`变量也是局部变量，因此`globals()`函数或者`globals`关键字，并不适用于解决此问题。这里可以使用 Python 提供的`nonlocal`关键字。
```py
#全局函数
def outdef ():
  name = "所在函数中定义的 name 变量"
  #局部函数
  def indef():
    nonlocal name
    print(name)
    #修改name变量的值
    name = "局部函数中定义的 name 变量"
  indef()
#调用全局函数
outdef()
```
程序执行结果为：
```
所在函数中定义的 name 变量
```
# 闭包
闭包，又称闭包函数或者闭合函数，其实和前面讲的嵌套函数类似，不同之处在于，闭包中外部函数返回的不是一个具体的值，而是一个函数。一般情况下，返回的函数会赋值给一个变量，这个变量可以在后面被继续执行调用。

例如，计算一个数的`n`次幂，用闭包可以写成下面的代码：
```py
#闭包函数，其中 exponent 称为自由变量
def nth_power(exponent):
  def exponent_of(base):
    return base ** exponent
  return exponent_of # 返回值是 exponent_of 函数
square = nth_power(2) # 计算一个数的平方
cube = nth_power(3) # 计算一个数的立方
print(square(2))  # 计算 2 的平方
print(cube(2)) # 计算 2 的立方
```
运行结果为：
```
4
8
```
在上面程序中，外部函数`nth_power()`的返回值是函数`exponent_of()`，而不是一个具体的数值。

需要注意的是，在执行完`square = nth_power(2)`和`cube = nth_power(3)`后，外部函数`nth_power()`的参数`exponent`会和内部函数`exponent_of`一起赋值给`squre`和`cube`，这样在之后调用`square(2)`或者`cube(2)`时，程序就能顺利地输出结果，而不会报错说参数`exponent`没有定义。

为什么要闭包呢？上面的程序，完全可以写成下面的形式：
```py
def nth_power_rewrite(base, exponent):
  return base ** exponent
```
上面程序确实可以实现相同的功能，不过使用闭包，可以让程序变得更简洁易读。设想一下，比如需要计算很多个数的平方，那么写成下面哪一种形式更好呢？
```py
# 不使用闭包
res1 = nth_power_rewrite(base1, 2)
res2 = nth_power_rewrite(base2, 2)
res3 = nth_power_rewrite(base3, 2)
# 使用闭包
square = nth_power(2)
res1 = square(base1)
res2 = square(base2)
res3 = square(base3)
```
显然第二种方式表达更为简洁，在每次调用函数时，都可以少输入一个参数。

其次，和缩减嵌套函数的优点类似，函数开头需要做一些额外工作，当需要多次调用该函数时，如果将那些额外工作的代码放在外部函数，就可以减少多次调用导致的不必要开销，提高程序的运行效率。
## Python闭包的__closure__属性
闭包比普通的函数多了一个`__closure__`属性，该属性记录着自由变量的地址。当闭包被调用时，系统就会根据该地址找到对应的自由变量，完成整体的函数调用。

以`nth_power()`为例，当其被调用时，可以通过`__closure__`属性获取自由变量（也就是程序中的`exponent`参数）存储的地址：
```py
def nth_power(exponent):
  def exponent_of(base):
    return base ** exponent
  return exponent_of
square = nth_power(2)
#查看 __closure__ 的值
print(square.__closure__)
```
输出结果为：
```
(<cell at 0x0000014454DFA948: int object at 0x00000000513CC6D0>,)
```
可以看到，显示的内容是一个`int`整数类型，这就是`square`中自由变量`exponent`的初始值。还可以看到，`__closure__`属性的类型是一个元组，这表明闭包可以支持多个自由变量的形式。
# lambda表达式（匿名函数）
对于定义一个简单的函数，Python 还提供了另外一种方法，即 lambda 表达式。

lambda 表达式，又称匿名函数，常用来表示内部仅包含 1 行表达式的函数。如果一个函数的函数体仅有 1 行表达式，则该函数就可以用 lambda 表达式来代替。
```
name = lambda [list] : 表达式
```
其中，定义 lambda 表达式，必须使用 lambda 关键字；`[list]`作为可选参数，等同于定义函数是指定的参数列表；`value`为该表达式的名称。

该语法格式转换成普通函数的形式，如下所示：
```py
def name(list):
  return 表达式
name(list)
```
显然，使用普通方法定义此函数，需要 3 行代码，而使用 lambda 表达式仅需 1 行。

举个例子，如果设计一个求 2 个数之和的函数，使用普通函数的方式，定义如下：
```py
def add(x, y):
  return x + y
print(add(3,4)) # 7
```
由于上面程序中，`add()`函数内部仅有 1 行表达式，因此该函数可以直接用 lambda 表达式表示：
```py
add = lambda x,y:x+y
print(add(3,4)) # 7
```
可以这样理解 lambda 表达式，其就是简单函数（函数体仅是单行的表达式）的简写版本。相比函数，lamba 表达式具有以下  2 个优势：
* 对于单行函数，使用 lambda 表达式可以省去定义函数的过程，让代码更加简洁；
* 对于不需要多次复用的函数，使用 lambda 表达式可以在用完之后立即释放，提高程序执行的性能。

# eval()和exec()函数
`eval()`和`exec()`函数都属于 Python 的内置函数。

`eval()`和`exec()`函数的功能是相似的，都可以执行一个字符串形式的 Python 代码（代码以字符串的形式提供），相当于一个 Python 的解释器。二者不同之处在于，`eval()`执行完要返回结果，而`exec()`执行完不返回结果。
## eval()和exec()的用法
```py
eval(expression, globals=None, locals=None, /)
exec(expression, globals=None, locals=None, /)
```
可以看到，二者的语法格式除了函数名，其他都相同，其中各个参数的具体含义如下：
* `expression`：这个参数是一个字符串，代表要执行的语句 。该语句受后面两个字典类型参数`globals`和`locals`的限制，只有在`globals`字典和`locals`字典作用域内的函数和变量才能被执行。
* `globals`：这个参数管控的是一个全局的命名空间，即`expression`可以使用全局命名空间中的函数。如果只是提供了`globals`参数，而没有提供自定义的`__builtins__`，则系统会将当前环境中的`__builtins__`复制到自己提供的`globals`中，然后才会进行计算；如果连`globals`这个参数都没有被提供，则使用 Python 的全局命名空间。
* `locals`：这个参数管控的是一个局部的命名空间，和`globals`类似，当它和`globals`中有重复或冲突时，以`locals`的为准。如果`locals`没有被提供，则默认为`globals`。

注意，`__builtins__`是 Python 的内建模块，平时使用的`int、str、abs`都在这个模块中。通过`print(dic["__builtins__"])`语句可以查看`__builtins__`所对应的`value`。

首先，通过如下的例子来演示参数`globals`作用域的作用，注意观察它是何时将`__builtins__`复制`globals`字典中去的：
```py
dic={} #定义一个字典
dic['b'] = 3 #在 dic 中加一条元素，key 为 b
print (dic.keys()) #先将 dic 的 key 打印出来，有一个元素 b
exec("a = 4", dic) #在 exec 执行的语句后面跟一个作用域 dic
print(dic.keys()) #exec 后，dic 的 key 多了一个
```
运行结果为：
```
dict_keys(['b'])
dict_keys(['b', '__builtins__', 'a'])
```
上面的代码是在作用域`dic`下执行了一句`a = 4`的代码。可以看出，`exec()`之前`dic`中的`key`只有一个`b`。执行完`exec()`之后，系统在`dic`中生成了两个新的`key`，分别是`a`和`__builtins__`。其中，`a`为执行语句生成的变量，系统将其放到指定的作用域字典里；`__builtins__`是系统加入的内置`key`。

`locals`参数的用法就很简单了：
```py
a=10
b=20
c=30
g={'a':6, 'b':8} #定义一个字典
t={'b':100, 'c':10} #定义一个字典
print(eval('a+b+c', g, t)) #定义一个字典 116
```
输出结果为：
```
116
```
## exec()和eval()的区别
它们的区别在于，`eval()`执行完会返回结果，而`exec()`执行完不返回结果。
```py
a = 1
exec("a = 2") #相当于直接执行 a=2
print(a)
a = exec("2+3") #相当于直接执行 2+3，但是并没有返回值，a 应为 None
print(a)
a = eval('2+3') #执行 2+3，并把结果返回给 a
print(a)
```
运行结果为：
```
2
None
5
```
可以看出，`exec()`中最适合放置运行后没有结果的语句，而`eval()`中适合放置有结果返回的语句。

如果`eval()`里放置一个没有结果返回的语句会怎样呢？例如下面代码：
```
a= eval("a = 2")
```
这时 Python 解释器会报`SyntaxError`错误，提示`eval()`中不识别等号语法。
#### eval() 和 exec() 函数的应用场景
在使用 Python 开发服务端程序时，这两个函数应用得非常广泛。例如，客户端向服务端发送一段字符串代码，服务端无需关心具体的内容，直接跳过`eval()`或`exec()`来执行，这样的设计会使服务端与客户端的耦合度更低，系统更易扩展。

需要注意的是，在使用`eval()`或是`exec()`来处理请求代码时，函数`eval()`和`exec()`常常会被黑客利用，成为可以执行系统级命令的入口点，进而来攻击网站。解决方法是：通过设置其命名空间里的可执行函数，来限制`eval()`和`exec()`的执行范围。
