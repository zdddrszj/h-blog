---
title: ReadableStream 简单实现
date: 2018-07-12 15:42:44
categories: [前端]
tags: [ReadableStream]
---
<!-- toc -->
今天的文章需要提前了解一下 `node` 中 `fs` 模块的相关 `api`，不太熟悉的同学可以[点这里](http://nodejs.cn/api/fs.html)。

> 众所周知，`node` 中的 `fs` 模块功能大都与文件相关，比如可以通过 `fs.createReadStream` 创建文件可读流，通过`fs.createWriteStream` 创建文件可写流，还可以通过监听 `open`、`data`、`end`、`error`、`readable` 事件对数据进行操作。由于时间有限，今天我们先来实现一下 `readable` 事件功能。

开始之前，先简单介绍一下可读流函数 `fs.createReadStream(path[, options])` 中各参数所代表的含义，如下所示：
- `path <string> | <Buffer> | <URL>` 创建可读流的路径
- `options <string> | <Object>` 可选参数
  - `flags <string>` 文件读写标识，默认为 r
  - `encoding <string>` 读取编码格式，默认为 null
  - `fd <integer>` 文件描述符，默认为 null
  - `mode <integer>` 文件操作权限，默认为 0o666
  - `autoClose <boolean>` 文件是否自动关闭，默认为 true
  - `start <integer>` 文件读取开始位置，默认为 0
  - `end <integer>` 文件读取结束位置，默认为 Infinity
  - `highWaterMark <integer>` 水位线，每次读取长度，默认为 64字节（64 * 1024）

## 一、创建可读流
首先我们需要实现一个可读流的类，不防定义为 `ReadableStream`，该类可以通过 `on` 函数进行事件监听，所以需要继承 `node` 中 `EventEmitter` 类；
当监听 `readable` 函数时可读取到文件内容，由此得知在构造函数中除了需要定义上面的变量，还需要调用打开文件和第一次读取文件的功能。代码如下：
```javascript
let fs = require('fs')
let EventEmitter = require('events')
class ReadableStream extends EventEmitter {
  constructor(path, options) {
    super()
    this.path = path
    this.flags = options.flags || 'r'
    this.encoding = options.encoding || null
    this.autoClose = options.autoClose || true
    this.highWaterMark = options.highWaterMark || 64 * 1024
    this.start = options.start || 0
    this.end = options.end || null
    this.mode = options.mode || 0o666

    // 是否正在读取文件
    this.reading = false
    // 当len=0时，触发readable事件
    this.emitReadable = false
    // 缓存中字节的长度
    this.len = 0
    // 缓存每次读取的内容，格式为[<Buffer />, <Buffer />, ...]
    this.arr = []
    // 文件读取的位置
    this.pos = this.start
    // 是否文件全部读取完
    this.finished = false
    // 打开文件
    this.open()
    // 判断用户是否监听了readable事件
    this.on('newListener', (type) => {
      if (type === 'readable') {
        // 第一次文件读取
        this.read()
      }
    })
  }
}
module.exports = ReadableStream
```
构造函数中其它变量可以先忽略，到实现阶段时我相信大家自然清晰其用处。
下面利用 `fs.open` 和 `fs.destory` 实现 `open` 和 `destory` 功能。实现代码如下：
```javascript
// 打开可读流
open() {
  fs.open(this.path, this.flags, (err, fd) => {
    if (err) {
      this.emit('error')
      if (this.autoClose) {
        this.destory()
      }
      return
    }
    this.fd = fd
    this.emit('open')
  })
}
// 关闭可读流，参数为文件描述符
destory() {
  if (typeof this.fd === 'number') {
    fs.close(this.fd, () => {
      this.emit('close')
    })
  }
  this.emit('close')
}
```
接下来看下初次读取时的 `read` 函数。
> 实现思路：在构造函数中，当触发第一次读取文件时，读取大小为 `highWaterMark` 个，不防我们将比较读取长度和缓存长度的方法设定为 `read`。然后再 `read` 函数中判断，如果缓存区长度为 `0` 时，表明可以触发 `readable` 事件；如果缓存区长度小于水位线时，则进行文件读取，此时我们将真正读取文件的函数命名为 `_read`；最后，根据编码格式进行返回数据。

实现代码如下：
```javascript
class ReadableStream extends EventEmitter {

  // 此处如上，省略...
  
  // 可读流实例调用的方法
  read () {
    let buffer = null

    // 如果缓存区长度为0时，表明可以触发readable事件
    if (this.len === 0) {
      this.emitReadable = true
    }

    // 如果缓存区长度小于水位线时，则进行文件读取
    if (this.len < this.highWaterMark) {
      if (!this.reading) {
        this.reading = true
        this._read()
      }
    }

    // 根据编码方式处理数据
    if (buffer) {
      buffer = this.encoding ? buffer.toString(this.encoding) : buffer
    }
    return buffer
  }
  // 真实读取文件的方法
  _read () {
    // 因为打开文件为异步操作，当读取时文件未打开，可以通过注册一次open事件，打开后执行回调即可拿到this.fd
    if (typeof this.fd !== 'number') {
      this.once('open', () => this._read())
      return
    }

    let howMuchToRead = this.end ? Math.min(this.highWaterMark, this.end - this.pos + 1) : this.highWaterMark
    let buffer = Buffer.alloc(howMuchToRead)
    fs.read(this.fd, buffer, 0, this.howMuchToRead, this.pos, (err, bytesRead) => {
      // bytesRead 为文件读取到的长度
      if (bytesRead > 0) {
        // 将读取的内容缓存到arr数组中
        this.arr.push(buffer)
        // 相关变量更新
        this.len += bytesRead
        this.pos += bytesRead
        this.reading = false
        // 缓存后触发实例上用户调用的read函数
        if (this.emitReadable) {
          this.emitReadable = false
          this.emit('readable')
        }
      }
    })
  }

  // 此处如上，省略...

}
```
当前 `1.txt` 文件中的内容为 `1234567890`。调用方式如下：
```javascript
let fs = require('fs')
let ReadableStream = require('./ReadableStream')
let rs = new ReadableStream('./1.txt', {
  autoClose: true,
  start: 0,
  flags: 'r',
  encoding: 'utf8',
  highWaterMark: 3
})
rs.on('readable', () => {
})
```
接下来从缓存区中读取数据。

## 二、读取长度小于水位线
当读取长度小于水位线时，使用原生方式调用，可以得到如下结果：
```javascript
let fs = require('fs')
let rs = fs.createReadStream('./1.txt', {
  autoClose: true,
  start: 0,
  flags: 'r',
  encoding: 'utf8',
  highWaterMark: 3
})
rs.on('readable', () => {
  let r = rs.read(2)
  // 输出结果为 12
  console.log(r)
})
```
由此可知，如果缓冲区内容够读，则返回结果结束读取。实现代码如下：
```javascript
class ReadableStream extends EventEmitter {

  // 此处如上，省略...

  // 可读流实例调用的方法
  read (n) {
    // 如果参数为空且不是在构造函数中调用此函数，n 默认按highWaterMark处理
    if (typeof n === 'undefined' && this.pos > this.start) {
      n = this.highWaterMark
    }

    // 如果读取长度小于缓存区长度，this.read(2) highWaterMark=3
    if (n > 0 && n <= this.len) {
      buffer = Buffer.alloc(n)
      let current
      let index = 0
      let flag = true
      while (flag && (current = this.arr.shift())) {
        for (let i = 0; i < current.length; i++) {
          buffer[index++] = current[i]
          if (index === n) {
            flag = false
            let other = current.slice(i + 1)
            if (other.length > 0) {
              this.arr.unshift(other)
            }
            this.len -= n
            break
          }
        }
      }
    }

    // 如果缓存区长度为0时，表明可以触发readable事件
    if (this.len === 0) {
      this.emitReadable = true
    }

    // 如果缓存区长度小于水位线时，则进行文件读取
    if (this.len < this.highWaterMark) {
      if (!this.reading) {
        this.reading = true
        this._read()
      }
    }

    // 根据编码方式处理数据
    if (buffer) {
      buffer = this.encoding ? buffer.toString(this.encoding) : buffer
    }
    return buffer
  }

  // 此处如上，省略...

}
```

## 三、读取长度等于水位线
当读取长度等于水位线时，使用原生方式调用，可以得到如下结果：
```javascript
let fs = require('fs')
let rs = fs.createReadStream('./1.txt', {
  autoClose: true,
  start: 0,
  flags: 'r',
  encoding: 'utf8',
  highWaterMark: 2
})
rs.on('readable', () => {
  let r = rs.read(2)
  // 输出结果为 12 34 56 78 90 null
  console.log(r)
})
```
由此可知，如果缓冲区内容读完为空，则返回结果继续读取。实现代码如下：
```javascript
class ReadableStream extends EventEmitter {

  // 此处如上，省略...

  read (n) {

    // 此处如上，省略...
    
    // 如果读取长度等于水位线，this.len 等于 0，表明可以触发readable事件，固_read后会触发readable函数
    if (this.len === 0) {
      this.emitReadable = true
    }

    // 如果缓存区长度小于水位线时，则进行文件读取
    if (this.len < this.highWaterMark) {
      if (!this.reading) {
        this.reading = true
        this._read()
      }
    }

    // 此处如上，省略...

    return buffer
  }

  // 此处如上，省略...

}
```

## 四、读取长度大于水位线
当读取长度大于水位线时，使用原生方式调用，可以得到如下结果：
```javascript
let fs = require('fs')
let rs = fs.createReadStream('./1.txt', {
  autoClose: true,
  start: 0,
  flags: 'r',
  encoding: 'utf8',
  highWaterMark: 3
})
rs.on('readable', () => {
  let r = rs.read(8)
  // 输出结果为 null 12345678 90
  console.log(r)
})
```
由此可知，如果缓冲区内容不够读，初次会返回 `null`，然后修改 `highWaterMark` 值继续读取返回，即为 `12345678`。此时，`this.len` 不等于 `0` 且小于 `this.highWaterMark`，会再次调用 `_read` 方法，如果读取文件为空，则需要手动触发一下 `readable` 事件。实现代码如下：
```javascript
class ReadableStream extends EventEmitter {

  // 此处如上，省略...

  // 可读流实例调用的方法
  read (n) {

    // 此处如上，省略...

    // 如果this.read(8) highWaterMark=3 
    if (n > this.len) {
      // 不够读时且文件没有读取完，修改highWaterMark继续读取
      if (!this.finished) {
        this.highWaterMark = computeNewHighWaterMark(n)
        this.reading = true
        this.emitReadable = true
        this._read()
      } else {
        // 否则直接返回缓存数据
        buffer = this.arr.shift()
      }
    }

    // 此处如上，省略...

    return buffer
  }
  // 真实读取文件的方法
  _read () {

    // 此处如上，省略...

    fs.read(this.fd, buffer, 0, howMuchToRead, this.pos, (err, bytesRead) => {
      // bytesRead 为文件读取到的长度
      if (bytesRead > 0) {

        // 此处如上，省略...

      } else {
        // this.len不等于0且小于this.highWaterMark，需要手动触发一下readable事件
        this.finished = true
        if (this.len) {
          this.emit('readable')
        } else {
          this.emit('end')
        }
      }
    })
  }

  // 此处如上，省略...

}
```
计算 `highWaterMark` 的函数如下：
```javascript
function computeNewHighWaterMark (n) {
  n--;
  n |= n >>> 1;
  n |= n >>> 2;
  n |= n >>> 4;
  n |= n >>> 8;
  n |= n >>> 16;
  n++;
  return n;
}
```

[源码](https://github.com/zdddrszj/code/blob/master/stream/readableStream/index.js)

😋😋😋，好了，全部功能已经实现，就到此结束吧！