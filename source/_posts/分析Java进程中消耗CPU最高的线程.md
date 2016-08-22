---
title: 分析Java进程中消耗CPU最高的线程
date: 2016-08-22 10:56:33 
comments: true 
categories: Java
---

第一步：查看JAVA进程（也可以使用top命令查看系统中消耗CPU最高的进程）
```
>jps -l
28070 com.yonyouup.openapi.netty.startup.Bootstrap
```
<!--more-->
第二步：查找进程中消耗CPU最高的线程ID
```
>top -p 28070H
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND    
28173 root      20   0 3673316 314988  11984 S  0.3  1.0   0:11.12 java                                          
28070 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.00 java                                          
28071 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.67 java                                          
28072 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.09 java                                          
28073 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.08 java                                          
28074 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.07 java                                          
28075 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.08 java                                          
28076 root      20   0 3673316 314988  11984 S  0.0  1.0   0:09.67 java                                          
28077 root      20   0 3673316 314988  11984 S  0.0  1.0   0:00.00 java 
```
第三步：查看线程栈信息
我们查看消耗0.3%CPU的线程28173 
将28173（10进制）转换成16进制（6e0d）
```
>jstack 28070|grep -A 12 6e0d
"nioEventLoopGroup-3-3" prio=10 tid=0x00007fb7d800b000 nid=0x6e0d runnable [0x00007fb7b48f4000]
   java.lang.Thread.State: RUNNABLE
        at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
        at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:269)
        at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:79)
        at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:87)
        - locked <0x00000000c08454f8> (a io.netty.channel.nio.SelectedSelectionKeySet)
        - locked <0x00000000c08455a8> (a java.util.Collections$UnmodifiableSet)
        - locked <0x00000000c0845400> (a sun.nio.ch.EPollSelectorImpl)
        at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:98)
        at io.netty.channel.nio.NioEventLoop.select(NioEventLoop.java:692)
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:352)
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:877)
```

A 12表示查找到所在行的后12行。28173用计算器转换为16进制6e0d，注意字母是小写。

***以上命令针对Linux系统由于Windows系统不支持查看摸个进程中的线程所以只能借助第三方工具查看（JProfiler）***