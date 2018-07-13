---
title: 实现 CommonJs 规范中的 Require 模块
date: 2018-06-25 13:22:26
categories: [前端]
tags: [require]
---

<!-- toc -->

开始之前大家要先熟悉下 `node` 中常用文件读写，路径操作等 `API`。
> 实现思路：我们通过定义一个 `req` 函数，代替 `node` 中的 `require` ，这个函数首先会根据路径参数进行路径解析，找到对应文件；然后判断缓存中是否存在该文件对象，存在则返回，否则创建该文件对象，此时不防通过 `new` 一个 `Module` 函数实例来创建；再然后，通过 `node` 中 `fs` 模块的 `readFileSync` 方法加载文件，并把文件内容放在闭包函数中，函数参数有`exports`、`require`、`module`，通过 `node` 中 `vm` 模块的 `runInThisContext` 方法创建一个沙箱环境，同时分别给 `exports`、`require`、`module` 参数赋值并执行，最后缓存模块并通过 `module.exports` 返回文件内容。
## 一、构造req函数
详细代码如下：
```javascript
function req (path) {
  // 先要根据路径变成一个绝对路径
  let filename = Module._resolveFilename(path)
  // 文件路径唯一
  if (Module._cache[filename]) {
    // 如果加载过 直接把加载过的结果返回
    return Module._cache[filename].exports
  }
  // 通过文件名创建一个模块
  let module = new Module(filename)
  // 加载模块，根据不同后缀加载不同内容
  module.load()
  // 进行模块缓存
  Module._cache[filename] = module
  // 返回最后的结果
  return module.exports
}
```
## 二、构造Module函数
详细代码如下：
```javascript
let path = require('path')
let fs = require('fs')

function Module(filename) {
  // 默认模块未加载过
  this.loaded = false
  // 模块的绝对路径
  this.filename = filename
  // 模块导出的结果
  this.exports = {}
}
```
路径解析方法如下：
```javascript
Module._resolveFilename = function (p) {
  p = path.join(__dirname, p)
  // 如果文件有后缀
  if (!/\.\w+$/.test(p)) {
    // 添加扩展名
    for (let i = 0; i < Module._extensions.length; i++) {
      // 拼出一个路径
      let filePath = p + Module._extensions[i]
      try {
        // 判断文件是否存在
        fs.accessSync(filePath)
        // 返回文件路径
        return filePath
      } catch (e) {
        // 如果accessSync方法报错，说明文件不存在，则抛出文件未找到异常
        if (i > Module._extensions.length) {
          throw new Error('module not found')
        }
      }
    }
  } else {
    // 否则直接返回文件路径
    return p
  }
}
```
`Module` 函数的主要功能即是 `load` 方法，如果目标文件是 `js` 文件，则按照 `js` 方式加载，如果目标文件是 `json` 文件，则按照 `json` 方式加载。详细代码如下：
```javascript
let vm = require('vm')

// 模块加载函数
Module.prototype.load = function () {
  // 取到文件名称后缀
  let extname = path.extname(this.filename)
  // 根据不同后缀读取文件内容
  Module._extensions[extname](this)
  // 标记模块已加载
  this.loaded = true
}

// 模块缓存对象
Module._cache = {}

// 文件后缀
Module._extensions = ['.js', '.json']

// json格式文件读取方式
Module._extensions['.json'] = function (module) {
  let content = fs.readFileSync(module.filename, 'utf8')
  module.exports = JSON.parse(content)
}

// 创建执行js沙箱环境时需要
Module.wrapper = ['(function (exports, require, module){', '\n})']

// js代码拼接
Module.wrap = function (content) {
  return Module.wrapper[0] + content + Module.wrapper[1]
}

// js格式文件读取方式
Module._extensions['.js'] = function (module) {
  // 读取文件
  let content = fs.readFileSync(module.filename, 'utf8')
  // 把文件内容放到闭包函数中
  let script = Module.wrap(content)
  // 创建不影响外界上下文的沙箱环境
  let fn = vm.runInThisContext(script)
  // 让闭包函数执行，同时给函数中export require module变量赋值
  fn.call(module.exports, module.exports, req, module)
}
```
好了，实现起来是不是很简单，下面测试一下：
```javascript
// 在同级目录下创建test.js
// module.exports = 'Hello World'
let str = req('./test')
console.log(str)
```

[源码](https://github.com/zdddrszj/code/blob/master/require/index.js)

好了，到这里全部代码已经给出，就到此为止吧~ 😋😋😋