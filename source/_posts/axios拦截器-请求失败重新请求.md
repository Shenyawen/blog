---
title: axios拦截器 + 请求失败重新请求
date: 2018-12-23 21:59:31
tags: axios
---
基于vue, 但是当然并没有什么强依赖关系，其他框架同样可以
### 引入依赖
```javascript
//request.js
import axios from "axios"
import qs from "qs" // 具体用途可以在npm官网查(序列化参数)
import app from "../main.js" // vue项目的启动文件
```

### axios默认配置
```javascript
/**
 * 区分开发/生产环境请求host
 * domain.js
*/
const env = process.env.DOMAIN_ENV === 'prod' ? 'prod' : 'dev'
export const domainConfig = {
  dev: {
    domain: 'http://***.***.***.***:4003'
  },
  prod: {
    domain: 'http://www.xxxx.com'
  }
}
function getDomin (domin) {
  return domainConfig[env][domin]
}
const DOMAIN = {
  domain: getDomin('domain')
}
export default DOMAIN
```
```javascript
//request.js
import DOMAIN from '@/utils/domain'
const service = axios.create({
  baseURL: DOMAIN.domain, // api 的 base_url
  timeout: 5000 // request timeout
})
```
### axios拦截器
```javascript
//request.js
// 对请求进行拦截
service.interceptors.request.use(
  config => {
    const token = getToken()
    if (token) {
      config.headers['Authorization'] = token // 我这就加了个token，还可以加入公参之类的信息(config带有完整的请求信息)
    }
    // app.$vux.loading.show() 显示loading图标
    return config
  },
  error => {
    console.log('error', error)
    Promise.reject('网络错误，请请重试') // eslint-disable-line
  }
)
// 对响应做处理
service.interceptors.response.use(
  response => {
    const { status, msg } = response.data
    if (status === 'error') {
      return Promise.reject(msg)
    } else {
      return response.data
    }
  },
  error => {
    console.log('err' + error) // for debug
    return Promise.reject('网络错误请重试！') // eslint-disable-line
  }
)
```
### 请求失败自动重新请求
改两个地方就行了
```javascript
// request.js
// 这两个默认配置也可以放在create函数参数里
axios.defaults.retry = 5
axios.defaults.retryDelay = 1000

service.interceptors.response.use(
  response => {
    const { status, msg } = response.data
    if (status === 'error') {
      return Promise.reject(msg)
    } else {
      return response.data
    }
  },
  err => {
    const config = err.config;
    if(!config || !config.retry) return Promise.reject(err)

    config.__retryCount = config.__retryCount || 0

    if(config.__retryCount >= config.retry) {
      return Promise.reject('网络错误，请重试')
    }

    config.__retryCount += 1

    const backoff = new Promise(function(resolve) {
      setTimeout(function() {
        resolve();
      }, config.retryDelay || 1)
    })

    return backoff.then(function() {
      return axios(config)
    })
  }
)
```
