


饰符是一种添加到定义以更改其含义的关键字。Java 语言有各种各样的修饰符，包括以下两种：
* Java 访问修饰符 - 例如：`private,protected,public`等。
* Java 非访问修饰符 - 例如：`static,final`等。

要使用修饰符，请在类，方法或变量的定义中包含修饰符关键字。修饰符位于语句之前。
```java
public class className {
  // ...
}

private boolean myFlag;
static final double weeks = 9.5;
protected static final int BOXWIDTH = 42;

public static void main(String[] arguments) {
  // body of method
}
```
# 访问修饰符
Java 提供了四个访问修饰符来设置类，变量，方法和构造函数的访问级别：
* 无关键字(不指定修饰符)：对包可见，不需要修饰符。
* `private`：仅对类内部可见。
* `public`：公开，对外部可见。
* `protected`：对包和所有子类可见。

## 默认访问修饰符 - 无关键字
默认访问修饰是指没有为类，字段，方法等显式声明访问修饰符。声明没有任何访问控制修饰符的变量或方法可用于同一包中的任何其他类。 接口中的字段隐式为`public static final`，接口中的方法默认为`public`。

可以在没有任何修饰符的情况下声明变量和方法：
```java
class Order{
  String version = "1.0.1";

  boolean processOrder() {
    return true;
  }
}
```
## 私有访问修饰符 - private
声明为`private`的方法，变量和构造函数只能在声明的类本身中访问。专用访问修饰符是限制性最强的访问级别。类和接口不能声明为：`private`。如果类中存在公共`getter`方法，则可以在类中将变量声明为`private`。使用`private`修饰符是对象封装自身并隐藏来自外部世界的数据的主要方式。
```java
public class Logger {
  private String format;

  public String getFormat() {
    return this.format;
  }

  public void setFormat(String format) {
    this.format = format;
  }
}
```
`Logger`类的`format`变量是`private`，因此其他类无法直接检索或设置它的值。

因此，为了使这个变量对外界可用，`Logger`类中定义了两个公共方法：`getFormat()`用于返回`format`的值，`setFormat(String)`用于设置它的值。
## 公共访问修饰符 - public
可以从任何其他类访问使用`public`声明的类，方法，构造函数，接口等。因此，可以从属于 Java 任何类访问在公共类中声明的字段，方法，块。

但是，如果尝试访问的公共类位于不同的包中，则仍需要导入公共类。 由于类继承，类的所有公共方法和变量都由其子类继承。
```java
public static void main(String[] arguments) {
  // ...
}
```
应用程序的`main()`方法必须声明为`public`。否则 Java 解释器无法调用它来运行该类。
## 受保护的访问修饰符 - protected
在超类中声明受保护的变量，方法和构造函数只能由其他包中的子类或受保护成员类的包中的任何类访问。受保护的访问修饰符不能应用于类和接口。方法，字段可以声明为`protected`，但是接口中的方法和字段不能声明为`protected`。受保护的访问使子类有机会使用辅助方法或变量，同时防止非相关类尝试使用它。

以下父类使用`protected`访问控制，以允许它的子类覆盖`openSpeaker()`方法。
```java
class AudioPlayer {
  protected boolean openSpeaker(Speaker sp) {
    // 实现详细代码...
  }
}

class StreamingAudioPlayer {
  boolean openSpeaker(Speaker sp) {
    // 实现详细代码...
  }
}
```
在这里，如果将`openSpeaker()`方法定义为`private`，那么它将无法从`AudioPlayer`以外的任何其他类访问。 如果将类定义为`public`，那么它将被所有外部世界类访问。但这里的目的是仅将此方法暴露给它的子类，所以只使用`protected`修饰符。
## 访问控制和继承
强制执行以下继承方法规则：
* 在超类中声明为`public`的方法也必须在所有子类中都是`public`。
* 在超类中声明为`protected`的方法必须在子类中也要声明为：`protected`或`public`; 不能声明为：`private`。
* 声明为`private`的方法根本不能被继承，因此没有规则。

# 非访问修饰符
Java提供了许多非访问修饰符来实现许多其他功能。
* `static`修饰符用于创建类方法和变量。
* `final`修饰符用于完成类，方法和变量的实现。
* `abstract`修饰符用于创建抽象类和方法。
* `synchronized`和`volatile`修饰符，用于线程。

## static修饰符
### 1. 静态变量
`static`关键字用于创建独立于类实例的变量。无论类的实例数有多少个，都只存在一个静态变量副本。静态变量也称为类变量。局部变量不能声明为`static`。
### 2. 静态方法
`static`关键字用于创建独立于类实例的方法。

静态方法不能使用作为类的对象的实例变量，静态方法也叫作类方法。静态方法从参数中获取所有数据并从这些参数计算某些内容，而不引用变量。可以使用类名后跟一个点(.)以及变量或方法的名称来访问类变量或方法。
```java
public class InstanceCounter {
  private static int numInstances = 0;

  protected static int getCount() {
    return numInstances;
  }

  private static void addInstance() {
    numInstances++;
  }

  InstanceCounter() {
    InstanceCounter.addInstance();
  }

  public static void main(String[] arguments) {
    System.out.println("Starting with " + InstanceCounter.getCount() + " instances");

    for (int i = 0; i < 500; ++i) {
      new InstanceCounter();
    }
    System.out.println("Created " + InstanceCounter.getCount() + " instances");
  }
}
// 执行上面代码，得到以下结果：
// Started with 0 instances
// Created 500 instances
```
## final修饰符
### 1. final变量
`final`变量只能显式地初始化一次，声明为`final`的引用变量永远不能重新分配以引用不同的对象。但是，可以更改对象内的数据。因此，可以更改对象的状态，但不能更改引用。对于变量，`final`修饰符通常与`static`一起使用，以使常量成为类变量。
```java
public class Test {
  final int value = 10;

  // 以下是声明常量的示例：
  public static final int BOXWIDTH = 6;
  static final String TITLE = "Manager";

  public void changeValue() {
    value = 12;   // 会出错，不能重新赋值
  }
}
```
### 2. final方法
任何子类都不能覆盖`final`方法。如前所述，`final`修饰符可防止在子类中修改方法。

声明`final`方法的主要目的是不让其它人改变方法的内容。
```java
public class Test {
  public final void changeName() {
    // 方法主体
  }
}
```
### 3. final类
使用声明为`final`的类的主要目的是防止类被子类化。 如果一个类被标记为`final`，那么这个类不能被其它类继承。
```java
public final class Test {
  // body of class
}
```
## abstract饰符
### 1. 抽象类
抽象(`abstract`)类不能实例化。如果一个类声明为抽象(`abstract`)，那么唯一的目的是扩展该类。

一个类不能是同时是`abstract`和`final`(因为`final`类不能被扩展)。 如果一个类包含抽象方法，那么该类应该被声明为`abstract`。否则，将抛出编译错误。

抽象类可以包含抽象方法以及普通方法。
```java
abstract class Caravan {
  private double price;
  private String model;
  private String year;
  public void getYear(String y){}；// 这是一个普通方法
  public abstract void goFast();   // 这是一个抽象方法
  public abstract void changeColor();// 这是一个抽象方法
}
```
### 2. 抽象方法
抽象方法是在没有任何实现的情况下声明的方法。方法体(实现)由子类提供。抽象方法永远不会是最终的或严格的。

扩展抽象类的任何类都必须实现超类的所有抽象方法，除非子类也是抽象类。

如果一个类包含一个或多个抽象方法，那么该类必须声明为`abstract`。抽象类不需要包含抽象方法。

抽象方法以分号结尾。示例：`public abstract sample();`
```java
public abstract class SuperClass {
  abstract void m();   // 抽象方法
}

class SubClass extends SuperClass {
  // 实现抽象方法
  void m() {
    // 实现代码.........
  }
}
```
## synchronized修饰符
`synchronized`关键字用于指示一次只能访问一个方法的方法。`synchronized`修饰符可以应用于四个访问级别修饰符中的任何一个。
```java
public synchronized void showDetails() {
  .......
}
```
## transient修饰符
实例变量标记为`transient`，表示 JVM 在序列化包含它的对象时跳过特定变量。
此修饰符包含在创建变量的语句中，位于变量的类或数据类型之前。
```java
public transient int limit = 55; // will not persist
public int b; // will persist
```
## volatile修饰符
`volatile`修饰符用于让 JVM 知道访问变量的线程必须始终将其自己的变量私有副本与内存中的主副本合并。
访问`volatile`变量会同步主内存中变量的所有缓存复制。`volatile`只能应用于实例变量，类型为`private`。`volatile`对象引用可以为`null`。
```java
public class MyRunnable implements Runnable {
  private volatile boolean active;

  public void run() {
    active = true;
    while (active) { // line 1
      // some code here
    }
  }

  public void stop() {
    active = false; // line 2
  }
}
```
通常，在一个线程(使用Runnable开始的线程)中调用run()，并从另一个线程调用stop()。 如果在第1行中使用了active的缓存值，那么当在第2行中将active设置为false时，循环可能不会停止。





