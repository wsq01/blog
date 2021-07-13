

# 继承
Java中的继承是一种机制，表示为一个对象获取父对象的所有属性和行为。

在Java中继承是：可以创建基于现有类构建新的类。 当您从现有类继承时，就可以重复使用父类的方法和字段，也可以在继承的新类中添加新的方法和字段。
## 为什么在java中使用继承？
对于方法覆盖(因此可以实现运行时的多态性)，提高代码可重用性。在Java中，子类可继承父类中的方法，而不需要重新编写相同的方法。但有时子类并不想原封不动地继承父类的方法，而是想作一定的修改，这就需要采用方法的重写(覆盖)。
```java
// Java继承的语法
class Subclass-name extends Superclass-name  
{  
  //methods and fields  
}
```
extends关键字表示正在从现有类派生创建的新类。继承的类称为父类或超类，新类称为子或子类。

java继承类型在类的基础上，在java中可以有三种类型的继承：单一，多级和分层。在Java编程中，仅能通过接口支持多重和混合继承。

{% asset_img 1.png %}

在java中的类不支持多继承。当一个类扩展多个类，即被称为多重继承。例如：

{% asset_img 2.png %}

## 示例
### 单一继承示例
```java
class Animal {
  void eat() {
    System.out.println("eating...");
  }
}

class Dog extends Animal {
  void bark() {
    System.out.println("barking...");
  }
}

class TestInheritance {
  public static void main(String args[]) {
    Dog d = new Dog();
    d.bark();
    d.eat();
  }
}
// 执行上面代码得到以下结果 - 
// barking...
// eating...
```
### 多级继承示例
```java
class Animal {
  void eat() {
    System.out.println("eating...");
  }
}

class Dog extends Animal {
  void bark() {
    System.out.println("barking...");
  }
}

class BabyDog extends Dog {
  void weep() {
    System.out.println("weeping...");
  }
}

class TestInheritance2 {
  public static void main(String args[]) {
    BabyDog d = new BabyDog();
    d.weep();
    d.bark();
    d.eat();
  }
}
// 执行上面代码得到以下结果 - 
// weeping...
// barking...
// eating...
```
## 多级继承示例
```java
class Animal {
  void eat() {
    System.out.println("eating...");
  }
}

class Dog extends Animal {
  void bark() {
      System.out.println("barking...");
  }
}

class Cat extends Animal {
  void meow() {
    System.out.println("meowing...");
  }
}

class TestInheritance3 {
  public static void main(String args[]) {
    Cat c = new Cat();
    c.meow();
    c.eat();
    // c.bark();//C.T.Error
  }
}
// 执行上面代码得到以下结果 - 
// meowing...
// eating...
```
## 问题：为什么在Java中不支持多重继承？
为了降低复杂性并简化语言，Java 中不支持多重继承。想象一个：A，B 和 C 是三个类。 C 类继承 A 和 B 类。 如果A和B类有相同的方法，并且从子类对象调用它，A 或 B 类的调用方法会有歧义。
因为编译时错误比运行时错误好，如果继承2个类，java会在编译时报告错误。 所以无论子类中是否有相同的方法，都会有报告编译时错误。
```java
class A {
  void msg() {
    System.out.println("Hello");
  }
}

class B {
  void msg() {
    System.out.println("Welcome");
  }
}

class C extends A,B {//suppose if it were  
 public static void main(String args[]) {
    C obj = new C();
    obj.msg();// Now which msg() method would be invoked?
  }
}
```
# 聚合
如果一个类有一个类的实体引用(类中的类)，则它称为聚合。 聚合表示`HAS-A`关系。考虑有一种情况，`Employee`对象包含许多信息，例如：`id`，`name`，`emailId`等。它包含另一个类对象：`address`，其包含它自己的信息，例如：城市，州，国家，邮政编码等，如下所示。
```java
class Employee {  
  int id;  
  String name;  
  Address address;//Address is a class  
  ...  
}
```
在这种情况下，`Employee`有一个实体引用地址(`Address`)，因此关系是：`Employee HAS-A Address`。
## 聚合的简单示例

{% asset_img 3.jpg %}

在这个例子中，在`Circle`类中创建了`Operation`类的引用。
```java
class Operation {
  int square(int n) {
    return n * n;
  }
}

class Circle {
  Operation op;// aggregation
  double pi = 3.14;

  double area(int radius) {
    op = new Operation();
    int rsquare = op.square(radius);
    return pi * rsquare;
  }

  public static void main(String args[]) {
    Circle c = new Circle();
    double result = c.area(5);
    System.out.println(result);
  }
}
// 执行上面代码，得到以下结果 - 
// 78.5
```
## 何时使用聚合？
当没有`is-a`关系时，通过聚合也能最好地实现代码重用。只有在所涉及的对象的整个生命周期内维持关系为`is-a`时，才应使用继承; 否则，聚合是最好的选择。
## 理解聚合的一个示例
在此示例中，`Employee`中拥有`Address`对象，`address`对象包含其自己的信息，例如城市，州，国家等。在这种情况下，关系是员工(`Employee`)`HAS-A`地址(`Address`)。
```java
// Address.java
public class Address {
  String city, province;

  public Address(String city, String province) {
    this.city = city;
    this.province = province;
  }
}
// Emp.java
public class Emp {
  int id;
  String name;
  Address address;

  public Emp(int id, String name, Address address) {
    this.id = id;
    this.name = name;
    this.address = address;
  }

  void display() {
    System.out.println(id + " " + name);
    System.out.println(address.city + " " + address.province);
  }

  public static void main(String[] args) {
    Address address1 = new Address("广州", "广东");
    Address address2 = new Address("海口", "海南");

    Emp e = new Emp(111, "Wang", address1);
    Emp e2 = new Emp(112, "Zhang", address2);

    e.display();
    e2.display();
  }
}
// 执行上面代码，得到以下结果 - 
//111 Wang
//广州 广东
//112 Zhang
//海口 海南
```
# super关键字
java中的`super`关键字是一个引用变量，用于引用直接父类对象。

每当创建子类的实例时，父类的实例被隐式创建，由`super`关键字引用变量引用。

java `super`关键字的用法如下：
* `super`可以用来引用直接父类的实例变量。
* `super`可以用来调用直接父类方法。
* `super()`可以用于调用直接父类构造函数。

## 1. super用于引用直接父类实例变量
可以使用`super`关键字来访问父类的数据成员或字段。如果父类和子类具有相同的字段，则使用`super`来指定为父类数据成员或字段。
```java
class Animal {
  String color = "white";
}

class Dog extends Animal {
  String color = "black";

  void printColor() {
    System.out.println(color);// prints color of Dog class
    System.out.println(super.color);// prints color of Animal class
  }
}

class TestSuper1 {
  public static void main(String args[]) {
    Dog d = new Dog();
    d.printColor();
  }
}
// 执行上面代码，输出结果如下 -
//black
//white
```
在上面的例子中，`Animal`和`Dog`都有一个共同的属性：`color`。如果我们打印`color`属性，它将默认打印当前类的颜色。要访问父属性，需要使用`super`关键字指定。
## 2. 通过 super 来调用父类方法
`super`关键字也可以用于调用父类方法。如果子类包含与父类相同的方法，则应使用`super`关键字指定父类的方法。换句话说，如果方法被覆盖就可以使用`super`关键字来指定父类方法。
```java
class Animal {
  void eat() {
    System.out.println("eating...");
  }
}

class Dog extends Animal {
  void eat() {
    System.out.println("eating bread...");
  }

  void bark() {
    System.out.println("barking...");
  }

  void work() {
    super.eat();
    bark();
  }
}

class TestSuper2 {
  public static void main(String args[]) {
    Dog d = new Dog();
    d.work();
  }
}
//执行上面代码，输出结果如下 -
//eating...
//barking...
```
在上面的例子中，`Animal`和`Dog`两个类都有`eat()`方法，如果要调用`Dog`类中的`eat()`方法，它将默认调用`Dog`类的`eat()`方法，因为当前类的优先级比父类的高。所以要调用父类方法，需要使用`super`关键字指定。
## 3. 使用 super 来调用父类构造函数
`super`关键字也可以用于调用父类构造函数。
```java
class Animal {
  Animal() {
    System.out.println("animal is created");
  }
}

class Dog extends Animal {
  Dog() {
    super();
    System.out.println("dog is created");
  }
}

class TestSuper3 {
  public static void main(String args[]) {
    Dog d = new Dog();
  }
}
```
注意：如果没有使用`super()`或`this()`，则`super()`在每个类构造函数中由编译器自动添加。

如果没有构造函数，编译器会自动提供默认构造函数。 但是，它还添加了`super()`作为第一个语句。
```java
class Animal {
  Animal() {
    System.out.println("animal is created");
  }
}

class Dog extends Animal {
  Dog() {
    System.out.println("dog is created");
  }
}

class TestSuper4 {
  public static void main(String args[]) {
    Dog d = new Dog();
  }
}
//执行上面代码，输出结果如下 -
//animal is created
//dog is created
```
## super实际使用示例
在这里，`Emp`类继承了`Person`类，所以`Person`的所有属性都将默认继承到`Emp`。要初始化所有的属性，可使用子类的父类构造函数。这样，我们重用了父类的构造函数。
```java
class Person {
  int id;
  String name;

  Person(int id, String name) {
    this.id = id;
    this.name = name;
  }
}

class Emp extends Person {
  float salary;

  Emp(int id, String name, float salary) {
    super(id, name);// reusing parent constructor
    this.salary = salary;
  }

  void display() {
    System.out.println(id + " " + name + " " + salary);
  }
}

class TestSuper5 {
  public static void main(String[] args) {
    Emp e1 = new Emp(1, "ankit", 45000f);
    e1.display();
  }
}
// 执行上面代码，输出结果如下 -
// 1 ankit 45000
```