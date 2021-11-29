---
title: Java 反射机制
date: 2020-10-14 18:54:15
tags: [java]
categories: java
---

# 编译期和运行期
编译期是指把源码交给编译器编译成计算机可以执行的文件的过程。也就是把 Java 代码编成`class`文件的过程。编译期只是做了一些翻译功能，并没有把代码放在内存中运行起来，而只是把代码当成文本进行操作，比如检查错误。

运行期是把编译后的文件交给计算机执行，直到程序运行结束。所谓运行期就把在磁盘中的代码放到内存中执行起来。

Java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能称为 Java 语言的反射机制。简单来说，反射机制指的是程序在运行时能够获取自身的信息。在 Java 中，只要给定类的名字，就可以通过反射机制来获得类的所有信息。

Java 反射机制在服务器程序和中间件程序中得到了广泛运用。在服务器端，往往需要根据客户的请求，动态调用某一个对象的特定方法。此外，在 ORM 中间件的实现中，运用 Java 反射机制可以读取任意一个`JavaBean`的所有属性，或者给这些属性赋值。

{% asset_img 1.png %}

Java 反射机制主要提供了以下功能，这些功能都位于`java.lang.reflect`包。
* 在运行时判断任意一个对象所属的类。
* 在运行时构造任意一个类的对象。
* 在运行时判断任意一个类所具有的成员变量和方法。
* 在运行时调用任意一个对象的方法。
* 生成动态代理。

要想知道一个类的属性和方法，必须先获取到该类的字节码文件对象。获取类的信息时，使用的就是`Class`类中的方法。所以先要获取到每一个字节码文件（`.class`）对应的`Class`类型的对象.

众所周知，所有 Java 类均继承了`Object`类，在`Object`类中定义了一个`getClass()`方法，该方法返回同一个类型为`Class`的对象。
```java
Class labelCls = label1.getClass();    // label1为 JLabel 类的对象
```
利用`Class`类的对象`labelCls`可以访问`labelCls`对象的描述信息、`JLabel`类的信息以及基类`Object`的信息。反射可访问的常用信息：

| 类型     | 访问方法              | 返回值类型      | 说明 |
| :--: | :--: | :--: | :--: |
| 包路径   | getPackage()         | Package 对象	| 获取该类的存放路径 |
| 类名称   | getName()            | String 对象     | 获取该类的名称 |
| 继承类   | getSuperclass()      | Class 对象      | 获取该类继承的类 |
| 实现接口 | getlnterfaces()      | Class 型数组	| 获取该类实现的所有接口 |
| 构造方法 | getConstructors()    | Constructor 型数组	| 获取所有权限为 public 的构造方法 |
| 构造方法 | getDeclaredContruectors() | Constructor 对象	| 获取当前对象的所有构造方法 |
| 方法     | getMethods()         | Methods 型数组  | 获取所有权限为 public 的方法 |
| 方法     | getDeclaredMethods() | Methods 对象    | 获取当前对象的所有方法 |
| 成员变量 | getFields()          | Field 型数组	| 获取所有权限为 public 的成员变量 |
| 成员变量 | getDeclareFileds()   | Field 对象      | 获取当前对象的所有成员变量 |
| 内部类   | getClasses()         | Class 型数组	| 获取所有权限为 public 的内部类 |
| 内部类   | getDeclaredClasses() | Class 型数组	| 获取所有内部类 |
| 内部类的声明类 | getDeclaringClass() | Class 对象 | 如果该类为内部类，则返回它的成员类，否则返回 null |

在调用`getFields()`和`getMethods()`方法时将会依次获取权限为`public`的字段和变量，然后将包含从超类中继承到的成员变量和方法。而通过`getDeclareFields()`和`getDeclareMethod()`只是获取在本类中定义的成员变量和方法。

## Java 反射机制的优缺点
优点：
* 能够运行时动态获取类的实例，大大提高系统的灵活性和扩展性。
* 与 Java 动态编译相结合，可以实现无比强大的功能。
* 对于 Java 这种先编译再运行的语言，能够让我们很方便的创建灵活的代码，这些代码可以在运行时装配，无需在组件之间进行源代码的链接，更加容易实现面向对象。

缺点：
* 反射会消耗一定的系统资源，因此，如果不需要动态地创建一个对象，那么就不需要用反射；
* 反射调用方法时可以忽略权限检查，获取这个类的私有方法和属性，因此可能会破坏类的封装性而导致安全问题。

Java 反射机制在一般的 Java 应用开发中很少使用，即便是 Java EE 阶段也很少使用。
# 反射机制API
实现反射机制的类都位于`java.lang.reflect`包中，`java.lang.Class`类是反射机制 API 中的核心类。
## java.lang.Class 类
`java.lang.Class`类是实现反射的关键所在，`Class`类的一个实例表示 Java 的一种数据类型，包括类、接口、枚举、注解（`Annotation`）、数组、基本数据类型和`void`。`Class`没有公有的构造方法，`Class`实例是由 JVM 在类加载时自动创建的。

在程序代码中获得`Class`实例可以通过如下代码实现：
```java
// 1. 通过类型class静态变量
Class clz1 = String.class;
String str = "Hello";
// 2. 通过对象的getClass()方法
Class clz2 = str.getClass();
```
每一种类型包括类和接口等，都有一个`class`静态变量可以获得`Class`实例。另外，每一个对象都有`getClass()`方法可以获得`Class`实例，该方法是由`Object`类提供的实例方法。

`Class`类提供了很多方法可以获得运行时对象的相关信息。
```java
public class ReflectionTest01 {
  public static void main(String[] args) {
    // 获得Class实例
    // 1.通过类型class静态变量
    Class clz1 = String.class;
    String str = "Hello";
    // 2.通过对象的getClass()方法
    Class clz2 = str.getClass();
    // 获得int类型Class实例
    Class clz3 = int.class;
    // 获得Integer类型Class实例
    Class clz4 = Integer.class;
    System.out.println("clz2类名称：" + clz2.getName());
    System.out.println("clz2是否为接口：" + clz2.isInterface());
    System.out.println("clz2是否为数组对象：" + clz2.isArray());
    System.out.println("clz2父类名称：" + clz2.getSuperclass().getName());
    System.out.println("clz2是否为基本类型：" + clz2.isPrimitive());
    System.out.println("clz3是否为基本类型：" + clz3.isPrimitive());
    System.out.println("clz4是否为基本类型：" + clz4.isPrimitive());
  }
}
```
运行结果如下：
```
clz2类名称：java.lang.String
clz2是否为接口：false
clz2是否为数组对象：false
clz2父类名称：java.lang.Object
clz2是否为基本类型：false
clz3是否为基本类型：true
clz4是否为基本类型：false
```
## java.lang.reflect 包
`java.lang.reflect`包提供了反射中用到类，主要的类说明如下：
* `Constructor`类：提供类的构造方法信息。
* `Field`类：提供类或接口中成员变量信息。
* `Method`类：提供类或接口成员方法信息。
* `Array`类：提供了动态创建和访问 Java 数组的方法。
* `Modifier`类：提供类和成员访问修饰符信息。

```java
public class ReflectionTest02 {
  public static void main(String[] args) {
    try {
      // 动态加载xx类的运行时对象
      Class c = Class.forName("java.lang.String");
      // 获取成员方法集合
      Method[] methods = c.getDeclaredMethods();
      // 遍历成员方法集合
      for (Method method : methods) {
        // 打印权限修饰符，如public、protected、private
        System.out.print(Modifier.toString(method.getModifiers()));
        // 打印返回值类型名称
        System.out.print(" " + method.getReturnType().getName() + " ");
        // 打印方法名称
        System.out.println(method.getName() + "();");
      }
    } catch (ClassNotFoundException e) {
      System.out.println("找不到指定类");
    }
  }
}
```
上述代码第 5 行是通过`Class`的静态方法`forName(String)`创建某个类的运行时对象，其中的参数是类全名字符串，如果在类路径中找不到这个类则抛出`ClassNotFoundException`异常。

代码第 7 行是通过`Class`的实例方法`getDeclaredMethods()`返回某个类的成员方法对象数组。代码第 9 行是遍历成员方法集合，其中的元素是`Method`类型。

代码第 11 行的`method.getModifiers()`方法返回访问权限修饰符常量代码，是`int`类型，例如 1 代表`public`，这些数字代表的含义可以通过`Modifier.toString(int)`方法转换为字符串。代码第 13 行通过`Method`的`getReturnType()`方法获得方法返回值类型，然后再调用`getName()`方法返回该类型的名称。代码第 15 行`method.getName()`返回方法名称。
# 通过反射访问构造方法
为了能够动态获取对象构造方法的信息，首先需要通过下列方法之一创建一个`Constructor`类型的对象或者数组。
```java
getConstructors()
getConstructor(Class<?>…parameterTypes)
getDeclaredConstructors()
getDeclaredConstructor(Class<?>...parameterTypes)
```
如果是访问指定的构造方法，需要根据该构造方法的入口参数的类型来访问。例如，访问一个入口参数类型依次为`int`和`String`类型的构造方法，下面的两种方式均可以实现。
```java
objectClass.getDeclaredConstructor(int.class,String.class);
objectClass.getDeclaredConstructor(new Class[]{int.class,String.class});
```
创建的每个`Constructor`对象表示一个构造方法，然后利用`Constructor`对象的方法操作构造方法。`Constructor`类的常用方法：

| 方法名称 | 说明 |
| :--: | :--: |
| isVarArgs()                    | 查看该构造方法是否允许带可变数量的参数，如果允许，返回 true，否则返回false |
| getParameterTypes()            | 按照声明顺序以 Class 数组的形式获取该构造方法各个参数的类型 |
| getExceptionTypes()            | 以 Class 数组的形式获取该构造方法可能抛出的异常类型 |
| newInstance(Object … initargs) | 通过该构造方法利用指定参数创建一个该类型的对象，如果未设置参数则表示采用默认无参的构造方法 |
| setAccessiable(boolean flag)   | 如果该构造方法的权限为 private，默认为不允许通过反射利用 netlnstance() 方法创建对象。如果先执行该方法，并将入口参数设置为 true，则允许创建对象 |
| getModifiers()                 | 获得可以解析出该构造方法所采用修饰符的整数 |

通过`java.lang.reflect.Modifier`类可以解析出`getMocMers()`方法的返回值所表示的修饰符信息。在该类中提供了一系列用来解析的静态方法，既可以查看是否被指定的修饰符修饰，还可以字符串的形式获得所有修饰符。`Modifier`类的常用静态方法：

| 静态方法名称         | 说明 |
| :--: | :--: |
| isStatic(int mod)    | 如果使用 static 修饰符修饰则返回 true，否则返回 false |
| isPublic(int mod)    | 如果使用 public 修饰符修饰则返回 true，否则返回 false |
| isProtected(int mod) | 如果使用 protected 修饰符修饰则返回 true，否则返回 false |
| isPrivate(int mod)   | 如果使用 private 修饰符修饰则返回 true，否则返回 false |
| isFinal(int mod)     | 如果使用 final 修饰符修饰则返回 true，否则返回 false |
| toString(int mod)    | 以字符串形式返回所有修饰符 |

例如，下列代码判断对象`con`所代表的构造方法是否被`public`修饰，以及以字符串形式获取该构造方法的所有修饰符。
```java
int modifiers = con.getModifiers();    // 获取构造方法的修饰符整数
boolean isPublic = Modifier.isPublic(modifiers);    // 判断修饰符整数是否为public 
string allModifiers = Modifier.toString(modifiers);
```

```java
public class Book {
  String name; // 图书名称
  int id, price; // 图书编号和价格
  // 空的构造方法
  private Book() {
  }
  // 带两个参数的构造方法
  protected Book(String _name, int _id) {
    this.name = _name;
    this.id = _id;
  }
  // 带可变参数的构造方法
  public Book(String... strings) throws NumberFormatException {
    if (0 < strings.length)
      id = Integer.valueOf(strings[0]);
    if (1 < strings.length)
      price = Integer.valueOf(strings[1]);
  }
  // 输出图书信息
  public void print() {
    System.out.println("name=" + name);
    System.out.println("id=" + id);
    System.out.println("price=" + price);
  }
}
```
```java
public class Test01 {
  public static void main(String[] args) {
    // 获取动态类Book
    Class book = Book.class;
    // 获取Book类的所有构造方法
    Constructor[] declaredContructors = book.getDeclaredConstructors();
    // 遍历所有构造方法
    for (int i = 0; i < declaredContructors.length; i++) {
      Constructor con = declaredContructors[i];
      // 判断构造方法的参数是否可变
      System.out.println("查看是否允许带可变数量的参数：" + con.isVarArgs());
      System.out.println("该构造方法的入口参数类型依次为：");
      // 获取所有参数类型
      Class[] parameterTypes = con.getParameterTypes();
      for (int j = 0; j < parameterTypes.length; j++) {
        System.out.println(" " + parameterTypes[j]);
      }
      System.out.println("该构造方法可能拋出的异常类型为：");
      // 获取所有可能拋出的异常类型
      Class[] exceptionTypes = con.getExceptionTypes();
      for (int j = 0; j < exceptionTypes.length; j++) {
        System.out.println(" " + parameterTypes[j]);
      }
      // 创建一个未实例化的Book类实例
      Book book1 = null;
      while (book1 == null) {
        try { // 如果该成员变量的访问权限为private，则拋出异常
          if (i == 1) {
            // 通过执行带两个参数的构造方法实例化book1
            book1 = (Book) con.newInstance("Java 教程", 10);
          } else if (i == 2) {
            // 通过执行默认构造方法实例化book1
            book1 = (Book) con.newInstance();
          } else {
            // 通过执行可变数量参数的构造方法实例化book1
            Object[] parameters = new Object[] { new String[] { "100", "200" } };
              book1 = (Book) con.newInstance(parameters);
          }
        } catch (Exception e) {
          System.out.println("在创建对象时拋出异常，下面执行 setAccessible() 方法");
          con.setAccessible(true); // 设置允许访问 private 成员
        }
      }
      book1.print();
      System.out.println("=============================\n");
    }
  }
}
```
运行结果：
```
查看是否允许带可变数量的参数：false
该构造方法的入口参数类型依次为：
该构造方法可能抛出的异常类型为：
在创建对象时抛出异常，下面执行setAccessible()方法
name = null
id = 0
price = 0
=============================
```
当通过反射访问两个参数的构造方法`Book(String_name,int_id)`时，将看到如下所示的输出。
```
查看是否允许带可变数量的参数：false
该构造方法的入口参数类型依次为：
class java.lang.String
int
该构造方法可能抛出的异常类型为：
在创建对象时抛出异常，下面执行setAccessible()方法
name = null
id = 0
price = 0
=============================
```
当通过反射访问可变参数数量的构造方法`Book(String...strings)`时，将看到如下所示的输出。
```
查看是否允许带可变数量的参数：true
该构造方法的入口参数类型依次为：
class java.lang.String;
该构造方法可能抛出的异常类型为：
class java.lang.String;
在创建对象时抛出异常，下面执行setAccessible()方法
name = null
id = 0
price = 0
=============================
```
# 通过反射访问方法
要动态获取一个对象方法的信息，首先需要通过下列方法之一创建一个`Method`类型的对象或者数组。
```java
getMethods()
getMethods(String name,Class<?> …parameterTypes)
getDeclaredMethods()
getDeclaredMethods(String name,Class<?>...parameterTypes)
```
如果是访问指定的构造方法，需要根据该方法的入口参数的类型来访问。例如，访问一个名称为`max`，入口参数类型依次为`int`和`String`类型的方法。

下面的两种方式均可以实现：
```java
objectClass.getDeclaredConstructor("max",int.class,String.class);
objectClass.getDeclaredConstructor("max",new Class[]{int.class,String.class});
```
`Method`类的常用方法。

| 静态方法名称                      | 说明 |
| :--: | :--: |
| getName()                         | 获取该方法的名称 |
| getParameterType()                | 按照声明顺序以 Class 数组的形式返回该方法各个参数的类型 |
| getReturnType()                   | 以 Class 对象的形式获得该方法的返回值类型 |
| getExceptionTypes()               | 以 Class 数组的形式获得该方法可能抛出的异常类型 |
| invoke(Object obj,Object...args)	| 利用 args 参数执行指定对象 obj 中的该方法，返回值为 Object 类型 |
| isVarArgs()                       | 查看该方法是否允许带有可变数量的参数，如果允许返回 true，否则返回 false |
| getModifiers()                    | 获得可以解析出该方法所采用修饰符的整数 |

```java
public class Book1 {
  // static 作用域方法
  static void staticMethod() {
    System.out.println("执行staticMethod()方法");
  }
  // public 作用域方法
  public int publicMethod(int i) {
    System.out.println("执行publicMethod()方法");
    return 100 + i;
  }
  // protected 作用域方法
  protected int protectedMethod(String s, int i) throws NumberFormatException {
    System.out.println("执行protectedMethod()方法");
    return Integer.valueOf(s) + i;
  }
  // private 作用域方法
  private String privateMethod(String... strings) {
    System.out.println("执行privateMethod()方法");
    StringBuffer sb = new StringBuffer();
    for (int i = 0; i < sb.length(); i++) {
      sb.append(strings[i]);
    }
    return sb.toString();
  }
}
```
```java
public class Test02 {
  public static void main(String[] args) {
    // 获取动态类Book1
    Book1 book = new Book1();
    Class class1 = book.getClass();
    // 获取Book1类的所有方法
    Method[] declaredMethods = class1.getDeclaredMethods();
    for (int i = 0; i < declaredMethods.length; i++) {
      Method method = declaredMethods[i];
      System.out.println("方法名称为：" + method.getName());
      System.out.println("方法是否带有可变数量的参数：" + method.isVarArgs());
      System.out.println("方法的参数类型依次为：");
      // 获取所有参数类型
      Class[] methodType = method.getParameterTypes();
      for (int j = 0; j < methodType.length; j++) {
        System.out.println(" " + methodType[j]);
      }
      // 获取返回值类型
      System.out.println("方法的返回值类型为：" + method.getReturnType());
      System.out.println("方法可能抛出的异常类型有：");
      // 获取所有可能抛出的异常
      Class[] methodExceptions = method.getExceptionTypes();
      for (int j = 0; j < methodExceptions.length; j++) {
        System.out.println(" " + methodExceptions[j]);
      }
      boolean isTurn = true;
      while (isTurn) {
        try { // 如果该成员变量的访问权限为private，则抛出异常
          isTurn = false;
          if (method.getName().equals("staticMethod")) { // 调用没有参数的方法
            method.invoke(book);
          } else if (method.getName().equals("publicMethod")) { // 调用一个参数的方法
            System.out.println("publicMethod(10)的返回值为：" + method.invoke(book, 10));
          } else if (method.getName().equals("protectedMethod")) { // 调用两个参数的方法
            System.out.println("protectedMethod(\"10\",15)的返回值为：" + method.invoke(book, "10", 15));
          } else if (method.getName().equals("privateMethod")) { // 调用可变数量参数的方法
            Object[] parameters = new Object[] { new String[] { "J", "A", "V", "A" } };
            System.out.println("privateMethod()的返回值为：" + method.invoke(book, parameters));
          }
        } catch (Exception e) {
          System.out.println("在设置成员变量值时抛出异常，下面执行setAccessible()方法");
          method.setAccessible(true); // 设置为允许访问private方法
          isTurn = true;
        }
      }
      System.out.println("=============================\n");
    }
  }
}
```
运行测试类 test02，程序将会依次动态访问`Book1`类中的所有方法。访问`staticMethod()`方法的运行效果如下所示：
```
方法名称为：staticMethod
方法是否带有可变数量的参数：false
方法的参数类型依次为：
方法的返回值类型为：void
方法可能抛出的异常类型有：
执行staticMethod()方法
=============================
```
访问`publicMethod()`方法的运行效果如下所示：
```
方法名称为：publicMethod
方法是否带有可变数量的参数：false
方法的参数类型依次为：
int
方法的返回值类型为：int
方法可能抛出的异常类型有：
执行publicMethod()方法
publicMethod(10)的返回值为：110
=============================
```
访问`protectedMethod()`方法的运行效果如下所示：
```
方法名称为：protectedMethod
方法是否带有可变数量的参数：false
方法的参数类型依次为：
class java.lang.String
int
方法的返回值类型为：int
方法可能抛出的异常类型有：
class java.lang.NumberFormatException
执行protectedMethod()方法
protectedMethod("10",15)的返回值为：25
=============================
```
访问`privateMethod()`方法的运行效果如下所示：
```
方法名称为：privateMethod
方法是否带有可变数量的参数：true
方法的参数类型依次为：
class java.lang.String;
方法的返回值类型为：class java.lang.String
方法可能抛出的异常类型有：
在设置成员变量值时抛出异常，下面执行setAccessible()方法
执行privateMethod()方法
privateMethod()的返回值为：
=============================
```
# 通过反射访问成员变量
通过下列任意一个方法访问成员变量时将返回`Field`类型的对象或数组。
```java
getFields()
getField(String name)
getDeclaredFields()
getDeclaredField(String name)
```
上述方法返回的`Field`对象代表一个成员变量。例如，要访问一个名称为`price`的成员变量，示例代码如下：
```
object.getDeciaredField("price");
```
`Field`类的常用方法

| 方法名称                          | 说明 |
| :--: | :--: |
| getName()                         | 获得该成员变量的名称 |
| getType()                         | 获取表示该成员变量的 Class 对象 |
| get(Object obj)                   | 获得指定对象 obj 中成员变量的值，返回值为 Object 类型 |
| set(Object obj, Object value)     | 将指定对象 obj 中成员变量的值设置为 value |
| getlnt(0bject obj)                | 获得指定对象 obj 中成员类型为 int 的成员变量的值 |
| setlnt(0bject obj, int i)         | 将指定对象 obj 中成员变量的值设置为 i |
| setFloat(Object obj, float f)     | 将指定对象 obj 中成员变量的值设置为 f |
| getBoolean(Object obj)            | 获得指定对象 obj 中成员类型为 boolean 的成员变量的值 |
| setBoolean(Object obj, boolean b)	| 将指定对象 obj 中成员变量的值设置为 b |
| getFloat(Object obj)              | 获得指定对象 obj 中成员类型为 float 的成员变量的值 |
| setAccessible(boolean flag)       | 此方法可以设置是否忽略权限直接访问 private 等私有权限的成员变量 |
| getModifiers()                    | 获得可以解析出该方法所采用修饰符的整数 |

```java
public class Book2 {
  String name;
  public int id;
  private float price;
  protected boolean isLoan;
}
```
```java
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
public class Test03 {
  public static void main(String[] args) {
    Book2 book = new Book2();
    // 获取动态类Book2
    Class class1 = book.getClass();
    // 获取Book2类的所有成员
    Field[] declaredFields = class1.getDeclaredFields();
    // 遍历所有的成员
    for(int i = 0;i < declaredFields.length;i++) {    
      // 获取类中的成员变量
      Field field = declaredFields[i];
      System.out.println("成员名称为：" + field.getName());
      Class fieldType = field.getType();
      System.out.println("成员类型为：" + fieldType);
      boolean isTurn = true;
      while(isTurn) {
        try {    
          // 如果该成员变量的访问权限为private，则抛出异常
          isTurn = false;
          System.out.println("修改前成员的值为：" + field.get(book));
          // 判断成员类型是否为int
          if(fieldType.equals(int.class)) {
            System.out.println("利用setInt()方法修改成员的值");
            field.setInt(book, 100);
          } else if(fieldType.equals(float.class)) {    
            // 判断成员变量类型是否为float
            System.out.println("利用setFloat()方法修改成员的值");
            field.setFloat(book, 29.815f);
          } else if(fieldType.equals(boolean.class)) {    
            // 判断成员变量是否为boolean
            System.out.println("利用setBoolean()方法修改成员的值");
            field.setBoolean(book, true);
          } else {
            System.out.println("利用set()方法修改成员的值");
            field.set(book, "Java编程");
          }
          System.out.println("修改后成员的值为：" + field.get(book));
        } catch (Exception e) {
          System.out.println("在设置成员变量值时抛出异常，下面执行setAccessible()方法");
          field.setAccessible(true);
          isTurn = true;
        }
      }
      System.out.println("=============================\n");
    }
  }
}
```
运行测试类 Test03，程序将会依次动态访问`Book2`类中的所有成员。访问`name`成员的运行效果如下所示：
```
成员名称为：name
成员类型为：class java.lang.String
修改前成员的值为：null
利用set()方法修改成员的值
修改后成员的值为：Java编程
=============================
```
访问`id`成员的运行效果如下所示：
```
成员名称为：id
成员类型为：int
修改前成员的值为：0
利用setInt()方法修改成员的值
修改后成员的值为：100
=============================
```
访问`price`成员的运行效果如下所示：
```
成员名称为：price
成员类型为：float
在设置成员变量值时抛出异常，下面执行setAccessible()方法
修改前成员的值为：0.0
利用setFloat()方法修改成员的值
修改后成员的值为：29.815
=============================
```
访问`isLoan`成员的运行效果如下所示：
```
成员名称为：isLoan
成员类型为：boolean
修改前成员的值为：false
利用setBoolean()方法修改成员的值
修改后成员的值为：true
=============================
```
# 在远程方法调用中运用反射机制
反射机制在网络编程中实现如何在客户端通过远程方法调用服务器端的方法。

假定在服务器端有一个`HelloService`接口：
```java
import java.util.Date;
public interface HelloService {
  public String echo(String msg);
  public Date getTime();
}
```
在服务器上创建一个`HelloServiceImpl`类并实现`HelloService`接口。
```java
import java.util.Date;
public class HelloServiceImpl implements HelloService {
  @Override
  public String echo(String msg) {
    return "echo:" + msg;
  }
  @Override
  public Date getTime() {
    return new Date();
  }
}
```
客户端要调用服务器端`Hello-ServiceImpl`类中的`getTime()`和`echo()`方法，具体方法是：客户端需要把调用的方法名、方法参数类型、方法参数值，以及方法所属的类名或接口名发送给服务器端。服务器端再调用相关对象的方法，然后把方法的返回值发送给客户端。

为了便于按照面向对象的方式来处理客户端与服务器端的通信，可以把它们发送的信息用`Call`类来表示。一个`Call`对象表示客户端发起的一个远程调用，它包括调用的类名或接口名、方法名、方法参数类型、方法参数值和方法执行结果。

`Call`类的实现代码如下：
```java
import java.io.Serializable;
public class Call implements Serializable {
  private static final long serialVersionUID = 6659953547331194808L;
  private String className; // 表示类名或接口名
  private String methodName; // 表示方法名
  private Class[] paramTypes; // 表示方法参数类型
  private Object[] params; // 表示方法参数值
  // 表示方法的执行结果
  // 如果方法正常执行，则result为方法返回值，如果方法抛出异常，那么result为该异常。
  private Object result;
  public Call() {
  }
  public Call(String className, String methodName, Class[] paramTypes, Object[] params) {
    this.className = className;
    this.methodName = methodName;
    this.paramTypes = paramTypes;
    this.params = params;
  }
  public String getClassName() {
    return className;
  }
  public void setClassName(String className) {
    this.className = className;
  }
  public String getMethodName() {
    return methodName;
  }
  public void setMethodName(String methodName) {
    this.methodName = methodName;
  }
  public Class[] getParamTypes() {
    return paramTypes;
  }
  public void setParamTypes(Class[] paramTypes) {
    this.paramTypes = paramTypes;
  }
  public Object[] getParams() {
    return params;
  }
  public void setParams(Object[] params) {
    this.params = params;
  }
  public Object getResult() {
    return result;
  }
  public void setResult(Object result) {
    this.result = result;
  }
  public String toString() {
    return "className=" + className + "methodName=" + methodName;
  }
}
```
假设客户端为`SimpleClient`，服务器端为`SimpleServer`。`SimpleClient`调用`SimpleServer`的`HelloServiceImpl`对象中`echo()`方法的流程如下：
* `SimpleClient`创建一个`Call`对象，它包含调用`HelloService`接口的`echo()`方法的信息。
* `SimpleClient`通过对象输出流把`Call`对象发送给`SimpleServer`。
* `SimpleServer`通过对象输入流读取`Call`对象，运用反射机制调用`HelloServiceImpl`对象的`echo()`方法，把`echo()`方法的执行结果保存到`Call`对象中。
* `SimpleServer`通过对象输出流把包含方法执行结果的`Call`对象发送给`SimpleClient`。
* `SimpleClient`通过对象输入流读取`Call`对象，从中获得方法执行结果。

客户端程序`SimpleClient`类的实现代码。
```java
import java.io.*;
import java.net.*;
import java.util.*;
import java.lang.reflect.*;
import java.io.*;
import java.net.*;
import java.util.*;
public class SimpleClient {
  public void invoke() throws Exception {
    Socket socket = new Socket("localhost", 8000);
    OutputStream out = socket.getOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(out);
    InputStream in = socket.getInputStream();
    ObjectInputStream ois = new ObjectInputStream(in);
    // 创建一个远程调用对象
    Call call = new Call("ch12.HelloService", "echo", new Class[] { String.class }, new Object[] { "Java" });
    oos.writeObject(call); // 向服务器发送Call对象
    call = (Call) ois.readObject(); // 接收包含了方法执行结果的Call对象
    System.out.println(call.getResult());
    ois.close();
    oos.close();
    socket.close();
  }
  public static void main(String args[]) throws Exception {
    new SimpleClient().invoke();
  }
}
```
客户端`SimpleClient`类的主要作用是建立与服务器的连接，然后将带有调用信息的`Call`对象发送到服务器端。

服务器端`SimpleServer`类在收到调用请求之后会使用反射机制动态调用指定对象的指定方法，再将执行结果返回给客户端。

`SimpleServer`类的实现代码如下：
```java
import java.io.*;
import java.net.*;
import java.util.*;
import java.lang.reflect.*;
public class SimpleServer {
  private Map remoteObjects = new HashMap(); // 存放远程对象的缓存
  /** 把一个远程对象放到缓存中 */
  public void register(String className, Object remoteObject) {
    remoteObjects.put(className, remoteObject);
  }
  public void service() throws Exception {
    ServerSocket serverSocket = new ServerSocket(8000);
    System.out.println("服务器启动.");
    while (true) {
      Socket socket = serverSocket.accept();
      InputStream in = socket.getInputStream();
      ObjectInputStream ois = new ObjectInputStream(in);
      OutputStream out = socket.getOutputStream();
      ObjectOutputStream oos = new ObjectOutputStream(out);
      Call call = (Call) ois.readObject(); // 接收客户发送的Call对象
      System.out.println(call);
      call = invoke(call); // 调用相关对象的方法
      oos.writeObject(call); // 向客户发送包含了执行结果的Call对象
      ois.close();
      oos.close();
      socket.close();
    }
  }
  public Call invoke(Call call) {
    Object result = null;
    try {
      String className = call.getClassName();
      String methodName = call.getMethodName();
      Object[] params = call.getParams();
      Class classType = Class.forName(className);
      Class[] paramTypes = call.getParamTypes();
      Method method = classType.getMethod(methodName, paramTypes);
      Object remoteObject = remoteObjects.get(className); // 从缓存中取出相关的远程对象
      if (remoteObject == null) {
        throw new Exception(className + "的远程对象不存在");
      } else {
        result = method.invoke(remoteObject, params);
      }
    } catch (Exception e) {
      result = e;
    }
    call.setResult(result); // 设置方法执行结果
    return call;
  }
  public static void main(String args[]) throws Exception {
    SimpleServer server = new SimpleServer();
    // 把事先创建的HelloServiceImpl对象加入到服务器的缓存中
    server.register("ch13.HelloService", new HelloServiceImpl());
    server.service();
  }
}
```
由于这是一个网络程序，首先需要运行服务器端`SimpleServer`，然后再运行客户端`SimpleClient`。运行结果是在客户端看到输出`echoJava`，这个结果是服务器端执行`HelloServicelmpl`对象的`echo()`方法的返回值。
