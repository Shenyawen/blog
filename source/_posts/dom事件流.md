---
title: dom事件流
date: 2018-12-27 18:01:39
tags: dom
---

## 事件
事件是文档或者浏览器窗口中发生的，特定的交互瞬间
事件是用户或浏览器自身执行的某种动作，如click,load和mouseover都是事件的名字。
事件是javaScript和DOM之间交互的桥梁。
你若触发，我便执行——事件发生，调用它的处理函数执行相应的JavaScript代码给出响应。
典型的例子有：页面加载完毕触发load事件；用户单击元素，触发click事件。

## Event对象
Event对象在event第一次触发的时候被创建出来，并且一直伴随着事件在DOM结构中流转的整个生命周期。event对象会被作为第一个参数传递给事件监听的回调函数。我们可以通过这个event对象来获取到大量当前事件相关的信息：
  ● *** type (String) *** — 事件的名称
  ● *** target (node) *** — 事件起源的DOM节点
  ● *** currentTarget?(node) *** — 当前回调函数被触发的DOM节点（后面会做比较详细的介绍）
  ● *** bubbles (boolean) *** — 指明这个事件是否是一个冒泡事件（接下来会做解释）
  ● *** preventDefault(function) *** — 这个方法将阻止浏览器中用户代理对当前事件的相关默认行为被触发。比如阻止 a 元素的click事件加载一个新的页面
  ● *** stopPropagation (function) *** — 这个方法将阻止当前事件链上后面的元素的回调函数被触发，当前节点上针对此事件的其他回调函数依然会被触发。（我们稍后会详细介绍。）
  ● *** stopImmediatePropagation (function) *** — 这个方法将阻止当前事件链上所有的回调函数被触发，也包括当前节点上针对此事件已绑定的其他回调函数。
  ● *** cancelable (boolean) *** — 这个变量指明这个事件的默认行为是否可以通过调用event.preventDefault来阻止。也就是说，只有cancelable为true的时候，调用event.preventDefault才能生效。
  ● *** defaultPrevented (boolean) *** — 这个状态变量表明当前事件对象的preventDefault方法是否被调用过
  ● *** isTrusted (boolean) *** — 如果一个事件是由设备本身（如浏览器）触发的，而不是通过JavaScript模拟合成的，那个这个事件被称为可信任的(trusted)
  ● *** eventPhase (number) *** — 这个数字变量表示当前这个事件所处的阶段(phase):none(0), capture(1),target(2),bubbling(3)。我们会在下一个部分介绍事件的各个阶段
  ● *** timestamp (number) *** — 事件发生的时间
此外事件对象还可能拥有很多其他的属性，但是他们都是针对特定的event的。比如，鼠标事件包含clientX和clientY属性来表明鼠标在当前视窗的位置。
我们可以使用熟悉的浏览器的调试工具或者通过console.log在控制台输出来更具体地查看事件对象以及它的属性。

## 事件流
事件发生时会在元素节点与根节点之间按照特定的顺序传播，路径所经过的所有节点都会收到该事件，这个传播过程即DOM事件流。
事件流模型：1）捕捉型事件流 (推荐使用)
          2）冒泡型事件流

## 事件阶段(Event Phases)
*** 1) 捕获阶段 ***
事件从文档的根节点出发，随着DOM树的结构向事件的目标节点流去，途中经过各个层次的DOM节点，并在各节点上触发捕获事件，直到到达事件的目标节点。捕获阶段的主要任务是建立传播路径，在冒泡阶段，事件会通过这个路径回溯到文档根节点。
我们可以通过将addEventListener的第三个参数设置成true来为事件的捕获阶段添加监听回调函数。在实际应用中，我们并没有太多使用捕获阶段监听的用例，但是通过在捕获阶段对事件的处理，我们可以阻止类似clicks事件在某个特定元素上被触发。
*** 2) 目标阶段 ***
当事件到达目标节点的，事件就进入了目标阶段。事件在目标节点上被触发，然后会逆向回流，直到传播至最外层的文档节点。
对于多层嵌套的节点，鼠标和指针事件经常会被定位到最里层的元素上。假设，你在一个<div>元素上设置了click事件的监听函数，而用户点击在了这个<div>元素内部的<p>元素上，那么<p>元素就是这个事件的目标元素。事件冒泡让我们可以在这个<div>（或者更上层的）元素上监听click事件，并且事件传播过程中触发回调函数。
*** 3) 冒泡阶段 ***
事件在目标元素上触发后，并不在这个元素上终止。它会随着DOM树一层层向上冒泡，直到到达最外层的根节点。也就是说，同一个事件会依次在目标节点的父节点，父节点的父节点。。。直到最外层的节点上被触发。
将DOM结构想象成一个洋葱，事件目标是这个洋葱的中心。在捕获阶段，事件从最外层钻入洋葱，穿过途径的每一层。在到达中心后，事件被触发（目标阶段）。然后事件开始回溯，再次经过每一层返回（冒泡阶段）。当到达洋葱表面的时候，这次旅程就结束了。
冒泡过程非常有用。它将我们从对特定元素的事件监听中释放出来，相反，我们可以监听DOM树上更上层的元素，等待事件冒泡的到达。如果没有事件冒泡，在某些情况下，我们需要监听很多不同的元素来确保捕获到想要的事件。

## 停止传播（Stopping Propagation）
可以通过调用事件对象的stopPropagation方法，在任何阶段（捕获阶段或者冒泡阶段）中断事件的传播。此后，事件不会在后面传播过程中的经过的节点上调用任何的监听函数。
调用event.stopPropagation()不会阻止当前节点上此事件其他的监听函数被调用。如果你希望阻止当前节点上的其他回调函数被调用的话，你可以使用更激进的event.stopImmediatePropagation()方法。

## 阻止浏览器默认行为
当特定事件发生的时候，浏览器会有一些默认的行为作为反应。最常见的事件不过于link被点击。当一个click事件在一个<a>元素上被触发时，它会向上冒泡直到DOM结构的最外层document，浏览器会解释href属性，并且在窗口中加载新地址的内容。
在web应用中，开发人员经常希望能够自行管理导航（navigation）信息，而不是通过刷新页面。为了实现这个目的，我们需要阻止浏览器针对点击事件的默认行为，而使用我们自己的处理方式。这时，我们就需要调用event.preventDefault().
我们可以阻止浏览器的很多其他默认行为。比如，我们可以在HTML5游戏中阻止敲击空格时的页面滚动行为，或者阻止文本选择框的点击行为。
调用event.stopPropagation()只会阻止传播链中后续的回调函数被触发。它不会阻止浏览器的自身的行为。


## 一些比较有用的事件
1.*** load *** 事件
load事件可以在任何资源（包括被依赖的资源）被加载完成时被触发，这些资源可以是图片，css，脚本，视频，音频等文件，也可以是document或者window。
```javascript
image.addEventListener('load', function(event) {
  image.classList.add('has-loaded')
});
```
2.*** onbeforeunload ***
window.onbeforeunload让开发人员可以在想用户离开一个页面的时候进行确认。这个在有些应用中非常有用，比如用户不小心关闭浏览器的tab，我们可以要求用户保存他的修改和数据，否则将会丢失他这次的操作。
```javascript
window.onbeforeunload = function() {
  if (textarea.value != textarea.defaultValue) {
    return 'Do you want to leave the page and discard changes?'
  }
}
```
3.*** resize ***
在一些复杂的响应式布局中，对window对象监听resize事件是非常常用的一个技巧。仅仅通过css来达到想要的布局效果比较困难。很多时候，我们需要使用JavaScript来计算并设置一个元素的大小。
```javascript
window.addEventListener('resize', function() {
  // do something
});
```

4.*** transitionend ***
现在在项目中，我们经常使用CSS来执行一些转换和动画的效果。有些时候，我们还是需要知道一个特定动画的结束时间。
```javascript
el.addEventListener('transitionEnd', function() {
 // do something
});
```
注意：
  ● 如果你使用@keyframe动画，那么使用animationEnd事件，而不是transitionEnd。
  ● 跟很多事件一样，transitionEnd也向上冒泡。记得在子节点上调用event.stopPropagation()或者检查event.target来防止回调函数在不该被调用的时候被调用。
  ● 事件名目前还是被各种供应商添加了不同的前缀（比如webkitTransitionEnd, msTransitionEnd等等）

5.*** animtioniteration ***
animationiteration事件会在当前的动画元素完成一个动画迭代的时候被触发。这个事件非常有用，特别是当我们想在某个迭代完成后停止一个动画，但又不是在动画过程中打断它。
```javascript
function start() {
  div.classList.add('spin');
}
function stop() {
  div.addEventListener('animationiteration', callback);
  function callback() {
    div.classList.remove('spin');
    div.removeEventListener('animationiteration', callback);
  }
}
```

6.*** error ***
当我们的应用在加载资源的时候发生了错误，我们很多时候需要去做点什么，尤其当用户处于一个不稳定的网络情况下。Financial Times中，我们使用error事件来监测文章中的某些图片加载失败，从而立刻隐藏它。由于“DOM Leven 3 Event”规定重新定义了error事件不再冒泡，我们可以使用如下的两种方式来处理这个事件。
```javascript
imageNode.addEventListener('error', function(event) {
  image.style.display = 'none'
})
```
