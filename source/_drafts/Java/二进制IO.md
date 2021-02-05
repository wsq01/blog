


# 文本I/O和二进制I/O
计算机并不区分二进制文件和文本文件。所有的文件都是以二进制形式来存储的，因此，从本质上来讲，所有的文件都是二进制文件。

文本I/O建立在二进制I/O的基础之上，它能提供字符层次的编码和解码的抽象。

在文本I/O中自动进行编码和解码。在写入一个字符时，Java 虚拟机会将统一码转化为文件指定的编码，而在读取字符时，将文件指定的编码转化为统一码。



二进制I/O不需要转化


# 二进制I/O类
实现二进制I/O的类。

{% asset_img img1.png %}

InputStream 类的所有方法：
* `read(): int`：从输入流读取数据的下一个字节，返回的字节的值是一个 0 到 255 之间的整数值。如果因为到达这个流的末尾而无字节可用，则返回值 -1。
* `read(b: byte[]): int`：从输入流读取大到`b.length`的字节给数组，然后返回读取的实际字节数，在流的末尾返回 -1.
* `read(b: byte[], off: int, len: int): int`：从输入流读取字节，并将它们存储到`b[off]、b[off + 1]、...、b[off + len - 1]`。返回实际读取的字节数，在流的末尾返回 -1.
* `available(): int`：返回能从输入流读取的字节数的估计值。
* `close(): void`：关闭这个输入流并释放它所占的系统资源。
* `skip(n: long): long`：跳过或丢弃输入流的`n`字节数据，返回实际跳过的字节数。
* `markSupported(): boolean`：测试这个输入流是否支持`mark`和`reset`方法。
* `mark(readlimit: int): void`：标记这个输入流的当前位置
* `reset(): void`：最后一次调用这个输入流上的`mark方法时复位这个流`。

OutputStream 类的所有方法：
* `write(int b): void`：向这个输出流写入指定字节，参数`b`是一个`int`值，`(byte)b`写入输出流。
* `write(b: byte[]): void`：将数组`b`中的所有字节写入输出流。
* `write(b: byte[], off: int, len: int): void`：将`b[off]、b[off + 1]、...、b[off + len - 1]`写入输出流。
* `close(): void`：关闭这个输出流并释放它所占的系统资源。
* `flush(): void`：刷新这个输出流并迫使所有缓冲的输出字节写出。

## FileInputStream类和FileOutputStream类
FileInputStream 类和 FileOutputStream 类的所有方法都是从 InputStream 类和 OutputStream 类继承的。FileInputStream 类和 FileOutputStream 类没有引入新的方法。

为了构造一个 FileInputStream 对象，使用下面的构造方法。
```java
FileInputStream(file: File) // 从一个 File 对象创建一个 FileInputStream
FileInputStream(filename: String) // 从一个文件名创建一个 FileInputStream
```
如果试图为一个不存在的文件创建 FileInputStream 对象，将会发生 java.io.File NotFound-Exception 异常。

要构造一个 FileOutputStream 对象，使用下面的构造方法。
```java
FileOutputStream(file: File)
FileOutputStream(filename: String)
FileOutputStream(file: File, append: boolean)
FileOutputStream(filename: String, append: boolean)
```
如果这个文件不存在，就会创建一个新文件。如果这个文件已经存在，前两个构造方法将会删除文件的当前内容。为了既保留文件现有的内容又可以给文件追加新的数据，将最后两个构造方法中的参数`append`设为`true`。

几乎所有的I/O类中的方法都会抛出异常`java.io.IOException`。因此，必须在方法中声明会抛出`java.io.IOException`异常，或将代码放到`try-catch`块中。
```java
// 在方法中声明异常
public static void main(String[] args) throws IOException {

}
// 使用 try-catch 块
public static void main(String[] args) {
  try {

  } catch (IOException ex) {
    ex.printStackTrace();
  }
}
```
下面的程序使用二进制I/O将从 1 到 10 的 10 个字节值写入一个名为`temp.dat`的文件，再把它们从文件中读出来。
```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class FileInputStreamAndFileOutputStream {
  public static void main(String[] args) throws IOException {
    FileOutputStream output = new FileOutputStream("temp.dat");
    for (int i = 1; i <= 10; i++) {
      output.write(i);
    }
    output.close();

    FileInputStream input = new FileInputStream("temp.dat");
    int value;
    while ((value = input.read()) != -1) {
      System.out.println(value + " ");
    }
    input.close();
  }
}
```
## FilterInputStream类和FilterOutputStream类
过滤器数据流是为某种目的过滤字节的数据流。基本字节输入流提供的读取方法`read`只能用来读取字节。如果要读取整数值、双精度值或字符串，那就需要一个过滤器类来包装字节输入流。

使用过滤器类就可以读取整数值、双精度值或字符串，而不是字节或字符。

FilterInputStream类和FilterOutputStream类是过滤数据的基类，需要处理基本数值类型时，就使用DataInputStream类和DataOutputStream类来过滤字节。
## DataInputStream类和DataOutputStream类
DataInputStream从数据流读取字节，并且将它们转换为正确的基本类型值或字符串。DataOutputStream将基本类型值或字符串转换为字节，并将字节输出到数据流。

DataInputStream类扩展FilterInputStream类，并实现DataInput接口。DataOutputStream类扩展FilterOutputStream类，并实现DataOutput接口。
```java
// DataInput
readBoolean(): boolean // 从输入流中读取一个布尔值
readByte(): byte // 从输入流读取一个字节
readChar(): char // 从输入流读取一个字符
readFloat(): float // 从输入流读取一个float值
readDouble(): float // 从输入流读取一个double值
readInt(): int // 从输入流读取一个int值
readLong(): long // 从输入流读取一个long值
readShort(): short // 从输入流读取一个short值
readLine(): String // 从输入读取一行字符
readUTF(): String // 读取UTF格式的字符串

// DataOutput
writeBoolean(): void // 向输出流写入一个布尔值
writeByte(v: int): void // 向输出流写入参数v的8个低阶位
writeBytes(s: String): void // 向输出流写入一个字符串的低字节字符
writeChar(c: char): void // 向输出流写入一个字符（由两个字节组成）
writeChars(s: String): void // 向输出流写入字符串s中的每一个字符，依序两个字节为一个字符
writeFloat(v: float): void // 向输出流写入一个float值
writeDouble(v: double): void // 向输出流写入一个double值
writeInt(v: int): void // 向输出流写入一个int值
writeLong(v: int): void // 向输出流写入一个long值
writeShort(v: short): void // 向输出流写入一个short值
writeUTF(s: String): void // 写入一个UTF格式的字符串s
```
DataInputStream实现了定义在DataInput接口中的方法来读取基本数据类型值和字符串。DataOutputStream实现了定义在DataOutput接口中的方法来写入基本数据类型值和字符串。基本类型的值不需要做任何转化就可以从内存复制到输出数据流。字符串中的字符可以写成多种形式。


## BufferedInputStream类和BufferedOutputStream类
BufferedInputStream类和BufferedOutputStream类可以通过减少读写次数来提高输入和输出的速度。BufferedInputStream类和BufferedOutputStream类没有包含新的方法。BufferedInputStream类和BufferedOutputStream类中的所有方法都是从InputStream类和OutputStream类继承而来的。BufferedInputStream类和BufferedOutputStream类为存储字节在流中添加一个缓冲区，以提高处理效率。

BufferedInputStream类和BufferedOutputStream类的构造方法：
```java
BufferedInputStream(in: InputStream) // 从一个 InputStream 对象创建一个 BufferedInputStream
// 从一个带指定缓冲区大小的 InputStream 对象创建一个 BufferedInputStream
BufferedInputStream(filename: String, bufferSize: int)

BufferedOutputStream(out: OutputStream) // 从一个 OutputStream 对象创建一个 BufferedOutputStream
// 从一个带指定缓冲区大小的 InputStream 对象创建一个 BufferedOutputStream
BufferedOutputStream(filename: String, bufferSize: int)
```
如果没有指定缓冲区大小，默认的大小是 512 个字节。缓冲区输入流会在每次读取调用中尽可能多的将数据读入缓冲区。相反的，只有当缓冲区已满或调用`flush()`方法时，缓冲输出流才会调用写入的方法。
```java
DataOutputStream output = new DataOutputStream(
  new BufferedOutputStream(new FileOutputStream("temp.dat"))
);
```
应该总是使用缓冲区IO来加速输入和输出。对于小文件，我们可能注意不到性能的提升。但对于超过 100MB 的大文件，我们会看到使用缓冲的IO带来的实质性的性能提升。