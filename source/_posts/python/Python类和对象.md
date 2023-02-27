---
title: Python类和对象
date: 2022-12-15 13:38:43
tags: [python]
categories: python
---


# class：定义类
定义一个类使用`class`关键字实现：
```
class 类名：
  多个（≥0）类属性...
  多个（≥0）类方法...
```
无论是类属性还是类方法，对于类来说，它们都不是必需的。另外，类中属性和方法所在的位置是任意的，即它们之间并没有固定的前后次序。

类属性指的就是包含在类中的变量；类方法指的是包含类中的函数。换句话说，类属性和类方法其实分别是包含类中的变量和函数的别称。

Python 类是由类头（`class`类名）和类体（统一缩进的变量和函数）构成。
```py
class TheFirstDemo:
  '''这是一个学习Python定义的第一个类'''
  # 下面定义了一个类属性
  add = 'hello'
  # 下面定义了一个say方法
  def say(self, content):
    print(content)
```
和函数一样，我们也可以为类定义说明文档，其要放到类头之后，类体之前的位置，如上面程序中第二行的字符串，就是`TheFirstDemo`这个类的说明文档。

另外分析上面的代码可以看到，我们创建了一个名为`TheFirstDemo`的类，其包含了一个名为`add`的类属性。注意，根据定义属性位置的不同，在各个类方法之外定义的变量称为类属性或类变量（如`add`属性）。

同时，`TheFirstDemo`类中还包含一个`say()`类方法，该方法包含两个参数，分别是`self`和`content`。`content`参数就只是一个普通参数，没有特殊含义，但`self`比较特殊，并不是普通的参数。

更确切地说，`say()`是一个实例方法，除此之外，Python 类中还可以定义类方法和静态方法。

事实上，我们完全可以创建一个没有任何类属性和类方法的类，换句话说，Python 允许创建空类：
```py
class Empty:
  pass
```
可以看到，如果一个类没有任何类属性和类方法，那么可以直接用`pass`关键字作为类体即可。
# __init__()类构造方法
在创建类时，我们可以手动添加一个`__init__()`方法，该方法是一个特殊的类实例方法，称为构造方法（或构造函数）。

构造方法用于创建对象时使用，每当创建一个类的实例对象时，Python 解释器都会自动调用它。
```
def __init__(self,...):
  代码块
```
另外，`__init__()`方法可以包含多个参数，但必须包含一个名为`self`的参数，且必须作为第一个参数。也就是说，类的构造方法最少也要有一个`self`参数。
```py
class TheFirstDemo:
  '''这是一个学习Python定义的第一个类'''
  #构造方法
  def __init__(self):
    print("调用构造方法")
  # 下面定义了一个类属性
  add = 'test'
  # 下面定义了一个say方法
  def say(self, content):
    print(content)
```
注意，即便不手动为类添加任何构造方法，Python 也会自动为类添加一个仅包含`self`参数的构造方法。

仅包含`self`参数的`__init__()`构造方法，又称为类的默认构造方法。
```py
zhangsan = TheFirstDemo()
```
这行代码的含义是创建一个名为`zhangsan`的`TheFirstDemo`类对象。运行代码可看到如下结果：
```
调用构造方法
```
显然，在创建`zhangsan`这个对象时，隐式调用了我们手动创建的`__init__()`构造方法。

在`__init__()`构造方法中，除了`self`参数外，还可以自定义一些参数，参数之间使用逗号进行分割。
```py
class CLanguage:
  '''这是一个学习Python定义的一个类'''
  def __init__(self, name, add):
    print(name, "的英文名为:", add)
#创建 add 对象，并传递参数给构造函数
add = CLanguage("小明","xiaoming")
```
可以看到，虽然构造方法中有`self、name、add`3 个参数，但实际需要传参的仅有`name`和`add`，也就是说，`self`不需要手动传递参数。
# 类对象的创建和使用
## 类的实例化
对已定义好的类进行实例化：
```
类名(参数)
```
定义类时，如果没有手动添加`__init__()`构造方法，又或者添加的`__init__()`中仅有一个`self`参数，则创建类对象时的参数可以省略不写。
```py
class CLanguage :
  # 下面定义了2个类变量
  name = "小明"
  add = "xiaoming"
  def __init__(self, name, add):
    #下面定义 2 个实例变量
    self.name = name
    self.add = add
    print(name,"的英文名为：",add)
  # 下面定义了一个say实例方法
  def say(self, content):
    print(content)
# 将该CLanguage对象赋给clanguage变量
clanguage = CLanguage("小明","xiaoming")
```
在上面的程序中，由于构造方法除`self`参数外，还包含 2 个参数，且这 2 个参数没有设置默认参数，因此在实例化类对象时，需要传入相应的`name`值和`add`值（`self`参数是特殊参数，不需要手动传值，Python 会自动传给它值）。

类变量和实例变量，简单地理解，定义在各个类方法之外（包含在类中）的变量为类变量（或者类属性），定义在类方法中的变量为实例变量（或者实例属性）。
## 类对象的使用
定义的类只有进行实例化，也就是使用该类创建对象之后，才能得到利用。总的来说，实例化后的类对象可以执行以下操作：
* 访问或修改类对象具有的实例变量，甚至可以添加新的实例变量或者删除已有的实例变量；
* 调用类对象的方法，包括调用现有的方法，以及给类对象动态添加方法。

#### 类对象访问变量或方法
使用已创建好的类对象访问类中实例变量的语法格式如下：
```
类对象名.变量名
```
使用类对象调用类中方法的语法格式如下：
```
对象名.方法名(参数)
```
```py
#输出name和add实例变量的值
print(clanguage.name,clanguage.add)
#修改实例变量的值
clanguage.name="小红"
clanguage.add="xiaohong"
#调用clanguage的say()方法
clanguage.say("人生苦短，我用Python")
#再次输出name和add的值
print(clanguage.name,clanguage.add)
```
#### 给类对象动态添加/删除变量
Python 支持为已创建好的对象动态增加实例变量：
```py
# 为clanguage对象增加一个money实例变量
clanguage.money= 159.9
print(clanguage.money) # 159.9
```
动态删除使用`del`语句即可实现：
```py
#删除新添加的 money 实例变量
del clanguage.money
#再次尝试输出 money，此时会报错
print(clanguage.money)
```
运行程序会发现，结果显示`AttributeError`错误：
```
Traceback (most recent call last):
  File "C:/Users/mengma/Desktop/1.py", line 29, in <module>
    print(clanguage.money)
AttributeError: 'CLanguage' object has no attribute 'money'
```
#### 给类对象动态添加方法
Python 也允许为对象动态增加方法。以`Clanguage`类为例，由于其内部只包含一个`say()`方法，因此该类实例化出的`clanguage`对象也只包含一个`say()`方法。但其实，我们还可以为`clanguage`对象动态添加其它方法。

需要注意的一点是，为`clanguage`对象动态增加的方法，Python 不会自动将调用者自动绑定到第一个参数（即使将第一个参数命名为`self`也没用）。
```py
# 先定义一个函数
def info(self):
  print("---info函数---", self)
# 使用info对clanguage的foo方法赋值（动态绑定方法）
clanguage.foo = info
# Python不会自动将调用者绑定到第一个参数，
# 因此程序需要手动将调用者绑定为第一个参数
clanguage.foo(clanguage)  # ①
# 使用lambda表达式为clanguage对象的bar方法赋值（动态绑定方法）
clanguage.bar = lambda self: print('--lambda表达式--', self)
clanguage.bar(clanguage) # ②
```
上面的第 8 行和第 10 行代码分别使用函数、lambda 表达式为`clanguage`对象动态增加了方法，但对于动态增加的方法，Python 不会自动将方法调用者绑定到它们的第一个参数，因此程序必须手动为第一个参数传入参数值，如上面程序中 ① 号、② 号代码所示。

有没有不用手动给`self`传值的方法呢？通过借助`types`模块下的`MethodType`可以实现：
```py
def info(self,content):
  print("小明的英文名为：%s" % content)
# 导入MethodType
from types import MethodType
clanguage.info = MethodType(info, clanguage)
# 第一个参数已经绑定了，无需传入
clanguage.info("xiaoming")
```
可以看到，由于使用`MethodType`包装`info()`函数时，已经将该函数的`self`参数绑定为`clanguage`，因此后续再使用`info()`函数时，就不用再给`self`参数绑定值了。
# self用法
在定义类的过程中，无论是显式创建类的构造方法，还是向类中添加实例方法，都要求将`self`参数作为方法的第一个参数。
```py
class Person:
  def __init__(self):
    print("正在执行构造方法")
  # 定义一个study()实例方法
  def study(self,name):
    print(name,"正在学Python")
```
事实上，Python 只是规定，无论是构造方法还是实例方法，最少要包含一个参数，并没有规定该参数的具体名称。之所以将其命名为`self`，只是程序员之间约定俗成的一种习惯（大家一看到`self`，就知道它的作用）。

那么，`self`参数的具体作用是什么呢？打个比方，如果把类比作造房子的图纸，那么类实例化后的对象是真正可以住的房子。根据一张图纸（类），我们可以设计出成千上万的房子（类对象），每个房子长相都是类似的（都有相同的类变量和类方法），但它们都有各自的主人，那么如何对它们进行区分呢？

当然是通过`self`参数，它就相当于每个房子的门钥匙，可以保证每个房子的主人仅能进入自己的房子（每个类对象只能调用自己的类变量和类方法）。

其实 Python 类方法中的`self`参数就相当于 C++ 中的`this`指针。

也就是说，同一个类可以产生多个对象，当某个对象调用类方法时，该对象会把自身的引用作为第一个参数自动传给该方法，换句话说，Python 会自动绑定类方法的第一个参数指向调用该方法的对象。如此，Python 解释器就能知道到底要操作哪个对象的方法了。

因此，程序在调用实例方法和构造方法时，不需要手动为第一个参数传值。
```py
class Person:
  def __init__(self):
    print("正在执行构造方法")
  # 定义一个study()实例方法
  def study(self):
    print(self, "正在学Python")
zhangsan = Person()
zhangsan.study()
lisi = Person()
lisi.study()
```
上面代码中，`study()`中的`self`代表该方法的调用者，即谁调用该方法，那么`self`就代表谁。因此，该程序的运行结果为：
```
正在执行构造方法
<__main__.Person object at 0x0000021ADD7D21D0> 正在学Python
正在执行构造方法
<__main__.Person object at 0x0000021ADD7D2E48> 正在学Python
```
另外，对于构造函数中的`self`参数，其代表的是当前正在初始化的类对象。
```py
class Person:
  name = "xxx"
  def __init__(self,name):
    self.name=name
zhangsan = Person("zhangsan")
print(zhangsan.name) # zhangsan
lisi = Person("lisi")
print(lisi.name) # lisi
```
可以看到，`zhangsan`在进行初始化时，调用的构造函数中`self`代表的是`zhangsan`；而`lisi`在进行初始化时，调用的构造函数中`self`代表的是`lisi`。

值得一提的是，除了类对象可以直接调用类方法，还有一种函数调用的方式：
```py
class Person:
  def who(self):
    print(self)
zhangsan = Person()
#第一种方式
zhangsan.who()
#第二种方式
who = zhangsan.who
who()#通过 who 变量调用zhangsan对象中的 who() 方法
```
运行结果为：
```
<__main__.Person object at 0x0000025C26F021D0>
<__main__.Person object at 0x0000025C26F021D0>
```
显然，无论采用哪种方法，`self`所表示的都是实际调用该方法的对象。

总之，无论是类中的构造函数还是普通的类方法，实际调用它们的谁，则第一个参数`self`就代表谁。
# 类属性和实例属性
无论是类属性还是类方法，都无法像普通变量或者函数那样，在类的外部直接使用它们。我们可以将类看做一个独立的空间，则类属性其实就是在类体中定义的变量，类方法是在类体中定义的函数。

在类体中，根据变量定义的位置不同，以及定义的方式不同，类属性又可细分为以下 3 种类型：
* 类体中、所有函数之外：此范围定义的变量，称为类属性或类变量；
* 类体中，所有函数内部：以`self.变量名`的方式定义的变量，称为实例属性或实例变量；
* 类体中，所有函数内部：以“变量名=变量值”的方式定义的变量，称为局部变量。

类方法也可细分为实例方法、静态方法和类方法。
## 类变量（类属性）
类变量指的是在类中，但在各个类方法外定义的变量。
```py
class CLanguage :
  # 下面定义了2个类变量
  name = "小明"
  add = "xiaoming"
  # 下面定义了一个say实例方法
  def say(self, content):
    print(content)
```
上面程序中，`name`和`add`就属于类变量。

类变量的特点是，所有类的实例化对象都同时共享类变量，也就是说，类变量在所有实例化对象中是作为公用资源存在的。类变量的调用方式有 2 种，既可以使用类名直接调用，也可以使用类的实例化对象调用。
```py
#使用类名直接调用
print(CLanguage.name) # 小明
print(CLanguage.add) # xiaoming
#修改类变量的值
CLanguage.name = "小红"
CLanguage.add = "xiaohong"
print(CLanguage.name) # 小红
print(CLanguage.add) # xiaohong
```
可以看到，通过类名不仅可以调用类变量，也可以修改它的值。

当然，也可以使用类对象来调用所属类中的类变量。
```py
clang = CLanguage()
print(clang.name)
print(clang.add)
```
注意，因为类变量为所有实例化对象共有，通过类名修改类变量的值，会影响所有的实例化对象。
```py
print("修改前，各类对象中类变量的值：")
clang1 = CLanguage()
print(clang1.name) # 小明
print(clang1.add) # xiaoming
clang2 = CLanguage()
print(clang2.name) # 小明
print(clang2.add) # xiaoming
print("修改后，各类对象中类变量的值：")
CLanguage.name = "小红"
CLanguage.add = "xiaohong"
print(clang1.name) # 小红
print(clang1.add) # xiaohong
print(clang2.name) # 小红
print(clang2.add) # xiaohong
```
显然，通过类名修改类变量，会作用到所有的实例化对象（例如这里的`clang1`和`clang2`）。

注意，通过类对象是无法修改类变量的。通过类对象对类变量赋值，其本质将不再是修改类变量的值，而是在给该对象定义新的实例变量。

值得一提的是，除了可以通过类名访问类变量之外，还可以动态地为类和对象添加类变量。
```py
clang = CLanguage()
CLanguage.catalog = 13
print(clang.catalog) # 13
```
## 实例变量（实例属性）
实例变量指的是在任意类方法内部，以“self.变量名”的方式定义的变量，其特点是只作用于调用方法的对象。另外，实例变量只能通过对象名访问，无法通过类名访问。
```py
class CLanguage :
  def __init__(self):
    self.name = "小明"
    self.add = "xiaoming"
    # 下面定义了一个say实例方法
  def say(self):
    self.catalog = 13
```
此`CLanguage`类中，`name、add`以及`catalog`都是实例变量。其中，由于`__init__()`函数在创建类对象时会自动调用，而`say()`方法需要类对象手动调用。因此，`CLanguage`类的类对象都会包含`name`和`add`实例变量，而只有调用了`say()`方法的类对象，才包含`catalog`实例变量。
```py
clang = CLanguage()
print(clang.name) # 小明
print(clang.add) # xiaoming
#由于 clang 对象未调用 say() 方法，因此其没有 catalog 变量，下面这行代码会报错
#print(clang.catalog)
clang2 = CLanguage()
print(clang2.name) # 小明
print(clang2.add) # xiaoming
#只有调用 say()，才会拥有 catalog 实例变量
clang2.say()
print(clang2.catalog) # 13
```
通过类对象可以访问类变量，但无法修改类变量的值。这是因为，通过类对象修改类变量的值，不是在给“类变量赋值”，而是定义新的实例变量。
```py
clang = CLanguage()
#clang访问类变量
print(clang.name) # 小明
print(clang.add) # xiaoming
clang.name = "小红"
clang.add = "xiaohong"
#clang实例变量的值
print(clang.name) # 小红
print(clang.add) # xiaohong
#类变量的值
print(CLanguage.name) # 小明
print(CLanguage.add) # xiaoming
```
显然，通过类对象是无法修改类变量的值的，本质其实是给`clang`对象新添加`name`和`add`这 2 个实例变量。

类中，实例变量和类变量可以同名，但这种情况下使用类对象将无法调用类变量，它会首选实例变量，这也是不推荐“类变量使用对象名调用”的原因。

另外，和类变量不同，通过某个对象修改实例变量的值，不会影响类的其它实例化对象，更不会影响同名的类变量。
```py
class CLanguage :
  name = "小明"  #类变量
  add = "xiaoming"  #类变量
  def __init__(self):
    self.name = "小红"   #实例变量
    self.add = "xiaohong"   #实例变量
  # 下面定义了一个say实例方法
  def say(self):
    self.catalog = 13  #实例变量
clang = CLanguage()
#修改 clang 对象的实例变量
clang.name = "小李"
clang.add = "xiaoli"
print(clang.name) # 小李
print(clang.add) # xiaoli
clang2 = CLanguage()
print(clang2.name) # 小红
print(clang2.add) # xiaohong
#输出类变量的值
print(CLanguage.name) # 小明
print(CLanguage.add) # xiaoming
```
不仅如此，Python 只支持为特定的对象添加实例变量。例如，在之前代码的基础上，为`clang`对象添加`money`实例变量：
```py
clang.money = 30
print(clang.money)
```
## 局部变量
除了实例变量，类方法中还可以定义局部变量。局部变量直接以“变量名=值”的方式进行定义：
```py
class CLanguage :
  # 下面定义了一个say实例方法
  def count(self,money):
    sale = 0.8*money
    print("优惠后的价格为：",sale)
clang = CLanguage()
clang.count(100)
```
通常情况下，定义局部变量是为了所在类方法功能的实现。需要注意的一点是，局部变量只能用于所在函数中，函数执行完成后，局部变量也会被销毁。
# 实例方法、静态方法和类方法
类方法可分为类方法、实例方法和静态方法。采用`@classmethod`修饰的方法为类方法；采用`@staticmethod`修饰的方法为静态方法；不用任何修饰的方法为实例方法。
## 类实例方法
通常情况下，在类中定义的方法默认都是实例方法。类的构造方法理论上也属于实例方法，只不过它比较特殊。
```py
class CLanguage:
  #类构造方法，也属于实例方法
  def __init__(self):
    self.name = "小明"
    self.add = "xiaoming"
  # 下面定义了一个say实例方法
  def say(self):
    print("正在调用 say() 实例方法")
```
实例方法最大的特点就是，它最少也要包含一个`self`参数，用于绑定调用此方法的实例对象（Python 会自动完成绑定）。实例方法通常会用类对象直接调用：
```py
clang = CLanguage()
clang.say() # 正在调用 say() 实例方法
```
当然，Python 也支持使用类名调用实例方法，但此方式需要手动给`self`参数传值。
```py
#类名调用实例方法，需手动给 self 参数传值
clang = CLanguage()
CLanguage.say(clang) # 正在调用 say() 实例方法
```
## 类方法
类方法和实例方法相似，它最少也要包含一个参数，只不过类方法中通常将其命名为`cls`，Python 会自动将类本身绑定给`cls`参数（注意，绑定的不是类对象）。也就是说，我们在调用类方法时，无需显式为`cls`参数传参。

和`self`一样，`cls`参数的命名也不是规定的（可以随意命名），只是 Python 程序员约定俗称的习惯而已。

和实例方法最大的不同在于，类方法需要使用`@classmethod`修饰符进行修饰：
```py
class CLanguage:
  #类构造方法，也属于实例方法
  def __init__(self):
    self.name = "小明"
    self.add = "xiaoming"
  #下面定义了一个类方法
  @classmethod
  def info(cls):
    print("正在调用类方法", cls)
```
如果没有`＠classmethod`，则 Python 解释器会将`info()`方法认定为实例方法，而不是类方法。

类方法推荐使用类名直接调用，当然也可以使用实例对象来调用（不推荐）。
```py
#使用类名直接调用类方法
CLanguage.info()
#使用类对象调用类方法
clang = CLanguage()
clang.info()
```
## 类静态方法
静态方法，其实就是函数，和函数唯一的区别是，静态方法定义在类这个空间（类命名空间）中，而函数则定义在程序所在的空间（全局命名空间）中。

静态方法没有类似`self、cls`这样的特殊参数，因此 Python 解释器不会对它包含的参数做任何类或对象的绑定。也正因为如此，类的静态方法中无法调用任何类属性和类方法。

静态方法需要使用`@staticmethod`修饰：
```py
class CLanguage:
  @staticmethod
  def info(name, add):
    print(name, add)
```
静态方法的调用，既可以使用类名，也可以使用类对象：
```py
#使用类名直接调用静态方法
CLanguage.info("小明", "xiaoming")
#使用类对象调用静态方法
clang = CLanguage()
clang.info("小红", "xiaohong")
```
在实际编程中，几乎不会用到类方法和静态方法，因为我们完全可以使用函数代替它们实现想要的功能，但在一些特殊的场景中（例如工厂模式中），使用类方法和静态方法也是很不错的选择。
# 类调用实例方法
实例方法的调用方式有 2 种，既可以采用类对象调用，也可以直接通过类名调用。

通常情况下，我们习惯使用类对象调用类中的实例方法。但如果想用类调用实例方法，不能像如下这样：
```py
class CLanguage:
  def info(self):
    print("我正在学 Python")
#通过类名直接调用实例方法
CLanguage.info()
```
运行上面代码，程序会报出如下错误：
```
Traceback (most recent call last):
  File "D:\python3.6\demo.py", line 5, in <module>
    CLanguage.info()
TypeError: info() missing 1 required positional argument: 'self'
```
其中，最后一行报错信息提示我们，调用`info()`类方式时缺少给`self`参数传参。这意味着，和使用类对象调用实例方法不同，通过类名直接调用实例方法时，Python 并不会自动给`self`参数传值。

`self`参数需要的是方法的实际调用者（是类对象），而这里只提供了类名，当然无法自动传值。

因此，如果想通过类名直接调用实例方法，就必须手动为`self`参数传值。
```py
class CLanguage:
  def info(self):
    print("我正在学 Python")
clang = CLanguage()
#通过类名直接调用实例方法
CLanguage.info(clang)
```
可以看到，通过手动将`clang`这个类对象传给了`self`参数，使得程序得以正确执行。实际上，这里调用实例方法的形式完全是等价于`clang.info()`。

值得一提的是，上面的报错信息只是让我们手动为`self`参数传值，但并没有规定必须传一个该类的对象，其实完全可以任意传入一个参数：
```py
class CLanguage:
  def info(self):
    print(self,"正在学 Python")
#通过类名直接调用实例方法
CLanguage.info("zhangsan")
```
运行结果为：
```
zhangsan 正在学 Python
```
可以看到，`"zhangsan"`这个字符串传给了`info()`方法的`self`参数。显然，无论是`info()`方法中使用`self`参数调用其它类方法，还是使用`self`参数定义新的实例变量，胡乱的给`self`参数传参都将会导致程序运行崩溃。

总的来说，Python 中允许使用类名直接调用实例方法，但必须手动为该方法的第一个`self`参数传递参数，这种调用方法的方式被称为“非绑定方法”。

用类的实例对象访问类成员的方式称为绑定方法，而用类名调用类成员的方式称为非绑定方法。
# 描述符
通过使用描述符，可以让程序员在引用一个对象属性时自定义要完成的工作。

本质上看，描述符就是一个类，只不过它定义了另一个类中属性的访问方式。换句话说，一个类可以将属性管理全权委托给描述符类。

描述符是 Python 中复杂属性访问的基础，它在内部被用于实现 property、方法、类方法、静态方法和`super`类型。

描述符类基于以下 3 个特殊方法，换句话说，这 3 个方法组成了描述符协议：
* `__set__(self, obj, type=None)`：在设置属性时将调用这一方法；
* `__get__(self, obj, value)`：在读取属性时将调用这一方法；
* `__delete__(self, obj)`：对属性调用`del`时将调用这一方法。

其中，实现了`setter`和`getter`方法的描述符类被称为数据描述符；反之，如果只实现了`getter`方法，则称为非数据描述符。

实际上，在每次查找属性时，描述符协议中的方法都由类对象的特殊方法`__getattribute__()`调用（注意不要和`__getattr__()`弄混）。也就是说，每次使用类对象.属性（或者`getattr(类对象，属性值)`）的调用方式时，都会隐式地调用`__getattribute__()`，它会按照下列顺序查找该属性：
* 验证该属性是否为类实例对象的数据描述符；
* 如果不是，就查看该属性是否能在类实例对象的`__dict__`中找到；
* 最后，查看该属性是否为类实例对象的非数据描述符。

```py
#描述符类
class revealAccess:
  def __init__(self, initval = None, name = 'var'):
    self.val = initval
    self.name = name
  def __get__(self, obj, objtype):
    print("Retrieving",self.name)
    return self.val
  def __set__(self, obj, val):
    print("updating",self.name)
    self.val = val
class myClass:
  x = revealAccess(10,'var "x"')
  y = 5
m = myClass()
print(m.x)
m.x = 20
print(m.x)
print(m.y)
```
运行结果为：
```
Retrieving var "x"
10
updating var "x"
Retrieving var "x"
20
5
```
从这个例子可以看到，如果一个类的某个属性有数据描述符，那么每次查找这个属性时，都会调用描述符的`__get__()`方法，并返回它的值；同样，每次在对该属性赋值时，也会调用`__set__()`方法。

注意，虽然上面例子中没有使用`__del__()`方法，但也很容易理解，当每次使用`del`类对象.属性（或者 delattr(类对象，属性)）语句时，都会调用该方法。
# property()函数
我们一直在用“类对象.属性”的方式访问类中定义的属性，其实这种做法是欠妥的，因为它破坏了类的封装原则。正常情况下，类包含的属性应该是隐藏的，只允许通过类提供的方法来间接实现对类属性的访问和操作。

因此，在不破坏类封装原则的基础上，为了能够有效操作类中的属性，类中应包含读（或写）类属性的多个`getter`（或`setter`）方法，这样就可以通过“类对象.方法(参数)”的方式操作属性：
```py
class CLanguage:
  #构造函数
  def __init__(self,name):
    self.name = name 
  #设置 name 属性值的函数 
  def setname(self,name):
    self.name = name
  #访问nema属性值的函数
  def getname(self):
    return self.name
  #删除name属性值的函数
  def delname(self):
    self.name="xxx"
clang = CLanguage("小明")
#获取name属性值
print(clang.getname()) # 小明
#设置name属性值
clang.setname("Python教程") # Python教程
print(clang.getname())
#删除name属性值
clang.delname()
print(clang.getname()) # xxx
```
Python 中提供了`property()`函数，可以实现在不破坏类封装原则的前提下，让开发者依旧使用“类对象.属性”的方式操作类中的属性。
```
属性名=property(fget=None, fset=None, fdel=None, doc=None)
```
其中，`fget`参数用于指定获取该属性值的类方法，`fset`参数用于指定设置该属性值的方法，`fdel`参数用于指定删除该属性值的方法，最后的`doc`是一个文档字符串，用于说明此函数的作用。

注意，在使用`property()`函数时，以上 4 个参数可以仅指定第 1 个、或者前 2 个、或者前 3 个，当前也可以全部指定。也就是说，`property()`函数中参数的指定并不是完全随意的。
```py
class CLanguage:
  #构造函数
  def __init__(self,n):
    self.__name = n
  #设置 name 属性值的函数
  def setname(self,n):
    self.__name = n
  #访问nema属性值的函数
  def getname(self):
    return self.__name
  #删除name属性值的函数
  def delname(self):
    self.__name="xxx"
  #为name 属性配置 property() 函数
  name = property(getname, setname, delname, '指明出处')
#调取说明文档的 2 种方式
#print(CLanguage.name.__doc__)
help(CLanguage.name)
clang = CLanguage("小明")
#调用 getname() 方法
print(clang.name)
#调用 setname() 方法
clang.name="Python教程"
print(clang.name)
#调用 delname() 方法
del clang.name
print(clang.name)
```
运行结果为：
```
Help on property:

    指明出处

小明
Python教程
xxx
```
注意，在此程序中，由于`getname()`方法中需要返回`name`属性，如果使用`self.name`的话，其本身又被调用`getname()`，这将会先入无限死循环。为了避免这种情况的出现，程序中的`name`属性必须设置为私有属性，即使用`__name`（前面有 2 个下划线）。

当然，`property()`函数也可以少传入几个参数。
```py
name = property(getname, setname)
```
这意味着，`name`是一个可读写的属性，但不能删除，因为`property()`函数中并没有为`name`配置用于函数该属性的方法。也就是说，即便`CLanguage`类中设计有`delname()`函数，这种情况下也不能用来删除`name`属性。 

同理，还可以像如下这样使用`property()`函数：
```py
name = property(getname) # name 属性可读，不可写，也不能删除
name = property(getname, setname,delname) # name属性可读、可写、也可删除，就是没有说明文档
```
# 封装
简单的理解封装，即在设计类时，刻意地将一些属性和方法隐藏在类的内部，这样在使用此类时，将无法直接以“类对象.属性名”（或者“类对象.方法名(参数)”）的形式调用这些属性（或方法），而只能用未隐藏的类方法间接操作这些隐藏的属性和方法。

封装机制保证了类内部数据结构的完整性，因为使用类的用户无法直接看到类中的数据结构，只能使用类允许公开的数据，很好地避免了外部对内部数据的影响，提高了程序的可维护性。

除此之外，对一个类实现良好的封装，用户只能借助暴露出来的类方法来访问数据，我们只需要在这些暴露的方法中加入适当的控制逻辑，即可轻松实现用户对类中属性或方法的不合理操作。
## Python 类如何进行封装
Python 类中的变量和函数，不是公有的（类似`public`属性），就是私有的（类似`private`）。

但是，Python 并没有提供`public、private`这些修饰符。为了实现类的封装，Python 采取了下面的方法：
* 默认情况下，Python 类中的变量和方法都是公有的，它们的名称前都没有下划线（`_`）；
* 如果类中的变量和函数，其名称以双下划线`__`开头，则该变量（函数）为私有变量（私有函数）。

除此之外，还可以定义以单下划线`_`开头的类属性或者类方法（例如`_name、_display(self)`），这种类属性和类方法通常被视为私有属性和私有方法，虽然它们也能通过类对象正常访问，但这是一种约定俗称的用法。

> 注意，Python 类中还有以双下划线开头和结尾的类方法（例如类的构造函数`__init__(self)`），这些都是 Python 内部定义的，用于 Python 内部调用。我们自己定义类属性或者类方法时，不要使用这种格式。

```py
class CLanguage :
  def setname(self, name):
    if len(name) < 1:
      raise ValueError('名称长度必须大于1！')
    self.__name = name
  def getname(self):
    return self.__name
  #为 name 配置 setter 和 getter 方法
  name = property(getname, setname)
  def setadd(self, add):
    if add.startswith("http://"):
      self.__add = add
    else:
      raise ValueError('地址必须以 http:// 开头') 
  def getadd(self):
    return self.__add
  
  #为 add 配置 setter 和 getter 方法
  add = property(getadd, setadd)
  #定义个私有方法
  def __display(self):
    print(self.__name,self.__add)
clang = CLanguage()
clang.name = "百度"
clang.add = "http://www.baidu.com"
print(clang.name) # 百度
print(clang.add) # http://www.baidu.com
```
上面程序中，`CLanguage`将`name`和`add`属性都隐藏了起来，但同时也提供了可操作它们的“窗口”，也就是各自的`setter`和`getter`方法，这些方法都是公有的。

不仅如此，以`add`属性的`setadd()`方法为例，通过在该方法内部添加控制逻辑，即通过调用`startswith()`方法，控制用户输入的地址必须以`http://`开头，否则程序将会执行`raise`语句抛出`ValueError`异常。

通过此程序的运行逻辑不难看出，通过对`CLanguage`类进行良好的封装，使得用户仅能通过暴露的`setter()`和`getter()`方法操作`name`和`add`属性，而通过对`setname()`和`setadd()`方法进行适当的设计，可以避免用户对类中属性的不合理操作，从而提高了类的可维护性和安全性。

`CLanguage`类中还有一个`__display()`方法，由于该类方法为私有（`private`）方法，且该类没有提供操作该私有方法的“窗口”，因此我们无法在类的外部使用它。换句话说，如下调用`__display()`方法是不可行的：
```py
#尝试调用私有的 display() 方法
clang.__display()
```
这会导致如下错误：
```
Traceback (most recent call last):
  File "D:\python3.6\1.py", line 33, in <module>
    clang.__display()
AttributeError: 'CLanguage' object has no attribute '__display'
```
# 继承
继承机制经常用于创建和现有类功能类似的新类，又或是新类只需要在现有类基础上添加一些成员（属性和方法），但又不想直接将现有类代码复制给新类。也就是说，通过使用继承这种机制，可以轻松实现类的重复使用。

举个例子，假设现有一个`Shape`类，该类的`draw()`方法可以在屏幕上画出指定的形状，现在需要创建一个`Form`类，要求此类不但可以在屏幕上画出指定的形状，还可以计算出所画形状的面积。要创建这样的类，笨方法是将`draw()`方法直接复制到新类中，并添加计算面积的方法。
```py
class Shape:
  def draw(self,content):
    print("画",content)
class Form:
  def draw(self,content):
    print("画",content)
  def area(self):
    #....
    print("此图形的面积为...")
```
当然还有更简单的方法，就是使用类的继承机制。实现方法为：让`Form`类继承`Shape`类，这样当`Form`类对象调用`draw()`方法时，Python 解释器会先去 Form 中找以`draw`为名的方法，如果找不到，它还会自动去`Shape`类中找。如此，我们只需在`Form`类中添加计算面积的方法即可：
```py
class Shape:
  def draw(self,content):
    print("画",content)
class Form(Shape):
  def area(self):
    #....
    print("此图形的面积为...")
```
上面代码中，`class Form(Shape)`就表示`Form`继承`Shape`。

Python 中，实现继承的类称为子类，被继承的类称为父类（也可称为基类、超类）。因此在上面这个样例中，`Form`是子类，`Shape`是父类。

子类继承父类时，只需在定义子类时，将父类（可以是多个）放在子类之后的圆括号里即可。
```py
class 类名(父类1, 父类2, ...)：
  #类定义部分
```
注意，如果该类没有显式指定继承自哪个类，则默认继承 object 类（object 类是 Python 中所有类的父类，即要么是直接父类，要么是间接父类）。另外，Python 的继承是多继承机制（和 C++ 一样），即一个子类可以同时拥有多个直接父类。

“派生”和继承是一个意思，只是观察角度不同而已。换句话说，继承是相对子类来说的，即子类继承自父类；而派生是相对于父类来说的，即父类派生出子类。
```py
class People:
  def say(self):
    print("我是一个人，名字是：",self.name)
class Animal:
  def display(self):
    print("人也是高级动物")
#同时继承 People 和 Animal 类
#其同时拥有 name 属性、say() 和 display() 方法
class Person(People, Animal):
  pass
zhangsan = Person()
zhangsan.name = "张三"
zhangsan.say()
zhangsan.display()
```
运行结果为：
```
我是一个人，名字是： 张三
人也是高级动物
```
可以看到，虽然 Person 类为空类，但由于其继承自 People 和 Animal 这 2 个类，因此实际上 Person 并不空，它同时拥有这 2 个类所有的属性和方法。
没错，子类拥有父类所有的属性和方法，即便该属性或方法是私有（private）的。
## 关于Python的多继承
事实上，大部分面向对象的编程语言，都只支持单继承，即子类有且只能有一个父类。而 Python 却支持多继承（C++也支持多继承）。
和单继承相比，多继承容易让代码逻辑复杂、思路混乱，一直备受争议，中小型项目中较少使用，后来的 Java、C#、PHP 等干脆取消了多继承。

使用多继承经常需要面临的问题是，多个父类中包含同名的类方法。对于这种情况，Python 的处置措施是：根据子类继承多个父类时这些父类的前后次序决定，即排在前面父类中的类方法会覆盖排在后面父类中的同名类方法。
```py
class People:
  def __init__(self):
    self.name = People
  def say(self):
    print("People类",self.name)
class Animal:
  def __init__(self):
    self.name = Animal
  def say(self):
    print("Animal类",self.name)
#People中的 name 属性和 say() 会遮蔽 Animal 类中的
class Person(People, Animal):
  pass
zhangsan = Person()
zhangsan.name = "张三"
zhangsan.say()
```
程序运行结果为：
```
People类 张三
```
可以看到，当`Person`同时继承`People`类和`Animal`类时，`People`类在前，因此如果`People`和`Animal`拥有同名的类方法，实际调用的是`People`类中的。
# 父类方法重写
在 Python 中，子类继承了父类，那么子类就拥有了父类所有的类属性和类方法。通常情况下，子类会在此基础上，扩展一些新的类属性和类方法。

但凡事都有例外，我们可能会遇到这样一种情况，即子类从父类继承得来的类方法中，大部分是适合子类使用的，但有个别的类方法，并不能直接照搬父类的，如果不对这部分类方法进行修改，子类对象无法使用。针对这种情况，我们就需要在子类中重复父类的方法。

举个例子，鸟通常是有翅膀的，也会飞，因此我们可以像如下这样定义个和鸟相关的类：
```py
class Bird:
  #鸟有翅膀
  def isWing(self):
    print("鸟有翅膀")
  #鸟会飞
  def fly(self):
    print("鸟会飞")
```
但是，对于鸵鸟来说，它虽然也属于鸟类，也有翅膀，但是它只会奔跑，并不会飞。针对这种情况，可以这样定义鸵鸟类：
```py
class Ostrich(Bird):
  # 重写Bird类的fly()方法
  def fly(self):
    print("鸵鸟不会飞")
```
可以看到，因为`Ostrich`继承自`Bird`，因此`Ostrich`类拥有`Bird`类的`isWing()`和`fly()`方法。其中，`isWing()`方法同样适合`Ostrich`，但`fly()`明显不适合，因此我们在`Ostrich`类中对`fly()`方法进行重写。
```py
class Bird:
  #鸟有翅膀
  def isWing(self):
    print("鸟有翅膀")
  #鸟会飞
  def fly(self):
    print("鸟会飞")
class Ostrich(Bird):
  # 重写Bird类的fly()方法
  def fly(self):
    print("鸵鸟不会飞")
# 创建Ostrich对象
ostrich = Ostrich()
#调用 Ostrich 类中重写的 fly() 类方法
ostrich.fly()
```
运行结果为：
```
鸵鸟不会飞
```
显然，`ostrich`调用的是重写之后的`fly()`类方法。
## 如何调用被重写的方法
事实上，如果我们在子类中重写了从父类继承来的类方法，那么当在类的外部通过子类对象调用该方法时，Python 总是会执行子类中重写的方法。

这就产生一个新的问题，即如果想调用父类中被重写的这个方法，该怎么办呢？

Python 中的类可以看做是一个独立空间，而类方法其实就是出于该空间中的一个函数。而如果想要全局空间中，调用类空间中的函数，只需要在调用该函数时备注类名即可。
```py
class Bird:
  #鸟有翅膀
  def isWing(self):
    print("鸟有翅膀")
  #鸟会飞
  def fly(self):
    print("鸟会飞")
class Ostrich(Bird):
  # 重写Bird类的fly()方法
  def fly(self):
    print("鸵鸟不会飞")
# 创建Ostrich对象
ostrich = Ostrich()
#调用 Bird 类中的 fly() 方法
Bird.fly(ostrich)
```
程序运行结果为：
```
鸟会飞
```
此程序中，需要大家注意的一点是，使用类名调用其类方法，Python 不会为该方法的第一个`self`参数自定绑定值，因此采用这种调用方法，需要手动为`self`参数赋值。

通过类名调用实例方法的这种方式，又被称为未绑定方法。
# super()函数
Python 中子类会继承父类所有的类属性和类方法。严格来说，类的构造方法其实就是实例方法，因此毫无疑问，父类的构造方法，子类同样会继承。

但我们知道，Python 支持多继承，如果子类继承的多个父类中包含同名的类实例方法，则子类对象在调用该方法时，会优先选择排在最前面的父类中的实例方法。显然，构造方法也是如此。
```py
class People:
  def __init__(self,name):
    self.name = name
  def say(self):
    print("我是人，名字为：",self.name)
class Animal:
  def __init__(self,food):
    self.food = food
  def display(self):
    print("我是动物,我吃",self.food)
#People中的 name 属性和 say() 会遮蔽 Animal 类中的
class Person(People, Animal):
  pass
per = Person("zhangsan")
per.say()
#per.display()
```
运行结果，结果为：
```
我是人，名字为： zhangsan
```
上面程序中，Person 类同时继承 People 和 Animal，其中 People 在前。这意味着，在创建 per 对象时，其将会调用从 People 继承来的构造函数。因此我们看到，上面程序在创建 per 对象的同时，还要给 name 属性进行赋值。

但如果去掉最后一行的注释，运行此行代码，Python 解释器会报如下错误：
```
Traceback (most recent call last):
  File "D:\python3.6\Demo.py", line 18, in <module>
    per.display()
  File "D:\python3.6\Demo.py", line 11, in display
    print("我是动物,我吃",self.food)
AttributeError: 'Person' object has no attribute 'food'
```
这是因为，从 Animal 类中继承的 display() 方法中，需要用到 food 属性的值，但由于 People 类的构造方法“遮蔽”了Animal 类的构造方法，使得在创建 per 对象时，Animal 类的构造方法未得到执行，所以程序出错。

反过来也是如此，如果将第 13 行代码改为如下形式：
```
class Person(Animal, People)
```
则在创建 per 对象时，会给 food 属性传值。这意味着，per.display() 能顺序执行，但 per.say() 将会报错。

针对这种情况，正确的做法是定义 Person 类自己的构造方法（等同于重写第一个直接父类的构造方法）。但需要注意，如果在子类中定义构造方法，则必须在该方法中调用父类的构造方法。

在子类中的构造方法中，调用父类构造方法的方式有 2 种，分别是：
* 类可以看做一个独立空间，在类的外部调用其中的实例方法，可以向调用普通函数那样，只不过需要额外备注类名（此方式又称为未绑定方法）；
* 使用 super() 函数。但如果涉及多继承，该函数只能调用第一个直接父类的构造方法。

也就是说，涉及到多继承时，在子类构造函数中，调用第一个父类构造方法的方式有以上 2 种，而调用其它父类构造方法的方式只能使用未绑定方法。

在 Python 3.x 中，`super()`函数的语法格式：
```
super().__init__(...)
```
```py
class People:
  def __init__(self,name):
    self.name = name
  def say(self):
    print("我是人，名字为：",self.name)
class Animal:
  def __init__(self,food):
    self.food = food
  def display(self):
    print("我是动物,我吃",self.food)
class Person(People, Animal):
  #自定义构造方法
  def __init__(self,name,food):
    #调用 People 类的构造方法
    super().__init__(name)
    #super(Person,self).__init__(name) #执行效果和上一行相同
    #People.__init__(self,name)#使用未绑定方法调用 People 类构造方法
    #调用其它父类的构造方法，需手动给 self 传值
    Animal.__init__(self,food)    
per = Person("zhangsan","熟食")
per.say() # 我是人，名字为： zhangsan
per.display() # 我是动物,我吃 熟食
```
可以看到，`Person`类自定义的构造方法中，调用`People`类构造方法，可以使用`super()`函数，也可以使用未绑定方法。但是调用`Animal`类的构造方法，只能使用未绑定方法。
# __slots__：限制类实例动态添加属性和方法
了解了如何动态的为单个实例对象添加属性，甚至如果必要的话，还可以为所有的类实例对象统一添加属性（通过给类添加属性）。

那么，Python 是否也允许动态地为类或实例对象添加方法呢？答案是肯定的。我们知道，类方法又可细分为实例方法、静态方法和类方法，Python 语言允许为类动态地添加这 3 种方法；但对于实例对象，则只允许动态地添加实例方法，不能添加类方法和静态方法。

为单个实例对象添加方法，不会影响该类的其它实例对象；而如果为类动态地添加方法，则所有的实例对象都可以使用。
```py
class CLanguage:
    pass
#下面定义了一个实例方法
def info(self):
    print("正在调用实例方法")
#下面定义了一个类方法
@classmethod
def info2(cls):
    print("正在调用类方法")
#下面定义个静态方法
@staticmethod
def info3():
    print("正在调用静态方法")
#类可以动态添加以上 3 种方法，会影响所有实例对象
CLanguage.info = info
CLanguage.info2 = info2
CLanguage.info3 = info3
clang = CLanguage()
#如今，clang 具有以上 3 种方法
clang.info()
clang.info2()
clang.info3()
#类实例对象只能动态添加实例方法，不会影响其它实例对象
clang1 = CLanguage()
clang1.info = info
#必须手动为 self 传值
clang1.info(clang1)
```
程序输出结果为：
正在调用实例方法
正在调用类方法
正在调用静态方法
正在调用实例方法

显然，动态给类或者实例对象添加属性或方法，是非常灵活的。但与此同时，如果胡乱地使用，也会给程序带来一定的隐患，即程序中已经定义好的类，如果不做任何限制，是可以做动态的修改的。

庆幸的是，Python 提供了`__slots__`属性，通过它可以避免用户频繁的给实例对象动态地添加属性或方法。

再次声明，`__slots__`只能限制为实例对象动态添加属性和方法，而无法限制动态地为类添加属性和方法。

`__slots__`属性值其实就是一个元组，只有其中指定的元素，才可以作为动态添加的属性或者方法的名称。举个例子：
```
class CLanguage:
    __slots__ = ('name','add','info')
```
可以看到，`CLanguage`类中指定了`__slots__`属性，这意味着，该类的实例对象仅限于动态添加`name、add、info`这 3 个属性以及`name()、add()`和`info()`这 3 个方法。

注意，对于动态添加的方法，`__slots__`限制的是其方法名，并不限制参数的个数。
```py
def info(self,name):
    print("正在调用实例方法",self.name)
clang = CLanguage()
clang.name = "小明"
#为 clang 对象动态添加 info 实例方法
clang.info = info
clang.info(clang,"Python教程")
```
程序运行结果为：
```
正在调用实例方法 小明
```
还是在`CLanguage`类的基础上，添加如下代码并运行：
```py
def info(self,name):
    print("正在调用实例方法",self.name)
clang = CLanguage()
clang.name = "小明"
clang.say = info
clang.say(clang,"Python教程")
```
运行程序，显示如下信息：
```
Traceback (most recent call last):
  File "D:\python3.6\1.py", line 9, in <module>
    clang.say = info
AttributeError: 'CLanguage' object has no attribute 'say'
```
显然，根据`__slots__`属性的设置，`CLanguage`类的实例对象是不能动态添加以 say 为名称的方法的。

`__slots__`属性限制的对象是类的实例对象，而不是类，因此下面的代码是合法的：
```py
def info(self):
  print("正在调用实例方法")
CLanguage.say = info
clang = CLanguage()
clang.say()
```
程序运行结果为：
```
正在调用实例方法
```
当然，还可以为类动态添加类方法和静态方法，这里不再给出具体实例，读者可自行编写代码尝试。


此外，`__slots__`属性对由该类派生出来的子类，也是不起作用的。例如如下代码：
```py
class CLanguage:
  __slots__ = ('name','add','info')
#Clanguage 的空子类
class CLangs(CLanguage):
  pass
#定义的实例方法
def info(self):
  print("正在调用实例方法")
clang = CLangs()
#为子类对象动态添加 say() 方法
clang.say = info
clang.say(clang)
```
运行结果为：
```
正在调用实例方法
```
显然，`__slots__`属性只对当前所在的类起限制作用。

因此，如果子类也要限制外界为其实例对象动态地添加属性和方法，必须在子类中设置`__slots__`属性。

注意，如果为子类也设置有`__slots__`属性，那么子类实例对象允许动态添加的属性和方法，是子类中`__slots__`属性和父类`__slots__`属性的和。
# type()函数：动态创建类
我们知道，`type()`函数属于 Python 内置函数，通常用来查看某个变量的具体类型。其实，`type()`函数还有一个更高级的用法，即创建一个自定义类型（也就是创建一个类）。

type() 函数的语法格式有 2 种：
```
type(obj) 
type(name, bases, dict)
```
以上这 2 种语法格式，各参数的含义及功能分别是：
* 第一种语法格式用来查看某个变量（类对象）的具体类型，obj 表示某个变量或者类对象。
* 第二种语法格式用来创建类，其中 name 表示类的名称；bases 表示一个元组，其中存储的是该类的父类；dict 表示一个字典，用于表示类内定义的属性或者方法。

```py
#查看 3.4 的类型
print(type(3.4))
#查看类对象的类型
class CLanguage:
    pass
clangs = CLanguage()
print(type(clangs))
```
输出结果为：
```
<class 'float'>
<class '__main__.CLanguage'>
```
`type()`函数的另一种用法，即创建一个新类，先来分析一个样例：
```py
#定义一个实例方法
def say(self):
    print("我要学 Python！")
#使用 type() 函数创建类
CLanguage = type("CLanguage",(object,),dict(say = say, name = "C语言中文网"))
#创建一个 CLanguage 实例对象
clangs = CLanguage()
#调用 say() 方法和 name 属性
clangs.say()
print(clangs.name)
```
注意，Python 元组语法规定，当`(object,)`元组中只有一个元素时，最后的逗号（,）不能省略。

可以看到，此程序中通过`type()`创建了类，其类名为`CLanguage`，继承自`objects`类，且该类中还包含一个`say()`方法和一个`name`属性。

运行上面的程序，其输出结果为：
```
我要学 Python！
C语言中文网
```
可以看到，使用`type()`函数创建的类，和直接使用`class`定义的类并无差别。事实上，我们在使用`class`定义类时，Python 解释器底层依然是用`type()`来创建这个类。
# MetaClass元类
`MetaClass`元类，本质也是一个类，但和普通类的用法不同，它可以对类内部的定义（包括类属性和类方法）进行动态的修改。可以这么说，使用元类的主要目的就是为了实现在创建类时，能够动态地改变类中定义的属性或者方法。

不要从字面上去理解元类的含义，事实上`MetaClass`中的`Meta`这个词根，起源于希腊语词汇 meta，包含“超越”和“改变”的意思。

举个例子，根据实际场景的需要，我们要为多个类添加一个 name 属性和一个 say() 方法。显然有多种方法可以实现，但其中一种方法就是使用 MetaClass 元类。

如果在创建类时，想用`MetaClass`元类动态地修改内部的属性或者方法，则类的创建过程将变得复杂：先创建 MetaClass 元类，然后用元类去创建类，最后使用该类的实例化对象实现功能。

如果想把一个类设计成`MetaClass`元类，其必须符合以下条件：
* 必须显式继承自`type`类；
* 类中需要定义并实现`__new__()`方法，该方法一定要返回该类的一个实例对象，因为在使用元类创建类时，该`__new__()`方法会自动被执行，用来修改新建的类。

我们先尝试定义一个`MetaClass`元类：
```py
#定义一个元类
class FirstMetaClass(type):
    # cls代表动态修改的类
    # name代表动态修改的类名
    # bases代表被动态修改的类的所有父类
    # attr代表被动态修改的类的所有属性、方法组成的字典
    def __new__(cls, name, bases, attrs):
        # 动态为该类添加一个name属性
        attrs['name'] = "C语言中文网"
        attrs['say'] = lambda self: print("调用 say() 实例方法")
        return super().__new__(cls,name,bases,attrs)
```
此程序中，首先可以断定 FirstMetaClass 是一个类。其次，由于该类继承自 type 类，并且内部实现了 __new__() 方法，因此可以断定 FirstMetaCLass 是一个元类。

可以看到，在这个元类的`__new__()`方法中，手动添加了一个 name 属性和 say() 方法。这意味着，通过 FirstMetaClass 元类创建的类，会额外添加 name 属性和 say() 方法。通过如下代码，可以验证这个结论：
```py
#定义类时，指定元类
class CLanguage(object,metaclass=FirstMetaClass):
  pass
clangs = CLanguage()
print(clangs.name)
clangs.say()
```
可以看到，在创建类时，通过在标注父类的同时指定元类（格式为metaclass=元类名），则当 Python 解释器在创建这该类时，FirstMetaClass 元类中的 __new__ 方法就会被调用，从而实现动态修改类属性或者类方法的目的。

运行上面的程序，输出结果为：
```
C语言中文网
调用 say() 实例方法
```
显然，`FirstMetaClass`元类的`__new__()`方法动态地为`Clanguage`类添加了`name`属性和`say()`方法，因此，即便该类在定义时是空类，它也依然有`name`属性和`say()`方法。

对于`MetaClass`元类，它多用于创建 API，因此我们几乎不会使用到它。
# 多态
我们都知道，Python 是弱类型语言，其最明显的特征是在使用变量时，无需为其指定具体的数据类型。这会导致一种情况，即同一变量可能会被先后赋值不同的类对象，例如：
```py
class CLanguage:
  def say(self):
    print("赋值的是 CLanguage 类的实例对象")
class CPython:
  def say(self):
    print("赋值的是 CPython 类的实例对象")
a = CLanguage()
a.say()
a = CPython()
a.say()
```
运行结果为：
```
赋值的是 CLanguage 类的实例对象
赋值的是 CPython 类的实例对象
```
可以看到，`a`可以被先后赋值为`CLanguage`类和`CPython`类的对象，但这并不是多态。类的多态特性，还要满足以下 2 个前提条件：
* 继承：多态一定是发生在子类和父类之间；
* 重写：子类重写了父类的方法。

下面程序是对上面代码的改写：
```py
class CLanguage:
  def say(self):
    print("调用的是 Clanguage 类的say方法")
class CPython(CLanguage):
  def say(self):
    print("调用的是 CPython 类的say方法")
class CLinux(CLanguage):
  def say(self):
    print("调用的是 CLinux 类的say方法")
a = CLanguage()
a.say()
a = CPython()
a.say()
a = CLinux()
a.say()
```
程序执行结果为：
```
调用的是 Clanguage 类的say方法
调用的是 CPython 类的say方法
调用的是 CLinux 类的say方法
```
可以看到，`CPython`和`CLinux`都继承自`CLanguage`类，且各自都重写了父类的`say()`方法。从运行结果可以看出，同一变量`a`在执行同一个`say()`方法时，由于`a`实际表示不同的类实例对象，因此`a.say()`调用的并不是同一个类中的`say()`方法，这就是多态。

其实，Python 在多态的基础上，衍生出了一种更灵活的编程机制。继续对上面的程序进行改写：
```py
class WhoSay:
  def say(self,who):
    who.say()
class CLanguage:
  def say(self):
    print("调用的是 Clanguage 类的say方法")
class CPython(CLanguage):
  def say(self):
    print("调用的是 CPython 类的say方法")
class CLinux(CLanguage):
  def say(self):
    print("调用的是 CLinux 类的say方法")
a = WhoSay()
#调用 CLanguage 类的 say() 方法
a.say(CLanguage())
#调用 CPython 类的 say() 方法
a.say(CPython())
#调用 CLinux 类的 say() 方法
a.say(CLinux())
```
程序执行结果为：
```
调用的是 Clanguage 类的say方法
调用的是 CPython 类的say方法
调用的是 CLinux 类的say方法
```
此程序中，通过给`WhoSay`类中的`say()`函数添加一个`who`参数，其内部利用传入的`who`调用`say()`方法。这意味着，当调用`WhoSay`类中的`say()`方法时，我们传给`who`参数的是哪个类的实例对象，它就会调用那个类中的`say()`方法。
# 枚举类
一些具有特殊含义的类，其实例化对象的个数往往是固定的，比如用一个类表示月份，则该类的实例对象最多有 12 个。对于这些实例化对象个数固定的类，可以用枚举类来定义。
```py
from enum import Enum
class Color(Enum):
  # 为序列值指定value值
  red = 1
  green = 2
  blue = 3
```
如果想将一个类定义为枚举类，只需要令其继承自`enum`模块中的`Enum`类即可。例如在上面程序中，`Color`类继承自`Enum`类，则证明这是一个枚举类。

在`Color`枚举类中，`red、green、blue`都是该类的成员（可以理解为是类变量）。注意，枚举类的每个成员都由 2 部分组成，分别为`name`和`value`，其中`name`属性值为该枚举值的变量名（如`red`），`value`代表该枚举值的序号（序号通常从 1 开始）。

和普通类的用法不同，枚举类不能用来实例化对象，但这并不妨碍我们访问枚举类中的成员。访问枚举类成员的方式有多种：
```py
#调用枚举成员的 3 种方式
print(Color.red)
print(Color['red'])
print(Color(1))
#调取枚举成员中的 value 和 name
print(Color.red.value)
print(Color.red.name)
#遍历枚举类中所有成员的 2 种方式
for color in Color:
  print(color)
```
程序输出结果为：
```
Color.red
Color.red
Color.red
1
red
Color.red
Color.green
Color.blue
```
枚举类成员之间不能比较大小，但可以用`==`或者`is`进行比较是否相等：
```py
print(Color.red == Color.green) # Flase
print(Color.red.name is Color.green.name) # Flase
```
需要注意的是，枚举类中各个成员的值，不能在类的外部做任何修改，也就是说，下面语法的做法是错误的：
```py
Color.red = 4
```
除此之外，该枚举类还提供了一个`__members__`属性，该属性是一个包含枚举类中所有成员的字典，通过遍历该属性，也可以访问枚举类中的各个成员。
```py
for name,member in Color.__members__.items():
  print(name,"->",member)
```
输出结果为：
```
red -> Color.red
green -> Color.green
blue -> Color.blue
```
值得一提的是，Python 枚举类中各个成员必须保证`name`互不相同，但`value`可以相同：
```py
from enum import Enum
class Color(Enum):
  # 为序列值指定value值
  red = 1
  green = 1
  blue = 3
print(Color['green'])
```
输出结果为：
```
Color.red
```
可以看到，`Color`枚举类中`red`和`green`具有相同的值（都是 1），Python 允许这种情况的发生，它会将`green`当做是`red`的别名，因此当访问`green`成员时，最终输出的是`red`。

在实际编程过程中，如果想避免发生这种情况，可以借助`@unique`装饰器，这样当枚举类中出现相同值的成员时，程序会报`ValueError`错误。
```py
#引入 unique
from enum import Enum,unique
#添加 unique 装饰器
@unique
class Color(Enum):
  # 为序列值指定value值
  red = 1
  green = 1
  blue = 3
print(Color['green'])
```
运行程序会报错：
```
Traceback (most recent call last):
  File "D:\python3.6\demo.py", line 3, in <module>
    class Color(Enum):
  File "D:\python3.6\lib\enum.py", line 834, in unique
    (enumeration, alias_details))
ValueError: duplicate values found in <enum 'Color'>: green -> red
```
除了通过继承`Enum`类的方法创建枚举类，还可以使用`Enum()`函数创建枚举类。
```py
from enum import Enum
#创建一个枚举类
Color = Enum("Color",('red','green','blue'))
#调用枚举成员的 3 种方式
print(Color.red)
print(Color['red'])
print(Color(1))
#调取枚举成员中的 value 和 name
print(Color.red.value)
print(Color.red.name)
#遍历枚举类中所有成员的 2 种方式
for color in Color:
  print(color)
```
`Enum()`函数可接受 2 个参数，第一个用于指定枚举类的类名，第二个参数用于指定枚举类中的多个成员。

如上所示，仅通过一行代码，即创建了一个和前面的`Color`类相同的枚举类。运行程序，其输出结果为：
```
Color.red
Color.red
Color.red
1
red
Color.red
Color.green
Color.blue
```