---
title: Promise 随笔
date: 2019-01-19 20:51:14
tags: js
---

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