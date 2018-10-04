---
title: node读写excel表格
date: 2018-10-04 19:25:20
tags: nodeJS
---

发现一个神器，可以读取excel表格内容并将其转化成json数据，卧槽，这个有点牛逼啊，既然这样就可以做很多事儿了。

### 安装
```bash
npm i node-xlsx -S
```
### 使用
```javascript
const xlsx = require('node-xlsx')

module.exports = function readExel () {
  const workExcel = xlsx.parse('app.xlsx')
  console.log('表格', workExcel[0].data)
}
```
效果如下
{% asset_img node_excel.png 目标图片 %}