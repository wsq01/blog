



# 比较两个数组是否相等
数组相等的条件不仅要求数组元素的个数必须相等，而且要求对应位置的元素也相等。`Arrays`类提供了`equals()`方法比较整个数组。
```java
Arrays.equals(arrayA, arrayB);
```
```java
public static void main(String[] args) {
  double[] score1 = { 99, 100, 98.5, 96.5, 72 };
  double[] score2 = new double[5];
  score2[0] = 99;
  score2[1] = 100;
  score2[2] = 98.5;
  score2[3] = 96.5;
  score2[4] = 72;
  double[] score3 = { 99, 96.5, 98.5, 100, 72 };
  if (Arrays.equals(score1, score2)) {
    System.out.println("score1 数组和 score2 数组相等");
  } else {
    System.out.println("score1 数组和 score2 数组不等");
  }
  if (Arrays.equals(score1, score3)) {
    System.out.println("score1 数组和 score3 数组相等");
  } else {
    System.out.println("score1 数组和 score3 数组不等");
  }
}
```
上述代码中定义 3 个数组，分别为`score1、score2`和`score3`。第一个数组直接给出了数组的值；第二个数组先定义数组的长度，然后为每个元素赋值；第三个数组中的元素和第一个数组中的元素相同，但是顺序不同。分别将`score1`数组与`score2`和`score3`数组进行比较，并输出比较的结果。

运行上述代码，输出结果如下：
```
score1 数组和 score2 数组相等
score1 数组和 score3 数组不等
```
# 数组填充
`Arrays`类提供了一个`fill()`方法，可以在指定位置进行数值填充。`fill()`方法虽然可以填充数组，但是它的功能有限制，只能使用同一个数值进行填充。语法如下：
```
Arrays.fill(array,value);
```
其中，`array`表示数组，`value`表示填充的值。
```java
public static void main(String[] args) {
  int[] number = new int[5];
  System.out.println("number —共有 " + number.length + " 个元素，它们分别是：");
  for (int i = 0; i < number.length; i++) {
    Arrays.fill(number, i);
    System.out.println("number[" + i + "]=" + i);
  }
}
```
输出结果如下所示。
```
number 一共有 5 个元素，它们分别是：
number[0]=0
number[1]=1
number[2]=2
number[3]=3
number[4]=4
```
# 数组查找指定元素
查找数组是指从数组中查询指定位置的元素，或者查询某元素在指定数组中的位置。使用 Arrays 类的 binarySearch() 方法可以实现数组的查找，该方法可使用二分搜索法来搜索指定数组，以获得指定对象，该方法返回要搜索元素的索引值。

binarySearch() 方法有多种重载形式来满足不同类型数组的查找需要，常用的重载形式有两种。

(1) 第一种形式如下：
binarySearch(Object[] a,Object key);
其中，a 表示要搜索的数组，key 表示要搜索的值。如果 key 包含在数组中，则返回搜索值的索引；否则返回 -1 或“-插入点”。插入点指搜索键将要插入数组的位置，即第一个大于此键的元素索引。

在进行数组查询之前，必须对数组进行排序（可以使用 sort() 方法）。如果没有对数组进行排序，则结果是不确定的。如果数组包含多个带有指定值的元素，则无法确认找到的是哪一个。
例 1
声明 double 类型的 score 数组，接着调用 Arrays 类的 sort() 方法对 score 数组排序，排序后分别查找数组中值为 100 和 60 的元素，分别将结果保存到 index1 和 index2 变量中，最后输出变量的值。代码如下：
public static void main(String[] args) {
    double[] score = { 99.5, 100, 98, 97.5, 100, 95, 85.5, 100 };
    Arrays.sort(score);
    int index1 = Arrays.binarySearch(score, 100);
    int index2 = Arrays.binarySearch(score, 60);
    System.out.println("查找到 100 的位置是：" + index1);
    System.out.println("查找到 60 的位置是：" + index2);
}
执行上述代码，输出结果如下：
查找到 100 的位置是：5
查找到 60 的位置是：-1

(2) 除了上述形式外，binarySearch() 还有另一种常用的形式，这种形式用于在指定的范围内查找某一元素。语法如下：
binarySearch(Object[] a,int fromIndex,int toIndex,Object key);
其中，a 表示要进行查找的数组，fromIndex 指定范围的开始处索引（包含开始处），toIndex 指定范围的结束处索引（不包含结束处），key 表示要搜索的元素。

在使用 binarySearch() 方法的上述重载形式时，也需要对数组进行排序，以便获取准确的索引值。如果要查找的元素 key 在指定的范围内，则返回搜索键的索引；否则返回 -1 或 “-插入点”。插入点指要将键插入数组的位置，即范围内第一个大于此键的元素索引。
例 2
对例 1 中创建的 score 数组进行查找元素，指定开始位置为 2，结束位置为 6。代码如下:
public static void main(String[] args) {
    double[] score = {99.5,100,98,97.5,100,95,85.5,100};
    Arrays.sort(score);
    int index1 = Arrays.binarySearch(score,2,6,100);
    int index2 = Arrays.binarySearch(score,2,6,60);
    System.out.println("查找到 100 的位置是："+index1);
    System.out.println("查找到 60 的位置是："+ index2);
}
执行上述代码，输出结果如下：
查找到 100 的位置是：5
查找到 60 的位置是：-3
# 复制数组
所谓复制数组，是指将一个数组中的元素在另一个数组中进行复制。本文主要介绍关于 Java 里面的数组复制（拷贝）的几种方式和用法。

在 Java 中实现数组复制分别有以下 4 种方法：
Arrays 类的 copyOf() 方法
Arrays 类的 copyOfRange() 方法
System 类的 arraycopy() 方法
Object 类的 clone() 方法

下面来详细介绍这 4 种方法的使用。
使用 copyOf() 方法和 copyOfRange() 方法
Arrays 类的 copyOf() 方法与 copyOfRange() 方法都可实现对数组的复制。copyOf() 方法是复制数组至指定长度，copyOfRange() 方法则将指定数组的指定长度复制到一个新数组中。
1. 使用 copyOf() 方法对数组进行复制
Arrays 类的 copyOf() 方法的语法格式如下：
Arrays.copyOf(dataType[] srcArray,int length);
其中，srcArray 表示要进行复制的数组，length 表示复制后的新数组的长度。

使用这种方法复制数组时，默认从原数组的第一个元素（索引值为 0）开始复制，目标数组的长度将为 length。如果 length 大于 srcArray.length，则目标数组中采用默认值填充；如果 length 小于 srcArray.length，则复制到第 length 个元素（索引值为 length-1）即止。

注意：目标数组如果已经存在，将会被重构。
例 1
假设有一个数组中保存了 5 个成绩，现在需要在一个新数组中保存这 5 个成绩，同时留 3 个空余的元素供后期开发使用。

使用 Arrays 类的 CopyOf() 方法完成数组复制的代码如下：
import java.util.Arrays;
public class Test19{
    public static void main(String[] args) {
        // 定义长度为 5 的数组
        int scores[] = new int[]{57,81,68,75,91};
        // 输出原数组
        System.out.println("原数组内容如下：");
        // 循环遍历原数组
        for(int i=0;i<scores.length;i++) {
            // 将数组元素输出
            System.out.print(scores[i]+"\t");
        }
        // 定义一个新的数组，将 scores 数组中的 5 个元素复制过来
        // 同时留 3 个内存空间供以后开发使用
        int[] newScores = (int[])Arrays.copyOf(scores,8);
        System.out.println("\n复制的新数组内容如下：");
        // 循环遍历复制后的新数组
        for(int j=0;j<newScores.length;j++) {
            // 将新数组的元素输出
            System.out.print(newScores[j]+"\t");
        }
    }
}
在上述代码中，由于原数组 scores 的长度为 5，而要复制的新数组 newScores 的长度为 8，因此在将原数组中的 5 个元素复制完之后，会采用默认值填充剩余 3 个元素的内容。

因为原数组 scores 的数据类型为 int，而使用 Arrays.copyOf(scores,8) 方法复制数组之后返回的是 Object[] 类型，因此需要将 Object[] 数据类型强制转换为 int[] 类型。同时，也正因为 scores 的数据类型为 int，因此默认值为 0。

运行的结果如下所示。
原数组内容如下：
57    81    68    75    91   
复制的新数组内容如下：
57    81    68    75    91    0    0    0
2. 使用 CopyOfRange() 方法对数组进行复制
Arrays 类的 CopyOfRange() 方法是另一种复制数组的方法，其语法形式如下：
Arrays.copyOfRange(dataType[] srcArray,int startIndex,int endIndex)
其中：
srcArray 表示原数组。
startIndex 表示开始复制的起始索引，目标数组中将包含起始索引对应的元素，另外，startIndex 必须在 0 到 srcArray.length 之间。
endIndex 表示终止索引，目标数组中将不包含终止索引对应的元素，endIndex 必须大于等于 startIndex，可以大于 srcArray.length，如果大于 srcArray.length，则目标数组中使用默认值填充。

注意：目标数组如果已经存在，将会被重构。
例 2
假设有一个名称为 scores 的数组其元素为 8 个，现在需要定义一个名称为 newScores 的新数组。新数组的元素为 scores 数组的前 5 个元素，并且顺序不变。

使用 Arrays 类 copyOfRange() 方法完成数组复制的代码如下：
public class Test20 {
    public static void main(String[] args) {
        // 定义长度为8的数组
        int scores[] = new int[] { 57, 81, 68, 75, 91, 66, 75, 84 };
        System.out.println("原数组内容如下：");
        // 循环遍历原数组
        for (int i = 0; i < scores.length; i++) {
            System.out.print(scores[i] + "\t");
        }
        // 复制原数组的前5个元素到newScores数组中
        int newScores[] = (int[]) Arrays.copyOfRange(scores, 0, 5);
        System.out.println("\n复制的新数组内容如下：");
        // 循环遍历目标数组，即复制后的新数组
        for (int j = 0; j < newScores.length; j++) {
            System.out.print(newScores[j] + "\t");
        }
    }
}
在上述代码中，原数组 scores 中包含有 8 个元素，使用 Arrays.copyOfRange() 方法可以将该数组复制到长度为 5 的 newScores 数组中，截取 scores 数组的前 5 个元素即可。

该程序运行结果如下所示。
原数组内容如下：
57    81    68    75    91    66    75    84   
复制的新数组内容如下：
57    81    68    75    91
使用 arraycopy() 方法
arraycopy() 方法位于 java.lang.System 类中，其语法形式如下：
System.arraycopy(dataType[] srcArray,int srcIndex,int destArray,int destIndex,int length)
其中，srcArray 表示原数组；srcIndex 表示原数组中的起始索引；destArray 表示目标数组；destIndex 表示目标数组中的起始索引；length 表示要复制的数组长度。

使用此方法复制数组时，length+srcIndex 必须小于等于 srcArray.length，同时 length+destIndex 必须小于等于 destArray.length。

注意：目标数组必须已经存在，且不会被重构，相当于替换目标数组中的部分元素。
例 3
假设在 scores 数组中保存了 8 名学生的成绩信息，现在需要复制该数组从第二个元素开始到结尾的所有元素到一个名称为 newScores 的数组中，长度为 12。scores 数组中的元素在 newScores 数组中从第三个元素开始排列。

使用 System.arraycopy() 方法来完成替换数组元素功能的代码如下：
public class Test21 {
    public static void main(String[] args) {
        // 定义原数组，长度为8
        int scores[] = new int[] { 100, 81, 68, 75, 91, 66, 75, 100 };
        // 定义目标数组
        int newScores[] = new int[] { 80, 82, 71, 92, 68, 71, 87, 88, 81, 79, 90, 77 };
        System.out.println("原数组中的内容如下：");
        // 遍历原数组
        for (int i = 0; i < scores.length; i++) {
            System.out.print(scores[i] + "\t");
        }
        System.out.println("\n目标数组中的内容如下：");
        // 遍历目标数组
        for (int j = 0; j < newScores.length; j++) {
            System.out.print(newScores[j] + "\t");
        }
        System.arraycopy(scores, 0, newScores, 2, 8);
        // 复制原数组中的一部分到目标数组中
        System.out.println("\n替换元素后的目标数组内容如下：");
        // 循环遍历替换后的数组
        for (int k = 0; k < newScores.length; k++) {
            System.out.print(newScores[k] + "\t");
        }
    }
}
在该程序中，首先定义了一个包含有 8 个元素的 scores 数组，接着又定义了一个包含有 12 个元素的 newScores 数组，然后使用 for 循环分别遍历这两个数组，输出数组中的元素。最后使用 System.arraycopy() 方法将 newScores 数组中从第三个元素开始往后的 8 个元素替换为 scores 数组中的 8 个元素值。

该程序运行的结果如下所示。
原数组中的内容如下：
100    81    68    75    91    66    75    100   
目标数组中的内容如下：
80    82    71    92    68    71    87    88    81    79    90    77   
替换元素后的目标数组内容如下：
80    82    100    81    68    75    91    66    75    100    90    77   

注意：在使用 arraycopy() 方法时要注意，此方法的命名违背了 Java 的命名惯例。即第二个单词 copy 的首字母没有大写，但按惯例写法应该为 arrayCopy。请读者在使用此方法时注意方法名的书写。
使用 clone() 方法
clone() 方法也可以实现复制数组。该方法是类 Object 中的方法，可以创建一个有单独内存空间的对象。因为数组也是一个 Object 类，因此也可以使用数组对象的 clone() 方法来复制数组。

clone() 方法的返回值是 Object 类型，要使用强制类型转换为适当的类型。其语法形式比较简单：
array_name.clone()

示例语句如下：
int[] targetArray=(int[])sourceArray.clone();

注意：目标数组如果已经存在，将会被重构。
例 4
有一个长度为 8 的 scores 数组，因为程序需要，现在要定义一个名称为 newScores 的数组来容纳 scores 数组中的所有元素，可以使用 clone() 方法来将 scores 数组中的元素全部复制到 newScores 数组中。代码如下：
public class Test22 {
    public static void main(String[] args) {
        // 定义原数组，长度为8
        int scores[] = new int[] { 100, 81, 68, 75, 91, 66, 75, 100 };
        System.out.println("原数组中的内容如下：");
        // 遍历原数组
        for (int i = 0; i < scores.length; i++) {
            System.out.print(scores[i] + "\t");
        }
        // 复制数组，将Object类型强制转换为int[]类型
        int newScores[] = (int[]) scores.clone();
        System.out.println("\n目标数组内容如下：");
        // 循环遍历目标数组
        for (int k = 0; k < newScores.length; k++) {
            System.out.print(newScores[k] + "\t");
        }
    }
}
在该程序中，首先定义了一个长度为 8 的 scores 数组，并循环遍历该数组输出数组中的元素，然后定义了一个名称为 newScores 的新数组，并使用 scores.clone() 方法将 scores 数组中的元素复制给 newScores 数组。最后循环遍历 newScores 数组，输出数组元素。

程序运行结果如下所示。
原数组中的内容如下：
100    81    68    75    91    66    75    100   
目标数组内容如下：
100    81    68    75    91    66    75    100   
从运行的结果可以看出，scores 数组的元素与 newScores 数组的元素是相同的。

注意：以上几种方法都是浅拷贝（浅复制）。浅拷贝只是复制了对象的引用地址，两个对象指向同一个内存地址，所以修改其中任意的值，另一个值都会随之变化。深拷贝是将对象及值复制过来，两个对象修改其中任意的值另一个值不会改变。
# Arrays工具类
`Arrays`类是一个工具类，其中包含了数组操作的很多方法。这个`Arrays`类里均为`static`修饰的方法（static 修饰的方法可以直接通过类名调用），可以直接通过`Arrays.xxx(xxx)`的形式调用方法。
1）int binarySearch(type[] a, type key)
使用二分法查询 key 元素值在 a 数组中出现的索引，如果 a 数组不包含 key 元素值，则返回负数。调用该方法时要求数组中元素己经按升序排列，这样才能得到正确结果。
2）int binarySearch(type[] a, int fromIndex, int toIndex, type key)
这个方法与前一个方法类似，但它只搜索 a 数组中 fromIndex 到 toIndex 索引的元素。调用该方法时要求数组中元素己经按升序排列，这样才能得到正确结果。
3）type[] copyOf(type[] original, int length)
这个方法将会把 original 数组复制成一个新数组，其中 length 是新数组的长度。如果 length 小于 original 数组的长度，则新数组就是原数组的前面 length 个元素，如果 length 大于 original 数组的长度，则新数组的前面元索就是原数组的所有元素，后面补充 0（数值类型）、false（布尔类型）或者 null（引用类型）。
4）type[] copyOfRange(type[] original, int from, int to)
这个方法与前面方法相似，但这个方法只复制 original 数组的 from 索引到 to 索引的元素。
5）boolean equals(type[] a, type[] a2)
如果 a 数组和 a2 数组的长度相等，而且 a 数组和 a2 数组的数组元素也一一相同，该方法将返回 true。
6）void fill(type[] a, type val)
该方法将会把 a 数组的所有元素都赋值为 val。
7）void fill(type[] a, int fromIndex, int toIndex, type val)
该方法与前一个方法的作用相同，区别只是该方法仅仅将 a 数组的 fromIndex 到 toIndex 索引的数组元素赋值为 val。
8）void sort(type[] a)
该方法对 a 数组的数组元素进行排序。
9）void sort(type[] a, int fromIndex, int toIndex)
该方法与前一个方法相似，区别是该方法仅仅对 fromIndex 到 toIndex 索引的元素进行排序。
10）String toString(type[] a)
该方法将一个数组转换成一个字符串。该方法按顺序把多个数组元素连缀在一起，多个数组元素使用英文逗号,和空格隔开。

下面程序示范了 Arrays 类的用法。
public class ArraysTest {
    public static void main(String[] args) {
        // 定义一个a数组
        int[] a = new int[] { 3, 4, 5, 6 };
        // 定义一个a2数组
        int[] a2 = new int[] { 3, 4, 5, 6 };
        // a数组和a2数组的长度相等，毎个元素依次相等，将输出true
        System.out.println("a数组和a2数组是否相等：" + Arrays.equals(a, a2));
        // 通过复制a数组，生成一个新的b数组
        int[] b = Arrays.copyOf(a, 6);
        System.out.println("a数组和b数组是否相等：" + Arrays.equals(a, b));
        // 输出b数组的元素，将输出[3, 4, 5, 6, 0, 0]
        System.out.println("b 数组的元素为：" + Arrays.toString(b));
        // 将b数组的第3个元素（包括）到第5个元素（不包括）賦值为1
        Arrays.fill(b, 2, 4, 1);
        // 输出b数组的元素，将输出[3, 4, 1, 1, 0, 0]
        System.out.println("b 数组的元素为：" + Arrays.toString(b));
        // 对b数组进行排序
        Arrays.sort(b);
        // 输出b数组的元素.将输出[0,0,1,1,3,4]
        System.out.println("b数组的元素为：" + Arrays.toString(b));
    }
}
Arrays 类处于 java.util 包下，为了在程序中使用 Arrays 类，必须在程序中导入 java.util.Arrays 类。

除此之外，在 System 类里也包含了一个static void arraycopy(Object src, int srePos, Object dest, int dcstPos, int length)方法，该方法可以将 src 数组里的元素值赋给 dest 数组的元素，其中 srcPos 指定从 src 数组的第几个元素开始赋值，length 参数指定将 src 数组的多少个元素值赋给 dest 数组的元素。

Java 8 增强了 Arrays 类的功能，为 Arrays 类增加了一些工具方法，这些工具方法可以充分利用多 CPU 并行的能力来提高设值、排序的性能。下面是 Java 8 为 Arrays 类增加的工具方法。

提示：由于计算机硬件的飞速发展，目前几乎所有家用 PC 都是 4 核、8 核的 CPU，而服务器的 CPU 则具有更好的性能，因此 Java 8 与时俱进地增加了并发支持，并发支持可以充分利用硬件设备来提高程序的运行性能。
1）oid parallelPrefix(xxx[] array, XxxBinaryOperator op)
该方法使用 op 参数指定的计算公式计算得到的结果作为新的元素。op 计算公式包括 left、right 两个形参，其中 left 代表数组中前一个索引处的元素，right 代表数组中当前索引处的元素，当计算第一个新数组元素时，left 的值默认为 1。
2）void parallelPrefix(xxx[] array, int fromIndex, int toIndex, XxxBinaryOperator op)
该方法与上一个方法相似，区别是该方法仅重新计算 fromIndex 到 toIndex 索引的元素。
3）void setAll(xxx[] array, IntToXxxFunction generator)
该方法使用指定的生成器（generator）为所有数组元素设置值，该生成器控制数组元素的值的生成算法。
4）void parallelSetAll(xxx[] array, IntToXxxFunction generator)
该方法的功能与上一个方法相同，只是该方法增加了并行能力，可以利用多 CPU 并行来提高性能。
5）void parallelSort(xxx[] a)
该方法的功能与 Arrays 类以前就有的 sort() 方法相似，只是该方法增加了并行能力，可以利用多 CPU 并行来提高性能。
6）void parallelSort(xxx[] a，int fromIndex, int toIndex)
该方法与上一个方法相似，区別是该方法仅对 fromIndex 到 toIndex 索引的元素进行排序。
7）Spliterator.OfXxx spliterator(xxx[] array)
将该数组的所有元素转换成对应的 Spliterator 对象。
8）Spliterator.OfXxx spliterator(xxx[] array, int startInclusive, int endExclusive)
该方法与上一个方法相似，区别是该方法仅转换 startInclusive 到 endExclusive 索引的元素。
9）XxxStream stream(xxx[] array)
该方法将数组转换为 Stream，Stream 是 Java 8 新增的流式编程的 API。
10）XxxStream stream(xxx[] array, int startInclusive, int endExclusive)
该方法与上一个方法相似，区别是该方法仅将 fromIndex 到 toIndex 索引的元索转换为 Stream。

上面方法列表中，所有以 parallel 开头的方法都表示该方法可利用 CPU 并行的能力来提高性能。上面方法中的 xxx 代表不同的数据类型，比如处理 int[] 型数组时应将 xxx 换成 int，处理 long[] 型数组时应将 XXX 换成 long。

下面程序示范了 Java 8 为 Arrays 类新增的方法。

下面程序用到了接口、匿名内部类的知识，读者阅读起来可能有一定的困难，此处只要大致知道 Arrays 新增的这些新方法就行，暂时并不需要读者立即掌握该程序，可以等到掌握了接口、匿名内部类后再来学习下面程序。
public class ArraysTest2 {
    public static void main(String[] args) {
        int[] arr1 = new int[] { 3, 4, 25, 16, 30, 18 };
        // 对数组arr1进行并发排序
        Arrays.parallelSort(arr1);
        System.out.println(Arrays.toString(arr1));
        int[] arr2 = new int[] { 13, -4, 25, 16, 30, 18 };
        Arrays.parallelPrefix(arr2, new IntBinaryOperator() {
            // left 代表数组中前一个索引处的元素，计算第一个元素时，left为1
            // right代表数组中当前索引处的元素
            public int applyAsInt(int left, int right) {
                return left * right;
            }
        });
        System.out.println(Arrays.toString(arr2));
        int[] arr3 = new int[5];
        Arrays.parallelSetAll(arr3, new IntUnaryOperator() {
            // operand代表正在计算的元素索引
            public int applyAsInt(int operand) {
                return operand * 5;
            }
        });
        System.out.println(Arrays.toString(arr3));
    }
}
上面程序中第一行粗体字代码调用了 parallelSort() 方法对数组执行排序，该方法的功能与传统 sort() 方法大致相似，只是在多 CPU 机器上会有更好的性能。

第二段粗体字代码使用的计算公式为 left * right，其中 left 代表数组中当前一个索引处的元素，right 代表数组中当前索引处的元素。程序使用的数组为：
{3, -4 , 25, 16, 30, 18)

计算新的数组元素的方式为：
{1*3=3, 3*-4—12, -12*25=-300, -300*16=—48000, -48000*30=—144000, -144000*18=-2592000}

因此将会得到如下新的数组元素：
{3, -12, -300, -4800, -144000, -2592000)

第三段粗体字代码使用 operand * 5 公式来设置数组元素，该公式中 operand 代表正在计算的数组元素的索引。因此第三段粗体字代码计算得到的数组为：
{0, 5, 10, 15, 20}

提示：上面两段粗体字代码都可以使用 Lambda 表达式进行简化。