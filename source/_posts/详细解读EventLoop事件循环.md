---
title: 详细解读EventLoop事件循环
date: 2019-02-03 03:35:53
tags: javascript
---

Event Loop 事件循环，是浏览器或者node解决javascript单线程运行时不会阻塞的一种机制，也就是我们经常说的 *** 异步 *** 的原理
在介绍Event loop之前，我想首先了解下堆栈队列的概念

## 堆、栈和队列
### 堆（Heap）
![堆、栈和队列](https://user-gold-cdn.xitu.io/2019/1/17/16859c984806c78d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
堆是一种数据结构，是利用完全二叉树维护的一组数据，堆分为两种，一种为最大堆，一种为最小堆，将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。
堆是线性数据结构，相当于一维数组，有唯一后继。(这个我也不是太明白)
### 栈（Stack）
![栈（Stack）](https://user-gold-cdn.xitu.io/2019/1/17/16859ed4f6143043?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
*** 栈 *** 在计算机科学中是限定仅在表尾进行插入或删除操作的线性表。 *** 栈 *** 是一种数据结构，它按照后进先出的原则存储数据，先进入的数据被压入栈底，最后的数据在栈顶，需要读数据的时候从栈顶开始弹出数据。
*** 栈是只能在某一端插入和删除的特殊线性表。 ***
### 队列（Queue）
![队列（Queue）](https://user-gold-cdn.xitu.io/2019/1/17/16859f2f4f5da2a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
特殊之处在于它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作，和栈一样，队列是一种操作受限制的线性表。
进行插入操作的端称为队尾，进行删除操作的端称为队头。 队列中没有元素时，称为空队列。
队列的数据元素又称为队列元素。在队列中插入一个队列元素称为入队，从队列中删除一个队列元素称为出队。因为队列只允许在一端插入，在另一端删除，所以只有最早进入队列的元素才能最先从队列中删除，故队列又称为先进先出（FIFO—first in first out）

## Event Loop
在JavaScript中，任务被分为两种，一种 *** 宏任务（MacroTask）*** 也叫Task，一种叫 *** 微任务（MicroTask） ***。
### MacroTask（宏任务）
script全部代码、setTimeout、setInterval、setImmediate（浏览器暂时不支持，只有IE10支持，具体可见MDN）、I/O、UI Rendering。
### MicroTask（微任务）
Process.nextTick（Node独有）、Promise、Object.observe(废弃)、MutationObserver（具体使用方式查看这里）
## 浏览器中的Event Loop
Javascript 有一个 main thread 主线程和 call-stack 调用栈(执行栈)，所有的任务都会被放到调用栈等待主线程执行。
### JS调用栈
JS调用栈采用的是后进先出的规则，当函数执行的时候，会被添加到栈的顶部，当执行栈执行完成后，就会从栈顶移出，直到栈内被清空。
### 同步任务和异步任务
![同步任务和异步任务](https://user-gold-cdn.xitu.io/2019/1/18/1685f03d7f88792b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
Javascript单线程任务被分为同步任务和异步任务，同步任务会在调用栈中按照顺序等待主线程依次执行(这里有一点需要注意：js是解释执行的语言，当遇到有函数执行的时候才会放到调用栈中, 并不是一开始就会按顺序直接放入调用栈中，这点可以参考：[
 调用栈](https://developer.mozilla.org/zh-CN/docs/Glossary/Call_stack))，异步任务会在异步任务有了结果后，将注册的回调函数放入任务队列中等待主线程空闲的时候（调用栈被清空），被读取到栈内等待主线程的执行。
任务队列Task Queue，即队列，是一种先进先出的一种数据结构。
![任务队列](https://user-gold-cdn.xitu.io/2019/1/18/1685f037d48da0de?imageslim)
### 事件循环的进程模型
1.选择当前要执行的任务队列，选择任务队列中最先进入的任务，如果任务队列为空即null，则执行跳转到微任务（MicroTask）的执行步骤。
2.将事件循环中的任务设置为已选择任务。
3.执行任务。
4.将事件循环中当前运行任务设置为null。
5.将已经运行完成的任务从任务队列中删除。
6.microtasks步骤：进入microtask检查点。
7.更新界面渲染。
8.返回第一步。
### 执行进入microtask检查点时，用户代理会执行以下步骤：
1.设置microtask检查点标志为true。
2.当事件循环microtask执行不为空时：选择一个最先进入的microtask队列的microtask，将事件循环的microtask设置为已选择的microtask，运行3.microtask，将已经执行完成的microtask为null，移出microtask中的microtask。
4.清理IndexDB事务
5.设置进入microtask检查点的标志为false。

执行栈在执行完同步任务后，查看执行栈是否为空，如果执行栈为空，就会去检查微任务(microTask)队列是否为空，如果为空的话，就执行Task（宏任务），否则就一次性执行完所有微任务。
每次单个宏任务执行完毕后，检查微任务(microTask)队列是否为空，如果不为空的话，会按照先入先出的规则全部执行完微任务(microTask)后，设置微任务(microTask)队列为null，然后再执行宏任务，如此循环。

可以举个例子
```javascript
console.log('script start'); // 同步任务

setTimeout(function() { // 宏任务
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() { // 微任务(比宏任务先执行)
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});
console.log('script end'); // 同步任务，所以会第一个执行
// 结果
// script start
// script end
// promise1
// promise2
// setTimeout
```
![avatar](https://user-gold-cdn.xitu.io/2019/1/18/16860ae5ad02f993?imageslim)

先c