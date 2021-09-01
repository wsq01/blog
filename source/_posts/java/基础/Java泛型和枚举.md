


前面我们提到 Java 集合有个缺点，就是把一个对象“丢进”集合里之后，集合就会“忘记”这个对象的数据类型，当再次取出该对象时，该对象的编译类型就变成了 Object 类型（其运行时类型没变）。

Java 集合之所以被设计成这样，是因为集合的设计者不知道我们会用集合来保存什么类型的对象，所以他们把集合设计成能保存任何类型的对象，只要求具有很好的通用性，但这样做带来如下两个问题：
集合对元素类型没有任何限制，这样可能引发一些问题。例如，想创建一个只能保存 Dog 对象的集合，但程序也可以轻易地将 Cat 对象“丢”进去，所以可能引发异常。
由于把对象“丢进”集合时，集合丢失了对象的状态信息，集合只知道它盛装的是 Object，因此取出集合元素后通常还需要进行强制类型转换。这种强制类型转换既增加了编程的复杂度，也可能引发 ClassCastException 异常。

所以为了解决上述问题，从 Java 1.5 开始提供了泛型。泛型可以在编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，提高了代码的重用率。本节将详细介绍 Java 中泛型的使用。
泛型集合
泛型本质上是提供类型的“类型参数”，也就是参数化类型。我们可以为类、接口或方法指定一个类型参数，通过这个参数限制操作的数据类型，从而保证类型转换的绝对安全。
例 1
下面将结合泛型与集合编写一个案例实现图书信息输出。

1）首先需要创建一个表示图书的实体类 Book，其中包括的图书信息有图书编号、图书名称和价格。Book 类的具体代码如下：
public class Book {
    private int Id; // 图书编号
    private String Name; // 图书名称
    private int Price; // 图书价格
    public Book(int id, String name, int price) { // 构造方法
        this.Id = id;
        this.Name = name;
        this.Price = price;
    }
    public String toString() { // 重写 toString()方法
        return this.Id + ", " + this.Name + "，" + this.Price;
    }
}
2）使用 Book 作为类型创建 Map 和 List 两个泛型集合，然后向集合中添加图书元素，最后输出集合中的内容。具体代码如下：
public class Test14 {
    public static void main(String[] args) {
        // 创建3个Book对象
        Book book1 = new Book(1, "唐诗三百首", 8);
        Book book2 = new Book(2, "小星星", 12);
        Book book3 = new Book(3, "成语大全", 22);
        Map<Integer, Book> books = new HashMap<Integer, Book>(); // 定义泛型 Map 集合
        books.put(1001, book1); // 将第一个 Book 对象存储到 Map 中
        books.put(1002, book2); // 将第二个 Book 对象存储到 Map 中
        books.put(1003, book3); // 将第三个 Book 对象存储到 Map 中
        System.out.println("泛型Map存储的图书信息如下：");
        for (Integer id : books.keySet()) {
            // 遍历键
            System.out.print(id + "——");
            System.out.println(books.get(id)); // 不需要类型转换
        }
        List<Book> bookList = new ArrayList<Book>(); // 定义泛型的 List 集合
        bookList.add(book1);
        bookList.add(book2);
        bookList.add(book3);
        System.out.println("泛型List存储的图书信息如下：");
        for (int i = 0; i < bookList.size(); i++) {
            System.out.println(bookList.get(i)); // 这里不需要类型转换
        }
    }
}
在该示例中，第 7 行代码创建了一个键类型为 Integer、值类型为 Book 的泛型集合，即指明了该 Map 集合中存放的键必须是 Integer 类型、值必须为 Book 类型，否则编译出错。在获取 Map 集合中的元素时，不需要将books.get(id);获取的值强制转换为 Book 类型，程序会隐式转换。在创建 List 集合时，同样使用了泛型，因此在获取集合中的元素时也不需要将bookList.get(i)代码强制转换为 Book 类型，程序会隐式转换。

执行结果如下：
泛型Map存储的图书信息如下：
1001——1, 唐诗三百首，8
1003——3, 成语大全，22
1002——2, 小星星，12
泛型List存储的图书信息如下：
1, 唐诗三百首，8
2, 小星星，12
3, 成语大全，22
泛型类
除了可以定义泛型集合之外，还可以直接限定泛型类的类型参数。语法格式如下：
public class class_name<data_type1,data_type2,…>{}
其中，class_name 表示类的名称，data_ type1 等表示类型参数。Java 泛型支持声明一个以上的类型参数，只需要将类型用逗号隔开即可。

泛型类一般用于类中的属性类型不确定的情况下。在声明属性时，使用下面的语句：
private data_type1 property_name1;
private data_type2 property_name2;
该语句中的 data_type1 与类声明中的 data_type1 表示的是同一种数据类型。
例 2
在实例化泛型类时，需要指明泛型类中的类型参数，并赋予泛型类属性相应类型的值。例如，下面的示例代码创建了一个表示学生的泛型类，该类中包括 3 个属性，分别是姓名、年龄和性别。
public class Stu<N, A, S> {
    private N name; // 姓名
    private A age; // 年龄
    private S sex; // 性别
    // 创建类的构造函数
    public Stu(N name, A age, S sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }
    // 下面是上面3个属性的setter/getter方法
    public N getName() {
        return name;
    }
    public void setName(N name) {
        this.name = name;
    }
    public A getAge() {
        return age;
    }
    public void setAge(A age) {
        this.age = age;
    }
    public S getSex() {
        return sex;
    }
    public void setSex(S sex) {
        this.sex = sex;
    }
}
接着创建测试类。在测试类中调用 Stu 类的构造方法实例化 Stu 对象，并给该类中的 3 个属性赋予初始值，最终需要输出学生信息。测试类的代码实现如下：
public class Test14 {
    public static void main(String[] args) {
        Stu<String, Integer, Character> stu = new Stu<String, Integer, Character>("张晓玲", 28, '女');
        String name = stu.getName();
        Integer age = stu.getAge();
        Character sex = stu.getSex();
        System.out.println("学生信息如下：");
        System.out.println("学生姓名：" + name + "，年龄：" + age + "，性别：" + sex);
    }
}
该程序的运行结果如下：
学生信息如下：
学生姓名：张晓玲，年龄：28，性别：女

在该程序的 Stu 类中，定义了 3 个类型参数，分别使用 N、A 和 S 来代替，同时实现了这 3 个属性的 setter/getter 方法。在主类中，调用 Stu 类的构造函数创建了 Stu 类的对象，同时指定 3 个类型参数，分别为 String、Integer 和 Character。在获取学生姓名、年龄和性别时，不需要类型转换，程序隐式地将 Object 类型的数据转换为相应的数据类型。
泛型方法
到目前为止，我们所使用的泛型都是应用于整个类上。泛型同样可以在类中包含参数化的方法，而方法所在的类可以是泛型类，也可以不是泛型类。也就是说，是否拥有泛型方法，与其所在的类是不是泛型没有关系。

泛型方法使得该方法能够独立于类而产生变化。如果使用泛型方法可以取代类泛型化，那么就应该只使用泛型方法。另外，对一个 static 的方法而言，无法访问泛型类的类型参数。因此，如果 static 方法需要使用泛型能力，就必须使其成为泛型方法。

定义泛型方法的语法格式如下：
[访问权限修饰符] [static] [final] <类型参数列表> 返回值类型 方法名([形式参数列表])
例如：
public static <T> List find(Class<T> cs,int userId){}

一般来说编写 Java 泛型方法，其返回值类型至少有一个参数类型应该是泛型，而且类型应该是一致的，如果只有返回值类型或参数类型之一使用了泛型，那么这个泛型方法的使用就被限制了。下面就来定义一个泛型方法，具体介绍泛型方法的创建和使用。
例 3
使用泛型方法打印图书信息。定义泛型方法，参数类型使用“T”来代替。在方法的主体中打印出图书信息。代码的实现如下：
public class Test16 {
    public static <T> void List(T book) { // 定义泛型方法
        if (book != null) {
            System.out.println(book);
        }
    }
    public static void main(String[] args) {
        Book stu = new Book(1, "细学 Java 编程", 28);
        List(stu); // 调用泛型方法
    }
}
该程序中的 Book 类为前面示例中使用到的 Book 类。在该程序中定义了一个名称为 List 的方法，该方法的返回值类型为 void，类型参数使用“T”来代替。在调用该泛型方法时，将一个 Book 对象作为参数传递到该方法中，相当于指明了该泛型方法的参数类型为 Book。

该程序的运行结果如下：
1, 细学 Java 编程，28
泛型的高级用法
泛型的用法非常灵活，除在集合、类和方法中使用外，本节将从三个方面介绍泛型的高级用法，包括限制泛型可用类型、使用类型通配符、继承泛型类和实现泛型接口。
1. 限制泛型可用类型
在 Java 中默认可以使用任何类型来实例化一个泛型类对象。当然也可以对泛型类实例的类型进行限制，语法格式如下：
class 类名称<T extends anyClass>

其中，anyClass 指某个接口或类。使用泛型限制后，泛型类的类型必须实现或继承 anyClass 这个接口或类。无论 anyClass 是接口还是类，在进行泛型限制时都必须使用 extends 关键字。

例如，在下面的示例代码中创建了一个 ListClass 类，并对该类的类型限制为只能是实现 List 接口的类。
// 限制ListClass的泛型类型必须实现List接口
public class ListClass<T extends List> {
    public static void main(String[] args) {
        // 实例化使用ArrayList的泛型类ListClass，正确
        ListClass<ArrayList> lc1 = new ListClass<ArrayList>();
        // 实例化使用LinkedList的泛型类LlstClass，正确
        ListClass<LinkedList> lc2 = new ListClass<LinkedList>();
        // 实例化使用HashMap的泛型类ListClass，错误，因为HasMap没有实现List接口
        // ListClass<HashMap> lc3=new ListClass<HashMap>();
    }
}
在上述代码中，定义 ListClass 类时设置泛型类型必须实现 List 接口。例如，ArrayList 和 LinkedList 都实现了 List 接口，所以可以实例化 ListClass 类。而 HashMap 没有实现 List 接口，所以在实例化 ListClass 类时会报错。

当没有使用 extends 关键字限制泛型类型时，其实是默认使用 Object 类作为泛型类型。因此，Object 类下的所有子类都可以实例化泛型类对象，如图 1 所示的这两种情况。


图1 两个等价的泛型类
2. 使用类型通配符
Java 中的泛型还支持使用类型通配符，它的作用是在创建一个泛型类对象时限制这个泛型类的类型必须实现或继承某个接口或类。

使用泛型类型通配符的语法格式如下：
泛型类名称<? extends List>a = null;
其中，“<? extends List>”作为一个整体表示类型未知，当需要使用泛型对象时，可以单独实例化。

例如，下面的示例代码演示了类型通配符的使用。
A<? extends List>a = null;
a = new A<ArrayList> ();    // 正确
b = new A<LinkedList> ();    // 正确
c = new A<HashMap> ();    // 错误
在上述代码中，同样由于 HashMap 类没有实现 List 接口，所以在编译时会报错。
3. 继承泛型类和实现泛型接口
定义为泛型的类和接口也可以被继承和实现。例如下面的示例代码演示了如何继承泛型类。
public class FatherClass<T1>{}
public class SonClass<T1,T2,T3> extents FatherClass<T1>{}

如果要在 SonClass 类继承 FatherClass 类时保留父类的泛型类型，需要在继承时指定，否则直接使用 extends FatherClass 语句进行继承操作，此时 T1、T2 和 T3 都会自动变为 Object，所以一般情况下都将父类的泛型类型保留。

下面的示例代码演示了如何在泛型中实现接口。
纯文本复制
interface interface1<T1>{}
interface SubClass<T1,T2,T3> implements
Interface1<T2>{}

# 枚举
枚举是一个被命名的整型常数的集合，用于声明一组带标识符的常数。枚举在曰常生活中很常见，例如一个人的性别只能是“男”或者“女”，一周的星期只能是 7 天中的一个等。类似这种当一个变量有几种固定可能的取值时，就可以将它定义为枚举类型。

在 JDK 1.5 之前没有枚举类型，那时候一般用接口常量来替代。而使用 Java 枚举类型 enum 可以更贴近地表示这种常量。
声明枚举
声明枚举时必须使用 enum 关键字，然后定义枚举的名称、可访问性、基础类型和成员等。枚举声明的语法如下：
enum-modifiers enum enumname:enum-base {
    enum-body,
}

其中，enum-modifiers 表示枚举的修饰符主要包括 public、private 和 internal；enumname 表示声明的枚举名称；enum-base 表示基础类型；enum-body 表示枚举的成员，它是枚举类型的命名常数。

任意两个枚举成员不能具有相同的名称，且它的常数值必须在该枚举的基础类型的范围之内，多个枚举成员之间使用逗号分隔。

提示：如果没有显式地声明基础类型的枚举，那么意味着它所对应的基础类型是 int。
例 1
下面代码定义了一个表示性别的枚举类型 SexEnum 和一个表示颜色的枚举类型 Color。
public enum SexEnum {
    male,female;
}
public enum Color {
    RED,BLUE,GREEN,BLACK;
}
之后便可以通过枚举类型名直接引用常量，如 SexEnum.male、Color.RED。

使用枚举还可以使 switch 语句的可读性更强，例如以下示例代码：
enum Signal {
    // 定义一个枚举类型
    GREEN,YELLOW,RED
}
public class TrafficLight {
    Signal color = Signal.RED;
    public void change() {
        switch(color) {
            case RED:
                color = Signal.GREEN;
                break;
            case YELLOW:
                color = Signal.RED;
                break;
            case GREEN:
                color = Signal.YELLOW;
                break;
        }
    }
}
枚举类
Java 中的每一个枚举都继承自 java.lang.Enum 类。当定义一个枚举类型时，每一个枚举类型成员都可以看作是 Enum 类的实例，这些枚举成员默认都被 final、public, static 修饰，当使用枚举类型成员时，直接使用枚举名称调用成员即可。

所有枚举实例都可以调用 Enum 类的方法，常用方法如表 1 所示。

表 1 Enum类的常用方法
方法名称	描述
values()	以数组形式返回枚举类型的所有成员
valueOf()	将普通字符串转换为枚举实例
compareTo()	比较两个枚举成员在定义时的顺序
ordinal()	获取枚举成员的索引位置
例 2
通过调用枚举类型实例的 values( ) 方法可以将枚举的所有成员以数组形式返回，也可以通过该方法获取枚举类型的成员。

下面的示例创建一个包含 3 个成员的枚举类型 Signal，然后调用 values() 方法输出这些成员。
enum Signal {
    // 定义一个枚举类型
    GREEN,YELLOW,RED;
}
public static void main(String[] args) {
    for(int i = 0;i < Signal.values().length;i++) {
        System.out.println("枚举成员："+Signal.values()[i]);
    }
}
输出结果如下：
枚举成员：GREEN
枚举成员：YELLOW
枚举成员：RED
例 3
创建一个示例，调用valueOf() 方法获取枚举的一个成员，再调用 compareTo() 方法进行比较，并输出结果。具体实现代码如下：
public class TestEnum {
    public enum Sex {
        // 定义一个枚举
        male,female;
    }
    public static void main(String[] args) {
        compare(Sex.valueOf("male"));    // 比较
    }
    public static void compare(Sex s) {
        for(int i = 0;i < Sex.values().length;i++) {
            System.out.println(s + "与" + Sex.values()[i] + "的比较结果是：" + s.compareTo(Sex.values()[i]));
        }
    }
}

上述代码中使用 Sex.valueOf("male") 取出枚举成员 male 对应的值，再将该值与其他枚举成员进行比较。最终输出结果如下：
male与male的比较结果是：0
male与female的比较结果是：-1
例 4
通过调用枚举类型实例的ordinal() 方法可以获取一个成员在枚举中的索引位置。下面的示例创建一个包含 3 个成员的枚举类型 Signal，然后调用 ordinal() 方法输出成员及对应索引位置。

具体实现代码如下：
public class TestEnum1 {
    enum Signal {
        // 定义一个枚举类型
        GREEN,YELLOW,RED;
    }
    public static void main(String[] args) {
        for(int i = 0;i < Signal.values().length;i++) {
            System.out.println("索引" + Signal.values()[i].ordinal()+"，值：" + Signal.values()[i]);
        }
    }
}

输出结果如下：
索引0，值：GREEN
索引1，值：YELLOW
索引2，值：RED
为枚举添加方法
Java 为枚举类型提供了一些内置的方法，同时枚举常量也可以有自己的方法。此时要注意必须在枚举实例的最后一个成员后添加分号，而且必须先定义枚举实例。
例 5
下面的代码创建了一个枚举类型 WeekDay，而且在该类型中添加了自定义的方法。
enum WeekDay {
    Mon("Monday"),Tue("Tuesday"),Wed("Wednesday"),Thu("Thursday"),Fri("Friday"),Sat("Saturday"),Sun("Sunday");
    // 以上是枚举的成员，必须先定义，而且使用分号结束
    private final String day;
    private WeekDay(String day) {
        this.day = day;
    }
    public static void printDay(int i) {
        switch(i) {
            case 1:
                System.out.println(WeekDay.Mon);
                break;
            case 2:
                System.out.println(WeekDay.Tue);
                break;
            case 3:
                System.out.println(WeekDay.Wed);
                break;
            case 4:
                System.out.println(WeekDay.Thu);
                break;
            case 5:
                System.out.println(WeekDay.Fri);
                break;
            case 6:
                System.out.println(WeekDay.Sat);
                break;
            case 7:
                System.out.println(WeekDay.Sun);
                break;
            default:
                System.out.println("wrong number!");
        }
    }
    public String getDay() {
        return day;
    }
}

上面代码创建了 WeekDay 枚举类型，下面遍历该枚举中的所有成员，并调用 printDay() 方法。示例代码如下：
public static void main(String[] args) {
    for(WeekDay day : WeekDay.values()) {
        System.out.println(day+"====>" + day.getDay());
    }
    WeekDay.printDay(5);
}

输出结果如下：
Mon====>Monday
Tue====>Tuesday
Wed====>Wednesday
Thu====>Thursday
Fri====>Friday
Sat====>Saturday
Sun====>Sunday
Fri

Java 中的 enum 还可以跟 Class 类一样覆盖基类的方法。下面示例代码创建的 Color 枚举类型覆盖了 toString() 方法。
public class Test {
    public enum Color {
        RED("红色",1),GREEN("绿色",2),WHITE("白色",3),YELLOW("黄色",4);
        // 成员变量
        private String name;
        private int index;
        // 构造方法
        private Color(String name,int index) {
            this.name = name;
            this.index = index;
        }
        // 覆盖方法
        @Override
        public String toString() {
            return this.index + "-" + this.name;
        }
    }
    public static void main(String[] args) {
        System.out.println(Color.RED.toString());    // 输出：1-红色
    }
}
EnumMap 与 EnumSet
为了更好地支持枚举类型，java.util 中添加了两个新类：EnumMap 和 EnumSet。使用它们可以更高效地操作枚举类型。
EnumMap 类
EnumMap 是专门为枚举类型量身定做的 Map 实现。虽然使用其他的 Map（如 HashMap）实现也能完成枚举类型实例到值的映射，但是使用 EnumMap 会更加高效。

HashMap 只能接收同一枚举类型的实例作为键值，并且由于枚举类型实例的数量相对固定并且有限，所以 EnumMap 使用数组来存放与枚举类型对应的值，使得 EnumMap 的效率非常高。
例 6
下面是使用 EnumMap 的一个代码示例。枚举类型 DataBaseType 里存放了现在支持的所有数据库类型。针对不同的数据库，一些数据库相关的方法需要返回不一样的值，例如示例中 getURL() 方法。
// 定义数据库类型枚举
public enum DataBaseType {
    MYSQUORACLE,DB2,SQLSERVER
}
// 某类中定义的获取数据库URL的方法以及EnumMap的声明
private EnumMap<DataBaseType,String>urls = new EnumMap<DataBaseType,String>(DataBaseType.class);
public DataBaseInfo() {
    urls.put(DataBaseType.DB2,"jdbc:db2://localhost:5000/sample");
    urls.put(DataBaseType.MYSQL,"jdbc:mysql://localhost/mydb");
    urls.put(DataBaseType.ORACLE,"jdbc:oracle:thin:@localhost:1521:sample");
    urls.put(DataBaseType.SQLSERVER,"jdbc:microsoft:sqlserver://sql:1433;Database=mydb");
}
//根据不同的数据库类型，返回对应的URL
// @param type DataBaseType 枚举类新实例
// @return
public String getURL(DataBaseType type) {
    return this.urls.get(type);
}

在实际使用中，EnumMap 对象 urls 往往是由外部负责整个应用初始化的代码来填充的。这里为了演示方便，类自己做了内容填充。

从本例中可以看出，使用 EnumMap 可以很方便地为枚举类型在不同的环境中绑定到不同的值上。本例子中 getURL 绑定到 URL 上，在其他的代码中可能又被绑定到数据库驱动上去。
EnumSet 类
EnumSet 是枚举类型的高性能 Set 实现，它要求放入它的枚举常量必须属于同一枚举类型。EnumSet 提供了许多工厂方法以便于初始化，如表 2 所示。

表 2 EnumSet 类的常用方法
方法名称	描述
allOf(Class<E> element type)	创建一个包含指定枚举类型中所有枚举成员的 EnumSet 对象
complementOf(EnumSet<E> s)	创建一个与指定 EnumSet 对象 s 相同的枚举类型 EnumSet 对象，
并包含所有 s 中未包含的枚举成员
copyOf(EnumSet<E> s)	创建一个与指定 EnumSet 对象 s 相同的枚举类型 EnumSet 对象，
并与 s 包含相同的枚举成员
noneOf(<Class<E> elementType)	创建指定枚举类型的空 EnumSet 对象
of(E first,e...rest)	创建包含指定枚举成员的 EnumSet 对象
range(E from ,E to)	创建一个 EnumSet 对象，该对象包含了 from 到 to 之间的所有枚
举成员
EnumSet 作为 Set 接口实现，它支持对包含的枚举常量的遍历。
纯文本复制
for(Operation op:EnumSet.range(Operation.PLUS,Operation.MULTIPLY)) {
    doSomeThing(op);
}
