


Spring Expression Language（简称 SpEL）是一种功能强大的表达式语言，支持运行时查询和操作对象图 。表达式语言一般是用最简单的形式完成最主要的工作，以此减少工作量。

Java 有许多可用的表达式语言，例如 JSP EL，OGNL，MVEL 和 JBoss EL，SpEL 语法类似于 JSP EL，功能类似于 Struts2 中的 OGNL，能在运行时构建复杂表达式、存取对象图属性、调用对象方法等，并且能与 Spring 功能完美整合，如 SpEL 可以用来配置 Bean 定义。

SpEL 并不与 Spring 直接相关，可以被独立使用。SpEL 表达式的创建是为了向 Spring 社区提供一种受良好支持的表达式语言，该语言适用于 Spring 家族中的所有产品。也就是说，SpEL 是一种与技术无关的 API，可以集成其它表达式语言。

SpEL 提供了以下接口和类：
Expression interface：该接口负责评估表达式字符串
ExpressionParser interface：该接口负责解析字符串
EvaluationContext interface：该接口负责定义上下文环境

SpEL 支持如下表达式：
1. 基本表达式
字面量表达式、关系、逻辑与算术运算表达式、字符串连接及截取表达式、三目运算表达式、正则表达式、括号优先级表达式；
2. 类相关表达式
类类型表达式、类实例化、instanceof 表达式、变量定义及引用、赋值表达式、自定义函数、对象属性存取及安全导航表达式、对象方法调用、Bean 引用；
3. 集合相关表达式
内联 List、内联数组、集合、字典访问、列表、字典、数组修改、集合投影、集合选择；不支持多维内联数组初始化；不支持内联字典定义；
4. 其他表达式
模板表达式。
注：SpEL 表达式中的关键字不区分大小写。

示例
例 1
下面使用 SpEL 输出一个简单的字符串“Hello，编程帮”。
package net.biancheng;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
public class Test {
    public static void main(String[] args) {
        // 构造解析器
        ExpressionParser parser = new SpelExpressionParser();
        // 解析器解析字符串表达式
        Expression exp = parser.parseExpression("'Hello，编程帮'");
        // 获取表达式的值
        String message = (String) exp.getValue();
        System.out.println(message);
        // OR
        // System.out.println(parser.parseExpression("'Hello，编程帮'").getValue());
    }
}
输出结果如下。
Hello，编程帮


SpEL 还可以调用方法、访问属性和调用构造函数。
例 2
使用 SpEL 调用 concat() 方法，代码如下。
package net.biancheng;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
public class Test {
    public static void main(String[] args) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'Welcome，编程帮'.concat('！')");
        String message = (String) exp.getValue();
        System.out.println(message);
    }
}
输出结果如下。
Welcome，编程帮！

例 3
使用 SpEL 调用 String 的属性 bytes，将字符串转换为字节数组，代码如下。
package net.biancheng;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
public class Test {
    public static void main(String[] args) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'Hello 编程帮'.bytes");
        byte[] bytes = (byte[]) exp.getValue();
        for (int i = 0; i < bytes.length; i++) {
            System.out.print(bytes[i] + " ");
        }
    }
}
输出结果如下。
72 101 108 108 111 32 -79 -32 -77 -52 -80 -17 

例 4
SpEL 还支持使用嵌套属性，下面将字符串转换为字节后获取长度，代码如下。
package net.biancheng;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
public class Test {
    public static void main(String[] args) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("'Hello 编程帮'.bytes.length");
        int length = (Integer) exp.getValue();
        System.out.println(length);
    }
}
输出结果如下。
12

例 5
字符串的构造函数可以被调用，而不是使用字符串文本，下面将字符串内容转换为大写字母，代码如下。
package net.biancheng;
import org.springframework.expression.Expression;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
public class Test {
    public static void main(String[] args) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression("new String('hello bianchengbang').toUpperCase()");
        String message = exp.getValue(String.class);
        System.out.println(message);
        // OR
        // System.out.println(parser.parseExpression("'hello
        // bianchengbang'.toUpperCase()").getValue());
    }
}
输出结果如下。
HELLO BIANCHENGBANG

SpEL对Bean定义的支持
SpEL 表达式可以与 XML 或基于注解的配置元数据一起使用，SpEL 表达式以#{开头，以}结尾，如#{'Hello'}。
1. 基于XML的配置
可以使用以下表达式来设置属性或构造函数的参数值。
<bean id="number" class="net.biancheng.Number">
    <property name="randomNumber" value="#{T(java.lang.Math).random() * 100.0}"/>
</bean>
也可以通过名称引用其它 Bean 属性，如以下代码。
<bean id="shapeGuess" class="net.biancheng.ShapeGuess">
    <property name="shapSeed" value="#{number.randomNumber}"/>
</bean>
2. 基于注解的配置
@Value 注解可以放在字段、方法、以及构造函数参数上，以指定默认值。

以下是一个设置字段变量默认值的例子。
public static class FieldValueTestBean
    @value("#{ systemProperties[ 'user.region'] }")
    private String defaultLocale;
    public void setDefaultLocale (String defaultLocale) {
        this.defaultLocale = defaultLocale;
    }
    public string getDefaultLocale() {
        return this.defaultLocale;
    }
}
SpEL中的运算符
我们可以在 SpEL 中使用运算符，例如算术运算符、关系运算符、逻辑运算符。
例 6
package net.biancheng;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
public class Test {
    public static void main(String[] args) {
        ExpressionParser parser = new SpelExpressionParser();
        // 算术运算符
        System.out.println(parser.parseExpression("'Hello SPEL'+'!'").getValue());
        System.out.println(parser.parseExpression("10 * 10/2").getValue());
        System.out.println(parser.parseExpression("'今天是：'+ new java.util.Date()").getValue());
        // 逻辑运算符
        System.out.println(parser.parseExpression("true and true").getValue());
        // 关系运算符
        System.out.println(parser.parseExpression("'sonoo'.length()==5").getValue());
    }
}

输出结果为
Hello SPEL!
50
今天是：Fri Feb 05 09:31:46 CST 2021
true
true

SpEL中的变量
在 SpEL 中，我们可以定义变量，并在方法中使用。定义变量需要用到 StandardEvaluationContext 类。
例 7
Calculation 类代码如下。
package net.biancheng;
public class Calculation {
    private int number;
    public int getNumber() {
        return number;
    }
    public void setNumber(int number) {
        this.number = number;
    }
    public int cube() {
        return number * number * number;
    }
}

Test 类代码如下。
纯文本复制
package net.biancheng;
import org.springframework.expression.ExpressionParser;
import org.springframework.expression.spel.standard.SpelExpressionParser;
import org.springframework.expression.spel.support.StandardEvaluationContext;
public class Test {
    public static void main(String[] args) {
        Calculation calculation = new Calculation();
        StandardEvaluationContext context = new StandardEvaluationContext(calculation);
        ExpressionParser parser = new SpelExpressionParser();
        parser.parseExpression("number").setValue(context, "5");
        System.out.println(calculation.cube());
    }
}