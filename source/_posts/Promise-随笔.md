---
title: Promise 随笔
date: 2019-01-19 20:51:14
tags: js
---

Promise如今在项目中会经常使用，在这里记录一些Promise的使用经验
Promise 只有三种状态pending(进行中)、fulfilled(已成功)、rejected(已失败), 要么成功要么失败，绝对不会有即成功又失败的情况出现
```javascript
new Promise((resolve, reject) => {
  // 执行函数
  // 只要在执行函数内部遇到resolve或者reject函数，执行函数就会进入then或者catch函数中，但不是说不会执行resove或者reject下面的代码，只是说会进入下一步的状态(成功或失败，且结果不会改变)
  resove(100)
  reject(200)
  console.log(300) // 这个依然会执行
}).then((result) => {
  // then 函数执行之后会返回一个promise实例，可以继续用then/catch进行调用
  // 成功状态
  return 300 // 内部会将300用new Promise进行封装然后return出去，这就是promise的链式调用
  // return new Promise()
}, reason => {
  // 失败状态
  return 400
  // return new Promise()
}).then((result) => {
  console.log(result) // 300/400
}).catch(reason => {
  // catch函数实际上就是 .then(null, reason => { // do some... })
  // 所以catch函数最终也会返回一个promise对象
})
```


今天在写项目接口的时候遇到一个问题，我有一篇文章的评论列表，这个列表里只有文章的内容跟发表人的userId, 页面上显示得评论肯定要有发表人的头像跟nickName的，nickName跟头像链接都在另外一张mongo表里，我就需要通过userId获取每个评论对应的发表人的userId拿到这两个信息，而且需要这些信息都拿到拼接之后再返回给前台，普通的async await肯定不能满足要求，刚开始想这是使用队列递归的方式，但是感觉那样写麻烦了，经过一段时间思考之后想到了一个简单点的办法，而且也解决了，就是用Promse.all的方法实现的，下面直接上代码
```javascript
// ... some code
comments = await Promise.all(comments.map(async item => {
  const { speakerId, commentContent, date } = item
  const { nickName, userType } = await PersonInfoModel.findOne({ userId: speakerId })
  return { nickName, userType, commentContent, date }
}))
```
上面的代码，map函数传了一个async函数，返回的是一个promise对象数组, 外面在用一个Promise.all封装一下之后会返回一个新的等待所有promise完成之后的promis对象，这个时候再用await就能拿到想要的结果了，这个问题困扰了很长一段时间了， 之前都是通过改变方法实现的，这回是靠代码解决了这个问题，所以意义重大，记录一下免得自己以后忘记！