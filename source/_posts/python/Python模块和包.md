


# 导入模块
`import`的用法主要有以下两种：
`import 模块名1 [as 别名1], 模块名2 [as 别名2]，…`：使用这种语法格式的`import`语句，会导入指定模块中的所有成员（包括变量、函数、类等）。不仅如此，当需要使用模块中的成员时，需用该模块名（或别名）作为前缀，否则 Python 解释器会报错。
`from 模块名 import 成员名1 [as 别名1]，成员名2 [as 别名2]，…`：使用这种语法格式的`import`语句，只会导入模块中指定的成员，而不是全部成员。同时，当程序中使用该成员时，无需附加任何前缀，直接使用成员名（或别名）即可。

注意，用`[]`括起来的部分，可以使用，也可以省略。

其中，第二种`import`语句也可以导入指定模块中的所有成员，即使用`form`模块名`import ＊`，但此方式不推荐使用。
## import 模块名 as 别名
下面程序使用导入整个模块的最简单语法来导入指定模块：
```py
# 导入sys整个模块
import sys
# 使用sys模块名作为前缀来访问模块中的成员
print(sys.argv[0])
```
上面第 2 行代码使用最简单的方式导入了`sys`模块，因此在程序中使用`sys`模块内的成员时，必须添加模块名作为前缀。

运行上面程序，可以看到如下输出结果（`sys`模块下的`argv`变量用于获取运行 Python 程序的命令行参数，其中`argv[0]`用于获取当前 Python 程序的存储路径）：
```
C:\Users\mengma\Desktop\hello.py
```
导入整个模块时，也可以为模块指定别名。例如如下程序：
```py
# 导入sys整个模块，并指定别名为s
import sys as s
# 使用s模块别名作为前缀来访问模块中的成员
print(s.argv[0])
```
第 2 行代码在导入`sys`模块时才指定了别名`s`，因此在程序中使用`sys`模块内的成员时，必须添加模块别名`s`作为前缀。运行该程序，可以看到如下输出结果：
```
C:\Users\mengma\Desktop\hello.py
```
也可以一次导入多个模块，多个模块之间用逗号隔开。
```py
# 导入sys、os两个模块
import sys,os
# 使用模块名作为前缀来访问模块中的成员
print(sys.argv[0])
# os模块的sep变量代表平台上的路径分隔符
print(os.sep)
```
上面第 2 行代码一次导入了 sys 和 os 两个模块，因此程序要使用 sys、os 两个模块内的成员，只要分别使用 sys、os 模块名作为前缀即可。在 Windows 平台上运行该程序，可以看到如下输出结果（os 模块的 sep 变量代表平台上的路径分隔符）：
```
C:\Users\mengma\Desktop\hello.py
\
```
在导入多个模块的同时，也可以为模块指定别名：
```py
# 导入sys、os两个模块，并为sys指定别名s，为os指定别名o
import sys as s,os as o
# 使用模块别名作为前缀来访问模块中的成员
print(s.argv[0])
print(o.sep)
```
上面第 2 行代码一次导入了sys 和 os 两个模块，并分别为它们指定别名为 s、o，因此程序可以通过 s、o 两个前缀来使用 sys、os 两个模块内的成员。在 Windows 平台上运行该程序，可以看到如下输出结果：
```
C:\Users\mengma\Desktop\hello.py
\
```
## from 模块名 import 成员名 as 别名
```py
# 导入sys模块的argv成员
from sys import argv
# 使用导入成员的语法，直接使用成员名访问
print(argv[0])
```
第 2 行代码导入了`sys`模块中的`argv`成员，这样即可在程序中直接使用 argv 成员，无须使用任何前缀。

导入模块成员时，也可以为成员指定别名，例如如下程序：
```py
# 导入sys模块的argv成员，并为其指定别名v
from sys import argv as v
# 使用导入成员（并指定别名）的语法，直接使用成员的别名访问
print(v[0])
```
第 2 行代码导入了`sys`模块中的`argv`成员，并为该成员指定别名`v`，这样即可在程序中通过别名`v`使用`argv`成员，无须使用任何前缀。

`form...import`导入模块成员时，支持一次导入多个成员：
```py
# 导入sys模块的argv,winver成员
from sys import argv, winver
# 使用导入成员的语法，直接使用成员名访问
print(argv[0])
print(winver)
```
上面第 2 行代码导入了`sys`模块中的`argv、 winver`成员，这样即可在程序中直接使用`argv、winver`两个成员，无须使用任何前缀。运行该程序，可以看到如下输出结果（`sys`的`winver`成员记录了该 Python 的版本号）：
```
C:\Users\mengma\Desktop\hello.py
3.11
```
一次导入多个模块成员时，也可指定别名，同样使用`as`关键字为成员指定别名：
```py
# 导入sys模块的argv,winver成员，并为其指定别名v、wv
from sys import argv as v, winver as wv
# 使用导入成员（并指定别名）的语法，直接使用成员的别名访问
print(v[0])
print(wv)
```
上面第 2 行代码导入了 sys 模块中的 argv、winver 成员，并分别为它们指定了别名 v、wv，这样即可在程序中通过 v 和 wv 两个别名使用 argv、winver 成员，无须使用任何前缀。
## 不推荐使用 from import 导入模块所有成员
在使用`from...import`语法时，可以一次导入指定模块内的所有成员（此方式不推荐）：
```py
#导入sys 棋块内的所有成员
from sys import *
#使用导入成员的语法，直接使用成员的别名访问
print(argv[0])
print(winver)
```
上面代码一次导入了`sys`模块中的所有成员，这样程序即可通过成员名来使用该模块内的所有成员。该程序的输出结果和前面程序的输出结果完全相同。

需要说明的是，一般不推荐使用“from 模块 import”这种语法导入指定模块内的所有成员，因为它存在潜在的风险。比如同时导入`module1`和`module2`内的所有成员，假如这两个模块内都有一个`foo()`函数，那么当在程序中执行如下代码时：
```
foo()
```
上面调用的这个`foo()`函数到底是`module1`模块中的还是`module2`模块中的？因此，这种导入指定模块内所有成员的用法是有风险的。

但如果换成如下两种导入方式：
```
import module1
import module2 as m2
```
接下来要分别调用这两个模块中的`foo()`函数就非常清晰。
```py
#使用模块module1 的模块名作为前缀调用foo()函数
module1.foo()
#使用module2 的模块别名作为前缀调用foo()函数
m2.foo()
或者使用 from...import 语句也是可以的：
#导入module1 中的foo 成员，并指定其别名为foo1
from module1 import foo as fool
#导入module2 中的foo 成员，并指定其别名为foo2
from module2 import foo as foo2
```
此时通过别名将 module1 和 module2 两个模块中的 foo 函数很好地进行了区分，接下来分别调用两个模块中 foo() 函数就很清晰：
```
foo1() #调用module1 中的foo()函数
foo2() #调用module2 中的foo()函数
```
# 自定义模块
Python 模块就是 Python 程序，换句话说，只要是 Python 程序，都可以作为模块导入。例如，下面定义了一个简单的模块（编写在 demo.py 文件中）：
```py
name = "小明"
add = "xiaoming"
print(name,add)
def say():
    print("人生苦短，我学Python！")
class CLanguage:
    def __init__(self,name,add):
        self.name = name
        self.add = add
    def say(self):
        print(self.name,self.add)
```
可以看到，我们在 demo.py 文件中放置了变量（name 和 add）、函数（ say() ）以及一个 Clanguage 类，该文件就可以作为一个模板。

但通常情况下，为了检验模板中代码的正确性，我们往往需要为其设计一段测试代码，例如：
```
say()
clangs = CLanguage("小红","xiaohong")
clangs.say()
运行 demo.py 文件，其执行结果为：
小明 xiaoming
人生苦短，我学Python！
小红 xiaohong
```
通过观察模板中程序的执行结果可以断定，模板文件中包含的函数以及类，是可以正常工作的。

在此基础上，我们可以新建一个 test.py 文件，并在该文件中使用 demo.py 模板文件，即使用 import 语句导入 demo.py：
```
import demo
```
注意，虽然 demo 模板文件的全称为 demo.py，但在使用 import 语句导入时，只需要使用该模板文件的名称即可。

此时，如果直接运行 test.py 文件，其执行结果为：
```
小明 xiaoming
人生苦短，我学Python！
小红 xiaohong
```
可以看到，当执行 test.py 文件时，它同样会执行 demo.py 中用来测试的程序，这显然不是我们想要的效果。正常的效果应该是，只有直接运行模板文件时，测试代码才会被执行；反之，如果是其它程序以引入的方式执行模板文件，则测试代码不应该被执行。

要实现这个效果，可以借助 Python 内置的`__name__`变量。当直接运行一个模块时，`name`变量的值为`__main__`；而将模块被导入其他程序中并运行该程序时，处于模块中的`__name__`变量的值就变成了模块名。因此，如果希望测试函数只有在直接运行模块文件时才执行，则可在调用测试函数时增加判断，即只有当`__name__ =='__main__'`时才调用测试函数。

因此，我们可以修改 demo.py 模板文件中的测试代码为：
```
if __name__ == '__main__':
    say()
    clangs = CLanguage("C语言中文网","http://c.biancheng.net")
    clangs.say()
```
这样，当我们直接运行 demo.py 模板文件时，其执行结果不变；而运行 test.py 文件时，其执行结果为：
```
Python教程 http://c.biancheng.net/python
```
显然，这里执行的仅是模板文件中的输出语句，测试代码并未执行。
## 自定义模块编写说明文档
我们知道，在定义函数或者类时，可以为其添加说明文档，以方便用户清楚的知道该函数或者类的功能。自定义模块也不例外。

为自定义模块添加说明文档，和函数或类的添加方法相同，即只需在模块开头的位置定义一个字符串即可。例如，为 demo.py 模板文件添加一个说明文档：
```
'''
demo 模块中包含以下内容：
name 字符串变量：初始值为“Python教程”
add    字符串变量：初始值为“http://c.biancheng.net/python”
say() 函数
CLanguage类：包含 name 和 add 属性和 say() 方法。
'''
```
在此基础上，我们可以通过模板的 __doc__ 属性，来访问模板的说明文档。例如，在 test.py 文件中添加如下代码：
```
import demo
print(demo.__doc__)
```
程序运行结果为：
Python教程 http://c.biancheng.net/python
```
demo 模块中包含以下内容：
name 字符串变量：初始值为“Python教程”
add    字符串变量：初始值为“http://c.biancheng.net/python”
say() 函数
CLanguage类：包含 name 和 add 属性和 say() 方法。
```
# 导入模块的3种方式
即自定义 Python 模板后，在其它文件中用 import（或 from...import） 语句引入该文件时，Python 解释器同时如下错误：
```
ModuleNotFoundError: No module named '模块名'
```
意思是 Python 找不到这个模块名，这是什么原因导致的呢？要想解决这个问题，读者要先搞清楚 Python 解释器查找模块文件的过程。

通常情况下，当使用 import 语句导入模块后，Python 会按照以下顺序查找指定的模块文件：
在当前目录，即当前执行的程序文件所在目录下查找；
到 PYTHONPATH（环境变量）下的每个目录中查找；
到 Python 默认的安装目录下查找。

以上所有涉及到的目录，都保存在标准模块 sys 的 sys.path 变量中，通过此变量我们可以看到指定程序文件支持查找的所有目录。换句话说，如果要导入的模块没有存储在 sys.path 显示的目录中，那么导入该模块并运行程序时，Python 解释器就会抛出 ModuleNotFoundError（未找到模块）异常。

解决“Python找不到指定模块”的方法有 3 种，分别是：
* 向 sys.path 中临时添加模块文件存储位置的完整路径；
* 将模块放在 sys.path 变量中已包含的模块加载路径中；
* 设置 path 系统环境变量。

```
#hello.py
def say ():
    print("Hello,World!")
#say.py
import hello
hello.say()
```
显然，hello.py 文件和 say.py 文件并不在同一目录，此时运行 say.py 文件，其运行结果为：
```
 Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\say.py", line 1, in <module>
    import hello
ModuleNotFoundError: No module named 'hello'
```
可以看到，Python 解释器抛出了 ModuleNotFoundError 异常。接下来，分别用以上 3 种方法解决这个问题。
## 导入模块方式一：临时添加模块完整路径
模块文件的存储位置，可以临时添加到 sys.path 变量中，即向 sys.path 中添加 D:\python_module（hello.py 所在目录），在 say.py 中的开头位置添加如下代码：
```
import sys
sys.path.append('D:\\python_module')
```
注意：在添加完整路径中，路径中的 '\' 需要使用 \ 进行转义，否则会导致语法错误。再次运行 say.py 文件，运行结果如下：
```
Hello,World!
```
可以看到，程序成功运行。在此基础上，我们在 say.py 文件中输出 sys.path 变量的值，会得到以下结果：
```
['C:\\Users\\mengma\\Desktop', 'D:\\python3.6\\Lib\\idlelib', 'D:\\python3.6\\python36.zip', 'D:\\python3.6\\DLLs', 'D:\\python3.6\\lib', 'D:\\python3.6', 'C:\\Users\\mengma\\AppData\\Roaming\\Python\\Python36\\site-packages', 'D:\\python3.6\\lib\\site-packages', 'D:\\python3.6\\lib\\site-packages\\win32', 'D:\\python3.6\\lib\\site-packages\\win32\\lib', 'D:\\python3.6\\lib\\site-packages\\Pythonwin', 'D:\\python_module']
```
该输出信息中，红色部分就是临时添加进去的存储路径。需要注意的是，通过该方法添加的目录，只能在执行当前文件的窗口中有效，窗口关闭后即失效。
## 导入模块方式二：将模块保存到指定位置
如果要安装某些通用性模块，比如复数功能支持的模块、矩阵计算支持的模块、图形界面支持的模块等，这些都属于对 Python 本身进行扩展的模块，这种模块应该直接安装在 Python 内部，以便被所有程序共享，此时就可借助于 Python 默认的模块加载路径。

Python 程序默认的模块加载路径保存在 sys.path 变量中，因此，我们可以在 say.py 程序文件中先看看 sys.path 中保存的默认加载路径，向 say.py 文件中输出 sys.path 的值，如下所示：
```
['C:\\Users\\mengma\\Desktop', 'D:\\python3.6\\Lib\\idlelib', 'D:\\python3.6\\python36.zip', 'D:\\python3.6\\DLLs', 'D:\\python3.6\\lib', 'D:\\python3.6', 'C:\\Users\\mengma\\AppData\\Roaming\\Python\\Python36\\site-packages', 'D:\\python3.6\\lib\\site-packages', 'D:\\python3.6\\lib\\site-packages\\win32', 'D:\\python3.6\\lib\\site-packages\\win32\\lib', 'D:\\python3.6\\lib\\site-packages\\Pythonwin']
```
上面的运行结果中，列出的所有路径都是 Python 默认的模块加载路径，但通常来说，我们默认将 Python 的扩展模块添加在 lib\site-packages 路径下，它专门用于存放 Python 的扩展模块和包。

所以，我们可以直接将我们已编写好的 hello.py 文件添加到  lib\site-packages 路径下，就相当于为 Python 扩展了一个 hello 模块，这样任何 Python 程序都可使用该模块。

移动工作完成之后，再次运行 say.py 文件，可以看到成功运行的结果：
```
Hello,World!
```
## 导入模块方式三：设置环境变量
PYTHONPATH 环境变量（简称 path 变量）的值是很多路径组成的集合，Python 解释器会按照 path 包含的路径进行一次搜索，直到找到指定要加载的模块。当然，如果最终依旧没有找到，则 Python 就报 ModuleNotFoundError 异常。

在 Windows 平台上设置环境变量
首先，找到桌面上的“计算机”（或者我的电脑），并点击鼠标右键，单击“属性”。此时会显示“控制面板\所有控制面板项\系统”窗口，单击该窗口左边栏中的“高级系统设置”菜单，出现“系统属性”对话框，如图 1 所示。


图 1 系统属性对话框

如图 1 所示，点击“环境变量”按钮，此时将弹出图 2 所示的对话框：


图 2 环境变量对话框

如图 2 所示，通过该对话框，就可以完成 path 环境变量的设置。需要注意的是，该对话框分为上下 2 部分，其中上面的“用户变量”部分用于设置当前用户的环境变量，下面的“系统变量”部分用于设置整个系统的环境变量。

通常情况下，建议大家设置设置用户的 path 变量即可，因为此设置仅对当前登陆系统的用户有效，而如果修改系统的 path 变量，则对所有用户有效。
对于普通用户来说，设置用户 path 变量和系统 path 变量的效果是相同的，但 Python 在使用 path 变量时，会先按照系统 path 变量的路径去查找，然后再按照用户 path 变量的路径去查找。

这里我们选择设置当前用户的 path 变量。单击用户变量中的“新建”按钮， 系统会弹出如图 3 所示的对话框。

图 3 新建PYTHONPATH环境变量

其中，在“变量名”文本框内输入 PYTHONPATH，表明将要建立名为 PYTHONPATH 的环境变量；在“变量值”文本框内输入 .;d:\python_ module。注意，这里其实包含了两条路径（以分号 ；作为分隔符）：
第一条路径为一个点（.），表示当前路径，当运行 Python 程序时，Python 将可以从当前路径加载模块；
第二条路径为 d:\python_ module，当运行 Python 程序时，Python 将可以从 d:\python_ module 中加载模块。

然后点击“确定”，即成功设置 path 环境变量。此时，我们只需要将模块文件移动到和引入该模块的文件相同的目录，或者移动到 d:\python_ module 路径下，该模块就能被成功加载。
在 Linux 上设置环境变量
启动 Linux 的终端窗口，进入当前用户的 home 路径下，然后在 home 路径下输入如下命令：
ls - a

该命令将列出当前路径下所有的文件，包括隐藏文件。Linux 平台的环境变量是通过 .bash_profile 文件来设置的，使用无格式编辑器打开该文件，在该文件中添加 PYTHONPATH 环境变量。也就是为该文件增加如下一行：
#设置PYTHON PATH 环境变量
PYTHONPATH=.:/home/mengma/python_module

Linux 与 Windows 平台不一样，多个路径之间以冒号（:）作为分隔符，因此上面一行同样设置了两条路径，点（.）代表当前路径，还有一条路径是 /home/mengma/python_module（mengma 是在 Linux 系统的登录名）。

在完成了 PYTHONPATH 变量值的设置后，在 .bash_profile 文件的最后添加导出 PYTHONPATH 变量的语句。
```
#导出PYTHONPATH 环境变量
export PYTHONPATH
```
重新登录 Linux 平台，或者执行如下命令：
```
source.bash_profile
```
这两种方式都是为了运行该文件，使在文件中设置的 PYTHONPATH 变量值生效。

在成功设置了上面的环境变量之后，接下来只要把前面定义的模块（Python 程序）放在与当前所运行 Python 程序相同的路径中（或放在 /home/mengma/python_module 路径下），该模块就能被成功加载了。
在Mac OS X 上设置环境变量
在 Mac OS X 上设置环境变量与 Linux 大致相同（因为 Mac OS X 本身也是类 UNIX 系统）。启动 Mac OS X 的终端窗口（命令行界面），进入当前用户的 home 路径下，然后在 home 路径下输入如下命令：
ls -a

该命令将列出当前路径下所有的文件，包括隐藏文件。Mac OS X 平台的环境变量也可通过，bash_profile 文件来设置，使用无格式编辑器打开该文件，在该文件中添加 PYTHONPATH 环境变量。也就是为该文件增加如下一行：
#设置PYTHON PATH 环境变盘
PYTHONPATH=.:/Users/mengma/python_module

Mac OS X 的多个路径之间同样以冒号（:）作为分隔符，因此上面一行同样设置了两条路径：点（.）代表当前路径，还有一条路径是 /Users/mengma/python_module（memgma 是作者在 Mac OS X 系统的登录名）。

在完成了 PYTHONPATH 变量值的设置后，在 .bash_profile 文件的最后添加导出 PYTHONPATH 变量的语句。
#导出PYTHON PATH 环境变量
export PYTHONPATH

重新登录 Mac OS X 系统，或者执行如下命令：
source.bash_profile

这两种方式都是为了运行该文件，使在文件中设置的 PYTHONPATH 变量值生效。

在成功设置了上面的环境变量之后，接下来只要把前面定义的模块（Python 程序）放在与当前所运行 Python 程序相同的路径中（或放在 Users/mengma/python_module 路径下），该模块就能被成功加载了。

# __all__变量
事实上，当我们向文件导入某个模块时，导入的是该模块中那些名称不以下划线（单下划线“_”或者双下划线“__”）开头的变量、函数和类。因此，如果我们不想模块文件中的某个成员被引入到其它文件中使用，可以在其名称前添加下划线。
```py
#demo.py
def say():
  print("人生苦短，我学Python！")
def CLanguage():
  print("小明")
def disPython():
  print("小红")
#test.py
from demo import *
say()
CLanguage()
disPython()
```
在此基础上，如果 demo.py 模块中的 disPython() 函数不想让其它文件引入，则只需将其名称改为 _disPython() 或者 __disPython()。修改之后，再次执行 test.py，其输出结果为：
```
人生苦短，我学Python！
C语言中文网：http://c.biancheng.net
Traceback (most recent call last):
  File "C:/Users/mengma/Desktop/2.py", line 4, in <module>
    disPython()
NameError: name 'disPython' is not defined
```
显然，test.py 文件中无法使用未引入的 disPython() 函数。
## Python模块__all__变量
除此之外，还可以借助模块提供的 __all__ 变量，该变量的值是一个列表，存储的是当前模块中一些成员（变量、函数或者类）的名称。通过在模块文件中设置 __all__ 变量，当其它文件以“from 模块名 import *”的形式导入该模块时，该文件中只能使用 __all__ 列表中指定的成员。
也就是说，只有以“from 模块名 import *”形式导入的模块，当该模块设有 __all__ 变量时，只能导入该变量指定的成员，未指定的成员是无法导入的。

举个例子，修改 demo.py 模块文件中的代码：
```
def say():
    print("人生苦短，我学Python！")
def CLanguage():
    print("C语言中文网：http://c.biancheng.net")
def disPython():
    print("Python教程：http://c.biancheng.net/python")
__all__ = ["say","CLanguage"]
```
可见，`__all__`变量只包含 say() 和 CLanguage() 的函数名，不包含 disPython() 函数的名称。此时直接执行  test.py 文件，其执行结果为：
```
人生苦短，我学Python！
C语言中文网：http://c.biancheng.net
Traceback (most recent call last):
  File "C:/Users/mengma/Desktop/2.py", line 4, in <module>
    disPython()
NameError: name 'disPython' is not defined
```
显然，对于 test.py 文件来说，demo.py 模块中的 disPython() 函数是未引入，这样调用是非法的。

再次声明，__all__ 变量仅限于在其它文件中以“from 模块名 import *”的方式引入。也就是说，如果使用以下 2 种方式引入模块，则 __all__ 变量的设置是无效的。

1) 以“import 模块名”的形式导入模块。通过该方式导入模块后，总可以通过模块名前缀（如果为模块指定了别名，则可以使用模快的别名作为前缀）来调用模块内的所有成员（除了以下划线开头命名的成员）。

仍以 demo.py 模块文件和 test.py 文件为例，修改它们的代码如下所示：
```
#demo.py
def say():
    print("人生苦短，我学Python！")
def CLanguage():
    print("C语言中文网：http://c.biancheng.net")
def disPython():
    print("Python教程：http://c.biancheng.net/python")
__all__ = ["say"]
#test.py
import demo
demo.say()
demo.CLanguage()
demo.disPython()
运行 test.py 文件，其输出结果为：
人生苦短，我学Python！
C语言中文网：http://c.biancheng.net
Python教程：http://c.biancheng.net/python
```
可以看到，虽然 demo.py 模块文件中设置有  __all__ 变量，但是当以“import demo”的方式引入后，__all__ 变量将不起作用。

2) 以“from 模块名 import 成员”的形式直接导入指定成员。使用此方式导入的模块，__all__ 变量即便设置，也形同虚设。

仍以 demo.py 和 test.py 为例，修改 test.py 文件中的代码，如下所示：
```
from demo import say
from demo import CLanguage
from demo import disPython
say()
CLanguage()
disPython()
运行 test.py，输出结果为：
人生苦短，我学Python！
C语言中文网：http://c.biancheng.net
Python教程：http://c.biancheng.net/python
```
# 包
实际开发中，一个大型的项目往往需要使用成百上千的 Python 模块，如果将这些模块都堆放在一起，势必不好管理。而且，使用模块可以有效避免变量名或函数名重名引发的冲突，但是如果模块名重复怎么办呢？因此，Python提出了包（Package）的概念。

简单理解，包就是文件夹，只不过在该文件夹下必须存在一个名为`__init__.py`的文件。

注意，这是 Python 2.x 的规定，而在 Python 3.x 中，`__init__.py`对包来说，并不是必须的。

每个包的目录下都必须建立一个`__init__.py`的模块，可以是一个空模块，可以写一些初始化代码，其作用就是告诉 Python 要将该目录当成包来处理。

注意，`__init__.py`不同于其他模块文件，此模块的模块名不是`__init__`，而是它所在的包名。例如，在`settings`包中的`__init__.py`文件，其模块名就是`settings`。

包是一个包含多个模块的文件夹，它的本质依然是模块，因此包中也可以包含包。

Python 库：相比模块和包，库是一个更大的概念，例如在 Python 标准库中的每个库都有好多个包，而每个包中都有若干个模块。
# 创建包，导入包
包其实就是文件夹，更确切的说，是一个包含`__init__.py`文件的文件夹。因此，如果我们想手动创建一个包，只需进行以下 2 步操作：
* 新建一个文件夹，文件夹的名称就是新建包的包名；
* 在该文件夹中，创建一个`__init__.py`文件（前后各有 2 个下划线‘_’），该文件中可以不编写任何代码。当然，也可以编写一些 Python 初始化代码，则当有其它程序文件导入包时，会自动执行该文件中的代码。

例如，现在我们创建一个非常简单的包，该包的名称为`my_package`，可以仿照以上 2 步进行：
* 创建一个文件夹，其名称设置为`my_package`；
* 在该文件夹中添加一个`__init__.py`文件，此文件中可以不编写任何代码。不过，这里向该文件编写如下代码：
```
'''
创建第一个 Python 包
'''
print('python')
```
可以看到，`__init__.py`文件中，包含了 2 部分信息，分别是此包的说明信息和一条`print`输出语句。

由此，我们就成功创建好了一个 Python 包。

创建好包之后，我们就可以向包中添加模块（也可以添加包）。这里给`my_package`包添加 2 个模块，分别是`module1.py、module2.py`，各自包含的代码分别如下所示：
```py
#module1.py模块文件
def display(arc):
  print(arc)
#module2.py 模块文件
class CLanguage:
  def display(self):
    print("python")
```
现在，我们就创建好了一个具有如下文件结构的包：
```
my_package
     ┠── __init__.py
     ┠── module1.py
     ┗━━  module2.py
```
## Python包的导入
包其实本质上还是模块，因此导入模块的语法同样也适用于导入包。无论导入我们自定义的包，还是导入从他处下载的第三方包，导入方法可归结为以下 3 种：
* `import 包名[.模块名 [as 别名]]`
* `from 包名 import 模块名 [as 别名]`
* `from 包名.模块名 import 成员名 [as 别名]`

用`[]`括起来的部分，是可选部分，即可以使用，也可以直接忽略。

注意，导入包的同时，会在包目录下生成一个含有`__init__.cpython-36.pyc`文件的`__pycache__`文件夹。

#### 1) import 包名[.模块名 [as 别名]]
```py
import my_package.module1
my_package.module1.display("java")
```
可以看到，通过此语法格式导入包中的指定模块后，在使用该模块中的成员（变量、函数、类）时，需添加“包名.模块名”为前缀。当然，如果使用`as`给包名.模块名”起一个别名的话，就使用直接使用这个别名作为前缀使用该模块中的方法了：
```py
import my_package.module1 as module
module.display("python")
```
另外，当直接导入指定包时，程序会自动执行该包所对应文件夹下的`__init__.py`文件中的代码。
```py
import my_package
my_package.module1.display("linux")
```
直接导入包名，并不会将包中所有模块全部导入到程序中，它的作用仅仅是导入并执行包下的`__init__.py`文件，因此，运行该程序，在执行`__init__.py`文件中代码的同时，还会抛出`AttributeError`异常（访问的对象不存在）：
```
python
Traceback (most recent call last):
  File "C:\Users\mengma\Desktop\demo.py", line 2, in <module>
    my_package.module1.display("linux")
AttributeError: module 'my_package' has no attribute 'module1'
```
我们知道，包的本质就是模块，导入模块时，当前程序中会包含一个和模块名同名且类型为`module`的变量，导入包也是如此：
```py
import my_package
print(my_package)
print(my_package.__doc__)
print(type(my_package))
```
运行结果为：
```
python
<module 'my_package' from 'C:\\Users\\mengma\\Desktop\\my_package\\__init__.py'>

创建第一个 Python 包

<class 'module'>
```
#### 2) from 包名 import 模块名 [as 别名]
```py
from my_package import module1
module1.display("golang")
```
运行结果为：
```
python
golang
```
可以看到，使用此语法格式导入包中模块后，在使用其成员时不需要带包名前缀，但需要带模块名前缀。

当然，我们也可以使用`as`为导入的指定模块定义别名：
```py
from my_package import module1 as module
module.display("golang")
```
同样，既然包也是模块，那么这种语法格式自然也支持`from 包名 import *`这种写法，它和`import`包名 的作用一样，都只是将该包的`__init__.py`文件导入并执行。
#### 3) from 包名.模块名 import 成员名 [as 别名]
此语法格式用于向程序中导入“包.模块”中的指定成员（变量、函数或类）。通过该方式导入的变量（函数、类），在使用时可以直接使用变量名（函数名、类名）调用：
```py
from my_package.module1 import display
display("shell")
```
运行结果为：
```
python
shell
```
当然，也可以使用`as`为导入的成员起一个别名：
```py
from my_package.module1 import display as dis
dis("shell")
```
该程序的运行结果和上面相同。

另外，在使用此种语法格式加载指定包的指定模块时，可以使用 * 代替成员名，表示加载该模块下的所有成员。
```py
from my_package.module1 import *
display("python")
```
# 查看模块
## 查看模块成员：dir()函数
通过`dir()`函数，我们可以查看某指定模块包含的全部成员（包括变量、函数和类）。注意这里所指的全部成员，不仅包含可供我们调用的模块成员，还包含所有名称以双下划线“__”开头和结尾的成员，而这些“特殊”命名的成员，是为了在本模块中使用的，并不希望被其它文件调用。
```py
import string
print(dir(string))
```
程序执行结果为：
```
['Formatter', 'Template', '_ChainMap', '_TemplateMetaclass', '__all__', '__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', '_re', '_string', 'ascii_letters', 'ascii_lowercase', 'ascii_uppercase', 'capwords', 'digits', 'hexdigits', 'octdigits', 'printable', 'punctuation', 'whitespace']
```
可以看到，通过`dir()`函数获取到的模块成员，不仅包含供外部文件使用的成员，还包含很多“特殊”（名称以 2 个下划线开头和结束）的成员，列出这些成员，对我们并没有实际意义。
```py
import string
print([e for e in dir(string) if not e.startswith('_')])
```
程序执行结果为：
```
['Formatter', 'Template', 'ascii_letters', 'ascii_lowercase', 'ascii_uppercase', 'capwords', 'digits', 'hexdigits', 'octdigits', 'printable', 'punctuation', 'whitespace']
```
显然通过列表推导式，可在`dir()`函数输出结果的基础上，筛选出对我们有用的成员并显示出来。
## 查看模块成员：__all__变量
`__all__`变量也可以查看模块（包）内包含的所有成员。
```py
import string
print(string.__all__)
```
程序执行结果为：
```
['ascii_letters', 'ascii_lowercase', 'ascii_uppercase', 'capwords', 'digits', 'hexdigits', 'octdigits', 'printable', 'punctuation', 'whitespace', 'Formatter', 'Template']
```
和`dir()`函数相比，`__all__`变量在查看指定模块成员时，它不会显示模块中的特殊成员，同时还会根据成员的名称进行排序显示。

不过需要注意的是，并非所有的模块都支持使用`__all__`变量，因此对于获取有些模块的成员，就只能使用`dir()`函数。
# __doc__属性：查看文档
在使用`dir()`函数和`__all__`变量的基础上，虽然我们能知晓指定模块（或包）中所有可用的成员（变量、函数和类）：
```py
import string
print(string.__all__)
```
程序执行结果为：
```
['ascii_letters', 'ascii_lowercase', 'ascii_uppercase', 'capwords', 'digits', 'hexdigits', 'octdigits', 'printable', 'punctuation', 'whitespace', 'Formatter', 'Template']
```
但对于以上的输出结果，对于不熟悉`string`模块的用户，还是不清楚这些名称分别表示的是什么意思，更不清楚各个成员有什么功能。

针对这种情况，我们可以使用`help()`函数来获取指定成员（甚至是该模块）的帮助信息。
```py
#***__init__.py 文件中的内容***
from my_package.module1 import *
from my_package.module2 import *
#***module1.py 中的内容***
#module1.py模块文件
def display(arc):
  '''
  直接输出指定的参数
  '''
  print(arc)
#***module2.py中的内容***
#module2.py 模块文件
class CLanguage:
  '''
  CLanguage是一个类，其包含：
  display() 方法
  '''
  def display(self):
    print("http://c.biancheng.net/python/")
```
现在，我们先借助`dir()`函数，查看`my_package`包中有多少可供我们调用的成员：
```py
import my_package
print([e for e in dir(my_package) if not e.startswith('_')])
```
程序输出结果为：
```
['CLanguage', 'display', 'module1', 'module2']
```
通过此输出结果可以得知，在`my_package`包中，有以上 4 个成员可供我们使用。接下来，我们使用`help()`函数来查看这些成员的具体含义（以 module1 为例）：
```py
import my_package
help(my_package.module1)
```
输出结果为：
```
Help on module my_package.module1 in my_package:

NAME
    my_package.module1 - #module1.py模块文件

FUNCTIONS
    display(arc)
        直接输出指定的参数

FILE
    c:\users\mengma\desktop\my_package\module1.py
```
通过输出结果可以得知，`module1`实际上是一个模块文件，其包含`display()`函数，该函数的功能是直接输出指定的`arc`参数。同时，还显示出了该模块具体的存储位置。

值得一提的是，之所以我们可以使用`help()`函数查看具体成员的信息，是因为该成员本身就包含表示自身身份的说明文档（本质是字符串，位于该成员内部开头的位置）。无论是函数还是类，都可以使用`__doc__`属性获取它们的说明文档，模块也不例外。
```py
import my_package
print(my_package.module1.display.__doc__)
```
其实，`help()`函数底层也是借助`__doc__`属性实现的。

如果使用`help()`函数或者`__doc__`属性，仍然无法满足我们的需求，还可以调用`__file__`属性，查看该模块或者包文件的具体存储位置，直接查看其源代码。
# __file__属性：查看模块的源文件路径
当指定模块（或包）没有说明文档时，仅通过`help()`函数或者`__doc__`属性，无法有效帮助我们理解该模块（包）的具体功能。在这种情况下，我们可以通过`__file__`属性查找该模块（或包）文件所在的具体存储位置，直接查看其源代码。
```py
import my_package
print(my_package.__file__) # C:\Users\mengma\Desktop\my_package\__init__.py
```
注意，因为当引入`my_package`包时，其实际上执行的是`__init__.py`文件，因此这里查看`my_package`包的存储路径，输出的`__init__.py`文件的存储路径。
```py
import string
print(string.__file__) # D:\python3.6\lib\string.py
```
注意，并不是所有模块都提供`__file__`属性，因为并不是所有模块的实现都采用 Python 语言，有些模块采用的是其它编程语言。
