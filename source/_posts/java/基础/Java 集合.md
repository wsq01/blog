---
title: Java 集合
date: 2020-10-09 11:31:41
tags: [java]
categories: java
---

在编程时，可以使用数组来保存多个对象，但数组长度不可变化，一旦在初始化数组时指定了数组长度，这个数组长度就是不可变的。如果需要保存数量变化的数据，数组就有点无能为力了。而且数组无法保存具有映射关系的数据。

为了保存数量不确定的数据，以及保存具有映射关系的数据（也被称为关联数组），Java 提供了集合类。集合类主要负责保存、盛装其他数据，因此集合类也被称为容器类。Java 所有的集合类都位于`java.util`包下，提供了一个表示和操作对象集合的统一构架，包含大量集合接口，以及这些接口的实现类和操作它们的算法。

集合类和数组不一样，数组元素既可以是基本类型的值，也可以是对象（实际上保存的是对象的引用变量），而集合里只能保存对象（实际上只是保存对象的引用变量，但通常习惯上认为集合里保存的是对象）。

Java 集合类型分为`Collection`和`Map`，它们是 Java 集合的根接口，这两个接口又包含了一些子接口或实现类。下图分别为`Collection`和`Map`的子接口及其实现类。黄色块为集合的接口，蓝色块为集合的实现类。

{% asset_img 1.png Collection接口基本结构 %}
{% asset_img 2.png Map接口基本结构 %}

Java集合接口的作用：

| 接口名称 | 作    用 |
| :--: | :--: |
| `Iterator`接口 | 集合的输出接口，主要用于遍历输出（即迭代访问）`Collection`集合中的元素，`Iterator`对象被称之为迭代器。迭代器接口是集合接口的父接口，实现类实现`Collection`时就必须实现`Iterator`接口。 |
| `Collection`接口 | 是`List、Set`和`Queue`的父接口，是存放一组单值的最大接口。所谓的单值是指集合中的每个元素都是一个对象。一般很少直接使用此接口直接操作。 |
| `Queue`接口 | `Queue`是 Java 提供的队列实现，有点类似于`List`。 |
| `Dueue`接口 | 是`Queue`的一个子接口，为双向队列。 |
| `List`接口 | 是有序集合，允许有相同的元素。使用`List`能够精确地控制每个元素插入的位置，用户能够使用索引（元素在`List`中的位置，类似于数组下标）来访问`List`中的元素，与数组类似。 |
| `Set`接口 | 不能包含重复的元素。 |
| `Map`接口 | 是存放一对值的最大接口，即接口中的每个元素都是一对，以`key➡value`的形式保存。 |

对于`Set、List、Queue`和`Map`这 4 种集合，最常用的实现类分别是：

| 类名称 | 作    用 |
| :--: | :--: |
| `HashSet` | 为优化査询速度而设计的`Set`。它是基于`HashMap`实现的，`HashSet`底层使用`HashMap`来保存所有元素，实现比较简单 |
| `TreeSet` | 实现了`Set`接口，是一个有序的`Set`，这样就能从`Set`里面提取一个有序序列 |
| `ArrayList` | 一个用数组实现的`List`，能进行快速的随机访问，效率高而且实现了可变大小的数组 |
| `ArrayDueue` | 是一个基于数组实现的双端队列，按“先进先出”的方式操作集合元素 |
| `LinkedList` | 对顺序访问进行了优化，但随机访问的速度相对较慢。此外它还有`addFirst()、addLast()、getFirst()、getLast()、removeFirst()`和`removeLast()`等方法，能把它当成栈（`Stac`k）或队列（`Queue`）来用 |
| `HsahMap` | 按哈希算法来存取键对象 |
| `TreeMap` | 可以对键对象进行排序 |

# Collection接口
`Collection`接口是`List、Set`和`Queue`接口的父接口，通常情况下不被直接使用。`Collection`接口定义了一些通用的方法，通过这些方法可以实现对集合的基本操作。定义的方法既可用于操作`Set`集合，也可用于操作`List`和`Queue`集合。

| 方法名称 | 说明 |
| :--: | :--: |
| `boolean add(E e)` | 向集合中添加一个元素，如果集合对象被添加操作改变了，则返回 true。E 是元素的数据类型 |
| `boolean addAll(Collection c)` | 向集合中添加集合 c 中的所有元素，如果集合对象被添加操作改变了，则返回 true。 |
| `void clear()` | 清除集合中的所有元素，将集合长度变为 0。 |
| `boolean contains(Object o)` | 判断集合中是否存在指定元素 |
| `boolean containsAll(Collection c)` | 判断集合中是否包含集合 c 中的所有元素 |
| `boolean isEmpty()` | 判断集合是否为空 |
| `Iterator<E> iterator()` | 返回一个 Iterator 对象，用于遍历集合中的元素 |
| `boolean remove(Object o)` | 从集合中删除一个指定元素，当集合中包含了一个或多个元素 o 时，该方法只删除第一个符合条件的元素，该方法将返回 true。 |
| `boolean removeAll(Collection c)` | 从集合中删除所有在集合 c 中出现的元素（相当于把调用该方法的集合减去集合 c）。如果该操作改变了调用该方法的集合，则该方法返回 true。 |
| `boolean retainAll(Collection c)` | 从集合中删除集合 c 里不包含的元素（相当于把调用该方法的集合变成该集合和集合 c 的交集），如果该操作改变了调用该方法的集合，则该方法返回 true。 |
| `int size()` | 返回集合中元素的个数 |
| `Object[] toArray()` | 把集合转换为一个数组，所有的集合元素变成对应的数组元素。 |

`Iterator`接口包含3个方法：
```java
public Interface Interator<E> {
  // 返回将要访问的下一个对象
  // 如果已经到达了集合的尾部，将抛出一个NoSuchElement Exception。
  E next();
  // 如果存在可访问的元素，返回true
  boolean hasNext(); 
  // 删除上次访问的对象。
  // 这个方法必须紧跟在访问一个元素之后执行。
  // 如果上次访问之后，集合已经发生了变化，这个方法将抛出一个llegalStateException。
  void remove(); // 
}
```
通过反复调用`next()`方法，可以逐个访问集合中的每个元素。但是如果到达了集合的末尾，`next`方法将抛出一个`NoSuchElementException`。因此，需要在调用`next`之前调用`hasNext`方法。

如果迭代器对象还有多个供访问的元素，`haxNext`方法就返回`true`。如果想要查看集合中的所有元素，就请求一个迭代器，并在`hasNext`返回`true`时反复地调用`next`方法。当调用`next`时，迭代器会越过下一个元素，并返回刚刚越过的那个元素的引用。
```java
public static void main(String[] args) {
  ArrayList list1 = new ArrayList(); // 创建集合 list1
  ArrayList list2 = new ArrayList(); // 创建集合 list2
  list1.add("one"); // 向 list1 添加一个元素
  list1.add("two"); // 向 list1 添加一个元素
  list2.addAll(list1); // 将 list1 的所有元素添加到 list2
  list2.add("three"); // 向 list2 添加一个元素
  System.out.println("list2 集合中的元素如下：");
  Iterator it1 = list2.iterator();
  while (it1.hasNext()) {
    System.out.print(it1.next() + "、");
  }
}
```
由于`Collection`是接口，不能对其实例化，所以上述代码中使用了`Collection`接口的`ArrayList`实现类来调用`Collection`的方法。结果如下：
```
list2 集合中的元素如下：
one、two、three、
```
# List集合
`List`是一个有序、可重复的集合，集合中每个元素都有其对应的顺序索引。`List`集合允许使用重复元素，可以通过索引来访问指定位置的集合元素。`List`集合默认按元素的添加顺序设置元素的索引，第一个添加到`List`集合中的元素的索引为 0，第二个为 1，依此类推。

`List`实现了`Collection`接口，它主要有两个常用的实现类：`ArrayList`类和`LinkedList`类。
## ArrayList 类
`ArrayList`类实现了可变数组的大小，存储在内的数据称为元素。它还提供了快速基于索引访问元素的方式，对尾部成员的增加和删除支持较好。使用`ArrayList`创建的集合，允许对集合中的元素进行快速的随机访问，不过，向`ArrayList`中插入与删除元素的速度相对较慢。

`ArrayList`类的常用构造方法有如下两种重载形式：
* `ArrayList()`：构造一个初始容量为 10 的空列表。
* `ArrayList(Collection<?extends E>c)`：构造一个包含指定`Collection`元素的列表，这些元素是按照该`Collection`的迭代器返回它们的顺序排列的。

`ArrayList`类除了包含`Collection`接口中的所有方法之外，还包括`List`接口中提供的方法。

| 方法名称 | 说明 |
| :--: | :--: |
| `E get(int index)` | 获取此集合中指定索引位置的元素，`E`为集合中元素的数据类型 |
| `int index(Object o)` | 返回此集合中第一次出现指定元素的索引，如果此集合不包含该元素，则返回 -1 |
| `int lastIndexOf(Object o)` | 返回此集合中最后一次出现指定元素的索引，如果此集合不包含该元素，则返回 -1 |
| `E set(int index, Eelement)` | 将此集合中指定索引位置的元素修改为`element`参数指定的对象。此方法返回此集合中指定索引位置的原元素 |
| `List<E> subList(int fromlndex, int tolndex)` | 返回一个新的集合，新集合中包含`fromlndex`和`tolndex`索引之间的所有元素。包含`fromlndex`处的元素，不包含`tolndex`索引处的元素 |

注意：当调用`List`的`set(int index, Object element)`方法来改变`List`集合指定索引处的元素时，指定的索引必须是`List`集合的有效索引。例如集合长度为 4，就不能指定替换索引为 4 处的元素，也就是说这个方法不会改变`List`集合的长度。
## LinkedList类
`LinkedList`类采用链表结构保存对象，这种结构的优点是便于向集合中插入或者删除元素。需要频繁向集合中插入和删除元素时，使用`LinkedList`类比`ArrayList`类效果高，但是`LinkedList`类随机访问元素的速度则相对较慢。这里的随机访问是指检索集合中特定索引位置的元素。

`LinkedList`类除了包含`Collection`接口和`List`接口中的所有方法之外，还特别提供了下面的方法。

| 方法名称 | 说明 |
| :--: | :--: |
| `void addFirst(E e)` | 将指定元素添加到此集合的开头 |
| `void addLast(E e)`  | 将指定元素添加到此集合的末尾 |
| `E getFirst()`       | 返回此集合的第一个元素       |
| `E getLast()`        | 返回此集合的最后一个元素     |
| `E removeFirst()`	   | 删除此集合中的第一个元素     |
| `E removeLast()`     | 删除此集合中的最后一个元素   |

```java
public class Test {
  public static void main(String[] args) {
    LinkedList<String> products = new LinkedList<String>(); // 创建集合对象
    String p1 = new String("六角螺母");
    String p2 = new String("10A 电缆线");
    String p3 = new String("5M 卷尺");
    String p4 = new String("4CM 原木方板");
    products.add(p1); // 将 p1 对象添加到 LinkedList 集合中
    products.add(p2); // 将 p2 对象添加到 LinkedList 集合中
    products.add(p3); // 将 p3 对象添加到 LinkedList 集合中
    products.add(p4); // 将 p4 对象添加到 LinkedList 集合中
    String p5 = new String("标准文件夹小柜");
    products.addLast(p5); // 向集合的末尾添加p5对象
    System.out.print("*************** 商品信息 ***************");
    System.out.println("\n目前商品有：");
    for (int i = 0; i < products.size(); i++) {
      System.out.print(products.get(i) + "\t");
    }
    System.out.println("\n第一个商品的名称为：" + products.getFirst());
    System.out.println("最后一个商品的名称为：" + products.getLast());
    products.removeLast(); // 删除最后一个元素
    System.out.println("删除最后的元素，目前商品有：");
    for (int i = 0; i < products.size(); i++) {
      System.out.print(products.get(i) + "\t");
    }
  }
}
```
`LinkedList<String>`中的`<String>`是 Java 中的泛型，用于指定集合中元素的数据类型，例如这里指定元素类型为`String`，则该集合中不能添加非`String`类型的元素。
## ArrayList 类和 LinkedList 类的区别
`ArrayList`与`LinkedList`都是`List`接口的实现类，因此都实现了`List`的所有未实现的方法，只是实现的方式有所不同。

`ArrayList`是基于动态数组数据结构的实现，访问元素速度优于`LinkedList`。`LinkedList`是基于链表数据结构的实现，占用的内存空间比较大，但在批量插入或删除数据时优于`ArrayList`。

对于快速访问对象的需求，使用`ArrayList`实现执行效率上会比较好。需要频繁向集合中插入和删除元素时，使用`LinkedList`类比`ArrayList`类效果高。

不同的结构对应于不同的算法，有的考虑节省占用空间，有的考虑提高运行效率，对于程序员而言，它们就像是“熊掌”和“鱼肉”，不可兼得。高运行速度往往是以牺牲空间为代价的，而节省占用空间往往是以牺牲运行速度为代价的。
# Set集合
`Set`集合类似于一个罐子，程序可以依次把多个对象“丢进”`Set`集合，而`Set`集合通常不能记住元素的添加顺序。也就是说`Set`集合中的对象不按特定的方式排序，只是简单地把对象加入集合。`Set`集合中不能包含重复的对象，并且最多只允许包含一个`null`元素。

`Set`实现了`Collection`接口，它主要有两个常用的实现类：`HashSet`类和`TreeSet`类。
## HashSet 类
`HashSet`是按照`Hash`算法来存储集合中的元素。因此具有很好的存取和查找性能。

`HashSet`具有以下特点：
* 不能保证元素的排列顺序，顺序可能与添加顺序不同，顺序也有可能发生变化。
* `HashSet`不是同步的，如果多个线程同时访问或修改一个`HashSet`，则必须通过代码来保证其同步。
* 集合元素值可以是`null`。

当向`HashSet`集合中存入一个元素时，`HashSet`会调用该对象的`hashCode()`方法来得到该对象的`hashCode`值，然后根据该`hashCode`值决定该对象在`HashSet`中的存储位置。如果有两个元素通过`equals()`方法比较返回的结果为`true`，但它们的`hashCode`不相等，`HashSet`将会把它们存储在不同的位置，依然可以添加成功。

也就是说，两个对象的`hashCode`值相等且通过`equals()`方法比较返回结果为`true`，则`HashSet`集合认为两个元素相等。

在`HashSet`类中实现了`Collection`接口中的所有方法。`HashSet`类的常用构造方法重载形式如下。
* `HashSet()`：构造一个新的空的`Set`集合。
* `HashSet(Collection<? extends E>c)`：构造一个包含指定`Collection`集合元素的新`Set`集合。其中，`< >`中的`extends`表示`HashSet`的父类，即指明该`Set`集合中存放的集合元素类型。`c`表示其中的元素将被存放在此`Set`集合中。

```java
HashSet hs = new HashSet();    // 调用无参的构造函数创建HashSet对象
HashSet<String> hss = new HashSet<String>();    // 创建泛型的 HashSet 集合对象
```
## TreeSet 类
`TreeSet`类同时实现了`Set`接口和`SortedSet`接口。`SortedSet`接口是`Set`接口的子接口，可以实现对集合进行自然排序，因此使用`TreeSet`类实现的`Set`接口默认情况下是自然排序的，这里的自然排序指的是升序排序。

`TreeSet`只能对实现了`Comparable`接口的类对象进行排序，因为`Comparable`接口中有一个`compareTo(Object o)`方法用于比较两个对象的大小。例如`a.compareTo(b)`，如果 a 和 b 相等，则该方法返回 0；如果 a 大于 b，则该方法返回大于 0 的值；如果 a 小于 b，则该方法返回小于 0 的值。

表列举了 JDK 类库中实现`Comparable`接口的类，以及这些类对象的比较方式。

| 类 | 比较方式 |
| :--: | :--: |
| 包装类（BigDecimal、Biglnteger、 Byte、<br>Double、Float、Integer、Long 及 Short) | 按数字大小比较 |
| Character | 按字符的 Unicode 值的数字大小比较 |
| String | 按字符串中字符的 Unicode 值的数字大小比较 |
| TreeSet | 类除了实现 Collection 接口的所有方法之外，还提供了如表 2 所示的方法。 |

`TreeSet`类除了实现`Collection`接口的所有方法之外，还提供了如表所示的方法。

| 方法名称 | 说明 |
| :--: | :--: |
| E first() | 返回此集合中的第一个元素。其中，E 表示集合中元素的数据类型 |
| E last() | 返回此集合中的最后一个元素 |
| E poolFirst() | 获取并移除此集合中的第一个元素 |
| E poolLast() | 获取并移除此集合中的最后一个元素 |
| `SortedSet<E> subSet(E fromElement,E toElement)` | 返回一个新的集合，新集合包含原集合中 fromElement 对象与 toElement对象之间的所有对象。包含 fromElement 对象，不包含 toElement 对象 |
| `SortedSet<E> headSet<E toElement>` | 返回一个新的集合，新集合包含原集合中 toElement 对象之前的所有对象。不包含 toElement 对象 |
| `SortedSet<E> tailSet(E fromElement)` | 返回一个新的集合，新集合包含原集合中 fromElement 对象之后的所有对象。包含 fromElement 对象注意：表面上看起来这些方法很多，其实很简单。因为 TreeSet 中的元素是有序的，所以增加了访问第一个、前一个、后一个、最后一个元素的方法，并提供了 3 个从 TreeSet 中截取子 TreeSet 的方法。 |

# Map集合
`Map`是一种键-值对（`key-value`）集合，`Map`集合中的每一个元素都包含一个键（`key`）对象和一个值（`value`）对象。用于保存具有映射关系的数据。

`Map`集合里保存着两组值，一组值用于保存`Map`里的`key`，另外一组值用于保存`Map`里的`value`，`key`和`value`都可以是任何引用类型的数据。`Map`的`key`不允许重复，`value`可以重复，即同一个`Map`对象的任何两个`key`通过`equals`方法比较总是返回`false`。

`Map`中的`key`和`value`之间存在单向一对一关系，即通过指定的`key`，总能找到唯一的、确定的`value`。从`Map`中取出数据时，只要给出指定的`key`，就可以取出对应的`value`。

`Map`接口主要有两个实现类：`HashMap`类和`TreeMap`类。其中，`HashMap`类按哈希算法来存取键对象，而`TreeMap`类可以对键对象进行排序。

| 方法名称 | 说明 |
| :--: | :--: |
| void clear() | 删除该 Map 对象中的所有 key-value 对。 |
| boolean containsKey(Object key) | 查询 Map 中是否包含指定的 key，如果包含则返回 true。 |
| boolean containsValue(Object value) | 查询 Map 中是否包含一个或多个 value，如果包含则返回 true。 |
| V get(Object key) | 返回 Map 集合中指定键对象所对应的值。V 表示值的数据类型 |
| V put(K key, V value) | 向 Map 集合中添加键-值对，如果当前 Map 中已有一个与该 key 相等的 key-value 对，则新的 key-value 对会覆盖原来的 key-value 对。 |
| void putAll(Map m) | 将指定 Map 中的 key-value 对复制到本 Map 中。 |
| V remove(Object key) | 从 Map 集合中删除 key 对应的键-值对，返回 key 对应的 value，如果该 key 不存在，则返回 null |
| boolean remove(Object key, Object value) | 这是 Java 8 新增的方法，删除指定 key、value 所对应的 key-value 对。如果从该 Map 中成功地删除该 key-value 对，该方法返回 true，否则返回 false。 |
| Set entrySet() | 返回 Map 集合中所有键-值对的 Set 集合，此 Set 集合中元素的数据类型为 Map.Entry |
| Set keySet() | 返回 Map 集合中所有键对象的 Set 集合 |
| boolean isEmpty() | 查询该 Map 是否为空（即不包含任何 key-value 对），如果为空则返回 true。 |
| int size() | 返回该 Map 里 key-value 对的个数 |
| Collection values() | 返回该 Map 里所有 value 组成的 Collection |

`Map`集合最典型的用法就是成对地添加、删除`key-value`对，接下来即可判断该`Map`中是否包含指定`key`，也可以通过`Map`提供的`keySet()`方法获取所有`key`组成的集合，进而遍历`Map`中所有的`key-value`对。
## 例
每名学生都有属于自己的唯一编号，即学号。在毕业时需要将该学生的信息从系统中移除。使用`HashMap`来存储学生信息，其键为学生学号，值为姓名。毕业时，需要用户输入学生的学号，并根据学号进行删除操作。
```java
public class Test09 {
  public static void main(String[] args) {
    HashMap users = new HashMap();
    users.put("11", "张浩太"); // 将学生信息键值对存储到Map中
    users.put("22", "刘思诚");
    users.put("33", "王强文");
    users.put("44", "李国量");
    users.put("55", "王路路");
    System.out.println("******** 学生列表 ********");
    Iterator it = users.keySet().iterator();
    while (it.hasNext()) {
      // 遍历 Map
      Object key = it.next();
      Object val = users.get(key);
      System.out.println("学号：" + key + "，姓名:" + val);
    }
    Scanner input = new Scanner(System.in);
    System.out.println("请输入要删除的学号：");
    int num = input.nextInt();
    if (users.containsKey(String.valueOf(num))) { // 判断是否包含指定键
      users.remove(String.valueOf(num)); // 如果包含就删除
    } else {
      System.out.println("该学生不存在！");
    }
    System.out.println("******** 学生列表 ********");
    it = users.keySet().iterator();
    while (it.hasNext()) {
      Object key = it.next();
      Object val = users.get(key);
      System.out.println("学号：" + key + "，姓名：" + val);
    }
  }
}
```
在该程序中，两次使用`while`循环遍历`HashMap`集合。当有学生毕业时，用户需要输入该学生的学号，根据学号使用`HashMap`类的`remove()`方法将对应的元素删除。

> 注意：`TreeMap`类的使用方法与`HashMap`类相同，唯一不同的是`TreeMap`类可以对键对象进行排序。

## 遍历Map集合的四种方式
`Map`集合的遍历与`List`和`Set`集合不同。`Map`有两组值，因此遍历时可以只遍历值的集合，也可以只遍历键的集合，也可以同时遍历。`Map`以及实现`Map`的接口类（如`HashMap、TreeMap、LinkedHashMap、Hashtable`等）都可以用以下几种方式遍历。

### 1.在`for`循环中使用`entries`实现`Map`的遍历（最常见和最常用的）。
```java
public static void main(String[] args) {
  Map<String, String> map = new HashMap<String, String>();
  map.put("zhangsan", "张三");
  map.put("lisi", "李四");
  for (Map.Entry<String, String> entry : map.entrySet()) {
    String mapKey = entry.getKey();
    String mapValue = entry.getValue();
    System.out.println(mapKey + "：" + mapValue);
  }
}
```
### 2.使用`for-each`循环遍历`key`或者`values`，一般适用于只需要`Map`中的`key`或者`value`时使用。性能上比`entrySet`较好。
```java
Map<String, String> map = new HashMap<String, String>();
map.put("zhangsan", "张三");
map.put("lisi", "李四");
// 打印键集合
for (String key : map.keySet()) {
  System.out.println(key);
}
// 打印值集合
for (String value : map.values()) {
  System.out.println(value);
}
```
### 3.使用迭代器（Iterator）遍历
```java
Map<String, String> map = new HashMap<String, String>();
map.put("zhangsan", "张三");
map.put("lisi", "李四");
Iterator<Entry<String, String>> entries = map.entrySet().iterator();
while (entries.hasNext()) {
  Entry<String, String> entry = entries.next();
  String key = entry.getKey();
  String value = entry.getValue();
  System.out.println(key + ":" + value);
}
```
### 4.通过键找值遍历，这种方式的效率比较低，因为本身从键取值是耗时的操作。
```java
for(String key : map.keySet()){
  String value = map.get(key);
  System.out.println(key+":"+value);
}
```
# Collections类操作集合
`Collections`类是 Java 提供的一个操作`Set、List`和`Map`等集合的工具类。`Collections`类提供了许多操作集合的静态方法，借助这些静态方法可以实现集合元素的排序、查找替换和复制等操作。
## 排序（正向和逆向）
`Collections`提供了如下方法用于对`List`集合元素进行排序。
* `void reverse(List list)`：对指定`List`集合元素进行逆向排序。
* `void shuffle(List list)`：对`List`集合元素进行随机排序（`shuffle`方法模拟了“洗牌”动作）。
* `void sort(List list)`：根据元素的自然顺序对指定`List`集合的元素按升序进行排序。
* `void sort(List list, Comparator c)`：根据指定`Comparator`产生的顺序对`List`集合元素进行排序。
* `void swap(List list, int i, int j)`：将指定`List`集合中的`i`处元素和`j`处元素进行交换。
* `void rotate(List list, int distance)`：当`distance`为正数时，将`list`集合的后`distance`个元素“整体”移到前面；当`distance`为负数时，将`list`集合的前`distance`个元素“整体”移到后面。该方法不会改变集合的长度。

下面程序简单示范了利用`Collections`工具类来操作`List`集合。

编写一个程序，对用户输入的 5 个商品价格进行排序后输出。这里要求使用`Collections`类中`sort()`方法按从低到高的顺序对其进行排序，最后将排序后的成绩输出。

具体实现代码如下：
```java
public class Test1 {
  public static void main(String[] args) {
    Scanner input = new Scanner(System.in);
    List prices = new ArrayList();
    for (int i = 0; i < 5; i++) {
      System.out.println("请输入第 " + (i + 1) + " 个商品的价格：");
      int p = input.nextInt();
      prices.add(Integer.valueOf(p)); // 将录入的价格保存到List集合中
    }
    Collections.sort(prices); // 调用sort()方法对集合进行排序
    System.out.println("价格从低到高的排列为：");
    for (int i = 0; i < prices.size(); i++) {
      System.out.print(prices.get(i) + "\t");
    }
  }
}
```
如上述代码，循环录入 5 个价格，并将每个价格都存储到已定义好的`List`集合`prices`中，然后使用`Collections`类的`sort()`方法对该集合元素进行升序排序。最后使用`for`循环遍历`users`集合，输出该集合中的元素。

该程序的执行结果如下所示。
```
请输入第 1 个商品的价格：
85
请输入第 2 个商品的价格：
48
请输入第 3 个商品的价格：
66
请输入第 4 个商品的价格：
80
请输入第 5 个商品的价格：
18
价格从低到高的排列为：
18    48    66    80    85
```
循环录入 5 个商品的名称，并按录入时间的先后顺序进行降序排序，即后录入的先输出。

下面编写程序，使用`Collections`类的`reverse()`方法对保存到`List`集合中的 5 个商品名称进行反转排序，并输出排序后的商品信息。具体的实现代码如下：
```java
public class Test2 {
  public static void main(String[] args) {
    Scanner input = new Scanner(System.in);
    List students = new ArrayList();
    System.out.println("******** 商品信息 ********");
    for (int i = 0; i < 5; i++) {
      System.out.println("请输入第 " + (i + 1) + " 个商品的名称：");
      String name = input.next();
      students.add(name); // 将录入的商品名称存到List集合中
    }
    Collections.reverse(students); // 调用reverse()方法对集合元素进行反转排序
    System.out.println("按录入时间的先后顺序进行降序排列为：");
    for (int i = 0; i < 5; i++) {
      System.out.print(students.get(i) + "\t");
    }
  }
}
```
如上述代码，首先循环录入 5 个商品的名称，并将这些名称保存到`List`集合中，然后调用`Collections`类中的`reverse()`方法对该集合元素进行反转排序。最后使用`for`循环将排序后的集合元素输出。

执行该程序，输出结果如下所示。
```
******** 商品信息 ********
请输入第 1 个商品的名称：
果粒橙
请输入第 2 个商品的名称：
冰红茶
请输入第 3 个商品的名称：
矿泉水
请输入第 4 个商品的名称：
软面包
请输入第 5 个商品的名称：
巧克力
按录入时间的先后顺序进行降序排列为：
巧克力    软面包    矿泉水    冰红茶    果粒橙   
```
## 查找、替换操作
`Collections`还提供了如下常用的用于查找、替换集合元素的方法。
* `int binarySearch(List list, Object key)`：使用二分搜索法搜索指定的`List`集合，以获得指定对象在`List`集合中的索引。如果要使该方法可以正常工作，则必须保证`List`中的元素已经处于有序状态。
* `Object max(Collection coll)`：根据元素的自然顺序，返回给定集合中的最大元素。
* `Object max(Collection coll, Comparator comp)`：根据`Comparator`指定的顺序，返回给定集合中的最大元素。
* `Object min(Collection coll)`：根据元素的自然顺序，返回给定集合中的最小元素。
* `Object min(Collection coll, Comparator comp)`：根据`Comparator`指定的顺序，返回给定集合中的最小元素。
* `void fill(List list, Object obj)`：使用指定元素`obj`替换指定`List`集合中的所有元素。
* `int frequency(Collection c, Object o)`：返回指定集合中指定元素的出现次数。
* `int indexOfSubList(List source, List target)`：返回子`List`对象在父`List`对象中第一次出现的位置索引；如果父`List` 中没有出现这样的子`List`，则返回 -1。
* `int lastIndexOfSubList(List source, List target)`：返回子`List`对象在父`List`对象中最后一次出现的位置索引；如果父`List`中没有岀现这样的子`List`，则返回 -1。
* `boolean replaceAll(List list, Object oldVal, Object newVal)`：使用一个新值`newVal`替换`List`对象的所有旧值`oldVal`。

下面程序简单示范了`Collections`工具类的用法。

编写一个程序，要求用户输入 3 个商品名称，然后使用`Collections`类中的`fill()`方法对商品信息进行重置操作，即将所有名称都更改为“未填写”。具体的实现代码如下：
```java
public class Test3 {
  public static void main(String[] args) {
    Scanner input = new Scanner(System.in);
    List products = new ArrayList();
    System.out.println("******** 商品信息 ********");
    for (int i = 0; i < 3; i++) {
      System.out.println("请输入第 " + (i + 1) + " 个商品的名称：");
      String name = input.next();
      products.add(name); // 将用户录入的商品名称保存到List集合中
    }
    System.out.println("重置商品信息，将所有名称都更改为'未填写'");
    Collections.fill(products, "未填写");
    System.out.println("重置后的商品信息为：");
    for (int i = 0; i < products.size(); i++) {
      System.out.print(products.get(i) + "\t");
    }
  }
}
```
如上述代码，首先循环录入 3 个商品名称，并将这些商品信息存储到`List`集合中，然后调用`Collections`类中的`fill()`方法将该集合中的所有元素值替换为“未填写”。最后使用`for`循环将替换后的集合元素输出。

运行该程序，执行结果如下所示。
```
******** 商品信息 ********
请输入第 1 个商品的名称：
苏打水
请输入第 2 个商品的名称：
矿泉水
请输入第 3 个商品的名称：
冰红茶
重置商品信息，将所有名称都更改为'未填写'
重置后的商品信息为：
未填写    未填写    未填写    
```
在一个集合中保存 4 个数据，分别输出最大最小元素和指定数据在集合中出现的次数。
```java
public class Test4 {
  public static void main(String[] args) {
    ArrayList nums = new ArrayList();
    nums.add(2);
    nums.add(-5);
    nums.add(3);
    nums.add(0);
    System.out.println(nums); // 输出：[2, -5, 3, 0]
    System.out.println(Collections.max(nums)); // 输出最大元素，将输出 3
    System.out.println(Collections.min(nums)); // 输出最小元素，将输出-5
    Collections.replaceAll(nums, 0, 1);// 将 nums中的 0 使用 1 来代替
    System.out.println(nums); // 输出：[2, -5, 3, 1]
    // 判断-5在List集合中出现的次数，返回1
    System.out.println(Collections.frequency(nums, -5));
    Collections.sort(nums); // 对 nums集合排序
    System.out.println(nums); // 输出：[-5, 1, 2, 3]
    // 只有排序后的List集合才可用二分法查询，输出3
    System.out.println(Collections.binarySearch(nums, 3));
  }
}
```
如上述代码，向`List`集合中添加 4 个数据，然后调用`Collections`类中的`max()`和`min()`方法输出集合中的最大最小元素，`replaceAll()`替换元素，`frequency()`判断指定数据在`List`集合中出现的次数，最后用`binarySearch()`进行二分法查询。

运行上述程序，执行结果如下：
```
[2, -5, 3, 0]
3
-5
[2, -5, 3, 1]
1
[-5, 1, 2, 3]
3
```
`Collections`类的`copy()`静态方法用于将指定集合中的所有元素复制到另一个集合中。执行`copy()`方法后，目标集合中每个已复制元素的索引将等同于源集合中该元素的索引。

`copy()`方法的语法格式如下：
```java
void copy(List <? super T> dest,List<? extends T> src)
```
其中，`dest`表示目标集合对象，`src`表示源集合对象。

注意：目标集合的长度至少和源集合的长度相同，如果目标集合的长度更长，则不影响目标集合中的其余元素。如果目标集合长度不够而无法包含整个源集合元素，程序将抛出`IndexOutOfBoundsException`异常。

在一个集合中保存了 5 个商品名称，现在要使用`Collections`类中的`copy()`方法将其中的 3 个替换掉。具体实现的代码如下：
```java
public class Test5 {
  public static void main(String[] args) {
    Scanner input = new Scanner(System.in);
    List srcList = new ArrayList();
    List destList = new ArrayList();
    destList.add("苏打水");
    destList.add("木糖醇");
    destList.add("方便面");
    destList.add("火腿肠");
    destList.add("冰红茶");
    System.out.println("原有商品如下：");
    for (int i = 0; i < destList.size(); i++) {
      System.out.println(destList.get(i));
    }
    System.out.println("输入替换的商品名称：");
    for (int i = 0; i < 3; i++) {
      System.out.println("第 " + (i + 1) + " 个商品：");
      String name = input.next();
      srcList.add(name);
    }
    // 调用copy()方法将当前商品信息复制到原有商品信息集合中
    Collections.copy(destList, srcList);
    System.out.println("当前商品有：");
    for (int i = 0; i < destList.size(); i++) {
      System.out.print(destList.get(i) + "\t");
    }
  }
}
```
如上述代码，首先创建了两个`List`对象`srcList`和`destList`，并向`destList`集合中添加了 5 个元素，向`srcList`集合中添加了 3 个元素，然后调用`Collections`类中`copy()`方法将`srcList`集合中的全部元素复制到`destList`集合中。由于`destList`集合中含有 5 个元素，故最后两个元素不会被覆盖。

运行该程序，具体的执行结果如下所示。
```
原有商品如下：
苏打水
木糖醇
方便面
火腿肠
冰红茶
输入替换的商品名称：
第 1 个商品：
燕麦片
第 2 个商品：
八宝粥
第 3 个商品：
软面包
当前商品有：
燕麦片    八宝粥    软面包    火腿肠    冰红茶
```
# Iterator（迭代器）遍历Collection集合元素
`Iterator`（迭代器）是一个接口，它的作用就是遍历容器的所有元素，也是 Java 集合框架的成员，但它与`Collection`和`Map`系列的集合不一样，`Collection`和`Map`系列集合主要用于盛装其他对象，而`Iterator`则主要用于遍历（即迭代访问）`Collection`集合中的元素。

`Iterator`接口隐藏了各种`Collection`实现类的底层细节，向应用程序提供了遍历`Collection`集合元素的统一编程接口。`Iterator`接口里定义了如下 4 个方法。
* `boolean hasNext()`：如果被迭代的集合元素还没有被遍历完，则返回`true`。
* `Object next()`：返回集合里的下一个元素。
* `void remove()`：删除集合里上一次`next`方法返回的元素。
* `void forEachRemaining(Consumer action)`：这是 Java 8 为`Iterator`新增的默认方法，该方法可使用`Lambda`表达式来遍历集合元素。

下面程序示范了通过`Iterator`接口来遍历集合元素。
```java
import java.util.Collection;
import java.util.HashSet;
import java.util.Iterator;
public class IteratorTest {
  public static void main(String[] args) {
    // 创建一个集合
    Collection objs = new HashSet();
    objs.add("C语言中文网Java教程");
    objs.add("C语言中文网C语言教程");
    objs.add("C语言中文网C++教程");
    // 调用forEach()方法遍历集合
    // 获取books集合对应的迭代器
    Iterator it = objs.iterator();
    while (it.hasNext()) {
      // it.next()方法返回的数据类型是Object类型，因此需要强制类型转换
      String obj = (String) it.next();
      System.out.println(obj);
      if (obj.equals("C语言中文网C语言教程")) {
          // 从集合中删除上一次next()方法返回的元素
          it.remove();
      }
      // 对book变量赋值，不会改变集合元素本身
      obj = "C语言中文网Python语言教程";
    }
    System.out.println(objs);
  }
}
```
从上面代码中可以看出，`Iterator`仅用于遍历集合，如果需要创建`Iterator`对象，则必须有一个被迭代的集合。没有集合的`Iterator`没有存在的价值。

注意：`Iterator`必须依附于`Collection`对象，若有一个`Iterator`对象，则必然有一个与之关联的`Collection`对象。`Iterator`提供了两个方法来迭代访问`Collection`集合里的元素，并可通过 remove() 方法来删除集合中上一次 next() 方法返回的集合元素。

上面程序中第 24 行代码对迭代变量`obj`进行赋值，但当再次输岀`objs`集合时，会看到集合里的元素没有任何改变。所以当使用`Iterator`对集合元素进行迭代时，`Iterator`并不是把集合元素本身传给了迭代变量，而是把集合元素的值传给了迭代变量，所以修改迭代变量的值对集合元素本身没有任何影响。

当使用`Iterator`迭代访问`Collection`集合元素时，`Collection`集合里的元素不能被改变，只有通过`Iterator`的`remove()`方法删除上一次`next()`方法返回的集合元素才可以，否则将会引发`“java.util.ConcurrentModificationException”`异常。下面程序示范了这一点。
```java
public class IteratorErrorTest {
  public static void main(String[] args) {
    // 创建一个集合
    Collection objs = new HashSet();
    objs.add("C语言中文网Java教程");
    objs.add("C语言中文网C语言教程");
    objs.add("C语言中文网C++教程");
    // 获取books集合对应的迭代器
    Iterator it = objs.iterator();
    while (it.hasNext()) {
      String obj = (String) it.next();
      System.out.println(obj);
      if (obj.equals("C语言中文网C++教程")) {
        // 使用Iterator迭代过程中，不可修改集合元素，下面代码引发异常
        objs.remove(obj);
      }
    }
  }
}
```
输出结果为：
```
C语言中文网C++教程
Exception in thread "main" java.util.ConcurrentModificationException
        at java.util.HashMap$HashIterator.nextNode(Unknown Source)
        at java.util.HashMap$KeyIterator.next(Unknown Source)
        at IteratorErrorTest.main(IteratorErrorTest.java:15)
```
上面程序中第 15 行代码位于`Iterator`迭代块内，也就是在`Iterator`迭代`Collection`集合过程中修改了`Collection`集合，所以程序将在运行时引发异常。

Iterator 迭代器采用的是快速失败（fail-fast）机制，一旦在迭代过程中检测到该集合已经被修改（通常是程序中的其他线程修改），程序立即引发 ConcurrentModificationException 异常，而不是显示修改后的结果，这样可以避免共享资源而引发的潜在问题。
快速失败（fail-fast）机制，是 Java Collection 集合中的一种错误检测机制。

注意：上面程序如果改为删除“C语言中文网C语言教程”字符串，则不会引发异常。这样可能有些读者会“心存侥幸”地想，在迭代时好像也可以删除集合元素啊。实际上这是一种危险的行为。对于 HashSet 以及后面的 ArrayList 等，迭代时删除元素都会导致异常。只有在删除集合中的某个特定元素时才不会抛出异常，这是由集合类的实现代码决定的，程序员不应该这么做。
# 使用Lambda表达式遍历Collection集合
Java 8 为 Iterable 接口新增了一个`forEach(Consumer action)`默认方法，该方法所需参数的类型是一个函数式接口，而`Iterable`接口是 Collection 接口的父接口，因此 Collection 集合也可直接调用该方法。

当程序调用`Iterable`的`forEach(Consumer action)`遍历集合元素时，程序会依次将集合元素传给`Consumer`的`accept(T t)`方法（该接口中唯一的抽象方法）。正因为`Consumer`是函数式接口，因此可以使用`Lambda`表达式来遍历集合元素。

如下程序示范了使用`Lambda`表达式来遍历集合元素。
```java
public class CollectionEach {
  public static void main(String[] args) {
    // 创建一个集合
    Collection objs = new HashSet();
    objs.add("C语言中文网Java教程");
    objs.add("C语言中文网C语言教程");
    objs.add("C语言中文网C++教程");
    // 调用forEach()方法遍历集合
    objs.forEach(obj -> System.out.println("迭代集合元素：" + obj));
  }
}
```
输出结果为：
```
迭代集合元素：C语言中文网C++教程
迭代集合元素：C语言中文网C语言教程
迭代集合元素：C语言中文网Java教程
```
上面程序中粗体字代码调用了`Iterable`的`forEach()`默认方法来遍历集合元素，传给该方法的参数是一个 Lambda 表达式，该 Lambda 表达式的目标类型是 Comsumer。forEach() 方法会自动将集合元素逐个地传给 Lambda 表达式的形参，这样 Lambda 表达式的代码体即可遍历到集合元素了。
# 使用Lambda表达式遍历Iterator迭代器
Java 8 为 Iterator 引入了一个`forEachRemaining(Consumer action)`默认方法，该方法所需的`Consumer`参数同样也是函数式接口。当程序调用 Iterator 的`forEachRemaining(Consumer action)`遍历集合元素时，程序会依次将集合元素传给`Consumer`的`accept(T t)`方法（该接口中唯一的抽象方法）。
`java.util.function`中的`Function、Supplier、Consumer、Predicate`和其他函数式接口被广泛用在支持`Lambda`表达式的 API 中。`“void accept(T t);”`是`Consumer`的核心方法，用来对给定的参数`T`执行定义操作。

如下程序示范了使用 Lambda 表达式来遍历集合元素。
```java
public class IteratorEach {
  public static void main(String[] args) {
    // 创建一个集合
    Collection objs = new HashSet();
    objs.add("aaa");
    objs.add("bbb");
    objs.add("ccc++教程");
    // 获取objs集合对应的迭代器
    Iterator it = objs.iterator();
    // 使用Lambda表达式（目标类型是Comsumer）来遍历集合元素
    it.forEachRemaining(obj -> System.out.println("迭代集合元素：" + obj));
  }
}
```
上面程序中第 11 行代码调用了`Iterator`的`forEachRemaining()`方法来遍历集合元素，传给该方法的参数是一个`Lambda`表达式，该`Lambda`表达式的目标类型是`Consumer`，因此上面代码也可用于遍历集合元素。
# 使用foreach循环遍历Collection集合
```java
public class ForeachTest {
  public static void main(String[] args) {
    // 创建一个集合
    Collection objs = new HashSet();
    objs.add("C语言中文网Java教程");
    objs.add("C语言中文网C语言教程");
    objs.add("C语言中文网C++教程");
    for (Object obj : objs) {
        // 此处的obj变量也不是集合元素本身
        String obj1 = (String) obj;
        System.out.println(obj1);
        if (obj1.equals("C语言中文网Java教程")) {
            // 下面代码会引发 ConcurrentModificationException 异常
            objs.remove(obj);
        }
    }
    System.out.println(objs);
  }
}
```
输出结果为：
```
C语言中文网C++教程
C语言中文网C语言教程
C语言中文网Java教程
[C语言中文网C++教程, C语言中文网C语言教程]
```
上面代码使用`foreach`循环来迭代访问`Collection`集合里的元素更加简洁。与使用`Iterator`接口迭代访问集合元素类似的是，`foreach`循环中的迭代变量也不是集合元素本身，系统只是依次把集合元素的值赋给迭代变量，因此在`foreach`循环中修改迭代变量的值也没有任何实际意义。

同样，当使用`foreach`循环迭代访问集合元素时，该集合也不能被改变，否则将引发`ConcurrentModificationException`异常。所以上面程序中第 14 行代码处将引发该异常。
