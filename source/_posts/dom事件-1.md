---
title: dom事件模型-dom0 dom2 dom3
date: 2018-12-27 17:29:42
tags: dom
---

## dom事件
dom: 文档对象模型，借助它可以实现动态访问和修改文档内容、结构或显示样式
DOM使js对html事件可以做出反应即在事件发生时执行特定javascript代码

## dom0事件模型
0级DOM有两种写法
1.标签内写onclick事件
```html
<input id="myButton" type="button" value="Press Me" onclick="alert('thanks');" >
```
2.js代码内实现
```javascript
document.getElementById("myButton").onclick = function () {
  alert('thanks')
}
```
## dom2事件模型
原生dom对象有两个方法来实现添加和移除事件处理函数: addEventListener 和 removeEventListener(ps: webkit ie方法不一样)
有三个参数: 1) 事件名(比如: 'click')
          2) 事件处理函数
          3) true: 在捕获阶段调用 false: 冒泡阶段调用
注意：
addEventListener 可以为元素添加多个事件处理函数，触发时会按照添加顺序依次调用
removeEventListener 不能移除匿名添加的处理函数

## dom0 跟 dom2事件区别
*** 1.如果定义了两个dom0事件，后定义的时间会将前面的覆盖 ***
*** 2.dom2 不会覆盖，会依次执行 ***
*** 3.dom0 dom2可以共存，不相互覆盖，但是dom0之间依然会覆盖 ***
*** 4.如果想移除dom0事件只需要把onclick赋值为null即可 ***
*** 5.dom0定义的事件处理函数会在事件流的冒泡阶段被处理 ***
dom2更加完善，可操作性更强, 但是只能在js执行之后才能绑定，dom0在加载html的时候就能绑定上,且所有浏览器都支持，没有兼容问题，但代码耦合较严重

dom3事件模型基本跟dom2一致，只是在dom2的基础上重新定义了这些事件，添加了一些新事件，最大的改变可能是dom3增加了自定义事件