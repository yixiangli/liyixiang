---
title: JDK命令详解
toc: true
date: 2017-03-06 11:02:22
category: 
	- 技术贴
	- Java
tags: 
    - jstat
    - jmap
---

# JDK bin下的java命令工具
jdk作为java提供的软件开发包，提供了众多基础工具，这些工具方便我们对java应用的各项指标有更准确的掌握。


## 什么是dump
** Java Dump ** 
   Java虚拟机的运行时快照。将Java虚拟机运行时的状态和信息保存到文件。
   线程Dump,包含所有线程的运行状态。纯文本格式。
   堆Dump,包含线程Dump,并包含所有堆对象的状态。二进制格式。
   
   jstack:打印线程的栈信息,制作线程Dump。
   jmap:打印内存映射,制作堆Dump。
   
说明：
   (1)不同的 JAVA虚机的线程 DUMP的创建方法和文件格式是不一样的，不同的 JVM版本，
    dump信息也有差别。
  
   (2)在实际运行中，往往一次 dump的信息，还不足以确认问题。建议产生三次 dump信息，
   如果每次 dump都指向同一个问题，我们才确定问题的典型性。 

### jps
<!--more-->
jps(Java Virtual Machine Process Status Tool)
#### 概述：
   jps位于jdk的bin目录下，其作用是显示当前系统的java进程情况，及其id号。
    jps相当于linux进程工具ps。但不像”pgrep java”或”ps -ef grep java”，
    jps并不使用应用程序名来查找JVM实例。因此，它查找所有的Java应用程序，
    包括即使没有使用java执行体的那种（例如，定制的启动 器）。另外，
    jps仅查找当前用户的Java进程，而不是当前系统中的所有进程。
    
#### 原理：
   java程序在启动以后，会在java.io.tmpdir指定的目录下，就是临时文件夹里，
   生成一个类似于hsperfdata_User的文件夹，（在Linux中为/tmp/hsperfdata_{userName}/）
   文件夹下有若干文件，名字就是java进程的pid，因此列出当前运行的java进程，
   只是把这个目录里的文件名列一下而已。 至于系统的参数什么，就可以解析这几个文件获得。
   
#### 命令：
   jps -help  帮助
   jps -l     输出主类全名或jar路径
   jps -q     显示进程号
   jps -m     显示main 传入的args参数
   jps -v     显示jvm参数设置
   
#### 用法：
   jps可以配合jstack等命令，快速查看到java进程的id号，不再需要ps -ef grep java
   面对一大堆java进程
   
#### 问题：
   jps可能会失效
   
#### 原因：
   （1）/tmp文件夹下不存在hsperfdata_{userName}文件夹 文件夹被清理
   （2）用户权限不足
   （3）进程存储地址被设置 java启动时提供了参数(-Djava.io.tmpdir)
    注意：但jps只会从/tmp下读取
   
   参考：
   http://www.hollischuang.com/archives/105

### jstat
#### 概述
jstat(JVM statistics Monitoring)是用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

#### 命令
jstat -gcutil pid 1000
 
### jstack 
 
#### 概述：
jstack用于生成java虚拟机当前时刻的线程快照,为指定的线程输出 java 的线程堆栈信息，包括了进程里的所有线程。每一个线程 frame ，包括类全名，方法名，代码行。
   
#### 用途：
   （1）java程序崩溃生成core文件(如jvm堆内存设置不当)
   （2）正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息
  
#### 命令：
   jstack -l pid
   
##### jvm线程
   JVM内部的后台线程，来执行譬如垃圾回收，或者低内存的检测等等任务，
   这些线程往往在JVM初始化的时候就存在

##### 用户级别的线程
   根据用户请求的不同而发生变化。该类线程的运行情况往往是我们所关注的重点。
   而且这一部分也是最容易产生死锁的地方。
   
##### 几种线程状态：
   NEW,未启动的。不会出现在Dump中。
   RUNNABLE,在虚拟机内执行的。
   BLOCKED,受阻塞并等待监视器锁。
   WATING,无限期等待另一个线程执行特定操作。
   TIMED_WATING,有时限的等待另一个线程的特定操作。
   TERMINATED,已退出的。
   
##### WATING on codition
   情况一：
   如果发现有大量的线程都在处在 Wait on condition，从线程 stack看， 
   正等待网络读写，这可能是一个网络瓶颈的征兆.
   因为网络阻塞导致线程无法执行。一种情况是网络非常忙，几乎消耗了所有的带宽，
   仍然有大量数据等待网络读写；另一种情况也可能是网络空闲，但由于路由等问题，
   导致包无法正常的到达。所以要结合系统的一些性能观察工具来综合分析，
   比如 netstat统计单位时间的发送包的数目，如果很明显超过了所在网络带宽的限制 ; 
   观察 cpu的利用率，如果系统态的 CPU时间，相对于用户态的 CPU时间比例较高；
   如果程序运行在 Solaris 10平台上，可以用 dtrace工具看系统调用的情况，
   如果观察到 read/write的系统调用的次数或者运行时间遥遥领先；
   这些都指向由于网络带宽所限导致的网络瓶颈。 
   情况二：
   线程在 sleep，等待 sleep的时间到了时候，将被唤醒
   
   waiting for monitor entry 和 in Object.wait() 
   waiting for monitor entry:
   synchronized
   in Object.wait() :
   在程序中有多个服务线程，设计成从一个队列里面读取请求数据。
   这个队列就是 lock以及 waiting on的对象。当队列为空的时候，
   这些线程都会在这个队列上等待，直到队列有了数据，这些线程被notify，
   当然只有一个线程获得了 lock，继续执行，而其它线程继续等待。 
##### 死锁：
   自行研究
 
### jmap
 
#### 概述：
   主要用于打印指定Java进程的共享对象内存映射或堆内存细节。
   可以使用jmap生成Heap Dump。
   
#### 用途：
   一般，在内存不足、GC异常等情况下，我们就会怀疑有内存泄露。
   这个时候我们就可以制作堆Dump来查看具体情况。分析原因。
   
#### 命令：查看堆使用情况
   jmap -heap pid
   
#### 分析：
   Attaching to process ID 22314, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 23.21-b01

using thread-local object allocation.
Parallel GC with 24 thread(s)   gc方式

Heap Configuration:    //堆内存初始化配置
   MinHeapFreeRatio = 40 //对应jvm启动参数-XX:MinHeapFreeRatio设置JVM堆最小空闲比率(default 40)
   MaxHeapFreeRatio = 70     //对应jvm启动参数 -XX:MaxHeapFreeRatio设置JVM堆最大空闲比率(default 70)
   MaxHeapSize      = 3221225472 (3072.0MB)  //对应jvm启动参数-XX:MaxHeapSize=设置JVM堆的最大大小
   NewSize          = 805306368 (768.0MB)  //对应jvm启动参数-XX:NewSize=设置JVM堆的‘新生代’的默认大小
   MaxNewSize       = 805306368 (768.0MB)  //对应jvm启动参数-XX:MaxNewSize=设置JVM堆的‘新生代’的最大大小
   OldSize          = 5439488 (5.1875MB)    //对应jvm启动参数-XX:OldSize=<value>:设置JVM堆的‘老生代’的大小
   NewRatio         = 2 //对应jvm启动参数-XX:NewRatio=:‘新生代’和‘老生代’的大小比率
   SurvivorRatio    = 8 //对应jvm启动参数-XX:SurvivorRatio=设置年轻代中Eden区与Survivor区的大小比值 
   PermSize         = 134217728 (128.0MB) //对应jvm启动参数-XX:PermSize=<value>:设置JVM堆的‘永生代’的初始大小
   MaxPermSize      = 536870912 (512.0MB) //对应jvm启动参数-XX:MaxPermSize=<value>:设置JVM堆的‘永生代’的最大大小
   G1HeapRegionSize = 0 (0.0MB)

Heap Usage: //堆内存使用情况
PS Young Generation
Eden Space: //Eden区内存分布
   capacity = 800587776 (763.5MB) //Eden区总容量
   used     = 32020768 (30.537384033203125MB) //Eden区已使用
   free     = 768567008 (732.9626159667969MB) //Eden区剩余容量
   3.999657371735838% used //Eden区使用比率
From Space: //其中一个Survivor区的内存分布
   capacity = 2359296 (2.25MB)
   used     = 2260992 (2.15625MB)
   free     = 98304 (0.09375MB)
   95.83333333333333% used
To Space: //另一个Survivor区的内存分布
   capacity = 2359296 (2.25MB)
   used     = 0 (0.0MB)
   free     = 2359296 (2.25MB)
   0.0% used
PS Old Generation //当前的Old区内存分布
   capacity = 2415919104 (2304.0MB)
   used     = 766750048 (731.2298278808594MB)
   free     = 1649169056 (1572.7701721191406MB)
   31.737405723995632% used
PS Perm Generation //当前的 “永生代” 内存分布
   capacity = 134217728 (128.0MB)
   used     = 27889792 (26.5977783203125MB)
   free     = 106327936 (101.4022216796875MB)
   20.77951431274414% used

9854 interned Strings occupying 880056 bytes.
   
#### 命令 ：查看堆内存中对象数量及大小
   jmap -histo pid
   jmap -histo:live 这个命令执行，JVM会先触发gc，然后再统计信息。
   
#### 输出到文件：
   jmap -dump:format=b,file=heapDump 31604
   jhat -port 5000 heapDump
   http://127.0.0.1:5000
   
   遇到如下信息：
   Reading from heapDump...
   Dump file created Wed Feb 24 10:43:27 CST 2016
   Snapshot read, resolving...
   Resolving 36404866 objects...
   Chasing references, expect 7280 dots
   
   有很多......
   当出现Snapshot resolved.
   Started HTTP server on port 5000
   Server is ready.
  
   
### jhat
 
#### 概述：
   一般查看堆异常情况主要看这个两个部分：
   Show instance counts for all classes (excluding platform)，平台外的所有对象信息。
   Show heap histogram 以树状图形式展示堆情况。
#### 方法：
   具体排查时需要结合代码，观察是否大量应该被回收的对象在一直被引用或者是否有
   占用内存特别大的对象无法被回收。
 
### jinfo
#### 概述：
   观察进程运行环境参数，包括Java System属性和JVM命令行参数
   
   jinfo <option> pid
   jdk1.8貌似不支持了
 
### jconsole
#### 概述：
jvm管理控制台
   
#### 例子：
   开启jconsole：      /Library/Java/Home/bin   ./jconsole
   输入远程连接：     （注意：是jmxremote的端口，不是服务端口）

### jvisualvm 
#### 概述：
   jdk1.8提供的jvm可视化图形控制台，相比jconsole新增了多种功能。
被监控的应用程序必须开启jmx端口

### javap
#### 概述：
  反编译
  
#### 命令：
  javap -c xxx.class 

### 常用性能分析工具

   HouseMD     
   BTrace     
   TProfile

 
