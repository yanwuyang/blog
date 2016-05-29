---
title: JavaScript面向对象编程
date: 2016-05-11 11:05:20
comments: true 
categories: JavaScript
toc: true
---

## this
javascript中的this和java一样都是表示当前对象，它用在对象的方法中。关键字 this 总是指向调用该方法的对象，但需要注意的是同一个函数调用的方式不一样this指向的对象也不一样如下：

```javascript
function Test(){
  alert(this);
}
//此时的this，是window弹出[object Window]
Test();
//此时的this，是当前对象的的object 弹出[object Object]
var t = new Test();
```
<!-- more -->
会出现前后不一样的原因其实很简单，因为我们直接在调用Test函数浏览器会把这个函数当成window中的一个函数，此时的this当然指向的是window。在后面我们通过new 实例化了Test此时浏览器会吧Test当做成了一个对象此时的this肯定是指向当前对象。
由此可以总结：函数在不同的环境下执行，this的指向也不同，谁调用了这个函数，this就指向谁。
### apply、call

call, apply都属于Function.prototype的一个方法,它是JavaScript引擎内在实现的,因为属于Function.prototype,所以每个Function对象实例,也就是每个方法都有call, apply属性.既然作为方法的属性,那它们的使用就当然是针对方法的了.这两个方法是容易混淆的,因为它们的作用一样,只是使用方式不同.

区分apply,call就一句话,
```javascript
foo.call(this, arg1,arg2,arg3) == foo.apply(this, arguments)==this.foo(arg1, arg2, arg3)
```
***相同点:两个方法产生的作用是完全一样的***
***不同点:方法传递的参数不同***
***call, apply作用就是借用别人的方法来调用,就像调用自己的一样.***
```javascript
//声明人类 人类含有属性、name、age、sex
function person(name,age,sex){
   this.name = name;
   this.age=age;
   this.sex=sex;
}
//声明中国人  
function chinese(name,age,sex){
  this.colour="yellow";
  //用call方式借用person,参数显式打散传递
  person.call(this, name, age, sex);
  //用apply方式借用person, 参数作为一个数组传递,
  //这里直接用JavaScript方法内本身有的arguments数组
  person.apply(this, arguments);
  //或者封装成数组
  person.apply(this, [name,age,sex]);
  
  this.sayHello=function(){
    alert("大家好我叫："+this.name+" 性别："+this.sex+" 年龄："+this.age+" 肤色："+this.colour);
  }
}
var p =new chinese("张三",18,"男");
p.sayHello();
//弹出  大家好我叫：张三 性别：男 年龄：18 肤色：yellow
chinese("张三",18,"男");
sayHello();
//弹出  大家好我叫：张三 性别：男 年龄：18 肤色：yellow
```
在这场景中, chinese方法内,name,age,sex作为方法传递的参数, 方法分别运用了apply, call去借person方法来调用,
23行由于直接调用chinese方法, 所以在该方法中的上下文对象this就是window对象.在chinese方法中 通过this.sayHello=function(){}等价于 windows.sayHello=function(){}所以最后一行syaHello调用不会报错

***应用场景:***
当参数明确时可用call, 当参数不明确时可用apply给合arguments

***总结：***通过call、apply可以改变一个函数内的this引用，使用这个特性可以实现JavaScript面向对象编程的三大特性其中的***继承***
## Prototype
javascript规定，每一个构造函数都有一个prototype属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。
这意味着。我们可以把那些不变的属性和方法，直接定义在prototype对象上。这样带来的好处就是再实例多一个对象的时候减少了内存开销，因为多个对象指向的是同一个内存地址，提高了允许的效率。类似于java中的静态方法（被static修饰的变量和方法）。

## 对于对象常用的操作

### constructor
每一个对象都会自动包含一个constructor属性，指向他们的构造函数。obj.constructor == Object
### instanceof
用于验证原型对象与实例对象之间的关系 obj  instanceof Object

### isPrototypeOf 
判断某个prototype对象和某个实例之间的关系 Object.prototype.isPrototypeOf(obj)

### hasOwnProperty
用于判断某一个属性到底是本地属性，还是继承自prototype对象的属性  obj.hasOwnProerty("name")


 ### in
某个实例是否含有某个属性，不管是不是本地属性。 “name” in Obj
in运算符还可以用来遍历某个对象的所有属性
```javascript
for(var prop in cat1) {
  alert("cat1["+prop+"]="+cat1[prop]); 
}
```

