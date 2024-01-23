

在类内部可定义成员变量和方法，且在类内部也可以定义另一个类。如果在类 Outer 的内部再定义一个类 Inner，此时类 Inner 就称为内部类（或称为嵌套类），而类 Outer 则称为外部类（或称为宿主类）。

内部类可以很好地实现隐藏，一般的非内部类是不允许有 private 与 protected 权限的，但内部类可以。内部类拥有外部类的所有元素的访问权限。

内部类可以分为：实例内部类、静态内部类和成员内部类，每种内部类都有它特定的一些特点。

在类 A 中定义类 B，那么类 B 就是内部类，也称为嵌套类，相对而言，类 A 就是外部类。如果有多层嵌套，例如类 A 中有内部类 B，而类 B 中还有内部类 C，那么通常将最外层的类称为顶层类（或者顶级类）。

内部类也可以分为多种形式，与变量非常类似。

{% asset_img 1.jpg %}

内部类的特点如下：
* 内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的`.class`文件，但是前面冠以外部类的类名和$符号。
* 内部类不能用普通的方式访问。内部类是外部类的一个成员，因此内部类可以自由地访问外部类的成员变量，无论是否为`private`的。
* 内部类声明成静态的，就不能随便访问外部类的成员变量，仍然是只能访问外部类的静态成员变量。

内部类的使用方法非常简单，例如下面的代码演示了内部类最简单的应用。
```java
public class Test {
  public class InnerClass {
    public int getSum(int x,int y) {
      return x + y;
    }
  }
  public static void main(String[] args) {
    Test.InnerClass ti = new Test().new InnerClass();
    int i = ti.getSum(2,3);
    System.out.println(i);    // 输出5
  }
}
```
有关内部类的说明有如下几点。
外部类只有两种访问级别：public 和默认；内部类则有 4 种访问级别：public、protected、 private 和默认。
在外部类中可以直接通过内部类的类名访问内部类。
```
InnerClass ic = new InnerClass();    // InnerClass为内部类的类名
```
在外部类以外的其他类中则需要通过内部类的完整类名访问内部类。
```
Test.InnerClass ti = newTest().new InnerClass();    // Test.innerClass是内部类的完整类名
```
内部类与外部类不能重名。

提示：内部类的很多访问规则可以参考变量和方法。另外使用内部类可以使程序结构变得紧凑，但是却在一定程度上破坏了 Java 面向对象的思想。
# 实例内部类
实例内部类是指没有用`static`修饰的内部类，有的地方也称为非静态内部类。
```java
public class Outer {
  class Inner {
    // 实例内部类
  }
}
```
上述示例中的 Inner 类就是实例内部类。实例内部类有如下特点。

1）在外部类的静态方法和外部类以外的其他类中，必须通过外部类的实例创建内部类的实例。
```java
public class Outer {
    class Inner1 {
    }
    Inner1 i = new Inner1(); // 不需要创建外部类实例
    public void method1() {
        Inner1 i = new Inner1(); // 不需要创建外部类实例
    }
    public static void method2() {
        Inner1 i = new Outer().new inner1(); // 需要创建外部类实例
    }
    class Inner2 {
        Inner1 i = new Inner1(); // 不需要创建外部类实例
    }
}
class OtherClass {
    Outer.Inner i = new Outer().new Inner(); // 需要创建外部类实例
}
```
2）在实例内部类中，可以访问外部类的所有成员。
```java
public class Outer {
    public int a = 100;
    static int b = 100;
    final int c = 100;
    private int d = 100;
    public String method1() {
        return "实例方法1";
    }
    public static String method2() {
        return "静态方法2";
    }
    class Inner {
        int a2 = a + 1; // 访问public的a
        int b2 = b + 1; // 访问static的b
        int c2 = c + 1; // 访问final的c
        int d2 = d + 1; // 访问private的d
        String str1 = method1(); // 访问实例方法method1
        String str2 = method2(); // 访问静态方法method2
    }
    public static void main(String[] args) {
        Inner i = new Outer().new Inner(); // 创建内部类实例
        System.out.println(i.a2); // 输出101
        System.out.println(i.b2); // 输出101
        System.out.println(i.c2); // 输出101
        System.out.println(i.d2); // 输出101
        System.out.println(i.str1); // 输出实例方法1
        System.out.println(i.str2); // 输出静态方法2
    }
}
```
提示：如果有多层嵌套，则内部类可以访问所有外部类的成员。

3）在外部类中不能直接访问内部类的成员，而必须通过内部类的实例去访问。如果类 A 包含内部类 B，类 B 中包含内部类 C，则在类 A 中不能直接访问类 C，而应该通过类 B 的实例去访问类 C。

4）外部类实例与内部类实例是一对多的关系，也就是说一个内部类实例只对应一个外部类实例，而一个外部类实例则可以对应多个内部类实例。

如果实例内部类 B 与外部类 A 包含有同名的成员 t，则在类 B 中 t 和 this.t 都表示 B 中的成员 t，而 A.this.t 表示 A 中的成员 t。
```java
public class Outer {
    int a = 10;
    class Inner {
        int a = 20;
        int b1 = a;
        int b2 = this.a;
        int b3 = Outer.this.a;
    }
    public static void main(String[] args) {
        Inner i = new Outer().new Inner();
        System.out.println(i.b1); // 输出20
        System.out.println(i.b2); // 输出20
        System.out.println(i.b3); // 输出10
    }
}
```
5）在实例内部类中不能定义 static 成员，除非同时使用 final 和 static 修饰。
# 静态内部类
静态内部类是指使用`static`修饰的内部类。
```java
public class Outer {
  static class Inner {
    // 静态内部类
  }
}
```
静态内部类有如下特点：
1. 在创建静态内部类的实例时，不需要创建外部类的实例。
```java
public class Outer {
  static class Inner {}
}
class OtherClass {
  Outer.Inner oi = new Outer.Inner();
}
```
1. 静态内部类中可以定义静态成员和实例成员。外部类以外的其他类需要通过完整的类名访问静态内部类中的静态成员，如果要访问静态内部类中的实例成员，则需要通过静态内部类的实例。
```java
public class Outer {
  static class Inner {
    int a = 0;    // 实例变量a
    static int b = 0;    // 静态变量 b
  }
}
class OtherClass {
  Outer.Inner oi = new Outer.Inner();
  int a2 = oi.a;    // 访问实例成员
  int b2 = Outer.Inner.b;    // 访问静态成员
}
```
1. 静态内部类可以直接访问外部类的静态成员，如果要访问外部类的实例成员，则需要通过外部类的实例去访问。
```java
public class Outer {
  int a = 0;    // 实例变量
  static int b = 0;    // 静态变量
  static class Inner {
    Outer o = new Outer;
    int a2 = o.a;    // 访问实例变量
    int b2 = b;    // 访问静态变量
  }
}
```
# 局部内部类
局部内部类是指在一个方法中定义的内部类。
```java
public class Test {
  public void method() {
    class Inner {
      // 局部内部类
    }
  }
}
```
局部内部类有如下特点：
1. 局部内部类与局部变量一样，不能使用访问控制修饰符（`public、private`和`protected`）和`static`修饰符修饰。
2. 局部内部类只在当前方法中有效。
```java
public class Test {
  Inner i = new Inner();    // 编译出错
  Test.Inner ti = new Test.Inner();    // 编译出错
  Test.Inner ti2 = new Test().new Inner();    // 编译出错
  public void method() {
    class Inner{}
    Inner i = new Inner();
  }
}
```
3. 局部内部类中不能定义`static`成员。
4. 局部内部类中还可以包含内部类，但是这些内部类也不能使用访问控制修饰符（`public、private`和`protected`）和`static`修饰符修饰。
5. 在局部内部类中可以访问外部类的所有成员。
6. 在局部内部类中只可以访问当前方法中 final 类型的参数与变量。如果方法中的成员与外部类中的成员同名，则可以使用`<OuterClassName>.this.<MemberName>`的形式访问外部类中的成员。
```java
public class Test {
  int a = 0;
  int d = 0;
  public void method() {
    int b = 0;
    final int c = 0;
    final int d = 10;
    class Inner {
      int a2 = a;    // 访问外部类中的成员
      // int b2 = b;    // 编译出错
      int c2 = c;    // 访问方法中的成员
      int d2 = d;    // 访问方法中的成员
      int d3 = Test.this.d;    //访问外部类中的成员
    }
    Inner i = new Inner();
    System.out.println(i.d2);    // 输出10
    System.out.println(i.d3);    // 输出0
  }
  public static void main(String[] args) {
    Test t = new Test();
    t.method();
  }
}
```
# 匿名内部类
匿名类是指没有类名的内部类，必须在创建时使用`new`语句来声明类。
```
new <类或接口>() {
    // 类的主体
};
```
这种形式的 new 语句声明一个新的匿名类，它对一个给定的类进行扩展，或者实现一个给定的接口。使用匿名类可使代码更加简洁、紧凑，模块化程度更高。

匿名类有两种实现方式：
* 继承一个类，重写其方法。
* 实现一个接口（可以是多个），实现其方法。

下面通过代码来说明。
```java
public class Out {
  void show() {
    System.out.println("调用 Out 类的 show() 方法");
  }
}
public class TestAnonymousInterClass {
  // 在这个方法中构造一个匿名内部类
  private void show() {
    Out anonyInter = new Out() {
      // 获取匿名内部类的实例
      void show() {
        System.out.println("调用匿名类中的 show() 方法");
      }
    };
    anonyInter.show();
  }
  public static void main(String[] args) {
    TestAnonymousInterClass test = new TestAnonymousInterClass();
    test.show();
  }
}
```
程序的输出结果如下：
```
调用匿名类中的 show() 方法
从输出结果可以看出，匿名内部类有自己的实现。
```
提示：匿名内部类实现一个接口的方式与实现一个类的方式相同。

匿名类有如下特点：
1）匿名类和局部内部类一样，可以访问外部类的所有成员。如果匿名类位于一个方法中，则匿名类只能访问方法中 final 类型的局部变量和参数。
```java
public static void main(String[] args) {
  int a = 10;
  final int b = 10;
  Out anonyInter = new Out() {
    void show() {
      // System.out.println("调用了匿名类的 show() 方法"+a);    // 编译出错
      System.out.println("调用了匿名类的 show() 方法"+b);    // 编译通过
    }
  };
  anonyInter.show();
}
```
从 Java 8 开始添加了 Effectively final 功能，在 Java 8 及以后的版本中代码第 6 行不会出现编译错误。

2）匿名类中允许使用非静态代码块进行成员初始化操作。
```java
Out anonyInter = new Out() {
  int i; {    // 非静态代码块
    i = 10;    //成员初始化
  }
  public void show() {
    System.out.println("调用了匿名类的 show() 方法"+i);
  }
};
```
3）匿名类的非静态代码块会在父类的构造方法之后被执行。