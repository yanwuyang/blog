---
title: Redux
date: 2016-12-09
comments: true
categories: JavaScript
toc: false 
---

**Redux是一个通用Javscript App模块，用做App State的管理。**
1、React有props和state: props意味着父级分发下来的属性，state意味着组件内部可以自行管理的状态，并且整个React没有数据向上回溯的能力，也就是说数据只能单向向下分发，或者自行内部消化。
2、一般构建的React组件内部可能是一个完整的应用，它自己工作良好，你可以通过属性作为API控制它。但是更多的时候发现React根本无法让两个组件互相交流，使用对方的数据。
然后这时候不通过DOM沟通（也就是React体制内）解决的唯一办法就是提升state，将state放到共有的父组件中来管理，再作为props分发回子组件。
<!--more-->
3、子组件改变父组件state的办法只能是通过onClick触发父组件声明好的回调，也就是父组件提前声明好函数或方法作为契约描述自己的state将如何变化，再将它同样作为属性交给子组件使用。
这样就出现了一个模式：数据总是单向从顶层向下分发的，但是只有子组件回调在概念上可以回到state顶层影响数据。这样state一定程度上是响应式的。
4、为了面临所有可能的扩展问题，最容易想到的办法就是把所有state集中放到所有组件顶层，然后分发给所有组件。
5、为了有更好的state管理，就需要一个库来作为更专业的顶层state分发给所有React应用，这就是Redux。让我们回来看看重现上面结构的需求：
- a. 需要回调通知state (等同于回调参数) -> action
- b. 需要根据回调处理 (等同于父级方法) -> reducer
- c. 需要state (等同于总状态) -> store

对Redux来说只有这三个要素：
- a. **action**是纯声明式的数据结构，只提供事件的所有要素，不提供逻辑。
- b. **reducer**是一个匹配函数，action的发送是全局的：所有的reducer都可以捕捉到并匹配与自己相关与否，相关就拿走action中的要素进行逻辑处理，修改store中的状态，不相关就不对state做处理原样返回。
- c. **store**负责存储状态并可以被react api回调，发布action.

下面是redux github中提供的代码样例
- 
```javascript
import { createStore } from 'redux'
// 这是一个reducer函数
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}
//创建一个包含应用程序状态的Redux存储。
let store = createStore(counter)

//订阅state改变事件  更新UI以响应状态更改。
store.subscribe(() =>
  console.log(store.getState())
)

store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```
这种设计模式采用观察者模式，让多个观察者对象同时监听某一个主题对象。这个主题对象在状态发生变化时，会通知所有观察者对象，使它们能够自动更新自己。