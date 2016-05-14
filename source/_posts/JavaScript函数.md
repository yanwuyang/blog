---
title: JavaScript函数
date: 2013-07-18
comments: true
categories: JavaScript
toc: true 
---

## 函数概念
函数对任何语言来说都是一个核心的概念。通过函数可以封装任何多条语句，而且可以在任何地方任何时候调用执行，ECMAScript中的函数使用function关键字来声明，后跟一组参数以及函数体。函数的基本语法如下所示
<!--more-->
```javascript
function funName(arg0,arg1,.....argN){
    statements
} 
//以下是函数示例
function syaHi(name,message){
    alert("hello "+ name + " ,"+ message);
}
```
这个函数可以通过其函数名来调用，后面还要加上一对括号和参数（圆括号中的参数如果有多个，可以用逗号隔开）。调用sayHi()函数代码如下所示：
```javascript
sayHi("Nicholas","how are you today?");
```
ECMAScript 中的函数在定义时不必指定是否返回值，实际上，任何函数在任何时候都可以通过return语句后跟要返回值来实现返回值
```javascript
function sum(num1,num2){
    return num1+num2;
}
```
这个sum()函数的作用是把两个值加起来返回一个结果。我们注意到，除了return语句之外，没有任何声明表示该函数会返回一个值。调用这个函数的示例代码如下：
```javascript
var result = sum(5,10);
```
这个函数会再执行完return语句之后停止并立即退出。因此，位于return语句之后的任何代码都永远不会执行。
推荐的做法是要么让函数始终返回一个值，要么永远都不要返回值。否则，如果函数有时返回值，有时不返回值，会给调试代码带来不便。

## 理解参数
ECMAScript 函数的参数与大多数其他语言中的函数参数有所不同。ECMAScript函数不介意传递进来多少个参数，也不在乎传进来参数是什么数据类型。也就是说，即便你定义的函数只接收两个参数，在调用这个函数时也未必一定要传递两个参数。可以传递一个、三个甚至不传，而解析器永远不会有什么怨言。只所以会这样，因为是ECMAScript中的参数在内部是用一个数组来表示的。函数接收到的始终都是这个数组，而不关心数组中包含哪些参数，在函数体内可以通过arguments对象来访问这个数组，从而获取传递给函数的每一个参数。
其实，arguments对象只是与数组类似（它并不是Array的实例），因为可以用使用方括号语法访问它的每一个元素（即第一个元素是arguments[0],第二个元素是arguments[1],依次类推），使用length属性来确定传递进来多少个参数。
```javascript
function doAdd(num1,num2){
    if(arguments[0]==num1){
        alert(true);
    }
    if(argumengs[1]==num2){
        alert(true);
    }
}
```
结果会弹出两个alert这说明 arguments[0]的值等于num1，因此他们可以互换使用。
关于arguments的行为，还有一点比较有意思。那就是它的值永远与对应命名参数的值保持同步。
```javascript
function doAdd(num1,num2){
    arguments[1]=10;
    alert(arguments[0]+num2)；
}
```
每次执行这个doAdd()函数都会重写第二个参数，将第二个参数的值改为10.因为arguments对象中的值会自动反映到对应的命名参数，所以修改了arguments[1],也就是修改了num2，结果他们的值都会变成10。不过，这并不是说明读取这两个值会访问相同的内存空间；它们的内存空间是独立的，但他们的值是同步的。另外只传入一个参数，那么为arguments[1]设置的值不会反应到命名参数中。这是因为arguments对象的长度是由传入的参数个数决定，不是由定义函数时的命名参数的个数决定的。

##  没有重载
ECMAScript函数不能像传统意义上那样实现重载。而在其他语言（如Java）中，可以为一个函数编写两个定义，只要这两个函数的签名（接受的参数的类型和数量）不同即可。ECMAScript函数没有签名，因为其参数是由包含零或多个值得数组来表示。而没有函数签名，真正的重载是不可能做到的。如果再ECMAScript中定义了两个名字相同的函数，则改名字只属于后定义的函数。
关于参数还要记住最后一点：没有传递值得命名参数将自动赋予undefined值。这跟定义了变量但又没有初始化一样，