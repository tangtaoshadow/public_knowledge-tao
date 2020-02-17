# JVM 垃圾回收

**Repository:** [Author: GitHub](https://github.com/tangtaoshadow/)  [Gitee](https://gitee.com/tangtao2099)

**Introduction:**  

**Author:**[杭州电子科技大学](http://www.hdu.edu.cn/)  管理学院 2016级 工商管理 [唐涛](https://www.promiselee.cn/tao) 

**CreateTime:**`2020-2-17 22:26:25`

**UpdateTime:**`2020-2-18 00:17:01`

**Copyright:**  唐涛 [HOME | 首页](https://www.promiselee.cn/tao) **©**  ***2020***  [<img alt="tangtao" style="width:35px;display:inline;" src="https://www.promiselee.cn/share_static/files/github/tao-logo.svg"/>](https://www.promiselee.cn/tao)  [<img style="width:25px;display:inline;margin-bottom:5px;" alt="github" src="https://www.promiselee.cn/share_static/files/github/github-logo.svg"/>](https://github.com/tangtaoshadow)

**Email:**  <tangtao2099@outlook.com> [16011324@hdu.edu.cn](mailto:16011324@hdu.edu.cn)

**Link:**  [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)   [GitHub](https://github.com/tangtaoshadow) 



[toc]



## 引用类型

### 强引用

就是我们一般声明对象是时虚拟机生成的引用，强引用环境下，垃圾回收时需要严格判断当前对象是否被强引用，如果被强引用，则不会被垃圾回收



### 软引用

软引用一般被做为缓存来使用。与强引用的区别是，软引用在垃圾回收时，虚拟机会根据当前系统的剩余内存来决定是否对软引用进行回收。如果剩余内存比较紧张，则虚拟机会回收软引用所引用的空间；如果剩余内存相对富裕，则不会进行回收。换句话说，虚拟机在发生OutOfMemory时，肯定是没有软引用存在的。



### 弱引用

弱引用与软引用类似，都是作为缓存来使用。但与软引用不同，弱引用在进行垃圾回收时，是一定会被回收掉的，因此其生命周期只存在于一个垃圾回收周期内。







## 基础垃圾回收算法

### 引用计数

比较古老的回收算法。原理是此对象有一个引用，即增加一个计数，删除一个引用则减少一个计数。垃圾回收时，只用收集计数为0的对象。此算法最致命的是无法处理循环引用的问题。



### 标记-清除

此算法执行分两阶段。第一阶段从引用根节点开始标记所有被引用的对象，第二阶段遍历整个堆，把未标记的对象清除。此算法需要暂停整个应用🔴，同时，会产生内存碎片。



### 复制

此算法把内存空间划为两个相等的区域，每次只使用其中一个区域。垃圾回收时，遍历当前使用区域，把正在使用中的对象复制到另外一个区域中。此算法每次只处理正在使用中的对象，因此复制成本比较小，同时复制过去以后还能进行相应的内存整理，不会出现"碎片”问题。当然，此算法的缺点也是很明显的，就是需要两倍内存空间。



### 标记-整理

此算法结合了**『标记-清除』**和**『复制』**两个算法的优点。也是分两阶段，第一阶段从根节点开始标记所有被引用对象，第二阶段遍历整个堆，清除未标记对象并且把存活对象**压缩**到堆的其中一块，按顺序排放。此算法避免了**『标记-清除』**的碎片问题，同时也避免了**复制**算法的空间问题。



### 增量收集

实时垃圾回收算法，即:在应用进行的同时进行垃圾回收。不知道什么原因JDK5.0中的收集器没有使用这种算法的。



### 分代收集

基于对对象生命周期分析后得出的垃圾回收算法。把对象分为年青代、年老代、持久代，对不同生命周期的对象使用不同的算法(上述方式中的一个）进行回收。现在的垃圾回收器（从J2SE1.2开始)都是使用此算法的。



### 串行收集

串行收集使用单线程处理所有垃圾回收工作，因为无需多线程交互，实现容易，而且效率比较高。但是，其局限性也比较明显，即无法使用多处理器的优势，所以此收集适合单处理器机器。当然，此收集器也可以用在小数据量(100M左右)情况下的多处理器机器上。



### 并行收集

并行收集使用多线程处理垃圾回收工作，因而速度快，效率高。而且理论上CPU数目越多，越能体现出并行收集器的优势。



### 并发收集

相对于串行收集和并行收集而言，前面两个在进行垃圾回收工作时，需要暂停整个运行环境，而只有垃圾回收程序在运行，因此，系统在垃圾回收时会有明显的暂停，而且暂停时间会因为堆越大而越长。





## JVM分代

### 年轻代

所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。年轻代分三个区。一个Eden区，两个Survivor区(一般而言)。

大部分对象在Eden区中生成，当Eden区满时，还存活的对象将被复制到Survivor区(两个中的一个），当这个Survivor年轻代区满时，此区的存活对象将被复制到另外一个Survivor区，当这个Survivor去也满了的时候，从第一个Survivor区复制过来的并且此时还存活的对象，将被复制**『年老区(Tenured)』**。

需要注意💥，Survivor的两个区是对称的，没有先后关系，所以同一个区中可能同时存在从Eden复制过来对象，和从前一个Survivor复制过来的对象，而复制到年老区的只有从第一个Survivor区过来的对象。而且, Survivor区总有一个是空的。同时，根据程序需要,Survivor区是可以配置为多个的(多于两个)，这样可以增加对象在年轻代中的存在时间，减少被放到年老代的可能。



### 年老代

在年轻代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。



### 持久代

用于存放静态文件，如Java类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些class， 例如Hibernate等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。持久代大小通过`-XX:MaxPermSize=<N>`进行设置。







## 触发垃圾回收机制

### Scavenge GC

一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发Scavenge GC,对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。

这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。



### Full GC

对整个堆进行整理，包括Young、 Tenured和Perm。 FullGC因为需要对整个堆进行回收，所以比ScavengeGC要慢，因此应该尽可能**『减少Full GC的次数和执行Full GC的时间』**。在对JVM调优的过程中，很大一部分工作就是对于FulIGC的调节。





## 选择合适的垃圾回收算法

### 串行收集器

用单线程处理所有垃圾回收工作，因为无需多线程交互，所以效率比较高。但是，也无法使用多处理器的优势，所以此收集器适合单处理器机器。
当然，此收集器也可以用在小数据量(100M左右)情况下的多处理器机器上。可以使用`-XX:+UseSerialGC`打开。



### 并行收集器

对年轻代进行并行垃圾回收，可以减少垃圾回收时间。一般在多线程多处理器机器上使用，使用`-XX:+UseParallelGC` 打开。

并行收集器在J2SE5.0第6更新上引入，在Java SE6.0中进行了增强(可以对年老代进行并行收集)。

如果年老代不使用并发收集的话，默认是使用单线程进行垃圾回收，因此会制约扩展能力。使用`-XX:+UseParallelOldGC`打开。
使用`-XX:ParallelGCThreads=<N>`设置并行垃圾回收的线程数。此值可以设置与机器处理器数量相等。



#### 最大垃圾回收暂停

指定垃圾回收时的最长暂停时间，通过`-XX:MaxGCPauseMillis= <N>`指定。`<N>`为毫秒.如果指定了此值的话，堆大小和垃圾回收相关参数会进行调整以达到指定值。设定此值可能会减少应用的吞吐量。



#### 吞吐量

吞吐量为垃圾回收时间与非垃圾回收时间的比值，通过`-XX:GCTimeRatio-<N>`来设定，公式为`1/(1+N)`。

例如，`-XX:GCTimeRatio=19`时，表示`5%`的时间用于垃圾回收。默认情况为`99`，即`1%`的时间用于垃圾回收。





### 并发收集器

可以保证大部分工作都并发进行(应用不停止)，垃圾回收只暂停很少的时间，此收集器适合对响应时间要求比较高的中、大规模应用。使用`-XX:+UseConcMarkSweepGC`打开。

并发收集器主要减少年老代的暂停时间，他在应用不停止的情况下使用独立的垃圾回收线程，跟踪可达对象。

在每个年老代垃圾回收周期中，在收集初期并发收集器会对整个应用进行简短的暂停，在收集中还会再暂停一次。第二次暂停会比第一次稍长，在此过程中多个线程同时进行垃圾回收工作。

并发收集器使用处理器换来短暂的停顿时间。在一个N个处理器的系统上，并发收集部分使用KN个可用处理器进行回收，一般情况下1<=K<=N4.

在只有一个处理器的主机上使用并发收集器，设置为incremental mode模式也可获得较短的停顿时间。



#### 浮动垃圾

由于在应用运行的同时进行垃圾回收，所以有些垃圾可能在垃圾回收进行完成时产生，这样就造成了**『Floating Garbage』**，这些垃圾需要在下次垃圾回收周期时才能回收掉。所以，并发收集器一般需要20%的预留空间用于这些浮动垃圾。



#### Concurrent Mode Failure

并发收集器在应用运行时进行收集，所以需要保证堆在垃圾回收的这段时间有足够的空间供程序使用，否则，垃圾回收还未完成，堆空间先满了。这种情况下将会发生**『并发模式失败』**，此时整个应用将会暂停，进行垃圾回收。



#### 启动并发收集器

因为并发收集在应用运行时进行收集，所以必须保证收集完成之前有足够的内存空间供程序使用，否则会出现**『Concurrent Mode Failure』**。通过设置`-XX:CMSInitiatingOccupancyFraction=<N>`指定还有多少剩余堆时开始执行并发收集





### 总结

#### 串行处理

-   适用情况：数据量比较小(100M左右)；单处理器下并且对响应时间无要求的应用。

-   缺点：只能用于小型应用



#### 并行处理器

-   适用情况：**『对吞吐量有高要求』**，多CPU、对应用响应时间无要求的中、大型应用。举例：后台处理、科学计算。
-   缺点：垃圾收集过程中应用响应时间可能加长



#### 并发处理器

-   适用情况：**『对响应时间有高要求』**，多CPU、对应用响应时间有较高要求的中、大型应用。举例：Web服务器/应用服务器、电信交换、集成开发环境。











## 回收器选择

### 堆大小设置

```sh
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0
```



#### 参数说明

##### Xmx3550m

设置JVM最大可用内存为3550M。



##### Xms3550m

设置JVM初使内存为3550m。此值可以设置与`-Xmx`相同，以避免每次垃圾回收完成后JVM重新分配内存。



##### Xmn2g

设置年轻代大小为2G。**整个堆大小 = 年轻代大小 + 年老代大小 + 持久代大小**。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。

>   🛑 此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。



##### Xss128k

设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000-5000左右。



##### XX:NewRatio=4

设置年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代)。设置为4，则年轻代与年老代所占比值为1:4，年轻代占整个堆栈的1/5



##### Xx:SurvivorRatio=4

设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6



##### XX:MaxPermSize=16m

设置持久代大小为16m。



##### XX:Max TenuringThreshold=0

设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论。





### 回收器选择



#### 吞吐量优先的并行收集器



##### 参数: `XX:+UseParallelGC`  `XX:ParallelGCThreads=20`

```sh
java -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20
```

###### XX:+UseParallelGC

选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下，年轻代使用并行收集，而年老代仍旧使用串行收集。



###### XX:ParallelGCThreads=20

配置并行收集器的线程数，即:同时多少个线程一起进行垃圾回收。此值最好配置与处理器的数目相等。



##### 参数: `XX:+UseParallel0IdGC`

```sh
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k-XX:+Use ParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
```

###### XX:+UseParallel0IdGC

配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。



##### 参数: `-XX:MaxGCPauseMillis=100`

```sh
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100
```

##### -XX:MaxGCPauseMillis=100

设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。



##### 参数: `-XX:+UseAdaptiveSizePolicy`

```sh
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k-XX:+UseParallelGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
```

##### XX:+UseAdaptiveSizePolicy

设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。





### 响应时间优先的并发收集器



#### 参数: `-XX:+UseConcMarkSweepGC `  `-XX:+UseParNewGC`

```sh
java -Xmx3550m -Xms3550m-Xmn2g -Xss128k-XX:ParallelGCThreads-20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
```

##### -XX:+UseConcMarkSweepGC

设置年老代为并发收集。测试中配置这个以后，`-XX:NewRatio=4`的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。



##### -XX:+UseParNewGC

设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上， JVM会根据系统配置自行设置，所以无需再设置此值。



#### 参数: `-XX:CMSFulIlGCsBeforeCompaction`   `-XX:+UseCMSCompactAtFullCollection`

```sh
java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
```



##### -XX:CMSFulIlGCsBeforeCompaction

由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生**『碎片』**，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。



##### -XX:+UseCMSCompactAtFullCollection

打开对年老代的压缩。可能会影响性能，但是可以消除碎片





## 辅助信息



### -XX:+PrintGC

#### 输出形式

```sh
[GC 118250K->113543K(130112K)，0.0094143 secs] [Full GC 121376K->10414K(130112K), 0.0650971 secs] 
```



### -XX:+PrintGCDetails

#### 输出形式

```sh
[GC [DefNew:8614K->781K(9088K)，0.0123035 secs] 118250K->113543K(130112K)，0.0124633 secs] [GC [DefNew: 8614K->8614K(9088K)， 0.0000665 secs][Tenured:112761K->10414K(121024K)，0.0433488 secs] 121376K->10414K(130112K)， 0.043 6268 secs] 
```



### -XX:+PrintGCTimeStamps -XX:+PrintGC

PrintGCTimeStamps可与上面两个混合使用 

#### 输出形式

```sh
11.851:[GC 98328K->93620K(130112K)，0.0082960 secs] 
```



### XX:+PrintGCApplicationConcurrentTime

打印每次垃圾回收前，程序未中断的执行时间。可与上面混合使用。

#### 输出形式

```sh
Application time: 0.5291524 seconds
```





### XX:+PrintGCApplicationStoppedTime

打印垃圾回收期间程序暂停的时间。可与上面混合使用。

#### 输出形式

```sh
Total time for which application thr eads were stopped:0.0468229 seconds
```





### -XX:PrintHeapAtGC

打印GC前后的详细堆栈信息。

#### 输出形式

```sh
34.702: [GC {Heap before gc invocations=7:
 def new generation   total 55296K, used 52568K [0x1ebd0000, 0x227d0000, 0x227d0000)
eden space 49152K,  99% used [0x1ebd0000, 0x21bce430, 0x21bd0000)
from space 6144K,  55% used [0x221d0000, 0x22527e10, 0x227d0000)
  to   space 6144K,   0% used [0x21bd0000, 0x21bd0000, 0x221d0000)
 tenured generation   total 69632K, used 2696K [0x227d0000, 0x26bd0000, 0x26bd0000)
the space 69632K,   3% used [0x227d0000, 0x22a720f8, 0x22a72200, 0x26bd0000)
 compacting perm gen  total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)
   the space 8192K,  35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)
    ro space 8192K,  66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)
    rw space 12288K,  46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)
34.735: [DefNew: 52568K->3433K(55296K), 0.0072126 secs] 55264K->6615K(124928K)Heap after gc invocations=8:
 def new generation   total 55296K, used 3433K [0x1ebd0000, 0x227d0000, 0x227d0000)
eden space 49152K,   0% used [0x1ebd0000, 0x1ebd0000, 0x21bd0000)
  from space 6144K,  55% used [0x21bd0000, 0x21f2a5e8, 0x221d0000)
  to   space 6144K,   0% used [0x221d0000, 0x221d0000, 0x227d0000)
 tenured generation   total 69632K, used 3182K [0x227d0000, 0x26bd0000, 0x26bd0000)
the space 69632K,   4% used [0x227d0000, 0x22aeb958, 0x22aeba00, 0x26bd0000)
 compacting perm gen  total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)
   the space 8192K,  35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)
    ro space 8192K,  66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)
    rw space 12288K,  46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)
}
, 0.0757599 secs]
```



### -Xloggc:filename

与上面几个配合使用，把相关日志信息记录到文件以便分析。





## 常见配置汇总

### 堆设置

-   **-Xms**:初始堆大小
-   **-Xmx**:最大堆大小
-   **-XX:NewSize=n**:设置年轻代大小
-   **-XX:NewRatio=n:**设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
-   **-XX:SurvivorRatio=n**:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
-   **-XX:MaxPermSize=n**:设置持久代大小



### 收集器设置

-   **-XX:+UseSerialGC**:设置串行收集器
-   **-XX:+UseParallelGC**:设置并行收集器
-   **-XX:+UseParalledlOldGC**:设置并行年老代收集器
-   **-XX:+UseConcMarkSweepGC**:设置并发收集器



### 垃圾回收统计信息

-   **-XX:+PrintGC**
-   **-XX:+PrintGCDetails**
-   **-XX:+PrintGCTimeStamps**
-   **-Xloggc:filename**



### 并行收集器设置

-   **-XX:ParallelGCThreads=n**:设置并行收集器收集时使用的CPU数。并行收集线程数。
-   **-XX:MaxGCPauseMillis=n**:设置并行收集最大暂停时间
-   **-XX:GCTimeRatio=n**:设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)



### 并发收集器设置

-   **-XX:+CMSIncrementalMode**:设置为增量模式。适用于单CPU情况。
-   **-XX:ParallelGCThreads=n**:设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。





## 调优总结

### 年轻代大小选择

-   **响应时间优先的应用**：**尽可能设大，直到接近系统的最低响应时间限制**（根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。

-   **吞吐量优先的应用**：尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。



### 年老代大小选择

**响应时间优先的应用**：年老代使用并发收集器，所以其大小需要小心设置，一般要考虑**并发会话率**和**会话持续时间**等一些参数。如果堆设置小了，可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了，则需要较长的收集时间。最优化的方案，一般需要参考以下数据获得：

-   并发垃圾收集信息
-   持久代并发收集次数
-   传统GC信息
-   花在年轻代和年老代回收上的时间比例

减少年轻代和年老代花费的时间，一般会提高应用的效率



### 吞吐量优先的应用

一般吞吐量优先的应用都有一个很大的年轻代和一个较小的年老代。原因是，这样可以尽可能回收掉大部分短期对象，减少中期的对象，而年老代尽存放长期存活对象。



### 较小堆引起的碎片问题

因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。当收集器回收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，如果并发收集器找不到足够的空间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。如果出现“碎片”，可能需要进行如下配置：

-   **-XX:+UseCMSCompactAtFullCollection**：使用并发收集器时，开启对年老代的压缩。
-   **-XX:CMSFullGCsBeforeCompaction=0**：上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩





## 调优方法



### JVM调优工具

#### Jconsole

jdk自带，功能简单，但是可以在系统有一定负荷的情况下使用。对垃圾回收算法有很详细的跟踪。

#### JProfiler

商业软件，需要付费。功能强大。

#### VisualVM

JDK自带，功能强大，与JProfiler类似。推荐。



### 如何调优

#### 堆信息查看

线程监控有了堆信息查看方面的功能，我们一般可以顺利解决以下问题:

-   年老代年轻代大小划分是否合理
-   内存泄漏
-   垃圾回收算法设置是否合理



#### 线程监控

-   线程信息监控：系统线程数量。
-   线程状态监控：各个线程都处在什么样的状态下



#### 热点分析

1.  CPU热点:检查系统哪些方法占用的大量CPU时间

-   内存热点:检查哪些对象在系统中数量最大(一定时间内存活对象和销毁对象一起统计)



#### 快照

快照是系统运行到某一时刻的一个定格。在我们进行调优的时候，不可能用眼睛去跟踪所有系统变化，依赖快照功能，我们就可以进行系统两个不同运行时刻，对象（或类、线程等)的不同，以便快速找到问题举例说，我要检查系统进行垃圾回收以后，是否还有该收回的对象被遗漏下来的了。那么，我可以在进行垃圾回收前后，分别进行一次堆情况的快照，然后对比两次快照的对象情况。





### 内存泄露检查



#### 年老代堆空间被占满

##### 异常 

```sh
java.lang.OutOfMemoryError:Java heap space
```



##### 解决

这种方式解决起来也比较容易，一般就是根据垃圾回收前后情况对比，同时根据对象引用情况（常见的集合对象引用)分析，基本都可以找到泄漏点。



#### 持久代被占满

##### 异常:

```sh
java.lang.OutOfMemoryError: PermGen space
```

##### 说明

Perm空间被占满。无法为新的class分配存储空间而引发的异常。这个异常以前是没有的，但是在Java反射大量使用的今天这个异常比较常见了。主要原因就是大量动态反射生成的类不断被加载，最终导致Perm区被占满。
更可怕的是，不同的classLoader即便使用了相同的类，但是都会对其进行加载，相当于同一个东西，如果有N个classLoader那么他将会被加载N次。因此，某些情况下，这个问题基本视为无解。当然，存在大量classLoader和大量反射类的情况其实也不多。

##### 解决

1.  -XX:MaxPermSize=16m 
2.  换用JDK，比如JRocket



#### 堆栈溢出

##### 异常

```sh
java.lang.StackOverflowError
```

##### 说明

这个就不多说了，一般就是递归没返回，或者循环调用造成



#### 线程堆栈满

##### 异常

```sh
Fatal: Stack size too small
```

##### 说明

java中一个线程的空间大小是有限制的。JDK5.0以后这个值是1M。与这个线程相关的数据将会保存在其中。但是当线程空间满了以后，将会出现上面异常。

##### 解决

增加线程栈大小: `-Xss2m`。但这个配置无法解决根本问题，还要看代码部分是否有造成泄漏的部分。



#### 系统内存被占满

##### 异常

```sh
java.lang.OutOfMemoryError:unable to create new native thread
```

##### 说明

这个异常是由于操作系统没有足够的资源来产生这个线程造成的。系统创建线程时，除了要在Java堆中分配内存外，操作系统本身也需要分配资源来创建线程。因此，当线程数量大到一定程度以后，堆中或许还有空间，但是操作系统分配不出资源来了，就出现这个异常了。

分配给Java虚拟机的内存愈多，系统剩余的资源就越少，因此，当系统内存固定时，分配给Java虚拟机的内存越多，那么，系统总共能够产生的线程也就越少，两者成反比的关系。同时，可以通过修改`-Xss`来减少分配给单个线程的空间，也可以增加系统总共内生产的线程数。

##### 解决

1.  重新设计系统减少线程数量。
2.  线程数量不能减少的情况下，通过`-Xss`减小单个线程大小。以便能生产更多的线程。



## 解决之道



### 数据库、文件系统

把所有数据都放入数据库或者文件系统，这是一种最为简单的方式。在这种方式下， Java应用的内存基本上等于处理一次峰值并发请求所需的内存。数据的获取都在每次请求时从数据库和文件系统中获取。也可以理解为，一次业务访问以后，所有对象都可以进行回收了。

>   这是一种内存使用最有效的方式，但是从应用角度来说，这种方式很低效。



### 内存-硬盘映射

上面的问题是因为我们使用了文件系统带来了低效。但是如果我们不是读写硬盘，而是写内存的话效率将会提高很多。

数据库和文件系统都是实实在在进行了持久化，但是当我们并不需要这样持久化的时候，我们可以做一些变通—把内存当硬盘使。

内存-硬盘映射很好很强大，既用了缓存又对Java应用的内存使用又没有影响。Java应用还是Java应用，他只知道读写的还是文件，但是实际上是内存。

这种方式兼得的Java应用与缓存两方面的好处。memcached的广泛使用也正是这一类的代表。



### 同一机器部署多个JVM

这也是一种很好的方式，可以分为纵拆和横拆。纵可以理解为把Java应用划分为不同模块，各个模块使用一个独立的Java进程。而横拆则是同样功能的应用部署多个JVM。

通过部署多个JVM，可以把每个JVM的内存控制一个垃圾回收可以忍受的范围内即可。但是这相当于进行了分布式的处理，其额外带来的复杂性也是需要评估的。另外，也有支持分布式的这种JVM可以考虑。



### 程序控制的对象生命周期

这种方式是理想当中的方式，目前的虚拟机还没有，纯属假设。即:考虑由编程方式配置哪些对象在垃圾收集过程中可以直接跳过，减少垃圾回收线程遍历标记的时间。

这种方式相当于在编程的时候告诉虚拟机某些对象你可以在指定时间后在进行收集或者由代码标识可以收集了（类似C、C++)，在这之前你即便去遍历他也是没有效果的，他肯定是还在被引用的。

这种方式如果JVM可以实现，个人认为将是一个飞跃，Java即有了垃圾回收的优势，又有了C、C++对内存的可控性。



### 线程分配

Java的阻塞式的线程模型基本上可以抛弃了，目前成熟的NIO框架也比较多了。阻塞式IO带来的问题是线程数量的线性增长，而NIO则可以转换成为常数线程。因此，对于服务端的应用而言，NIO还是唯一选择。不过，JDK7中为我们带来的AIO是否能让人眼前一亮呢?我们拭目以待。



### 其他的JDK

本文说的都是Sun的JDK，目前常见的JDK还有JRocket和IBM的JDK。其中JRocket在10方面比Sun的高很多，不过Sun JDK6.0以后提高也很大。而且JRocket在垃圾回收方面，也具有优势，其可设置垃圾回收的最大暂停时间也是很吸引人的。不过，系统Sun的G1实现以后，在这方面会有一个质的飞跃。



[toc]



---

**原文链接**：[JVM垃圾回收](https://www.promiselee.cn/share_static/files/public_knowledge/zhihu/JVM垃圾回收.html) [GitHub](https://github.com/tangtaoshadow/public_knowledge-tao/blob/master/zhihu/JVM垃圾回收.md)

**Author**：[TangTao](https://www.promiselee.cn/tao)  **©**  ***2020***  [<img alt="tangtao" style="width:35px;display:inline;" src="https://www.promiselee.cn/share_static/files/github/tao-logo.svg"/>](https://www.promiselee.cn/tao)  [<img style="width:25px;display:inline;margin-bottom:5px;" alt="github" src="https://www.promiselee.cn/share_static/files/github/github-logo.svg"/>](https://github.com/tangtaoshadow)

**createTime**：`2020-2-18 00:08:16`

**updateTime**：`2020-2-18 00:20:21`







