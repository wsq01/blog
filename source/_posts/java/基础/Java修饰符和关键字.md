---
title: Java 修饰符和关键字
date: 2020-09-17 18:03:24
tags: [java]
categories: java
---

# 访问控制修饰符
在 Java 语言中提供了多个作用域修饰符，其中常用的有`public、private、protected、final、abstract、static、transient`和`volatile`，这些修饰符有类修饰符、变量修饰符和方法修饰符。

通过使用访问控制修饰符来限制对对象私有属性的访问，可以获得 3 个重要的好处。
* 防止对封装数据的未授权访问。
* 有助于保证数据完整性。
* 当类的私有实现细节必须改变时，可以限制发生在整个应用程序中的“连锁反应”。

访问控制符是一组限定类、属性或方法是否可以被程序里的其他部分访问和调用的修饰符。类的访问控制符只能是空或者`public`，方法和属性的访问控制符有 4 个，分别是`public、 private、protected`和`friendly`，其中`friendly`是一种没有定义专门的访问控制符的默认情况。

| 访问范围         | private  | friendly(默认) | protected | public |
| :--: | :--: | :--: | :--: | :--: |
| 同一个类         | 可访问   | 可访问   | 可访问   | 可访问 |
| 同一包中的其他类 | 不可访问 | 可访问   | 可访问	  | 可访问 |
| 不同包中的子类   | 不可访问 | 不可访问 | 可访问	  | 可访问 |
| 不同包中的非子类 | 不可访问 | 不可访问 | 不可访问 | 可访问 |

1. `private`
用 private 修饰的类成员，只能被该类自身的方法访问和修改，而不能被任何其他类（包括该类的子类）访问和引用。因此，`private`修饰符具有最高的保护级别。
2. `friendly`（默认）
如果一个类没有访问控制符，说明它具有默认的访问控制特性。这种默认的访问控制权规定，该类只能被同一个包中的类访问和引用，而不能被其他包中的类使用，即使其他包中有该类的子类。这种访问特性又称为包访问性（`package private`）。
同样，类内的成员如果没有访问控制符，也说明它们具有包访问性，或称为友元（`friend`）。定义在同一个文件夹中的所有类属于一个包，所以前面的程序要把用户自定义的类放在同一个文件夹中（Java 项目默认的包），以便不加修饰符也能运行。
3. `protected`
用保护访问控制符`protected`修饰的类成员可以被三种类所访问：该类自身、与它在同一个包中的其他类以及在其他包中的该类的子类。使用`protected`修饰符的主要作用，是允许其他包中它的子类来访问父类的特定属性和方法，否则可以使用默认访问控制符。
4. `public`
当一个类被声明为`public`时，它就具有了被其他包中的类访问的可能性，只要包中的其他类在程序中使用`import`语句引入`public`类，就可以访问和引用这个类。
类中被设定为`public`的方法是这个类对外的接口部分，避免了程序的其他部分直接去操作类内的数据，实际就是数据封装思想的体现。每个 Java 程序的主类都必须是`public`类，也是基于相同的原因。

```java
class Student {
  // 姓名，其访问权限为默认(friendly)
  String name;
  // 定义私有变量，身份证号码
  private String idNumber;
  // 定义受保护变量，学号
  protected String no;
  // 定义共有变量，邮箱
  public String email;
  // 定义共有方法，显示学生信息
  public String info() {
    return"姓名："+name+"，身份证号码："+idNumber+"，学号："+no+"，邮箱："+email;
  }
}
```
```java
public class StudentTest {
  public static void main(String[] args) {
    // 创建Student类对象
    Student stu = new Student();
    // 向Student类对象中的属性赋值
    stu.name = "zhht";
    // stu.idNumber="043765290763137806";
    // 这是不允许的。提示stu.idNumber是不可见的，必须注释掉才可运行
    stu.no = "20lil01637";
    stu.email = "zhht@qq.com";
    System.out.println(stu.info());
  }
}
```
从上面的例子中可以看出，范围控制修饰符成功地限制了访问者访问不同修饰符的属性（成员变量），从而实现了数据的隐藏。
# static关键字
在类中，使用`static`修饰符修饰的属性（成员变量）称为静态变量，也可以称为类变量，常量称为静态常量，方法称为静态方法或类方法，它们统称为静态成员，归整个类所有。

静态成员不依赖于类的特定实例，被类的所有实例共享，就是说`static`修饰的方法或者变量不需要依赖于对象来进行访问，只要这个类被加载，Java 虚拟机就可以根据类名找到它们。

调用静态成员的语法形式如下：
```
类名.静态成员
```
注意：
* `static`修饰的成员变量和方法，从属于类。
* 普通变量和方法从属于对象。
* 静态方法不能调用非静态成员，编译会报错。

## 静态变量
类的成员变量可以分为以下两种：
* 静态变量（或称为类变量），指被`static`修饰的成员变量。
* 实例变量，指没有被`static`修饰的成员变量。

静态变量与实例变量的区别如下：
1. 静态变量
 * 运行时，Java 虚拟机只为静态变量分配一次内存，在加载类的过程中完成静态变量的内存分配。
 * 在类的内部，可以在任何方法内直接访问静态变量。
 * 在其他类中，可以通过类名访问该类中的静态变量。
2. 实例变量
 * 每创建一个实例，Java 虚拟机就会为实例变量分配一次内存。
 * 在类的内部，可以在非静态方法中直接访问实例变量。
 * 在本类的静态方法或其他类中则需要通过类的实例对象进行访问。

静态变量在类中的作用如下：
* 静态变量可以被类的所有实例共享，因此静态变量可以作为实例之间的共享数据，增加实例之间的交互性。
* 如果类的所有实例都包含一个相同的常量属性，则可以把这个属性定义为静态常量类型，从而节省内存空间。例如，在类中定义一个静态常量 PI。
```java
public static double PI = 3.14159256;
```

创建一个带静态变量的类，然后在`main()`方法中访问该变量并输出结果。
```java
public class StaticVar {
  public static String str1 = "Hello";
  public static void main(String[] args) {
    String str2 = "World!";
    // 直接访问str1
    String accessVar1 = str1+str2;
    System.out.println("第 1 次访问静态变量，结果为："+accessVar1);
    // 通过类名访问str1
    String accessVar2 = StaticVar.str1+str2;
    System.out.println("第 2 次访问静态变量，结果为："+accessVar2);
    // 通过对象svt1访问str1
    StaticVar svt1 = new StaticVar();
    svt1.str1 = svt1.str1+str2;
    String accessVar3 = svt1.str1;
    System.out.println("第3次访向静态变量，结果为："+accessVar3);
    // 通过对象svt2访问str1
    StaticVar svt2 = new StaticVar();
    String accessVar4 = svt2.str1+str2;
    System.out.println("第 4 次访问静态变量，结果为："+accessVar4);
  }
}
```
运行该程序后的结果如下所示。
```
第 1 次访问静态变量，结果为：HelloWorld!
第 2 次访问静态变量，结果为：HelloWorld!
第 3 次访向静态变量，结果为：HelloWorld!
第 4 次访问静态变量，结果为：HelloWorld!World!
```
从运行结果可以看出，在类中定义静态的属性（成员变量），在`main()`方法中可以直接访问，也可以通过类名访问，还可以通过类的实例对象来访问。

注意：静态变量是被多个实例所共享的。
## 静态方法
与成员变量类似，成员方法也可以分为以下两种：
* 静态方法（或称为类方法），指被`static`修饰的成员方法。
* 实例方法，指没有被`static`修饰的成员方法。

静态方法与实例方法的区别如下：
* 静态方法不需要通过它所属的类的任何实例就可以被调用，因此在静态方法中不能使用`this`关键字，也不能直接访问所属类的实例变量和实例方法，但是可以直接访问所属类的静态变量和静态方法。另外，和`this`关键字一样，`super`关键字也与类的特定实例相关，所以在静态方法中也不能使用`super`关键字。
* 在实例方法中可以直接访问所属类的静态变量、静态方法、实例变量和实例方法。

创建一个带静态变量的类，添加几个静态方法对静态变量的值进行修改，然后在`main()`方法中调用静态方法并输出结果。
```java
public class StaticMethod {
  public static int count = 1;    // 定义静态变量count
  public int method1() {    
    // 实例方法method1
    count++;    // 访问静态变量count并赋值
    System.out.println("在静态方法 method1()中的 count="+count);    // 打印count
    return count;
  }
  public static int method2() {    
    // 静态方法method2
    count += count;    // 访问静态变量count并赋值
    System.out.println("在静态方法 method2()中的 count="+count);    // 打印count
    return count;
  }
  public static void PrintCount() {    
    // 静态方法PrintCount
    count += 2;
    System.out.println("在静态方法 PrintCount()中的 count="+count);    // 打印count
  }
  public static void main(String[] args) {
    StaticMethod sft = new StaticMethod();
    // 通过实例对象调用实例方法
    System.out.println("method1() 方法返回值 intro1="+sft.method1());
    // 直接调用静态方法
    System.out.println("method2() 方法返回值 intro1="+method2());
    // 通过类名调用静态方法，打印 count
    StaticMethod.PrintCount();
  }
}
```
运行该程序后的结果如下所示。
```
在静态方法 method1()中的 count=2
method1() 方法返回值 intro1=2
在静态方法 method2()中的 count=4
method2() 方法返回值 intro1=4
在静态方法 PrintCount()中的 count=6
```
在该程序中，静态变量`count`作为实例之间的共享数据，因此在不同的方法中调用`count`，值是不一样的。从该程序中可以看出，在静态方法`method1()`和`PrintCount()`中是不可以调用非静态方法`method1()`的，而在`method1()`方法中可以调用静态方法`method2()`和`PrintCount()`。

在访问非静态方法时，需要通过实例对象来访问，而在访问静态方法时，可以直接访问，也可以通过类名来访问，还可以通过实例化对象来访问。
## 静态代码块
静态代码块指 Java 类中的`static{ }`代码块，主要用于初始化类，为类的静态变量赋初始值，提升程序性能。

静态代码块的特点如下：
* 静态代码块类似于一个方法，但它不可以存在于任何方法体中。
* 静态代码块可以置于类中的任何地方，类中可以有多个静态初始化块。 
* Java 虚拟机在加载类时执行静态代码块，所以很多时候会将一些只需要进行一次的初始化操作都放在`static`代码块中进行。
* 如果类中包含多个静态代码块，则 Java 虚拟机将按它们在类中出现的顺序依次执行它们，每个静态代码块只会被执行一次。
* 静态代码块与静态方法一样，不能直接访问类的实例变量和实例方法，而需要通过类的实例对象来访问。

编写一个 Java 类，在类中定义一个静态变量，然后使用静态代码块修改静态变量的值。最后在 main() 方法中进行测试和输出。
```java
public class StaticCode {
  public static int count = 0;
  {
    count++;
    System.out.println("非静态代码块 count=" + count);
  }
  static {
    count++;
    System.out.println("静态代码块1 count=" + count);
  }
  static {
    count++;
    System.out.println("静态代码块2 count=" + count);
  }
  public static void main(String[] args) {
    System.out.println("*************** StaticCode1 执行 ***************");
    StaticCode sct1 = new StaticCode();
    System.out.println("*************** StaticCode2 执行 ***************");
    StaticCode sct2 = new StaticCode();
  }
}
```
如上述示例，为了说明静态代码块只被执行一次，特地添加了非静态代码块作为对比，并在主方法中创建了两个类的实例对象。上述示例的执行结果为：
```
静态代码块1 count=1
静态代码块2 count=2
*************** StaticCode1 执行 ***************
非静态代码块 count=3
*************** StaticCode2 执行 ***************
非静态代码块 count=4
```
上述代码中`{ }`代码块为非静态代码块，非静态代码块是在创建对象时自动执行的代码，不创建对象不执行该类的非静态代码块。代码域中定义的变量都是局部的，只有域中的代码可以调用。 
# final修饰符
`final`在 Java 中的意思是最终，表示对象是最终形态的，不可改变的意思。`final`应用于类、方法和变量时意义是不同的，但本质是一样的，都表示不可改变。

使用`final`关键字声明类、变量和方法需要注意以下几点：
* `final`用在变量的前面表示变量的值不可以改变，此时该变量可以被称为常量。
* `final`用在方法的前面表示方法不可以被重写。
* `final`用在类的前面表示该类不能有子类，即该类不可以被继承。

## final 修饰变量
`final`修饰的变量即成为常量，只能赋值一次，但是`final`所修饰局部变量和成员变量有所不同。
* `final`修饰的局部变量必须使用之前被赋值一次才能使用。
* `final`修饰的成员变量在声明时没有赋值的叫“空白`final`变量”。空白`final`变量必须在构造方法或静态代码块中初始化。

> 注意：`final`修饰的变量不能被赋值这种说法是错误的，严格的说法是，`final`修饰的变量不可被改变，一旦获得了初始值，该`final`变量的值就不能被重新赋值。

当使用`final`修饰基本类型变量时，不能对基本类型变量重新赋值，因此基本类型变量不能被改变。 但对于引用类型变量而言，它保存的仅仅是一个引用，final 只保证这个引用类型变量所引用的地址不会改变，即一直引用同一个对象，但这个对象完全可以发生改变。

如果一个程序中的变量使用`public static final`声明，则此变量将称为全局变量。
```
public static final String SEX= "女";
```
## final修饰方法
`final`修饰的方法不可被重写，如果出于某些原因，不希望子类重写父类的某个方法，则可以使用`final`修饰该方法。

Java 提供的`Object`类里就有一个`final`方法`getClass()`，因为 Java 不希望任何类重写这个方法，所以使用`final`把这个方法密封起来。但对于该类提供的`toString()`和`equals()`方法，都允许子类重写，因此没有使用`final`修饰它们。
```java
public class FinalMethodTest {
  public final void test() {
  }
}
class Sub extends FinalMethodTest {
  // 下面方法定义将出现编译错误，不能重写final方法
  public void test() {
  }
}
```
上面程序中父类是`FinalMethodTest`，该类里定义的`test()`方法是一个`final`方法，如果其子类试图重写该方法，将会引发编译错误。

对于一个`private`方法，因为它仅在当前类中可见，其子类无法访问该方法，所以子类无法重写该方法——如果子类中定义一个与父类`private`方法有相同方法名、相同形参列表、相同返回值类型的方法，也不是方法重写，只是重新定义了一个新方法。因此，即使使用`final`修饰一个`private`访问权限的方法，依然可以在其子类中定义与该方法具有相同方法名、相同形参列表、相同返回值类型的方法。
```java
public class PrivateFinalMethodTest {
  private final void test() {
  }
}
class Sub extends PrivateFinalMethodTest {
  // 下面的方法定义不会出现问题
  public void test() {
  }
}
```
上面程序没有任何问题，虽然子类和父类同样包含了同名的`void test()`方法，但子类并不是重写父类的方法，因此即使父类的`void test()`方法使用了`final`修饰，子类中依然可以定义`void test()`方法。

`final`修饰的方法仅仅是不能被重写，并不是不能被重载，因此下面程序完全没有问题。
```java
public class FinalOverload {
  // final 修饰的方法只是不能被重写，完全可以被重载
  public final void test(){}
  public final void test(String arg){}
}
```
## final修饰类
`final`修饰的类不能被继承。当子类继承父类时，将可以访问到父类内部数据，并可通过重写父类方法来改变父类方法的实现细节，这可能导致一些不安全的因素。为了保证某个类不可被继承，则可以使用`final`修饰这个类。
```java
final class SuperClass {
}
class SubClass extends SuperClass {    //编译错误
}
```
## final 修饰符使用总结
1. `final`修饰类中的变量
表示该变量一旦被初始化便不可改变，这里不可改变的意思对基本类型变量来说是其值不可变，而对对象引用类型变量来说其引用不可再变。其初始化可以在两个地方：一是其定义处，也就是说在`final`变量定义时直接给其赋值；二是在构造方法中。这两个地方只能选其一，要么在定义时给值，要么在构造方法中给值，不能同时既在定义时赋值，又在构造方法中赋予另外的值。
2. `final`修饰类中的方法
说明这种方法提供的功能已经满足当前要求，不需要进行扩展，并且也不允许任何从此类继承的类来重写这种方法，但是继承仍然可以继承这个方法，也就是说可以直接使用。在声明类中，一个`final`方法只被实现一次。
3. `final`修饰类
表示该类是无法被任何其他类继承的，意味着此类在一个继承树中是一个叶子类，并且此类的设计已被认为很完美而不需要进行修改或扩展。

对于`final`类中的成员，可以定义其为`final`，也可以不是`final`。而对于方法，由于所属类为`final`的关系，自然也就成了`final`型。也可以明确地给`final`类中的方法加上一个`final`，这显然没有意义。