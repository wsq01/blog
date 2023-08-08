---
title: Bash 基本语法
date: 2023-07-02 17:42:56
tags: [Bash, Linux]
categories: [Linux]
---

# Bash 的基本语法
## echo 命令
`echo`命令的作用是在屏幕输出一行文本，可以将该命令的参数原样输出。
```bash
$ echo hello world
hello world
```
上面例子中，`echo`的参数是`hello world`，可以原样输出。

如果想要输出的是多行文本，即包括换行符。这时就需要把多行文本放在引号里面。
```shell
$ echo "<HTML>
  <HEAD>
    <TITLE>Page Title</TITLE>
  </HEAD>
  <BODY>
    Page body.
  </BODY>
</HTML>"
```
#### -n 参数
默认情况下，`echo`输出的文本末尾会有一个回车符。`-n`参数可以取消末尾的回车符，使得下一个提示符紧跟在输出内容的后面。
```bash
$ echo -n hello world
hello world$
```
上面例子中，`world`后面直接就是下一行的提示符`$`。
```bash
$ echo a;echo b
a
b

$ echo -n a;echo b
ab
```
上面例子中，`-n`参数可以让两个`echo`命令的输出连在一起，出现在同一行。
#### -e 参数
`-e`参数会解释引号（双引号和单引号）里面的特殊字符（比如换行符`\n`）。如果不使用`-e`参数，即默认情况下，引号会让特殊字符变成普通字符，`echo`不解释它们，原样输出。
```bash
$ echo "Hello\nWorld"
Hello\nWorld

# 双引号的情况
$ echo -e "Hello\nWorld"
Hello
World

# 单引号的情况
$ echo -e 'Hello\nWorld'
Hello
World
```
上面代码中，`-e`参数使得`\n`解释为换行符，导致输出内容里面出现换行。
## 命令格式
命令行环境中，主要通过使用 Shell 命令，进行各种操作。Shell 命令基本都是下面的格式。
```bash
$ command [ arg1 ... [ argN ]]
```
上面代码中，`command`是具体的命令或者一个可执行文件，`arg1 ... argN`是传递给命令的参数，它们是可选的。
```bash
$ ls -l
```
上面这个命令中，`ls`是命令，`-l`是参数。

有些参数是命令的配置项，这些配置项一般都以一个连词线开头，比如上面的`-l`。同一个配置项往往有长和短两种形式，比如`-l`是短形式，`--list`是长形式，它们的作用完全相同。短形式便于手动输入，长形式一般用在脚本之中，可读性更好，利于解释自身的含义。
```bash
# 短形式
$ ls -r

# 长形式
$ ls --reverse
```
Bash 单个命令一般都是一行，用户按下回车键，就开始执行。有些命令比较长，写成多行会有利于阅读和编辑，这时可以在每一行的结尾加上反斜杠，Bash 就会将下一行跟当前行放在一起解释。
```bash
$ echo foo bar

# 等同于
$ echo foo \
bar
```
## 空格
Bash 使用空格（或 Tab 键）区分不同的参数。
```bash
$ command foo bar
```
上面命令中，`foo`和`bar`之间有一个空格，所以 Bash 认为它们是两个参数。

如果参数之间有多个空格，Bash 会自动忽略多余的空格。
```bash
$ echo this is a     test
this is a test
```
上面命令中，`a`和`test`之间有多个空格，Bash 会忽略多余的空格。
## 分号
分号（`;`）是命令的结束符，使得一行可以放置多个命令，上一个命令执行结束后，再执行第二个命令。
```bash
$ clear; ls
```
上面例子中，Bash 先执行`clear`命令，执行完成后，再执行`ls`命令。

> 注意，使用分号时，第二个命令总是接着第一个命令执行，不管第一个命令执行成功或失败。

## 命令的组合符&&和||
除了分号，Bash 还提供两个命令组合符`&&`和`||`，允许更好地控制多个命令之间的继发关系。
```bash
Command1 && Command2
```
上面命令的意思是，如果`Command1`命令运行成功，则继续运行`Command2`命令。
```bash
Command1 || Command2
```
上面命令的意思是，如果`Command1`命令运行失败，则继续运行`Command2`命令。
```bash
$ cat filelist.txt ; ls -l filelist.txt
```
上面例子中，只要`cat`命令执行结束，不管成功或失败，都会继续执行`ls`命令。
```bash
$ cat filelist.txt && ls -l filelist.txt
```
上面例子中，只有`cat`命令执行成功，才会继续执行`ls`命令。如果`cat`执行失败（比如不存在文件`flielist.txt`），那么`ls`命令就不会执行。
```bash
$ mkdir foo || mkdir bar
```
上面例子中，只有`mkdir foo`命令执行失败（比如`foo`目录已经存在），才会继续执行`mkdir bar`命令。如果`mkdir foo`命令执行成功，就不会创建`bar`目录了。
## type 命令
Bash 本身内置了很多命令，同时也可以执行外部程序。怎么知道一个命令是内置命令，还是外部程序呢？

`type`命令用来判断命令的来源。
```bash
$ type echo
echo is a shell builtin
$ type ls
ls is hashed (/bin/ls)
```
上面代码中，`type`命令告诉我们，`echo`是内部命令，`ls`是外部程序（`/bin/ls`）。

`type`命令本身也是内置命令。
```bash
$ type type
type is a shell builtin
```
如果要查看一个命令的所有定义，可以使用`type`命令的`-a`参数。
```bash
$ type -a echo
echo is shell builtin
echo is /usr/bin/echo
echo is /bin/echo
```
上面代码表示，`echo`命令既是内置命令，也有对应的外部程序。

`type`命令的`-t`参数，可以返回一个命令的类型：别名（`alias`），关键词（`keyword`），函数（`function`），内置命令（`builtin`）和文件（`file`）。
```bash
$ type -t bash
file
$ type -t if
keyword
```
上面例子中，`bash`是文件，`if`是关键词。
## 快捷键
Bash 提供很多快捷键，可以大大方便操作。下面是一些最常用的快捷键。

- `Ctrl + L`：清除屏幕并将当前行移到页面顶部。
- `Ctrl + C`：中止当前正在执行的命令。
- `Shift + PageUp`：向上滚动。
- `Shift + PageDown`：向下滚动。
- `Ctrl + U`：从光标位置删除到行首。
- `Ctrl + K`：从光标位置删除到行尾。
- `Ctrl + W`：删除光标位置前一个单词。
- `Ctrl + D`：关闭 Shell 会话。
- `↑，↓`：浏览已执行命令的历史记录。

除了上面的快捷键，Bash 还具有自动补全功能。命令输入到一半的时候，可以按下 Tab 键，Bash 会自动完成剩下的部分。比如，输入`tou`，然后按一下 Tab 键，Bash 会自动补上`ch`。

除了命令的自动补全，Bash 还支持路径的自动补全。有时，需要输入很长的路径，这时只需要输入前面的部分，然后按下 Tab 键，就会自动补全后面的部分。如果有多个可能的选择，按两次 Tab 键，Bash 会显示所有选项，让你选择。
# 引号和转义
Bash 只有一种数据类型，就是字符串。不管用户输入什么数据，Bash 都视为字符串。因此，字符串相关的引号和转义，对 Bash 来说就非常重要。
## 转义
某些字符在 Bash 里面有特殊含义（比如`$、&、*`）。
```bash
$ echo $date

$
```
上面例子中，输出`$date`不会有任何结果，因为`$`是一个特殊字符。

如果想要原样输出这些特殊字符，就必须在它们前面加上反斜杠，使其变成普通字符。这就叫做“转义”。
```bash
$ echo \$date
$date
```
反斜杠本身也是特殊字符，如果想要原样输出反斜杠，就需要对它自身转义，连续使用两个反斜线（`\\`）。
```bash
$ echo \\
\
```
反斜杠除了用于转义，还可以表示一些不可打印的字符。
* `\a`：响铃
* `\b`：退格
* `\n`：换行
* `\r`：回车
* `\t`：制表符

如果想要在命令行使用这些不可打印的字符，可以把它们放在引号里面，然后使用`echo`命令的`-e`参数。
```bash
$ echo a\tb
atb

$ echo -e "a\tb"
a        b
```
上面例子中，命令行直接输出不可打印字符`\t`，Bash 不能正确解释。必须把它们放在引号之中，然后使用echo命令的-e参数。

换行符是一个特殊字符，表示命令的结束，Bash 收到这个字符以后，就会对输入的命令进行解释执行。换行符前面加上反斜杠转义，就使得换行符变成一个普通字符，Bash 会将其当作长度为0的空字符处理，从而可以将一行命令写成多行。
```bash
$ mv \
/path/to/foo \
/path/to/bar

# 等同于
$ mv /path/to/foo /path/to/bar
```
上面例子中，如果一条命令过长，就可以在行尾使用反斜杠，将其改写成多行。这是常见的多行命令的写法。
## 单引号
Bash 允许字符串放在单引号或双引号之中，加以引用。

单引号用于保留字符的字面含义，各种特殊字符在单引号里面，都会变为普通字符，比如星号（*）、美元符号（$）、反斜杠（\）等。
```bash
$ echo '*'
*

$ echo '$USER'
$USER

$ echo '$((2+2))'
$((2+2))

$ echo '$(echo foo)'
$(echo foo)
```
上面命令中，单引号使得 Bash 扩展、变量引用、算术运算和子命令，都失效了。如果不使用单引号，它们都会被 Bash 自动扩展。

由于反斜杠在单引号里面变成了普通字符，所以如果单引号之中，还要使用单引号，不能使用转义，需要在外层的单引号前面加上一个美元符号（$），然后再对里层的单引号转义。
```bash
# 不正确
$ echo it's

# 不正确
$ echo 'it\'s'

# 正确
$ echo $'it\'s'
```
不过，更合理的方法是改在双引号之中使用单引号。
```bash
$ echo "it's"
it's
```
## 双引号
双引号比单引号宽松，大部分特殊字符在双引号里面，都会失去特殊含义，变成普通字符。
```bash
$ echo "*"
*
```
上面例子中，通配符*是一个特殊字符，放在双引号之中，就变成了普通字符，会原样输出。这一点需要特别留意，这意味着，双引号里面不会进行文件名扩展。

但是，三个特殊字符除外：美元符号（`$`）、反引号（`\``）和反斜杠（`\`）。这三个字符在双引号之中，依然有特殊含义，会被 Bash 自动扩展。
```bash
$ echo "$SHELL"
/bin/bash

$ echo "`date`"
Mon Jan 27 13:33:18 CST 2020
```
上面例子中，美元符号（$）和反引号（`）在双引号中，都保持特殊含义。美元符号用来引用变量，反引号则是执行子命令。
```bash
$ echo "I'd say: \"hello!\""
I'd say: "hello!"

$ echo "\\"
\
```
上面例子中，反斜杠在双引号之中保持特殊含义，用来转义。所以，可以使用反斜杠，在双引号之中插入双引号，或者插入反斜杠本身。

换行符在双引号之中，会失去特殊含义，Bash 不再将其解释为命令的结束，只是作为普通的换行符。所以可以利用双引号，在命令行输入多行文本。
```bash
$ echo "hello
world"
hello
world
```
上面命令中，Bash 正常情况下会将换行符解释为命令结束，但是换行符在双引号之中就失去了这种特殊作用，只用来换行，所以可以输入多行。`echo`命令会将换行符原样输出，显示的时候正常解释为换行。

双引号的另一个常见的使用场合是，文件名包含空格。这时就必须使用双引号（或单引号），将文件名放在里面。
```bash
$ ls "two words.txt"
```
上面命令中，`two words.txt`是一个包含空格的文件名，如果不放在双引号里面，就会被 Bash 当作两个文件。

双引号会原样保存多余的空格。
```bash
$ echo "this is a     test"
this is a     test
```
双引号还有一个作用，就是保存原始命令的输出格式。
```bash
# 单行输出
$ echo $(cal)
一月 2020 日 一 二 三 四 五 六 1 2 3 ... 31

# 原始格式输出
$ echo "$(cal)"
      一月 2020
日 一 二 三 四 五 六
          1  2  3  4
 5  6  7  8  9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30 31
```
上面例子中，如果`$(cal)`不放在双引号之中，`echo`就会将所有结果以单行输出，丢弃了所有原始的格式。
## Here 文档
Here 文档是一种输入多行字符串的方法，格式如下。
```bash
<< token
text
token
```
它的格式分成开始标记（`<< token`）和结束标记（`token`）。开始标记是两个小于号`+ Here`文档的名称，名称可以随意取，后面必须是一个换行符；结束标记是单独一行顶格写的 Here 文档名称，如果不是顶格，结束标记不起作用。两者之间就是多行字符串的内容。

下面是一个通过 Here 文档输出 HTML 代码的例子。
```bash
$ cat << _EOF_
<html>
<head>
    <title>
    The title of your page
    </title>
</head>

<body>
    Your page content goes here.
</body>
</html>
_EOF_
```
Here 文档内部会发生变量替换，同时支持反斜杠转义，但是不支持通配符扩展，双引号和单引号也失去语法作用，变成了普通字符。
```bash
$ foo='hello world'
$ cat << _example_
$foo
"$foo"
'$foo'
_example_

hello world
"hello world"
'hello world'
```
上面例子中，变量`$foo`发生了替换，但是双引号和单引号都原样输出了，表明它们已经失去了引用的功能。

如果不希望发生变量替换，可以把 Here 文档的开始标记放在单引号之中。
```bash
$ foo='hello world'
$ cat << '_example_'
$foo
"$foo"
'$foo'
_example_

$foo
"$foo"
'$foo'
```
上面例子中，Here 文档的开始标记（`_example_`）放在单引号之中，导致变量替换失效了。

Here 文档的本质是重定向，它将字符串重定向输出给某个命令，相当于包含了`echo`命令。
```bash
$ command << token
  string
token

# 等同于

$ echo string | command
```
上面代码中，Here 文档相当于`echo`命令的重定向。

所以，Here 字符串只适合那些可以接受标准输入作为参数的命令，对于其他命令无效，比如`echo`命令就不能用 Here 文档作为参数。
```bash
$ echo << _example_
hello
_example_
```
上面例子不会有任何输出，因为 Here 文档对于`echo`命令无效。

此外，Here 文档也不能作为变量的值，只能用于命令的参数。
## Here 字符串
Here 文档还有一个变体，叫做 Here 字符串，使用三个小于号（`<<<`）表示。
```
<<< string
```
它的作用是将字符串通过标准输入，传递给命令。

有些命令直接接受给定的参数，与通过标准输入接受参数，结果是不一样的。所以才有了这个语法，使得将字符串通过标准输入传递给命令更方便，比如cat命令只接受标准输入传入的字符串。
```bash
$ cat <<< 'hi there'
# 等同于
$ echo 'hi there' | cat
```
上面的第一种语法使用了 Here 字符串，要比第二种语法看上去语义更好，也更简洁。
```bash
$ md5sum <<< 'ddd'
# 等同于
$ echo 'ddd' | md5sum
```
上面例子中，`md5sum`命令只能接受标准输入作为参数，不能直接将字符串放在命令后面，会被当作文件名，即`md5sum ddd`里面的`ddd`会被解释成文件名。这时就可以用 Here 字符串，将字符串传给`md5sum`命令。