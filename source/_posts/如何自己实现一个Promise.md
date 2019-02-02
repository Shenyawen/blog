---
title: 如何自己实现一个Promise
date: 2019-02-02 17:40:05
tags: javascript
---
Promise在项目中会经常用用到，特别是涉及一些异步操作的时候，用了那么长时间，趁着放假特地花点时间去看看如何去自己实现一个Promise库

## Promise的使用方法
首先我先简单说下Promise的使用方法
```javascript
// 最普通的promise使用方法
new Promise((resolve, reject) => {
  // 执行函数中可以管控异步操作也可以不管控，但是不管控的话Promise就没有意义了
  Math.random() > 0.5 ? resolve(100) : reject(200) 
}).then(result => console.log(result))
  .catch((reason) => console.log('error', reason))

const p1 = new Promise(...)
const p2 = new Promose(...)
...
Promise.all([p1, p2...])
  .then(reselt => {...})
  .catch(reason => {...})
// Promise.all 会返回一个Promise实例，只有所有的异步操作成功了才会进入then回调函数，并将所有promise实例结果的数组给返回出来，只要有一个失败就会进入catch回调函数
// catch函数实际上就相当于then(null, reason => {})
```

## 实现romise类
### 第一步先实现一个简单的Promise类，不包含异常及特殊情况处理

```javascript
class Promise {
  constructor (exeCallback) {
    let _status = 'pending'
    this._value = null
    this._resolveCbs = []
    this._rejectCbs = []
    const resolveFn = (result) => {
      if (_status !== 'pending') return
      _status = 'fulfilled'
      this._value = result
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._resolveCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    const rejectFn = (reason) => {
      if (_status !== 'pending') return
      _status = 'rejected'
      this._value = reason
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._rejectCbs.forEach(cd => {
           cd(this._value)
        })
      })
    }
    exeCallback(resolveFn, rejectFn)
  }

  // then是Promise实例的方法
  // 第一个参数是成功之后的回调，第二个参数是失败之后的回调, 两者都可以是null或者不传
  then (resolveCb, rejectCb) {
    this._resolveCbs.push(resolveCb)
    this._rejectCbs.push(rejectCb)
  }
}

// 尝试运行一下
new Promise((resolve, reject) => {
  Math.random() > 0.1 ? resolve(100) : reject(200) 
})
  .then(result => {
    console.log(result)
  }, reason => {
    console.log(reason)
  })
```
上面这段代码是可以跑的，但是有个问题，如果then的第二个参数没有传，会报一个错, cd not a function, 接下来我们需要做的异常处理
还是会基于上面的代码

```javascript
class Promise {
  constructor (exeCallback) {
    let _status = 'pending'
    this._value = null
    this._resolveCbs = []
    this._rejectCbs = []
    const resolveFn = (result) => {
      if (_status !== 'pending') return
      _status = 'fulfilled'
      this._value = result
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._resolveCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    const rejectFn = (reason) => {
      if (_status !== 'pending') return
      _status = 'rejected'
      this._value = reason
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._rejectCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    try {
      exeCallback(resolveFn, rejectFn) // 因为执行函数可能会存在执行错误，需要捕获错误并直接跳转到rejectFn函数中
    } catch (error) {
      rejectFn(error)
    }
  }

  // then是Promise实例的方法
  // 第一个参数是成功之后的回调，第二个参数是失败之后的回调, 两者都可以是null或者不传
  then (resolveCb, rejectCb) {
    // 首先判断回调函数是否是函数，不是的话就默认一个回调23
    resolveCb = typeof resolveCb !== 'function' ? result => result : resolveCb
    rejectCb = typeof rejectCb !== 'function' ? reason => {
      throw new Error(reason instanceof Error ? reason.message : reason)
    } : rejectCb
    // then函数执行之后会返回一个Promise函数
    return new Promise((resolve, reject) => {
      this._resolveCbs.push(() => {
        try {
          const x = resolveCb(this._value)
          x instanceof Promise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      })
      this._rejectCbs.push(() => {
        try {
          const x = rejectCb(this._value)
          x instanceof Promise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      })
    })
  }

  // catch之前说过就是相当于then在不传resolveCb函数的then函数
  catch (rejectCb) {
    return this.then(null, rejectCb)
  }
}

// 尝试运行一下
new Promise((resolve, reject) => {
  Math.random() > 0.1 ? resolve(100) : reject(200) 
})
  .then(result => {
    console.log(result)
  }, reason => {
    console.log(reason)
  })
```

*** 这里面有一点我觉得需要你们把握一下，就是我们实际上并不关注结果，只是希望在有结果的时候then传递的回调函数可以执行就行 ***

上面的写法有一个问题，就是如果刚开始你只是实例化一个Promise对象，并没有立即执行then方法，当该promise对象已经有结果了，你再执行then函数是没有结果的，原因是在你把回调函数加入到数组之前, resolve已经执行了。

下面我在完善一下写法，增加对状态的判断
```javascript
class Promise {
  constructor (exeCallback) {
    this._status = 'pending'
    this._value = null
    this._resolveCbs = []
    this._rejectCbs = []
    const resolveFn = (result) => {
      if (this._status !== 'pending') return
      this._status = 'fulfilled'
      this._value = result
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._resolveCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    const rejectFn = (reason) => {
      if (this._status !== 'pending') return
      this._status = 'rejected'
      this._value = reason
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._rejectCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    try {
      exeCallback(resolveFn, rejectFn) // 因为执行函数可能会存在执行错误，需要捕获错误并直接跳转到rejectFn函数中
    } catch (error) {
      rejectFn(error)
    }
  }

  // then是Promise实例的方法
  // 第一个参数是成功之后的回调，第二个参数是失败之后的回调, 两者都可以是null或者不传
  then (resolveCb, rejectCb) {
    // 首先判断回调函数是否是函数，不是的话就默认一个回调23
    resolveCb = typeof resolveCb !== 'function' ? result => result : resolveCb
    rejectCb = typeof rejectCb !== 'function' ? reason => {
      throw new Error(reason instanceof Error ? reason.message : reason)
    } : rejectCb
    // then函数执行之后会返回一个Promise函数
    return new Promise((resolve, reject) => {
      
      if (this._status === 'fulfilled') {
        // 成功态
        const timer = setTimeout(() => {
          clearTimeout(timer)
          const x = resolveCb(this._value)
          x instanceof Promise ? x.then(resolve, reject) : resolve(x)
        })
      } else if (this._status === 'rejected') {
        // 失败态
        const timer = setTimeout(() => {
          clearTimeout(timer)
          const x = rejectCb(this._value)
          x instanceof Promise ? x.then(resolve, reject) : resolve(x)
        })
      } else {
        // 等待状态
        this._resolveCbs.push(() => {
          try {
            const x = resolveCb(this._value)
            x instanceof Promise ? x.then(resolve, reject) : resolve(x)
          } catch (error) {
            reject(error)
          }
        })
        this._rejectCbs.push(() => {
          try {
            const x = rejectCb(this._value)
            x instanceof Promise ? x.then(resolve, reject) : resolve(x)
          } catch (error) {
            reject(error)
          }
        })
      }
    })
  }

  // catch之前说过就是相当于then在不传resolveCb函数的then函数
  catch (rejectCb) {
    return this.then(null, rejectCb)
  }
}
```
下面说一下Promise.all的是如何实现的
```javascript
class Promise {
  constructor (exeCallback) {
    this._status = 'pending'
    this._value = null
    this._resolveCbs = []
    this._rejectCbs = []
    const resolveFn = (result) => {
      if (this._status !== 'pending') return
      this._status = 'fulfilled'
      this._value = result
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._resolveCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    const rejectFn = (reason) => {
      if (this._status !== 'pending') return
      this._status = 'rejected'
      this._value = reason
      const timer = setTimeout(() => {
        clearTimeout(timer)
        this._rejectCbs.forEach(cd => {
          cd(this._value)
        })
      })
    }
    try {
      exeCallback(resolveFn, rejectFn) // 因为执行函数可能会存在执行错误，需要捕获错误并直接跳转到rejectFn函数中
    } catch (error) {
      rejectFn(error)
    }
  }

  // then是Promise实例的方法
  // 第一个参数是成功之后的回调，第二个参数是失败之后的回调, 两者都可以是null或者不传
  then (resolveCb, rejectCb) {
    // 首先判断回调函数是否是函数，不是的话就默认一个回调23
    resolveCb = typeof resolveCb !== 'function' ? result => result : resolveCb
    rejectCb = typeof rejectCb !== 'function' ? reason => {
      throw new Error(reason instanceof Error ? reason.message : reason)
    } : rejectCb
    // then函数执行之后会返回一个Promise函数
    return new Promise((resolve, reject) => {
      
      if (this._status === 'fulfilled') {
        // 成功态
        const timer = setTimeout(() => {
          clearTimeout(timer)
          const x = resolveCb(this._value)
          x instanceof Promise ? x.then(resolve, reject) : resolve(x)
        })
      } else if (this._status === 'rejected') {
        // 失败态
        const timer = setTimeout(() => {
          clearTimeout(timer)
          const x = rejectCb(this._value)
          x instanceof Promise ? x.then(resolve, reject) : resolve(x)
        })
      } else {
        // 等待状态
        this._resolveCbs.push(() => {
          try {
            const x = resolveCb(this._value)
            x instanceof Promise ? x.then(resolve, reject) : resolve(x)
          } catch (error) {
            reject(error)
          }
        })
        this._rejectCbs.push(() => {
          try {
            const x = rejectCb(this._value)
            x instanceof Promise ? x.then(resolve, reject) : resolve(x)
          } catch (error) {
            reject(error)
          }
        })
      }
    })
  }

  // catch之前说过就是相当于then在不传resolveCb函数的then函数
  catch (rejectCb) {
    return this.then(null, rejectCb)
  }

  static all (promiseAry) {
    return new promise((resolve, reject) => {
      let index = 0
      let result = []
      for (let i = 0; i < promiseAry.length; i++) {
        promiseAry[i].then(val => {
          index++
          result[i] = val
          if (index === promiseAry.length) {
            resolve(result)
          }
        }, reject)
      }
    })
  }
}
```

这是Promise的一个大概实现思路，当然还有更好的实现方式[Promise详解与实现（Promise/A+规范）](https://www.jianshu.com/p/459a856c476f)
有点绕，但是多看几遍应该是能看懂的
