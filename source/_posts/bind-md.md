---
title: bind 简单介绍
date: 2018-09-13 19:02:49
categories: [前端]
tags: [bind] 
---
<!-- toc -->

### 一、bind 方法的作用
#### 1、修改 this 指向
请看如下代码：
```js
const $ = document.querySelector
const div = $('div')
// Uncaught TypeError: Illegal invocation
```

这段代码报类型错误，因为window对象下并没有querySelector方法。可以用bind解决，代码如下：
```js
const $ = document.querySelector.bind(document)
const div = $('div')
// 返回结果 <div>......</div>
```

其他实现方式：
```js
const $ = document.querySelector
const div = $.call(document,'div')
// 返回结果 <div>......</div>
```
call 和 apply 函数的缺陷是不能返回一个新方法。

#### 2、可以实现偏函数的效果
bind方法可以返回一个新函数，而且可以在绑定的时候传入其他参数，比如：
```js
const sum = (...args) => args.reduce((prev,next) => prev+next, 0)
const sum2 = sum.bind(null, 1)
console.log(sum2(1, 2)) // 4
```

#### 3、bind 多次时 this 总是指向第一次 bind 的对象
运行如下代码：
```js
const getName = function () {
  console.log(this.name)
}
const p1 = {
  name:'zhangsan'
}
const p2= {
  name: 'lisi'
}
const name = getName.bind(p1).bind(p2)
name() // zhangsan
```

#### 4、绑定构造函数
运行如下代码：
```js
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
  console.log(this.firstName + this.lastName)
}
const p = new Person('zhang', 'san') // zhangsan

const P = Person.bind(null, 'li')
const p3 = new P('si') // lisi

console.log(p instanceof Person) // true
console.log(p3 instanceof Person) // true
```
从如上结果可以看出，`bind` 实现中要根据当前调用函数 `this` 是否是被绑定函数的实例，如果是，修改 `this` 为当前调用者的 `this`

### 二、bind 方法的实现
> 语法：`fun.bind(thisArg[, arg1[, arg2[, ...]]])`
  参数：`thisArg` -> 调用绑定函数时作为this参数传递给目标函数的值。
		`arg1, arg2, ... ->` 当绑定函数被调用时，这些参数将置于实参之前传递给被绑定的方法。
 返回值：返回由指定的 `this` 值和初始化参数改造的原函数拷贝

#### 1、修改 this 指向
```js
Function.prototype.bind = function (oThis) {
  const self = this
  return function () {
    self.apply(oThis, arguments)
  }
}
```

#### 2、实现类似 curry 化的效果
```js
Function.prototype.bind = function (oThis) {
  const self = this
  var aArgs = Array.prototype.slice.call(arguments, 1)
  return function () {
   return self.apply(oThis, aArgs.concat(Array.prototype.slice.call(arguments)))
  }
}
```
#### 3、绑定构造函数
```js
Function.prototype.bind = function (oThis) {
  const self = this
  var aArgs = Array.prototype.slice.call(arguments, 1)
  const fun = function () {}
  const fBound = function () {
	const currentThis = this instanceof fun ? this : oThis
    return self.apply(currentThis, aArgs.concat(Array.prototype.slice.call(arguments)))
  }
  fun.prototype = this.prototype
  fBound.prototype = new fun()
  return fBound
}
```

#### 4、mozilla 官方实现
```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          // this instanceof fNOP === true时,说明返回的fBound被当做new的构造函数调用
          return fToBind.apply(this instanceof fNOP
                 ? this
                 : oThis,
                 // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    // 维护原型关系
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype; 
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```