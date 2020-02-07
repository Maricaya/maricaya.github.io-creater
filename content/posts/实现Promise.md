---
title: "实现Promise"
date: 2020-01-10T09:45:20+08:00
draft: true
---
手写Promise是前端面试中常考的一道题目，本章将和大家一起完成一个符合Promises/A+标准的Promise方法，希望对你有所帮助。

## Promise 简介
- 解决什么问题
  - 回调地狱
- 如何解决的 - how
- 什么优点（对比其他技术）
  - 减少缩进
  - 消灭if(err)
- 缺点
- 如何解决这些缺点

## Promises/A+标准
https://promisesaplus.com/

## Promise 的完整API是什么
### promise是一个类
- js中类是特殊的函数
- 类属性：length（可忽略）
- 类方法：all/...
- 对象属性: **then**/finally/catch
- 对象内部属性: **state** = pending/fulfilled/rejected

## 实现promise基本方法
##### promise是一个类，只接受函数fn，并立即执行该函数
```js
class Promise2 {
  constructor(fn) {
    if (typeof fn !== "function") {
      throw new Error("只接受函数")
    }
    fn()
  }
}
```
##### fn执行的时候接受resolve和reject两个函数
```js
class Promise2 {
  constructor(fn) {
    if (typeof fn !== "function") {
      throw new Error("只接受函数")
    }
    /*********************************/
    fn(() => {}, () => {})
    /*********************************/
  }
}
```
##### Promise有then方法
###### promise.then(succeed,fail) 中的succeed 会在resolve被调用的时候执行,fail会在reject被调用的时候执行
```js
class Promise2 {
  succeed = null
  fail = null
  constructor(fn) {
    if (typeof fn !== "function") {
      throw new Error("只接受函数")
    }
  /*********************************/
    fn(() => {
      // 如果不加setTimeout，此时this.succeed 还是 null
      setTimeout(() => {
        this.succeed()
      })
    }, () => {
      setTimeout(() => {
        this.fail()
      })
    })
  }
  then (succeed, fail) {
    this.succeed = succeed
    this.fail = fail
  }
  /*********************************/
}
```
##### 将resolve和reject方法提取出来
```js
class Promise2 {
    succeed = null
    fail = null
    constructor(fn) {
        if (typeof fn !== "function") {
            throw new Error("只接受函数")
        }
        /*
         * 在内部调用，this.resolve是函数
         * this.resolve() 相当于 undefined.this.resolve().call(undefined)
         * 
         */
        fn(this.resolve.bind(this), this.reject.bind(this))
    }
    resolve () {
        setTimeout(() => {
            this.succeed()
        })
    }
    reject () {
        setTimeout(() => {
            this.fail()
        })
    }
    then(succeed, fail) {
        this.succeed = succeed
        this.fail = fail
    }
}
```


### 开始根据规范写代码
##### 2.2.1 onFulfilled和onRejected都是可选的参数：(注：onFulfilled为succeed函数，onRejected为fail函数)
###### 2.2.1.1 如果 onFulfilled不是函数，必须忽略
###### 2.2.1.1 如果 onRejected不是函数，必须忽略
```js
class Promise2 {
  resolve () {
    setTimeout(() => {
      /********************************************/
      if (typeof this.succeed === 'function') {
        this.succeed()
      }
      /********************************************/
    })
  }
  reject () {
    setTimeout(() => {
      if (typeof this.fail === 'function') {
        this.fail()
      }
    })
  }
  then(succeed?, fail?) {
    this.succeed = succeed
    this.fail = fail
  }
}
```
##### 2.2.2 如果onFulfilled是函数:（2.2.3 onRejected同理）
###### 2.2.2.1 此函数必须在promise 完成(fulfilled)后被调用,并把promise 的值作为它的第一个参数
###### 2.2.2.2 此函数在promise完成(fulfilled)之前绝对不能被调用
###### 2.2.2.2 此函数绝对不能被调用超过一次
```js
class Promise2 {
  succeed = null
  fail = null
  /****************************************/
  // 加入三个状态
  state = 'pending'
  resolve (result) {
    setTimeout(() => {
      /* 2.2.2.2 此函数绝对不能被调用超过一次 */
      if (this.state !== 'pending') {return}
      this.state = 'fulfilled'
      if (typeof this.succeed === 'function') {
        // 2.2.2.1 promise 的值作为它的第一个参数
        this.succeed(result)
      }
      /****************************************/
    })
  }
  reject (reason) {
    setTimeout(() => {
      if (this.state !== 'pending') {return}
      this.state = 'rejected'
      if (typeof this.fail === 'function') {
        this.fail(reason)
      }
    })
  }
}
```
##### 2.2.4 在你的代码没有执行完毕之前，不得调用then后面的两个函数
目前的代码符合这条标准，因为then后的函数都是放在setTimeout中执行的。

##### 2.2.5  onFulfilled 和 onRejected 必须被当做函数调用（eg.不能有this传递过来）
call 就完了
```js
// ...
if (typeof this.succeed === 'function') {
  this.succeed.call(undefined, result)
}
// ...
```
##### 2.2.6 then可以在同一个promise里被多次调用
###### 2.2.6.1 如果/当 promise 完成执行（fulfilled）,各个相应的onFulfilled回调
必须根据最原始的then 顺序来调用
###### 2.2.6.2 如果/当 promise 被拒绝（rejected）,各个相应的onRejected回调
必须根据最原始的then 顺序来调用
```js
class Promise2 {
    state = 'pending'
    /****************************************/
    // 将函数放入数组中，保存下来
    callbacks = []
    /****************************************/
    constructor(fn) {
      if (typeof fn !== "function") {
        throw new Error("只接受函数")
      }
      fn(this.resolve.bind(this), this.reject.bind(this))
    }
    resolve (result) {
      setTimeout(() => {
        if (this.state !== 'pending') {return}
        this.state = 'fulfilled'
        /****************************************/
        // 依次调用
        this.callbacks.forEach(handle => {
          if (typeof handle[0] === 'function') {
            handle[0].call(undefined, result)
          }
        })
        /****************************************/
      })
    }
    reject (reason) {
      setTimeout(() => {
        if (this.state !== 'pending') {return}
        this.state = 'rejected'
        this.callbacks.forEach(handle => {
          if (typeof handle[1] === 'function') {
            handle[1].call(undefined, reason)
          }
        })
      })
    }
    then(succeed?, fail?) {
      /****************************************/
      this.callbacks.push([succeed, fail])
      /****************************************/
    }
}
```

2.2.7 then必须返回一个promise
```js
  promise2 = promise1.then(onFulfilled, onRejected);
```
2.2.7.1 如果onFulfilled或onRejected返回一个值x, 运行
Promise Resolution Procedure  [[Resolve]](promise2, x)
2.2.7.2 如果onFulfilled或onRejected抛出一个异常e,promise2
必须被拒绝（rejected）并把e当作原因
2.2.7.3 如果onFulfilled不是一个方法，并且promise1已经完成（fulfilled）,
promise2必须使用与promise1相同的值来完成（fulfiled）
2.2.7.4  如果onRejected不是一个方法，并且promise1已经被拒绝（rejected）,
promise2必须使用与promise1相同的原因来拒绝（rejected）
```js
class Promise2 {
    state = 'pending'
    callbacks = []

    constructor(fn) {
        if (typeof fn !== "function") {
            throw new Error("只接受函数")
        }
        fn(this.resolve.bind(this), this.reject.bind(this))
    }
    resolve (result) {
        process.nextTick(() => {
            if (this.state !== 'pending') {return}
            this.state = 'fulfilled'
            this.callbacks.forEach(handle => {
                if (typeof handle[0] === 'function') {
                  /**********************************************/
                    const x = handle[0].call(undefined, result)
                    handle[2].resolveWith(x)
                  /**********************************************/
                }
            })
        })
    }
    reject (reason) {
        process.nextTick(() => {
            if (this.state !== 'pending') {return}
            this.state = 'rejected'
            this.callbacks.forEach(handle => {
                if (typeof handle[1] === 'function') {
                  /**********************************************/
                    const x = handle[1].call(undefined, reason)
                    handle[2].resolveWith(x)
                  /**********************************************/
                }
            })
        })
    }
    then (succeed?, fail?) {
        const handle = []
        handle[0] = succeed
        handle[1] = fail
        handle[2] = new Promise2(() => {})
        this.callbacks.push(handle)
        return handle[2]
    }
    /**********************************************/
    resolveWith (x) {
        if (this === x) {
            return this.reject(new TypeError())
        }
        else if (x instanceof Promise2) {
            x.then((result) => {
                this.resolve(result)
            }, reason => {
                this.reject(reason)
            })
        }
        else if (x instanceof Object) {
            let then
            try {
                then = x.then
            } catch (e) {
                this.reject(e)
            }
            if (then instanceof Function) {
                try {
                    x.then(y => {
                        this.resolveWith(y)
                    }, r => {
                        this.reject(r)
                    })
                } catch (e) {
                    this.reject(e)
                }
            }else {
                this.resolve(x)
            }
        } else {
            this.resolve(x)
        }
    }
      /**********************************************/
}
export default Promise2
```
#### 重构
- 将 resolve 和 reject抽取为一个函数 resolveOrReject
- 从 resolveWith 中抽取三个方法resolveWithSelf、resolveWithPromise、resolveWithObject
```js
class Promise2 {
    state = 'pending'
    callbacks = []

    constructor(fn) {
        if (typeof fn !== "function") {
            throw new Error("只接受函数")
        }
        fn(this.resolve.bind(this), this.reject.bind(this))
    }
    resolve (result) {
        this.resolveOrReject('fulfilled', result, 0)
    }
    reject (reason) {
        this.resolveOrReject('rejected', reason, 1)
    }
    resolveOrReject (state, data, i) {
        process.nextTick(() => {
            if (this.state !== 'pending') {return}
            this.state = state
            this.callbacks.forEach(handle => {
                if (typeof handle[i] === 'function') {
                    let x = handle[i].call(undefined, data)
                    handle[2].resolveWith(x)
                }
            })
        })
    }
    then (succeed?, fail?) {
        const handle = []
        handle[0] = succeed
        handle[1] = fail
        handle[2] = new Promise2(() => {})
        this.callbacks.push(handle)
        return handle[2]
    }
    resolveWithSelf () {
        return this.reject(new TypeError())
    }
    resolveWithPromise (x) {
        x.then(result => {this.resolve(result)}, reason => {this.reject(reason)})
    }
    resolveWithObject (x) {
        let then
        try { then = x.then } catch (e) { this.reject(e) }
        if (then instanceof Function) {
            try { x.then(y => { this.resolveWith(y) }, r => { this.reject(r) }) }
            catch (e) { this.reject(e) }
        }
        else { this.resolve(x) }
    }
    resolveWith (x) {
        if (this === x) {
            this.resolveWithSelf();
        }
        else if (x instanceof Promise2) {
            this.resolveWithPromise(x);
        }
        else if (x instanceof Object) {
            this.resolveWithObject(x);
        }
        else { this.resolve(x) }
    }
}
```