---
title: 浅谈react原理
date: 2018-10-11 10:53:56
tags: react
---

react 起源于facebook的内部项目，因为该公司对当时市场上所有的MVC框架都不满意，所以就自己写了一套框架，用来架设他自己的Instagram网站。完成之后发现这套系统很好用，于是便在2013年5月开源了。react在公司项目中用了一段时间，现在简单地聊一些对于react的理解。


## 对react的认识及其优缺点
react并不是一个完整的MVC框架，最多只能当做其中的view渲染层。所以想通过react构建一个大型项目还需要引入路由(react-router)和状态管理(redux)等其他第三方库，我觉得这样挺好的，极大地提高的扩展性，如果是一些中小型项目完全没有必要引入路由及状态管理，通过组件的state就能实现绝大部分功能，但这里也有个一个问题，如果不是什么中大型项目而且功能很简单，那也完全没有必要引入react啊。。。。

### 1.主要原理
在web开发中，我们经常需要将变化的数据实时反映到UI上，这时候就需要对DOM进行操作。然而频繁的DOM操作又是产生性能瓶颈的重要原因。这个时候react引入一个堪称革命性的解决方法，在此向facebook的技术小哥致敬！！！====>虚拟DOM(Virtual DOM)：在浏览器端用javascript实现了一套DOM API(实际就是一种数据结构)
基于react开发时所有的DOM构造都是通过虚拟DOM进行，每当数据发生变化时，react会重新构建整个DOM树，然后React将当前整个DOM树和上一次的DOM树进行对比，得到DOM结构的区别，然后仅仅将需要变化的部分进行实际的浏览器DOM更新。而且React能够批处理虚拟DOM的刷新，在一个事件循环（Event Loop）内的两次数据变化会被合并，例如你连续的先将节点内容从A变成B，然后又从B变成A，React会认为UI不发生任何变化，而如果通过手动控制，这种逻辑通常是极其复杂的。尽管每一次都需要构造完整的虚拟DOM树，但是因为虚拟DOM是内存数据，性能是极高的，而对实际DOM进行操作的仅仅是Diff部分，因而能达到提高性能的目的。这样，在保证性能的同时，开发者将不再需要关注某个数据的变化如何更新到一个或多个具体的DOM元素，而只需要关心在任意一个数据状态下，整个界面是如何Render的。*** 数据驱动开发, 开发过程中更多地应该关注如何设计数据结构 ***

### 2.组件化
组件化的目的不外乎两个：项目可维护 | 代码可复用
react 完整地支持了组件化开发。官方也推荐以组件的方式去思考UI构成，将UI上的每个功能相对独立的模块定义成组件，然后将小的组件通过组合或者嵌套的方式构成大的组件，最终完成整体UI的构建。在React中，你按照界面模块自然划分的方式来组织和编写你的代码，每个组件只关心自己部分的逻辑，彼此独立，极大地方便了项目后期的维护，例如有一个项目中可能有很多地方使用了一个组件，当后面需求发生变更的时候，你只需要修改这个组件的代码且完全不用担心会对其他的影响。当你的组件拆分颗粒度足够小之后，还会带来另外一个好处，提高性能，因为如果一个组件很大或者组件拆分不彻底，会很容易导致一个问题，某一个数据发生变化却触发了整个组件的重新render，当把这个大组件拆分成多个子组件之后，单个数据的变化可能只会触发某个子组件的重新render，其他子组件则不会发生变化。

### 3.React合成事件系统
React快速的原因之一就是React很少直接操作DOM，浏览器事件也是一样。原因是太多的浏览器事件会占用很大内存。
React为此自己实现了一套合成系统，在DOM事件体系基础上做了很大改进，减少了内存消耗，简化了事件逻辑，最大化解决浏览器兼容问题。
其基本原理就是，所有在JSX声明的事件都会被委托在顶层document节点上，并根据事件名和组件名存储回调函数(listenerBank)。每次当某个组件触发事件时，在document节点上绑定的监听函数（dispatchEvent）就会找到这个组件和它的所有父组件(ancestors)，对每个组件创建对应React合成事件(SyntheticEvent)并批处理(runEventQueueInBatch(events))，从而根据事件名和组件名调用(invokeGuardedCallback)回调函数。

so 如果你实现了如下写法，并且这样的div标签有很多
```javascript
listView = list.map((item,index) => {
  return (
    <p onClick={this.handleClick} key={item.id}>{item.text}</p>
  )
})
```
React会自动帮你实现事件委托，此时就不要多此一举自己实现一套事件委托了。
由于React合成事件系统模拟事件冒泡的方法是构建一个自己及父组件队列，因此也带来一个问题，合成事件不能阻止原生事件，原生事件可以阻止合成事件。用 event.stopPropagation() 并不能停止事件传播，应该使用  event.preventDefault()。

如果你想了解更多关于react合成事件系统可以参考博客：[React源码分析7 — React合成事件系统](https://blog.csdn.net/u013510838/article/details/61224760)


### 4.组件的生命周期
首先可以看一下react组件的构成
```javascript
import React,{ Component } from 'react';

class Demo extends Component {
  constructor(props,context) {
      super(props,context)
      this.state = {
          //定义state
      }
  }
componentWillMount () {
}
componentDidMount () {
}
componentWillReceiveProps (nextProps) {
}
shouldComponentUpdate (nextProps,nextState) {
}
componentWillUpdate (nextProps,nextState) {
}
componentDidUpdate (prevProps,prevState) {
}
render () {
    return (
        <div></div>
    )
}
componentWillUnmount () {
}
}
export default Demo;
```

react的生命周期主要可以分成三个部分
1) *** 装载过程 ***
2) *** 更新过程 ***
3) *** 卸载过程 ***

** 装载过程 **
*** constructor ***
ES6中每个类的构造函数，要创造一个类的实例，必须调用这个构造函数
作为组件中第一个被调用的函数，是初始化state最好的地方，因为state可能组件的任何周期中被访问
绑定this，在构造函数中，this就是当前组件实例。并且只会执行一次this绑定，避免多次绑定造成内存泄漏

*** componentWillMount ***
这个方法基本没有啥用处，在这个函数中执行setState，组件会更新state，但是渲染一次，设置state可以放在constructor中进行

*** render ***
render其实是返回一个JSX的描述结构，最终由react进行渲染

*** componentDidMount ***
1.其被调用时，render函数返回的东西已经引发了渲染，组件已经被挂载到DOM树上。
2.当多个组件render的时候，DidMount并不是紧贴自己组件的render函数后执行，而是所有的render调用后才会执行
3.因为render函数本身并不往DOM树上渲染内容，它只是返回一个JSX对象，由React库来根据对象决定如何渲染
4.在该函数中，进行ajax请求是最合适的时机，在其中使用setState会重新触发组件渲染
5.在需要react配合其它库使用时，componentDidMount时，真实dom已经存在，此时可以获取dom进行操作

** 更新过程 **
更新过程是父组件向下传递的props或者组件自身执行setState时发生的更新动作

*** componentWillReceiveProps ***
1.弄清楚该函数被调用的时机，才能正确的使用它=>父组件的render函数被调用，那么子组件的该函数被调用
2.父组件传入的props发生变化，就会先执行这个方法，此方法可以作为props传入后，渲染之前setState的机会，并且在这个方法中调用的setState方法是不会二次渲染的

初始化组件时候此生命周期不会被调用

注：这里说的不会造成第二次的渲染，并不是说这里的setState不会生效。在这个方法里调用setState会在组件更新完成之后在render方法执行之前更新状态，将两次的渲染合并在一起。可以在componentWillReceiveProps执行setState，但是如果你想在这个方法里获取this.state得到的将会是上一次的状态

*** shouldComponentUpdate ***
1.决定了一个组件什么时候不被渲染
2.更新过程中，react首先调用该函数
3.接收新的props和state，让开发者增加必要的条件判断，有需要时更新，没有需要时返回false，组件不在向下执行生命周期方法

*** componentWillUpdate ***
初始化组件的过程中该生命周期并不会被触发。
shouldComponentUpdate返回true以后，组件进入重新渲染的流程，进入componentWillUpdate,这里同样可以拿到nextProps和nextState
可以使用该方法做一些状态更新前的准备工作
不要在此生命周期中触发setState否则会引起循环调用陷入死循环

*** render ***
render函数会插入jsx生成的dom结构，react会生成一份虚拟dom树，在每一次组件更新时，在此react会通过其diff算法比较更新前后的新旧DOM树，比较以后，找到最小的有差异的DOM节点，并重新渲染

注：react16中 render函数允许返回一个数组，单个字符串等，不在只限制为一个顶级DOM节点，可以减少很多不必要的div(当然注意升级你的react版本，将现有项目升到react16并不会出现什么bug，唯一注意的是proTypes类型检测换了名字~)即你可以想下面这样
```javascript
render () {
  return [
    <div></div>
    <div></div>
  ]
}
// 或者
render () {
  return ''
}
```

*** componentDidUpdate ***
当一个组件被挂载后，props的变化只会触发更新过程，那么怎么使其重新挂载呢？
注：使用key值，每次改变组件的key时，都会使组件重新mount。其实这并不是多此一举，虽然react框架的核心在于以极小的时间和性能成本去更新组件，每次都力求只重新渲染组件中变化的那个部分，而不是整个组件的重新渲染挂载。但是某些时候，我们需要组件的重新挂载。

*** componentWillUnmount ***
componentWillUnmount也是会经常用到的一个生命周期，初学者可能用到的比较少，但是用好这个确实很重要的哦
1.clear你在组建中所有的setTimeout,setInterval
2.移除所有组建中的监听 removeEventListener
3.也许你会经常遇到这个warning:
```javascript
an only update a mounted or mounting component. This usually means you called setState() on an       
 unmounted component. This is a no-op. Please check the code for the undefined component.
```
是因为你在组建中的ajax请求返回中setState,而你组件销毁的时候，请求还未完成，因此会报warning

可以看看下面这张生命周期的流程图可以加深对于生命周期的理解
{% asset_img react_shengmingzhouqi.png 目标图片 %}

