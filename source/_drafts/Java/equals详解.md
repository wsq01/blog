

判断对象是否相等的`equals`有两种方法：
* `java.lang.Object`包下的`a.equals(b)`;
* `java.util.Objects`包下的`Objects.equals(Object a, Object b)`

```java
// java.lang.Object
public boolean equals(Object obj) {
  return (this == obj);
}
```

```java
// java.util.Objects
public static boolean equals(Object a, Object b) {
  return (a == b) || (a != null && a.equals(b));
}
```
首先，进行了对象地址的判断，如果是真，则不再继续判断。

如果不相等，先判断`a`不为空，然后根据上面的知识点，就不会再出现空指针。

所以，如果都是`null`，在第一个判断上就为`true`了。如果不为空，地址不同，就要判断`a.equals(b)`。

值是`null`的情况：
1. `a.equals(b)`, `a`是`null`, 抛出`NullPointException`异常。
2. `a.equals(b)`, `a`不是`null`, `b`是`null`,  返回`false`
3. `Objects.equals(a, b)`比较时， 若`a`和`b`都是`null`, 则返回`true`, 如果`a`和`b`其中一个是`null`, 另一个不是`null`, 则返回`false`。注意：不会抛出空指针异常。

```java
null.equals("abc")    // 抛出 NullPointerException 异常
"abc".equals(null)    // 返回 false
null.equals(null)     // 抛出 NullPointerException 异常
Objects.equals(null, "abc") // 返回 false
Objects.equals("abc",null)  // 返回 false
Objects.equals(null, null)  // 返回 true
```
值是空字符串的情况：
1. `a`和`b`如果都是空值字符串：`""`, 则`a.equals(b)`, 返回的值是`true`, 如果`a`和`b`其中有一个不是空值字符串，则返回`false`;
2. 这种情况下`Objects.equals`与情况 1 行为一致。

```java
"abc".equals("")  // 返回 false
"".equals("abc")  // 返回 false
"".equals("")     // 返回 true
Objects.equals("abc", "") // 返回 false
Objects.equals("","abc")  // 返回 false
Objects.equals("","")     // 返回 true
```
`a==b`和`a.equals(b)`区别：
* 如果`a`和`b`都是对象，则`a==b`是比较两个对象的引用，只有当`a`和`b`指向的是堆中的同一个对象才会返回`true`。
* 而`a.equals(b)`是进行逻辑比较，当内容相同时，返回`true`，所以通常需要重写该方法来提供逻辑一致性的比较。