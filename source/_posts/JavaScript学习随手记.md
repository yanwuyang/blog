---
title: JavaScript学习随手记
date: 2016-12-05
comments: true
categories: JavaScript
toc: false 
---

- 今天学习react-redux时学到connect函数组件时对connect函数有点蒙圈，表示不能理解，请看如下代码:
```javascript
function Users(props){
   return (<div></div>);
}
function mapStateToProps({ users }) {
  return { users };
}
export default connect(mapStateToProps)(Users);
```
<!--more-->
以为这种写发属于ES6语法糖，于是翻阅ES6标志并没有此项。苦思冥想最后灵光一闪既然是dva的函数那他的源码中肯定有此函数的定义，打开源码一看其实原理很简单,就是connect函数中返回了一个函数，然后在返回函数中在传入了参数Users。参考代码定义如下；
- 
```javascript
//获取函数名称displayName和name属于Function的原型属性
function getDisplayName(WrappedComponent) {
    return WrappedComponent.displayName || WrappedComponent.name || 'Component'
}

function connect(mapStateToProps){
   return function wrapWithConnect(WrappedComponent) {
	 var name = getDisplayName(WrappedComponent);
	 console.log("function name:"+name);
   }
}

//测试代码
function Users(props){
   return (<div></div>);
}
function mapStateToProps({ users }) {
  return { users };
}
connect(mapStateToProps)(Users);

//控制台输出
function name:Users
```

**箭头函数**
```javascript
//ES6写法
()=>{
	
}
//ES5 写法
function arrow(){
  return {}; 
}
```