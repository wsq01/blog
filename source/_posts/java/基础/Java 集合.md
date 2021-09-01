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
| Iterator 接口 | 集合的输出接口，主要用于遍历输出（即迭代访问）Collection 集合中的元素，Iterator 对象被称之为迭代器。迭代器接口是集合接口的父接口，实现类实现 Collection 时就必须实现 Iterator 接口。 |
| Collection 接口 | 是 List、Set 和 Queue 的父接口，是存放一组单值的最大接口。所谓的单值是指集合中的每个元素都是一个对象。一般很少直接使用此接口直接操作。 |
| Queue 接口 | Queue 是 Java 提供的队列实现，有点类似于 List。 |
| Dueue 接口 | 是 Queue 的一个子接口，为双向队列。 |
| List 接口 | 是最常用的接口。是有序集合，允许有相同的元素。使用 List 能够精确地控制每个元素插入的位置，用户能够使用索引（元素在 List 中的位置，类似于数组下标）来访问 List 中的元素，与数组类似。 |
| Set 接口 | 不能包含重复的元素。 |
| Map 接口 | 是存放一对值的最大接口，即接口中的每个元素都是一对，以 key➡value 的形式保存。 |

对于`Set、List、Queue`和`Map`这 4 种集合，Java 最常用的实现类分别是`HashSet、TreeSet、ArrayList、ArrayDueue、LinkedList`和`HashMap、TreeMap`等。

| 类名称 | 作    用 |
| :--: | :--: |
| HashSet | 为优化査询速度而设计的 Set。它是基于 HashMap 实现的，HashSet 底层使用 HashMap 来保存所有元素，实现比较简单 |
| TreeSet | 实现了 Set 接口，是一个有序的 Set，这样就能从 Set 里面提取一个有序序列 |
| ArrayList | 一个用数组实现的 List，能进行快速的随机访问，效率高而且实现了可变大小的数组 |
| ArrayDueue | 是一个基于数组实现的双端队列，按“先进先出”的方式操作集合元素 |
| LinkedList | 对顺序访问进行了优化，但随机访问的速度相对较慢。此外它还有 addFirst()、addLast()、getFirst()、getLast()、removeFirst() 和 removeLast() 等方法，能把它当成栈（Stack）或队列（Queue）来用 |
| HsahMap | 按哈希算法来存取键对象 |
| TreeMap | 可以对键对象进行排序 |

# Collection接口
`Collection`接口是`List、Set`和`Queue`接口的父接口，通常情况下不被直接使用。`Collection`接口定义了一些通用的方法，通过这些方法可以实现对集合的基本操作。定义的方法既可用于操作`Set`集合，也可用于操作`List`和`Queue`集合。

| 方法名称 | 说明 |
| :--: | :--: |
| boolean add(E e) | 向集合中添加一个元素，如果集合对象被添加操作改变了，则返回 true。E 是元素的数据类型 |
| boolean addAll(Collection c) | 向集合中添加集合 c 中的所有元素，如果集合对象被添加操作改变了，则返回 true。 |
| void clear() | 清除集合中的所有元素，将集合长度变为 0。 |
| boolean contains(Object o) | 判断集合中是否存在指定元素 |
| boolean containsAll(Collection c) | 判断集合中是否包含集合 c 中的所有元素 |
| boolean isEmpty() | 判断集合是否为空 |
| `Iterator<E>iterator()` | 返回一个 Iterator 对象，用于遍历集合中的元素 |
| boolean remove(Object o) | 从集合中删除一个指定元素，当集合中包含了一个或多个元素 o 时，该方法只删除第一个符合条件的元素，该方法将返回 true。 |
| boolean removeAll(Collection c) | 从集合中删除所有在集合 c 中出现的元素（相当于把调用该方法的集合减去集合 c）。如果该操作改变了调用该方法的集合，则该方法返回 true。 |
| boolean retainAll(Collection c) | 从集合中删除集合 c 里不包含的元素（相当于把调用该方法的集合变成该集合和集合 c 的交集），如果该操作改变了调用该方法的集合，则该方法返回 true。 |
| int size() | 返回集合中元素的个数 |
| Object[] toArray() | 把集合转换为一个数组，所有的集合元素变成对应的数组元素。 |

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

| 方法名称 | 说明 |
| :--: | :--: |
| `void addFirst(E e)` | 将指定元素添加到此集合的开头 |
| `void addLast(E e)`  | 将指定元素添加到此集合的末尾 |
| `E getFirst()`       | 返回此集合的第一个元素       |
| `E getLast()`        | 返回此集合的最后一个元素     |
| `E removeFirst()`	   | 删除此集合中的第一个元素     |
| `E removeLast()`     | 删除此集合中的最后一个元素   |

# ArrayList 类和 LinkedList 类的区别
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

当向`HashSet`集合中存入一个元素时，`HashSet`会调用该对象的`hashCode()`方法来得到该对象的`hashCode`值，然后根据该`hashCode`值决定该对象在`HashSet`中的存储位置。如果有两个元素通过`equals()`方法比较返回的结果为`true`，但它们的`hashCode`不相等，HashSet 将会把它们存储在不同的位置，依然可以添加成功。

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

`TreeSet`只能对实现了 Comparable 接口的类对象进行排序，因为 Comparable 接口中有一个`compareTo(Object o)`方法用于比较两个对象的大小。例如`a.compareTo(b)`，如果 a 和 b 相等，则该方法返回 0；如果 a 大于 b，则该方法返回大于 0 的值；如果 a 小于 b，则该方法返回小于 0 的值。

表 1 列举了 JDK 类库中实现 Comparable 接口的类，以及这些类对象的比较方式。

表 1 实现Comparable接口类对象的比较方式
类	比较方式
包装类（BigDecimal、Biglnteger、 Byte、Double、
Float、Integer、Long 及 Short)	按数字大小比较
Character	按字符的 Unicode 值的数字大小比较
String	按字符串中字符的 Unicode 值的数字大小比较
TreeSet 类除了实现 Collection 接口的所有方法之外，还提供了如表 2 所示的方法。

表 2 TreeSet类的常用方法
方法名称	说明
E first()	返回此集合中的第一个元素。其中，E 表示集合中元素的数据类型
E last()	返回此集合中的最后一个元素
E poolFirst()	获取并移除此集合中的第一个元素
E poolLast()	获取并移除此集合中的最后一个元素
SortedSet<E> subSet(E fromElement,E toElement)	返回一个新的集合，新集合包含原集合中 fromElement 对象与 toElement
对象之间的所有对象。包含 fromElement 对象，不包含 toElement 对象
SortedSet<E> headSet<E toElement〉	返回一个新的集合，新集合包含原集合中 toElement 对象之前的所有对象。
不包含 toElement 对象
SortedSet<E> tailSet(E fromElement)	返回一个新的集合，新集合包含原集合中 fromElement 对象之后的所有对
象。包含 fromElement 对象注意：表面上看起来这些方法很多，其实很简单。因为 TreeSet 中的元素是有序的，所以增加了访问第一个、前一个、后一个、最后一个元素的方法，并提供了 3 个从 TreeSet 中截取子 TreeSet 的方法。

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
每名学生都有属于自己的唯一编号，即学号。在毕业时需要将该学生的信息从系统中移除。下面编写 Java 程序，使用`HashMap`来存储学生信息，其键为学生学号，值为姓名。毕业时，需要用户输入学生的学号，并根据学号进行删除操作。具体的实现代码如下：
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
在该程序中，两次使用`while`循环遍历`HashMap`集合。当有学生毕业时，用户需要输入该学生的学号，根据学号使用`HashMap`类的`remove()`方法将对应的元素删除。程序运行结果如下所示。
```
******** 学生列表 ********
学号：44，姓名:李国量
学号：55，姓名:王路路
学号：22，姓名:刘思诚
学号：33，姓名:王强文
学号：11，姓名:张浩太
请输入要删除的学号：
22
******** 学生列表 ********
学号：44，姓名：李国量
学号：55，姓名：王路路
学号：33，姓名：王强文
学号：11，姓名：张浩太
******** 学生列表 ********
学号：44，姓名:李国量
学号：55，姓名:王路路
学号：22，姓名:刘思诚
学号：33，姓名:王强文
学号：11，姓名:张浩太
请输入要删除的学号：
44
******** 学生列表 ********
学号：55，姓名：王路路
学号：22，姓名：刘思诚
学号：33，姓名：王强文
学号：11，姓名：张浩太
```
注意：`TreeMap`类的使用方法与`HashMap`类相同，唯一不同的是`TreeMap`类可以对键对象进行排序。
# 遍历Map集合的四种方式
`Map`集合的遍历与`List`和`Set`集合不同。`Map`有两组值，因此遍历时可以只遍历值的集合，也可以只遍历键的集合，也可以同时遍历。`Map`以及实现`Map`的接口类（如`HashMap、TreeMap、LinkedHashMap、Hashtable`等）都可以用以下几种方式遍历。

1）在`for`循环中使用`entries`实现`Map`的遍历（最常见和最常用的）。
```java
public static void main(String[] args) {
  Map<String, String> map = new HashMap<String, String>();
  map.put("Java入门教程", "http://c.biancheng.net/java/");
  map.put("C语言入门教程", "http://c.biancheng.net/c/");
  for (Map.Entry<String, String> entry : map.entrySet()) {
    String mapKey = entry.getKey();
    String mapValue = entry.getValue();
    System.out.println(mapKey + "：" + mapValue);
  }
}
```
2）使用`for-each`循环遍历`key`或者`values`，一般适用于只需要`Map`中的`key`或者`value`时使用。性能上比`entrySet`较好。
```java
Map<String, String> map = new HashMap<String, String>();
map.put("Java入门教程", "http://c.biancheng.net/java/");
map.put("C语言入门教程", "http://c.biancheng.net/c/");
// 打印键集合
for (String key : map.keySet()) {
  System.out.println(key);
}
// 打印值集合
for (String value : map.values()) {
  System.out.println(value);
}
```
3）使用迭代器（Iterator）遍历
```java
Map<String, String> map = new HashMap<String, String>();
map.put("Java入门教程", "http://c.biancheng.net/java/");
map.put("C语言入门教程", "http://c.biancheng.net/c/");
Iterator<Entry<String, String>> entries = map.entrySet().iterator();
while (entries.hasNext()) {
  Entry<String, String> entry = entries.next();
  String key = entry.getKey();
  String value = entry.getValue();
  System.out.println(key + ":" + value);
}
```
4）通过键找值遍历，这种方式的效率比较低，因为本身从键取值是耗时的操作。
```java
for(String key : map.keySet()){
  String value = map.get(key);
  System.out.println(key+":"+value);
}
```