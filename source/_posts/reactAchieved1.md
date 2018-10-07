---
title: React 简单实现（一）
date: 2018-10-07 09:46:10
categories: [前端]
tags: [react]
---

<!-- toc -->
`React` 是一款用于构建用户界面的 `JavaScript` 库。它以声明式编写 `UI`，创建拥有各自状态的组件，再由组件构成更加复杂的界面。

## 一、前言
在 `React` 中我们使用 `JSX` 语法来替代常规的  `JavaScript`，并通过 `Babel` 编译，比如例子1：
```javascript
// JSX语法
let str = <h1 className="title">hello</h1>

// Babel编译后
var str = React.createElement(
  "h1",
  { className: "title" },
  "hello"
)
```
然后我们控制台输出 `str`，得到结果如下：
![](https://user-gold-cdn.xitu.io/2018/7/27/164dc02769830bd2?w=24&f=png&s=30515)
例子2：
```javascript
// JSX语法
let App = function MyComponent (props) {
  return <h1>hello{props.value}</h1>
}

// Babel编译后
var App = function MyComponent(props) {
  return React.createElement(
    "h1",
    null,
    "hello",
    props.value
  )
}
```
控制台输出 `App`，得到结果如下：
![](https://user-gold-cdn.xitu.io/2018/7/27/164dc63f58c8a996?w=378&h=50&f=png&s=31369)

这个对象就是虚拟 `DOM` 对象，最后通过 `ReactDOM.render` 方法将虚拟 `DOM` 解析渲染到页面上。下面我们就分别来实现 `createElement` 和 `render` 方法。

## 二、createElement 实现
此方法就是实现一个数据结构，把 `JSX` 编译后的结构以嵌套形式保存在数据结构对象中，实现代码如下：

```javascript
/**
 * Create and return a new ReactElement of the given type.
 */
function createElement (type, props, ...children) {
  return {
    type,
    props: {
      ...props,
      children: children.length <= 1 ? children[0] : children
    }
  }
}
```

## 三、render 实现
`render` 函数就是解析虚拟 `DOM`，然后通过 `appendChild` 渲染到页面上，实现代码如下：

```javascript
/**
 * 将真实dom渲染到页面
 * @param {*虚拟dom} vnode 
 * @param {*页面容器} container 
 * @param {*渲染后的回调} callback 
 */
function render (vnode, container, callback) {
  container.appendChild(_render(vnode))
  callback && callback()
}
/**
 * 将虚拟dom转成真实节点
 * @param {*虚拟dom} vnode 
 */
function _render (vnode) {}
```
但是虚拟 `DOM` 的有三种类型，不同类型有不同的渲染方式，下面我们分别来分析一下。
### 1、字符串类型
字符串类型即是文本，比如上面的 `hello`，直接创建文本节点返回即可，实现代码如下：
```javascript
function _render (vnode) {
  // 如果是普通字符
  if (util.isString(vnode) || util.isNumber(vnode)) {
    return document.createTextNode(vnode)
  }
}
```
### 2、标签类型
常见标签如 `div/h1/span` 等，创建元素标签后需要遍历设置属性，实现代码如下：
```javascript
function _render (vnode) {
  let { type, props } = vnode
  let { children, ...attrs } = props
  // 如果是标签元素，如 div span
  let dom = document.createElement(type)
  if (children) {
    if (util.isArray(children)) {
      children.forEach(child => {
        render(child, dom)
      })
    } else {
      dom.appendChild(document.createTextNode(children))
    }
  }
  // 设置属性
  setAttributes(attrs, dom)
  return dom
}
```
### 3、函数类型
函数类型也分二种情况，一种是无状态普通函数即函数组件，另一种则是继承了 `React.Component` 类的有状态函数，即类组件。开始之前需要定义一个  `Component` 父类以便继承，实现代码如下：
```javascript
/**
 * Create React.Component class
 */
class Component {
  constructor (props) {
    this.props = props
    this.state = {}
  }
  componentWillMount () {
    console.log('Component: componentWillMount')
  }
  componentDidMount () {
    console.log('Component: componentDidMount')
  }
  setState () {
    console.log('Component: setState')
  }
}
```


不管哪种组件，为了保证行为统一性，我们可以分别经过组件创建，组件渲染，然后返回真实 `DOM` 节点，最后渲染到页面上的步骤。实现代码如下：
```javascript
function _render (vnode) {
  let { type, props } = vnode
  let { children, ...attrs } = props
  // 如果是函数，说明是组件（包括函数组件和类组件）
  if (util.isFunction(type)) {
    // 创建组件
    let comp = createComponent(type, props)
    comp.props = props
    // 渲染组件
    let dom = renderComponent(comp)
    comp.dom = dom
    // 设置属性
    setAttributes(attrs, dom)
    return dom
  }
}
// 创建组件
function createComponent (comp, props) {
  // 如果是类组件，需要new一个实例，可以调用render方法
  if (comp.prototype.render) {
    comp = new comp(props)
  } else {
    comp.render = function () {
      return comp(props)
    }
  }
  return comp
}

// 渲染组件：执行组件的render方法，然后通过_render将虚拟dom转成真实dom，最后返回真实节点
function renderComponent (comp) {
  let dom
  // 如果未渲染过组件
  if (!comp.dom) {
    if (comp.componentWillMount) {
      comp.componentWillMount()
    }
  }
  dom = _render(comp.render())
  if (comp.componentDidMount) {
    comp.componentDidMount()
  }
  return dom
}
```
到这里解析和渲染功能大体完成，下面介绍一下如何修改组件状态。

## 四、setState 实现
在上面我们通过 `setAttribute` 已经将事件绑定到元素上，事件中很有可能会修改组件状态，比如`this.setState({a: this.state.a + 1})` 语句，以此更新视图。不过通过上面虚拟 `DOM` 解析成真实 `DOM` 的学习，在这里先忽略 `DOM Diff` 功能，重新渲染还是很好实现的，代码如下：
```javascript
class Component {
  setState (newState) {
    console.log('Component: setState')
    Object.assign(this.state, newState)
    let old = this.dom
    // renderComponent：整体重新渲染，然后返回真实节点
    let newDom = renderComponent(this)
    old.parentNode.replaceChild(newDom, old)
    this.dom = newDom
  }
}
```

[源码](https://github.com/zdddrszj/react-source-study)

好了，今天的学习就先到这里吧😋😋😋～
