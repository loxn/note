## 1.概念

###1.1 数据类型

基本类型：变量保存原始值。byte,short,int,long,char,float,double,Boolean,returnAddress 

引用类型：变量保存的是引用，对象在引用表示的地址的位置。**类类型**，**接口类型**和**数组** 

### 1.2 内存模型

![1534485691636](D:\笔记\img\14)

绿色部分是线程独享，蓝色部分是线程共享

**虚拟机栈**：每个线程有一个私有的栈，随着线程的创建而创建。栈里面存放着一种叫做“栈帧”的东西，每个方法会创建一个栈帧，栈帧中存放了局部变量表(基本数据类型和对象引用)、操作数栈、方法出口等信息。栈的大小可以固定也可以动态扩展。当栈调用深度大于JVM所允许的范围，会抛出StackOverflowError的错误 

**堆**：堆内存是JVM所有线程共享的部分，在虚拟机启动的时候就已经创建。所有的对象和数组都在堆上进行分配。这部分空间可通过GC进行回收。当申请不到空间时，会抛出OutOfMemoryError 

|      | 存储内容                                           | 特点                                             | 作用     |
| :--: | :------------------------------------------------- | :----------------------------------------------- | -------- |
|  栈  | 基本类型。局部变量，<br />方法返回值，程序运行状态 | 线程独享，速度快                                 | 程序运行 |
|  堆  | 存放引用类型                                       | 线程共享，节省了空间，大小可动态分配<br />所以慢 | 数据存储 |

Java中的参数传递时传值呢？还是传引用？

值传递。在运行栈中，基本类型和引用的处理是一样的，都是传值 。值本身无法修改

### 1.3 堆内存垃圾回收

垃圾分代回收，为什么要分代？如果不分代 的话，每次都对整个堆进行回收，太浪费时间

**年轻代**

​	Eden，新new的对象在这里，一次gc后，存活的对象会被复制到幸存区（1或者2，可以配置多个，尽量阻止对象到老年代）

​	Survivor，一个幸存区的对象经过gc后存活，就会被复制到另一个幸存区

**老年代**

​	经历多次幸存区的gc还存活的，就被复制到老年代

#### G1垃圾回收器

-XX:+UseG1GC

G1垃圾回集器为以下应用设计：

- 类似CMS收集器，可以和应用线程同时并发的执行
- 压缩空闲空间时没有GC引起的暂停时间
- 需要更可预言的GC暂停时间
- 不想牺牲大量的吞吐量性能
- 不需要特别大的Java堆

### 1.4 永久代

放置类的信息、常量池、方法数据、方法代码

方法区是jvm的规范，永久代是一种实现，java8中废弃了持久代，改为==matespace==

典型的场景是jsp过多，导致oom，java.lang.OutOfMemoryError：PermGen space

### 1.5 Matespace

​	元空间在系统内存中，不在虚拟机内存中，在一定程度上避免的类加载过多导致的oom

​	另外，jdk8将常量池和静态变量放到了堆中以字符串为例，如果字符串过多，在jdk6会包永久代溢出，而

jdk7,8会报堆溢出

### 1.6 gc触发条件

**Scavenge GC** ：Eden区满了，就触发，Eden的内存不会很大，gc会频繁进行

**Full GC** ：

​	· 年老代（Tenured）被写满

​	· 持久代（Perm）被写满 

​	· System.gc()被显示调用 

​	·上一次GC之后Heap的各域分配策略动态变化



## 2.调优工具

### 2.1 jps

查看jvm进程状态信息

jps -lv

```
-q 不输出类名、Jar名和传入main方法的参数 

-m 输出传入main方法的参数 

-l 输出main类或Jar的全限名 

-v 输出传入JVM的参数 
```

### 2.2 jstack

查看线程堆栈信息

jstack -l pid，能查看死锁

```
-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）
```



### 2.3 jstat

jvm统计监测工具

查看 1954 进程的内存使用情况，250ms时间间隔，取4组样本

==jstat -gc 1954 250 4==

```
S0C、S1C、S0U、S1U：Survivor 0/1区容量（Capacity）和使用量（Used）
EC、EU：Eden区容量和使用量
OC、OU：年老代容量和使用量
MC、MU：方法区大小和使用量
CCSC、CCSU：压缩类空间大小和使用量
YGC、YGT：年轻代GC次数和GC耗时
FGC、FGCT：Full GC次数和Full GC耗时
GCT：GC总耗时
```

### 2.4 jmap

使用==jmap -heap pid==查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况 

使用==jmap -histo[:live] pid==查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象 

使用==jmap -dump:format=b,file=/soft/app/dump.dat 1225==生成堆dump文件

使用==jhat -port 9998 /soft/app/dump.dat==查看，web访问9998端口



## 3.定位cpu占用高的线程

1. 定位app的进程id，==jps -l | grap springboot.jar==
2. 定位cpu占用高的线程，==top -Hp pid==，找到 time时间最长的线程

3. 将线程id转为16进制，==printf "%x\n" 1701==，6a5
4. jstack 查看进程堆栈信息，==jstack 1684 > tmp==，将堆栈信息复制到文本文件中
5. 搜索线程id的16进制，  6a5

## 4.jvm参数

使用**jmap -heap pid**查看jvm8配置

```shell
java -Xmx512m -Xms512m -XX:NewRatio=3 -XX:SurvivorRatio=6 -XX:+UseG1GC -jar springboot-1.0-SNAPSHOT.jar
```

```shell
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

## 5.jvm原理

![1534485863664](D:\笔记\img\15)











