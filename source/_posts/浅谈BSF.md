---
title: 浅谈BSF
date: 2013-01-30 21:28:01
comments: true
categories: BSF
toc: true 
---

### BSF是什么
 BSF(Bean Scripting Framework)是一个支持在java应用程序内调用脚本语言，并且支持脚本语言直接访问java对象和方法的一个开源项目，有了他，你能在你的java application中使用javascript、Python、XSLT、tcl等等。反过来也可以在script language中调用任何注册过了的javaBean、java Object 它提供了完整的API实现通过Java访问脚本语言的引擎。
 
<!--more-->
### 如何使用
 在你的java 应用程序中导入bsf.jar包如果你使用的脚本是javascript那还需要导入js.jar包，其他脚本语言类似导入相对应的jar即可下面我就以javascript为列简单说明下BSF的使用将会已代码的形式展示给大家。
 
### Java Bean中
```java
package cn.com.bsf.test; 
         
public class BeanDemo{ 
     private String name; 
              
     public void setName(String name){ 
         this.name=name; 
     } 
     public String getName(){ 
          return name 
     } 
} 
         
package cn.com.bsf.test; 
import java.io.BufferedInputStream; 
import java.io.BufferedReader; 
import java.io.IOException; 
import java.io.InputStream; 
import java.io.InputStreamReader; 
import java.util.HashMap; 
import java.util.Map; 
         
import org.apache.bsf.BSFEngine; 
import org.apache.bsf.BSFException; 
import org.apache.bsf.BSFManager; 
public class BSFExample{ 
         
    public static void main(String[] args){ 
        BSFManager bsfManager= new BSFManager(); 
        BeanDemo demo= new BeanDemo(); 
        //在BSF中注册java bean 
        bsfManager.declareBean("demo", demo, demo.getClass()); 
        bsfManager.declareBean("out", System.out, System.out.getClass()); 
        //得到BSF引擎 
        BSFEngine bsfEngine = bsfManager.loadScriptingEngine("javascript"); 
        //读取js文件 
        InputStream stream = BSFDemo.class.getResourceAsStream("/bsf_001.js"); 
        InputStreamReader reader = new InputStreamReader(new BufferedInputStream(stream)); 
        BufferedReader bufR = new BufferedReader(reader); 
        StringBuffer sb = new StringBuffer(); 
        String content = bufR.readLine(); 
        while (content != null) { 
           sb.append(content); 
           content = bufR.readLine(); 
        } 
        bsfEngine.exec("javascript", 0, 0, sb.toString()); 
       //执行脚本中的某个方法 
        Object o = bsfEngine.eval("javascript", 0, 0, "test()");  
        Map map = (HashMap) o; 
        System.out.println(map.get("age")); 
        System.out.println(demo.getName()); 
    } 
} 
```

输出结果
```
hello world 
10
张三
```
### 脚本
*** bsf_001.js***
```javascript
importPackage(java.util); 
function test(){ 
    reflection.setName("张三"); 
    out.println("hello world"); 
    var map = new HashMap(); 
    map.put("name","张三"); 
    map.put("age","10"); 
    return map; 
}
```

*** bsf_002.js***
```javascript
importClass(Packages.cn.com.bsf.test.AppDemo); 
var appDemo = new AppDemo(); 
var str=appDemo.query("select * from bsf"); 
out.println(str);
```
bsf_002.js与bsf_001.js的不同的是脚本代码没有包含在function中所以bsf执行稍微会有所不同
Object o = bsfEngine.eval("javascript", 0, 0, "bsf_002.js的脚本内容");

不知道大家注意到了importClass、importPackage没有，下面我就解释下他们的用处

***importClass 是在我们的脚本中引入我们的javaBean如果是jdk中自带的javaBea不需要packages关键字反之需要例如***
***importClass(Packages.cn.com.bsf.test.BeanDemo);***
***importClass(java.util.HashMap);***
***importPackage是在我们的脚本中引入包 引入包后就可以使用这个包中所有javaBean其语法如下他会importClass类似自定义的java包需要加Packages为前缀。***
***importPackage(Packages.cn.com.bsf.test）;***
***importPackage(java.util);***
