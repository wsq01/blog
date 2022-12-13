


# 字符串拼接
在 Python 中拼接（连接）字符串很简单，可以直接将两个字符串紧挨着写在一起：
```py
strname = "str1" "str2"
```
`strname`表示拼接以后的字符串变量名，`str1`和`str2`是要拼接的字符串内容。使用这种写法，Python 会自动将两个字符串拼接在一起。

需要注意的是，这种写法只能拼接字符串常量。如果需要使用变量，就得借助`+`运算符来拼接：
```py
strname = str1 + str2
```
当然，`+`运算符也能拼接字符串常量。
# 字符串和数字的拼接
Python 不允许直接拼接数字和字符串，所以我们必须先将数字转换成字符串。可以借助`str()`和`repr()`函数将数字转换为字符串：
```py
str(obj)
repr(obj)
```
`obj`表示要转换的对象，它可以是数字、列表、元组、字典等多种类型的数据。
```py
name = "小明"
age = 8
course = 3
info = name + "已经" + str(age) + "岁了，已经上" + repr(course) + "年级了。"
print(info) # 小明已经8岁了，已经上3年级了。
```
## str() 和 repr() 的区别
`str()`和`repr()`函数虽然都可以将数字转换成字符串，但它们之间是有区别的：
* `str()`用于将数据转换成适合人类阅读的字符串形式。
* `repr()`用于将数据转换成适合解释器阅读的字符串形式（Python 表达式的形式），适合在开发和调试阶段使用；如果没有等价的语法，则会发生`SyntaxError`异常。

```py
s = "http://www.baidu.com/test/"
s_str = str(s)
s_repr = repr(s)
print(type(s_str)) # <class 'str'>
print (s_str) # http://www.baidu.com/test/
print(type(s_repr)) # <class 'str'>
print (s_repr) # 'http://www.baidu.com/test/'
```
本例中，`s`本身就是一个字符串，但是我们依然使用`str()`和`repr()`对它进行了转换。从运行结果可以看出，`str()`保留了字符串最原始的样子，而`repr()`使用引号将字符串包围起来，这就是 Python 字符串的表达式形式。

另外，在 Python 交互式编程环境中输入一个表达式（变量、加减乘除、逻辑运算等）时，Python 会自动使用`repr()`函数处理该表达式。
# 截取字符串（字符串切片）
从本质上讲，字符串是由多个字符构成的，字符之间是有顺序的，这个顺序号就称为索引（`index`）。Python 允许通过索引来操作字符串中的单个或者多个字符，比如获取指定索引处的字符，返回指定字符的索引值等。
## 获取单个字符
知道字符串名字以后，在方括号`[ ]`中使用索引即可访问对应的字符：
```
strname[index]
```
`strname`表示字符串名字，`index`表示索引值。

Python 允许从字符串的两端使用索引：
* 当以字符串的左端（字符串的开头）为起点时，索引是从 0 开始计数的；字符串的第一个字符的索引为 0，第二个字符的索引为 1，第三个字符串的索引为 2 ……
* 当以字符串的右端（字符串的末尾）为起点时，索引是从 -1 开始计数的；字符串的倒数第一个字符的索引为 -1，倒数第二个字符的索引为 -2，倒数第三个字符的索引为 -3 ……

```py
url = 'http://www.test.com/'
#获取索引为10的字符
print(url[10]) # .
#获取索引为 6 的字符
print(url[-6]) # t
```
# 获取多个字符（字符串截去/字符串切片）
使用`[ ]`除了可以获取单个字符外，还可以指定一个范围来获取多个字符，也就是一个子串或者片段：
```
strname[start : end : step]
```
对各个部分的说明：
* `strname`：要截取的字符串；
* `start`：表示要截取的第一个字符所在的索引（截取时包含该字符）。如果不指定，默认为 0，也就是从字符串的开头截取；
* `end`：表示要截取的最后一个字符所在的索引（截取时不包含该字符）。如果不指定，默认为字符串的长度；
* `step`：指的是从`start`索引处的字符开始，每`step`个距离获取一个字符，直至`end`索引出的字符。`step`默认值为 1，当省略该值时，最后一个冒号也可以省略。

```py
url = 'http://www.testdem.com/java/'
#获取索引从7处到22（不包含22）的子串
print(url[7: 22]) # www.testdem.com
#获取索引从7处到-6的子串
print(url[7: -6]) # www.testdem.com
#获取索引从-21到6的子串
print(url[-21: -6]) # www.testdem.com
#从索引3开始，每隔4个字符取出一个字符，直到索引22为止
print(url[3: 22: 4]) # pwtdo
```
```py
url = 'http://www.testdem.com/java/'
#获取从索引5开始，直到末尾的子串
print(url[7: ]) # www.testdem.com
#获取从索引-21开始，直到末尾的子串
print(url[-21: ]) # www.testdem.com
#从开头截取字符串，直到索引22为止
print(url[: 22]) # http://www.testdem.com
#每隔3个字符取出一个字符
print(url[:: 3]) # hp/wed.ma/
```
# len()函数
要想知道一个字符串有多少个字符（获得字符串长度），或者一个字符串占用多少个字节，可以使用`len`函数。
```
len（string）
```
其中`string`用于指定要进行长度统计的字符串。
```py
a='http://www.test.com'
len(a) # 19
```
在实际开发中，除了常常要获取字符串的长度外，有时还要获取字符串的字节数。

在 Python 中，不同的字符所占的字节数不同，数字、英文字母、小数点、下划线以及空格，各占一个字节，而一个汉字可能占 2~4 个字节，具体占多少个，取决于采用的编码方式。例如，汉字在 GBK/GB2312 编码中占用 2 个字节，而在 UTF-8 编码中一般占用 3 个字节。

我们可以通过使用`encode()`方法，将字符串进行编码后再获取它的字节数。例如，采用 UTF-8 编码方式，计算“人生苦短，我用Python”的字节数，可以执行如下代码：
```py
str1 = "人生苦短，我用Python"
len(str1.encode()) # 27
```
因为汉字加中文标点符号共 7 个，占 21 个字节，而英文字母和英文的标点符号占 6 个字节，一共占用 27 个字节。

同理，如果要获取采用 GBK 编码的字符串的长度，可以执行如下代码：
```py
str1 = "人生苦短，我用Python"
len(str1.encode('gbk')) # 20
```
# split()方法
`split()`方法可以实现将一个字符串按照指定的分隔符切分成多个子串，这些子串会被保存到列表中（不包含分隔符），作为方法的返回值反馈回来。
```
str.split(sep,maxsplit)
```
此方法中各部分参数的含义分别是：
* `str`：表示要进行分割的字符串；
* `sep`：用于指定分隔符，可以包含多个字符。此参数默认为`None`，表示所有空字符，包括空格、换行符`\n`、制表符`\t`等。
* `maxsplit`：可选参数，用于指定分割的次数，最后列表中子串的个数最多为`maxsplit+1`。如果不指定或者指定为 -1，则表示分割次数没有限制。

在`split`方法中，如果不指定`sep`参数，需要以`str.split(maxsplit=xxx)`的格式指定`maxsplit`参数。

同内建函数（如`len`）的使用方式不同，字符串变量所拥有的方法，只能采用“字符串.方法名()”的方式调用。\
```py
str = "小明 >>> xiaoming"
list1 = str.split() #采用默认分隔符进行分割
list1 # ['小明', '>>>', 'xiaoming']
list2 = str.split('>>>') #采用多个字符进行分割
list2 # ['小明 ', 'xiaoming']
list3 = str.split('>') #采用 > 字符进行分割
list3 # ['小明 ', '', '', 'xiaoming']
```
需要注意的是，在未指定`sep`参数时，`split()`方法默认采用空字符进行分割，但当字符串中有连续的空格或其他空字符时，都会被视为一个分隔符对字符串进行分割：
```py
str = "小明   >>>   xiaoming"  #包含 3 个连续的空格
list6 = str.split()
list6 # ['小明', '>>>', 'xiaoming']
```
# join()方法
`join()`方法是`split()`方法的逆方法，用来将列表（或元组）中包含的多个字符串连接成一个字符串。

使用`join()`方法合并字符串时，它会将列表（或元组）中多个字符串采用固定的分隔符连接在一起。例如，字符串`www.baidu.com`就可以看做是通过分隔符“.”将`['www','baidu','com']`列表合并为一个字符串的结果。
```
newstr = str.join(iterable)
```
各参数的含义如下：
* `newstr`：表示合并后生成的新字符串；
* `str`：用于指定合并时的分隔符；
* `iterable`：做合并操作的源字符串数据，允许以列表、元组等形式提供。

```py
list = ['www','baidu','com']
'.'.join(list) # 'www.baidu.com'
```
```py
dir = '','usr','bin','env'
type(dir) # <class 'tuple'>
'/'.join(dir) # '/usr/bin/env'
```
# count()方法
`count`方法用于检索指定字符串在另一字符串中出现的次数，如果检索的字符串不存在，则返回 0，否则返回出现的次数。
```py
str.count(sub[,start[,end]])
```
各参数的具体含义如下：
* `str`：表示原字符串；
* `sub`：表示要检索的字符串；
* `start`：指定检索的起始位置，也就是从什么位置开始检测。如果不指定，默认从头开始检索；
* `end`：指定检索的终止位置，如果不指定，则表示一直检索到结尾。

```py
str = "www.baidu.com"
str.count('.') # 2
```
```py
str = "www.baidu.com"
str.count('.', 3) # 2
str.count('.', 4) # 1
```
字符串中各字符对应的检索值，从 0 开始，因此，本例中检索值 1 对应的是第 2 个字符‘.’，从输出结果可以分析出，从指定索引位置开始检索，其中也包含此索引位置。
```py
str = "www.baidu.com"
str.count('.', 4, -3) # 1
>>> str.count('.', 4, -4) # 0
```
# find()方法
`find()`方法用于检索字符串中是否包含目标字符串，如果包含，则返回第一次出现该字符串的索引；反之，则返回 -1。
```py
str.find(sub[,start[,end]])
```
各参数的含义如下：
* `str`：表示原字符串；
* `sub`：表示要检索的目标字符串；
* `start`：表示开始检索的起始位置。如果不指定，则默认从头开始检索；
* `end`：表示结束检索的结束位置。如果不指定，则默认一直检索到结尾。

```py
str = "www.baidu.com"
str.find('.') # 3
```
```py
str = "www.baidu.com"
str.find('.', 4) # 9
```
```py
str = "www.baidu.com"
str.find('.', 4, -4) # -1
```
位于索引`（4，-4）`之间的字符串为`baidu`，由于其不包含“.”，因此`find()`方法的返回值为 -1。

注意，Python 还提供了`rfind()`方法，与`find()`方法最大的不同在于，`rfind()`是从字符串右边开始检索。
```py
str = "www.baidu.com"
str.rfind('.') # 9
```
# index()方法
同`find()`方法类似，`index()`方法也可以用于检索是否包含指定的字符串，不同之处在于，当指定的字符串不存在时，`index()`方法会抛出异常。
```py
str.index(sub[,start[,end]])
```
各参数的含义：
* `str`：表示原字符串；
* `sub`：表示要检索的子字符串；
* `start`：表示检索开始的起始位置，如果不指定，默认从头开始检索；
* `end`：表示检索的结束位置，如果不指定，默认一直检索到结尾。

```py
str = "www.baidu.com"
str.index('.') # 3
```
```py
str = "www.baidu.com"
str.index('z')
Traceback (most recent call last):
  File "<pyshell#49>", line 1, in <module>
    str.index('z')
ValueError: substring not found
```
同`find()`和`rfind()`一样，字符串变量还具有`rindex()`方法，其作用和`index()`方法类似，不同之处在于它是从右边开始检索：
```py
str = "www.baidu.com"
str.rindex('.') # 9
```
# 字符串对齐方法（ljust()、rjust()和center()）
`str`提供了 3 种可用来进行文本对齐的方法，分别是`ljust()、rjust()`和`center()`方法。
## ljust()方法
`ljust()`方法的功能是向指定字符串的右侧填充指定字符，从而达到左对齐文本的目的。
```py
S.ljust(width[, fillchar])
```
各个参数的含义：
* `S`：表示要进行填充的字符串；
* `width`：表示包括`S`本身长度在内，字符串要占的总长度；
* `fillchar`：作为可选参数，用来指定填充字符串时所用的字符，默认情况使用空格。

```py
S = 'http://www.baidu.com/python/'
addr = 'http://www.baidu.com'
print(S.ljust(35)) # http://www.baidu.com/python/
print(addr.ljust(35)) # http://www.baidu.com/
```
注意，该输出结果中除了明显可见的网址字符串外，其后还有空格字符存在，每行一共 35 个字符长度。
```py
S = 'http://www.baidu.com/python/'
addr = 'http://www.baidu.com'
print(S.ljust(35,'-')) # http://www.baidu.com/python/------
print(addr.ljust(35,'-')) # http://www.baidu.com--------------
```
## rjust()方法
`rjust()`和`ljust()`方法类似，唯一的不同在于，`rjust()`方法是向字符串的左侧填充指定字符，从而达到右对齐文本的目的。
```
S.rjust(width[, fillchar])
```
其中各个参数的含义和`ljust()`完全相同。
```py
S = 'http://www.baidu.com/python/'
addr = 'http://www.baidu.com'
print(S.rjust(35)) # http://www.baidu.com/python/
print(addr.rjust(35)) # http://www.baidu.com
```          
可以看到，每行字符串都占用 35 个字节的位置，实现了整体的右对齐效果。
```py
S = 'http://www.baidu.com/python/'
addr = 'http://www.baidu.com/'
print(S.rjust(35,'-')) # ------http://www.baidu.com/python/
print(addr.rjust(35,'-')) # ----------http://www.baidu.com/python/
```
# center()方法
`center()`字符串方法与`ljust()`和`rjust()`的用法类似，但它让文本居中，而不是左对齐或右对齐。
```
S.center(width[, fillchar])
```
其中各个参数的含义和`ljust()、rjust()`方法相同。
```py
S = 'http://www.baidu.com/python/'
addr = 'http://www.baidu.com/'
print(S.center(35,)) # http://www.baidu.com/python/
print(addr.center(35,)) # http://www.baidu.com
```   
```py
S = 'http://www.baidu.com/python/'
addr = 'http://www.baidu.com/'
print(S.center(35,'-')) # ---http://www.baidu.com/python/---
print(addr.center(35,'-')) # --------http://www.baidu.com--------
```
# startswith()和endswith()方法
## startswith()方法
`startswith()`方法用于检索字符串是否以指定字符串开头，如果是返回`True`；反之返回`False`。
```py
str.startswith(sub[,start[,end]])
```
各个参数的具体含义：
* `str`：表示原字符串；
* `sub`：要检索的子串；
* `start`：指定检索开始的起始位置索引，如果不指定，则默认从头开始检索；
* `end`：指定检索的结束位置索引，如果不指定，则默认一直检索在结束。

```py
str = "www.baidu.com"
str.startswith("w")
True
```
```py
str = "www.baidu.com"
str.startswith("http")
False
```
```py
str = "www.baidu.com"
str.startswith("w", 2)
True
```
## endswith()方法
`endswith()`方法用于检索字符串是否以指定字符串结尾，如果是则返回`True`；反之则返回`False`。
```py
str.endswith(sub[,start[,end]])
```
此格式中各参数的含义如下：
* `str`：表示原字符串；
* `sub`：表示要检索的字符串；
* `start`：指定检索开始时的起始位置索引（字符串第一个字符对应的索引值为 0），如果不指定，默认从头开始检索。
* `end`：指定检索的结束位置索引，如果不指定，默认一直检索到结束。

```py
str = "www.baidu.com"
str.endswith("com")
True
```
# 字符串大小写转换
为了方便对字符串中的字母进行大小写转换，字符串变量提供了 3 种方法，分别是`title()、lower()`和`upper()`。
## title()方法
`title()`方法用于将字符串中每个单词的首字母转为大写，其他字母全部转为小写，转换完成后，此方法会返回转换得到的字符串。如果字符串中没有需要被转换的字符，此方法会将字符串原封不动地返回。
```
str.title()
```
其中，`str`表示要进行转换的字符串。
```py
str = "www.baidu.com"
str.title() # 'Www.Baidu.Co,'
str = "I LIKE C"
str.title() # 'I Like C'
```
## lower()方法
`lower()`方法用于将字符串中的所有大写字母转换为小写字母，转换完成后，该方法会返回新得到的字符串。如果字符串中原本就都是小写字母，则该方法会返回原字符串。
```
str.lower()
```
其中，`str`表示要进行转换的字符串。
```py
str = "I LIKE C"
str.lower() # 'i like c'
```
## upper()方法
`upper()`的功能和`lower()`方法恰好相反，它用于将字符串中的所有小写字母转换为大写字母，和以上两种方法的返回方式相同，即如果转换成功，则返回新字符串；反之，则返回原字符串。
```
str.upper()
```
其中，`str`表示要进行转换的字符串。
```py
str = "i like C"
str.upper() # 'I LIKE C'
```
需要注意的是，以上 3 个方法都仅限于将转换后的新字符串返回，而不会修改原字符串。
# 去除字符串中空格
用户输入数据时，很有可能会无意中输入多余的空格，或者在一些场景中，字符串前后不允许出现空格和特殊字符，此时就需要去除字符串中的空格和特殊字符。

这里的特殊字符，指的是制表符（`\t`）、回车符（`\r`）、换行符（`\n`）等。

字符串变量提供了 3 种方法来删除字符串中多余的空格和特殊字符：
* `strip()`：删除字符串前后（左右两侧）的空格或特殊字符。
* `lstrip()`：删除字符串前面（左边）的空格或特殊字符。
* `rstrip()`：删除字符串后面（右边）的空格或特殊字符。

注意，Python 的`str`是不可变的（不可变的意思是指，字符串一旦形成，它所包含的字符序列就不能发生任何改变），因此这三个方法只是返回字符串前面或后面空白被删除之后的副本，并不会改变字符串本身。
## strip()方法
`strip()`方法用于删除字符串左右两个的空格和特殊字符：
```
str.strip([chars])
```
其中，`str`表示原字符串，`[chars]`用来指定要删除的字符，可以同时指定多个，如果不手动指定，则默认会删除空格以及制表符、回车符、换行符等特殊字符。
```py
str = "  www.baidu.com \t\n\r"
str.strip() # 'www.baidu.com'
str.strip(" ,\r") # 'www.baidu.com \t\n'
str # '  www.baidu.com \t\n\r'
```
分析运行结果不难看出，通过`strip()`确实能够删除字符串左右两侧的空格和特殊字符，但并没有真正改变字符串本身。
## lstrip()方法
`lstrip()`方法用于去掉字符串左侧的空格和特殊字符：
```
str.lstrip([chars])
```
其中，`str`和`chars`参数的含义，分别同`strip()`语法格式中的`str`和`chars`完全相同。
```py
str = "  www.baidu.com \t\n\r"
str.lstrip() # 'www.baidu.com \t\n\r'
```
## rstrip()方法
`rstrip()`方法用于删除字符串右侧的空格和特殊字符：
```
str.rstrip([chars])
```
`str`和`chars`参数的含义和前面 2 种方法语法格式中的参数完全相同。
```py
str = "  www.baidu.com \t\n\r"
str.rstrip() # '  www.baidu.com'
```
# format()格式化输出方法
使用`%`操作符对各种类型的数据进行格式化输出，这是早期 Python 提供的方法。自 Python 2.6 版本开始，字符串类型提供了`format()`方法对字符串进行格式化。
```
str.format(args)
```
此方法中，`str`用于指定字符串的显示样式；`args`用于指定要进行格式转换的项，如果有多项，之间有逗号进行分割。

在创建显示样式模板时，需要使用`{}`和`:`来指定占位符，其完整的语法格式为：
```
{ [index][ : [ [fill] align] [sign] [#] [width] [.precision] [type] ] }
```
注意，格式中用 [] 括起来的参数都是可选参数，即可以使用，也可以不使用。各个参数的含义如下：
* `index`：指定：后边设置的格式要作用到`args`中第几个数据，数据的索引值从 0 开始。如果省略此选项，则会根据`args`中数据的先后顺序自动分配。
* `fill`：指定空白处填充的字符。注意，当填充字符为逗号(,)且作用于整数或浮点数时，该整数（或浮点数）会以逗号分隔的形式输出，例如（1000000会输出 1,000,000）。
* `align`：指定数据的对齐方式，具体的对齐方式如表。
| align	| 含义 |
| :--: | :--: |
| <	    | 数据左对齐。|
| >	    | 数据右对齐。|
| =	    | 数据右对齐，同时将符号放置在填充内容的最左侧，该选项只对数字类型有效。 |
| ^	    | 数据居中，此选项需和 width 参数一起使用。 |
* `sign`：指定有无符号数，此参数的值以及对应的含义如表。
| sign参数	| 含义 |
| :--: | :--: |
| +	        | 正数前加正号，负数前加负号。 |
| -	        | 正数前不加正号，负数前加负号。| 
| 空格      |  正数前加空格，负数前加负号。 |
| #         |  对于二进制数、八进制数和十六进制数，使用此参数，各进制数前会分别显示 0b、0o、0x前缀；反之则不显示前缀。|
* `width`：指定输出数据时所占的宽度。
* `.precision`：指定保留的小数位数。
* `type`：指定输出数据的具体类型，`type`占位符类型及含义：
| type类型值 | 含义 |
| :--: | :--: |
| s	| 对字符串类型格式化。 |
| d	| 十进制整数。 |
| c	| 将十进制整数自动转换成对应的 Unicode 字符。 |
| e | 或者 E 	转换成科学计数法后，再格式化输出。 |
| g | 或 G	自动在 e 和 f（或 E 和 F）中切换。 |
| b	| 将十进制数自动转换成二进制表示，再格式化输出。 |
| o	| 将十进制数自动转换成八进制表示，再格式化输出。 |
| x | 或者 X	将十进制数自动转换成十六进制表示，再格式化输出。 |
| f | 或者 F	转换为浮点数（默认小数点后保留 6 位），再格式化输出。 |
| %	| 显示百分比（默认显示小数点后 6 位）。 |

```py
str="网站名称：{:>9s}\t网址：{:s}"
print(str.format("百度","www.baidu.com")) # 网站名称：   百度 网址：www.baidu.com
```
在实际开发中，数值类型有多种显示需求，比如货币形式、百分比形式等，使用`format()`方法可以将数值格式化为不同的形式。
```py
#以货币形式显示
print("货币形式：{:,d}".format(1000000)) # 货币形式：1,000,000
#科学计数法表示
print("科学计数法：{:E}".format(1200.12)) # 科学计数法：1.200120E+03
#以十六进制表示
print("100的十六进制：{:#x}".format(100)) # 100的十六进制：0x64
#输出百分比形式
print("0.01的百分比表示：{:.0%}".format(0.01)) # 0.01的百分比表示：1%
```
# encode()和decode()方法
最早的字符串编码是 ASCII 编码，它仅仅对 10 个数字、26 个大小写英文字母以及一些特殊字符进行了编码。ASCII 码做多只能表示 256 个符号，每个字符只需要占用 1 个字节。

随着信息技术的发展，各国的文字都需要进行编码，于是相继出现了 GBK、GB2312、UTF-8 编码等，其中 GBK 和 GB2312 是我国制定的中文编码标准，规定英文字符母占用 1 个字节，中文字符占用 2 个字节；而 UTF-8 是国际通过的编码格式，它包含了全世界所有国家需要用到的字符，其规定英文字符占用 1 个字节，中文字符占用 3 个字节。

Python 3.x 默认采用 UTF-8 编码格式，有效地解决了中文乱码的问题。

在 Python 中，有 2 种常用的字符串类型，分别为`str`和`bytes`类型，其中`str`用来表示 Unicode 字符，`bytes`用来表示二进制数据。`str`类型和`bytes`类型之间就需要使用`encode()`和`decode()`方法进行转换。
## encode()方法
`encode()`方法为字符串类型（`str`）提供的方法，用于将`str`类型转换成`bytes`类型，这个过程也称为“编码”。
```
str.encode([encoding="utf-8"][,errors="strict"])
```
注意，格式中用`[]`括起来的参数为可选参数，也就是说，在使用此方法时，可以使用`[]`中的参数，也可以不使用。

`encode()`参数及含义：
* `str`表示要进行转换的字符串。
* `encoding = "utf-8"`指定进行编码时采用的字符编码，该选项默认采用 utf-8 编码。例如，如果想使用简体中文，可以设置 gb2312。
当方法中只使用这一个参数时，可以省略前边的`encoding=`，直接写编码格式，例如`str.encode("UTF-8")`。
* `errors = "strict"`指定错误处理方式，其可选择值可以是：
 * `strict`：遇到非法字符就抛出异常。
 * `ignore`：忽略非法字符。
 * `replace`：用“？”替换非法字符。
 * `xmlcharrefreplace`：使用`xml`的字符引用。该参数的默认值为`strict`。

注意，使用`encode()`方法对原字符串进行编码，不会直接修改原字符串，如果想修改原字符串，需要重新赋值。
```py
str = "测试"
str.encode()
b'C\xe8\xaf\xad\xe8'
```
此方式默认采用 UTF-8 编码，也可以手动指定其它编码格式：
```py
str = "测试"
str.encode('GBK')
b'C\xd3\xef\xd1\xd4'
```
## decode()方法
和`encode()`方法正好相反，`decode()`方法用于将`bytes`类型的二进制数据转换为`str`类型，这个过程也称为“解码”。
```
bytes.decode([encoding="utf-8"][,errors="strict"])
```
`decode()`参数及含义：
* `bytes`表示要进行转换的二进制数据。
* `encoding="utf-8"`指定解码时采用的字符编码，默认采用 utf-8 格式。当方法中只使用这一个参数时，可以省略`encoding=`，直接写编码方式即可。注意，对`bytes`类型数据解码，要选择和当初编码时一样的格式。
* `errors = "strict"`指定错误处理方式，其可选择值可以是：
 * `strict`：遇到非法字符就抛出异常。
 * `ignore`：忽略非法字符。
 * `replace`：用`?`替换非法字符。
 * `xmlcharrefreplace`：使用`xml`的字符引用。该参数的默认值为`strict`。

```py
str = "小明"
bytes=str.encode()
bytes.decode()
'小明'
```
注意，如果编码时采用的不是默认的 UTF-8 编码，则解码时要选择和编码时一样的格式，否则会抛出异常：
```py
str = "小明"
bytes = str.encode("GBK")
bytes.decode()  #默认使用 UTF-8 编码，会抛出以下异常
Traceback (most recent call last):
  File "<pyshell#10>", line 1, in <module>
    bytes.decode()
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xd3 in position 1: invalid continuation byte
bytes.decode("GBK")
'小明'
```
# dir()和help()帮助函数
`dir()`函数用来列出某个类或者某个模块中的全部内容，包括变量、方法、函数和类等。
```
dir(obj)
```
`obj`表示要查看的对象。`obj`可以不写，此时`dir()`会列出当前范围内的变量、方法和定义的类型。

`help()`函数用来查看某个函数或者模块的帮助文档：
```
help(obj)
```
`obj`表示要查看的对象。`obj`可以不写，此时`help()`会进入帮助子程序。

掌握了以上两个函数，我们就可以自行查阅 Python 中所有方法、函数、变量、类的用法和功能了。
```py
dir(str)
['__add__', '__class__', '__contains__', '__delattr__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__getitem__', '__getnewargs__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__iter__', '__le__', '__len__', '__lt__', '__mod__', '__mul__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__rmod__', '__rmul__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'capitalize', 'casefold', 'center', 'count', 'encode', 'endswith', 'expandtabs', 'find', 'format', 'format_map', 'index', 'isalnum', 'isalpha', 'isascii', 'isdecimal', 'isdigit', 'isidentifier', 'islower', 'isnumeric', 'isprintable', 'isspace', 'istitle', 'isupper', 'join', 'ljust', 'lower', 'lstrip', 'maketrans', 'partition', 'replace', 'rfind', 'rindex', 'rjust', 'rpartition', 'rsplit', 'rstrip', 'split', 'splitlines', 'startswith', 'strip', 'swapcase', 'title', 'translate', 'upper', 'zfill']
```
在 Python 标准库中，以`__`开头和结尾的方法都是私有的，不能在类的外部调用。
```py
help(str.lower)
Help on method_descriptor:

lower(self, /)
    Return a copy of the string converted to lowercase.
```
注意，使用`help()`查看某个函数的用法时，函数名后边不能带括号，例如将上面的命令写作`help(str.lower())`就是错误的。
