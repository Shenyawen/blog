---
title: mongoose 常用api
date: 2018-12-16 16:39:27
tags: nodeJS
---

一般情况下，我们并不会直接使用MongoDB的函数来操作MongoDB数据库 mongose就是一款在node.js环境中操作MongoDB数据库的接口工具库(另外推荐一个GUI的工具: *** Robo 3T ***)
```base
npm i mongoose -S
```
## 基本方法
### 如何连接数据库
因为mongose连接数据库不是一直有效，在每次操作数据库之前都需要连接下数据库，所以需要废话不多说直接上代码
```javascript
// db.js
const mongoose = require('mongoose')
// collectionName 数据库名
module.exports = async (collectionName) => {
  const DB_URL = `mongodb://localhost:27017/${collectionName}`
  return new Promise((resolve, reject) => {
    mongoose.connect(DB_URL, { useNewUrlParser: true }, function (err) {
      if (err) {
        console.log('连接失败' + err)
        reject(err)
      } else {
        console.log('连接成功' + DB_URL)
        resolve(mongoose)
      }
    })
  })
}
```

利用mongose操作数据库，首先需要理解Schema 跟 Model

### Schema(类似于表结构)
一种以文件形式存储的数据库模型骨架，无法直接通往数据库端，也就是说它不具备对数据库的操作能力.可以说是数据属性模型(传统意义的表结构)，又或着是“集合”的模型骨架
  1.添加属性
  ```javascript
  Schema.add( { name: 'String', email: 'String', age: 'Number' } )
  ```
  2.创建公共方法
  ```javascript
  Schema.method( 'say', function(){console.log('hello')} )
  ```
  3.添加静态方法
  ```javascript
  // 静态方法，只限于在Model层就能使用
  Schema.static( 'say', function(){console.log('hello')} )
  ```
  4.追加方法
  ```javascript
  // 静态方法，只限于在Model层就能使用
  Schema.methods.say = function(){console.log('hello')}
  ```
### Model
由Schema构造生成的模型，除了Schema定义的数据库骨架以外，还具有数据库操作的行为，类似于管理数据库属性、行为的类

一般情况下，我会将Schema跟Model定义在一个文件中，直接通过导出的Model来操作数据库
```javascript
const mongoose = require('mongoose')
const Schema = mongoose.Schema

const AgentApply = new Schema({
  handlers: [{
    code: String,
    isAgreed: Boolean
  }],
  applicant: String,
  inviteCode: String,
  referCode: String,
  agentType: { type: String, default: 'B' },
  isCompleted: { type: Boolean, default: false },
  isInvalid: { type: Boolean, default: false },
  date: { type: Number, default: Date.now() }
}, {
  collection: 'AgentApply'
})
// AgentApply是集合名，不存在则会自动创建该集合
module.exports = mongoose.Model('AgentApply', AgentApply) // <= Model
```

当定义完Model之后就可以通过Model来增删查改数据了(在一定程度上你可以认为那些操作方法都是Model对象上的方法)

### Entity
由Model创建的实体，使用save方法保存数据，Model和Entity都有能影响数据库的操作，但Model比Entity更具操作性
```javascript
// 表示在AgentApply集合中创建一条数据 (注意这是一个异步行为)
new AgentApplyModel({ applicant: xxxxx, inviteCode: xxxx... })
```

### 游标
MongoDB 使用游标返回find的执行结果.客户端对游标的实现通常能够对最终结果进行有效的控制。可以限制结果的数量，略过部分结果，根据任意键按任意顺序的组合对结果进行各种排序，或者是执行其他一些强的操作。

### ObjectId
存储在mongodb集合中的每个文档（document）都有一个默认的主键_id，这个主键名称是固定的，它可以是mongodb支持的任何数据类型，默认是ObjectId。

ObjectId是一个12字节的 BSON 类型字符串。按照字节顺序，依次代表： 
- 4字节：UNIX时间戳 
- 3字节：表示运行MongoDB的机器 
- 2字节：表示生成此_id的进程 
- 3字节：由一个随机数开始的计数器生成的值

## Model-文档操作
### 查询
查询语句一般结构
Model.find({ 查询参数 }, { 过滤参数-field }, callback) // 也可以采用promise的方式调用
查询参数为忽略或者{}则返回所有集合文档

```javascript
Model.find({}, callback) // 如果查询参数忽略或者为{}则返回所有集合文档
Model.find({}, { _id: 0, name: 1, age: 0 }, callback) // 过滤参数查询结果包含name, 不包含 age跟_id字段 (_id默认是1)
Model.find({}, null, { limit:20 }) // 参数3: 游标操作 limit限制返回结果数量为20个，如不足二十个则返回所有
Model.findOne({}, callback) // 返回查询到的第一个文档
Model.findById('obj._id', callback) // 查询找到的第一个文档，但只接受_id的值查询
```

### 条件查询
$gt ——– greater than >
$gte ——— gt equal >=
$lt ——– less than <
$lte ——— lt equal <=
$ne ———– not equal !=
$eq ——– equal =

```javascript
Model.find({ age: { '$gt': 18, '$lt': 30 } }) // 查询age大于18小于30的文档集合
```

### 或查询 OR
'$in' 一个键对应多个值
'$nin' 同上取反, 一个键不对应指定值
'$or' 多个条件匹配,可以嵌套in 使用
'$not' 同上取反, 查询与特定模式不匹配的文档

```javascript
Model.find({ age: { '$in': [18, 20, 24] } }) // 查询age等于20或21或21或a的文档
Model.find({“$or” : [ {'age':18} , {'name':'xueyou'} ] }) // 查询 age等于18 或 name等于'xueyou' 的文档
```

### 类型查询
null 能匹配自身和不存在的值, 想要匹配键的值 为null, 就要通过 “exists”条件判定键值已经存在“exists” (表示是否存在的意思)
```javascript
Model.find({ age: { '$in': [null] ,'$exists': true } ) // 查询 age值为null的文档
Model.find({ name: { $exists: true }}) //查询所有存在name属性的文档
Model.find({ telephone: { $exists: false }}) //查询所有不存在telephone属性的文档
```

### 正则查询
MongoDb 使用 Prel兼容的正则表达式库来匹配正则表达式
```javascript
Model.find({ name : /joe/i } ) // 查询name为 joe 的文档, 并忽略大小写
```

### 查询数组
1.'$all' 匹配数组中多个元素
```javascript
Model.find({ array : [5, 10] } ) // 查询 匹配array数组中 既有5又有10的文档
```
2.'$size' 匹配数组长度
```javascript
Model.find({ array : { '$size' : 3 } } ) // 查询 匹配array数组长度为3 的文档
```
3.'$slice' 查询子集合返回
```javascript
Model.find({ array: { '$slice': 10} } ) // 查询 匹配array数组的前10个元素

Model.find({ array: { '$slice': [5,10] } } ) // 查询 匹配array数组的第5个到第10个元素

Model.find({ array: 10} ) // 查询 array(数组类型)键中有10的文档,  array : [1,2,3,4,5,10]  会匹配到

Model.find({ array[5]: 10} ) // 查询 array(数组类型)键中下标5对应的值是10,  array : [1,2,3,4,5,10]  会匹配到
```
4.$elemMatch数组元素命中查询
```javascript
Model.find({ array : { $elemMatch: { id: 1, flag: 0 } } } ) // 查询数组内同一条记录同时满足两个条件的查询
```

### $where
```javascript
Model.find({"$where" : function(){
  //这个函数中的 this 就是文档
  for( var x in this ){
  }
  if(this.x !== null && this.y !== null){
    return this.x + this.y === 10 ? true : false
  }else{
    return true
  }
}})
// 简化版本
Model.find( {'$where': 'this.x + this.y === 10' } )
Model.find( {'$where': 'function(){ return this.x + this.y ===10}' } )
```

### 修改器和更新器
一般情况下只需要使用Model.update({ 查询参数 }, { 更新之后的值 })
```javascript
Model.update({ userName: '1111' }, { userName: '2222' }) // 把userName等于1111的所有记录更新成2222 (updateOne 表示只更新查询到的第一个元素)
```
1.'$inc' 增减修改器,只对数字有效.下面的实例: 找到 age=22的文档,修改文档的age值自增1
```javascript
Model.update({ age: 22}, {'$inc': { age: 1 } }) // 执行后: age=23
```
2.'$set' 指定一个键的值,这个键不存在就创建它.可以是任何MondoDB支持的类型.
```javascript
Model.update({ age: 22 }, { '$set': { age: 'haha' } }) // 执行后: age='haha'
```
3.'$unset' 同上取反,删除一个键
```javascript
Model.update({ age: 22 }, { '$unset': { age: 'haha' } }) // 执行后: age键不存在
```

### 数组修改器
1.'$push' 给一个键push一个数组成员,键不存在会创建
```javascript
Model.update({ 'age':22 }, { '$push': { 'array': 10 } }) // 执行后: 增加一个array键,类型为数组, 有一个成员 10
```
2.'$addToSet' 向数组中添加一个元素,如果存在就不添加
```javascript
Model.update({ 'age': 22 }, { '$addToSet': { 'array': 10 } }) // 执行后: array中有10所以不会添加
```
3.'$each′遍历数组,和push 修改器配合可以插入多个值
```javascript
Model.update({ 'age': 22 }, { '$push': { 'array': { '$each': [1,2,3,4,5]}} }) // 执行后: array : [10,1,2,3,4,5]
```
4.'$pop' 向数组中尾部删除一个元素
```javascript
Model.update({ 'age': 22 }, { '$pop': { 'array': 1 } }) // 执行后: array : [10,1,2,3,4]  tips: 将1改成-1可以删除数组首部元素
```
5.'$pull' 向数组中删除指定元素
```javascript
Model.update({ 'age': 22 }, {'$pull': { 'array': 10 } }) // 执行后: array : [1,2,3,4]  匹配到array中的10后将其删除
```