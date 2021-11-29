---
title: Java 继承和多态
date: 2020-09-19 17:08:23
tags: [java]
categories: java
---

# 类的封装
封装将类的某些信息隐藏在类内部，不允许外部程序直接访问，只能通过该类提供的方法来实现对隐藏信息的操作和访问。

封装的特点：
* 只能通过规定的方法访问数据。
* 隐藏类的实例细节，方便修改和实现。

实现封装的步骤：
* 修改属性的可见性来限制对属性的访问，一般设为`private`。
* 为每个属性创建一对赋值（`setter`）方法和取值（`getter`）方法，一般设为`public`，用于属性的读写。
* 在赋值和取值方法中，加入属性控制语句（对属性值的合法性进行判断）。

```java
public class Employee {
  private String name; // 姓名
  private int age; // 年龄
  private String phone; // 联系电话
  private String address; // 家庭住址
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }
  public int getAge() {
    return age;
  }
  public void setAge(int age) {
    // 对年龄进行限制
    if (age < 18 || age > 40) {
      System.out.println("年龄必须在18到40之间！");
      this.age = 20; // 默认年龄
    } else {
      this.age = age;
    }
  }
  public String getPhone() {
    return phone;
  }
  public void setPhone(String phone) {
    this.phone = phone;
  }
  public String getAddress() {
    return address;
  }
  public void setAddress(String address) {
    this.address = address;
  }
}
```
如上述代码所示，使用`private`关键字修饰属性，这就意味着除了`Employee`类本身外，其他任何类都不可以访问这些属性。但是，可以通过这些属性的`setXxx()`方法来对其进行赋值，通过`getXxx()`方法来访问这些属性。
```java
public class EmployeeTest {
  public static void main(String[] args) {
    Employee people = new Employee();
    people.setName("王丽丽");
    people.setAge(35);
    people.setPhone("13653835964");
    people.setAddress("河北省石家庄市");
    System.out.println("姓名：" + people.getName());
    System.out.println("年龄：" + people.getAge());
    System.out.println("电话：" + people.getPhone());
    System.out.println("家庭住址：" + people.getAddress());
  }
}
```
# 继承（extends）
Java 中的继承就是在已经存在类的基础上进行扩展，从而产生新的类。已经存在的类称为父类、基类或超类，而新产生的类称为子类或派生类。在子类中，不仅包含父类的属性和方法，还可以增加新的属性和方法。

子类继承父类的语法格式：
```java
修饰符 class class_name extends extend_class {
    // 类的主体
}
```
其中，`class_name`表示子类（派生类）的名称；`extend_class`表示父类（基类）的名称；`extends`关键字直接跟在子类名之后，其后面是该类要继承的父类名称。

Java 的继承通过`extends`关键字来实现，`extends`的英文意思是扩展，而不是继承。`extends`很好的体现了子类和父类的关系，即子类是对父类的扩展，子类是一种特殊的父类。

类的继承不改变类成员的访问权限，也就是说，如果父类的成员是公有的、被保护的或默认的，它的子类仍具有相应的这些特性，并且子类不能获得父类的构造方法。

教师和学生都属于人，他们具有共同的属性：姓名、年龄、性别和身份证号，而学生还具有学号和所学专业两个属性，教师还具有教龄和所教专业两个属性。下面编写 Java 程序代码，使教师（`Teacher`）类和学生（`Student`）类都继承于人（`People`）类，具体的实现步骤如下。
```java
public class People {
  public String name; // 姓名
  public int age; // 年龄
  public String sex; // 性别
  public String sn; // 身份证号
  public People(String name, int age, String sex, String sn) {
    this.name = name;
    this.age = age;
    this.sex = sex;
    this.sn = sn;
  }
  public String toString() {
    return "姓名：" + name + "\n年龄：" + age + "\n性别：" + sex + "\n身份证号：" + sn;
  }
}

public class Student extends People {
  private String stuNo; // 学号
  private String department; // 所学专业
  public Student(String name, int age, String sex, String sn, String stuno, String department) {
    super(name, age, sex, sn); // 调用父类中的构造方法
    this.stuNo = stuno;
    this.department = department;
  }
  public String toString() {
    return "姓名：" + name + "\n年龄：" + age + "\n性别：" + sex + "\n身份证号：" + sn + "\n学号：" + stuNo + "\n所学专业：" + department;
  }
}
```
由于`Student`类继承自`People`类，因此，在`Student`类中同样具有`People`类的属性和方法，这里重写了父类中的`toString()`方法。

注意：如果在父类中存在有参的构造方法而并没有重载无参的构造方法，那么在子类中必须含有有参的构造方法，因为如果在子类中不含有构造方法，默认会调用父类中无参的构造方法，而在父类中并没有无参的构造方法，因此会出错。
```java
public class Teacher extends People {
  private int tYear; // 教龄
  private String tDept; // 所教专业
  public Teacher(String name, int age, String sex, String sn, int tYear, String tDept) {
    super(name, age, sex, sn); // 调用父类中的构造方法
    this.tYear = tYear;
    this.tDept = tDept;
  }
  public String toString() {
    return "姓名：" + name + "\n年龄：" + age + "\n性别:" + sex + "\n身份证号：" + sn + "\n教龄：" + tYear + "\n所教专业：" + tDept;
  }
}
```
编写测试类`PeopleTest`，在该类中创建`People`类的不同对象，分别调用它们的`toString()`方法，输出不同的信息。
```java
public class PeopleTest {
  public static void main(String[] args) {
    // 创建Student类对象
    People stuPeople = new Student("王丽丽", 23, "女", "410521198902145589", "00001", "计算机应用与技术");
    System.out.println("----------------学生信息---------------------");
    System.out.println(stuPeople);
    // 创建Teacher类对象
    People teaPeople = new Teacher("张文", 30, "男", "410521198203128847", 5, "计算机应用与技术");
    System.out.println("----------------教师信息----------------------");
    System.out.println(teaPeople);
  }
}
```
输出的结果如下：
```
----------------学生信息---------------------
姓名：王丽丽
年龄：23
性别：女
身份证号：410521198902145589
学号：00001
所学专业：计算机应用与技术
----------------教师信息----------------------
姓名：张文
年龄：30
性别:男
身份证号：410521198203128847
教龄：5
所教专业：计算机应用与技术
```
## 单继承
Java 不支持多继承，只允许一个类直接继承另一个类，即子类只能有一个直接父类，`extends`关键字后面只能有一个类名。但是它可以有多个间接的父类。

如果定义一个类时并未显式指定这个类的直接父类，则这个类默认继承`java.lang.Object`类。因此，`java.lang.Object`类是所有类的父类，要么是其直接父类，要么是其间接父类。因此所有的 Java 对象都可调用`java.lang.Object`类所定义的实例方法。

使用继承的注意点：
* 子类一般比父类包含更多的属性和方法。
* 父类中的`private`成员在子类中是不可见的，因此在子类中不能直接使用它们。
* 父类和其子类间必须存在“是一个”即`is-a`的关系，否则不能用继承。但也并不是所有符合`is-a`关系的都应该用继承。例如，正方形是一个矩形，但不能让正方形类来继承矩形类，因为正方形不能从矩形扩展得到任何东西。正确的继承关系是正方形类继承图形类。
* Java 只允许单一继承（即一个子类只能有一个直接父类）。

## 继承的优缺点
优点：
* 实现代码共享，减少创建类的工作量，使子类可以拥有父类的方法和属性。
* 提高代码维护性和可重用性。
* 提高代码的可扩展性，更好的实现父类的方法。

缺点：
* 继承是侵入性的。只要继承，就必须拥有父类的属性和方法。
* 降低代码灵活性。子类拥有父类的属性和方法后多了些约束。
* 增强代码耦合性（开发项目的原则为高内聚低耦合）。当父类的常量、变量和方法被修改时，需要考虑子类的修改，有可能会导致大段的代码需要重构。

# super关键字
由于子类不能继承父类的构造方法，因此，如果要调用父类的构造方法，可以使用`super`关键字。`super`可以用来访问父类的构造方法、普通方法和属性。

`super`关键字的功能：
* 在子类的构造方法中显式的调用父类构造方法
* 访问父类的成员方法和变量。

## super调用父类构造方法
`super`关键字可以在子类的构造方法中显式地调用父类的构造方法，基本格式如下：
```
super(parameter-list);
```
其中，`parameter-list`指定了父类构造方法中的所有参数。`super()`必须是在子类构造方法的方法体的第一行。

声明父类`Person`和子类`Student`，在`Person`类中定义一个带有参数的构造方法：
```java
public class Person {
  public Person(String name) {}
}
public class Student extends Person {}
```
会发现`Student`类出现编译错误，提示必须显式定义构造方法，错误信息如下：
```
Implicit super constructor Person() is undefined for default constructor. Must define an explicit constructor
```
在本例中 JVM 默认给`Student`类加了一个无参构造方法，而在这个方法中默认调用了`super()`，但是`Person`类中并不存在该构造方法，所以会编译错误。

如果一个类中没有写任何的构造方法，JVM 会生成一个默认的无参构造方法。在继承关系中，由于在子类的构造方法中，第一条语句默认为调用父类的无参构造方法（即默认为`super()`，一般这行代码省略了）。所以当在父类中定义了有参构造方法，但是没有定义无参构造方法时，编译器会强制要求我们定义一个相同参数类型的构造方法。

声明父类`Person`，类中定义两个构造方法。
```java
public class Person {
    public Person(String name, int age) {}
    public Person(String name, int age, String sex) {}
}
```
子类`Student`继承了`Person`类，使用`super`语句来定义`Student`类的构造方法。
```java
public class Student extends Person {
  public Student(String name, int age, String birth) {
    super(name, age); // 调用父类中含有2个参数的构造方法
  }
  public Student(String name, int age, String sex, String birth) {
    super(name, age, sex); // 调用父类中含有3个参数的构造方法
  }
}
```
从上述`Student`类构造方法代码可以看出，`super`可以用来直接调用父类中的构造方法，使编写代码也更加简洁方便。

编译器会自动在子类构造方法的第一句加上`super();`来调用父类的无参构造方法，必须写在子类构造方法的第一句，也可以省略不写。通过`super`来调用父类其它构造方法时，只需要把相应的参数传过去。
## super访问父类成员
当子类的成员变量或方法与父类同名时，可以使用`super`关键字来访问。如果子类重写了父类的某一个方法，即子类和父类有相同的方法定义，但是有不同的方法体，此时，我们可以通过`super`来调用父类里面的这个方法。

使用`super`访问父类中的成员与`this`关键字的使用相似，只不过它引用的是子类的父类，语法格式如下：
```
super.member
```
其中，`member`是父类中的属性或方法。使用`super`访问父类的属性和方法时不用位于第一行。
## super调用成员属性
```java
class Person {
  int age = 12;
}
class Student extends Person {
  int age = 18;
  void display() {
    System.out.println("学生年龄：" + super.age);
  }
}
class Test {
  public static void main(String[] args) {
    Student stu = new Student();
    stu.display();
  }
}

// 输出结果为：
// 学生年龄：12
```
## super调用成员方法
可以使用`super`关键字访问父类的方法。
```java
class Person {
  void message() {
    System.out.println("This is person class");
  }
}
class Student extends Person {
  void message() {
    System.out.println("This is student class");
  }
  void display() {
    message();
    super.message();
  }
}
class Test {
  public static void main(String args[]) {
    Student s = new Student();
    s.display();
  }
}
```
输出结果为：
```
This is student class
This is person class
```
## super和this的区别
`this`指的是当前对象的引用，`super`是当前对象的父对象的引用。

`super`关键字的用法：
* `super.父类属性名`：调用父类中的属性
* `super.父类方法名`：调用父类中的方法
* `super()`：调用父类的无参构造方法
* `super(参数)`：调用父类的有参构造方法

如果构造方法的第一行代码不是`this()`和`super()`，则系统会默认添加`super()`。

`this`关键字的用法：
* `this.属性名`：表示当前对象的属性
* `this.方法名(参数)`：表示调用当前对象的方法

当局部变量和成员变量发生冲突时，使用`this.`进行区分。

`super`和`this`关键字的异同，可简单总结为以下几条。
* 子类和父类中变量或方法名称相同时，用`super`关键字来访问。可以理解为`super`是指向自己父类对象的一个指针。在子类中调用父类的构造方法。
* `this`是自身的一个对象，代表对象本身，可以理解为`this`是指向对象本身的一个指针。在同一个类中调用其它方法。
* `this`和`super`不能同时出现在一个构造方法里面，因为`this`必然会调用其它的构造方法，其它的构造方法中肯定会有`super`语句的存在，所以在同一个构造方法里面有相同的语句，就失去了语句的意义，编译器也不会通过。
* `this()`和`super()`都指的是对象，所以，均不可以在`static`环境中使用，包括`static`变量、`static`方法和`static`语句块。
* 从本质上讲，`this`是一个指向对象本身的指针, 然而`super`是一个 Java 关键字。

```java
// 父类Animal的定义
public class Animal {
  public String name; // 动物名字
}
//子类Cat的定义
public class Cat extends Animal {
  private String name; // 名字
  public Cat(String aname, String dname) {
    super.name = aname; // 通过super关键字来访问父类中的name属性
    this.name = dname; // 通过this关键字来访问本类中的name属性
  }
  public String toString() {
    return "我是" + super.name + "，我的名字叫" + this.name;
  }
  public static void main(String[] args) {
    Animal cat = new Cat("动物", "喵星人");
    System.out.println(cat);
  }
}
// 输出结果：
// 我是动物，我的名字叫喵星人
```
# 对象类型转换
将一个类型强制转换成另一个类型的过程被称为类型转换。对象类型转换，是指存在继承关系的对象，不是任意类型的对象。当对不存在继承关系的对象进行强制类型转换时，会抛出 Java 强制类型转换（`java.lang.ClassCastException`）异常。

Java 语言允许某个类型的引用变量引用子类的实例，而且可以对这个引用变量进行类型转换。Java 中引用类型之间的类型转换（前提是两个类是父子关系）主要有两种，分别是向上转型和向下转型。
### 向上转型
父类引用指向子类对象为向上转型，语法格式如下：
```java
fatherClass obj = new sonClass();
```
其中，`fatherClass`是父类名称或接口名称，`obj`是创建的对象，`sonClass`是子类名称。

向上转型就是把子类对象直接赋给父类引用，不用强制转换。使用向上转型可以调用父类类型中的所有成员，不能调用子类类型中特有成员，最终运行效果看子类的具体实现。
### 向下转型
与向上转型相反，子类对象指向父类引用为向下转型：
```java
sonClass obj = (sonClass) fatherClass;
```
其中，`fatherClass`是父类名称，`obj`是创建的对象，`sonClass`是子类名称。

向下转型可以调用子类类型中所有的成员，不过需要注意的是如果父类引用对象指向的是子类对象，那么在向下转型的过程中是安全的，也就是编译是不会出错误。但是如果父类引用对象是父类本身，那么在向下转型的过程中是不安全的，编译不会出错，但是运行时会出现我们开始提到的 Java 强制类型转换异常，一般使用`instanceof`运算符来避免出此类错误。

例如，`Animal`类表示动物类，该类对应的子类有`Dog`类，使用对象类型表示如下：
```java
Animal animal = new Dog();    // 向上转型，把Dog类型转换为Animal类型
Dog dog = (Dog) animal; // 向下转型，把Animal类型转换为Dog类型
```
下面通过具体的示例演示对象类型的转换。例如，父类`Animal`和子类`Cat`中都定义了实例变量`name`、静态变量`staticName`、实例方法`eat()`和静态方法`staticEat()`。此外，子类`Cat`中还定义了实例变量`str`和实例方法`eatMethod()`。
```java
// 父类 Animal 
public class Animal {
  public String name = "Animal：动物";
  public static String staticName = "Animal：可爱的动物";
  public void eat() {
    System.out.println("Animal：吃饭");
  }
  public static void staticEat() {
    System.out.println("Animal：动物在吃饭");
  }
}
// 子类 Cat： 
public class Cat extends Animal {
  public String name = "Cat：猫";
  public String str = "Cat：可爱的小猫";
  public static String staticName = "Dog：我是喵星人";
  public void eat() {
    System.out.println("Cat：吃饭");
  }
  public static void staticEat() {
    System.out.println("Cat：猫在吃饭");
  }
  public void eatMethod() {
    System.out.println("Cat：猫喜欢吃鱼");
  }
  public static void main(String[] args) {
    Animal animal = new Cat();
    Cat cat = (Cat) animal; // 向下转型
    System.out.println(animal.name); // 输出Animal类的name变量
    System.out.println(animal.staticName); // 输出Animal类的staticName变量
    animal.eat(); // 输出Cat类的eat()方法
    animal.staticEat(); // 输出Animal类的staticEat()方法
    System.out.println(cat.str); // 调用Cat类的str变量
    cat.eatMethod(); // 调用Cat类的eatMethod()方法
  }
}
```
通过引用类型变量来访问所引用对象的属性和方法时，Java 虚拟机将采用以下绑定规则：
* 实例方法与引用变量实际引用的对象的方法进行绑定，这种绑定属于动态绑定，因为是在运行时由 Java 虚拟机动态决定的。例如，`animal.eat()`是将`eat()`方法与`Cat`类绑定。
* 静态方法与引用变量所声明的类型的方法绑定，这种绑定属于静态绑定，因为是在编译阶段已经做了绑定。例如，`animal.staticEat()`是将`staticEat()`方法与`Animal`类进行绑定。
* 成员变量（包括静态变量和实例变量）与引用变量所声明的类型的成员变量绑定，这种绑定属于静态绑定，因为在编译阶段已经做了绑定。例如，`animal.name`和`animal.staticName`都是与`Animal`类进行绑定。

对于`Cat`类，运行时将会输出如下结果：
```
Animal：动物
Animal：可爱的动物
Cat：吃饭
Animal：动物在吃饭
Cat：可爱的小猫
Cat：猫喜欢吃鱼
```
## 强制对象类型转换
Java 编译器允许在具有直接或间接继承关系的类之间进行类型转换。对于向下转型，必须进行强制类型转换；对于向上转型，不必使用强制类型转换。

例如，对于一个引用类型的变量，Java 编译器按照它声明的类型来处理。如果使用`animal`调用`str`和`eatMethod()`方法将会出错：
```java
animal.str = "";    // 编译出错，提示Animal类中没有str属性
animal.eatMethod();    // 编译出错，提示Animal类中没有eatMethod()方法
```
如果要访问`Cat`类的成员，必须通过强制类型转换：
```java
((Cat)animal).str = "";    // 编译成功
((Cat)animal).eatMethod();    // 编译成功
```
把`Animal`对象类型强制转换为`Cat`对象类型，这时上面两句编译成功。对于如下语句，由于使用了强制类型转换，所以也会编译成功：
```java
Cat cat = (Cat)animal;    // 编译成功，将Animal对象类型强制转换为Cat对象类型
```
类型强制转换时想运行成功就必须保证父类引用指向的对象一定是该子类对象，最好使用`instanceof`运算符判断后，再强转：
```java
Animal animal = new Cat();
if (animal instanceof Cat) {
  Cat cat = (Cat) animal; // 向下转型
  ...
}
```
子类的对象可以转换成父类类型，而父类的对象实际上无法转换为子类类型。因为通俗地讲，父类拥有的成员子类肯定也有，而子类拥有的成员，父类不一定有。因此，对于向上转型，不必使用强制类型转换。例如：
```java
Cat cat = new Cat();
Animal animal = cat;    // 向上转型，不必使用强制类型转换
```
如果两种类型之间没有继承关系，那么将不允许进行类型转换。
```java
Dog dog = new Dog();
Cat cat = (Cat)dog;    // 编译出错，不允许把Dog对象类型转换为Cat对象类型
```
# 方法重载
Java 允许同一个类中定义多个同名方法，只要它们的形参列表不同即可。如果同一个类中包含了两个或两个以上方法名相同的方法，但形参列表不同，这种情况被称为方法重载。
```java
public void println(int i){…}
public void println(double d){…}
public void println(String s){…}
```
它们之间就构成了方法的重载。实际调用时，根据实参的类型来决定调用哪一个方法。
```java
System.out.println(102);    // 调用println(int i)方法
System.out.println(102.25);    // 调用println(double d)方法
System.out.println("价格为 102.25");    // 调用println(String s)方法
```
方法重载的要求是两同一不同：同一个类中方法名相同，参数列表不同。至于方法的其他部分，如方法返回值类型、修饰符等，与方法重载没有任何关系。

使用方法重载其实就是避免出现繁多的方法名，有些方法的功能是相似的，如果重新建立一个方法，重新取个方法名称，会降低程序可读性。
# 方法重写
在子类中如果创建了一个与父类中相同名称、相同返回值类型、相同参数列表的方法，只是方法体中的实现不同，以实现不同于父类的功能，这种方式被称为方法重写（override），又称为方法覆盖。当父类中的方法无法满足子类需求或子类具有特有功能的时候，需要方法重写。

子类可以根据需要，定义特定于自己的行为。既沿袭了父类的功能名称，又根据子类的需要重新实现父类方法，从而进行扩展增强。

在重写方法时，需要遵循下面的规则：
* 参数列表必须完全与被重写的方法参数列表相同。
* 返回的类型必须与被重写的方法的返回类型相同（返回值类型必须小于或者等于父类方法的返回值类型）。
* 访问权限不能比父类中被重写方法的访问权限更低（`public>protected>default>private`）。
* 重写方法一定不能抛出新的检査异常或者比被重写方法声明更加宽泛的检査型异常。例如，父类的一个方法声明了一个检査异常`IOException`，在重写这个方法时就不能抛出`Exception`，只能拋出`IOException`的子类异常，可以抛出非检査异常。

另外还要注意以下几条：
* 重写的方法可以使用`@Override`注解来标识。
* 父类的成员方法只能被它的子类重写。
* 声明为`final`的方法不能被重写。
* 声明为`static`的方法不能被重写，但是能够再次声明。
* 构造方法不能被重写。
* 子类和父类在同一个包中时，子类可以重写父类的所有方法，除了声明为`private`和`final`的方法。
* 子类和父类不在同一个包中时，子类只能重写父类的声明为`public`和`protected`的非`final`方法。
* 如果不能继承一个方法，则不能重写这个方法。

每种动物都有名字和年龄属性，但是喜欢吃的食物是不同的，比如狗喜欢吃骨头、猫喜欢吃鱼等，因此每种动物的介绍方式是不一样的。在父类`Animal`中定义`getInfo()`方法，并在子类`Cat`中重写该方法，实现猫的介绍方式。
```java
public class Animal {
  public String name; // 名字
  public int age; // 年龄
  public Animal(String name, int age) {
    this.name = name;
    this.age = age;
  }
  public String getInfo() {
    return "我叫" + name + "，今年" + age + "岁了。";
  }
}

public class Cat extends Animal {
  private String hobby;
  public Cat(String name, int age, String hobby) {
    super(name, age);
    this.hobby = hobby;
  }
  public String getInfo() {
    return "喵！大家好！我叫" + this.name + "，我今年" + this.age + "岁了，我爱吃" + hobby + "。";
  }
  public static void main(String[] args) {
    Animal animal = new Cat("小白", 2, "鱼");
    System.out.println(animal.getInfo());
  }
}

// 输出的结果如下：
喵！大家好！我叫小白，我今年2岁了，我爱吃鱼。
```
如果子类中创建了一个成员变量，而该变量的类型和名称都与父类中的同名成员变量相同，我们则称作变量隐藏。
# 多态性
多态性是指在父类中定义的属性和方法被子类继承之后，可以具有不同的数据类型或表现出不同的行为，这使得同一个属性或方法在父类及其各个子类中具有不同的含义。

多态分为编译时多态和运行时多态。其中编译时多态是静态的，主要是指方法的重载，它是根据参数列表的不同来区分不同的方法。通过编译之后会变成两个不同的方法，在运行时谈不上多态。而运行时多态是动态的，它是通过动态绑定来实现的，也就是大家通常所说的多态性。

实现多态有 3 个必要条件：继承、重写和向上转型。只有满足这 3 个条件，开发人员才能够在同一个继承结构中使用统一的逻辑实现代码处理不同的对象，从而执行不同的行为。
* 继承：在多态中必须存在有继承关系的子类和父类。
* 重写：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法。
* 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才既能可以调用父类的方法，又能调用子类的方法。

下面通过一个例子来演示重写如何实现多态性。例子使用了类的继承和运行时多态机制，具体步骤如下。

1）创建`Figure`类，在该类中首先定义存储二维对象的尺寸，然后定义有两个参数的构造方法，最后添加`area()`方法，该方法计算对象的面积。
```java
public class Figure {
  double dim1;
  double dim2;
  Figure(double d1, double d2) {
    // 有参的构造方法
    this.dim1 = d1;
    this.dim2 = d2;
  }
  double area() {
    // 用于计算对象的面积
    System.out.println("父类中计算对象面积的方法，没有实际意义，需要在子类中重写。");
    return 0;
  }
}
```
2）创建继承自`Figure`类的`Rectangle`子类，该类调用父类的构造方法，并且重写父类中的`area()`方法。
```java
public class Rectangle extends Figure {
  Rectangle(double d1, double d2) {
    super(d1, d2);
  }
  double area() {
    System.out.println("长方形的面积：");
    return super.dim1 * super.dim2;
  }
}
```
3）创建继承自`Figure`类的`Triangle`子类，该类与`Rectangle`相似。
```java
public class Triangle extends Figure {
  Triangle(double d1, double d2) {
    super(d1, d2);
  }
  double area() {
    System.out.println("三角形的面积：");
    return super.dim1 * super.dim2 / 2;
  }
}
```
4）创建`Test`测试类，在该类的`main()`方法中首先声明`Figure`类的变量`figure`，然后分别为`figure`变量指定不同的对象，并调用这些对象的`area()`方法。
```java
public class Test {
  public static void main(String[] args) {
    Figure figure; // 声明Figure类的变量
    figure = new Rectangle(9, 9);
    System.out.println(figure.area());
    System.out.println("===============================");
    figure = new Triangle(6, 8);
    System.out.println(figure.area());
    System.out.println("===============================");
    figure = new Figure(10, 10);
    System.out.println(figure.area());
  }
}
```
从上述代码可以发现，无论`figure`变量的对象是`Rectangle`还是`Triangle`，它们都是`Figure`类的子类，因此可以向上转型为该类，从而实现多态。

5）输出结果如下：
```
长方形的面积：
81.0
===============================
三角形的面积：
24.0
===============================
父类中计算对象面积的方法，没有实际意义，需要在子类中重写。
0.0
```
# instanceof关键字
`instanceof`关键字判断一个对象是否为一个类（或接口、抽象类、父类）的实例。
```java
boolean result = obj instanceof Class
```
其中，`obj`是一个对象，`Class`表示一个类或接口。`obj`是`class`类（或接口）的实例或者子类实例时，结果`result`返回`true`，否则返回`false`。

`instanceof`关键字的几种用法。
1. 声明一个`class`类的对象，判断`obj`是否为`class`类的实例对象（很普遍的一种用法）：
```java
Integer integer = new Integer(1);
System.out.println(integer instanceof  Integer);    // true
```
2. 声明一个`class`接口实现类的对象`obj`，判断`obj`是否为`class`接口实现类的实例对象：
```java
ArrayList arrayList = new ArrayList();
System.out.println(arrayList instanceof List);    // true
// 或者反过来也是返回 true
List list = new ArrayList();
System.out.println(list instanceof ArrayList);    // true
```
3. `obj`是`class`类的直接或间接子类
```java
public class Person {}
public class Man extends Person {}

Person p1 = new Person();
Person p2 = new Man();
Man m1 = new Man();
System.out.println(p1 instanceof Man);    // false
System.out.println(p2 instanceof Man);    // true
System.out.println(m1 instanceof Man);    // true
```

值得注意的是`obj`必须为引用类型，不能是基本类型。
```java
int i = 0;
System.out.println(i instanceof Integer);    // 编译不通过
System.out.println(i instanceof Object);    // 编译不通过
```
所以，`instanceof`运算符只能用作对象的判断。

当`obj`为`null`时，直接返回`false`，因为`null`没有引用任何对象。
```java
Integer i = 1;
System.out.println(i instanceof null);    // false
```
所以，`obj`的类型必须是引用类型或空类型，否则会编译错误。

当`class`为`null`时，会发生编译错误，所以`class`只能是类或者接口。

编译器会检查`obj`能否转换成右边的`class`类型，如果不能转换则直接报错，如果不能确定类型，则通过编译。
```java
Person p1 = new Person();
System.out.println(p1 instanceof String);    // 编译报错
System.out.println(p1 instanceof List);    // false
System.out.println(p1 instanceof List<?>);    // false
System.out.println(p1 instanceof List<Person>);    // 编译报错
```
上述代码中，`Person`的对象`p1`很明显不能转换为`String`对象，那么`p1 instanceof String`不能通过编译，但`p1 instanceof List`却能通过编译，而`instanceof List<Person>`又不能通过编译了。

可以理解成以下代码：
```java
boolean result;
if (obj == null) {
  result = false;    // 当obj为null时，直接返回false
} else {
  try {
    // 判断obj是否可以强制转换为T
    T temp = (T) obj; 
    result = true;
  } catch (ClassCastException e) {
    result = false;
  }
}
```
在`T`不为`null`和`obj`不为`null`时，如果`obj`可以转换为`T`而不引发异常（`ClassCastException`），则该表达式值为`true`，否则值为`false`。所以对于上面提出的问题就很好理解了，`p1 instanceof String`会编译报错，是因为`(String) p1`是不能通过编译的，而`(List) p1`可以通过编译。

`instanceof`也经常和三目（条件）运算符一起使用，代码如下：
```
A instanceof B ? A : C
```
判断`A`是否可以转换为`B`，如果可以转换返回`A`，不可以转换则返回`C`。
```java
public class Test {
  public Object animalCall(Animal a) {
    String tip = "这个动物不是牛！";
    // 判断参数a是不是Cow的对象
    return a instanceof Cow ? (Cow) a : tip;
  }
  public static void main(String[] args) {
    Sheep sh = new Sheep();
    Test test = new Test();
    System.out.println(test.animalCall(sh));
  }
}
class Animal {}
class Cow extends Animal {}
class Sheep extends Animal {}
```
在`Test`类的`main`函数中创建类`Sheep`的对象作为形参传递到`animalCall`方法中，因为`Sheep`类的对象不能转换为`Cow`类型，所以输出结果为“这个动物不是牛！”。
# 抽象（abstract）类
在面向对象的概念中，所有的对象都是通过类来描绘的，但是反过来，并不是所有的类都是用来描绘对象的，如果一个类中没有包含足够的信息来描绘一个具体的对象，那么这样的类称为抽象类。

抽象类的语法格式如下：
```java
<abstract> class <class_name> {
  <abstract><type><method_name>(parameter-iist);
}
```
其中，`abstract`表示该类或该方法是抽象的；`class_name`表示抽象类的名称；`method_name`表示抽象方法名称，`parameter-list`表示方法参数列表。

如果一个方法使用`abstract`来修饰，则说明该方法是抽象方法，抽象方法只有声明没有实现。需要注意的是`abstract`关键字只能用于普通方法，不能用于`static`方法或者构造方法中。

抽象方法的 3 个特征如下：
* 抽象方法没有方法体
* 抽象方法必须存在于抽象类中
* 子类重写父类时，必须重写父类所有的抽象方法

注意：在使用`abstract`关键字修饰抽象方法时不能使用`private`修饰，因为抽象方法必须被子类重写，而如果使用了`private`声明，则子类是无法重写的。

抽象类的定义和使用规则如下：
* 抽象类和抽象方法都要使用`abstract`关键字声明。
* 如果一个方法被声明为抽象的，那么这个类也必须声明为抽象的。而一个抽象类中，可以有`0~n`个抽象方法，以及`0~n`个具体方法。
* 抽象类不能实例化，也就是不能使用`new`关键字创建对象。

不同几何图形的面积计算公式是不同的，但是它们具有的特性是相同的，都具有长和宽这两个属性，也都具有面积计算的方法。那么可以定义一个抽象类，在该抽象类中含有两个属性和一个抽象方法`area()`，具体步骤如下。
```java
public abstract class Shape {
  public int width; // 几何图形的长
  public int height; // 几何图形的宽
  public Shape(int width, int height) {
    this.width = width;
    this.height = height;
  }
  public abstract double area(); // 定义抽象方法，计算面积
}

public class Square extends Shape {
  public Square(int width, int height) {
    super(width, height);
  }
  // 重写父类中的抽象方法，实现计算正方形面积的功能
  @Override
  public double area() {
    return width * height;
  }
}

public class Triangle extends Shape {
  public Triangle(int width, int height) {
    super(width, height);
  }
  // 重写父类中的抽象方法，实现计算三角形面积的功能
  @Override
  public double area() {
    return 0.5 * width * height;
  }
}

public class ShapeTest {
  public static void main(String[] args) {
    Square square = new Square(5, 4); // 创建正方形类对象
    System.out.println("正方形的面积为：" + square.area());
    Triangle triangle = new Triangle(2, 5); // 创建三角形类对象
    System.out.println("三角形的面积为：" + triangle.area());
  }
}
```
在该程序中，创建了 4 个类，分别为图形类`Shape`、正方形类`Square`、三角形类`Triangle`和测试类`ShapeTest`。其中图形类`Shape`是一个抽象类，创建了两个属性，分别为图形的长度和宽度，并通过构造方法`Shape()`给这两个属性赋值。

在`Shape`类的最后定义了一个抽象方法`area()`，用来计算图形的面积。在这里，`Shape`类只是定义了计算图形面积的方法，而对于如何计算并没有任何限制。也可以这样理解，抽象类`Shape`仅定义了子类的一般形式。

输出的结果如下：
```
正方形的面积为：20.0
三角形的面积为：5.0
```
# 接口
抽象类是从多个类中抽象出来的模板，如果将这种抽象进行的更彻底，则可以提炼出一种更加特殊的“抽象类”——接口（`Interface`）。它可以被理解为一种特殊的类，不同的是接口的成员没有执行体，是由全局常量和公共的抽象方法所组成。
## 定义接口
接口定义使用的关键字是`interface`：
```java
[public] interface interface_name [extends interface1_name[, interface2_name,…]] {
  // 接口体，其中可以包含定义常量和声明方法
  [public] [static] [final] type constant_name = value;    // 定义常量
  [public] [abstract] returnType method_name(parameter_list);    // 声明方法
}
```
对以上语法的说明如下：
* `public`表示接口的修饰符，当没有修饰符时，则使用默认的修饰符，此时该接口的访问权限仅局限于所属的包；
* `interface_name`表示接口的名称。接口名只要是合法的标识符即可。
* `extends`表示接口的继承关系；
* `interface1_name`表示要继承的接口名称；
* `constant_name`表示变量名称，一般是`static`和`final`型的；
* `returnType`表示方法的返回值类型；
* `parameter_list`表示参数列表，在接口中的方法是没有方法体的。

注意：一个接口可以有多个直接父接口，但接口只能继承接口，不能继承类。

接口对于其声明、变量和方法都做了许多限制，这些限制作为接口的特征归纳如下：
* 具有`public`访问控制符的接口，允许任何类使用；没有指定`public`的接口，其访问将局限于所属的包。
* 方法的声明不需要其他修饰符，在接口中声明的方法，将隐式地声明为公有的（`public`）和抽象的（`abstract`）。
* 在接口中声明的变量其实都是常量，接口中的变量声明，将隐式地声明为`public、static`和`final`，即常量，所以接口中定义的变量必须初始化。
* 接口没有构造方法，不能被实例化。
* 一个接口不能够实现另一个接口，但它可以继承多个其他接口。子接口可以对父接口的方法和常量进行重写。
```java
public interface StudentInterface extends PeopleInterface {
  // 接口 StudentInterface 继承 PeopleInterface
  int age = 25;    // 常量age重写父接口中的age常量
  void getInfo();    // 方法getInfo()重写父接口中的getInfo()方法
}
```

```java
public interface MyInterface {    // 接口myInterface
  String name;    // 不合法，变量name必须初始化
  int age = 20;    // 合法，等同于 public static final int age = 20;
  void getInfo();    // 方法声明，等同于 public abstract void getInfo();
}
```
## 实现接口
接口的主要用途就是被实现类实现，一个类可以实现一个或多个接口，继承使用`extends`关键字，实现接口则使用`implements`关键字。类实现接口的语法格式如下：
```java
<public> class <class_name> [extends superclass_name] [implements interface1_name[, interface2_name…]] {
  // 主体
}
```
对以上语法的说明如下：
* `public`：类的修饰符；
* `superclass_name`：需要继承的父类名称；
* `interface1_name`：要实现的接口名称。

实现接口需要注意以下几点：
* 实现接口与继承父类相似，一样可以获得所实现接口里定义的常量和方法。如果一个类需要实现多个接口，则多个接口之间以逗号分隔。
* 一个类可以继承一个父类，并同时实现多个接口，`implements`部分必须放在`extends`部分之后。
* 一个类实现了一个或多个接口之后，这个类必须完全实现这些接口里所定义的全部抽象方法（也就是重写这些抽象方法）；否则，该类将保留从父接口那里继承到的抽象方法，该类也必须定义成抽象类。

```java
public interface IMath {
  public int sum();    // 完成两个数的相加
  public int maxNum(int a,int b);    // 获取较大的数
}

public class MathClass implements IMath {
  private int num1;    // 第 1 个操作数
  private int num2;    // 第 2 个操作数
  public MathClass(int num1,int num2) {
    // 构造方法
    this.num1 = num1;
    this.num2 = num2;
  }
  // 实现接口中的求和方法
  public int sum() {
    return num1 + num2;
  }
  // 实现接口中的获取较大数的方法
  public int maxNum(int a,int b) {
    if(a >= b) {
      return a;
    } else {
      return b;
    }
  }
}
```
在实现类中，所有的方法都使用了`public`访问修饰符声明。无论何时实现一个由接口定义的方法，它都必须实现为`public`，因为接口中的所有成员都显式声明为`public`。

最后创建测试类`NumTest`，实例化接口的实现类`MathClass`，调用该类中的方法并输出结果。
```java
public class NumTest {
  public static void main(String[] args) {
    // 创建实现类的对象
    MathClass calc = new MathClass(100, 300);
    System.out.println("100 和 300 相加结果是：" + calc.sum());
    System.out.println("100 比较 300，哪个大：" + calc.maxNum(100, 300));
  }
}
```
程序运行结果如下所示。
```
100 和 300 相加结果是：400
100 比较 300，哪个大：300
```
在该程序中，首先定义了一个`IMath`的接口，在该接口中只声明了两个未实现的方法，这两个方法需要在接口的实现类中实现。在实现类`MathClass`中定义了两个私有的属性，并赋予两个属性初始值，同时创建了该类的构造方法。因为该类实现了`MathClass`接口，因此必须实现接口中的方法。在最后的测试类中，需要创建实现类对象，然后通过实现类对象调用实现类中的方法。
