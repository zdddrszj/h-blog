---
title: ES6中类的实现及继承原理
date: 2018-06-19 13:41:16
categories: [前端]
tags: [es6, class]
---
<!-- toc -->
## 一、用ES5实现继承

### 1、创建父类
```javascript
function Father () {
  // 私有属性
  this.name = 'father'
  // 私有方法
  this.eat = function () {
    console.log('eat')
  }
}
// 公有方法
Father.prototype.sleep = function () {
  console.log('sleep')
}

Father() // 此时控制台输出挂载在window对象上的方法，可知没有创建实例时this指向window

let father = new Father()
father.eat() // eat
father.sleep() // sleep
console.log(father) // Father { name: 'father', eat: [Function] }
```

### 2、创建子类并继承
```javascript
function Child () {
  // 继承父类的私有属性和方法
  Father.call(this)
  // 自己的私有属性
  this.name = 'child'
  // 自己的私有方法
  this.smoke = function () {
    console.log('smoke')
  }
}
// 继承父类的公共方法：将Child的prototype指向Father的prototype，同时修改Child.prototype.constructor为Child本身
Child.prototype = Object.create(Father.prototype, { constructor: { 
  value: Child,
  enumerable: false,
  writable: true,
  configurable: true
}})
// 修改原型链：将Child的原型链指向所从属的原型上，即Child.__proto__ = Father，或者
Object.setPrototypeOf(Child, Father)

let child = new Child()
child.eat() // eat
child.sleep() // sleep
child.smoke() // smoke
console.log(Child.__proto__.prototype === Father.prototype) // true
```

## 二、用es6实现继承
```javascript
class Father {
  constructor () {
    // 私有属性
    this.name = 'father'
  }
  // 公有方法
  eat () {
    console.log('eat')
  }
  // 静态方法，可直接访问的私有方法
  static sleep () {
    console.log('sleep')
  }
}

class Child extends Father {
  constructor() {
    super()
    // 私有属性
    this.name = 'child'
    this.age = 12
  }
  // 公有方法
  smoke () {
    console.log('smoke')
  }
}

let child = new Child()
child.eat() // eat
Child.sleep() // sleep
child.smoke() // smoke
```

## 三、手动实现ES6中类及继承
### 1、实现父类
```javascript
// 因为类实例需要new创建，不能直接使用，所以需要一个类调用检查函数，即_classCallCheck函数
function _classCallCheck (instance, Constructor) {
  if (!instance instanceof constructor) {
    throw new TypeError('cannot call a class as a function')
  }
}
// 类创建函数： Constructor是原型 protoProps是公有方法 staticProps是静态方法，也是私有方法
function _createClass(Constructor, protoProps, staticProps) {
  function defineProperties(target, props) {
    for (let i = 0; i < props.length; i++) {
      Object.defineProperty(target, props[i].key, {
        value: props[i].value,
        enumerable: true,
        writable: true,
        configurable: true
      })
    }
  }
  // 添加公有方法
  if (protoProps) {
    defineProperties(Constructor.prototype, protoProps)
  }
  // 添加静态方法，也是私有方法
  if (staticProps) {
    defineProperties(Constructor, staticProps)
  }
  return Constructor
}
let Father = function () {
  function Father () {
    // 类调用检查
    _classCallCheck(this, Father)
    // 私有属性
    this.name = 'father'
    // return {
    //   hobby: 'basketball'
    // }
  }
	// 下面是实现类，即把公有方法绑定到Father的prototype上，静态方法直接放到Father上
  _createClass(Father,[
    {
      key: 'eat',
      value: function eat () {
        console.log('eat')
      }
    }
  ],
  [
    {
      key: 'sleep',
      value: function sleep () {
        console.log('sleep')
      }
    }
  ])
  return Father
}()
```

### 2、实现子类及继承
```javascript
// 函数作用：继承父类的公共方法和修改原型链
function _inherits(subClass, superClass) {
  if (typeof superClass !== 'function' && superClass !== null) {
    throw new TypeError('super expression must either be null or a function,not' + typeof superClass)
  }
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: {
      value: subClass,
      enumerable: false,
      writable: true,
      configurable: true
    }
  })
  if (subClass) {
    Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass
  }
}
let Child = function (Father) {
  // 继承父类的公有方法：将Child的prototype指向Father的prototype，同时修改Child.prototype.constructor为Child本身
  // 修改原型链：将Child的原型链指向所从属的原型上，即Child.__proto__ = Father
  _inherits(Child, Father)

  function Child() {
    // 类调用检查
    _classCallCheck(this, Child)
    // 继承私有属性
    let obj = Father.call(this)
    // 这里是因为父类函数可以返回一个新对象（见父类注释）
    let that = this
    if (typeof obj === 'object') {
      that = obj
    }
    that.name = 'Child'
    that.age = 12
    return that
  }
  _createClass(Child, [{
    key: 'smoking',
    value: function smoking() {
      console.log('smoking')
    }
  }])
  return Child
}(Father)
```