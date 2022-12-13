



# class：定义类
Python 中定义一个类使用`class`关键字实现：
```
class 类名：
  多个（≥0）类属性...
  多个（≥0）类方法...
```
无论是类属性还是类方法，对于类来说，它们都不是必需的。另外，Python 类中属性和方法所在的位置是任意的，即它们之间并没有固定的前后次序。

其实，类属性指的就是包含在类中的变量；而类方法指的是包含类中的函数。换句话说，类属性和类方法其实分别是包含类中的变量和函数的别称。

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
```
class Empty:
  pass
```
可以看到，如果一个类没有任何类属性和类方法，那么可以直接用`pass`关键字作为类体即可。
# __init__()类构造方法
在创建类时，我们可以手动添加一个`__init__()`方法，该方法是一个特殊的类实例方法，称为构造方法（或构造函数）。

构造方法用于创建对象时使用，每当创建一个类的实例对象时，Python 解释器都会自动调用它。
···
def __init__(self,...):
  代码块
···
注意，此方法的方法名中，开头和结尾各有 2 个下划线，且中间不能有空格。Python 中很多这种以双下划线开头、双下划线结尾的方法，都具有特殊的意义。

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

在`__init__()`构造方法中，除了`self`参数外，还可以自定义一些参数，参数之间使用逗号“,”进行分割。
```py
class CLanguage:
  '''这是一个学习Python定义的一个类'''
  def __init__(self,name,add):
    print(name,"的英文名为:",add)
#创建 add 对象，并传递参数给构造函数
add = CLanguage("小明","xiaoming")
```
注意，由于创建对象时会调用类的构造方法，如果构造函数有多个参数时，需要手动传递参数，传递方式如代码中所示。

可以看到，虽然构造方法中有`self、name、add`3 个参数，但实际需要传参的仅有`name`和`add`，也就是说，`self`不需要手动传递参数。
# 类对象的创建和使用
## 类的实例化
对已定义好的类进行实例化，其语法格式如下：
```
类名(参数)
```
定义类时，如果没有手动添加`__init__()`构造方法，又或者添加的`__init__()`中仅有一个`self`参数，则创建类对象时的参数可以省略不写。

例如，如下代码创建了名为 CLanguage 的类，并对其进行了实例化：
```py
class CLanguage :
  # 下面定义了2个类变量
  name = "小明"
  add = "xiaoming"
  def __init__(self,name,add):
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
注意，对象名和变量名以及方法名之间用点 "." 连接。
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
可以看到，通过直接增加一个新的实例变量并为其赋值，就成功地为`clanguage`对象添加了`money`变量。

既然能动态添加，那么是否能动态删除呢？答案是肯定的，使用`del`语句即可实现：
```py
#删除新添加的 money 实例变量
del clanguage.money
#再次尝试输出 money，此时会报错
print(clanguage.money)
运行程序会发现，结果显示 AttributeError 错误：
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
上面的第 5 行和第 11 行代码分别使用函数、lambda 表达式为`clanguage`对象动态增加了方法，但对于动态增加的方法，Python 不会自动将方法调用者绑定到它们的第一个参数，因此程序必须手动为第一个参数传入参数值，如上面程序中 ① 号、② 号代码所示。

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
在定义类的过程中，无论是显式创建类的构造方法，还是向类中添加实例方法，都要求将 self 参数作为方法的第一个参数。例如，定义一个 Person 类：
```py
class Person:
  def __init__(self):
    print("正在执行构造方法")
  # 定义一个study()实例方法
  def study(self,name):
    print(name,"正在学Python")
```
事实上，Python 只是规定，无论是构造方法还是实例方法，最少要包含一个参数，并没有规定该参数的具体名称。之所以将其命名为`self`，只是程序员之间约定俗成的一种习惯，遵守这个约定，可以使我们编写的代码具有更好的可读性（大家一看到`self`，就知道它的作用）。

那么，`self`参数的具体作用是什么呢？打个比方，如果把类比作造房子的图纸，那么类实例化后的对象是真正可以住的房子。根据一张图纸（类），我们可以设计出成千上万的房子（类对象），每个房子长相都是类似的（都有相同的类变量和类方法），但它们都有各自的主人，那么如何对它们进行区分呢？

当然是通过`self`参数，它就相当于每个房子的门钥匙，可以保证每个房子的主人仅能进入自己的房子（每个类对象只能调用自己的类变量和类方法）。
其实 Python 类方法中的`self`参数就相当于 C++ 中的`this`指针。

也就是说，同一个类可以产生多个对象，当某个对象调用类方法时，该对象会把自身的引用作为第一个参数自动传给该方法，换句话说，Python 会自动绑定类方法的第一个参数指向调用该方法的对象。如此，Python解释器就能知道到底要操作哪个对象的方法了。

因此，程序在调用实例方法和构造方法时，不需要手动为第一个参数传值。
```py
class Person:
  def __init__(self):
    print("正在执行构造方法")
  # 定义一个study()实例方法
  def study(self):
    print(self,"正在学Python")
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

不仅如此，类方法也可细分为实例方法、静态方法和类方法。
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

类变量的特点是，所有类的实例化对象都同时共享类变量，也就是说，类变量在所有实例化对象中是作为公用资源存在的。类方法的调用方式有 2 种，既可以使用类名直接调用，也可以使用类的实例化对象调用。
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
除了实例变量，类方法中还可以定义局部变量。和前者不同，局部变量直接以“变量名=值”的方式进行定义：
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
和类属性一样，类方法也可以进行更细致的划分，具体可分为类方法、实例方法和静态方法。

和类属性的分类不同，区分这 3 种类方法是非常简单的，即采用`@classmethod`修饰的方法为类方法；采用`@staticmethod`修饰的方法为静态方法；不用任何修改的方法为实例方法。

其中`@classmethod`和`@staticmethod`都是函数装饰器。
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

和实例方法最大的不同在于，类方法需要使用`＠classmethod`修饰符进行修饰：
```py
class CLanguage:
  #类构造方法，也属于实例方法
  def __init__(self):
    self.name = "C语言中文网"
    self.add = "http://c.biancheng.net"
  #下面定义了一个类方法
  @classmethod
  def info(cls):
    print("正在调用类方法",cls)
```
注意，如果没有`＠classmethod`，则 Python 解释器会将`fly()`方法认定为实例方法，而不是类方法。

类方法推荐使用类名直接调用，当然也可以使用实例对象来调用（不推荐）。
```py
#使用类名直接调用类方法
CLanguage.info()
#使用类对象调用类方法
clang = CLanguage()
clang.info()
运行结果为：
正在调用类方法 <class '__main__.CLanguage'>
正在调用类方法 <class '__main__.CLanguage'>
```
## 类静态方法
静态方法，其实就是函数，和函数唯一的区别是，静态方法定义在类这个空间（类命名空间）中，而函数则定义在程序所在的空间（全局命名空间）中。

静态方法没有类似`self、cls`这样的特殊参数，因此 Python 解释器不会对它包含的参数做任何类或对象的绑定。也正因为如此，类的静态方法中无法调用任何类属性和类方法。

静态方法需要使用`＠staticmethod`修饰：
```py
class CLanguage:
  @staticmethod
  def info(name,add):
    print(name,add)
```
静态方法的调用，既可以使用类名，也可以使用类对象：
```py
#使用类名直接调用静态方法
CLanguage.info("小明","xiaoming")
#使用类对象调用静态方法
clang = CLanguage()
clang.info("小红","xiaohong")
```
在实际编程中，几乎不会用到类方法和静态方法，因为我们完全可以使用函数代替它们实现想要的功能，但在一些特殊的场景中（例如工厂模式中），使用类方法和静态方法也是很不错的选择。
# 类调用实例方法
实例方法的调用方式其实有 2 种，既可以采用类对象调用，也可以直接通过类名调用。

通常情况下，我们习惯使用类对象调用类中的实例方法。但如果想用类调用实例方法，不能像如下这样：
```py
class CLanguage:
  def info(self):
    print("我正在学 Python")
#通过类名直接调用实例方法
CLanguage.info()
运行上面代码，程序会报出如下错误：
Traceback (most recent call last):
  File "D:\python3.6\demo.py", line 5, in <module>
    CLanguage.info()
TypeError: info() missing 1 required positional argument: 'self'
```
其中，最后一行报错信息提示我们，调用 info() 类方式时缺少给 self 参数传参。这意味着，和使用类对象调用实例方法不同，通过类名直接调用实例方法时，Python 并不会自动给 self 参数传值。

self 参数需要的是方法的实际调用者（是类对象），而这里只提供了类名，当然无法自动传值。

因此，如果想通过类名直接调用实例方法，就必须手动为 self 参数传值。
```py
class CLanguage:
  def info(self):
    print("我正在学 Python")
clang = CLanguage()
#通过类名直接调用实例方法
CLanguage.info(clang)
```
再次运行程序，结果为：
```
我正在学 Python
```
可以看到，通过手动将`clang`这个类对象传给了`self`参数，使得程序得以正确执行。实际上，这里调用实例方法的形式完全是等价于`clang.info()`。

值得一提的是，上面的报错信息只是让我们手动为`self`参数传值，但并没有规定必须传一个该类的对象，其实完全可以任意传入一个参数：
```py
class CLanguage:
    def info(self):
        print(self,"正在学 Python")
#通过类名直接调用实例方法
CLanguage.info("zhangsan")
运行结果为：
zhangsan 正在学 Python
```
可以看到，"zhangsan" 这个字符串传给了 info() 方法的 self 参数。显然，无论是 info() 方法中使用 self 参数调用其它类方法，还是使用 self 参数定义新的实例变量，胡乱的给 self 参数传参都将会导致程序运行崩溃。

总的来说，Python 中允许使用类名直接调用实例方法，但必须手动为该方法的第一个`self`参数传递参数，这种调用方法的方式被称为“非绑定方法”。

用类的实例对象访问类成员的方式称为绑定方法，而用类名调用类成员的方式称为非绑定方法。
# 描述符

# property()函数
# 封装
# 继承
# 父类方法重写
# super()函数
# __slots__：限制类实例动态添加属性和方法
# type()函数：动态创建类
# MetaClass元类
# 多态
# 枚举类

