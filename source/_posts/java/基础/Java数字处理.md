---
title: Java 数字处理
date: 2020-09-05 17:16:38
tags: [java]
categories: java
---


# Math类的常用方法
`Math`类位于`java.lang`包，它的构造方法是`private`的，因此无法创建`Math`类的对象，并且`Math`类中的所有方法都是类方法，可以直接通过类名来调用它们。
## 静态常量
`Math`类中包含`E`和`PI`两个静态常量，它们的值分别等于 `e`（自然对数）和`π`（圆周率）。
```java
System.out.println("E 常量的值：" + Math.E);
System.out.println("PI 常量的值：" + Math.PI);
```
## 求最大值、最小值和绝对值
| 方法                                 | 说明 |
| :--: | :--: |
| static int abs(int a)                | 返回 a 的绝对值 |
| static long abs(long a)              | 返回 a 的绝对值 |
| static float abs(float a)            | 返回 a 的绝对值 |
| static double abs(double a)          | 返回 a 的绝对值 |
| static int max(int x,int y)          | 返回 x 和 y 中的最大值 |
| static double max(double x,double y) | 返回 x 和 y 中的最大值 |
| static long max(long x,long y)       | 返回 x 和 y 中的最大值 |
| static float max(float x,float y)    | 返回 x 和 y 中的最大值 |
| static int min(int x,int y)          | 返回 x 和 y 中的最小值 |
| static long min(long x,long y)       | 返回 x 和 y 中的最小值 |
| static double min(double x,double y) | 返回 x 和 y 中的最小值 |
| static float min(float x,float y)    | 返回 x 和 y 中的最小值 |

```java
public class Test02 {
    public static void main(String[] args) {
        System.out.println("10 和 20 的较大值：" + Math.max(10, 20));
        System.out.println("15.6 和 15 的较小值：" + Math.min(15.6, 15));
        System.out.println("-12 的绝对值：" + Math.abs(-12));
    }
}
```
## 求整运算

| 方法                          | 说明 |
| :--: | :--: |
| static double ceil(double a)  | 返回大于或等于 a 的最小整数 |
| static double floor(double a) | 返回小于或等于 a 的最大整数 |
| static double rint(double a)  | 返回最接近 a 的整数值，如果有两个同样接近的整数，则结果取偶数 |
| static int round(float a)     | 将参数加上 1/2 后返回与参数最近的整数 |
| static long round(double a)   | 将参数加上 1/2 后返回与参数最近的整数，然后强制转换为长整型 |

## 三角函数运算

| 方法 | 说明 |
| :--: | :--: |
| static double sin(double a)	| 返回角的三角正弦值，参数以孤度为单位 |
| static double cos(double a)	| 返回角的三角余弦值，参数以孤度为单位 |
| static double asin(double a)	| 返回一个值的反正弦值，参数域在 [-1,1]，值域在 [-PI/2,PI/2] |
| static double acos(double a)	| 返回一个值的反余弦值，参数域在 [-1,1]，值域在 [0.0,PI] |
| static double tan(double a)	| 返回角的三角正切值，参数以弧度为单位 |
| static double atan(double a)	| 返回一个值的反正切值，值域在 [-PI/2,PI/2] |
| static double toDegrees(double angrad) | 将用孤度表示的角转换为近似相等的用角度表示的角 |
| staticdouble toRadians(double angdeg)  | 将用角度表示的角转换为近似相等的用弧度表示的角 |

每个方法的参数和返回值都是`double`类型，参数以弧度代替角度来实现，其中 1 度等于`π/180`弧度，因此平角就是`π`弧度。
## 指数运算

| 方法                                 | 说明 |
| :--: | :--: |
| static double exp(double a)          | 返回 e 的 a 次幂 |
| static double pow(double a,double b) | 返回以 a 为底数，以 b 为指数的幂值 |
| static double sqrt(double a)         | 返回 a 的平方根 |
| static double cbrt(double a)         | 返回 a 的立方根 |
| static double log(double a)          | 返回 a 的自然对数，即 lna 的值 |
| static double log10(double a)        | 返回以 10 为底 a 的对数 |

# 生成随机数
要生成一个指定范围之内的随机数字有两种方法：一种是调用`Math`类的`random()`方法，一种是使用`Random`类。

`Random`类提供了丰富的随机数生成方法，可以产生`boolean、int、long、float、byte`数组以及`double`类型的随机数，这是它与`random()`方法最大的不同之处。`random()`方法只能产生`double`类型的 0~1 的随机数。

`Random`类位于`java.util`包中，该类常用的有如下两个构造方法。
* `Random()`：该构造方法使用一个和当前系统时间对应的数字作为种子数，然后使用这个种子数构造`Random`对象。
* `Random(long seed)`：使用单个`long`类型的参数创建一个新的随机数生成器。

`Random`类提供的所有方法生成的随机数字都是均匀分布的，也就是说区间内部的数字生成的概率是均等的，`Random`类中常用的方法：

| 方法 | 说明 |
| :--: | :--: |
| boolean nextBoolean()	  | 生成一个随机的 boolean 值，生成 true 和 false 的值概率相等 |
| double nextDouble()     | 生成一个随机的 double 值，数值介于 [0,1.0)，含 0 而不包含 1.0 |
| int nextlnt()           | 生成一个随机的 int 值，该值介于 int 的区间，也就是 -231~231-1。如果需要生成指定区间的 int 值，则需要进行一定的数学变换 |
| int nextlnt(int n)      | 生成一个随机的 int 值，该值介于 [0,n)，包含 0 而不包含 n。如果想生成指定区间的 int 值，也需要进行一定的数学变换 |
| void setSeed(long seed) | 重新设置 Random 对象中的种子数。设置完种子数以后的 Random 对象和相同种子数使用 new 关键字创建出的 Random 对象相同 |
| long nextLong()         | 返回一个随机长整型数字 |
| boolean nextBoolean()   | 返回一个随机布尔型值 |
| float nextFloat()       | 返回一个随机浮点型数字 |
| double nextDouble()     | 返回一个随机双精度值 |

```java
import java.util.Random;
public class Test06 {
    public static void main(String[] args) {
        Random r = new Random();
        double d1 = r.nextDouble(); // 生成[0,1.0]区间的小数
        double d2 = r.nextDouble() * 7; // 生成[0,7.0]区间的小数
        int i1 = r.nextInt(10); // 生成[0,10]区间的整数
        int i2 = r.nextInt(18) - 3; // 生成[-3,15)区间的整数
        long l1 = r.nextLong(); // 生成一个随机长整型值
        boolean b1 = r.nextBoolean(); // 生成一个随机布尔型值
        float f1 = r.nextFloat(); // 生成一个随机浮点型值
        System.out.println("生成的[0,1.0]区间的小数是：" + d1);
        System.out.println("生成的[0,7.0]区间的小数是：" + d2);
        System.out.println("生成的[0,10]区间的整数是：" + i1);
        System.out.println("生成的[-3,15]区间的整数是：" + i2);
        System.out.println("生成一个随机长整型值：" + l1);
        System.out.println("生成一个随机布尔型值：" + b1);
        System.out.println("生成一个随机浮点型值：" + f1);
        System.out.print("下期七星彩开奖号码预测：");
        for (int i = 1; i < 8; i++) {
            int num = r.nextInt(9); // 生成[0,9]区间的整数
            System.out.print(num);
        }
    }
}
```
该程序的运行结果如下：
```
生成的[0,1.0]区间的小数是：0.8773165855918825
生成的[0,7.0]区间的小数是：6.407083074782282
生成的[0,10]区间的整数是：5
生成的[-3,15]区间的整数是：4
生成一个随机长整型值：-8462847591661221914
生成一个随机布尔型值：false
生成一个随机浮点型值：0.6397003
下期七星彩开奖号码预测：0227168
```
`Math`类的`random()`方法没有参数，它默认会返回大于等于 0.0、小于 1.0 的`double`类型随机数，即`0<=随机数<1.0`。对`random()`方法返回的数字稍加处理，即可实现产生任意范围随机数的功能。
```java
public class Test07 {
    public static void main(String[] args) {
        int min = 2; // 定义随机数的最小值
        int max = 102; // 定义随机数的最大值
        // 产生一个2~100的数
        int s = (int) min + (int) (Math.random() * (max - min));
        if (s % 2 == 0) {
            // 如果是偶数就输出
            System.out.println("随机数是：" + s);
        } else {
            // 如果是奇数就加1后输出
            System.out.println("随机数是：" + (s + 1));
        }
    }
}
```
由于`m+(int)(Math.random()*n)`语句可以获取`m~m+n`的随机数，所以`2+(int)(Math.random()*(102-2))`表达式可以求出 2~100 的随机数。在产生这个区间的随机数后还需要判断是否为偶数，这里使用了对 2 取余数，如果余数不是零，说明随机数是奇数，此时将随机数加 1 后再输出。
# 数字格式化
可以使用`DedmalFormat`类对结果进行格式化处理。例如，将小数位统一成 2 位，不足 2 位的以 0 补齐。

`DecimalFormat`是`NumberFormat`的一个子类，用于格式化十进制数字。`DecimalFormat`类包含一个模式和一组符号，常用符号：

| 符号 | 说明 |
| :--: | :--: |
| 0	| 显示数字，如果位数不够则补 0 |
| #	| 显示数字，如果位数不够不发生变化 |
| .	| 小数分隔符 |
| -	| 减号 |
| ,	| 组分隔符 |
| E	| 分隔科学记数法中的尾数和小数 |
| %	| 前缀或后缀，乘以 100 后作为百分比显示 |
| ?	| 乘以 1000 后作为千进制货币符显示。用货币符号代替。如果双写，用国际货币符号代替；如果出现在一个模式中，用货币十进制分隔符代替十进制分隔符 |

```java
import java.text.DecimalFormat;
import java.util.Scanner;
public class Test08 {
    public static void main(String[] args) {
        // 实例化DecimalFormat类的对象，并指定格式
        DecimalFormat df1 = new DecimalFormat("0.0");
        DecimalFormat df2 = new DecimalFormat("#.#");
        DecimalFormat df3 = new DecimalFormat("000.000");
        DecimalFormat df4 = new DecimalFormat("###.###");
        Scanner scan = new Scanner(System.in);
        System.out.print("请输入一个float类型的数字：");
        float f = scan.nextFloat();
        // 对输入的数字应用格式，并输出结果
        System.out.println("0.0 格式：" + df1.format(f));
        System.out.println("#.# 格式：" + df2.format(f));
        System.out.println("000.000 格式：" + df3.format(f));
        System.out.println("###.### 格式：" + df4.format(f));
    }
}
```
输出结果如下所示：
```
请输入一个float类型的数字：5487.45697
0.0 格式：5487.5
#.# 格式：5487.5
000.000 格式：5487.457
###.### 格式：5487.457
请输入一个float类型的数字：5.0
0.0 格式：5.0
#.# 格式：5
000.000 格式：005.000
###.### 格式：5
```
# 大数字运算
在 Java 中提供了用于大数字运算的类，即`java.math.BigInteger`类和`java.math.BigDecimal`类。这两个类用于高精度计算，其中`BigInteger`类是针对整型大数字的处理类，而`BigDecimal`类是针对大小数的处理类。
## BigInteger 类
如果要存储比`Integer`更大的数字，可以使用`BigInteger`类来处理更大的数字。`BigInteger`支持任意精度的整数，也就是说在运算中`BigInteger`类型可以准确地表示任何大小的整数值。

除了基本的加、减、乘、除操作之外，`BigInteger`类还封装了很多操作，像求绝对值、相反数、最大公约数以及判断是否为质数等。

要使用`BigInteger`类，首先要创建一个`BigInteger`对象。`BigInteger`类提供了很多种构造方法，其中最直接的一种是参数以字符串形式代表要处理的数字。
```
BigInteger(String val)
```
这里的`val`是数字十进制的字符串。例如，要将数字 5 转换为`BigInteger`对象，语句如下：
```java
BigInteger bi = new BigInteger("5")
```
注意：这里数字 5 的双引号是必需的，因为`BigInteger`类构造方法要求参数是字符串类型。

创建`BigInteger`对象之后，便可以调用`BigInteger`类提供的方法进行各种数学运算操作，`BigInteger`类的常用运算方法：

| 方法名称 | 说明 |
| :--: | :--: |
| add(BigInteger val)       | 做加法运算 |
| subtract(BigInteger val)  | 做减法运算 |
| multiply(BigInteger val)  | 做乘法运算 |
| divide(BigInteger val)    | 做除法运算 |
| remainder(BigInteger val) | 做取余数运算 |
| divideAndRemainder(BigInteger val) | 做除法运算，返回数组的第一个值为商，第二个值为余数 |
| pow(int exponent)         | 做参数的 exponent 次方运算 |
| negate()                  | 取相反数 |
| shiftLeft(int n)          | 将数字左移 n 位，如果 n 为负数，则做右移操作 |
| shiftRight(int n)         | 将数字右移 n 位，如果 n 为负数，则做左移操作 |
| and(BigInteger val)       | 做与运算 |
| or(BigInteger val)        | 做或运算 |
| compareTo(BigInteger val) | 做数字的比较运算 |
| equals(Object obj)        | 当参数 obj 是 Biglnteger 类型的数字并且数值相等时返回 true, 其他返回 false |
| min(BigInteger val)       | 返回较小的数值 |
| max(BigInteger val)       | 返回较大的数值 |

```java
import java.math.BigInteger;
import java.util.Scanner;
public class Test09 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        System.out.println("请输入一个整型数字：");
        // 保存用户输入的数字
        int num = input.nextInt();
        // 使用输入的数字创建BigInteger对象
        BigInteger bi = new BigInteger(num + "");
        // 计算大数字加上99的结果
        System.out.println("加法操作结果：" + bi.add(new BigInteger("99")));
        // 计算大数字减去25的结果
        System.out.println("减法操作结果：" + bi.subtract(new BigInteger("25")));
        // 计算大数字乘以3的结果
        System.out.println("乘法橾作结果：" + bi.multiply(new BigInteger("3")));
        // 计算大数字除以2的结果
        System.out.println("除法操作结果：" + bi.divide(new BigInteger("2")));
        // 计算大数字除以3的商
        System.out.println("取商操作结果：" + bi.divideAndRemainder(new BigInteger("3"))[0]);
        // 计算大数字除以3的余数
        System.out.println("取余操作结果：" + bi.divideAndRemainder(new BigInteger("3"))[1]);
        // 计算大数字的2次方
        System.out.println("取 2 次方操作结果：" + bi.pow(2));
        // 计算大数字的相反数
        System.out.println("取相反数操作结果：" + bi.negate());
    }
}
```
运行效果下所示。
```
请输入一个整型数字：
125
加法操作结果：224
减法操作结果：100
乘法橾作结果：375
除法操作结果：62
取商操作结果：41
取余操作结果：2
取 2 次方操作结果：15625
取相反数操作结果：-125
```
## BigDecimal 类
`BigInteger`和`BigDecimal`都能实现大数字的运算，不同的是`BigDecimal`加入了小数的概念。一般的`float`和`double`类型数据只能用来做科学计算或工程计算，但由于在商业计算中要求数字精度比较高，所以要用到`BigDecimal`类。`BigDecimal`类支持任何精度的浮点数，可以用来精确计算货币值。

`BigDecimal`常用的构造方法：
* `BigDecimal(double val)`：实例化时将双精度型转换为`BigDecimal`类型。
* `BigDecimal(String val)`：实例化时将字符串形式转换为`BigDecimal`类型。

`BigDecimal`类的方法可以用来做超大浮点数的运算，像加、减、乘和除等。在所有运算中，除法运算是最复杂的，因为在除不尽的情况下，末位小数的处理方式是需要考虑的。
```java
BigDecimal add(BigDecimal augend)    // 加法操作
BigDecimal subtract(BigDecimal subtrahend)    // 减法操作
BigDecimal multiply(BigDecimal multiplieand)    // 乘法操作
BigDecimal divide(BigDecimal divisor,int scale,int roundingMode )    // 除法操作
```
其中，`divide()`方法的 3 个参数分别表示除数、商的小数点后的位数和近似值处理模式。

`roundingMode`参数支持的处理模式：

| 模式名称 | 说明 |
| :--: | :--: |
| BigDecimal.ROUND_UP           | 商的最后一位如果大于 0，则向前进位，正负数都如此 |
| BigDecimal.ROUND_DOWN         | 商的最后一位无论是什么数字都省略 |
| BigDecimal.ROUND_CEILING	    | 商如果是正数，按照 ROUND_UP 模式处理；如果是负数，按照 ROUND_DOWN 模式处理 |
| BigDecimal.ROUND_FLOOR        | 与 ROUND_CELING 模式相反，商如果是正数，按照 ROUND_DOWN 模式处理；如果是负数，按照 ROUND_UP 模式处理 |
| BigDecimal.ROUND_HALF_ DOWN	| 对商进行五舍六入操作。如果商最后一位小于等于 5，则做舍弃操作，否则对最后一位进行进位操作 |
| BigDecimal.ROUND_HALF_UP	    | 对商进行四舍五入操作。如果商最后一位小于 5，则做舍弃操作，否则对最后一位进行进位操作 |
| BigDecimal.ROUND_HALF_EVEN	| 如果商的倒数第二位是奇数，则按照 ROUND_HALF_UP 处理；如果是偶数，则按照 ROUND_HALF_DOWN 处理 |

```java
import java.math.BigDecimal;
import java.util.Scanner;
public class Test10 {
    public static void main(String[] args) {
        Scanner input = new Scanner(System.in);
        System.out.println("请输入一个数字：");
        // 保存用户输入的数字
        double num = input.nextDouble();
        // 使用输入的数字创建BigDecimal对象
        BigDecimal bd = new BigDecimal(num);
        // 计算大数字加上99.154的结果
        System.out.println("加法操作结果：" + bd.add(new BigDecimal(99.154)));
        // 计算大数字减去-25.157904的结果
        System.out.println("减法操作结果：" + bd.subtract(new BigDecimal(-25.157904)));
        // 计算大数字乘以3.5的结果
        System.out.println("乘法操作结果：" + bd.multiply(new BigDecimal(3.5)));
        // 计算大数字除以3.14的结果，并保留小数后2位
        System.out.println("除法操作结果(保留 2 位小数)：" + bd.divide(new BigDecimal(3.14), 2, BigDecimal.ROUND_CEILING));
        // 计算大数字除以3.14的结果，并保留小数后5位
        System.out.println("除法操作结果(保留 5 位小数)：" + bd.divide(new BigDecimal(3.14), 5, BigDecimal.ROUND_CEILING));
    }
}
```
运行效果如下所示。
```
请输入一个数字：
100
加法操作结果：199.15399999999999636202119290828704833984375
减法操作结果：125.157903999999998490011421381495893001556396484375
乘法操作结果：350.0
除法操作结果(保留 2 位小数)：31.85
除法操作结果(保留 5 位小数)：31.84714
```