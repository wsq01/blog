---
title: Java 异常处理
date: 2020-09-25 19:08:23
tags: [java]
categories: java
---

Java 中的异常是一个在程序执行期间发生的事件，它中断正在执行程序的正常指令流。为了能够及时有效地处理程序中的运行错误，必须使用异常类，这可以让程序具有极好的容错性且更加健壮。 

一个异常的产生，主要有如下三种原因：
* Java 内部错误发生异常，Java 虚拟机产生的异常。
* 编写的程序代码中的错误所产生的异常，例如空指针异常、数组越界异常等。
* 通过`throw`语句手动生成的异常，一般用来告知该方法的调用者一些必要信息。

Java 通过面向对象的方法来处理异常。在一个方法的运行过程中，如果发生了异常，则这个方法会产生代表该异常的一个对象，并把它交给运行时的系统，运行时系统寻找相应的代码来处理这一异常。
## 异常类型
为了能够及时有效地处理程序中的运行错误，Java 专门引入了异常类。在 Java 中所有异常类型都是内置类`java.lang.Throwable`类的子类，即`Throwable`位于异常类层次结构的顶层。`Throwable`类下有两个异常分支`Exception`和`Error`。

`Throwable`类是所有异常和错误的超类，下面有`Error`和`Exception`两个子类分别表示错误和异常。其中异常类`Exception`又分为运行时异常和非运行时异常，这两种异常有很大的区别，也称为不检查异常（`Unchecked Exception`）和检查异常（`Checked Exception`）。
* `Exception`类用于用户程序可能出现的异常情况，它也是用来创建自定义异常类型类的类。
* `Error`定义了在通常环境下不希望被程序捕获的异常。一般指的是 JVM 错误，如堆栈溢出。

运行时异常都是`RuntimeException`类及其子类异常，如`NullPointerException、IndexOutOfBoundsException`等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般由程序逻辑错误引起，程序应该从逻辑角度尽可能避免这类异常的发生。

非运行时异常是指`RuntimeException`以外的异常，类型上都属于`Exception`类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如`IOException、ClassNotFoundException`等以及用户自定义的`Exception`异常（一般情况下不自定义检查异常）。

常见运行时异常：

| 异常类型 | 说明 |
| :--: | :--: |
| ArithmeticException        | 算术错误异常，如以零做除数 |
| ArraylndexOutOfBoundException | 数组索引越界 |
| ArrayStoreException        | 向类型不兼容的数组元素赋值 |
| ClassCastException         | 类型转换异常 |
| IllegalArgumentException   | 使用非法实参调用方法 |
| lIIegalStateException      | 环境或应用程序处于不正确的状态 |
| lIIegalThreadStateException | 被请求的操作与当前线程状态不兼容 |
| IndexOutOfBoundsException  | 某种类型的索引越界 |
| NullPointerException       | 尝试访问 null 对象成员，空指针异常 |
| NegativeArraySizeException | 再负数范围内创建的数组 |
| NumberFormatException      | 数字转化格式异常，比如字符串到 float 型数字的转换无效 |
| TypeNotPresentException    | 类型未找到 |

常见非运行时异常：

| 异常类型 | 说明 |
| :--: | :--: |
| ClassNotFoundException        | 没有找到类 |
| IllegalAccessException        | 访问类被拒绝 |
| InstantiationException        | 试图创建抽象类或接口的对象 |
| InterruptedException          | 线程被另一个线程中断 |
| NoSuchFieldException          | 请求的域不存在 |
| NoSuchMethodException         | 请求的方法不存在 |
| ReflectiveOperationException  | 与反射有关的异常的超类 |

# Error和Exception的异同
`Error`（错误）和`Exception`（异常）都是`java.lang.Throwable`类的子类，在 Java 代码中只有继承了`Throwable`类的实例才能被`throw`或者`catch`。

`Exception`和`Error`体现了 Java 平台设计者对不同异常情况的分类，`Exception`是程序正常运行过程中可以预料到的意外情况，并且应该被开发者捕获，进行相应的处理。`Error`是指正常情况下不大可能出现的情况，绝大部分的`Error`都会导致程序处于非正常、不可恢复状态。所以不需要被捕获。

`Error`错误是任何处理技术都无法恢复的情况，肯定会导致程序非正常终止。并且`Error`错误属于未检查类型，大多数发生在运行时。`Exception`又分为可检查（`checked`）异常和不检查（`unchecked`）异常，可检查异常在源码里必须显示的进行捕获处理，这里是编译期检查的一部分。不检查异常就是所谓的运行时异常，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并不会在编译器强制要求。

如下是常见的`Error`和`Exception`：
1. 运行时异常（`RuntimeException`）：
 * `NullPropagation`：空指针异常；
 * `ClassCastException`：类型强制转换异常
 * `IllegalArgumentException`：传递非法参数异常
 * `IndexOutOfBoundsException`：下标越界异常
 * `NumberFormatException`：数字格式异常
2. 非运行时异常：
 * `ClassNotFoundException`：找不到指定`class`的异常
 * `IOException`：IO 操作异常
3. 错误（`Error`）：
 * `NoClassDefFoundError`：找不到`class`定义异常
 * `StackOverflowError`：深递归导致栈被耗尽而抛出的异常
 * `OutOfMemoryError`：内存溢出异常

下面代码会导致 Java 堆栈溢出错误。
```java
// 通过无限递归演示堆栈溢出错误
class StackOverflow {
  public static void test(int i) {
    if (i == 0) {
      return;
    } else {
      test(i++);
    }
  }
}
public class ErrorEg {
  public static void main(String[] args) {
    // 执行StackOverflow方法
    StackOverflow.test(5);
  }
}
```
运行输出为：
```
Exception in thread "main" java.lang.StackOverflowError
    at ch11.StackOverflow.test(ErrorEg.java:9)
    at ch11.StackOverflow.test(ErrorEg.java:9)
    at ch11.StackOverflow.test(ErrorEg.java:9)
    at ch11.StackOverflow.test(ErrorEg.java:9)
```
# 异常处理机制
Java 的异常处理通过 5 个关键字来实现：`try、catch、throw、throws`和`finally`。`try catch`语句用于捕获并处理异常，`finally`语句用于在任何情况下（除特殊情况外）都必须执行的代码，`throw`语句用于拋出异常，`throws`语句用于声明可能会出现的异常。

Java 的异常处理机制提供了一种结构性和控制性的方式来处理程序执行期间发生的事件。异常处理的机制如下：
* 在方法中用`try catch`语句捕获并处理异常，`catch`语句可以有多个，用来匹配多个异常。
* 对于处理不了的异常或者要转型的异常，在方法的声明处通过`throws`语句拋出异常，即由上层的调用方法来处理。

```java
try {
  //逻辑程序块
} catch (ExceptionType1 e) {
  //处理代码块1
} catch (ExceptionType2 e) {
  //处理代码块2
  throw(e);    // 再抛出这个"异常"
} finally {
  //释放资源代码块
}
```
# try catch语句
在 Java 中通常采用`try catch`语句来捕获异常并处理。把可能引发异常的语句封装在`try`语句块中，用以捕获可能发生的异常。`catch`后的`()`里放匹配的异常类，指明`catch`语句可以处理的异常类型，发生异常时产生异常类的实例化对象。

如果`try`语句块中发生异常，那么一个相应的异常对象就会被拋出，然后`catch`语句就会依据所拋出异常对象的类型进行捕获，并处理。处理之后，程序会跳过`try`语句块中剩余的语句，转到`catch`语句块后面的第一条语句开始执行。

如果`try`语句块中没有异常发生，那么`try`块正常结束，后面的`catch`语句块被跳过，程序将从`catch`语句块后的第一条语句开始执行。

在上面语法的处理代码块 1 中，可以使用以下 3 个方法输出相应的异常信息。
* `printStackTrace()`方法：指出异常的类型、性质、栈层次及出现在程序中的位置。
* `getMessage()`方法：输出错误的性质。
* `toString()`方法：给出异常的类型与性质。

编写一个录入学生姓名、年龄和性别的程序，要求能捕捉年龄不为数字时的异常。
```java
import java.util.Scanner;
public class Test02 {
  public static void main(String[] args) {
    Scanner scanner = new Scanner(System.in);
    System.out.println("---------学生信息录入---------------");
    String name = ""; // 获取学生姓名
    int age = 0; // 获取学生年龄
    String sex = ""; // 获取学生性别
    try {
      System.out.println("请输入学生姓名：");
      name = scanner.next();
      System.out.println("请输入学生年龄：");
      age = scanner.nextInt();
      System.out.println("请输入学生性别：");
      sex = scanner.next();
    } catch (Exception e) {
      e.printStackTrace();
      System.out.println("输入有误！");
    }
    System.out.println("姓名：" + name);
    System.out.println("年龄：" + age);
  }
}
```
上述代码在`main()`方法中使用`try catch`语句来捕获异常，将可能发生异常的`age = scanner.nextlnt();`代码放在了`try`块中，在`catch`语句中指定捕获的异常类型为`Exception`，并调用异常对象的`printStackTrace()`方法输出异常信息。
## 多重catch语句
如果`try`代码块中有很多语句会发生异常，而且发生的异常种类又很多。那么可以在`try`后面跟有多个`catch`代码块。

在多个`catch`代码块的情况下，当一个`catch`代码块捕获到一个异常时，其它的`catch`代码块就不再进行匹配。

注意：当捕获的多个异常类之间存在父子关系时，捕获异常时一般先捕获子类，再捕获父类。所以子类异常必须在父类异常的前面，否则子类捕获不到。
```java
public class Test03 {
  public static void main(String[] args) {
    Date date = readDate();
    System.out.println("读取的日期 = " + date);
  }
  public static Date readDate() {
    FileInputStream readfile = null;
    InputStreamReader ir = null;
    BufferedReader in = null;
    try {
      readfile = new FileInputStream("readme.txt");
      ir = new InputStreamReader(readfile);
      in = new BufferedReader(ir);
      // 读取文件中的一行数据
      String str = in.readLine();
      if (str == null) {
          return null;
      }
      DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
      Date date = df.parse(str);
      return date;
    } catch (FileNotFoundException e) {
      System.out.println("处理FileNotFoundException...");
      e.printStackTrace();
    } catch (IOException e) {
      System.out.println("处理IOException...");
      e.printStackTrace();
    } catch (ParseException e) {
      System.out.println("处理ParseException...");
      e.printStackTrace();
    }
    return null;
  }
}
```
上述代码通过 Java I/O（输入输出）流技术从文件`readme.txt`中读取字符串，然后解析成为日期。

在`try`代码块中调用`FileInputStream`构造方法可能会发生`FileNotFoundException`异常。调用`BufferedReader`输入流的`readLine()`方法可能会发生`IOException`异常。`FileNotFoundException`异常是`IOException`异常的子类，应该先捕获`FileNotFoundException`异常；后捕获 `IOException`异常。

如果将`FileNotFoundException`和`IOException`捕获顺序调换，那么捕获`FileNotFoundException`异常代码块将永远不会进入，`FileNotFoundException`异常处理永远不会执行。上述代码第 29 行`ParseException`异常与`IOException`和`FileNotFoundException`异常没有父子关系，所以捕获`ParseException`异常位置可以随意放置。
# try catch finally语句
在实际开发中，根据`try catch`语句的执行过程，`try`语句块和`catch`语句块有可能不被完全执行，而有些处理代码则要求必须执行。例如，程序在`try`块里打开了一些物理资源（如数据库连接、网络连接和磁盘文件等），这些物理资源都必须显式回收。

Java的垃圾回收机制不会回收任何物理资源，垃圾回收机制只回收堆内存中对象所占用的内存。

所以为了确保一定能回收`try`块中打开的物理资源，异常处理机制提供了`finally`代码块，并且 Java 7 之后提供了自动资源管理技术。
```java
try {
  // 可能会发生异常的语句
} catch(ExceptionType e) {
  // 处理异常语句
} finally {
  // 清理代码块
}
```
对于以上格式，无论是否发生异常（除特殊情况外），`finally`语句块中的代码都会被执行。此外，`finally`语句也可以和`try`语句匹配使用：
```java
try {
  // 逻辑代码块
} finally {
  // 清理代码块
}
```
使用`try-catch-finally`语句时需注意以下几点：
* 异常处理语法结构中只有`try`块是必需的，也就是说，如果没有`try`块，则不能有后面的`catch`块和`finally`块；
* `catch`块和`finally`块都是可选的，但`catch`块和`finally`块至少出现其中之一，也可以同时出现；
* 可以有多个`catch`块，捕获父类异常的`catch`块必须位于捕获子类异常的后面；
* 不能只有`try`块，既没有`catch`块，也没有`finally`块；
* 多个`catch`块必须位于`try`块之后，`finally`块必须位于所有的`catch`块之后。

`try catch finally`语句块的执行情况可以细分为以下 3 种情况：
* 如果`try`代码块中没有拋出异常，则执行完`try`代码块之后直接执行`finally`代码块，然后执行`try catch finally`语句块之后的语句。
* 如果`try`代码块中拋出异常，并被`catch`子句捕捉，那么在拋出异常的地方终止`try`代码块的执行，转而执行相匹配的`catch`代码块，之后执行`finally`代码块。如果`finally`代码块中没有拋出异常，则继续执行`try catch finally`语句块之后的语句；如果`finally`代码块中拋出异常，则把该异常传递给该方法的调用者。
* 如果`try`代码块中拋出的异常没有被任何`catch`子句捕捉到，那么将直接执行`finally`代码块中的语句，并把该异常传递给该方法的调用者。

除非在`try`块、`catch`块中调用了退出虚拟机的方法`System.exit(int status)`，否则不管在`try`块或者`catch`块中执行怎样的代码，出现怎样的情况，异常处理的`finally`块总会执行。

通常情况下不在`finally`代码块中使用`return`或`throw`等导致方法终止的语句，否则将会导致`try`和`catch`代码块中的`return`和`throw`语句失效。
# 自动资源管理
当程序使用`finally`块关闭资源时，程序会显得异常臃肿，例如以下代码。
```java
public static void main(String[] args) {
  FileInputStream fis = null;
  try {
    fis = new FileInputStream("a.txt");
  } catch (FileNotFoundException e) {
    e.printStackTrace();
  } finally {
    // 关闭磁盘文件，回收资源
    if (fis != null) {
      try {
        fis.close();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }
}
```
Java 7 以前，上面程序中的`finally`代码块是不得不写的“臃肿代码”，为了解决这种问题，Java 7 增加了一个新特性，该特性提供了另外一种管理资源的方式，这种方式能自动关闭文件，被称为自动资源管理。该特性是在`try`语句上的扩展，主要释放不再需要的文件或其他资源。

自动资源管理替代了`finally`代码块，并优化了代码结构和提高程序可读性。语法如下：
```java
try (声明或初始化资源语句) {
    // 可能会生成异常语句
} catch(Throwable e1){
    // 处理异常e1
} catch(Throwable e2){
    // 处理异常e1
} catch(Throwable eN){
    // 处理异常eN
}
```
当`try`代码块结束时，自动释放资源。不再需要显式的调用`close()`方法，该形式也称为“带资源的`try`语句”。

注意：
* `try`语句中声明的资源被隐式声明为`final`，资源的作用局限于带资源的`try`语句。
* 可以在一条`try`语句中声明或初始化多个资源，每个资源以;隔开即可。
* 需要关闭的资源必须实现了`AutoCloseable`或`Closeable`接口。
* `Closeable`是`AutoCloseable`的子接口，`Closeable`接口里的`close()`方法声明抛出了`IOException`，因此它的实现类在实现`close()`方法时只能声明抛出`IOException`或其子类；`AutoCloseable`接口里的`close()`方法声明抛出了`Exception`，因此它的实现类在实现`close()`方法时可以声明抛出任何异常。

下面示范如何使用自动关闭资源的`try`语句。
```java
public class AutoCloseTest {
  public static void main(String[] args) throws IOException {
    try (
      // 声明、初始化两个可关闭的资源
      // try语句会自动关闭这两个资源
      BufferedReader br = new BufferedReader(new FileReader("AutoCloseTest.java"));
      PrintStream ps = new PrintStream(new FileOutputStream("a.txt"))) {
      // 使用两个资源
      System.out.println(br.readLine());
      ps.println("C语言中文网");
    }
  }
}
```
上面程序中粗体字代码分别声明、初始化了两个 IO 流，`BufferedReader`和`PrintStream`都实现了`Closeable`接口，并在`try`语句中进行了声明和初始化，所以`try`语句会自动关闭它们。

自动关闭资源的`try`语句相当于包含了隐式的`finally`块（这个`finally`块用于关闭资源），因此这个`try`语句可以既没有`catch`块，也没有`finally`块。
Java 7 几乎把所有的“资源类”（包括文件 IO 的各种类、JDBC 编程的`Connection`和`Statement`等接口）进行了改写，改写后的资源类都实现了`AutoCloseable`或`Closeable`接口。

如果程序需要，自动关闭资源的`try`语句后也可以带多个`catch`块和一个`finally`块。

Java 9 再次增强了这种`try`语句。Java 9 不要求在`try`后的圆括号内声明并创建资源，只需要自动关闭的资源有`final`修饰或者是有效的`final (effectively final)`，Java 9 允许将资源变量放在`try`后的圆括号内。上面程序在 Java 9 中可改写为如下形式。
```java
public class AutoCloseTest {
  public static void main(String[] args) throws IOException {
    // 有final修饰的资源
    final BufferedReader br = new BufferedReader(new FileReader("AutoCloseTest.java"));
    // 没有显式使用final修饰，但只要不对该变量重新赋值，该变量就是有效的
    final PrintStream ps = new PrintStream(new FileOutputStream("a. txt"));
    // 只要将两个资源放在try后的圆括号内即可
    try (br; ps) {
      // 使用两个资源
      System.out.println(br.readLine());
      ps.println("C语言中文网");
    }
  }
}
```
# 声明和抛出异常
Java 中的异常处理除了捕获异常和处理异常之外，还包括声明异常和拋出异常。实现声明和抛出异常的关键字非常相似，它们是`throws`和`throw`。可以通过`throws`关键字在方法上声明该方法要拋出的异常，然后在方法内部通过`throw`拋出异常对象。
## throws 声明异常
当一个方法产生一个它不处理的异常时，那么就需要在该方法的头部声明这个异常，以便将该异常传递到方法的外部进行处理。使用`throws`声明的方法表示此方法不处理异常。`throws`具体格式如下：
```
returnType method_name(paramList) throws Exception 1,Exception2,…{…}
```
其中，`returnType`表示返回值类型；`method_name`表示方法名；`paramList`表示参数列表；`Exception 1，Exception2，… `表示异常类。

如果有多个异常类，它们之间用逗号分隔。这些异常类可以是方法中调用了可能拋出异常的方法而产生的异常，也可以是方法体中生成并拋出的异常。

使用`throws`声明抛出异常的思路是，当前方法不知道如何处理这种类型的异常，该异常应该由向上一级的调用者处理；如果`main`方法也不知道如何处理这种类型的异常，也可以使用`throws`声明抛出异常，该异常将交给 JVM 处理。JVM 对异常的处理方法是，打印异常的跟踪栈信息，并中止程序运行，这就是前面程序在遇到异常后自动结束的原因。

创建一个`readFile()`方法，该方法用于读取文件内容，在读取的过程中可能会产生`IOException`异常，但是在该方法中不做任何的处理，而将可能发生的异常交给调用者处理。在`main()`方法中使用`try catch`捕获异常，并输出异常信息。代码如下：
```java
import java.io.FileInputStream;
import java.io.IOException;
public class Test04 {
  public void readFile() throws IOException {
    // 定义方法时声明异常
    FileInputStream file = new FileInputStream("read.txt"); // 创建 FileInputStream 实例对象
    int f;
    while ((f = file.read()) != -1) {
      System.out.println((char) f);
      f = file.read();
    }
    file.close();
  }
  public static void main(String[] args) {
    Throws t = new Test04();
    try {
      t.readFile(); // 调用 readFHe()方法
    } catch (IOException e) {
      // 捕获异常
      System.out.println(e);
    }
  }
}
```
使用`throws`声明抛出异常时有一个限制，是方法重写中的一条规则：子类方法声明抛出的异常类型应该是父类方法声明抛出的异常类型的子类或相同，子类方法声明抛出的异常不允许比父类方法声明抛出的异常多。
```java
public class OverrideThrows {
  public void test() throws IOException {
    FileInputStream fis = new FileInputStream("a.txt");
  }
}
class Sub extends OverrideThrows {
  // 子类方法声明抛出了比父类方法更大的异常
  // 所以下面方法出错
  public void test() throws Exception {
  }
}
```
上面程序中`Sub`子类中的`test()`方法声明抛出`Exception`，该`Exception`是其父类声明抛出异常`IOException`类的父类，这将导致程序无法通过编译。

所以在编写类继承代码时要注意，子类在重写父类带`throws`子句的方法时，子类方法声明中的`throws`子句不能出现父类对应方法的`throws`子句中没有的异常类型，因此`throws`子句可以限制子类的行为。也就是说，子类方法拋出的异常不能超过父类定义的范围。
## throw 拋出异常
`throw`语句用来直接拋出一个异常，后接一个可拋出的异常类对象，其语法格式如下：
```
throw ExceptionObject;
```
其中，`ExceptionObject`必须是`Throwable`类或其子类的对象。如果是自定义异常类，也必须是`Throwable`的直接或间接子类。例如，以下语句在编译时将会产生语法错误：
```
throw new String("拋出异常");    // String类不是Throwable类的子类
```
当`throw`语句执行时，它后面的语句将不执行，此时程序转向调用者程序，寻找与之相匹配的`catch`语句，执行相应的异常处理程序。如果没有找到相匹配的`catch`语句，则再转向上一层的调用程序。这样逐层向上，直到最外层的异常处理程序终止程序并打印出调用栈情况。

`throw`关键字不会单独使用，它的使用完全符合异常的处理机制，但是，一般来讲用户都在避免异常的产生，所以不会手工抛出一个新的异常类的实例，而往往会抛出程序中已经产生的异常类的实例。

在某仓库管理系统中，要求管理员的用户名需要由 8 位以上的字母或者数字组成，不能含有其他的字符。当长度在 8 位以下时拋出异常，并显示异常信息；当字符含有非字母或者数字时，同样拋出异常，显示异常信息。代码如下：
```java
import java.util.Scanner;
public class Test05 {
  public boolean validateUserName(String username) {
    boolean con = false;
    if (username.length() > 8) {
      // 判断用户名长度是否大于8位
      for (int i = 0; i < username.length(); i++) {
        char ch = username.charAt(i); // 获取每一位字符
        if ((ch >= '0' && ch <= '9') || (ch >= 'a' && ch <= 'z') || (ch >= 'A' && ch <= 'Z')) {
          con = true;
        } else {
          con = false;
          throw new IllegalArgumentException("用户名只能由字母和数字组成！");
        }
      }
    } else {
      throw new IllegalArgumentException("用户名长度必须大于 8 位！");
    }
    return con;
  }
  public static void main(String[] args) {
    Test05 te = new Test05();
    Scanner input = new Scanner(System.in);
    System.out.println("请输入用户名：");
    String username = input.next();
    try {
      boolean con = te.validateUserName(username);
      if (con) {
        System.out.println("用户名输入正确！");
      }
    } catch (IllegalArgumentException e) {
      System.out.println(e);
    }
  }
}
```
`throws`关键字和`throw`关键字在使用上的几点区别如下：
* `throws`用来声明一个方法可能抛出的所有异常信息，表示出现异常的一种可能性，但并不一定会发生这些异常；`throw`则是指拋出的一个具体的异常类型，执行`throw`则一定抛出了某种异常对象。
通常在一个方法（类）的声明处通过`throws`声明方法（类）可能拋出的异常信息，而在方法（类）内部通过`throw`声明一个具体的异常信息。
* `throws`通常不用显示地捕获异常，可由系统自动将所有捕获的异常信息抛给上级方法；`throw`则需要用户自己捕获相关的异常，而后再对其进行相关包装，最后将包装后的异常信息抛出。

# 多异常捕获
多`catch`代码块虽然客观上提高了程序的健壮性，但是也导致了程序代码量大大增加。如果有些异常种类不同，但捕获之后的处理是相同的，例如以下代码。
```java
try{
  // 可能会发生异常的语句
} catch (FileNotFoundException e) {
  // 调用方法methodA处理
} catch (IOException e) {
  // 调用方法methodA处理
} catch (ParseException e) {
  // 调用方法methodA处理
}
```
3 个不同类型的异常，要求捕获之后的处理都是调用`methodA`方法。为了解决这种问题，Java 7 推出了多异常捕获技术，可以把这些异常合并处理。
```java
try{
  // 可能会发生异常的语句
} catch (IOException | ParseException e) {
  // 调用方法methodA处理
}
```
注意：由于`FileNotFoundException`属于`IOException`异常，`IOException`异常可以捕获它的所有子类异常。所以不能写成`FileNotFoundException | IOException | ParseException`。

使用一个`catch`块捕获多种类型的异常时需要注意如下两个地方。
* 捕获多种类型的异常时，多种异常类型之间用竖线|隔开。
* 捕获多种类型的异常时，异常变量有隐式的`final`修饰，因此程序不能对异常变量重新赋值。

```java
public class ExceptionTest {
  public static void main(String[] args) {
    try {
      int a = Integer.parseInt(args[0]);
      int b = Integer.parseInt(args[1]);
      int c = a / b;
      System.out.println("您输入的两个数相除的结果是：" + c);
    } catch (IndexOutOfBoundsException | NumberFormatException | ArithmeticException e) {
      System.out.println("程序发生了数组越界、数字格式异常、算术异常之一");
      // 捕获多异常时，异常变量默认有final修饰
      // 所以下面代码有错
      e = new ArithmeticException("test");
    } catch (Exception e) {
      System.out.println("未知异常");
      // 捕获一种类型的异常时，异常变量没有final修饰
      // 所以下面代码完全正确
      e = new RuntimeException("test");
    }
  }
}
```
捕获多种类型的异常时，异常变量使用隐式的`final`修饰，因此上面程序的第 12 行代码将产生编译错误；捕获一种类型的异常时，异常变量没有`final`修饰，因此上面程序的第 17 行代码完全正确。
# 自定义异常
如果 Java 提供的内置异常类型不能满足程序设计的需求，这时我们可以自己设计 Java 类库或框架，其中包括异常类型。实现自定义异常类需要继承`Exception`类或其子类，如果自定义运行时异常类需继承`RuntimeException`类或其子类。

自定义异常的语法形式为：
```
<class> <自定义异常名> <extends> <Exception>
```
一般将自定义异常类的类名命名为`XXXException`，其中`XXX`用来代表该异常的作用。

自定义异常类一般包含两个构造方法：一个是无参的默认构造方法，另一个构造方法以字符串的形式接收一个定制的异常消息，并将该消息传递给超类的构造方法。

创建一个名称为`IntegerRangeException`的自定义异常类：
```java
class IntegerRangeException extends Exception {
  public IntegerRangeException() {
    super();
  }
  public IntegerRangeException(String s) {
    super(s);
  }
}
```
编写一个程序，对会员注册时的年龄进行验证，即检测是否在 0~100 岁。
```java
public class MyException extends Exception {
  public MyException() {
    super();
  }
  public MyException(String str) {
    super(str);
  }
}
```
```java
import java.util.InputMismatchException;
import java.util.Scanner;
public class Test07 {
  public static void main(String[] args) {
    int age;
    Scanner input = new Scanner(System.in);
    System.out.println("请输入您的年龄：");
    try {
      age = input.nextInt();    // 获取年龄
      if(age < 0) {
        throw new MyException("您输入的年龄为负数！输入有误！");
      } else if(age > 100) {
        throw new MyException("您输入的年龄大于100！输入有误！");
      } else {
        System.out.println("您的年龄为："+age);
      }
    } catch(InputMismatchException e1) {
      System.out.println("输入的年龄不是数字！");
    } catch(MyException e2) {
      System.out.println(e2.getMessage());
    }
  }
}
```
程序的运行结果如下。
```
请输入您的年龄：
-2
您输入的年龄为负数！输入有误！
```
在该程序的主方法中，使用了`if…else if…else`语句结构判断用户输入的年龄是否为负数和大于 100 的数，如果是，则拋出自定义异常`MyException`，调用自定义异常类`MyException`中的含有一个`String`类型的构造方法。在`catch`语句块中捕获该异常，并调用`getMessage()`方法输出异常信息。

提示：因为自定义异常继承自`Exception`类，因此自定义异常类中包含父类所有的属性和方法。
# 异常跟踪栈
异常对象的`printStackTrace()`方法用于打印异常的跟踪栈信息，根据`printStackTrace()`方法的输出结果，可以找到异常的源头，并跟踪到异常一路触发的过程。
```java
class SelfException extends RuntimeException {
  SelfException() {}
  SelfException(String msg) {
    super(msg);
  }
}
public class PrintStackTraceTest {
  public static void main(String[] args) {
    firstMethod();
  }
  public static void firstMethod() {
    secondMethod();
  }
  public static void secondMethod() {
    thirdMethod();
  }
  public static void thirdMethod() {
    throw new SelfException("自定义异常信息");
  }
}
```
运行上面程序，会看到如下所示的结果。
```
Exception in thread "main" Test.SelfException: 自定义异常信息
        at Test.PrintStackTraceTest.thirdMethod(PrintStackTraceTest.java:26)
        at Test.PrintStackTraceTest.secondMethod(PrintStackTraceTest.java:22)
        at Test.PrintStackTraceTest.firstMethod(PrintStackTraceTest.java:18)
        at Test.PrintStackTraceTest.main(PrintStackTraceTest.java:14)
```
上面运行结果的第 2 行到第 5 行之间的内容是异常跟踪栈信息，从打印的异常信息我们可以看出，异常从`thirdMethod`方法开始触发，传到`secondMethod`方法，再传到`firstMethod`方法，最后传到`main`方法，在`main`方法终止，这个过程就是 Java 的异常跟踪栈。

面向对象的应用程序运行时，经常会发生一系列方法调用，从而形成“方法调用栈”，异常的传播则相反：只要异常没有被完全捕获（包括异常没有被捕获，或异常被处理后重新抛出了新异常），异常从发生异常的方法逐渐向外传播，首先传给该方法的调用者，该方法调用者再次传给其调用者……，直至最后传到`main`方法，如果`main`方法依然没有处理该异常，则 JVM 会中止该程序，并打印异常的跟踪栈信息。

异常跟踪栈信息的第一行一般详细显示异常的类型和异常的详细消息，接下来是所有异常的发生点，各行显示被调用方法中执行的停止位置，并标明类、类中的方法名、与故障点对应的文件的行。一行行地往下看，跟踪栈总是最内部的被调用方法逐渐上传，直到最外部业务操作的起点，通常就是程序的入口`main`方法或`Thread`类的`run`方法（多线程的情形）。

下面例子程序示范了多线程程序中发生异常的情形。
```java
public class ThreadExceptionTest implements Runnable {
  public void run() {
    firstMethod();
  }
  public void firstMethod() {
    secondMethod();
  }
  public void secondMethod() {
    int a = 5;
    int b = 0;
    int c = a / b;
  }
  public static void main(String[] args) {
    new Thread(new ThreadExceptionTest()).start();
  }
}
```
运行结果如下。
```
Exception in thread "Thread-0" java.lang.ArithmeticException: / by zero
        at Test.ThreadExceptionTest.secondMethod(ThreadExceptionTest.java:14)
        at Test.ThreadExceptionTest.firstMethod(ThreadExceptionTest.java:8)
        at Test.ThreadExceptionTest.run(ThreadExceptionTest.java:4)
        at java.lang.Thread.run(Unknown Source)
```
多线程异常的跟踪栈，从发生异常的方法开始，到线程的`run`方法结束。从上面的运行结果可以看出，程序在`Thread`的`run`方法中出现了`ArithmeticException`异常，这个异常的源头是`ThreadExcetpionTest`的`secondMethod`方法，位于`ThreadExcetpionTest.java`文件的 14 行。这个异常传播到`Thread`类的`run`方法就会结束（如果该异常没有得到处理，将会导致该线程中止运行）。

调用`Exception`的`printStackTrace()`方法就是打印该异常的跟踪栈信息，也就会看到上面两个示例运行结果中的信息。当然，如果方法调用的层次很深，将会看到更加复杂的异常跟踪栈。

提示：虽然`printStackTrace()`方法可以很方便地用于追踪异常的发生情况，可以用它来调试程序，但在最后发布的程序中，应该避免使用它。应该对捕获的异常进行适当的处理，而不是简单地将异常的跟踪栈信息打印出来。