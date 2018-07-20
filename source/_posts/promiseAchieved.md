---
title: Promise 简单实现
date: 2018-07-19 11:42:24
categories: [前端]
tags: [promise]
---
<!-- toc -->
今天的任务是根据`Promise A+` 规范简单实现一个 `Promise` 库。开始之前，我们可以先通读一下这个规范，[戳这里](https://promisesaplus.com/)。

静心读一下这个规范真的很重要，很重要，很重要。虽然是英文版，但是读起来也许会比汉语更能让人理解。

`A promise represents the eventual result of an asynchronous operation. The primary way of interacting with a promise is through its then method, which registers callbacks to receive either a promise’s eventual value or the reason why the promise cannot be fulfilled.`

提到 `Promise` 自然想到异步，异步想到回调，回调就是回头在调。回头在调的前提是提前存储回调内容，等到时间后从存储的地方取到对应的内容再执行即可。

## 一、Promise 构造函数实现
`2.1 A promise must be in one of three states: pending, fulfilled, or rejected. `

`2.1.1 A pending promise may transition to either the fulfilled or rejected state.`

`2.1.2 A fulfilled promise must have a value, which must not change. `

`2.1.3 A rejected promise must have a reason, which must not change.`

我们平时但凡做什么事情，大概都有三种基本状态，正在进行、成功、失败。一个 `Promise` 代表一个异步操作，也有这三种状态，且状态只能从正在进行到成功或者失败，不可逆转。不管事情最终什么状态，既然结果出来了，那就通知之前缓存回调的数组（因为可以 `then` 多次，故为数组）并把结果给它吧。

`Promise` 基本使用：
```javascript
let promise = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('success')
    }, 1000)
})
promise.then((data) => {console.log(data)})
```
首先 `Promise` 是一个类，通过 `new` 创建一个实例，参数是一个函数，函数参数默认命名为 `resolve` 和 `reject`，我们称该函数为一个执行器 `executor`，默认执行。构造函中还需要一个状态变量 `status`，保存当前执行状态；以及执行成功 `value` 或失败 `reason` 的结果变量；最后还需要上面提及的缓存回调的数组。实现代码如下：
```javascript
class Promise () {
    constructor (executor) {
        // 默认等待态
        this.status = 'pending'
        // 成功结果
        this.value = undefined
        // 失败原因
        this.reason = undefined
        // 存储成功回调数组
        this.onResolvedCallbacks = []
        // 存储失败回调数组
        this.onRejectedCallbacks = []
        // 执行器
        executor(resolve, reject)
    }
}
```
执行器函数参数是 `resolve` 和 `reject`， 同样也是函数，功能是有结果后触发缓存回调数组，也可以理解为发布。实现代码如下：
```javascript
class Promise () {
    constructor (executor) {
        // 变量声明
        this.status = 'pending'
        this.value = undefined
        this.reason = undefined
        this.onResolvedCallbacks = []
        this.onRejectedCallbacks = []
        // 结果成功，通知函数定义
        let resolve = (value) => {
            if (this.status === 'pending') {
                this.status = 'resolved'
                this.value = value
                this.onFulfilledCallbacks.forEach(fn => fn())
            }
        }
        // 结果失败，通知函数定义
        let reject = (reason) => {
            if (this.status === 'pending') {
                this.status = 'rejected'
                this.reason = reason
                this.onRejectedCallbacks.forEach(fn => fn())
            }
        }
        try {
            executor(resolve, reject)
        } catch (e) {
            reject(e)
        }
    }
}
```
既然有结果后要触发缓存回调数组，那么接下来我们看一下这个回调数组如何缓存。

## 二、Promise.then 实现
`2.2 A promise must provide a then method to access its current or eventual value or reason.`
### 1、then 订阅功能
我们都知道，`promise` 实例只有通过调用 `then` 函数才能拿到结果，`then` 函数参数分别为成功回调和失败回调，回调参数为成功值和失败值。固 `then` 函数的主要功能即是缓存回调函数，也可以理解为订阅。实现代码如下：

```javascript
class Promise () {
    // constructor省略...
    then (onFulfilled, onRejected) {
        // 如果exector内容为同步代码，resolve函数执行后，状态已变为resolved，则直接执行回调
        if (this.status === 'resolved') {
            onFulfilled(this.value)
        } 
        // 如果exector内容为同步代码，reject函数执行后，状态已变为rejected，直接执行回调
        if (this.status === 'rejected') {
           onRejected(this.reason)
        }
        // 如果exector内容为异步代码，状态为pending时，则缓存回调函数，我们也可以理解为订阅
        if (this.status === 'pending') {
            this.onResolvedCallbacks.push(() => {
               onFulfilled(this.value)
            })
            this.onRejectedCallbacks.push(() => {
                onRejected(this.reason)
            })
        }
    }
}
```
### 2、then 返回 promise
`2.2.6 Then may be called multiple times on the same promise. `

`2.2.7 Then must return a promise. promise2 = promise1.then(onFulfilled, onRejected)`

实现代码如下：
```javascript
class Promise () {
    // constructor省略...
    then (onFulfilled, onRejected) {
        let promise2
        if (this.status === 'resolved') {
            promise2 = new Promise((resolve, reject) => {
                onFulfilled(this.value)
            })
        }
        if (this.status === 'rejected') {
            promise2 = new Promise((resolve, reject) => {
                onRejected(this.reason)
            })
        }
        if (this.status === 'pending') {
            promise2 = new Promise((resolve, reject) => {
                this.onResolvedCallbacks.push(() => {
                    onFulfilled(this.value)
                })
                this.onRejectedCallbacks.push(() => {
                    onRejected(this.reason)
                })
            })
        }
        return promise2
    }
}
```
### 2、then 调用多次
`2.2.7.1 onFulfilled or onRejected returns a value x, run the Promise Resolution Procedure [[Resolve]](promise2, x).`

`then` 可以调用多次，且每次 `then` 参数 `onFulFilled` 函数的参数值为上一次 `promise` 函数的执行结果，即为 `onFulfilled(this.value)` 的执行结果，我们设为 `x`。根据 `x` 的不同情况做不同处理，详情见 `Promise A+` 规范 `2.3 Promise Resolution Procedure`。

实现代码如下：
```javascript
class Promise () {
    // constructor省略...
    then (onFulfilled, onRejected) {
        let promise2
        if (this.status === 'resolved') {
            promise2 = new Promise((resolve, reject) => {
                let x = onFulfilled(this.value)
                // Promise Resolution Procedure
                resolvePromise(promise2, x, resolve. reject)
            })
        }
        if (this.status === 'rejected') {
            promise2 = new Promise((resolve, reject) => {
                let x = onRejected(this.reason)
                // Promise Resolution Procedure
                resolvePromise(promise2, x, resolve. reject)
            })
        }
        if (this.status === 'pending') {
            promise2 = new Promise((resolve, reject) => {
                this.onResolvedCallbacks.push(() => {
                    let x = onFulfilled(this.value)
                    // Promise Resolution Procedure
                    resolvePromise(promise2, x, resolve. reject)
                })
                this.onRejectedCallbacks.push(() => {
                    let x = onRejected(this.reason)
                    resolvePromise(promise2, x, resolve, reject)
                })
            })
        }
        return promise2
    }
}
// 处理函数：处理promise2和x的关系
function resolvePromise (promise2, x, resolve, reject) {
    // 2.3.1 If promise and x refer to the same object, reject promise with a TypeError as the reason.
    if (promise2 === x) {
        return reject(new TypeError('循环引用'))
    }
    //  2.3.3 if x is an object or function
    if (x !== null && typeof x === 'object' || typeof x === 'function' ){
        try {
            // 2.3.3.1 Let then be x.then.
            let then = x.then
            // 2.3.3.3 If then is a function，call it with x as this.
            if (typeof then === 'function') {
                // 2.3.3.3.1 If/when resolvePromise is called with a value y, run [[Resolve]](promise, y)
                then.call(x, y => {
                    resolvePromise(promise2, y, resolve, reject)
                }, r => {
                    // 2.3.3.3.2 If/when rejectPromise is called with a reason r, reject promise with r.
                    reject(r)
                })
            }
        } catch (e) {
            // 2.3.3.4 If calling then throws an exception e
            reject(e)
        }
    } else {
        // 2.3.4 If x is not an object or function, fulfill promise with x.
        resolve(x)
    }
}
```

### 3、状态不可逆转
实现代码如下：
```javascript
// 处理函数：处理promise2和x的关系
function resolvePromise (promise2, x, resolve, reject) {
    // 2.3.1 If promise and x refer to the same object, reject promise with a TypeError as the reason.
    if (promise2 === x) {
        return reject(new TypeError('循环引用'))
    }
    let called
    //  2.3.3 if x is an object or function
    if (x !== null && typeof x === 'object' || typeof x === 'function' ){
        try {
            // 2.3.3.1 Let then be x.then.
            let then = x.then
            // 2.3.3.3 If then is a function，call it with x as this.
            if (typeof then === 'function') {
                // 2.3.3.3.1 If/when resolvePromise is called with a value y, run [[Resolve]](promise, y)
                then.call(x, y => {
                    // `2.3.3.3.3 If both resolvePromise and rejectPromise are called, or multiple calls to the same argument are made, the first call takes precedence, and any further calls are ignored.`
                    if (called) return
                    called = true
                    resolvePromise(promise2, y, resolve, reject)
                }, r => {
                    if (called) return
                    called = true
                    // 2.3.3.3.2 If/when rejectPromise is called with a reason r, reject promise with r.
                    reject(r)
                })
            }
        } catch (e) {
            // `2.3.3.3.4 If calling then throws an exception e，2.3.3.3.4.1 If resolvePromise or rejectPromise have been called, ignore it.`
            if (called) return
            called = true
            // 2.3.3.4 If calling then throws an exception e
            reject(e)
        }
    } else {
        // 2.3.4 If x is not an object or function, fulfill promise with x.
        resolve(x)
    }
}
```
### 4、Promise.then 异步执行
实现代码如下：
```javascript
class Promise () {
    then (onFulfilled, onRejected) {
        let promise2
        if (this.status === 'resolved') {
            promise2 = new Promise((resolve, reject) => {
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value)
                        resolvePromise(promise2, x, resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                }, 0)
            })
        }
        if (this.status === 'rejected') {
            promise2 = new Promise((resolve, reject) => {
                setTimeout(() => {
                    try {
                        let x = onRejected(this.reason)
                        resolvePromise(promise2, x, resolve, reject)
                    } catch (e) {
                        reject(e)
                    }
                }. 0)
            })
        }
        if (this.status === 'pending') {
            promise2 = new Promise((resolve, reject) => {
                this.onResolvedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value)
                            promiseResolve(promise2, x, resolve, reject)
                        } catch (e) {
                            reject(e)
                        }
                    }, 0)
                })
                this.onRejectedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onRejected(this.reason)
                            promiseResolve(promise2, x, resolve, reject)
                        } catch (e) {
                            reject(e)
                        }
                    }, 0)
                })
            })
        }
        return promise2
    }
}
```
### 5、Promise.then 穿透
`promise.then().then()` 多次，都可以获取到最终结果，固当 `then` 参数 `onFulfilled` 为空时，需要自动 `return value`；当 `onRejected`  为空时，需要 `return throw err`。实现代码如下：
```javascript
class Promise () {
    then (onFulfilled, onRejected) {
        onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : (value) => { return value }
        onRejected = typeof onRejected === 'function' ? onRejected : (err) => { throw err }
        // promise2省略...
    }
}
```
## 三、其它方法实现
### 1、Promise.resolve 
实现代码如下：
```javascript
Promise.resolve = function (val) {
    return new Promise((resolve, reject) => {
        resolve(val)
    })
}
```
### 2、Promise.reject 
实现代码如下：
```javascript
Promise.reject = function (val) {
    return new Promise((resolve, reject) => {
        reject(val)
    })
}
```
### 3、Promise.all 
实现代码如下：
```javascript
Promise.all = function (promises) {
    return new Promise((resolve, reject) => {
        let dataArr = []
        let count = 0
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(data => {
                dataArr[i] = data
                count++
                if (count === promises.lenth) {
                    resolve(dataArr)
                }
            }, reject)
        }
    })
}
```
### 4、Promise.race 
实现代码如下：
```javascript
Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            promise[i].then(resolve, reject)
        }
    })
}
```
### 5、Promise.promisify 
实现代码如下：
```javascript
Promise.promisify = function (fn) {
    return function (...args) {
        return new Promise((resolve, reject) => {
            fn(...args, (err, data) => {
                if (err) reject(err)
                resolve(data)
            })
        })
    }
}
```
## 四、Promise 校验
要想验证自己实现的 `Promise` 是否符合 `Promise A+` 规范，可以全局安装 `promises-aplus-tests` 工具包。

`Adapters`：

`In order to test your promise library, you must expose a very minimal adapter interface. These are written as Node.js modules with a few well-known exports:`

- `resolved(value): creates a promise that is resolved with value.`
- `rejected(reason): creates a promise that is already rejected with reason.`
- `deferred(): creates an object consisting of { promise, resolve, reject }`

> `The resolved and rejected exports are actually optional, and will be automatically created by the test runner using deferred`

`resolved` 和 `rejected` 是可选的，下面我们实现一下 `deferred`，实现代码如下：
```javascript
Promise.deferred = function () {
    let dfd = {}
    dfd.promise = new Promise((resolve, reject) => {
        dfd.resolve = resolve
        dfd.reject = reject
    })
    return dfd
}
```
到这里一切准备就绪，校验步骤如下：
```
npm install promises-aplus-tests -g
promises-aplus-tests ./Promise.js
```
<br/>

[源码](https://github.com/zdddrszj/promise)

好了，到这里全部代码都已呈现，赶快自己亲手试一下吧！😋😋😋