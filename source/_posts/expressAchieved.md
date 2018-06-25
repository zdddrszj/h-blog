---
title: Express简单实现
date: 2018-06-22 17:11:24
categories: [前端]
tags: [express]
---
<!-- toc -->

## 一、实现express启动服务
- 原生 `express` 启动服务
```javascript
let express = require('express')
let app = express()
let server = app.listen(3000, 'localhost', function () {
	console.log(`app is listening at http://${server.address().address}:${server.address().port}`)
})
```
- 若实现如上功能，首先我们确定的是 `express` 是一个函数，执行后返回一个 `app` 函数，且有 `listen` 方法，我们不如称这个 `app` 函数为监听函数。实现代码如下：
```javascript
let http = require('http')
function createApplication () {
  // app是一个监听函数
  let app = function (req, res) {
    res.end('hello world')
  }
  app.listen = function () {
    let server = http.createServer(app)
    // arguments就是参数(3000, 'localhost', function () {})
    server.listen(...arguments)
    return server
  }
  return app
}
module.exports = createApplication
```

## 二、实现express路由
- 原生 `express` 使用路由
```javascript
let express = require('express')
let app = express()
app.get('/name', function (req, res) {
  res.end('get name')
})
app.listen(3000, 'localhost', function () {
  console.log('app is listening')
})
```
- 若实现如上功能，我们的 `app` 监听函数需要实现一个 `get` 方法，该方法可以把本次调用存储在 `app` 的路由数组中，当服务启动成功后，监听到匹配的路由时即可调用对应的回调。实现代码如下：
```javascript
let http = require('http')
function createApplication () {
  // app是一个监听函数
  let app = function (req, res) {
    // 当前请求方法
    let m = req.method.toLocaleLowerCase()
    // 当前请求路径
    let { pathname } = url.parse(req.url, true)
    for (let i = 0; i < app.routes.length; i++) {
      let { path, method, handler } = app.routes[i]
      // 如果该路由项匹配到当前请求，则执行回调
      if (path === pathname && method === m) {
        handler(req, res)
      }
    }
  }
  // 存储所有的请求，以便监听函数调用
  app.routes = []
  app.get = function (path, handler) {
    let layer = {
      path,
      method: 'get',
      handler
    }
    app.routes.push(layer)
  }
  // app.listen = function () {}
  return app
}
module.exports = createApplication
```
- 其他 `RESTFUL` 方法同理。实现代码如下：
```javascript
let http = require('http')
function createApplication () {
  // app是一个监听函数
  let app = function (req, res) {
    // 当前请求方法
    let m = req.method.toLocaleLowerCase()
    // 当前请求路径
    let { pathname } = url.parse(req.url, true)
    for (let i = 0; i < app.routes.length; i++) {
      let { path, method, handler } = app.routes[i]
      // 如果该路由项匹配到当前请求，则执行回调
      if ((path === pathname || path === '*') && (method === m || method === 'all')) {
        handler(req, res)
      }
    }
  }
  // 存储所有的请求，以便监听函数调用
  app.routes = []
  // http.METHODS 获取RESTFUL所有方法
  http.METHODS.forEach(method => {
    method = method.toLocaleLowerCase()
    app[method] = function (path, handler) {
      let layer = {
        method,
        path,
        handler
      }
      app.routes.push(layer)
    }
  })
  // 如果没有匹配成功，最终执行all函数所存储的回调
  app.all = function (path, handler) {
    let layer = {
      method: 'all', // 表示全部匹配
      path,
      handler
    }
    app.routes.push(layer)
  }
  // app.listen = function () {}
  return app
}
module.exports = createApplication
```

## 三、实现express中间件
- 原生 `express` 使用中间件
```javascript
let express = require('express')
let app = express()
app.use(function (req, res, next) {
  res.setHeader('Content-Type', 'text/html; charset=utf-8')
  next()
})
app.use('/name', function (req, res, next) {
  next()
})
app.get('/name', function (req, res) {
  res.end('获取姓名')
})
app.listen(3000, 'localhost', function () {
  console.log('app is listening')
})
```
- 由此可见，`use` 方法和 `method` 方法大同小异，重点是实现 `next` 方法。
`next` 函数的作用即是在请求到达前更改一些上下文环境，比如修改返回字符集编码等，且按顺序执行，固可用迭代的方式实现。实现代码如下：
```javascript
let http = require('http')
let url = require('url')
function createApplication () {
  // app是一个监听函数
  let app = function (req, res) {
    // 当前请求方法
    let m = req.method.toLocaleLowerCase()
    // 当前请求路径
    let { pathname } = url.parse(req.url, true)
    // 迭代次数索引
    let index = 0
    // 用next代替for循环
    function next () {
      // 如果全部路由数组都不满足，则返回找不到
      if (index === app.routes.length) {
        return res.end(`can not ${m} ${pathname}`)
      }
      // 处理中间件
      let { method, path, handler } = app.routes[index++]
      if (method === 'middle') {
        // 如果该中间件path是/，匹配全部请求，执行回调；如果相等，执行回调；如果该中间件path被包含在当前请求url中，也执行回调
        if (path === '/' || path === pathname || pathname.startsWith(path + '/')) {
          handler(req, res, next)
        } else {
          next()
        }
      } else {
        if ((path === pathname || path === '*') && (method === m || method === 'all')) {
          handler(req, res, next)
        } else {
          next()
        }
      }
    }
    // 直接调用next函数，根据路径匹配查找对应回调并执行
    next()
  }
  // app.routes = []
  // http.METHODS.forEach(() => {}
  // app.all = function (path, handler) {}
  // 中间件：参数可以传path，也可以不传，默认'/'
  app.use = function (path, handler) {
    if (typeof handler !== 'function') {
      handler = path
      path = '/'
    }
    let layer = {
      method: 'middle',
      path,
      handler
    }
    app.routes.push(layer)
  }
  // app.listen = function () {}
  return app
}
module.exports = createApplication
```
- 此时，`express` 的主要功能已经实现，下面来看下如果执行错误通过 `next` 函数参数进行返回的情况。
如果 `next` 函数有参数，会跳过接下来的所有中间件和路由，直接返回错误参数消息，所以在处理中间件之前要先判断错误情况，并且将错误继续向下传递，只有匹配到有四个参数的回调时才执行。实现代码如下：
```javascript
let http = require('http')
function createApplication () {
  // app是一个监听函数
  let app = function (req, res) {
    // 此处省略...
    function next (err) {
      // 此处省略...
      if (err) {
        res.end(err)
        if (handler.length === 4) {
          handler(err, req, res, next)
        } else {
          next(err)
        }
      } else {
        // 处理中间件，此处省略...
      }
    }
    next()
  }
  // app.routes = []
  // http.METHODS.forEach(() => {}
  // app.all = function (path, handler) {}
  // app.use = function (path, handler) {}
  // app.listen = function () {}
  return app
}
module.exports = createApplication
```

[源码](https://github.com/zdddrszj/code/blob/master/express/index.js)

好了，到这里全部代码已经给出，就到此为止吧~ 😋😋😋
