---
title: Java并发编程-线程饥饿死锁
date: 2017-10-16 21:43:27 
comments: true 
categories: Java
toc: false
---

  最近公司项目上遇到一个问题，同事反映服务一直没有收到回调（异步接口），查看日志没看到任何的异常信息，重启服务后偶尔又出现，通过分析可能存在三种情况：第一种是数据库发生死锁了，第二种情况线程死锁。第三种情况redis连接池为空一直在等待空闲的资源
  使用查看数据库锁查询语句查询没有任何锁，那就只可能存在第二、第三种情况了。
  
<!--more-->
  其实第一、二、三种情况都可以通过线程的堆栈信息分析得出。
  输出线程堆栈信息
```java
jstack -l pid > thread.stack
```
发现线程池中所有的线程状态都是waiting。是什么原因导致了线程的状态都是waiting呢，原来是任务依赖于其他的任务，任务提交到了同一个Executor中，并且等待这个被提交任务的结果，引发死锁。第二个任务停留在工作队列中，并等待第一个任务的完成，而第一个任务又无法完成，因为他在等待第二个任务的完成。在更大的线程池中，如果所有正在执行任务的线程都由于等待其他仍处于工作队列中的任务二阻塞，那么会发生同样的问题。这种现象被称为线程饥饿死锁。

```java
public class ThreadDeadlock{

    ExecutorService exec = Executors.newSingleThreadExecutor();
	
    private AtomicInteger index = new AtomicInteger();
	
    public class Task implements Callable<String>{

        public String call() throws Exception{
            return getName();
        }
    }
	
    public String service(){
        Future<String> future=exec.submit(new Task());
        return future.get();
    }
	
    public String getName(){
        if(index.getAndIncrement()==0){
            return service();
        }
        return "hello world";
    }
	
}
```

**引发思考-任务与执行策略之间的隐性耦合**

我们已经知道，Executor框架可以将任务的提交与任务的执行策略解耦开来。就像许多对复杂过程的解耦操作那样，这种论断多少有些言过其实。虽然Executor框架为制定和修改执行策略都提供了相当大的灵活性，但并非所有的任务都能适用所有的执行策略。有些类型的任务需要明确地执行策略

**依赖性任务**
大多数行为正确的任务都是独立的：它们不依赖与其他任务的执行时序。执行结果或其他效果。当在线程池中执行独立的认为是，可以随意地改变线程池的大小和配置，这些修改只会对执行性能产生影响。然而，如果提交给线程池的任务需要依赖其他的任务，那么就隐含地给执行策略带来了约束，此时必须小心地维持这些执行策略以避免产生活跃性的问题。除非线程池无限大，否则将可能造成死锁。



 
 