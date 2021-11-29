---
title: Java 内置包装类
date: 2020-09-09 17:51:12
tags: [java]
categories: java
---

# 包装类、装箱和拆箱
在 Java 的设计中提倡一种思想，即一切皆对象。但是从数据类型的划分中，我们知道 Java 中的数据类型分为基本数据类型和引用数据类型，但是基本数据类型怎么能够称为对象呢？于是 Java 为每种基本数据类型分别设计了对应的类，称之为包装类。

包装类和基本数据类型的关系：

|	基本数据类型 | 包装类 |
| :--: | :--: |
| byte | Byte |
| short | Short |
| int | Integer |
| long | Long |
|	char | Character |
|	float | Float |
|	double | Double |
|	boolean | Boolean |

## 装箱和拆箱
基本数据类型转换为包装类的过程称为装箱，例如把`int`包装成`Integer`类的对象；包装类变为基本数据类型的过程称为拆箱，例如把`Integer`类的对象重新简化为`int`。

在进行基本数据类型和对应的包装类转换时，系统将自动进行装箱及拆箱操作。
```java
public class Demo {
  public static void main(String[] args) {
    int m = 500;
    Integer obj = m;  // 自动装箱
    int n = obj;  // 自动拆箱
    System.out.println("n = " + n);
  
    Integer obj1 = 500;
    System.out.println("obj等价于obj1返回结果为" + obj.equals(obj1));
  }
}
```
运行结果：
```
n = 500
obj等价于obj1返回结果为true
```
## 包装类的应用
### 实现 int 和 Integer 的相互转换
可以通过`Integer`类的构造方法将`int`装箱，通过`Integer`类的`intValue`方法将`Integer`拆箱。
```java
public class Demo {
  public static void main(String[] args) {
    int m = 500;
    Integer obj = new Integer(m);  // 手动装箱
    int n = obj.intValue();  // 手动拆箱
    System.out.println("n = " + n);
    
    Integer obj1 = new Integer(500);
    System.out.println("obj等价于obj1的返回结果为" + obj.equals(obj1));
  }
}
```
运行结果：
```
n = 500
obj等价于obj1的返回结果为true
```
### 将字符串转换为数值类型
在`Integer`和`Float`类中分别提供了以下两种方法：
1. `Integer`类（`String`转`int`型）
```
int parseInt(String s);
```
`s`为要转换的字符串。
2. `Float`类（`String`转`float`型）
```
float parseFloat(String s)
```
注意：使用以上两种方法时，字符串中的数据必须由数字组成，否则转换时会出现程序错误。
```java
public class Demo {
  public static void main(String[] args) {
    String str1 = "30";
    String str2 = "30.3";
    // 将字符串变为int型
    int x = Integer.parseInt(str1);
    // 将字符串变为float型
    float f = Float.parseFloat(str2);
    System.out.println("x = " + x + "；f = " + f);
  }
}
```
运行结果：
```
x = 30；f = 30.3
```
### 将整数转换为字符串
`Integer`类有一个静态的`toString()`方法，可以将整数转换为字符串。
```java
public class Demo {
  public static void main(String[] args) {
    int m = 500;
    String s = Integer.toString(m);
    System.out.println("s = " + s);
  }
}
```
运行结果：
```
s = 500
```
# Object类
`Object`是 Java 类库中的一个特殊类，也是所有类的父类。也就是说，Java 允许把任何类型的对象赋给`Object`类型的变量。当一个类被定义后，如果没有指定继承的父类，那么默认父类就是`Object`类。因此，以下两个类表示的含义是一样的。
```java
public class MyClass{…}
// 等价于
public class MyClass extends Object {…}
```
由于 Java 所有的类都是`Object`类的子类，所以任何 Java 对象都可以调用`Object`类的方法。

`Object`类的常用方法：

| 方法 | 说明 |
| :--: | :--: |
| Object clone()    | 创建与该对象的类相同的新对象 |
| boolean equals(Object) | 比较两对象是否相等 |
| void finalize()   | 当垃圾回收器确定不存在对该对象的更多引用时，对象垃圾回收器调用该方法 |
| Class getClass()	| 返回一个对象运行时的实例类 |
| int hashCode()    | 返回该对象的散列码值 |
| void notify()     | 激活等待在该对象的监视器上的一个线程 |
| void notifyAll()	| 激活等待在该对象的监视器上的全部线程 |
| String toString()	| 返回该对象的字符串表示 |
| void wait()       | 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待 |

## toString() 方法
`toString()`方法返回该对象的字符串，当程序输出一个对象或者把某个对象和字符串进行连接运算时，系统会自动调用该对象的`toString()`方法返回该对象的字符串表示。

`Object`类的`toString()`方法返回“运行时类名@十六进制哈希码”格式的字符串，但很多类都重写了`Object`类的`toString()`方法，用于返回可以表述该对象信息的字符串。

> 哈希码（`hashCode`），每个 Java 对象都有哈希码属性，哈希码可以用来标识对象，提高对象在集合操作中的执行效率。

```java
// 定义Demo类，实际上继承Object类
class Demo {}
public class ObjectDemo01 {
  public static void main(String[] args) {
    Demo d = new Demo(); // 实例化Demo对象
    System.out.println("不加toString()输出：" + d);
    System.out.println("加上toString()输出：" + d.toString());
  }
}
```
输出结果为：
```
不加toString()输出：Demo@15db9742
加上toString()输出：Demo@15db9742
```
以上的程序是随机输出了一些地址信息，从程序的运行结果可以清楚的发现，加和不加`toString()`的最终输出结果是一样的，也就是说对象输出时一定会调用`Object`类中的`toString()`方法打印内容。所以利用此特性就可以通过`toString()`取得一些对象的信息，如下面代码。
```java
public class Person {
  private String name;
  private int age;
  public Person(String name, int age) {
    this.name = name;
    this.age = age;
  }
  public String toString() {
    return "姓名：" + this.name + "：年龄" + this.age;
  }
  public static void main(String[] args) {
    Person per = new Person("张三", 30);// 实例化Person对象
    System.out.println("对象信息：" + per);// 打印对象调用toString()方法
  }
}
```
输出结果为：
```
对象信息：姓名：张三：年龄30
```
## equals() 方法
`==`运算符是比较两个引用变量是否指向同一个实例，`equals()`方法是比较两个对象的内容是否相等，通常字符串的比较只是关心内容是否相等。

其使用格式如下：
```java
boolean result = obj.equals(Object o);
```
其中，`obj`表示要进行比较的一个对象，`o`表示另一个对象。
## getClass() 方法
`getClass()`方法返回对象所属的类，是一个`Class`对象。通过`Class`对象可以获取该类的各种信息，包括类名、父类以及它所实现接口的名字等。
```java
public class Test02 {
  public static void printClassInfo(Object obj) {
    // 获取类名
    System.out.println("类名：" + obj.getClass().getName());
    // 获取父类名
    System.out.println("父类：" + obj.getClass().getSuperclass().getName());
    System.out.println("实现的接口有：");
    // 获取实现的接口并输出
    for (int i = 0; i < obj.getClass().getInterfaces().length; i++) {
      System.out.println(obj.getClass().getInterfaces()[i]);
    }
  }
  public static void main(String[] args) {
    String strObj = new String();
    printClassInfo(strObj);
  }
}
```
该程序的运行结果如下：
```
类名：java.lang.String
父类：java.lang.Object
实现的接口有：
interface java.io.Serializable
interface java.lang.Comparable
interface java.lang.CharSequence
```
## 接收任意引用类型的对象
既然`Object`类是所有对象的父类，则所有的对象都可以向`Object`进行转换，在这其中也包含了数组和接口类型，即一切的引用数据类型都可以使用`Object`进行接收。
```java
interface A {
  public String getInfo();
}
class B implements A {
  public String getInfo() {
    return "Hello World!!!";
  }
}
public class ObjectDemo04 {
  public static void main(String[] args) {
    // 为接口实例化
    A a = new B();
    // 对象向上转型
    Object obj = a;
    // 对象向下转型
    A x = (A) obj;
    System.out.println(x.getInfo());
  }
}
```
输出结果为：
```
Hello World!!!
```
通过以上代码可以发现，虽然接口不能继承一个类，但是依然是`Object`类的子类，因为接口本身是引用数据类型，所以可以进行向上转型操作。

同理，也可以使用`Object`接收一个数组，因为数组本身也是引用数据类型。
```java
public class ObjectDemo05 {
  public static void main(String[] args) {
    int temp[] = { 1, 3, 5, 7, 9 };
    // 使用object接收数组
    Object obj = temp;
    // 传递数组引用
    print(obj);
  }
  public static void print(Object o) {
    // 判断对象的类型
    if (o instanceof int[]) {
      // 向下转型
      int x[] = (int[]) o;
      // 循环输出
      for (int i = 0; i < x.length; i++) {
        System.out.print(x[i] + "\t");
      }
    }
  }
}
```
输出结果为：
```
1 3 5 7 9
```
以上程序使用`Object`接收一个整型数组，因为数组本身属于引用数据类型，所以可以使用`Object`接收数组内容，在输出时通过`instanceof`判断类型是否是一个整型数组，然后进行输出操作。

提示：因为`Object`类可以接收任意的引用数据类型，所以在很多的类库设计上都采用`Object`作为方法的参数，这样操作起来会比较方便。
# Integer类
`Integer`类在对象中包装了一个基本类型`int`的值。`Integer`类对象包含一个`int`类型的字段。此外，该类提供了多个方法，能在`int`类型和`String`类型之间互相转换，还提供了处理`int`类型时非常有用的其他一些常量和方法。
## Integer 类的构造方法
`Integer`类中的构造方法有以下两个：
* `Integer(int value)`：构造一个新分配的`Integer`对象，它表示指定的`int`值。
* `Integer(String s)`：构造一个新分配的`Integer`对象，它表示`String`参数所指示的`int`值。

```java
Integer integer1 = new Integer(100);    // 以 int 型变量作为参数创建 Integer 对象
Integer integer2 = new Integer("100");    // 以 String 型变量作为参数创建 Integer 对象
```
## Integer 类的常用方法
`Integer`类中的常用方法：

| 方法	返回值	功能
| :--: | :--: | :--: |
| byteValue()                       | byte    | 以 byte 类型返回该 Integer 的值 |
| shortValue()                      | short   | 以 short 类型返回该 Integer 的值 |
| intValue()                        | int     | 以 int 类型返回该 Integer 的值 |
| toString()                        | String	| 返回一个表示该 Integer 值的 String 对象 |
| equals(Object obj)                | boolean	| 比较此对象与指定对象是否相等 |
| compareTo(Integeranotherlnteger)  | int     | 在数字上比较两个 Integer 对象，如相等，则返回 0；<br>如调用对象的数值小于 anotherlnteger 的数值，则返回负值；<br>如调用对象的数值大于 anotherlnteger 的数值，则返回正值 |
| valueOf(String s)                 | Integer	| 返回保存指定的 String 值的 Integer 对象 |
| parseInt(String s)                | int     | 将数字字符串转换为 int 数值 |

```java
String str = "456";
int num = Integer.parseInt(str);    // 将字符串转换为int类型的数值
int i = 789;
String s = Integer.toString(i);    // 将int类型的数值转换为字符串
```
注意：在实现将字符串转换为 int 类型数值的过程中，如果字符串中包含非数值类型的字符，则程序执行将出现异常。
例 1
编写一个程序，在程序中创建一个 String 类型变量，然后将它转换为二进制、八进制、十进制和十六进制输出。
```java
public class Test03 {
  public static void main(String[] args) {
    int num = 40;
    String str = Integer.toString(num); // 将数字转换成字符串
    String str1 = Integer.toBinaryString(num); // 将数字转换成二进制
    String str2 = Integer.toHexString(num); // 将数字转换成八进制
    String str3 = Integer.toOctalString(num); // 将数字转换成十六进制
    System.out.println(str + "的二进制数是：" + str1);
    System.out.println(str + "的八进制数是：" + str3);
    System.out.println(str + "的十进制数是：" + str);
    System.out.println(str + "的十六进制数是：" + str2);
  }
}
```
运行后的输出结果如下：
```
40的二进制数是：101000
40的八进制数是：50
40的十进制数是：40
40的十六进制数是：28
```
## Integer 类的常量
`Integer`类包含以下 4 个常量。
* `MAX_VALUE`：值为 231-1 的常量，它表示`int`类型能够表示的最大值。
* `MIN_VALUE`：值为 -231 的常量，它表示`int`类型能够表示的最小值。
* `SIZE`：用来以二进制补码形式表示`int`值的比特位数。
* `TYPE`：表示基本类型`int`的`Class`实例。

```java
int max_value = Integer.MAX_VALUE;    // 获取 int 类型可取的最大值
int min_value = Integer.MIN_VALUE;    // 获取 int 类型可取的最小值
int size = Integer.SIZE;    // 获取 int 类型的二进制位
Class c = Integer.TYPE;    // 获取基本类型 int 的 Class 实例
```
# Float类
`Float`类在对象中包装了一个基本类型`float`的值。`Float`类对象包含一个`float`类型的字段。此外，该类提供了多个方法，能在`float`类型与`String`类型之间互相转换，同时还提供了处理`float`类型时比较常用的常量和方法。
## Float 类的构造方法
`Float`类中的构造方法有以下 3 个。
* `Float(double value)`：构造一个新分配的`Float`对象，它表示转换为`float`类型的参数。
* `Float(float value)`：构造一个新分配的`Float`对象，它表示基本的`float`参数。
* `Float(String s)`：构造一个新分配的`Float`对象，它表示`String`参数所指示的`float`值。

```java
Float float1 = new Float(3.14145);    // 以 double 类型的变量作为参数创建 Float 对象
Float float2 = new Float(6.5);    // 以 float 类型的变量作为参数创建 Float 对象
Float float3 = new Float("3.1415");    // 以 String 类型的变量作为参数创建 Float 对象
```
`Float`类中的常用方法：

| 方法 | 返回值 | 功能 |
| :--: | :--: | :--: |
| byteValue()           | byte    | 以 byte 类型返回该 Float 的值 |
| doubleValue()         | double	| 以 double 类型返回该 Float 的值 |
| floatValue()          | float   | 以 float 类型返回该 Float 的值 |
| intValue()            | int     | 以 int 类型返回该 Float 的值（强制转换为 int 类型） |
| longValue()           | long    | 以 long 类型返回该 Float 的值（强制转换为 long 类型） |
| shortValue()          | short   | 以 short 类型返回该 Float 的值（强制转换为 short 类型） |
| isNaN()               | boolean | 如果此 Float 值是一个非数字值，则返回 true，否则返回 false |
| isNaN(float v)        | boolean | 如果指定的参数是一个非数字值，则返回 true，否则返回 false |
| toString()            | String  | 返回一个表示该 Float 值的 String 对象 |
| valueOf(String s)     | Float   | 返回保存指定的 String 值的 Float 对象 |
| parseFloat(String s)  | float   | 将数字字符串转换为 float 数值 |

```java
String str = "456.7";
float num = Float.parseFloat(str);    // 将字符串转换为 float 类型的数值
float f = 123.4f;
String s = Float.toString(f);    // 将 float 类型的数值转换为字符串
```
注意：在实现将字符串转换为`float`类型数值的过程中，如果字符串中包含非数值类型的字符，则程序执行将出现异常。
## Float 类的常用常量
在`Float`类中包含了很多常量，其中较为常用的常量如下。
* `MAX_VALUE`：值为`1.4E38`的常量，它表示`float`类型能够表示的最大值。
* `MIN_VALUE`：值为`3.4E-45`的常量，它表示`float`类型能够表示的最小值。
* `MAX_EXPONENT`：有限`float`变量可能具有的最大指数。
* `MIN_EXPONENT`：标准化`float`变量可能具有的最小指数。
* `MIN_NORMAL`：保存`float`类型数值的最小标准值的常量，即 2-126。
* `NaN`：保存`float`类型的非数字值的常量。
* `SIZE`：用来以二进制补码形式表示`float`值的比特位数。
* `TYPE`：表示基本类型`float`的`Class`实例。

```java
float max_value = Float.MAX_VALUE;    // 获取 float 类型可取的最大值
float min_value = Float.MIN_VALUE;    // 获取 float 类型可取的最小值
float min_normal = Float.MIN_NORMAL;    // 获取 float 类型可取的最小标准值
float size = Float.SIZE;    // 获取 float 类型的二进制位
```
# Double类
`Double`类在对象中包装了一个基本类型`double`的值。`Double`类对象包含一个`double`类型的字段。此外，该类还提供了多个方法，可以将`double`类型与`String`类型相互转换，同时 还提供了处理`double`类型时比较常用的常量和方法。
## Double 类的构造方法
`Double`类中的构造方法有如下两个。
* `Double(double value)`：构造一个新分配的`Double`对象，它表示转换为`double`类型的参数。
* `Double(String s)`：构造一个新分配的`Double`对象，它表示`String`参数所指示的`double`值。

```java
Double double1 = new Double(5.456);    // 以 double 类型的变量作为参数创建 Double 对象
Double double2 = new Double("5.456");       // 以 String 类型的变量作为参数创建 Double 对象
```
## Double 类的常用方法
`Double`类中的常用方法：

| 方法 | 返回值 | 功能 |
| :--: | :--: | :--: |
| byteValue()           | byte	  | 以 byte 类型返回该 Double 的值 |
| doubleValue()         | double	| 以 double 类型返回该 Double 的值 |
| fioatValue()          | float	  | 以 float 类型返回该 Double 的值 |
| intValue()            | int	    | 以 int 类型返回该 Double 的值（强制转换为 int 类型） |
| longValue()           | long	  | 以 long 类型返回该 Double 的值（强制转换为 long 类型） |
| shortValue()          | short	  | 以 short 类型返回该 Double 的值（强制转换为 short 类型） |
| isNaN()               | boolean	| 如果此 Double 值是一个非数字值，则返回 true，否则返回 false |
| isNaN(double v)       | boolean	| 如果指定的参数是一个非数字值，则返回 true，否则返回 false |
| toString()            | String	| 返回一个表示该 Double 值的 String 对象 |
| valueOf(String s)     | Double	| 返回保存指定的 String 值的 Double 对象 |
| parseDouble(String s) | double	| 将数字字符串转换为 Double 数值 |

```java
String str = "56.7809";
double num = Double.parseDouble(str);    // 将字符串转换为 double 类型的数值
double d = 56.7809;
String s = Double.toString(d);    // 将double类型的数值转换为字符串
```
在将字符串转换为`double`类型的数值的过程中，如果字符串中包含非数值类型的字符，则程序执行将出现异常。
## Double 类的常用常量
在`Double`类中包含了很多常量，其中较为常用的常量如下。
* `MAX_VALUE`：值为`1.8E308`的常量，它表示`double`类型的最大正有限值的常量。
* `MIN_VALUE`：值为`4.9E-324`的常量，它表示`double`类型数据能够保持的最小正非零值的常量。
* `NaN`：保存`double`类型的非数字值的常量。
* `NEGATIVE_INFINITY`：保持`double`类型的负无穷大的常量。
* `POSITIVE_INFINITY`：保持`double`类型的正无穷大的常量。
* `SIZE`：用秦以二进制补码形式表示`double`值的比特位数。
* `TYPE`：表示基本类型`double`的`Class`实例。

# Number类
`Number`是一个抽象类，也是一个超类（即父类）。`Number`类属于`java.lang`包，所有的包装类（如`Double、Float、Byte、Short、Integer`以及`Long`）都是抽象类`Number`的子类。

`Number`类定义了一些抽象方法，以各种不同数字格式返回对象的值。如`xxxValue()`方法，它将`Number`对象转换为`xxx`数据类型的值并返回。这些方法如下表所示：

| 方法 | 说明 |
| :--: | :--: |
| byte byteValue(); | 返回 byte 类型的值 |
| double doubleValue(); | 返回 double 类型的值 |
| float floatValue(); | 返回 float 类型的值 |
| int intValue(); | 返回 int 类型的值 |
| long longValue(); | 返回 long 类型的值 |
| short shortValue(); | 返回 short 类型的值 |

抽象类不能直接实例化，而是必须实例化其具体的子类。
```java
Number num = new Double(12.5);
System.out.println("返回 double 类型的值：" + num.doubleValue());
System.out.println("返回 int 类型的值：" + num.intValue());
System.out.println("返回 float 类型的值：" + num.floatValue());
```
执行上述代码，输出结果如下：
```
返回 double 类型的值：12.5
返回 int 类型的值：12
返回 float 类型的值：12.5
```
# Character类
`Character`类是字符数据类型`char`的包装类。`Character`类的对象包含类型为`char`的单个字段，这样能把基本数据类型当对象来处理，其常用方法如表所示。

| 方法 | 描述 |
| :--: | :--: |
| void Character(char value) | 构造一个新分配的 Character 对象，用以表示指定的 char 值 |
| char charValue() | 返回此 Character 对象的值，此对象表示基本 char 值 |
| int compareTo(Character anotherCharacter) | 根据数字比较两个 Character 对象 |
| boolean equals(Character anotherCharacter) | 将此对象与指定对象比较，当且仅当参数不是 null，而是一个与此对象包含相同 char 值的 Character 对象时， 结果才是 true |
| boolean isDigit(char ch) | 确定指定字符是否为数字，如果通过 Character. getType(ch) 提供的字符的常规类别类型为 DECIMAL_DIGIT_NUMBER，则字符为数字 |
| boolean isLetter(int codePoint) | 确定指定字符（Unicode 代码点）是否为字母 |
| boolean isLetterOrDigit(int codePoint) | 确定指定字符（Unicode 代码点）是否为字母或数字 |
| boolean isLowerCase(char ch) | 确定指定字符是否为小写字母 |
| boolean isUpperCase(char ch) | 确定指定字符是否为大写字母 |
| char toLowerCase(char ch) | 使用来自 UnicodeData 文件的大小写映射信息将字符参数转换为小写 |
| char toUpperCase(char ch) | 使用来自 UnicodeData 文件的大小写映射信息将字符参数转换为大写 |

可以从`char`值中创建一个`Character`对象。例如，下列语句为字符`S`创建了一个`Character`对象。
```java
Character character = new Character('S');
```
`CompareTo()`方法将这个字符与其他字符比较，并且返回一个整型数组，这个值是两个字符比较后的标准代码差值。当且仅当两个字符相同时，`equals()`方法的返回值才为`true`。
```java
Character character = new Character('A');
int result1 = character.compareTo(new Character('V'));
System.out.println(result1);    // 输出：0
int result2 = character.compareTo(new Character('B'));
System.out.println(resuit2);    // 输出：-1
int result3 = character.compareTo(new Character('1'));
System.out.println(result3);    // 输出：-2
```
# Boolean类
`Boolean`类将基本类型为`boolean`的值包装在一个对象中。一个`Boolean`类的对象只包含一个类型为`boolean`的字段。此外，此类还为`boolean`和`String`的相互转换提供了很多方法，并提供了处理`boolean`时非常有用的其他一些常用方法。
## Boolean 类的构造方法
`Boolean`类有以下两种构造形式：
```java
Boolean(boolean boolValue);
Boolean(String boolString);
```
其中`boolValue`必须是`true`或`false`（不区分大小写），`boolString`包含字符串`true`（不区分大小写），那么新的`Boolean`对象将包含`true`；否则将包含`false`。
## Boolean 类的常用方法
在`Boolean`类内部包含了一些和`Boolean`操作有关的方法。

| 方法 | 返回值 | 功能 |
| :--: | :--: | :--: |
| booleanValue() | boolean | 将 Boolean 对象的值以对应的 boolean 值返回 |
| equals(Object obj) | boolean | 判断调用该方法的对象与 obj 是否相等。当且仅当参数不是 null，且与调用该方法的对象一样都表示同一个 boolean 值的 Boolean 对象时，才返回 true |
| parseBoolean(String s) | boolean | 将字符串参数解析为 boolean 值 |
| toString() | string | 返回表示该 boolean 值的 String 对象 |
| valueOf(String s) | boolean | 返回一个用指定的字符串表示的 boolean 值 |

```java
public class Test05 {
  public static void main(String[] args) {
    Boolean b1 = new Boolean(true);
    Boolean b2 = new Boolean("ok");
    Boolean b3 = new Boolean("true");
    System.out.println("b1 转换为 boolean 值是：" + b1);
    System.out.println("b2 转换为 boolean 值是：" + b2);
    System.out.println("b3 转换为 boolean 值是：" + b3);
  }
}
```
运行后的输出结果如下：
```
b1 转换为 boolean 值是：true
b2 转换为 boolean 值是：false
b3 转换为 boolean 值是：true
```
## Boolean 类的常用常量
在`Boolean`类中包含了很多的常量，其中较为常用的常量如下。
* `TRUE`：对应基值`true`的`Boolean`对象。
* `FALSE`：对应基值`false`的`Boolean`对象。
* `TYPE`：表示基本类型`boolean`的`Class`对象。

# Byte类
`Byte`类将基本类型为`byte`的值包装在一个对象中。一个`Byte`类的对象只包含一个类型为`byte`的字段。此外，该类还为`byte`和`String`的相互转换提供了方法，并提供了一些处理`byte`时非常有用的常量和方法。
## Byte 类的构造方法
`Byte`类提供了两个构造方法来创建`Byte`对象。
1. `Byte(byte value)`
通过这种方法创建的`Byte`对象，可以表示指定的`byte`值。例如，下面的示例将 5 作为`byte`类型变量，然后再创建`Byte`对象。
```java
byte my_byte = 5;
Byte b = new Byte(my_byte);
```
2. `Byte(String s)`
通过这个方法创建的`Byte`对象，可表示`String`参数所指定的`byte`值。例如，下面的示例将 5 作为`String`类型变量，然后再创建`Byte`对象。
```java
String my_byte = "5";
Byte b = new Byte(my_byte);
```
注意：必须使用数值型的`String`变量作为参数才能创建成功，否则会抛出`NumberFormatException`异常。
## Byte 类的常用方法
在`Byte`类内部包含了一些和`Byte`操作有关的方法。

| 方法 | 返回值 | 功能 |
| :--: | :--: | :--: |
| byteValue() | byte | 以一个 byte 值返回 Byte 对象 |
| compareTo(Byte bytel) | int | 在数字上比较两个 Byte 对象 |
| doubleValue()	double	以一个 double 值返回此 Byte 的值 |
| intValue() | int | 以一个 int 值返回此 Byte 的值 |
| parseByte(String s) | byte | 将 String 型参数解析成等价的 byte 形式 |
| toStringO | String | 返回表示此 byte 值的 String 对象 |
| valueOf(String s) | Byte | 返回一个保持指定 String 所给出的值的 Byte 对象 |
| equals(Object obj) | boolean | 将此对象与指定对象比较，如果调用该方法的对象与 obj 相等 则返回 true，否则返回 false |

## Byte 类的常用常量
在`Byte`类中包含了很多的常量，其中较为常用的常量如下。
* `MIN_VALUE`：`byte`类可取的最小值。
* `MAX_VALUE`：`byte`类可取的最大值。
* `SIZE`：用于以二进制补码形式表示的`byte`值的位数。
* `TYPE`：表示基本类`byte`的`Class`实例。

# System类
`System`类位于`java.lang`包，代表当前 Java 程序的运行平台，系统级的很多属性和控制方法都放置在该类的内部。由于该类的构造方法是`private`的，所以无法创建该类的对象，也就是无法实例化该类。

`System`类提供了一些类变量和类方法，允许直接通过`System`类来调用这些类变量和类方法。
## System 类的成员变量
`System`类有 3 个静态成员变量，分别是`PrintStream out、InputStream in`和`PrintStream err`。
1. `PrintStream out`
标准输出流。此流已打开并准备接收输出数据。通常，此流对应于显示器输出或者由主机环境或用户指定的另一个输出目标。
```java
System.out.println(data);
```
其中，`println`方法是属于流类`PrintStream`的方法，而不是`System`中的方法。
2. `InputStream in`
标准输入流。此流已打开并准备提供输入数据。通常，此流对应于键盘输入或者由主机环境或用户指定的另一个输入源。
3. `PrintStream err`
标准的错误输出流。其语法与`System.out`类似，不需要提供参数就可输出错误信息。也可以用来输出用户指定的其他信息，包括变量的值。

```java
import java.io.IOException;
public class Test06 {
  public static void main(String[] args) {
    System.out.println("请输入字符，按回车键结束输入:");
    int c;
    try {
      c = System.in.read();    // 读取输入的字符
      while(c != '\r') {    // 判断输入的字符是不是回车
        System.out.print((char) c);    // 输出字符
        c = System.in.read();
      }
    } catch(IOException e) {
      System.out.println(e.toString());
    } finally {
      System.err.println();
    }
  }
}
```
以上代码中，`System.in.read()`语句读入一个字符，`read()`方法是`InputStream`类拥有的方法。变量`c`必须用`int`类型而不能用`char`类型，否则会因为丢失精度而导致编译失败。

以上的程序如果输入汉字将不能正常输出。如果要正常输出汉字，需要把`System.in`声明为`InputStreamReader`类型的实例，最终在`try`语句块中的代码为：
```java
InputStreamReader in = new InputStreamReader(System.in, "GB2312");
c = in.read();
while(c != '\r') {
  System.out.print((char) c);
  c = in.read();
}
```
如上述代码所示，语句`InputStreamReader in=new InputStreamReader(System.in,"GB2312")`声明一个新对象`in`，它从`Reader`继承而来，此时就可以读入完整的`Unicode`码，显示正常的汉字。
## System 类的成员方法
`System`类中提供了一些系统级的操作方法，常用的方法有`arraycopy()、currentTimeMillis()、exit()、gc()`和`getProperty()`。
### 1. arraycopy() 方法
该方法的作用是数组复制，即从指定源数组中复制一个数组，复制从指定的位置开始，到目标数组的指定位置结束。该方法的具体定义如下：
```java
public static void arraycopy(Object src,int srcPos,Object dest,int destPos,int length)
```
其中，`src`表示源数组，`srcPos`表示从源数组中复制的起始位置，`dest`表示目标数组，`destPos`表示要复制到的目标数组的起始位置，`length`表示复制的个数。

```java
public class System_arrayCopy {
  public static void main(String[] args) {
    char[] srcArray = {'A','B','C','D'};
    char[] destArray = {'E','F','G','H'};
    System.arraycopy(srcArray,1,destArray,1,2);
    System.out.println("源数组：");
    for(int i = 0;i < srcArray.length;i++) {
      System.out.println(srcArray[i]);
    }
    System.out.println("目标数组：");
    for(int j = 0;j < destArray.length;j++) {
      System.out.println(destArray[j]);
    }
  }
}
```
如上述代码，将数组 srcArray 中，从下标 1 开始的数据复制到数组`destArray`从下标 1 开始的位置，总共复制两个。也就是将`srcArray[1]`复制给`destArray[1]`，将`srcArray[2]`复制给`destArray[2]`。这样经过复制之后，数组`srcArray`中的元素不发生变化，而数组`destArray`中的元素将变为`E、B、C、 H`，下面为输出结果：
```
源数组：
A
B
C
D
目标数组：
E
B
C
H
```
### 2. currentTimeMillis() 方法
该方法的作用是返回当前的计算机时间，时间的格式为当前计算机时间与 GMT 时间（格林尼治时间）1970 年 1 月 1 日 0 时 0 分 0 秒所差的毫秒数。一般用它来测试程序的执行时间。
```java
long m = System.currentTimeMillis();
```
上述语句将获得一个长整型的数字，该数字就是以差值表达的当前时间。

使用`currentTimeMillis()`方法来显示时间不够直观，但是可以很方便地进行时间计算。例如，计算程序运行需要的时间就可以使用如下的代码：
```java
public class System_currentTimeMillis {
  public static void main(String[] args) {
    long start = System.currentTimeMillis();
    for(int i = 0;i < 100000000;i++) {
        int temp = 0;
    }
    long end = System.currentTimeMillis();
    long time = end - start;
    System.out.println("程序执行时间" + time + "秒");
  }
}
```
### 3. exit() 方法
该方法的作用是终止当前正在运行的 Java 虚拟机，具体的定义格式如下：
```java
public static void exit(int status)
```
其中，`status`的值为 0 时表示正常退出，非零时表示异常退出。使用该方法可以在图形界面编程中实现程序的退出功能等。
### 4. gc() 方法
该方法的作用是请求系统进行垃圾回收，完成内存中的垃圾清除。至于系统是否立刻回收，取决于系统中垃圾回收算法的实现以及系统执行时的情况。定义如下：
```java
public static void gc()
```
### 5. getProperty() 方法
该方法的作用是获得系统中属性名为 key 的属性对应的值，具体的定义如下：
```java
public static String getProperty(String key)
```
系统中常见的属性名以及属性的说明：

| 属性名 | 属性说明 |
| :--: | :--: |
| java.version | Java 运行时环境版本 |
| java.home | Java 安装目录 |
| os.name | 操作系统的名称 |
| os.version | 操作系统的版本 |
| user.name | 用户的账户名称 |
| user.home | 用户的主目录 |
| user.dir	用户的当前工作目录 |

```java
public class System_getProperty {
  public static void main(String[] args) {
    String jversion = System.getProperty("java.version");
    String oName = System.getProperty("os.name");
    String user = System.getProperty("user.name");
    System.out.println("Java 运行时环境版本："+jversion);
    System.out.println("当前操作系统是："+oName);
    System.out.println("当前用户是："+user);
  }
}
```
运行该程序，输出的结果如下：
```
Java 运行时环境版本：1.6.0_26
当前操作系统是：Windows 7
当前用户是：Administrator
```