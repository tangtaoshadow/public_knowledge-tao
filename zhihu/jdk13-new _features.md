# Java 13中的新功能和API



**Author：**[杭州电子科技大学](http://www.hdu.edu.cn/)  `2016级 ` `唐涛` [16011324@hdu.edu.cn](mailto:16011324@hdu.edu.cn)

**CreateTime：**`2019-10-13 18:06:26`

**UpdateTime：**`2019-10-14 23:50:56`

**Copyright:**  唐涛 2019 ©  [HOME](https://www.promiselee.cn/tao) 

**Email：**[tangtao2099@outlook.com](mailto:tangtao2099@outlook.com)

**Link：**  [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)  [GitHub](https://github.com/tangtaoshadow)  [Gitee](https://gitee.com/tangtao_2099)





 `Java 13`于2019年9月17日发布，除了大约2300个错误修复和小的增强功能之外，Java的新版本还包含5个主要增强功能，这些增强功能也称为JEP。





# 文本块 (Preview)

 `Java`一直以来都在定义字符串的方式上受了点苦，字符串以双引号开头，以双引号结尾。

有时我们需要在`Java`中定义一个很长的多行字符串。例如，它可以是HTML页面代码或SQL查询，我们通常是这样写的：

```java
String html = "<html>\n" +
              "    <body>\n" +
              "        <p>Hello, world</p>\n" +
              "    </body>\n" +
              "</html>\n";
```

您可能已经注意到很多\ n，+和“”字符，这会使得代码更长而且更难阅读。某些编程语言（如Python）允许使用三引号”“”来定义和终止多行字符串。这种新的语言功能将有助于更方便的编写和阅读代码。在`JEP 355`中将引入一种更好的方法来在Java中定义多行字符串。



> Add text blocks to the Java language. A text block is a multi-line string literal that avoids the need for most escape sequences, automatically formats the string in a predictable way, and gives the developer control over format when desired.

如果之前我们这样写代码：
```java
String smallBlock = """Only one line""";
```


 这将报以下错误： 

```java

TextBlock.java:3: error: illegal text block open delimiter sequence, missing line terminator
   String smallBlock = """Text Block""";
                          ^
TextBlock.java:3: error: illegal text block open delimiter sequence, missing line terminator
   String smallBlock = """Text Block""";
               
```

现在你可以将一个简单的`HTML`分配给一个`String` 

```java
String htmlBlock = """
                   <html>
                     <body>
                       <p>My web page</p>
                     </body>
                   <html>
                  """;
```

**注意：**右边引号的位置很重要，因为它决定了如何处理*附带的*空格。在上面的示例中，右引号与HTML文本的缩进对齐。在这种情况下，编译器将删除缩进空间以产生如下字符串： 

```html
< html >
  < 身体>
    < p >我的网页</ p >
  </ body >
</ html >
```

 **请注意**，这将在字符串末尾包含换行符，如果你想这样做，你可以直接在字符串末尾添加“”“





然而，不幸的是，新语法只是一个预览语言功能，它默认不可用。要启用新语法，您必须使用`--enable-preview --release 13`选项运行Java编译器，然后`java`使用`--enable-preview`选项启动。例如：

```bash
$ javac -d classes --enable-preview --release 13 Test.java
$ java -classpath classes --enable-preview Test
```

即使新功能看起来很简单，JEP 355还是围绕新文本块讨论了很多重要的主题，例如*行终止符，缩进，附带空格，转义序列，串联*。此外，JEP增加了三个新的方法：`String::stripIndent()`，`String::translateEscapes()`和`String::formatted(Object… args)`。





# Switch 表达式 (Preview)

`switch`在`JDK 12`中引入了新的表达式作为预览语言功能。长话短说，JEP 325增添了一个新的简化形式`switch`与块`case L ->`的标签。[您可以在上一篇文章中找到更多详细信息](https://blog.gypsyengineer.com/en/tech/what-is-new-in-java-12.html)。

在以前的`switch`表达式预览版本中，建议添加一种新形式的`break`带有值的语句，该值将用于从`switch`表达式。在新版本的`switch`表达式中，它将替换为新的`yield`语句，我们可以用作语句（不返回内容）或表达式（返回内容）。引入新的关键字 `yield` 以从`switch`返回值。 

> To yield a value from a switch expression, the* `break` *with value statement is dropped in favor of a* `yield` *statement.* 



在**Java 12** `switch`语句之前，我们可以返回如下值：

```java
	private static String getText(int number) {
        String result = "";
        switch (number) {
            case 1, 2:
                result = "one or two";
                break;
            case 3:
                result = "three";
                break;
            case 4, 5, 6:
                result = "four or five or six";
                break;
            default:
                result = "unknown";
                break;
        };
        return result;
    }

```



在**Java 12**中，我们可以用`break`返回值。

```java
	private static String getText(int number) {
        String result = switch (number) {
            case 1, 2:
                break "one or two";
            case 3:
                break "three";
            case 4, 5, 6:
                break "four or five or six";
            default:
                break "unknown";
        };
        return result;
    }

```



到了**Java 13**中，上面的**Java 12** `value break`不再被编译，我们应该使用它`yield`来返回一个值。

```java
	private static String getText(int number) {
        return switch (number) {
            case 1, 2:
                yield "one or two";
            case 3:
                yield "three";
            case 4, 5, 6:
                yield "four or five or six";
            default:
                yield "unknown";
        };
    }

```



**Java 13**仍支持**Java 12** 规则标签或箭头语法。

```java
	private static String getText(int number) {
        return switch (number) {
            case 1, 2 -> "one or two";
            case 3 -> "three";
            case 4, 5, 6 -> "four or five or six";
            default -> "unknown";
        };
    }
```



或者：

```java
int result = switch (s) {
    case "Foo" -> 1;
    case "Bar" -> 2;
    default    -> {
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
    }
};
```



**重点**

新`switch`表达式仍仅在预览模式下可用**(｡•́︿•̀｡)(｡•́︿•̀｡)**。

要启用新语法，您必须使用`--enable-preview --release 13`选项运行Java编译器，然后`java`使用`--enable-preview`选项启动（请参见上面的示例）。





# 动态CDS存档

该JEP简化了应用程序类数据共享（AppCDS）的使用。当程序存在时，可以通过使用而不是提供类列表来生成存档`-XX:ArchiveClassesAtExit=`。然后，通过运行该应用程序，`-XX:SharedArchiveFile=`可以在JDK中的系统默认归档文件之上共享类数据。



**CDS**代表类数据共享。`CDS`允许将一组类预处理为共享的存档文件。为什么我们需要这样的档案？因为我们在加载类时，JVM会帮我们做很多事情，例如解析类时，它会将他们存储在内部结构中，对它们执行多次检查，解析链接和符号等。只有这样，类才能开始正常工作，所有这些步骤都是需要一些时间的。另外，JVM实例通常是会加载同一组类包括两个应用程序类和系统类诸如应用`String`，`Integer`和`ArrayList`。所有这些类也是都需要内存的。因此如果我们有一个包含预处理类的共享存档，则可以在运行时对其进行内存映射。这样做的好处就是它可以减少启动时间和内存占用。

在低于10的`Java`版本中仅允许将系统类添加到共享归档文件中，`Java 10`扩展了`CDS`功能，以允许将应用程序类也放置在共享档案中。

您需要执行两个步骤，首先，构建应用程序加载的类的列表：

```bash
java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=classes.lst \
     -classpath application.jar Application
```

next，再创建一个包含这些类的共享存档：

```bash
java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=classes.lst \
     -XX:SharedArchiveFile=application.jsa -classpath application.jar Application
```

 在 `JEP 350`中扩展了应用程序类数据共享功能，允许你在Java应用程序执行结束时动态归档类，这样我们就可以将以上两条合并为一个命令**ヾ(๑╹◡╹)ﾉ"**：

```bash
java -XX:ArchiveClassesAtExit=application.jsa \
     -classpath application.jar Application
```



**注意：** 只有在应用程序执行期间加载的类才会添加到存档中，也就是说，存在于`application.jar`执行过程中，但是未在执行过程中加载的类将不包含在归档中。

不幸的是，首次运行应用程序时无法自动创建档案。JEP的作者明确提到：[链接](https://openjdk.java.net/jeps/350)

> *A follow-up enhancement to this JEP could perform automatic archive generation during the first run of an application. This would eliminate the explicit archive creation step (step 2 above). The usage of CDS/AppCDS could then be completely transparent and automatic.* 





# ZGC：取消提交未使用的内存

[ZGC垃圾收集器](https://wiki.openjdk.java.net/display/zgc/Main)是一个并发垃圾收集器，它试图减少垃圾收集时的暂停时间，当Java线程继续执行时，此垃圾收集器（GC）尝试完成所有工作。

GC收集垃圾时，通常应取消提交并将内存释放给操作系统。在`Hotspot`中的`G1`和`Shenandoah`垃圾收集器已经这样做了，但是ZGS当前不会释放内存，即使该内存已经使用了很长时间了。在`JEP 351`中通过引入一个新选项 `-XX:ZUncommitDelay` 来解决此问题，该选项告诉`ZGC`何时应释放未使用的内存。JEP的作者稍微解释了`ZGC`和新选项的工作方式，[链接](https://openjdk.java.net/jeps/351)





# 重新实现旧版套接字API

`java.net.Socket`和`java.net.ServerSocket`的基础实现非常古老，它可以追溯到`JDK 1.0` **(・ω≦)**，它是Java和C代码的混合，难以维护和调试。该JEP为Socket API引入了新的基础实现，它是Java 13中的默认实现。

该JEP替代了`java.net.Socket`和`java.net.ServerSocket`APIs 的基础实现，尽管可能没有人直接使用这些API，但是此更新可能会影响几乎所有使用JDK的人，因为Java套接字是联网用的主要API。这是JEP的作者解释为什么需要进行更新的原因： 

> *The java.net.Socket and java.net.ServerSocket APIs, and their underlying implementations, date back to JDK 1.0. The implementation is a mix of legacy Java and C code that is painful to maintain and debug. The implementation uses the thread stack as the I/O buffer, an approach that has required increasing the default thread stack size on several occasions. The implementation uses a native data structure to support asynchronous close, a source of subtle reliability and porting issues over the years. The implementation also has several concurrency issues that require an overhaul to address properly. In the context of a future world of fibers that park instead of blocking threads in native methods, the current implementation is not fit for purpose.* 

也就是说，更新API将使开发JDK更加容易，**请注意**，JEP并不会涉及的基础实现`java.net.DatagramSocket`。如果更新未引入 regression ，那么很可能JDK用户不会看到很大的不同。幸运的是，作者知道这种风险，并且他们不会删除旧实现。如果出现问题，用户可以通过设置`jdk.net.usePlainSocketImpl`系统属性来切换回旧实现。但是，旧实现和系统属性将在下一个JDK版本中删除。可以在[JEP官方页面](https://openjdk.java.net/jeps/353)上找到更多详细信息。 

 在Java 13之前，它将`PlainSocketImpl`用作`SocketImpl` 

```java
public class ServerSocket implements java.io.Closeable {
    
    /**
     * The implementation of this Socket.
     */
    private SocketImpl impl;
	
}
```

在Java 13中，它引入了一个新`NioSocketImpl`类来代替`PlainSocketImpl`。但是，如果出现问题，我们仍然可以`PlainSocketImpl`通过设置`jdk.net.usePlainSocketImpl`系统属性来切换回旧实现

我们来看一个`Socket`示例：

```java
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

public class JEP353 {

    public static void main(String[] args) {

        try (ServerSocket serverSocket = new ServerSocket(8888)){

            boolean running = true;
            while(running){

                Socket clientSocket = serverSocket.accept();
                //do something with clientSocket
            }

        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

跟踪上述`Socket`类的类加载，我们发现，在Java 13中，默认实现是`NioSocketImpl` 

```bash
D:\test>javac JEP353.java

D:\test>java JEP353

D:\test>java -XX:+TraceClassLoading JEP353  | findStr Socket

[0.040s][info   ][class,load] java.net.ServerSocket source: jrt:/java.base
[0.040s][info   ][class,load] jdk.internal.access.JavaNetSocketAccess source: jrt:/java.base
[0.040s][info   ][class,load] java.net.ServerSocket$1 source: jrt:/java.base
[0.040s][info   ][class,load] java.net.SocketOptions source: jrt:/java.base
[0.040s][info   ][class,load] java.net.SocketImpl source: jrt:/java.base
[0.044s][info   ][class,load] java.net.SocketImpl$$Lambda$1/0x0000000800ba0840 source: java.net.SocketImpl
[0.047s][info   ][class,load] sun.net.PlatformSocketImpl source: jrt:/java.base

[0.047s][info   ][class,load] sun.nio.ch.NioSocketImpl source: jrt:/java.base

[0.047s][info   ][class,load] sun.nio.ch.SocketDispatcher source: jrt:/java.base
[0.052s][info   ][class,load] java.net.SocketAddress source: jrt:/java.base
[0.052s][info   ][class,load] java.net.InetSocketAddress source: jrt:/java.base
[0.052s][info   ][class,load] java.net.InetSocketAddress$InetSocketAddressHolder source: jrt:/java.base
[0.053s][info   ][class,load] sun.net.ext.ExtendedSocketOptions source: jrt:/java.base
[0.053s][info   ][class,load] jdk.net.ExtendedSocketOptions source: jrt:/jdk.net
[0.053s][info   ][class,load] java.net.SocketOption source: jrt:/java.base
[0.053s][info   ][class,load] jdk.net.ExtendedSocketOptions$ExtSocketOption source: jrt:/jdk.net
[0.053s][info   ][class,load] jdk.net.SocketFlow source: jrt:/jdk.net
[0.053s][info   ][class,load] jdk.net.ExtendedSocketOptions$PlatformSocketOptions source: jrt:/jdk.net
[0.053s][info   ][class,load] jdk.net.ExtendedSocketOptions$PlatformSocketOptions$1 source: jrt:/jdk.net
[0.054s][info   ][class,load] jdk.net.ExtendedSocketOptions$1 source: jrt:/jdk.net
[0.054s][info   ][class,load] sun.nio.ch.NioSocketImpl$FileDescriptorCloser source: jrt:/java.base
[0.055s][info   ][class,load] java.net.Socket source: jrt:/java.base
```

 我们可以`PlainSocketImpl`通过设置`Djdk.net.usePlainSocketImpl`系统属性来切换原来的实现

```bash
D:\test>java -Djdk.net.usePlainSocketImpl -XX:+TraceClassLoading JEP353  | findStr Socket

[0.041s][info   ][class,load] java.net.ServerSocket source: jrt:/java.base
[0.041s][info   ][class,load] jdk.internal.access.JavaNetSocketAccess source: jrt:/java.base
[0.041s][info   ][class,load] java.net.ServerSocket$1 source: jrt:/java.base
[0.041s][info   ][class,load] java.net.SocketOptions source: jrt:/java.base
[0.041s][info   ][class,load] java.net.SocketImpl source: jrt:/java.base
[0.045s][info   ][class,load] java.net.SocketImpl$$Lambda$1/0x0000000800ba0840 source: java.net.SocketImpl
[0.048s][info   ][class,load] sun.net.PlatformSocketImpl source: jrt:/java.base
[0.048s][info   ][class,load] java.net.AbstractPlainSocketImpl source: jrt:/java.base

[0.048s][info   ][class,load] java.net.PlainSocketImpl source: jrt:/java.base

[0.048s][info   ][class,load] java.net.AbstractPlainSocketImpl$1 source: jrt:/java.base
[0.050s][info   ][class,load] sun.net.ext.ExtendedSocketOptions source: jrt:/java.base
[0.050s][info   ][class,load] jdk.net.ExtendedSocketOptions source: jrt:/jdk.net
[0.050s][info   ][class,load] java.net.SocketOption source: jrt:/java.base
[0.051s][info   ][class,load] jdk.net.ExtendedSocketOptions$ExtSocketOption source: jrt:/jdk.net
[0.051s][info   ][class,load] jdk.net.SocketFlow source: jrt:/jdk.net
[0.051s][info   ][class,load] jdk.net.ExtendedSocketOptions$PlatformSocketOptions source: jrt:/jdk.net
[0.051s][info   ][class,load] jdk.net.ExtendedSocketOptions$PlatformSocketOptions$1 source: jrt:/jdk.net
[0.051s][info   ][class,load] jdk.net.ExtendedSocketOptions$1 source: jrt:/jdk.net
[0.051s][info   ][class,load] java.net.StandardSocketOptions source: jrt:/java.base
[0.051s][info   ][class,load] java.net.StandardSocketOptions$StdSocketOption source: jrt:/java.base
[0.053s][info   ][class,load] sun.net.ext.ExtendedSocketOptions$$Lambda$2/0x0000000800ba1040 source: sun.net.ext.ExtendedSocketOptions
[0.056s][info   ][class,load] java.net.SocketAddress source: jrt:/java.base
[0.056s][info   ][class,load] java.net.InetSocketAddress source: jrt:/java.base
[0.058s][info   ][class,load] java.net.InetSocketAddress$InetSocketAddressHolder source: jrt:/java.base
[0.059s][info   ][class,load] java.net.SocketCleanable source: jrt:/java.base
```



# 参考

**`Java 13`中的一些新功能**

- [JEP 350：动态CDS存档](https://openjdk.java.net/jeps/350)
- [JEP-351：ZGC：取消提交未使用的内存](https://openjdk.java.net/jeps/351)
- [JEP-353：重新实现旧版的套接字API](https://openjdk.java.net/jeps/353)
- [JEP-354：switch表达式（预览）（开发）](https://openjdk.java.net/jeps/354)
- [JEP-355：文本块（预览）（开发）](https://openjdk.java.net/jeps/355)



原文链接