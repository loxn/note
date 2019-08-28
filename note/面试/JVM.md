<https://www.toutiao.com/i6670037145454903811/?timestamp=1553487683&app=news_article&group_id=6670037145454903811>

## **1.数据类型**

基本类型：变量保存原始值。byte 1,short 2,int 4,long 8,char 2,float 4,double 8,Boolean,returnAddress

引用类型：变量保存的是引用，占4字节，对象在引用表示的地址的位置。类类型，接口类型和数组

## **2 内存模型**

- 方法区：类信息，常量，静态变量，JIT（将常用的机器码翻译成机器指令，保存起来，如循环）
- 程序计数器：指向当前线程正在执行的字节码指令的地址，行号，线程独有
- 虚拟机栈：当前线程运行方法所需要的数据，指令，返回地址
- 永久代：类的元信息，常量，典型的场景是jsp过多，导致oom。java8改为matespace，用的是系统内存。一定程度减少了oom

![1560349134275](D:\笔记\img\1560349192916.png)

![1560349281316](D:\笔记\img\1560349281316.png)



|      | 存储内容                                     | 特点                                       | 作用     |
| ---- | -------------------------------------------- | ------------------------------------------ | -------- |
| 栈   | 基本类型。局部变量，方法返回值，程序运行状态 | 线程独享，速度快                           | 程序运行 |
| 堆   | 存放引用类型                                 | 线程共享，节省了空间，大小可动态分配所以慢 | 数据存储 |

Java中的参数传递时传值呢？还是传引用？

值传递。在运行栈中，基本类型和引用的处理是一样的，都是传值 。值本身无法修改

## **3 堆内存垃圾回收**

![1560349313368](D:\笔记\img\1560349313368.png)

- 引用计数器，如果没有对象引用就会被回收，解决不了相互引用的问题
- **可达性分析**，gc root不能达到的，直接回收

- 内存泄露：新生成的对象，gcroot一直可达，回收不了。本该被回收，却回收不了	

**3.1 引用**

<https://www.cnblogs.com/yw-ah/p/5830458.html>

**3.2回收算法**

复制回收法（年轻代使用）：幸存区之间复制来复制去

标记清除法（老年代使用）：首先扫描一次所有的GC Root，标记不可达的对象，然后回收，会有内存碎片，浪费内存

标记整理法（老年代使用）：首先标记出所有需要回收的对象，让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

**3.3年轻代回收，Minor GC**

 Eden，新new的对象在这里，一次gc后，存活的对象会被复制到幸存区（1或者2，可以配置多个，尽量阻止对象到老年代）

 Survivor，一个幸存区的对象经过gc后存活，就会被复制到另一个幸存区，年龄 +1

 经历多次幸存区的gc还存活的，就被复制到老年代，或者对象大于幸存区的容量，也直接到老年区



-XX:PretenureSizeThreshold=3145728   3M，默认的，当对象的大小大于3m，直接进入老年代

-XX:MaxTenuringThreshold=15，默认的，对象的年龄到15了就进入老年代

**老年代回收，Major GC**

老年代空间不足时，（新生对象进入）会触发Major GC，时间较长，标记清除法

 1.连续内存不足，不能分配大的对象

![1560393637945](D:\笔记\img\1560393637945.png)

![1560393661551](D:\笔记\img\1560393661551.png)

**3.4垃圾收集器**

![1560393681449](D:\笔记\img\1560393681449.png)



![1560393713724](D:\笔记\img\1560393713724.png)

- Seria：最老的，单线程收集，全程stw
- ParNew：多线程版的Seria，可设置线程数，全程stw
- Parallel Scavenge：类似ParNew，吞吐量收集器，使用户线程停止的时间最短，默认新生代收集器
- Parallel Old：Parallel Scavenge的老年代收集器，标记整理法，默认老年代收集器
- Seria old：Seria收集老年代的收集器，老年代的后备方案
- CMS（concurrent mark sweep）：标记清除法，减少回收停顿的时间，四步

 1.标记GCroot能直接关联的对象，快，stw

 2.并发标记，GCroot追踪，并发，不stw

 3.重新标记，因第二部用户线程运行时变动的对象，快，stw，比第一部慢，比第二部快

 4.并发清除，有碎片

- G1：标记整理法，适用于大内存。把堆内存分成一块块的，可以指定垃圾回收的时间，并且会优先回收 回收价值 大的块



Parallel Scavenge 和 CMS

前者减少垃圾回收时间

后者减少回收停顿时间



**3.5 G1收集器**

- 初始标记，仅标记GC Roots能直接关联到的对象。
- 并发标记

进行GC Roots Tracing的过程，并不能保证可以标记出所有的存活对象。这个阶段若发现区域对象中的所有对象都是垃圾，那个这个区域会被立即回收。

- 最终标记

    为了修正并发标记期间因用户程序继续运作而导致标记变动的那一部分对象的标记记录，G1中采用了比CMS更快的初始快照算法:snapshot-at-the-beginning (SATB)。

- 筛选回收

    首先排序各个Region的回收价值和成本，然后根据用户期望的GC停顿时间来制定回收计划，最后按计划回收一些价值高的Region中垃圾对象，回收时采用”复制”算法，从一个或多个Region复制存活对象到堆上的另一个空的Region，并且在此过程中压缩和释放内存。



**3.6 CMS和G1**

cms缺点：

1. cpu占用大
2. 内存碎片多



**3.7 安全点和安全区域**

**safepoint**可以理解成是在代码执行过程中的一些特殊位置，当线程执行到这些位置的时候，说明虚拟机当前的状态是安全的，如果有需要，可以在这个位置暂停，比如发生GC时，需要暂停暂停所以活动线程，但是线程在这个时刻，还没有执行到一个安全点，所以该线程应该继续执行，到达下一个安全点的时候暂停，等待GC结束



安全点的位置

1. 方法返回前
2. 循环跳出前
3. 异常跳出前



**safearea**，当线程在睡眠或者阻塞状态时，gc不可能等线程恢复了再运行到安全点，再停止线程。安全区域就是在一段代码中，任意位置开始gc都是安全的。当线程进入安全区域标识自己进入了安全区域，这时如果要gc，就不管这个线程了；离开安全区域时，检查gc是否完成，是继续运行，否就等待



**3.8 GC日志**

1. 查看默认的gc收集器 和 jvm默认参数

-XX:+PrintFlagsFinal

-XX:+PrintCommandLineFlags

\2. 打印日志

-XX:+PrintGCTimeStamps 

-XX:+PrintGCDetails 

-Xloggc:/home/administrator/james/gc.log

-XX:+PrintHeapAtGC

\3. 日志文件控制

-XX:-UseGCLogFileRotation

-XX:GCLogFileSize=8K

4 .查看堆dump

-XX:+HeapDumpOnOutOfMemoryError 

-XX:HeapDumpPath=/home/administrator/james/error.hprof

\5. mat分析dump文件

1.打开dominator tree，Retained Heap 代表有关系的可回收对象总和

2.看有没有GC Root指向，没有就排除这个对象，有的话，问题可能就在这里



**3.6.3 yangGC日志**

![1560393757667](D:\笔记\img\1560393757667.png)

**3.6.4 fullGC日志**

![1560393802631](D:\笔记\img\1560393802631.png)



**3.6.5 查看日志**



jmap -heap pid 堆使用情况

jstat  -gcutil （gccause）pid 1000

jstack  线程dump 

jvisualvm

jconsole



**3.6.7 内存调整**

确定活跃数据大小，fullgc后查看old去剩余的大小，多次取评价值



总堆 = 3到4倍的  活跃数据

yang = 1 到 1.5倍的 活跃数据

old = 2 到 3倍的 活跃数据

永久代 = 1.2 到 1.5倍的 fullgc 后永久代的大小 



2.调优工具

**2.1 jps**

查看jvm进程状态信息

jps -lv

-q 不输出类名、Jar名和传入main方法的参数  -m 输出传入main方法的参数  -l 输出main类或Jar的全限名  -v 输出传入JVM的参数 

 

2.2 jstack

查看线程堆栈信息，栈dump，线程的三种状态，runable waiting blocking 

jstack pid，查看栈dump	![1560393838966](D:\笔记\img\1560393838966.png)

jstack -l pid，能查看死锁

![1560393871055](D:\笔记\img\1560393871055.png)

-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况 -m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）

 

2.3 jstat

jvm统计监测工具

查看 1954 进程的内存使用情况，250ms时间间隔，取4组样本

jstat -gc 1954 250 4

![1560393903103](D:\笔记\img\1560393903103.png)

 S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
EC、EU：Eden区容量和使用量
OC、OU：年老代容量和使用量
MC、MU：方法区大小和使用量
CCSC、CCSU：压缩类空间大小和使用量
YGC、YGT：年轻代GC次数和GC耗时
FGC、FGCT：Full GC次数和Full GC耗时
GCT：GC总耗时

2.4 jmap

使用jmap -heap pid查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况

```
Heap Configuration:
   MinHeapFreeRatio         = 40 # 堆内存的空闲比例到达此值，就放大堆内存的最大预估值,<=Xmx
   MaxHeapFreeRatio         = 70 # 堆内存的空闲比例到达此值，就缩小堆内存的最大预估值,>=Xms
   MaxHeapSize              = 536870912 (512.0MB) # Xmx
   NewSize                  = 134217728 (128.0MB) # 年轻代的默认值
   MaxNewSize               = 134217728 (128.0MB) # 年轻代的最大值
   OldSize                  = 402653184 (384.0MB) # 老年代默认大小
   NewRatio                 = 3 # 年轻代大小 ：老年代大小 = 128 ：384 = 1 : 3
   SurvivorRatio            = 6 # 幸存区 ：Eden = 1 : 6
   MetaspaceSize            = 12582912 (12.0MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 4294963200 (4095.99609375MB)
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
New Generation (Eden + 1 Survivor Space):
   capacity = 117440512 (112.0MB)
   used     = 8179480 (7.800559997558594MB)
   free     = 109261032 (104.1994400024414MB)
   6.964785712105887% used
Eden Space:
   capacity = 100663296 (96.0MB) # 96 : 16 = 6 : 1
   used     = 8179480 (7.800559997558594MB)
   free     = 92483816 (88.1994400024414MB)
   8.125583330790201% used
From Space:
   capacity = 16777216 (16.0MB)
   used     = 0 (0.0MB)
   free     = 16777216 (16.0MB)
   0.0% used
To Space:
   capacity = 16777216 (16.0MB)
   used     = 0 (0.0MB)
   free     = 16777216 (16.0MB)
   0.0% used
```

使用jmap -histo[:live] pid查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象

![1560394074709](D:\笔记\img\1560394074709.png)

```
#instance 是对象的实例个数 
#bytes 是总占用的字节数 
class name 对应的就是 Class 文件里的 class 的标识 
B 代表 byte
C 代表 char
D 代表 double
F 代表 float
I 代表 int
J 代表 long
Z 代表 boolean
前边有 [ 代表数组， [I 就相当于 int[]
对象用 [L+ 类名表示
```

jmap -histo:live pid>a.log

使用jmap -dump:format=b,file=/soft/app/dump.dat 1225生成堆dump文件

使用jhat -port 9998 /soft/app/dump.dat查看，web访问9998端口



3.定位cpu占用高的线程

1. 定位app的进程id，jps -l | grap springboot.jar
2. 定位cpu占用高的线程，top -Hp pid，找到 time时间最长的线程
3. 将线程id转为16进制，printf "%x\n" 1701，6a5
4. jstack 查看进程堆栈信息，jstack 1684 > tmp，将堆栈信息复制到文本文件中
5. 搜索线程id的16进制， 6a5



## 4.jvm参数

jinfo -flag PrintGCDetails pid，查看参数

jinfo -flags pid 	查看所有参数

参数类型

**布尔类型**：-XX:+  /  -XX:-，+ 开启，- 关闭

**KV类型**：-XX:k=v

​	

**4.1堆参数**

![1560394170162](D:\笔记\img\1560394170162.png)

**4.2 回收器**

![1560394231255](D:\笔记\img\1560394231255.png)

**4.3 常用配置**

![1560395789128](D:\笔记\img\1560395789128.png)

5.调优策略**



减少创建对象的数量；

减少使用全局变量和大对象；

将转移到老年代的对象数量降低到最小；

减少 GC 的执行时间。



1. 尽量将新生对象留在新生代，-Xmn调节新生代的大小
2. 设置大对象直接进入老年代，XX:PretenureSizeThreshold 可以设置直接进入老年代的对象大小
3. 合理设置进入老年代对象的年龄，-XX:MaxTenuringThreshold 设置对象进入老年代的年龄大小，减少老年代的内存占用，降低 full gc 发生的频率



**以下情况不用调优：**

MinorGC 执行时间不到50ms；

Minor GC 执行不频繁，约10秒一次；

Full GC 执行时间不到1s；

Full GC 执行频率不算频繁，不低于10分钟1次。