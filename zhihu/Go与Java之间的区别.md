# Go Vs Java



**Author：**[杭州电子科技大学](http://www.hdu.edu.cn/)  `2016级管理学院` `工商管理` `唐涛` [16011324@hdu.edu.cn](mailto:16011324@hdu.edu.cn)

**CreateTime：**`2019-10-8 09:31:18`

**UpdateTime：**`2019-10-8 09:31:22`

**Copyright:**  唐涛 2019 ©  [HOME](https://www.promiselee.cn/tao) 

**Email：**[tangtao2099@outlook.com](mailto:tangtao2099@outlook.com)

**Link：**  [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)  [GitHub](https://github.com/tangtaoshadow)  [Gitee](https://gitee.com/tangtao_2099)



---



Go，也称为Golang，是一种编程语言。作为一种用于编程的开源语言，Go使构建**可靠，简单和高效的**软件变得更加容易。Go利用`goroutines`代替了线程，它可以通过多核并发计算能大幅度提高程序的性能，但是Golang的协程如果使用不当反而会成为影响程序执行的瓶颈，它的各种浪费特性使Go变得非常突出。

Java是一种通用的计算机编程语言，它是基于类，并发和面向对象的。Java经过专门设计，只包含很少的实现依赖项。Java应用程序在JVM（Java虚拟机）上运行，它是当今最著名的编程语言之一，Java是一种用于为多种平台开发软件的编程语言。





# **详细研究GO和Java**



- Java的应用程序已编译代码或字节码可以在包括Linux，Mac操作系统和Linux在内的大多数操作系统上运行。Java的大多数语法都源自C ++和C语言。
- Java由James A. Gosling在1990年代开发，它通过产生浏览器运行的程序或applet来促进Internet用户和GUI（图形用户界面）之间的对象互通。要使用Java开发程序，我们需要一个SDK或软件开发套件，该套件通常包括解释器，文档生成器，编译器以及用于开发功能良好的应用程序的其他各种工具。
- 作为一种面向对象的编程语言，Java比Go和其他编程语言相对容易地开发OOP应用程序。Java增强了系统的可扩展性和灵活性，并使之模块化，而且在Java中没有很多实现依赖项。
- Java程序提供了在网络中的可移植性，Java对象不包含对外部数据的任何引用。此外，基于Java的网站和应用程序要等到您的设备上安装了Java后才能运行。
- Go是静态编译的语言。它由Robert Griesemer，Ken Thompson和Rob Pike于2009年创建。此语言提供垃圾回收，CSP样式的并发，内存安全和结构化类型。





 

# GO与Java之间的主要区别

- Java和Go都处理完全不同的壁垒。
- Go的指针仅限于数组和对象，它们可以提供指向任何类型的值的指针。
- Go不使用异常来显示运行时和寿命终止之类的事件，而是使用错误来显示此类事件。
- Go基本上被编译为机器代码。
- Java支持省略检查以处理和捕获错误。
- Go提供垃圾回收，但是像Java一样，它不支持完整的GC。
- Go上不允许函数重载，必须具有唯一的方法和函数名称。
- Java中没有原始的无符号数字类型，这就是Java不适合进行底层编程的原因。
- Java中的命名空间不告诉源文件位置。
- Go提供了内置数据类型（例如map和切片），以及一些通用功能（例如复制和追加等）。
- Java仅允许其中包含公共类的源文件。
- Go提供了在OS线程上运行的轻量级线程例程。
- Java 在最佳编程语言列表中排名第18 位，而Go排在第 8 位。
- Go支持复数，因为它对此具有内置支持。
- Java vs Go在多态性方面有不同的看法，Java默认情况下允许多态，而Go则不会。
- Go的API完全由Google处理。
- Java API由开放社区流程控制。
- Java默认使用虚拟方法。
- Java不允许运算符重载，这使其更轻松。



 

### Go vs Java比较表

| **The Basis Of**  Go vs Java | **GO**                                                       | **Java**                                                     |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Architecture                 | Go不提供任何VM，例如Java JVM，这种语言只能编译为c ++ / c之类的金属。 | 它结合了解释和编译方法，字节码由Java虚拟机解释，由JVM生成并由运行Java程序的系统执行的机器代码。 |
| Language                     | 它是一种独立的编程语言，至少具有两个编译器，例如gccgo和go。  | Java是一种独立的语言。                                       |
| Expression Syntax            | Go上的语法通过使用扩展Backus-Naur形式（EBNF）指定。          | 语法无处不在–独立于IDE或编译器                               |
| Mobile Support               | Go移动子存储库包括了对iOS和Android等移动平台的移动支持，并提供用于构建移动应用程序的工具。 | 取决于设备制造商                                             |
| Routing                      | 使用HTTP协议进行路由配置                                     | 使用`Akka.routing.ConsistentHashingRouter`和Akka.routing.ScatterGatherFirstCompletedRouter进行路由配置 |
| Dependency Injection         | 使用依赖注入                                                 | 使用依赖注入并允许修改                                       |
| Structure                    | 易于管理                                                     | 更好的结构，用户友好，更易于创建和维护大型应用程序。         |
| Speed                        | 比Java快                                                     | Java比Go慢                                                   |







# 结论– Go vs Java

Go由Google工程师组成，其创建目的是为软件提供快速的反应和进步，为当今的处理方法提供更好的帮助，并且比诸如C或C ++的不同框架语言提供清晰得多的人性化代码。

如果您是C或C ++开发人员，那么到那时，您可能会发现GO比它的任何外观都要优越得多。Java基本上受C语言的影响，它的大部分语法都来自C ++和C。但是，Java的底层功能少于C或C ++，C＃只是一种多范例编程语言，它取决于C编程语言，C＃或C是为.NET Framework为Microsoft开发的。

而Java使程序员能够在各种平台上运行相同的代码。因此，基于Java的应用程序通常会编译为字节码，到2012年，Java成为最著名的编程语言之一。尤其是在客户端-服务器Web应用程序平台中，Go和Java都具有类似的功能，但是在分析时它们思路是异曲同工。在Java中，偶然地有人在暗示一个函数，他们实际上是在暗示代码的特定主体，该主体包含名称和参数，而不是仅函数本身。同样，如果人们在类中提到函数，它实际上是指有时甚至是技术的一部分的功能。





## [原文链接](https://github.com/taoshadow/public_knowledge-tao/blob/master/zhihu/Go-Vs-Java.md)

































