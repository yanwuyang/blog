---
title: Java线程中断interrupt
date: 2016-09-12 16:55:01
comments: true
categories: Java
toc: false 
---

首先一个线程的终止不应该由其他线程来强制中断或停止。而是应该由线程自己自行停止。所以线程的stop,suspend,resume 都已经被抛弃了。而线程interrupt的作用也不是
中断线程，而是通知线程应该中断了。具体到底中断还是继续运行，应该有被通知的线程自己处理。
<!--more-->
在以前通过一个线程去stop停止一个线程，这种方式太过暴力而且是很不安全的。怎么说呢，线程A调用线程B的stop方法来停止线程B,调用这个方法的时候线程A并不知道线程
线程B的执行情况，这种突然间的停止会导致线程B的一些清理工作无法完成还有一个情况是执行stop方法后线程B会马上释放锁，这有可能会引发数据不同步问题。基于以上这些问题，
stop()方法被抛弃了。

如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，
则其中断状态将被清除，它还将收到一个 InterruptedException。 
如果该线程在可中断的通道上的 I/O 操作中受阻，则该通道将被关闭，该线程的中断状态将被设置并且该线程将收到一个 ClosedByInterruptException。 
如果该线程在一个 Selector 中受阻，则该线程的中断状态将被设置，它将立即从选择操作返回，并可能带有一个非零值，就好像调用了选择器的 wakeup 方法一样。 

至于为什么如果线程在调用 Object 类的 wait()、wait(long) 或 wait(long, int) 方法，或者该类的 join()、join(long)、join(long, int)、sleep(long) 或 sleep(long, int) 方法过程中受阻，
会清除中断状态，还会收到一个InterruptedException异常。调用线程interrupt方法表示线程应该终止，其他的阻塞中断也没有意义。如果继续阻塞线程也没办法收到中断信号。
所以对象的阻塞状态都会终止。并且通过异常的方式告诉对象，你的阻塞是被interrupt清除的。不是通过正常的notify(), notifyAll() 释放等待。由此可见java的设计是很合理的。





