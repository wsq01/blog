---
title: Java 包（package）
date: 2020-09-18 11:23:14
tags: [java]
categories: java
---


包允许将类组合成较小的单元（类似文件夹），它基本上隐藏了类，并避免了名称上的冲突。包允许在更广泛的范围内保护类、数据和方法。你可以在包内定义类，而在包外的代码不能访问该类。这使你的类相互之间有隐私，但不被其他世界所知。

包的 3 个作用如下：
* 区分相同名称的类。
* 能够较好地管理大量的类。
* 控制访问范围。

# 包定义
Java 中使用`package`语句定义包，`package`语句应该放在源文件的第一行，在每个源文件中只能有一个包定义语句，并且`package`语句适用于所有类型（类、接口、枚举和注释）的文件。定义包语法格式如下：
```
package 包名;
```
Java 包的命名规则如下：
* 包名全部由小写字母（多个单词也全部小写）。
* 如果包名包含多个层次，每个层次用“.”分割。
* 包名一般由倒置的域名开头，比如`com.baidu`，不要有`www`。
* 自定义包不能`java`开头。

注意：如果在源文件中没有定义包，那么类、接口、枚举和注释类型文件将会被放进一个无名的包中，也称为默认包。在实际企业开发中，通常不会把类定义在默认包下。
# 包导入
如果使用不同包中的其它类，需要使用该类的全名（包名+类名）。
```java
example.Test test = new example.Test();
```
其中，`example`是包名，`Test`是包中的类名，`test`是类的对象。

为了简化编程，Java 引入了`import`关键字，`import`可以向某个 Java 文件中导入指定包层次下的某个类或全部类。`import`语句位于`package`语句之后，类定义之前。一个 Java 源文件只能包含一个`package`语句，但可以包含多个`import`语句。

使用`import`导入单个类的语法格式如下：
```
import 包名+类名;
```
使用`import`语句导入指定包下全部类的用法按如下：
```
import example.*;
```
上面`import`语句中的星号（*）只能代表类，不能代表包，表明导入`example`包下的所有类。

提示：使用星号（*）可能会增加编译时间，特别是引入多个大包时，所以明确的导入你想要用到的类是一个好方法，需要注意的是使用星号对运行时间和类的大小没有影响。

通过使用`import`语句可以简化编程，但`import`语句并不是必需的，如果在类里使用其它类的全名，可以不使用`import`语句。

Java 默认为所有源文件导入`java.lang`包下的所有类，因此前面在 Java 程序中使用`String、System`类时都无须使用`import`语句来导入这些类。但对于`Arrays`类，其位于`java.util`包下，则必须使用`import`语句来导入该类。

在一些极端的情况下，`import`语句也帮不了我们，此时只能在源文件中使用类全名。例如，需要在程序中使用`java.sql`包下的类，也需要使用`java.util`包下的类，则可以使用如下两行`import`语句：
```java
import java.util.*;
import java.sql.*;
```
如果接下来在程序中需要使用`Date`类，则会引起如下编译错误：
```
Test.java:25:对Date的引用不明确，
java.sql中的类java.sql.Date和java.util中的类java.util.Date都匹配
```
上面错误提示：在`Test.java`文件的第 25 行使用了`Date`类，而`import`语句导入的`java.sql`和`java.util`包下都包含了`Date`类，系统不知道使用哪个包下的`Date`类。在这种情况下，如果需要指定包下的`Date`类，则只能使用该类的全名：
```java
// 为了让引用更加明确，即使使用了 import 语句，也还是需要使用类的全名
java.sql.Date d = new java.sql.Date();
```
## 系统包
常用的系统包：

| 包 | 说明 |
| :--: | :--: |
| java.lang | Java 的核心类库，包含运行 Java 程序必不可少的系统类，如基本数据类型、基本数学函数、字符串处理、异常处理和线程类等，系统默认加载这个包 |
| java.io | Java 语言的标准输入/输出类库，如基本输入/输出流、文件输入/输出、过滤输入/输出流等 |
| java.util | 包含如处理时间的 Date 类，处理动态数组的 Vector 类，以及 Stack 和 HashTable 类 |
| java.awt | 构建图形用户界面（GUI）的类库，低级绘图操作 Graphics 类、图形界面组件和布局管理（如 Checkbox 类、Container 类、LayoutManger 接口等），以及用户界面交互控制和事件响应（如 Event 类）|
| java.awt.image | 处理和操纵来自网上的图片的 Java 工具类库 |
| java.wat.peer | 很少在程序中直接用到，使得同一个 Java 程序在不同的软硬件平台上运行 |
| java.net | 实现网络功能的类库有 Socket 类、ServerSocket 类 |
| java.lang.reflect | 提供用于反射对象的工具 |
| java.util.zip | 实现文件压缩功能 |
| java.awt.datatransfer | 处理数据传输的工具类，包括剪贴板、字符串发送器等 |
| java.sql | 实现 JDBC 的类库 |
| java.rmi | 提供远程连接与载入的支持 |
| java. security | 提供安全性方面的有关支持 |

# 使用自定义包
1. 创建一个名为`com.dao`的包。
2. 向`com.dao`包中添加一个`Student`类，该类包含一个返回`String`类型数组的`GetAll()`方法。
```java
package com.dao;
public class Student {
  public static String[] GetAll() {
    String[] namelist = {"李潘","邓国良","任玲玲","许月月","欧阳娜","赵晓慧"};
    return namelist;
  }
}
```
3. 创建`com.test`包，在该包里创建带`main()`方法的`Test`类。
4. 在`main()`方法中遍历`Student`类的`GetAll()`方法中的元素内容，在遍历内容之前，使用`import`引入`com.dao`整个包。
```java
package com.test;
import com.dao.Student;
public class Test {
  public static void main(String[] args) {
    System.out.println("学生信息如下：");
    for(String str:Student.GetAll()) {
      System.out.println(str);
    }
  }
}
```
输出结果如下：
```
学生信息如下：
李潘
邓国良
任玲玲
许月月
欧阳娜
赵晓慧
```