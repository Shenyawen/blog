---
title: react redux项目集成Immutable不可变对象
date: 2018-12-19 15:21:58
tags: redux
---

## Immutable不可变对象库介绍
Immutable.js 是facebook的大神花了三年时间打造的一个不可变对象库。使用Immutable.js创建的对象都不能被更改。对其对象的任何修改或添加删除操作都会返回一个新的Immutable对象。Immutable实现的原理是持久化数据结构，即在使用旧数据创建新数据时，要保证旧数据同时可用且不变，同时为了避免deepCopy把所有的节点都复制一遍带来的性能损耗，Immutable 使用了结构共享，即如果对象树种的一个节点发生了变化，只修改这个节点和受它影响的父节点，其他节点共享。

## 中文文档
https://yq.aliyun.com/articles/69516#83
https://zhuanlan.zhihu.com/p/20295971?spm=a2c4e.11153940.blogcont69516.18.4f275a00eririY&columnSlug=purerender

## 与redux集成
主要有两点
1.将redux的combineReducers替换为redux-immutable的combineReducers
2.如果使用react-router-redux，因为router-reducer只支持普通对象，需要自定义router-reducer

```javascript
// 替换combineReducers
import { combineReducers } from 'redux-immutable' // eslint-disable-line
import { fromJS } from 'immutable'
// import { combineReducers } from 'redux'
import { LOCATION_CHANGE } from 'react-router-redux'
import homeReducer from '../features/home/redux/reducer'
import commonReducer from '../features/common/redux/reducer'
// 自定义routerReducer
function routerReducer(state = fromJS({ location: null }), action) {
  const type = action.type
  const payload = action.payload
  if (type === LOCATION_CHANGE) {
    return state.merge({ location: payload })
  }
  return state
}

const reducerMap = {
  router: routerReducer,
  home: homeReducer,
  common: commonReducer
}

export default combineReducers(reducerMap)
```
在这里安利一个react项目构建工具rekit, 集成了redux react-router-redux, 自带命令行工具，可以用命令行创建 component/action/feature...
官方文档: http://rekit.js.org/