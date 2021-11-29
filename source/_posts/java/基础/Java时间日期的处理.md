---
title: Java 时间日期的处理
date: 2020-09-06 19:33:52
tags: [java]
categories: java
---


在 Java 中获取当前时间，可以使用`java.util.Date`类和`java.util.Calendar`类完成。其中，`Date`类主要封装了系统的日期和时间的信息，`Calendar`类则会根据系统的日历来解释`Date`对象。
# Date 类
`Date`类表示系统特定的时间戳，可以精确到毫秒。`Date`对象表示时间的默认顺序是星期、月、日、小时、分、秒、年。
## 构造方法
`Date`类有两个构造方法。
* `Date()`：此种形式表示分配`Date`对象并初始化此对象，以表示分配它的时间（精确到毫秒），使用该构造方法创建的对象可以获取本地的当前时间。
* `Date(long date)`：此种形式表示从 GMT 时间（格林尼治时间）1970 年 1 月 1 日 0 时 0 分 0 秒开始经过参数`date`指定的毫秒数。

```java
Date date1 = new Date();    // 调用无参数构造函数
System.out.println(date1.toString());    // 输出：Wed May 18 21:24:40 CST 2016
Date date2 = new Date(60000);    // 调用含有一个long类型参数的构造函数
System.out.println(date2);    // 输出：Thu Jan 0108:01:00 CST 1970
```
`Date`类的无参数构造方法获取的是系统当前的时间，显示的顺序为星期、月、日、小时、分、秒、年。

`Date`类带`long`类型参数的构造方法获取的是距离 GMT 指定毫秒数的时间，60000 毫秒是一分钟，而 GMT（格林尼治标准时间）与 CST（中央标准时间）相差 8 小时，也就是说 1970 年 1 月 1 日 00:00:00 GMT 与 1970 年 1 月 1 日 08:00:00 CST 表示的是同一时间。 因此距离 1970 年 1 月 1 日 00:00:00 CST 一分钟的时间为 1970 年 1 月 1 日 00:01:00 CST，即使用`Date`对象表示为`Thu Jan 01 08:01:00 CST 1970`。
## 常用方法
`Date`类中的常用方法：

| 方法                            | 描述 |
| :--: | :--: |
| boolean after(Date when)        | 判断此日期是否在指定日期之后 |
| boolean before(Date when)       | 判断此日期是否在指定日期之前 |
| int compareTo(Date anotherDate) | 比较两个日期的顺序 |
| boolean equals(Object obj)      | 比较两个日期的相等性 |
| long getTime()                  | 返回自 1970 年 1 月 1 日 00:00:00 GMT 以来，此 Date 对象表示的毫秒数 |
| String toString()               | 把此 Date 对象转换为以下形式的 String: dow mon dd hh:mm:ss zzz yyyy。其中 dow 是一周中的某一天(Sun、Mon、Tue、Wed、Thu、Fri 及 Sat)|

## Calendar 类
`Calendar`类是一个抽象类，它为特定瞬间与`YEAR、MONTH、DAY_OF—MONTH、HOUR`等日历字段之间的转换提供了一些方法，并为操作日历字段（如获得下星期的日期） 提供了一些方法。

创建`Calendar`对象不能使用`new`关键字，因为`Calendar`类是一个抽象类，但是它提供了一个`getInstance()`方法来获得`Calendar`类的对象。`getInstance()`方法返回一个`Calendar`对象，其日历字段已由当前日期和时间初始化。
```java
Calendar c = Calendar.getInstance();
```
当创建了一个`Calendar`对象后，就可以通过`Calendar`对象中的一些方法来处理日期、时间。`Calendar`类的常用方法：

| 方法                                    | 描述 |
| :--: | :--: |
| void add(int field, int amount)         | 根据日历的规则，为给定的日历字段 field 添加或减去指定的时间量 amount |
| boolean after(Object when)              | 判断此 Calendar 表示的时间是否在指定时间 when 之后，并返回判断结果 |
| boolean before(Object when)             | 判断此 Calendar 表示的时间是否在指定时间 when 之前，并返回判断结果 |
| void clear()                            | 清空 Calendar 中的日期时间值 |
| int compareTo(Calendar anotherCalendar) | 比较两个 Calendar 对象表示的时间值（从格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒至现在的毫秒偏移量），大则返回 1，小则返 回 -1，相等返回 0 |
| int get(int field)                | 返回指定日历字段的值 |
| int getActualMaximum(int field)	| 返回指定日历字段可能拥有的最大值 |
| int getActualMinimum(int field)	| 返回指定日历字段可能拥有的最小值 |
| int getFirstDayOfWeek()	        | 获取一星期的第一天。根据不同的国家地区，返回不同的值 |
| static Calendar getInstance()	| 使用默认时区和语言坏境获得一个日历 |
| static Calendar getInstance(TimeZone zone)	| 使用指定时区和默认语言环境获得一个日历 |
| static Calendar getInstance(<br>TimeZone zone, Locale aLocale)	| 使用指定时区和语言环境获得一个日历 |
| Date getTime()                  | 返回一个表示此 Calendar 时间值（从格林威治时间 1970 年 01 月 01 日 00 时 00 分 00 秒至现在的毫秒偏移量）的 Date 对象 |
| long getTimeInMillis()          | 返回此 Calendar 的时间值，以毫秒为单位 |
| void set(int field, int value)	| 为指定的日历字段设置给定值 |
| void set(int year, int month, int date)	| 设置日历字段 YEAR、MONTH 和 DAY_OF_MONTH 的值 |
| void set(int year, int month, int date,<br> int hourOfDay, int minute, int second)	| 设置字段 YEAR、MONTH、DAY_OF_MONTH、HOUR、 MINUTE 和 SECOND 的值 |
| void setFirstDayOfWeek(int value)	| 设置一星期的第一天是哪一天 |
| void setTimeInMillis(long millis)	| 用给定的 long 值设置此 Calendar 的当前时间值 |

`Calendar`对象可以调用`set()`方法将日历翻到任何一个时间，当参数`year`取负数时表示公元前。`Calendar`对象调用`get()`方法可以获取有关年、月、日等时间信息，参数`field`的有效值由`Calendar`静态常量指定。

`Calendar`类中定义了许多常量，分别表示不同的意义。
* `Calendar.YEAR`：年份。
* `Calendar.MONTH`：月份。
* `Calendar.DATE`：日期。
* `Calendar.DAY_OF_MONTH`：日期，和上面的字段意义完全相同。
* `Calendar.HOUR`：12小时制的小时。
* `Calendar.HOUR_OF_DAY`：24 小时制的小时。
* `Calendar.MINUTE`：分钟。
* `Calendar.SECOND`：秒。
* `Calendar.DAY_OF_WEEK`：星期几。

例如，要获取当前月份可用如下代码：
```java
int month = Calendar.getInstance().get(Calendar.MONTH);
```
如果整型变量`month`的值是 0，表示当前日历是在 1 月份；如果值是 11，则表示当前日历在 12 月份。
```java
Calendar calendar = Calendar.getInstance(); // 如果不设置时间，则默认为当前时间
calendar.setTime(new Date()); // 将系统当前时间赋值给 Calendar 对象
System.out.println("现在时刻：" + calendar.getTime()); // 获取当前时间
int year = calendar.get(Calendar.YEAR); // 获取当前年份
System.out.println("现在是" + year + "年");
int month = calendar.get(Calendar.MONTH) + 1; // 获取当前月份（月份从 0 开始，所以加 1）
System.out.print(month + "月");
int day = calendar.get(Calendar.DATE); // 获取日
System.out.print(day + "日");
int week = calendar.get(Calendar.DAY_OF_WEEK) - 1; // 获取今天星期几（以星期日为第一天）
System.out.print("星期" + week);
int hour = calendar.get(Calendar.HOUR_OF_DAY); // 获取当前小时数（24 小时制）
System.out.print(hour + "时");
int minute = calendar.get(Calendar.MINUTE); // 获取当前分钟
System.out.print(minute + "分");
int second = calendar.get(Calendar.SECOND); // 获取当前秒数
System.out.print(second + "秒");
int millisecond = calendar.get(Calendar.MILLISECOND); // 获取毫秒数
System.out.print(millisecond + "毫秒");
int dayOfMonth = calendar.get(Calendar.DAY_OF_MONTH); // 获取今天是本月第几天
System.out.println("今天是本月的第 " + dayOfMonth + " 天");
int dayOfWeekInMonth = calendar.get(Calendar.DAY_OF_WEEK_IN_MONTH); // 获取今天是本月第几周
System.out.println("今天是本月第 " + dayOfWeekInMonth + " 周");
int many = calendar.get(Calendar.DAY_OF_YEAR); // 获取今天是今年第几天
System.out.println("今天是今年第 " + many + " 天");
Calendar c = Calendar.getInstance();
c.set(2012, 8, 8); // 设置年月日，时分秒将默认采用当前值
System.out.println("设置日期为 2012-8-8 后的时间：" + c.getTime()); // 输出时间
```
```java
import java.util.Calendar;
public class CalendarDemo {
    public static void main(String[] args) {
        Calendar calendar = Calendar.getInstance();
        calendar.set(2016, 5, 1); // 实际的calendar对象所表示的日期为2016年6月1日
        // 判断2016年6月1日为一周中的第几天
        int index = calendar.get(Calendar.DAY_OF_WEEK) - 1;
        char[] title = { '日', '一', '二', '三', '四', '五', '六' }; // 存放曰历的头部
        int daysArray[][] = new int[6][7];// 存放日历的数据
        int daysInMonth = 31; // 该月的天数
        int day = 1; // 自动增长
        for (int i = index; i < 7; i++) {
            // 填充第一周的日期数据，即日历中的第一行
            daysArray[0][i] = day++;
        }
        for (int i = 1; i < 6; i++) {
            // 填充其他周的日历数据，控制行
            for (int j = 0; j < 7; j++) {
                // 如果当前day表示的是本月最后一天，则停止向数组中继续赋值
                if (day > daysInMonth) {
                    i = 6;
                    break;
                }
                daysArray[i][j] = day++;
            }
        }
        System.out.println("------------------2016 年 6 月--------------------\n");
        for (int i = 0; i < title.length; i++) {
            System.out.print(title[i] + "\t");
        }
        System.out.print("\n");
        // 输出二元数组daysArray中的元素
        for (int i = 0; i < 6; i++) {
            for (int j = 0; j < 7; j++) {
                if (daysArray[i][j] == 0) {
                    if (i != 0) {
                        // 如果到月末，则完成显示日历的任务，停止该方法的执行
                        return;
                    }
                    System.out.print("\t");
                    continue;
                }
                System.out.print(daysArray[i][j] + "\t");
            }
            System.out.print("\n");
        }
    }
}
```
因为`Calendar`类所表示的时间月份是`set()`方法中表示月份的参数值 +1，因此`Calendar`类的实际时间为 2016 年 6 月 1 日。在下面的代码中分别获取 6 月 1 日为本周中的第几天，以便在相应的星期下开始输出 6 月份的日历。程序中的`daysArray`是一个二元数组，该二元数组控制了日历的格式输出，第一个子数组控制日历的行，第二个子数组控制曰历的列，即可输出二元数组中的每一个元素。

运行程序，执行结果如下所示。
```
------------------2016 年 6 月--------------------

日 一 二 三 四 五 六
   1 2 3 4
5 6 7 8 9 10 11
12 13 14 15 16 17 18
19 20 21 22 23 24 25
26 27 28 29 30 31 
```
# 日期格式化
格式化日期表示将日期/时间格式转换为预先定义的日期/时间格式。例如将日期`Fri May 18 15:46:24 CST2016`格式转换为`2016-5-18 15:46:24 星期五`的格式。

在 Java 中，可以使用`DateFormat`类和`SimpleDateFormat`类来格式化日期。
## DateFormat 类
`DateFormat`是日期/时间格式化子类的抽象类，它以与语言无关的方式格式化并解析日期或时间。日期/时间格式化子类（如`SimpleDateFormat`）允许进行格式化（也就是日期→文本）、解析（文本→日期）和标准化日期。

在创建`DateFormat`对象时不能使用`new`关键字，而应该使用`DateFormat`类中的静态方法`getDateInstance()`：
```java
DateFormat df = DateFormat.getDatelnstance();
```
在创建了一个`DateFormat`对象后，可以调用该对象中的方法来对日期/时间进行格式化。`DateFormat`类中常用方法：

| 方法 | 描述 |
| :--: | :--: |
| String format(Date date)	| 将 Date 格式化日期/时间字符串 |
| Calendar getCalendar()	| 获取与此日期/时间格式相关联的日历 |
| static DateFormat getDateInstance()	| 获取具有默认格式化风格和默认语言环境的日期格式 |
| static DateFormat getDateInstance(int style)	| 获取具有指定格式化风格和默认语言环境的日期格式 |
| static DateFormat getDateInstance(int style, Locale locale)	| 获取具有指定格式化风格和指定语言环境的日期格式 |
| static DateFormat getDateTimeInstance()	| 获取具有默认格式化风格和默认语言环境的日期/时间格式 |
| static DateFormat getDateTimeInstance(<br>int dateStyle,int timeStyle)	| 获取具有指定日期/时间格式化风格和默认语言环境的日期/时间格式 |
| static DateFormat getDateTimeInstance(<br>int dateStyle,int timeStyle,Locale locale)	| 获取具有指定日期/时间格式化风格和指定语言环境的日期/时间格式 |
| static DateFormat getTimeInstance()	| 获取具有默认格式化风格和默认语言环境的时间格式 |
| static DateFormat getTimeInstance(int style)	| 获取具有指定格式化风格和默认语言环境的时间格式 |
| static DateFormat getTimeInstance(<br>int style, Locale locale)	| 获取具有指定格式化风格和指定语言环境的时间格式 |
| void setCalendar(Calendar newCalendar)	| 为此格式设置日历 |
| Date parse(String source)	| 将给定的字符串解析成日期/时间 |

格式化样式主要通过`DateFormat`常量设置。将不同的常量传入到表所示的方法中，以控制结果的长度。`DateFormat`类的常量如下。
* `SHORT`：完全为数字，如 12.5.10 或 5:30pm。
* `MEDIUM`：较长，如 May 10，2016。
* `LONG`：更长，如 May 12，2016 或 11:15:32am。
* `FULL`：是完全指定，如`Tuesday、May 10、2012 AD`或`11:l5:42am CST`。

```java
// 获取不同格式化风格和中国环境的日期
DateFormat df1 = DateFormat.getDateInstance(DateFormat.SHORT, Locale.CHINA);
DateFormat df2 = DateFormat.getDateInstance(DateFormat.FULL, Locale.CHINA);
DateFormat df3 = DateFormat.getDateInstance(DateFormat.MEDIUM, Locale.CHINA);
DateFormat df4 = DateFormat.getDateInstance(DateFormat.LONG, Locale.CHINA);
// 获取不同格式化风格和中国环境的时间
DateFormat df5 = DateFormat.getTimeInstance(DateFormat.SHORT, Locale.CHINA);
DateFormat df6 = DateFormat.getTimeInstance(DateFormat.FULL, Locale.CHINA);
DateFormat df7 = DateFormat.getTimeInstance(DateFormat.MEDIUM, Locale.CHINA);
DateFormat df8 = DateFormat.getTimeInstance(DateFormat.LONG, Locale.CHINA);
// 将不同格式化风格的日期格式化为日期字符串
String date1 = df1.format(new Date());
String date2 = df2.format(new Date());
String date3 = df3.format(new Date());
String date4 = df4.format(new Date());
// 将不同格式化风格的时间格式化为时间字符串
String time1 = df5.format(new Date());
String time2 = df6.format(new Date());
String time3 = df7.format(new Date());
String time4 = df8.format(new Date());
// 输出日期
System.out.println("SHORT：" + date1 + " " + time1);
System.out.println("FULL：" + date2 + " " + time2);
System.out.println("MEDIUM：" + date3 + " " + time3);
System.out.println("LONG：" + date4 + " " + time4);
```
运行该段代码，输出的结果如下：
```
SHORT：18-10-15 上午9:30
FULL：2018年10月15日 星期一 上午09时30分43秒 CST
MEDIUM：2018-10-15 9:30:43
LONG：2018年10月15日 上午09时30分43秒
```
## SimpleDateFormat 类
如果使用`DateFormat`类格式化日期/时间并不能满足要求，那么就需要使用`DateFormat`类的子类——`SimpleDateFormat`。

`SimpleDateFormat`是一个以与语言环境有关的方式来格式化和解析日期的具体类，它允许进行格式化（日期→文本）、解析（文本→日期）和规范化。`SimpleDateFormat`使得可以选择任何用户定义的日期/时间格式的模式。

`SimpleDateFormat`类主要有如下 3 种构造方法。
* `SimpleDateFormat()`：用默认的格式和默认的语言环境构造`SimpleDateFormat`。
* `SimpleDateFormat(String pattern)`：用指定的格式和默认的语言环境构造`SimpleDateFormat`。
* `SimpleDateFormat(String pattern,Locale locale)`：用指定的格式和指定的语言环境构造`SimpleDateFormat`。

`SimpleDateFormat`自定义格式中常用的字母及含义：

| 字母 | 含义 | 示例 |
| :--: | :--: | :--: |
| y	 | 年份。一般用 yy 表示两位年份，yyyy 表示 4 位年份	使用 yy 表示的年扮，如 11； | 使用 yyyy 表示的年份，如 2011 |
| M	 | 月份。一般用 MM 表示月份，如果使用 MMM，则会根据语言环境显示不同语言的月份 | 使用 MM 表示的月份，如 05；使用 MMM 表示月份，在 Locale.CHINA 语言环境下，如“十月”；在 Locale.US 语言环境下，如 Oct |
| d	 | 月份中的天数。一般用 dd 表示天数 | 使用 dd 表示的天数，如 10 |
| D	 | 年份中的天数。表示当天是当年的第几天， 用 D 表示 | 使用 D 表示的年份中的天数，如 295 |
| E	 | 星期几。用 E 表示，会根据语言环境的不同， 显示不同语言的星期几 | 使用 E 表示星期几，在 Locale.CHINA 语言环境下，如“星期四”；在 Locale.US 语言环境下，如 Thu |
| H	 | 一天中的小时数（0~23)。一般用 HH 表示小时数 | 使用 HH 表示的小时数，如 18 |
| h	 | 一天中的小时数（1~12)。一般使用 hh 表示小时数  | 使用 hh 表示的小时数，如 10 (注意 10 有可能是 10 点，也可能是 22 点） |
| m	 | 分钟数。一般使用 mm 表示分钟数      | 使用 mm 表示的分钟数，如 29 |
| s	 | 秒数。一般使用 ss 表示秒数          | 使用 ss 表示的秒数，如 38 |
| S	 | 毫秒数。一般使用 SSS 表示毫秒数     | 使用 SSS 表示的毫秒数，如 156 |

```java
import java.text.SimpleDateFormat;
import java.util.Date;
public class Test13 {
    public static void main(String[] args) {
        Date now = new Date(); // 创建一个Date对象，获取当前时间
        // 指定格式化格式
        SimpleDateFormat f = new SimpleDateFormat("今天是 " + "yyyy 年 MM 月 dd 日 E HH 点 mm 分 ss 秒");
        System.out.println(f.format(now)); // 将当前时间袼式化为指定的格式
    }
}
// 运行结果：
今天是 2018 年 10 月 15 日 星期一 09 点 26 分 23 秒
```