---
title: JDK常用性能分析命令行工具
date: 2016-08-22 13:56:33 
comments: true 
categories: Java
toc: true 
---

# jps
jps（JVM Process Status Too)可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class,main（）函数所在的类）名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier,LVMID）。虽然功能比较单一，但它是使用频率最高的JDK命令行工具，因为其他的JDK工具大多需要输入它查询到的LVMID来确定要监控的是哪一个虚拟机进程。对于本地虚拟机进程来说，LVMID与操作系统的进程ID（Process Identifier,PID）是一致的，使用Windows的任务管理器或者UNIX的ps命令也可以查询到虚拟机进程的LVMID，但如果同时启动了多个虚拟机进程，无法根据进程名称定位时，那就只能依赖jps命令显示主类的功能才能区分了。
<!--more-->
***语法:***
```
jps [-q] [-mlvV] [<hostid>]
```

***可用的options:***

| 选项    |描叙                                         |
|:--------|:--------------------------------------------|
|-l       |输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名|
|-m		  |输出传递给main 方法的参数|
|-q		  |显示pid|
|-v       |输出传递给JVM的参数|

***实例:***
```
[root@localhost bin]# jps -l -m
63706 sun.tools.jps.Jps -l -m
79105 org.apache.catalina.startup.Bootstrap start
80838 com.yonyouup.openapi.netty.startup.Bootstrap
```

# jinfo
jinfo(Java Configuration Information)，主要用于查看指定Java进程(或核心文件、远程调试服务器)的Java配置信息。 
***语法:***
```
#指定进程号(pid)的进程
jinfo [option] <pid>
#指定核心文件
jinfo [option] <executable <core>
#指定远程调试服务器
jinfo [option] [server_id@]<remote server IP or hostname>
```

***参数:***

| 参数   |描叙                                          |
|:-------|:--------------------------------------------|
|options |选项参数是互斥的(不可同时使用)。想要使用选项参数，直接跟在命令名称后即可。|
|pid     |需要打印配置信息的进程ID。该进程必须是一个Java进程。想要获取运行的Java进程列表，你可以使用jps。|
|executable|产生核心dump的Java可执行文件。|
|core   |需要打印配置信息的核心文件。 |
|remote-hostname-or-IP   |远程调试服务器的(请查看jsadebugd)主机名或IP地址。 |
|server-id |可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。 |

***可用的options:***

| 选项            |描叙                              |
|:----------------|:---------------------------------|
| none            |打印命令行标识参数和系统属性键值对|
|-flag name       |打印指定的命令行标识参数的名称和值|
|-flag [+/-]name  |启用或禁用指定的boolean类型的命令行标识参数|
|-flag name=value |为给定的命令行标识参数设置指定的值|
|-flags           |成对打印传递给JVM的命令行标识参数|
|-sysprops        |以键值对形式打印Java系统属性|
|-h               |打印帮助信息|
|-help            |打印帮助信息|

***实例:***
```
[root@localhost bin]# jinfo 80838
Attaching to process ID 80838, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.65-b04
Java System Properties:
此处省略n行。。。。
VM Flags:

-Xms512m -Xmx1024m -Dnettyserver_home=/data/server/NettyServer
```

# jmap
jmap是java虚拟机自带的一种内存映像工具。（如：产生那些对象，及其数量）。
***语法:***
```
#指定进程号(pid)的进程
jmap [option] <pid>
#指定核心文件
jmap [option] <executable <core>
#指定远程调试服务器
jmap [option] [server_id@]<remote server IP or hostname>
```

***参数:***

| 参数   |描叙                                          |
|:-------|:--------------------------------------------|
|options |选项参数是互斥的(不可同时使用)。想要使用选项参数，直接跟在命令名称后即可。|
|pid     |需要打印配置信息的进程ID。该进程必须是一个Java进程。想要获取运行的Java进程列表，你可以使用jps。|
|executable|产生核心dump的Java可执行文件。|
|core   |需要打印配置信息的核心文件。 |
|remote-hostname-or-IP   |远程调试服务器的(请查看jsadebugd)主机名或IP地址。 |
|server-id |可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。 |

***可用的options:***

| 选项            |描叙                              |
|:----------------|:---------------------------------|
|-heap            |打印jvm heap的情况|
|-histo[:live]    |打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀”*”. 如果live子参数加上后,只统计活的对象数量. |
|-permstat        |打印classload和jvm heap长久层的信息. 包含每个classloader的名字,活泼性,地址,父classloader和加载的class数量. 另外,内部String的数量和占用内存数也会打印出来. |
|-finalizerinfo   |打印正等候回收的对象的信息.|
| -dump:dump-options| 使用hprof二进制形式,输出jvm的heap内容到文件=. live子选项是可选的，假如指定live选项,那么只输出活的对象到文件.|
|-F               |强迫.在pid没有相应的时候使用-dump或者-histo参数. 在这个模式下,live子参数无效. |
|-h/-help         |打印帮助信息|
|-j               |传递参数给jmap启动的jvm. |

***实例:***
```
[root@localhost bin]# jmap -heap 80838
Attaching to process ID 80838, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.65-b04

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio = 0
   MaxHeapFreeRatio = 100
   MaxHeapSize      = 1073741824 (1024.0MB)
   NewSize          = 1310720 (1.25MB)
   MaxNewSize       = 17592186044415 MB
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 21757952 (20.75MB)
   MaxPermSize      = 85983232 (82.0MB)
   G1HeapRegionSize = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 135266304 (129.0MB)
   used     = 94343312 (89.97279357910156MB)
   free     = 40922992 (39.02720642089844MB)
   69.74635161170664% used
From Space:
   capacity = 22020096 (21.0MB)
   used     = 17064032 (16.273529052734375MB)
   free     = 4956064 (4.726470947265625MB)
   77.49299548921131% used
To Space:
   capacity = 22020096 (21.0MB)
   used     = 0 (0.0MB)
   free     = 22020096 (21.0MB)
   0.0% used
PS Old Generation
   capacity = 358088704 (341.5MB)
   used     = 4098176 (3.9083251953125MB)
   free     = 353990528 (337.5916748046875MB)
   1.1444583295204978% used
PS Perm Generation
   capacity = 25690112 (24.5MB)
   used     = 25659024 (24.470352172851562MB)
   free     = 31088 (0.0296478271484375MB)
   99.87898846061863% used

9192 interned Strings occupying 743744 bytes.
   
```
或者
```
[root@localhost bin]#jmap -dump:format=b,file=netty.bin 80838
Attaching to process ID 80838, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.10-b01
Dumping heap to tomcat.bin ...
Finding object size using Printezis bits and skipping over...
Heap dump file created
```

# jhat

jhat(Java Heap Analysis Tool)，是JDK自带的Java堆内存分析工具

***语法:***
```
jhat [ options ] <heap-dump-file>
```


***参数:***

| 参数   |描叙                                          |
|:-------|:--------------------------------------------|
|options |选项参数.如果使用，请直接跟在命令名称之后|
|heap-dump-file |指定用于浏览的Java二进制heap dump文件。对于一个包含多个heap dump的dump文件，你可以在文件名称后面追加"#<number>"来指定文件中的某个dump|

***可用的options:***

| 选项            |描叙                                          |
|:----------------|:--------------------------------------------|
|-stack false/true|关闭跟踪对象分配调用堆栈。注意，如果heap dump中的分配位置信息不可用，你必须设置此标识为false。此选项的默认值为|
|-refs false/true |关闭对象的引用跟踪。默认为true。默认情况下，反向指针(指向给定对象的对象，又叫做引用或外部引用)用于计算堆中的所有对象|
|-port port-number|设置jhat的HTTP服务器的端口号。默认为7000|
|-exclude exclude-file|指定一个数据成员列表的文件，这些数据成员将被排除在"reachable objects"查询的范围之外。举个例子，如果文件列有java.lang.String.value，那么，当计算指定对象"o"的可达对象列表时，涉及到java.lang.String.value字段的引用路径将会被忽略掉|
|-baseline baseline-dump-file|指定一个基线heap dump。在两个heap dump(当前heap dump和基线heap dump)中存在相同对象ID的对象，不会被标记为"new"。其他的对象将被标记为"new"。这在比较两个不同的heap dump时非常有用|
|-debug int           |设置此工具的调试级别。0意味着没有调试输出。设置的值越高，输出的信息就越详细|
|-version    |报告版本号并退出|
|-h         |输出帮助信息并退出|
|-help   |输出帮助信息并退出|
|-J  |将运行时参数传递给运行jhat的JVM。例如，-J-Xmx512m设置使用的最大堆内存大小为512MB|

***实例:***
```
[root@localhost bin]# jhat netty.bin
.....
Started HTTP server on port 7000
Server is ready.
```

访问 http://localhost:7000，就可以查看详细的内存信息
有时你dump出来的堆很大，在启动时会报堆空间不足的错误，可以使用如下参数：
```
[root@localhost bin]# jhat -J-Xmx512m <heap dump file>
```

# jstat

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据，在没有GUI图形界面，只提供了纯文本控制台环境的服务器上，它将是运行期定位虚拟机性能问题的首选工具。

***语法：***
```
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```
***参数：***

| 参数   |描叙                                          |
|:-------|:--------------------------------------------|
|options |选项，主要有-gcutil,-class,-gcnew，我们一般使用 -gcutil 查看gc情况|
|vmid    |VM的进程号，即当前运行的java进程号|
|interval|间隔时间，单位为秒或者毫秒|
|count   |打印次数，如果缺省则打印无数次|

***可用的options:***

| 选项            |描叙                                          |
|:----------------|:--------------------------------------------|
|-class           |监视类装载、卸载数量、总空间以及类装载所耗时间|
|-compiler		  |输出JIT编译器编译的方法，耗时等信息|
|-gc			  |监视Java堆状况，包括Eden区，两个survivor区，老年代，永久代等的容量，已用空间，GC时间合计信息|
|-gccapacity      |监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大，最小空间|
|-gccause         |与-gcutil功能一样，单输出主要关注已使用空间占总空间的百分比|
|-gcnew           |监视新生代GC状况|
|-gcnewcapacity   |监视内容yu-gcnew基本相同，输出主要关注使用到的最大、最少空间|
|-gcold           |监视老年代GC状况|
|-gcoldcapacity   |监视内容与-gcold基本相同,输出主要关注使用到的最大、最少空间|
|-gcpermcapacity  |输出永久代使用到的最大、最少空间|
|-gcutil          |与-gc功能基本相同，单输出主要关注已使用空间占总空间的百分比|
|-printcompilation|输出已经被JIT编译的方法

***实例:***
```
[root@localhost bin]# jstat -gcutil 80838 100 5
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT   
 77.49   0.00  69.75   1.14  99.88      2    4.719     0    0.000    4.719
 77.49   0.00  69.75   1.14  99.88      2    4.719     0    0.000    4.719
 77.49   0.00  69.75   1.14  99.88      2    4.719     0    0.000    4.719
 77.49   0.00  69.75   1.14  99.88      2    4.719     0    0.000    4.719
 77.49   0.00  69.75   1.14  99.88      2    4.719     0    0.000    4.719
```
***数据含义:***

| 列   | 说明                                         |
|:-----|:--------------------------------------------|
|S0    |Heap上的 Survivor space 0 区已使用空间的百分比|
|S1	   |Heap上的 Survivor space 1 区已使用空间的百分比|
|E	   |Heap上的 Eden space 区已使用空间的百分比|
|O     |Heap上的 Old space 区已使用空间的百分比|
|P     |Perm space 区已使用空间的百分比|
|YGC   |从应用程序启动到采样时发生 Young GC 的次数|
|YGCT  |从应用程序启动到采样时 Young GC 所用的时间(单位秒)|
|FGC   |从应用程序启动到采样时发生 Full GC 的次数|
|FGCT  |从应用程序启动到采样时 Full GC 所用的时间(单位秒)|
|GCT   |从应用程序启动到采样时用于垃圾回收的总时间(单位秒)|

# jstack
jstack: 查看进程线程栈信息
如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。 如果运行的java程序呈现hung的状态，jstack可以查看线程堆栈。

***语法：***
```
#指定进程号(pid)的进程
jstack [-l] <pid>
jstack -F [-m] [-l] <pid>
#指定核心文件
jstack [-m] [-l] <executable> <core>
#指定远程调试服务器
jstack [-m] [-l] [server_id@]<remote server IP or hostname>
```
***可用的options:***

| 选项   |描叙                                          |
|:-------|:--------------------------------------------|
|-F      |pid无法响应时，强制打印堆栈|
|-m		 |打印混合模式(Java和本地C/C++帧)的堆栈跟踪信息|
|-l		 |长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表|
|-h      |输出帮助信息|

***实例:***
```
[root@localhost bin]# jstack -l 80838
2016-08-22 15:40:02
Full thread dump Java HotSpot(TM) 64-Bit Server VM (24.65-b04 mixed mode):

"RMI Scheduler(0)" daemon prio=10 tid=0x00007feb98005800 nid=0xea72 waiting on condition [0x00007febac1e5000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000000f4174500> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:186)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2043)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1079)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:807)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1068)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1130)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
        at java.lang.Thread.run(Thread.java:745)

   Locked ownable synchronizers:
        - None
.......


"VM Thread" prio=10 tid=0x00007fec1c068800 nid=0x13bcc runnable 

"GC task thread#0 (ParallelGC)" prio=10 tid=0x00007fec1c01e800 nid=0x13bc8 runnable 

"GC task thread#1 (ParallelGC)" prio=10 tid=0x00007fec1c020800 nid=0x13bc9 runnable 

"GC task thread#2 (ParallelGC)" prio=10 tid=0x00007fec1c022800 nid=0x13bca runnable 

"GC task thread#3 (ParallelGC)" prio=10 tid=0x00007fec1c024800 nid=0x13bcb runnable 

"VM Periodic Task Thread" prio=10 tid=0x00007fec1c0a8000 nid=0x13bd3 waiting on condition 

JNI global references: 255
```
将线程栈信息输出到指定文件
```
[root@localhost logs]# jstack -l 80838 > netty.stack
```


***注意***
在jdk bin目录下提供了一个集成以上命令行的一个可视化工具jvisualvm能更加直观的查看数据信息