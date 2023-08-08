

# Shell变量
在 Bash shell 中，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。

这意味着，Bash shell 在默认情况下不会区分变量类型，即使你将整数和小数赋值给变量，它们也会被视为字符串。

## 单引号和双引号的区别
```sh
#!/bin/bash
url="http://www.baidu.com"
website1='百度：${url}'
website2="百度：${url}"
echo $website1
echo $website2
```
运行结果：
```
百度：${url}
百度：:http://www.baidu.com
```
以单引号`' '`包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。

以双引号`" "`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义。

如果变量的内容是数字，那么可以不加引号；如果真的需要原样输出就加单引号；其他没有特别要求的字符串等最好都加上双引号，定义变量时加双引号是最常见的使用场景。
## 将命令的结果赋值给变量
Shell 也支持将命令的执行结果赋值给变量，常见的有以下两种方式：
```sh
variable=`command`
variable=$(command)
```
第一种方式把命令用反引号\` \`（位于 Esc 键的下方）包围起来，反引号和单引号非常相似，容易产生混淆，所以不推荐使用这种方式；第二种方式把命令用`$()`包围起来，区分更加明显，所以推荐使用这种方式。
## 只读变量
使用`readonly`命令可以将变量定义为只读变量，只读变量的值不能被改变。
```sh
#!/bin/bash
myUrl="http://abc.com/shell/"
readonly myUrl
myUrl="http://abc.com/shell/"
```
运行脚本，结果如下：
```
bash: myUrl: This variable is read only.
```
## 删除变量
使用`unset`命令可以删除变量。
```sh
unset variable_name
```
变量被删除后不能再次使用；`unset`命令不能删除只读变量。
```sh
#!/bin/sh
myUrl="http://c.biancheng.net/shell/"
unset myUrl
echo $myUrl
```
上面的脚本没有任何输出。
# Shell位置参数（命令行参数）
运行 Shell 脚本文件时我们可以给它传递一些参数，这些参数在脚本文件内部可以使用$n的形式来接收，例如，`$1`表示第一个参数，`$2`表示第二个参数，依次类推。

同样，在调用函数时也可以传递参数。Shell 函数参数的传递和其它编程语言不同，没有所谓的形参和实参，在定义函数时也不用指明参数的名字和数目。换句话说，定义 Shell 函数时不能带参数，但是在调用函数时却可以传递参数，这些传递进来的参数，在函数内部就也使用$n的形式接收，例如，`$1`表示第一个参数，`$2`表示第二个参数，依次类推。

这种通过`$n`的形式来接收的参数，在 Shell 中称为位置参数。

变量的名字必须以字母或者下划线开头，不能以数字开头；但是位置参数却偏偏是数字，这和变量的命名规则是相悖的，所以我们将它们视为“特殊变量”。
## 1) 给脚本文件传递位置参数
```sh
#!/bin/bash
echo "Language: $1"
echo "URL: $2"
```
运行`test.sh`，并附带参数：
```
[mozhiyan@localhost demo]$ ./test.sh Shell http://abc.com/
Language: Shell
URL: http://abc.com/
```
其中`Shell`是第一个位置参数，`http://abc.com/`是第二个位置参数，两者之间以空格分隔。
## 2) 给函数传递位置参数
```sh
#!/bin/bash
#定义函数
function func(){
    echo "Language: $1"
    echo "URL: $2"
}
#调用函数
func C++ http://c.biancheng.net/cplus/
```
运行`test.sh`：
```
[mozhiyan@localhost demo]$ ./test.sh
Language: C++
URL: http://c.biancheng.net/cplus/
```
如果参数个数太多，达到或者超过了 10 个，那么就得用`${n}`的形式来接收了，例如`${10}、${23}`。`{ }`的作用是为了帮助解释器识别参数的边界，这跟使用变量时加`{ }`是一样的效果。
# Shell特殊变量

| 变量	      |含义 |
| :--: | :--: |
| `$0`	     |  当前脚本的文件名。 |
| `$n（n≥1）` |	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是 $1，第二个参数是 $2。 |
| `$#`	     |  传递给脚本或函数的参数个数。 |
| `$*`	     |  传递给脚本或函数的所有参数。 |
| `$@`	     |  传递给脚本或函数的所有参数。当被双引号" "包含时，$@ 与 $* 稍有不同。 |
| `$?`	     |  上个命令的退出状态，或函数的返回值 |
| `$$`	     |  当前 Shell 进程 ID。 |

## 1) 给脚本文件传递参数
```sh
#!/bin/bash
echo "Process ID: $$"
echo "File Name: $0"
echo "First Parameter : $1"
echo "Second Parameter : $2"
echo "All parameters 1: $@"
echo "All parameters 2: $*"
echo "Total: $#"
```
运行`test.sh`，并附带参数：
```
[mozhiyan@localhost demo]$ ./test.sh Shell Linux
Process ID: 5943
File Name: bash
First Parameter : Shell
Second Parameter : Linux
All parameters 1: Shell Linux
All parameters 2: Shell Linux
Total: 2
```
## 2) 给函数传递参数
```sh
#!/bin/bash
#定义函数
function func(){
    echo "Language: $1"
    echo "URL: $2"
    echo "First Parameter : $1"
    echo "Second Parameter : $2"
    echo "All parameters 1: $@"
    echo "All parameters 2: $*"
    echo "Total: $#"
}
#调用函数
func Java abc
```
# $?：获取函数返回值或者上一个命令的退出状态
`$?`是一个特殊变量，用来获取上一个命令的退出状态，或者上一个函数的返回值。

所谓退出状态，就是上一个命令执行后的返回结果。退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1。

不过，也有一些命令返回其他值，表示不同类型的错误。
## 1) $? 获取上一个命令的退出状态
```sh
#!/bin/bash
if [ "$1" == 100 ]
then
  exit 0  #参数正确，退出状态为0
else
  exit 1  #参数错误，退出状态1
fi
```
`exit`表示退出当前 Shell 进程，我们必须在新进程中运行`test.sh`，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法取得它的退出状态了。

运行`test.sh`时传递参数 100：
```
[mozhiyan@localhost demo]$ bash ./test.sh 100  #作为一个新进程运行
[mozhiyan@localhost demo]$ echo $?
0
```
运行`test.sh`时传递参数 89：
```
[mozhiyan@localhost demo]$ bash ./test.sh 89  #作为一个新进程运行
[mozhiyan@localhost demo]$ echo $?
1
```
## 2) $? 获取函数的返回值
```sh
#!/bin/bash
#得到两个数相加的和
function add(){
    return `expr $1 + $2`
}
add 23 50  #调用函数
echo $?  #获取函数返回值
# 运行结果：
# 73
```


# echo命令：输出字符串
`echo`是一个 Shell 内建命令，用来在终端输出字符串，并在最后默认加上换行符。
```sh
#!/bin/bash
name="Shell教程"
url="http://abc.com/shell/"
echo "你好！"  #直接输出字符串
echo $url  #输出变量
echo "${name}的网址是：${url}"  #双引号包围的字符串中可以解析变量
echo '${name}的网址是：${url}'  #单引号包围的字符串中不能解析变量
```
运行结果：
```
你好！
http://abc.com/shell/
Shell教程的网址是：http://abc.com/shell/
${name}的网址是：${url}
```
## 不换行
`echo`命令输出结束后默认会换行，如果不希望换行，可以加上`-n`参数：
```sh
#!/bin/bash
name="Tom"
age=20
height=175
weight=62
echo -n "${name} is ${age} years old, "
echo -n "${height}cm in height "
echo "and ${weight}kg in weight."
echo "Thank you!"
```
运行结果：
```
Tom is 20 years old, 175cm in height and 62kg in weight.
Thank you!
```
## 输出转义字符
默认情况下，`echo`不会解析以反斜杠`\`开头的转义字符。比如，`\n`表示换行，`echo` 默认会将它作为普通字符对待。
```
[root@localhost ~]# echo "hello \nworld"
hello \nworld
```
我们可以添加`-e`参数来让`echo`命令解析转义字符。
```
[root@localhost ~]# echo -e "hello \nworld"
hello
world
```
## \c 转义字符
有了`-e`参数，我们也可以使用转义字符`\c`来强制`echo`命令不换行了。
```sh
#!/bin/bash
name="Tom"
age=20
height=175
weight=62
echo -e "${name} is ${age} years old, \c"
echo -e "${height}cm in height \c"
echo "and ${weight}kg in weight."
echo "Thank you!"
```
运行结果：
```
Tom is 20 years old, 175cm in height and 62kg in weight.
Thank you!
```
# exit命令：退出当前进程
`exit`是一个 Shell 内置命令，用来退出当前 Shell 进程，并返回一个退出状态；使用`$?`可以接收这个退出状态。

`exit`命令可以接受一个整数值作为参数，代表退出状态。如果不指定，默认状态值是 0。

一般情况下，退出状态为 0 表示成功，退出状态为非 0 表示执行失败（出错）了。

`exit`退出状态只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败。

Shell 进程执行出错时，可以根据退出状态来判断具体出现了什么错误，比如打开一个文件时，我们可以指定 1 表示文件不存在，2 表示文件没有读取权限，3 表示文件类型不对。
```
#!/bin/bash
echo "befor exit"
exit 8
echo "after exit"
```
运行该脚本：
```
[mozhiyan@localhost ~]$ bash ./test.sh
befor exit
```
可以看到，`"after exit"`并没有输出，这说明遇到`exit`命令后，`test.sh`执行就结束了。

注意，`exit`表示退出当前 Shell 进程，我们必须在新进程中运行`test.sh`，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法看到输出结果了。

我们可以紧接着使用`$?`来获取`test.sh`的退出状态：
```
[mozhiyan@localhost ~]$ echo $?
8
```
# if else语句
## if 语句
```sh
if  condition
then
  statement(s)
fi
```
`condition`是判断条件，如果`condition`成立（返回“真”），那么`then`后边的语句将会被执行；如果`condition`不成立（返回“假”），那么不会执行任何语句。

从本质上讲，`if`检测的是命令的退出状态。

注意，最后必须以`fi`来闭合，`fi`就是`if`倒过来拼写。也正是有了`fi`来结尾，所以即使有多条语句也不需要用`{ }`包围起来。

也可以将`then`和`if`写在一行：
```sh
if  condition;  then
  statement(s)
fi
```
请注意`condition`后边的分号`;`，当`if`和`then`位于同一行的时候，这个分号是必须的，否则会有语法错误。
```sh
#!/bin/bash
read a
read b
if (( $a == $b ))
then
  echo "a和b相等"
fi
```
运行结果：
```
84↙
84↙
a和b相等
```
`(())`是一种数学计算命令，它除了可以进行最基本的加减乘除运算，还可以进行大于、小于、等于等关系运算，以及与、或、非逻辑运算。当`a`和`b`相等时，`(( $a == $b ))`判断条件成立，进入`if`，执行`then`后边的`echo`语句。
## if else 语句
```sh
if  condition
then
  statement1
else
  statement2
fi
```
如果`condition`成立，那么`then`后边的`statement1`语句将会被执行；否则，执行`else`后边的`statement2`语句。
```sh
#!/bin/bash
read a
read b
if (( $a == $b ))
then
  echo "a和b相等"
else
  echo "a和b不相等，输入错误"
fi
```
运行结果：
```
10↙
20↙
a 和 b 不相等，输入错误
```
## if elif else 语句
```sh
if  condition1
then
  statement1
elif condition2
then
  statement2
elif condition3
then
  statement3
……
else
  statementn
fi
```
注意，`if`和`elif`后边都得跟着`then`。
```sh
#!/bin/bash
read age
if (( $age <= 2 )); then
    echo "婴儿"
elif (( $age >= 3 && $age <= 8 )); then
    echo "幼儿"
elif (( $age >= 9 && $age <= 17 )); then
    echo "少年"
elif (( $age >= 18 && $age <=25 )); then
    echo "成年"
elif (( $age >= 26 && $age <= 40 )); then
    echo "青年"
elif (( $age >= 41 && $age <= 60 )); then
    echo "中年"
else
    echo "老年"
fi
```
# Shell退出状态
每一条 Shell 命令，不管是 Bash 内置命令（例如`cd、echo`），还是外部的 Linux 命令（例如`ls、awk`），还是自定义的 Shell 函数，当它退出（运行结束）时，都会返回一个比较小的整数值给调用（使用）它的程序，这就是命令的退出状态。

很多 Linux 命令其实就是一个C语言程序，熟悉C语言的读者都知道，`main()`函数的最后都有一个`return 0`，如果程序想在中间退出，还可以使用`exit 0`，这其实就是C语言程序的退出状态。当有其它程序调用这个程序时，就可以捕获这个退出状态。

`if`语句的判断条件，从本质上讲，判断的就是命令的退出状态。

按照惯例来说，退出状态为 0 表示“成功”；也就是说，程序执行完成并且没有遇到任何问题。除 0 以外的其它任何退出状态都为“失败”。

之所以说这是“惯例”而非“规定”，是因为也会有例外，比如`diff`命令用来比较两个文件的不同，对于“没有差别”的文件返回 0，对于“找到差别”的文件返回 1，对无效文件名返回 2。

在 Shell 中，有多种方式取得命令的退出状态，其中`$?`是最常见的一种。
```sh
#!/bin/bash
read a
read b
(( $a == $b ));
echo "退出状态："$?
```
运行结果1：
```
26
26
退出状态：0
```
运行结果2：
```
17
39
退出状态：1
```
## 退出状态和逻辑运算符的组合
Shell `if`语句的一个神奇之处是允许我们使用逻辑运算符将多个退出状态组合起来，这样就可以一次判断多个条件了。

运算符	使用格式	说明
| && | expression1 && expression2 | 逻辑与运算符，当 expression1 和 expression2 同时成立时，整个表达式才成立。<br>如果检测到 expression1 的退出状态为 0，就不会再检测 expression2 了，因为不管 expression2 的退出状态是什么，整个表达式必然都是不成立的，检测了也是多此一举。 |
| `||` | expression1 || expression2 | 逻辑或运算符，expression1 和 expression2 两个表达式中只要有一个成立，整个表达式就成立。<br>如果检测到 expression1 的退出状态为 1，就不会再检测 expression2 了，因为不管 expression2 的退出状态是什么，整个表达式必然都是成立的，检测了也是多此一举。 |
| ! | !expression | 逻辑非运算符，相当于“取反”的效果。如果 expression 成立，那么整个表达式就不成立；如果 expression 不成立，那么整个表达式就成立。 |

将用户输入的 URL 写入到文件中。
```sh
#!/bin/bash
read filename
read url
if test -w $filename && test -n $url
then
  echo $url > $filename
  echo "写入成功"
else
  echo "写入失败"
fi
```
在 Shell 脚本文件所在的目录新建一个文本文件并命名为`urls.txt`，然后运行 Shell 脚本，运行结果为：
```
urls.txt↙
http://c.biancheng.net/shell/↙
写入成功
```
`test`是 Shell 内置命令，可以对文件或者字符串进行检测，其中，`-w`选项用来检测文件是否存在并且可写，`-n`选项用来检测字符串是否非空。

`>`表示重定向，默认情况下，echo 向控制台输出，这里我们将输出结果重定向到文件。
