---
title: System.setOut
---
   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;今天一同事问我System.out.println()输出的内容为什么没有输出到日志文件中。他告诉我的是他使用的weblogic中间件。说起这个真的平时没太注意。因为我之前一直使用的是jboss，查看日志也是看的jboss日志文件，通过System.out.println输出的内容也显示在了日志文件中。带着好奇心去看了下自己系统下的日志文件果然没有。于是打开jboss的log4j.xml 与自己系统下的log4j.xml进行了一下对比发现并无不同。于是打开jboss代码看了下发现jboss org.jboss.logging.Log4jService  installSystemAdapters方法中对System.setOut 有设置。
   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表示对于System.setOut不懂到网上百度了下，官方说明是通过System.setOut方法允许程序员自行定义System.out输出流， 我们可以将我们改造好的PrintStream替换java原来的System.out对象。于是他这个问题就很好解决了。
<!-- more --> 
```java
public void init() {
  PrintStream printStream = new PrintStream(System.out) {
   public void println(boolean x) {
    log(Boolean.valueOf(x));
   }

   public void println(char x) {
    log(Character.valueOf(x));
   }

   public void println(char[] x) {
    log(x == null ? null : new String(x));
   }

   public void println(double x) {
    log(Double.valueOf(x));
   }

   public void println(float x) {
    log(Float.valueOf(x));
   }

   public void println(int x) {
    log(Integer.valueOf(x));
   }

   public void println(long x) {
    log(x);
   }

   public void println(Object x) {
    log(x);
   }

   public void println(String x) {
    log(x);
   }
  };
  System.setOut(printStream);
  System.setErr(printStream);

 }

 private void log(Object info) {
  LogFactoryImpl.getLog(getClass()).info(info);
 }
```

在web的监听器里面初始下这个就可以了

下面是log4j的配置文件

```
log4j.rootLogger=INFO,Stdout,R
log4j.appender.Stdout=org.apache.log4j.ConsoleAppender
log4j.appender.Stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.Stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n
log4j.appender.R=org.apache.log4j.DailyRollingFileAppender
log4j.appender.R.File=F:/stdout.log
log4j.appender.R.datePattern='.'yyyy-MM-dd'.txt'
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n
```

