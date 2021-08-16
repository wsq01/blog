


# 一维数组
当数组中每个元素都只带有一个下标时，这种数组就是“一维数组”。一维数组实质上是一组相同类型数据的线性集合，是数组中最简单的一种数组。

数组是引用数据类型，引用数据类型在使用之前一定要做两件事情：声明和初始化。所以本文将重点介绍一维数组的创建、初始化和使用。
## 创建一维数组
为了在程序中使用一个数组，必须声明一个引用该数组的变量，并指明整个变量可以引用的数组类型。声明一维数组的语法格式为：
```
type[] arrayName;    // 数据类型[] 数组名;
// 或者
type arrayName[];    // 数据类型 数组名[];
```
对于以上两种语法格式而言，Java 更推荐采用第一种声明格式，因为第一种格式不仅具有更好的语意，而且具有更好的可读性。对于第一种格式type[] arrayName，很容易理解这是定义一个变量，其中变量名是 arrayName，而变量类型是 type[]。

以上两种格式都可以声明一个数组，其中的数据类型既可以是基本数据类型，也可以是引用数据类型。数组名可以是任意合法的变量名。声明数组就是要告诉计算机该数组中数据的类型是什么。
```java
int[] score;    // 存储学生的成绩，类型为整型
double[] price;    // 存储商品的价格，类型为浮点型
String[] name;    // 存储商品名称，类型为字符串型
```
在声明数组时不需要规定数组的长度，例如：
```
int score[10];    // 这是错误的
```
## 分配空间
声明了数组，只是得到了一个存放数组的变量，并没有为数组元素分配内存空间，不能使用。因此要为数组分配内存空间，这样数组的每一个元素才有一个空间进行存储。

简单地说，分配空间就是要告诉计算机在内存中为它分配几个连续的位置来存储数据。在 Java 中可以使用 new 关键字来给数组分配空间。分配空间的语法格式如下：
```java
arrayName = new type[size];    // 数组名 = new 数据类型[数组长度];
```
其中，数组长度就是数组中能存放的元素个数，显然应该为大于 0 的整数，例如：
```
score = new int[10];
price = new double[30];
name = new String[20];
```
这里的 score 是已经声明过的 int[] 类型的变量，当然也可以在声明数组时就给它分配空间，语法格式如下：
```java
type[] arrayName = new type[size];    // 数据类型[] 数组名 = new 数据类型[数组长度];
```
例如，声明并分配一个长度为 5 的 int 类型数组 arr，代码如下：
```
int[] arr = new int[5];
```
执行后 arr 数组在内存中的格式如图所示。

{% asset_img 1.jpg 一维数组的内存格式 %}

在图中 arr 为数组名称，方括号“[]”中的值为数组的下标。数组通过下标来区分数组中不同的元素，并且下标是从 0 开始的。因此这里包含 5 个元素的 arr 数组最大下标为 4。

> 注意：一旦声明了数组的大小，就不能再修改。这里的数组长度也是必需的，不能少。

尽管数组可以存储一组基本数据类型的元素，但是数组整体属于引用数据类型。当声明一个数组变量时，其实是创建了一个类型为“数据类型[]”（如 int[]、double[]、String[]）的数组对象，它具有表 1 所示的方法和属性。

| 名称 | 返回值 |
| :--: | :--: |
| clone() | Object |
| equals(Object obj) | boolean |
| getClass() | Class<?> |
| hashCode() | int |
| notify() | void |
| notify All() | void |
| toString() | String |
| wait() | void |
| wait(long timeout) | void |
| wait(long timeout,int nanos) | void |
| length | int |

## 初始化一维数组
Java 语言中数组必须先初始化，然后才可以使用。所谓初始化，就是为数组的数组元素分配内存空间，并为每个数组元素赋初始值。

能不能只分配内存空间，不赋初始值呢？

不行，一旦为数组的每个数组元素分配了内存空间，每个内存空间里存储的内容就是该数组元素的值，即使这个内存空间存储的内容为空，这个空也是一个值（`null`）。不管以哪种方式来初始化数组，只要为数组元素分配了内存空间，数组元素就具有了初始值。初始值的获得有两种形式，一种由系统自动分配，另一种由程序员指定。

数组在初始化数组的同时，可以指定数组的大小，也可以分别初始化数组中的每一个元素。在 Java 语言中，初始化数组有以下 3 种方式。
### 1.使用 new 指定数组大小后进行初始化
使用 new 关键字创建数组，在创建时指定数组的大小。语法如下：
```java
type[] arrayName = new int[size];
```
创建数组之后，元素的值并不确定，需要为每一个数组的元素进行赋值，其下标从 0 开始。

创建包含 5 个元素的`int`类型的数组，然后分别将元素的值设置为 1、2、3、5 和 8。代码如下：
```java
int[] number = new int[5];
number[0] = 1;
number[1] = 2;
number[2] = 3;
number[3] = 5;
number[4] = 8;
```
如果只指定了数组的长度，那么系统将负责为这些数组元素分配初始值。指定初始值时，系统按如下规则分配初始值。
* 数组元素的类型是基本类型中的整数类型（`byte、short、int`和`long`），则数组元素的值是 0。
* 数组元素的类型是基本类型中的浮点类型（`float、double`），则数组元素的值是 0.0。
* 数组元素的类型是基本类型中的字符类型（`char`），则数组元素的值是`\u0000`。
* 数组元素的类型是基本类型中的布尔类型（`boolean`），则数组元素的值是`false`。
* 数组元素的类型是引用类型（类、接口和数组），则数组元素的值是`null`。

### 2.使用 new 指定数组元素的值
使用上述方式初始化数组时，只有在为元素赋值时才确定值。可以不使用上述方式，而是在初始化时就已经确定值。语法如下：
```java
type[] arrayName = new type[]{值 1,值 2,值 3,值 4,• • •,值 n};
```
使用`new`直接指定数组元素的值。代码如下：
```java
int[] number = new int[]{1, 2, 3, 5, 8};
```

注意：不要在进行数组初始化时，既指定数组的长度，也为每个数组元素分配初始值，这样会造成代码错误。例如下面代码：
```java
int[] number = new int [5] {1,2,3,4,​5};
```
### 3.直接指定数组元素的值
在上述两种方式的语法中，`type`可以省略，如果已经声明数组变量，那么直接使用这两种方式进行初始化。如果不想使用上述两种方式，那么可以不使用`new`直接指定数组元素的值。语法如下：
```java
type[] arrayName = {值 1,值 2,值 3,...,值 n};
```
```java
int[] number = {1,2,3,5,8};
```
使用这种方式时，数组的声明和初始化操作要同步，即不能省略数组变量的类型。如下的代码就是错误的：
```java
int[] number;
number = {1,2,3,5,8};
```
## 获取单个元素
获取单个元素是指获取数组中的一个元素。获取单个元素的方法非常简单，指定元素所在数组的下标即可。语法如下：
```java
arrayName[index];
```
其中，`arrayName`表示数组变量，`index`表示下标，下标为 0 表示获取第一个元素，下标为`array.length-1`表示获取最后一个元素。当指定的下标值超出数组的总长度时，会拋出`ArraylndexOutOfBoundsException`异常。

获取`number`数组中的第一个元素、最后一个元素和第六个元素，并将元素的值输出。
```java
int[] number = {1,2,3,5,8};
System.out.println("获取第一个元素："+number[0]);
System.out.println("获取最后一个元素："+number[number.length-1]);
System.out.println("获取第6个元素："+number[5]);
//输出结果如下所示：
//获取第一个元素：1
//获取最后一个元素：8
//java.lang.ArrayIndexOutOfBoundsException: 5
```
编写一个 Java 程序，使用数组存放录入的 5 件商品价格，然后使用下标访问第 3 个元素的值。
```java
import java.util.Scanner;
public class Test06 {
  public static void main(String[] args) {
    int[] prices = new int[5]; // 声明数组并分配空间
    Scanner input = new Scanner(System.in); // 接收用户从控制台输入的数据
    for (int i = 0; i < prices.length; i++) {
      System.out.println("请输入第" + (i + 1) + "件商品的价格：");
      prices[i] = input.nextInt(); // 接收用户从控制台输入的数据
    }
    System.out.println("第 3 件商品的价格为：" + prices[2]);
  }
}
```
## 获取全部元素
利用 for 循环语句遍历`number`数组中的全部元素，并将元素的值输出。
```java
int[] number = {1,2,3,5,8};
for (int i=0;i<number.length;i++) {
    System.out.println("第"+(i+1)+"个元素的值是："+number[i]);
}
```
除了使用`for`语句，还可以使用`foreach`遍历数组中的元素，并将元素的值输出。
```java
for(int val:number) {
  System.out.print("元素的值依次是："+val+"\t");
}
```
# 二维数组
创建二维数组
在 Java 中二维数组被看作数组的数组，即二维数组为一个特殊的一维数组，其每个元素又是一个一维数组。Java 并不直接支持二维数组，但是允许定义数组元素是一维数组的一维数组，以达到同样的效果。声明二维数组的语法如下：
```java
type arrayName[][];    // 数据类型 数组名[][];
// 或
type[][] arrayName;    // 数据类型[][] 数组名;
```
其中，`type`表示二维数组的类型，`arrayName`表示数组名称，第一个中括号表示行，第二个中括号表示列。
```java
int[][] age;
char[][] sex;
```
## 初始化二维数组
二维数组可以初始化，和一维数组一样，可以通过 3 种方式来指定元素的初始值。这 3 种方式的语法如下：
```java
type[][] arrayName = new type[][]{值 1,值 2,值 3,…,值 n};    // 在定义时初始化
type[][] arrayName = new type[size1][size2];    // 给定空间，在赋值
type[][] arrayName = new type[size][];    // 数组第二维长度为空，可变化
```
使用第一种方式声明`int`类型的二维数组，然后初始化该二维数组。
```java
int[][] temp = new int[][]{{1,2},{3,4}};
```
上述代码创建了一个二行二列的二维数组`temp`，并对数组中的元素进行了初始化。

{% asset_img 2.jpg 二维数组内存结构 %}

使用第二种方式声明 int 类型的二维数组，然后初始化该二维数组。
```java
int[][] temp = new int[2][2];
```
使用第三种方式声明 int 类型的二维数组，并且初始化数组。
```java
int[][] temp = new int[2][];
```
## 获取单个元素
当需要获取二维数组中元素的值时，也可以使用下标来表示。语法如下：
```
arrayName[i-1][j-1];
```
其中，`arrayName`表示数组名称，`i`表示数组的行数，`j`表示数组的列数。例如，要获取第二行第二列元素的值，应该使用`temp[1][1]`来表示。这

通过下标获取`class_score`数组中第二行第二列元素的值与第四行第一列元素的值。
```java
public static void main(String[] args) {
  double[][] class_score = {{10.0,99,99},{100,98,97},{100,100,99.5},{99.5,99,98.5}};
  System.out.println("第二行第二列元素的值："+class_score[1][1]);
  System.out.println("第四行第一列元素的值："+class_score[3][0]);
}
```
执行上述代码，输出结果如下：
```
第二行第二列元素的值：98.0
第四行第一列元素的值：99.5
```
## 获取全部元素
在一维数组中直接使用数组的`length`属性获取数组元素的个数。而在二维数组中，直接使用`length`属性获取的是数组的行数，在指定的索引后加上`length`（如`array[0].length`）表示的是该行拥有多少个元素，即列数。

如果要获取二维数组中的全部元素，最简单、最常用的办法就是使用`for`语句。在一维数组全部输出时，我们使用一层`for`循环，而二维数组要想全部输出，则使用嵌套`for`循环（2 层`for`循环）。

使用`for`循环语句遍历`double`类型的`class_score`数组的元素，并输出每一行每一列元素的值。代码如下：
```java
public static void main(String[] args) {
  double[][] class_score = { { 100, 99, 99 }, { 100, 98, 97 }, { 100, 100, 99.5 }, { 99.5, 99, 98.5 } };
  for (int i = 0; i < class_score.length; i++) { // 遍历行
    for (int j = 0; j < class_score[i].length; j++) {
      System.out.println("class_score[" + i + "][" + j + "]=" + class_score[i][j]);
    }
  }
}
```
上述代码使用嵌套`for`循环语句输出二维数组。在输出二维数组时，第一个 for 循环语句表示以行进行循环，第二个 for 循环语句表示以列进行循环，这样就实现了获取二维数组中每个元素的值的功能。

假设有一个矩阵为 5 行 5 列，该矩阵是由程序随机产生的 10 以内数字排列而成。下面使用二维数组来创建该矩阵，代码如下：
```
public class Test11 {
    public static void main(String[] args) {
        // 创建一个二维矩阵
        int[][] matrix = new int[5][5];
        // 随机分配值
        for (int i = 0; i < matrix.length; i++) {
            for (int j = 0; j < matrix[i].length; j++) {
                matrix[i][j] = (int) (Math.random() * 10);
            }
        }
        System.out.println("下面是程序生成的矩阵\n");
        // 遍历二维矩阵并输出
        for (int k = 0; k < matrix.length; k++) {
            for (int g = 0; g < matrix[k].length; g++) {
                System.out.print(matrix[k][g] + "");
            }
            System.out.println();
        }
    }
}
```
在该程序中，首先定义了一个二维数组，然后使用两个嵌套的 for 循环向二维数组中的每个元素赋值。其中，Math.random() 方法返回的是一个 double 类型的数值，数值为 0.6、0.9 等，因此乘以 10 之后为 10 以内的整数。最后又使用了两个嵌套的 for 循环遍历二维数组，输出二维数组中的值，从而产生矩阵。

运行该程序的结果如下所示。
下面是程序生成的矩阵
```
78148
69230
43823
75663
05688
```
for each 循环语句不能自动处理二维数组的每一个元素。它是按照行， 也就是一维数组处理的。要想访问二维教组 a 的所有元素， 需要使用两个嵌套的循环， 如下所示：
```
for (double[] row : a) {
    for (double value : row) {
        ......
    }
}
```
使用 for each 循环语句输出，代码如下所示：
```
public static void main(String[] args) {
    double[][] class_score = { { 100, 99, 99 }, { 100, 98, 97 }, { 100, 100, 99.5 }, { 99.5, 99, 98.5 } };
    for (double[] row : class_score) {
        for (double value : row) {
            System.out.println(value);
        }
    }
}
```
输出结果为：
```
100.0
99.0
99.0
100.0
98.0
97.0
100.0
100.0
99.5
99.5
99.0
98.5
```
提示：要想快速地打印一个二维数组的数据元素列表，可以调用：
```
System.out.println(Arrays.deepToString(arrayName));
```
代码如下：
```
System.out.println(Arrays.deepToString(class_score));
```
输出格式为：
```
[[100.0, 99.0, 99.0], [100.0, 98.0, 97.0], [100.0, 100.0, 99.5], [99.5, 99.0, 98.5]]
```
# 多维数组
除了一维数组和二维数组外，Java 中还支持更多维的数组，如三维数组、四维数组和五维数组等，它们都属于多维数组。想要提高数组的维数，只要在声明数组时将索引与中括号再加一组即可，所以三维数组的声明为`int score[][][]`，而四维数组为`int score[][][][]`，以此类推。

三维数组有三个层次，可以将三维数组理解为一个一维数组，其内容的每个元素都是二维数组。依此类推，可以获取任意维数的数组。
# 不规则数组
规则的 4×3 二维数组有 12 个元素，而不规则数组就不一定了。如下代码静态初始化了一个不规则数组。
```java
int intArray[][] = {{1,2}, {11}, {21,22,23}, {31,32,33}};
```
动态初始化不规则数组比较麻烦，不能使用`new int[4][3]`语句，而是先初始化高维数组，然后再分别逐个初始化低维数组。代码如下：
```
int intArray[][] = new int[4][]; //先初始化高维数组为4
// 逐一初始化低维数组
intArray[0] = new int[2];
intArray[1] = new int[1];
intArray[2] = new int[3];
intArray[3] = new int[3];
```
从上述代码初始化数组完成之后，不是有 12 个元素而是 9 个元素，它们的下标索引如图 2 所示，可见其中下标 [0][2]、[1][1] 和 [1][2] 是不存在的，如果试图访问它们则会抛出下标越界异常。
不规则数组访问
图 2  不规则数组访问

提示：下标越界异常（ArrayIndexOutOfBoundsException）是试图访问不存在的下标时引发的。例如一个一维 array 数组如果有 10 个元素，那么表达式 array[10] 就会发生下标越界异常，这是因为数组下标是从 0 开始的，最后一个元素下标是数组长度减 1，所以 array[10] 访问的元素是不存在的。
```java 
import java.util.Arrays;
public class HelloWorld {
    public static void main(String[] args) {
        int intArray[][] = new int[4][]; // 先初始化高维数组为4
        // 逐一初始化低维数组
        intArray[0] = new int[2];
        intArray[1] = new int[1];
        intArray[2] = new int[3];
        intArray[3] = new int[3];
        // for循环遍历
        for (int i = 0; i < intArray.length; i++) {
            for (int j = 0; j < intArray[i].length; j++) {
                intArray[i][j] = i + j;
            }
        }
        // for-each循环遍历
        for (int[] row : intArray) {
            for (int column : row) {
                System.out.print(column);
                // 在元素之间添加制表符，
                System.out.print('\t');
            }
            // 一行元素打印完成后换行
            System.out.println();
        }
        System.out.println(intArray[0][2]); // 发生运行期错误
    }
}
```
不规则数组访问和遍历可以使用 for 和 for-each 循环，但要注意下标越界异常发生。

上述代码第 18 行和第 19 行采用 for-each 循环遍历不规则数组，其中代码第 18 行 for-each 循环取出的数据是 int 数组，所以 row 类型是 int[]。代码第 19 行 for-each 循环取出的数据是 int 数据，所以 column 的类型 int。另外，注意代码第 27 行试图访问 intArray[0][2]元素，由于 [0][2] 不存在所以会发生下标越界异常。
