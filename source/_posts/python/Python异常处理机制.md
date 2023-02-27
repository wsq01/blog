


编写程序时遇到的错误可大致分为 2 类，分别为语法错误和运行时错误。
# Python语法错误
语法错误，也就是解析代码时出现的错误。当代码不符合 Python 语法规则时，Python解释器在解析时就会报出`SyntaxError`语法错误，与此同时还会明确指出最早探测到错误的语句。

语法错误多是开发者疏忽导致的，属于真正意义上的错误，是解释器无法容忍的，因此，只有将程序中的所有语法错误全部纠正，程序才能执行。
# Python运行时错误
运行时错误，即程序在语法上都是正确的，但在运行时发生了错误。
```
a = 1/0
```
上面这句代码的意思是“用 1 除以 0，并赋值给 a 。因为 0 作除数是没有意义的，所以运行后会产生如下错误：
```
>>> a = 1/0
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    a = 1/0
ZeroDivisionError: division by zero
```
以上运行输出结果中，前两段指明了错误的位置，最后一句表示出错的类型。在 Python 中，把这种运行时产生错误的情况叫做异常。这种异常情况还有很多，常见的几种异常情况：

| 异常类型          |  含义 |
| :--: | :--: |
| AssertionError    | 当 assert 关键字后的条件为假时，程序运行会停止并抛出 AssertionError 异常 |
| AttributeError    | 当试图访问的对象属性不存在时抛出的异常 |
| IndexError        | 索引超出序列范围会引发此异常 |
| KeyError          | 字典中查找一个不存在的关键字时引发此异常 |
| NameError         | 尝试访问一个未声明的变量时，引发此异常 |
| TypeError         | 不同类型数据之间的无效操作 |
| ZeroDivisionError	| 除法运算中除数为 0 引发此异常	 |

当一个程序发生异常时，代表该程序在执行时出现了非正常的情况，无法再执行下去。默认情况下，程序是要终止的。如果要避免程序退出，可以使用捕获异常的方式获取这个异常的名称，再通过其他的逻辑代码让程序继续运行，这种根据异常做出的逻辑处理叫作异常处理。
# try except异常处理
Python 中，用`try except`语句块捕获并处理异常：
```
try:
  可能产生异常的代码块
except [ (Error1, Error2, ... ) [as e] ]:
  处理异常的代码块1
except [ (Error3, Error4, ... ) [as e] ]:
  处理异常的代码块2
except  [Exception]:
  处理其它异常
```
该格式中，[] 括起来的部分可以使用，也可以省略。其中各部分的含义如下：
* `(Error1, Error2,...) 、(Error3, Error4,...)`：其中，`Error1、Error2、Error3`和`Error4`都是具体的异常类型。显然，一个`except`块可以同时处理多种异常。
* `[as e]`：作为可选参数，表示给异常类型起一个别名`e`，这样做的好处是方便在`except`块中调用异常类型。
* `[Exception]`：作为可选参数，可以代指程序可能发生的所有异常情况，其通常用在最后一个`except`块。

从`try except`的基本语法格式可以看出，`try`块有且仅有一个，但`except`代码块可以有多个，且每个`except`块都可以同时处理多种异常。
当程序发生不同的意外情况时，会对应特定的异常类型，Python 解释器会根据该异常类型选择对应的`except`块来处理该异常。

`try except`语句的执行流程如下：
* 首先执行`try`中的代码块，如果执行过程中出现异常，系统会自动生成一个异常类型，并将该异常提交给 Python 解释器，此过程称为捕获异常。
* 当 Python 解释器收到异常对象时，会寻找能处理该异常对象的`except`块，如果找到合适的`except`块，则把该异常对象交给该`except`块处理，这个过程被称为处理异常。如果 Python 解释器找不到处理异常的`except`块，则程序运行终止，Python 解释器也将退出。

事实上，不管程序代码块是否处于`try`块中，甚至包括`except`块中的代码，只要执行该代码块时出现了异常，系统都会自动生成对应类型的异常。但是，如果此段程序没有用`try`包裹，又或者没有为该异常配置处理它的`except`块，则 Python 解释器将无法处理，程序就会停止运行；反之，如果程序发生的异常经`try`捕获并由`except`处理完成，则程序可以继续执行。
```py
try:
  a = int(input("输入被除数："))
  b = int(input("输入除数："))
  c = a / b
  print("您输入的两个数相除的结果是：", c )
except (ValueError, ArithmeticError):
  print("程序发生了数字格式异常、算术异常之一")
except :
  print("未知异常")
print("程序继续运行")
```
程序运行结果为：
```
输入被除数：a
程序发生了数字格式异常、算术异常之一
程序继续运行
```
上面程序中，第 6 行代码使用了（`ValueError, ArithmeticError`）来指定所捕获的异常类型，这就表明该`except`块可以同时捕获这 2 种类型的异常；第 8 行代码只有`except`关键字，并未指定具体要捕获的异常类型，这种省略异常类的`except`语句也是合法的，它表示可捕获所有类型的异常，一般会作为异常捕获的最后一个`except`块。

除此之外，由于`try`块中引发了异常，并被`except`块成功捕获，因此程序才可以继续执行，才有了“程序继续运行”的输出结果。
## 获取特定异常的有关信息
由于一个`except`可以同时处理多个异常，那么我们如何知道当前处理的到底是哪种异常呢？

其实，每种异常类型都提供了如下几个属性和方法，通过调用它们，就可以获取当前处理异常类型的相关信息：
* `args`：返回异常的错误编号和描述字符串；
* `str(e)`：返回异常信息，但不包括异常信息的类型；
* `repr(e)`：返回较全的异常信息，包括异常信息的类型。

```py
try:
    1/0
except Exception as e:
  # 访问异常的错误编号和详细信息
  print(e.args)
  print(str(e))
  print(repr(e))
```
输出结果为：
```
('division by zero',)
division by zero
ZeroDivisionError('division by zero',)
```
从程序中可以看到，由于`except`可能接收多种异常，因此为了操作方便，可以直接给每一个进入到此`except`块的异常，起一个统一的别名 e。
# try except else
在原本的`try except`结构的基础上，Python 异常处理机制还提供了一个`else`块，也就是原有`try except`语句的基础上再添加一个`else`块，即`try except else`结构。

使用`else`包裹的代码，只有当`try`块没有捕获到任何异常时，才会得到执行；反之，如果`try`块捕获到异常，即便调用对应的`except`处理完异常，`else`块中的代码也不会得到执行。
```py
try:
  result = 20 / int(input('请输入除数:'))
  print(result)
except ValueError:
  print('必须输入整数')
except ArithmeticError:
  print('算术错误，除数不能为 0')
else:
  print('没有出现异常')
print("继续执行")
```
可以看到，在原有`try except`的基础上，我们为其添加了`else`块。现在执行该程序：
```
请输入除数:4
5.0
没有出现异常
继续执行
```
如上所示，当我们输入正确的数据时，`try`块中的程序正常执行，Python 解释器执行完`try`块中的程序之后，会继续执行`else`块中的程序，继而执行后续的程序。

既然 Python 解释器按照顺序执行代码，那么`else`块有什么存在的必要呢？直接将`else`块中的代码编写在`try except`块的后面，不是一样吗？

当然不一样，现在再次执行上面的代码：
```
请输入除数:a
必须输入整数
继续执行
```
可以看到，当我们试图进行非法输入时，程序会发生异常并被`try`捕获，Python 解释器会调用相应的`except`块处理该异常。但是异常处理完毕之后，Python 解释器并没有接着执行 `else`块中的代码，而是跳过`else`，去执行后续的代码。

也就是说，`else`的功能，只有当`try`块捕获到异常时才能显现出来。在这种情况下，`else`块中的代码不会得到执行的机会。而如果我们直接把`else`块去掉，将其中的代码编写到`try except`的后面：
```py
try:
  result = 20 / int(input('请输入除数:'))
  print(result)
except ValueError:
  print('必须输入整数')
except ArithmeticError:
  print('算术错误，除数不能为 0')
print('没有出现异常')
print("继续执行")
```
程序执行结果为：
```
请输入除数:a
必须输入整数
没有出现异常
继续执行
```
可以看到，如果不使用`else`块，`try`块捕获到异常并通过`except`成功处理，后续所有程序都会依次被执行。
# try except finally：资源回收
Python 异常处理机制还提供了一个`finally`语句，通常用来为`try`块中的程序做扫尾清理工作。

> 注意，和`else`语句不同，`finally`只要求和`try`搭配使用，而至于该结构中是否包含`except`以及`else`，对于`finally`不是必须的（`else`必须和`try except`搭配使用）。

在整个异常处理机制中，`finally`语句的功能是：无论`try`块是否发生异常，最终都要进入`finally`语句，并执行其中的代码块。

基于`finally`语句的这种特性，在某些情况下，当`try`块中的程序打开了一些物理资源（文件、数据库连接等）时，由于这些资源必须手动回收，而回收工作通常就放在`finally`块中。

Python 垃圾回收机制，只能帮我们回收变量、类对象占用的内存，而无法自动完成类似关闭文件、数据库连接等这些的工作。

回收这些物理资源，必须使用`finally`块吗？当然不是，但使用`finally`块是比较好的选择。首先，`try`块不适合做资源回收工作，因为一旦`try`块中的某行代码发生异常，则其后续的代码将不会得到执行；其次`except`和`else`也不适合，它们都可能不会得到执行。而`finally`块中的代码，无论`try`块是否发生异常，该块中的代码都会被执行。
```py
try:
  a = int(input("请输入 a 的值:"))
  print(20/a)
except:
  print("发生异常！")
else:
  print("执行 else 块中的代码")   
finally :
  print("执行 finally 块中的代码")
```
运行此程序：
```
请输入 a 的值:4
5.0
执行 else 块中的代码
执行 finally 块中的代码
```
可以看到，当`try`块中代码为发生异常时，`except`块不会执行，`else`块和`finally`块中的代码会被执行。

再次运行程序：
```
请输入 a 的值:a
发生异常！
执行 finally 块中的代码
```
可以看到，当`try`块中代码发生异常时，`except`块得到执行，而`else`块中的代码将不执行，`finally`块中的代码仍然会被执行。

`finally`块的强大还远不止此，即便当`try`块发生异常，且没有合适和`except`处理异常时，`finally`块中的代码也会得到执行。
```py
try:
  #发生异常
  print(20/0)
finally :
  print("执行 finally 块中的代码")
```
程序执行结果为：
```
执行 finally 块中的代码
Traceback (most recent call last):
  File "D:\python3.6\1.py", line 3, in <module>
    print(20/0)
ZeroDivisionError: division by zero
```
可以看到，当`try`块中代码发生异常，导致程序崩溃时，在崩溃前 Python 解释器也会执行`finally`块中的代码。
# raise
Python 允许我们在程序中手动设置异常，使用`raise`语句即可。
```
raise [exceptionName [(reason)]]
```
其中，用`[]`括起来的为可选参数，其作用是指定抛出的异常名称，以及异常信息的相关描述。如果可选参数全部省略，则`raise`会把当前错误原样抛出；如果仅省略 (`reason`)，则在抛出异常时，将不附带任何的异常描述信息。

也就是说，`raise`语句有如下三种常用的用法：
* `raise`：单独一个`raise`。该语句引发当前上下文中捕获的异常（比如在`except`块中），或默认引发`RuntimeError`异常。
* `raise`异常类名称：`raise`后带一个异常类名称，表示引发执行类型的异常。
* `raise`异常类名称(描述信息)：在引发指定类型的异常的同时，附带异常的描述信息。

显然，每次执行`raise`语句，都只能引发一次执行的异常。首先，我们来测试一下以上 3 种 raise 的用法：
```
>>> raise
Traceback (most recent call last):
  File "<pyshell#1>", line 1, in <module>
    raise
RuntimeError: No active exception to reraise
>>> raise ZeroDivisionError
Traceback (most recent call last):
  File "<pyshell#0>", line 1, in <module>
    raise ZeroDivisionError
ZeroDivisionError
>>> raise ZeroDivisionError("除数不能为零")
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    raise ZeroDivisionError("除数不能为零")
ZeroDivisionError: 除数不能为零
```
当然，我们手动让程序引发异常，很多时候并不是为了让其崩溃。事实上，`raise`语句引发的异常通常用`try except（else finally）`异常处理结构来捕获并进行处理。
```py
try:
  a = input("输入一个数：")
  #判断用户输入的是否为数字
  if(not a.isdigit()):
    raise ValueError("a 必须是数字")
except ValueError as e:
  print("引发异常：",repr(e))
```
程序运行结果为：
```
输入一个数：a
引发异常： ValueError('a 必须是数字',)
```
可以看到，当用户输入的不是数字时，程序会进入 if 判断语句，并执行 raise 引发 ValueError 异常。但由于其位于 try 块中，因为 raise 抛出的异常会被 try 捕获，并由 except 块进行处理。

因此，虽然程序中使用了 raise 语句引发异常，但程序的执行是正常的，手动抛出的异常并不会导致程序崩溃。
## raise 不需要参数
正如前面所看到的，在使用 raise 语句时可以不带参数：
```py
try:
  a = input("输入一个数：")
  if(not a.isdigit()):
    raise ValueError("a 必须是数字")
except ValueError as e:
  print("引发异常：",repr(e))
  raise
```
程序执行结果为：
```
输入一个数：a
引发异常： ValueError('a 必须是数字',)
Traceback (most recent call last):
  File "D:\python3.6\1.py", line 4, in <module>
    raise ValueError("a 必须是数字")
ValueError: a 必须是数字
```
这里重点关注位于`except`块中的`raise`，由于在其之前我们已经手动引发了`ValueError`异常，因此这里当再使用`raise`语句时，它会再次引发一次。

当在没有引发过异常的程序使用无参的`raise`语句时，它默认引发的是`RuntimeError`异常。
```py
try:
  a = input("输入一个数：")
  if(not a.isdigit()):
      raise
except RuntimeError as e:
  print("引发异常：",repr(e))
```
程序执行结果为：
```
输入一个数：a
引发异常： RuntimeError('No active exception to reraise',)
```
# sys.exc_info()方法：获取异常信息
在实际调试程序的过程中，有时只获得异常的类型是远远不够的，还需要借助更详细的异常信息才能解决问题。

捕获异常时，有 2 种方式可获得更多的异常信息，分别是：
* 使用`sys`模块中的`exc_info`方法；
* 使用`traceback`模块中的相关函数。

模块`sys`中，有两个方法可以返回异常的全部信息，分别是`exc_info()`和`last_traceback()`，这两个函数有相同的功能和用法。

`exc_info()`方法会将当前的异常信息以元组的形式返回，该元组中包含 3 个元素，分别为`type、value`和`traceback`，它们的含义分别是：
* `type`：异常类型的名称，它是`BaseException`的子类
* `value`：捕获到的异常实例。
* `traceback`：是一个`traceback`对象。

```py
#使用 sys 模块之前，需使用 import 引入
import sys
try:
  x = int(input("请输入一个被除数："))
  print("30除以",x,"等于",30/x)
except:
  print(sys.exc_info())
  print("其他异常...")
```
当输入 0 时，程序运行结果为：
```
请输入一个被除数：0
(<class 'ZeroDivisionError'>, ZeroDivisionError('division by zero',), <traceback object at 0x000001FCF638DD48>)
其他异常...
```
输出结果中，第 2 行是抛出异常的全部信息，这是一个元组，有 3 个元素，第一个元素是一个`ZeroDivisionError`类；第 2 个元素是异常类型`ZeroDivisionError`类的一个实例；第 3 个元素为一个`traceback`对象。其中，通过前 2 个元素可以看出抛出的异常类型以及描述信息，对于第 3 个元素，是一个`traceback`对象，无法直接看出有关异常的信息，还需要对其做进一步处理。

要查看`traceback`对象包含的内容，需要先引进`traceback`模块，然后调用`traceback`模块中的`print_tb`方法，并将`sys.exc_info()`输出的`traceback`对象作为参数参入。
```py
#使用 sys 模块之前，需使用 import 引入
import sys
#引入traceback模块
import traceback
try:
  x = int(input("请输入一个被除数："))
  print("30除以",x,"等于",30/x)
except:
  #print(sys.exc_info())
  traceback.print_tb(sys.exc_info()[2])
  print("其他异常...")
```
输入 0，程序运行结果为：
```
请输入一个被除数：0
  File "C:\Users\mengma\Desktop\demo.py", line 7, in <module>
    print("30除以",x,"等于",30/x)
其他异常...
```
可以看到，输出信息中包含了更多的异常信息，包括文件名、抛出异常的代码所在的行数、抛出异常的具体代码。
# traceback模块：获取异常信息
除了使用`sys.exc_info()`方法获取更多的异常信息之外，还可以使用`traceback`模块，该模块可以用来查看异常的传播轨迹，追踪异常触发的源头。
```py
class SelfException(Exception):
  pass
def main():
  firstMethod()
def firstMethod():
  secondMethod()
def secondMethod():
  thirdMethod()
def thirdMethod():
  raise SelfException("自定义异常信息")
main()
```
运行上面程序，将会看到如下所示的结果：
```
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\1.py", line 11, in <module>
    main()
  File "C:\Users\mengma\Desktop\1.py", line 4, in main           <--mian函数
    firstMethod()
  File "C:\Users\mengma\Desktop\1.py", line 6, in firstMethod    <--第三个
    secondMethod()
  File "C:\Users\mengma\Desktop\1.py", line 8, in secondMethod   <--第二个
    thirdMethod()
  File "C:\Users\mengma\Desktop\1.py", line 10, in thirdMethod   <--异常源头
    raise SelfException("自定义异常信息")
SelfException: 自定义异常信息
```
从输出结果可以看出，异常从`thirdMethod()`函数开始触发，传到`secondMethod()`函数，再传到`firstMethod()`函数，最后传到`main()`函数，在`main()`函数止，这个过程就是整个异常的传播轨迹。

当应用程序运行时，经常会发生一系列函数或方法调用，从而形成“函数调用栈”。异常的传播则相反，只要异常没有被完全捕获（包括异常没有被捕获，或者异常被处理后重新引发了新异常），异常就从发生异常的函数或方法逐渐向外传播，首先传给该函数或方法的调用者，该函数或方法的调用者再传给其调用者，直至最后传到 Python 解释器，此时 Python 解释器会中止该程序，并打印异常的传播轨迹信息。

其实，上面程序的运算结果显示的异常传播轨迹信息非常清晰，它记录了应用程序中执行停止的各个点。最后一行信息详细显示了异常的类型和异常的详细消息。从这一行向上，逐个记录了异常发生源头、异常依次传播所经过的轨迹，并标明异常发生在哪个文件、哪一行、哪个函数处。

使用`traceback`模块查看异常传播轨迹，首先需要将`traceback`模块引入，该模块提供了如下两个常用方法：
* `traceback.print_exc()`：将异常传播轨迹信息输出到控制台或指定文件中。
* `format_exc()`：将异常传播轨迹信息转换成字符串。

从上面方法看不出它们到底处理哪个异常的传播轨迹信息。实际上我们常用的`print_exc()`是`print_exc([limit[, file]])`省略了`limit、file`两个参数的形式。而`print_exc([limit[, file]])`的完整形式是`print_exception(etype, value, tb[,limit[, file]])`，在完整形式中，前面三个参数用于分别指定异常的如下信息：
* `etype`：指定异常类型；
* `value`：指定异常值；
* `tb`：指定异常的`traceback`信息；

当程序处于`except`块中时，该`except`块所捕获的异常信息可通过`sys`对象来获取，其中`sys.exc_type、sys.exc_value、sys.exc_traceback`就代表当前`except`块内的异常类型、异常值和异常传播轨迹。

简单来说，`print_exc([limit[, file]])`相当于如下形式：
```
print_exception(sys.exc_etype, sys.exc_value, sys.exc_tb[, limit[, file]])
```
也就是说，使用`print_exc([limit[, file]])`会自动处理当前`except`块所捕获的异常。该方法还涉及两个参数：
* `limit`：用于限制显示异常传播的层数，比如函数`A`调用函数`B`，函数`B`发生了异常，如果指定`limit=1`，则只显示函数`A`里面发生的异常。如果不设置`limit`参数，则默认全部显示。
* `file`：指定将异常传播轨迹信息输出到指定文件中。如果不指定该参数，则默认输出到控制台。

借助于`traceback`模块的帮助，我们可以使用`except`块捕获异常，并在其中打印异常传播信息，包括把它输出到文件中。
```py
# 导入trackback模块
import traceback
class SelfException(Exception): pass
def main():
  firstMethod()
def firstMethod():
  secondMethod()
def secondMethod():
  thirdMethod()
def thirdMethod():
  raise SelfException("自定义异常信息")
try:
  main()
except:
  # 捕捉异常，并将异常传播信息输出控制台
  traceback.print_exc()
  # 捕捉异常，并将异常传播信息输出指定文件中
  traceback.print_exc(file=open('log.txt', 'a'))
```
