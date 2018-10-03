---
title: process.env.NODE_ENV用途
date: 2018-09-03 14:52:52
tags: javascript
---

process是nodeJS的一个全局对象，在node代码的所有位置都能访问到该对象不需要通过reqiure方法

## process.env.NODE_ENV
process.env: [process.env属性返回一个包含用户环境信息的对象](http://nodejs.cn/api/process.html#process_process_env)。
即返回项目运行所在环境的一些信息

### 用途
NODE_ENV属性并不属于process.env对象，是开发人员自定义的一个属性，你可以使用这个属性也可以不使用这个属性来区分不同环境(开发，生产和测试等等)而使用不同的应用程序打包、构建、运行策略，但是目前NODE_ENV已经成为一个前端工程化的使用规范

一般情况下会有以下两个值：
```bash
process.env.NODE_ENV === 'development' // 也可以简写成dev,表示开发环境
process.env.NODE_ENV === 'production' // 也可以简写成prod,表示生产环境
```
### 如何webpack打包中使用
1.如果只想在webpack配置中使用：
```json
// package.json
{
  "scripts": {
    "dev": "NODE_ENV=development webpack --config webpack.dev.config.js"
  }
}
```
这样就可以在webpack.config.js中通过process.env.NODE_ENV访问到值，但是无法在项目代码(即业务代码)中访问到process.env.NODE_ENV
注意：
当你使用NODE_ENV=development来设置环境变量的时候在windows上有问题，当你希望可以在windows mac上同时跑项目的时候，可以使用cross-env，这个包可以让同样的写法在两端表现一致
```json
// package.json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack --config webpack.dev.config.js"
  }
}
```
2.如果想在项目代码中访问到process.env.NODE_ENV
可以通过webpack pluging实现
```javascript
const webpack = require('webpack');
module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"development"'
    })
  ]
}
```




