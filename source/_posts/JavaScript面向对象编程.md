---
title: JavaScript面向对象编程
date: 2016-05-11 11:05:20
comments: true 
categories: JavaScript
toc: true
---

## this
javascript中的this和java一样都是表示当前对象，但需要注意的是同一个函数调用的方式不一样this指向的对象也不一样如下：

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

