


# 类的封装
封装将类的某些信息隐藏在类内部，不允许外部程序直接访问，只能通过该类提供的方法来实现对隐藏信息的操作和访问。例如：一台计算机内部极其复杂，有主板、CPU、硬盘和内存， 而一般用户不需要了解它的内部细节，不需要知道主板的型号、CPU 主频、硬盘和内存的大小，于是计算机制造商将用机箱把计算机封装起来，对外提供了一些接口，如鼠标、键盘和显示器等，这样当用户使用计算机就非常方便。

封装的特点：
* 只能通过规定的方法访问数据。
* 隐藏类的实例细节，方便修改和实现。

实现封装的具体步骤如下：
* 修改属性的可见性来限制对属性的访问，一般设为`private`。
* 为每个属性创建一对赋值（`setter`）方法和取值（`getter`）方法，一般设为`public`，用于属性的读写。
* 在赋值和取值方法中，加入属性控制语句（对属性值的合法性进行判断）。

下面以一个员工类的封装为例介绍封装过程。一个员工的主要属性有姓名、年龄、联系电话和家庭住址。假设员工类为`Employee`，示例如下：
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

在`age`属性的`setAge()`方法中，首先对用户传递过来的参数`age`进行判断，如果`age`的值不在 18 到 40 之间，则将`Employee`类的`age`属性值设置为 20，否则为传递过来的参数值。

编写测试类`EmployeeTest`，在该类的`main()`方法中调用`Employee`属性的`setXxx()`方法对其相应的属性进行赋值，并调用`getXxx()`方法访问属性，代码如下：
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
运行该示例，输出结果如下：
```
姓名：王丽丽
年龄：35
电话：13653835964
家庭住址：河北省石家庄市
```
通过封装，实现了对属性的数据访问限制，满足了年龄的条件。在属性的赋值方法中可以对属性进行限制操作，从而给类中的属性赋予合理的值，并通过取值方法获取类中属性的值（也可以直接调用类中的属性名称来获取属性值）。
# 继承（extends）简明教程