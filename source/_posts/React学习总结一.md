---
title: React学习总结一
date: 2016-12-29
comments: true
categories: JavaScript
toc: true 
---

## React是什么
- React是用于构建用户界面的Javascript库 
- 1、仅仅是UI
 - 许多人使用React作为MVC架构的V层。 尽管React并没有假设过你的其余技术栈， 但它仍可以作为一个小特征轻易地在已有项目中使用 
- 2、虚拟DOM
 - React为了更高超的性能而使用虚拟DOM作为其不同的实现。 它同时也可以由服务端Node.js渲染 － 而不需要过重的浏览器DOM支持 
- 3、数据流
 - React实现了单向响应的数据流，从而减少了重复代码，这也是它为什么比传统数据绑定更简单
<!--more-->

## React翻译器经历的几个阶段
- React有一套自己的语法糖。在浏览器运行时需将React代码翻译成浏览器可识别的代码，因此需要一条React的翻译引擎。说道React的翻译引擎经历了如下几个阶段
 - **第一个阶段：在react 0.14前，浏览器端实现对jsx的编译依赖jsxtransformer.js **
 - **第一个阶段：在react 0.14后，这个依赖的库改为browser.js，页面script标签的type也由text/jsx改为text/babel**
 - **第三个阶段：react-tool更换为babel**
 
- 以上第一、第二个节点只是用来测试学习react使用，生产环境需要借助编译工具事先将jsx编译成js 



## 定义组件的三种方式
第一种：使用 React.createClass() 这种方式是传统的，将组件定义为一个类
- 
```javascript
var Welcome = React.createClass({
  //初始化state 通过getInitialState 方法中实现
  getInitialState: function() {
    return {date: []};
  },
  render: function() {
    return (
      <div className="welcomeBox">
        Hello, world! I am a WelcomeBox.
      </div>
    );
  }
});
```
第二种：使用继承React.Component，此种方式是采用ES6语法定义将组件定义为一个类
- 
```javascript
class Welcome extends React.Component {
   //初始化state 只能通过构造函数constructor中进行初始化
   constructor(props) {
	super(props);
	this.state = {date: new Date()};
   }
   render() {
	return <h1>Hello, {this.props.name}</h1>;
   }
}
```

第三种：就是一个简单的javascript函数将组件定义为一个函数。
- 当React看到表示用户定义组件的元素时，它将JSX属性作为单个对象传递给此组件。 我们称这个对象为“props”。
```javascript
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

```

***注意事项：***
- **1、无论是是将组件定义为一个函数或者类都不能修改自己的属性。**
- **2、建议从组件自己的角度来命名属性，而不是使用它的上下文。**
- **3、不能直接修改状态，或者不会重新渲染组件必须得使用setState()。**
- **4、状态的更新不要依赖this.state或者this.props来计算一下个值 因为this.state和this.props存在异步更新**。
- **5、this.setState 只会将对象进行合并不会替换之前的state。**

***问题：***
- **1、函数组件与类组件有什么区别？**
 - 函数组件与类组件直接的唯一区别就是函数组件中没有state，state是类组件特有的功能。
- **2、组件state更新会不会调用 组件卸载componentWillUnmount方法？**
 - 不会调用因为组件没有被移除Dom 只是进行了更新 此时会调用shouldComponentUpdate(object nextProps, object nextState)

