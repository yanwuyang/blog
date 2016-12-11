---
title: react-redux
date: 2016-12-10
comments: true
categories: JavaScript
toc: false 
---

react-redux是基于redux javascript轻量级框架，为了更好的和react结合对redux提供了一些封装，一种更科学的代码组织方式，让我们更舒服地在React的代码中使用Redux。
react-redux很简单他就提供了两个模块：Provider和connect。
<!--more-->
### Provider

Provider是一个普通组件，可以作为顶层app的分发点，它只需要store属性就可以了。它会将state分发给所有被connect的组件，不管它在哪里，被嵌套多少层。
这个模块是作为整个App的容器，在你原有的App Container的基础上再包上一层，它的工作很简单，就是接受Redux的store作为props，并将其声明为context的属性之一，子组件可以在声明了contextTypes之后可以方便的通过this.context.store访问到store。不过我们的组件通常不需要这么做，将store放在context里，是为了给下面的connect用的。
- 
```javascript
ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
)
```

如下是Provider组件的核心代码：
- 
```javascript
export default class Provider extends Component {
   //在context设置store，可以被子组件引用
   getChildContext() {
       return { store: this.store }
   }

   constructor(props, context) {
      super(props, context)
      this.store = props.store
   }
   render() {
      return Children.only(this.props.children)
   }
}

```

### connect
connect是真正的重点，它是一个柯里化函数，意思是先接受两个参数（数据绑定mapStateToProps和事件绑定mapDispatchToProps），再接受一个参数（将要绑定的组件本身）：

**mapStateToProps：**构建好Redux系统的时候，它会被自动初始化，但是你的React组件并不知道它的存在，因此你需要分拣出你需要的Redux状态，所以你需要绑定一个函数，它的参数是state，简单返回你关心的几个值。
**mapDispatchToProps：**声明好的action作为回调，也可以被注入到组件里，就是通过这个函数，它的参数是dispatch，通过redux的辅助方法bindActionCreator绑定所有action以及参数的dispatch，就可以作为属性在组件里面作为函数简单使用了，不需要手动dispatch。这个mapDispatchToProps是可选的，如果不传这个参数redux会简单把dispatch作为属性注入给组件，可以手动当做store.dispatch使用。这也是为什么要柯里化的原因。

在connect函数中其实是对我们的组件进行了一层包装，在connect中获取到Provider组件在context属性中设置的store。然后在componentDidMount函数中订阅handleChange事件，在handleChange函数中获取了store.getState()并且重新设置了最新的状态this.setState({ storeState })，这样组件状态发生了修改，组件将重新被渲染。

如下是connect组件的核心代码：
- 
```javascript
export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
  return function wrapWithConnect(WrappedComponent) {
      class Connect extends Component {
	    constructor(props, context) {
	        super(props, context)
	        this.store = props.store || context.store
	    }
	    trySubscribe() {
	        if (shouldSubscribe && !this.unsubscribe) {
	            this.unsubscribe = this.store.subscribe(this.handleChange.bind(this))
	            this.handleChange()
	        }
	    }
	    componentDidMount() {
	        this.trySubscribe()
	    }
		
	    handleChange() {
	        const storeState = this.store.getState()
	        this.setState({ storeState })
	    }
		
	    render() {
	        if (withRef) {
                 this.renderedElement = createElement(WrappedComponent, {
                   ...this.mergedProps,
                    ref: 'wrappedInstance'
                 })
	        } else {
	            this.renderedElement = createElement(WrappedComponent,this.mergedProps)
	        }
	        return this.renderedElement
	    }
    }
 }
```

做好以上流程Redux和React就可以工作了。简单地说就是：
1.顶层分发状态，让React组件被动地渲染。
2.监听事件，事件有权利回到所有状态顶层影响状态。