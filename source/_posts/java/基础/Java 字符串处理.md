


# 定义字符串
Java 没有内置的字符串类型，而是在标准 Java 类库中提供了一个 String 类来创建和操作字符串。

在 Java 中定义一个字符串最简单的方法是用双引号把它包围起来。这种用双引号括起来的一串字符实际上都是 String 对象，如字符串“Hello”在编译后即成为 String 对象。因此也可以通过创建 String 类的实例来定义字符串。

不论使用哪种形式创建字符串，字符串对象一旦被创建，其值是不能改变的，但可以使用其他变量重新赋值的方式进行更改。
## 直接定义字符串
直接定义字符串是指使用双引号表示字符串中的内容。
```java
String str = "Hello Java";
```
注意：字符串变量必须经过初始化才能使用。
## 使用 String 类定义
在 Java 中每个双引号定义的字符串都是一个`String`类的对象。因此，可以通过使用`String`类的构造方法来创建字符串，该类位于`java.lang`包中。

`String`类的构造方法有多种重载形式，每种形式都可以定义字符串。
1. `String()`
初始化一个新创建的`String`对象，表示一个空字符序列。
2. `String(String original)`
初始化一个新创建的`String`对象，使其表示一个与参数相同的字符序列。换句话说，新创建的字符串是该参数字符串的副本。
```java
String str1 = new String("Hello Java");
String str2 = new String(str1);
```
这里`str1`和`str2`的值是相等的。
3. `String(char[ ]value)`
分配一个新的字符串，将参数中的字符数组元素全部变为字符串。该字符数组的内容已被复制，后续对字符数组的修改不会影响新创建的字符串。
```java
char a[] = {'H','e','l','l','0'};
String sChar = new String(a);
a[1] = 's';
```
上述`sChar`变量的值是字符串`"Hello"`。 即使在创建字符串之后，对`a`数组中的第 2 个元素进行了修改，但未影响`sChar`的值。
4. `String(char[] value,int offset,int count)`
分配一个新的`String`，它包含来自该字符数组参数一个子数组的字符。`offset`参数是子数组第一个字符的索引，`count`参数指定子数组的长度。该子数组的内容已被赋值，后续对字符数组的修改不会影响新创建的字符串。
```java
char a[]={'H','e','l','l','o'};
String sChar=new String(a,1,4);
a[1]='s';
```
上述`sChar`变量的值是字符串`"ello"`。该构造方法使用字符数组中的部分连续元素来创建字符串对象。
# String字符串和整型int的相互转换
## String转换为int
String 字符串转整型 int 有以下两种方式：
* `Integer.parseInt(str)`
* `Integer.valueOf(str).intValue()`

```java
public static void main(String[] args) {
  String str = "123";
  int n = 0;
  // 第一种转换方法：Integer.parseInt(str)
  n = Integer.parseInt(str);
  System.out.println("Integer.parseInt(str) : " + n);
  // 第二种转换方法：Integer.valueOf(str).intValue()
  n = 0;
  n = Integer.valueOf(str).intValue();
  System.out.println("Integer.parseInt(str) : " + n);
}
//输出结果为：
//Integer.parseInt(str) : 123
//Integer.parseInt(str) : 123
```
在`String`转换`int`时，`String`的值一定是整数，否则会报数字转换异常（`java.lang.NumberFormatException`）。
## int转换为String
整型`int`转`String`字符串类型有以下 3 种方法：
* `String s = String.valueOf(i);`
* `String s = Integer.toString(i);`
* `String s = "" + i;`

```java
public static void main(String[] args) {
  int num = 10;
  // 第一种方法：String.valueOf(i);
  num = 10;
  String str = String.valueOf(num);
  System.out.println("str:" + str);
  // 第二种方法：Integer.toString(i);
  num = 10;
  String str2 = Integer.toString(num);
  System.out.println("str2:" + str2);
  // 第三种方法："" + i;
  String str3 = num + "";
  System.out.println("str3:" + str3);
}
//输出结果为：
//str:10
//str2:10
//str3:10
```

使用第三种方法相对第一第二种耗时比较大。在使用第一种`valueOf()`方法时，注意`valueOf`括号中的值不能为空，否则会报空指针异常（`NullPointerException`）。
## valueOf() 、parse()和toString()
### valueOf()
valueOf() 方法将数据的内部格式转换为可读的形式。它是一种静态方法，对于所有 Java 内置的类型，在字符串内被重载，以便每一种类型都能被转换成字符串。valueOf() 方法还被类型 Object 重载，所以创建的任何形式类的对象也可被用作一个参数。这里是它的几种形式：
```java
static String valueOf(double num)
static String valueOf(long num)
static String valueOf(Object ob)
static String valueOf(char chars[])
```
调用 valueOf() 方法可以得到其他类型数据的字符串形式——例如在进行连接操作时。对各种数据类型，可以直接调用这种方法得到合理的字符串形式。所有的简单类型数据转换成相应于它们的普通字符串形式。任何传递给 valueOf() 方法的对象都将返回对象的 toString() 方法调用的结果。事实上，也可以通过直接调用 toString() 方法而得到相同的结果。

对大多数数组，valueOf() 方法返回一个相当晦涩的字符串，这说明它是一个某种类型的数组。然而对于字符数组，它创建一个包含了字符数组中的字符的字符串对象。valueOf() 方法有一种特定形式允许指定字符数组的一个子集。

它具有如下的一般形式：
static String valueOf(char chars[ ], int startIndex, int numChars)

这里 chars 是存放字符的数组，startIndex 是字符数组中期望得到的子字符串的首字符下标，numChars 指定子字符串的长度。
### parse()
parseXxx(String) 这种形式，是指把字符串转换为数值型，其中 Xxx 对应不同的数据类型，然后转换为 Xxx 指定的类型，如 int 型和 float 型。
### toString()
toString() 可以把一个引用类型转换为 String 字符串类型，是 sun 公司开发 Java 的时候为了方便所有类的字符串操作而特意加入的一个方法。
# 字符串拼接
String 字符串虽然是不可变字符串，但也可以进行拼接只是会产生一个新的对象。String 字符串拼接可以使用“+”运算符或 String 的 concat(String str) 方法。“+”运算符优势是可以连接任何类型数据拼接成为字符串，而 concat 方法只能拼接 String 类型字符串。
## 使用连接运算符“+”
Java 语言允许使用“+”号连接（拼接）两个字符串。在使用“+”运算符连接字符串和 int 型（或 double 型）数据时，“+”将 int（或 double）型数据自动转换成 String 类型。

只要“+”运算符的一个操作数是字符串，编译器就会将另一个操作数转换成字符串形式，所以应该谨慎地将其他数据类型与字符串相连，以免出现意想不到的结果。
## 使用 concat() 方法
在 Java 中，String 类的 concat() 方法实现了将一个字符串连接到另一个字符串的后面。concat() 方法语法格式如下：
字符串 1.concat(字符串 2);
执行结果是字符串 2 被连接到字符串 1 后面，形成新的字符串。

concat() 方法一次只能连接两个字符串，如果需要连接多个字符串，需要调用多次 concat() 方法。
# 获取字符串长度
在 Java 中，要获取字符串的长度，可以使用 String 类的 length() 方法，其语法形式如下：
```
字符串名.length();
```
# 字符串大小写转换
String 类的 toLowerCase() 方法可以将字符串中的所有字符全部转换成小写，而非字母的字符不受影响。语法格式如下：
字符串名.toLowerCase()    // 将字符串中的字母全部转换为小写，非字母不受影响
toUpperCase() 则将字符串中的所有字符全部转换成大写，而非字母的字符不受影响。语法格式如下：
字符串名.toUpperCase()    // 将字符串中的字母全部转换为大写，非字母不受影响
```java
String str="abcdef 我 ghijklmn";
System.out.println(str.toLowerCase());    // 输出：abcdef 我 ghijklmn
System.out.println(str.toUpperCase());    // 输出：ABCDEF 我 GHIJKLMN
```
# 去除字符串中的空格
字符串中存在的首尾空格一般情况下都没有任何意义，如字符串“ Hello ”，但是这些空格会影响到字符串的操作，如连接字符串或比较字符串等，所以应该去掉字符串中的首尾空格，这需要使用 String 类提供的 trim() 方法。

trim() 方法的语法形式如下：
字符串名.trim()

使用 trim() 方法的示例如下：
String str = " hello ";
System.out.println(str.length());    // 输出 7
System.out.println(str.trim().length());    // 输出 5
从该示例中可以看出，字符串中的每个空格占一个位置，直接影响了计算字符串的长度。

如果不确定要操作的字符串首尾是否有空格，最好在操作之前调用该字符串的 trim() 方法去除首尾空格，然后再对其进行操作。

注意：trim() 只能去掉字符串中前后的半角空格（英文空格），而无法去掉全角空格（中文空格）。可用以下代码将全角空格替换为半角空格再进行操作，其中替换是 String 类的 replace() 方法，《Java字符串的替换》一节会详细介绍这个方法。
str = str.replace((char) 12288, ' ');    // 将中文空格替换为英文空格
str = str.trim();

其中，12288 是中文全角空格的 unicode 编码。
# 截取（提取）子字符串
在 String 中提供了两个截取字符串的方法，一个是从指定位置截取到字符串结尾，另一个是截取指定范围的内容。
## 1. substring(int beginIndex) 形式
此方式用于提取从索引位置开始至结尾处的字符串部分。调用时，括号中是需要提取字符串的开始位置，方法的返回值是提取的字符串。
```java
String str = "我爱 Java 编程";
String result = str.substring(3);
System.out.println(result);    // 输出：Java 编程
```
## 2. substring(int beginIndex，int endIndex) 形式
`beginIndex`表示截取的起始索引，截取的字符串中包括起始索引对应的字符；`endIndex`表示结束索引，截取的字符串中不包括结束索引对应的字符，如果不指定`endIndex`，则表示截取到目标字符串末尾。该方法用于提取位置`beginIndex`和位置`endIndex`位置之间的字符串部分。

注意：substring() 方法是按字符截取，而不是按字节截取。
# 分割字符串
String 类的 split() 方法可以按指定的分割符对目标字符串进行分割，分割后的内容存放在字符串数组中。该方法主要有如下两种重载形式：
str.split(String sign)
str.split(String sign,int limit)
其中它们的含义如下：
str 为需要分割的目标字符串。
sign 为指定的分割符，可以是任意字符串。
limit 表示分割后生成的字符串的限制个数，如果不指定，则表示不限制，直到将整个目标字符串完全分割为止。

使用分隔符注意如下：

1）“.”和“|”都是转义字符，必须得加“\\”。
如果用“.”作为分隔的话，必须写成String.split("\\.")，这样才能正确的分隔开，不能用String.split(".")。
如果用“|”作为分隔的话，必须写成String.split("\\|")，这样才能正确的分隔开，不能用String.split("|")。

2）如果在一个字符串中有多个分隔符，可以用“|”作为连字符，比如：“acount=? and uu =? or n=?”，把三个都分隔出来，可以用String.split("and|or")。
# 字符串的替换
String 类提供了 3 种字符串替换方法，分别是 replace()、replaceFirst() 和 replaceAll()。
replace() 方法
replace() 方法用于将目标字符串中的指定字符（串）替换成新的字符（串），其语法格式如下：
字符串.replace(String oldChar, String newChar)
其中，oldChar 表示被替换的字符串；newChar 表示用于替换的字符串。replace() 方法会将字符串中所有 oldChar 替换成 newChar。
replaceFirst() 方法
replaceFirst() 方法用于将目标字符串中匹配某正则表达式的第一个子字符串替换成新的字符串，其语法形式如下：
字符串.replaceFirst(String regex, String replacement)
其中，regex 表示正则表达式；replacement 表示用于替换的字符串。例如：
String words = "hello java，hello php";
String newStr = words.replaceFirst("hello","你好 ");
System.out.println(newStr);    // 输出：你好 java,hello php
replaceAll() 方法
replaceAll() 方法用于将目标字符串中匹配某正则表达式的所有子字符串替换成新的字符串，其语法形式如下：
字符串.replaceAll(String regex, String replacement)
其中，regex 表示正则表达式，replacement 表示用于替换的字符串。例如：
String words = "hello java，hello php";
String newStr = words.replaceAll("hello","你好 ");
System.out.println(newStr);    // 输出：你好 java,你好 php

public static void main(String[] args) {
    // 定义原始字符串
    String intro = "今天时星其天，外面时下雨天。妈米去买菜了，漏网在家写作业。" + "语文作业时”其”写 5 行，数学使第 10 页。";
    // 将文本中的所有"时"和"使"都替换为"是"
    String newStrFirst = intro.replaceAll("[时使]", "是");
    // 将文本中的所有"妈米"改为"妈妈"
    String newStrSecond = newStrFirst.replaceAll("妈米", "妈妈");
    // 将文本中的所有"漏网"改为"留我"
    String newStrThird = newStrSecond.replaceAll("漏网", "留我");
    // 将文本中第一次出现的"其"改为"期"
    String newStrFourth = newStrThird.replaceFirst("[其]", "期");
    // 输出最终字符串
    System.out.println(newStrFourth);
}
运行该程序，输出的正确字符串内容如下：
今天是星期天，外面是下雨天。妈妈去买菜了，留我在家写作业。语文作业是”其”写 5 行，数学是第 10 页。
# 字符串比较
