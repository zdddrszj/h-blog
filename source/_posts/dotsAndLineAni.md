---
title: 点线图动画
date: 2017-10-17 19:08:32
categories: [前端]
tags: [html5, canvas] 
---

世界浩瀚无边，然而都是由粒子组成；一句老话复杂的东西都是由简单的东西组成。

本文主题是一个大家经常看到的点线图简单动画，即下图，所以直接来代码吧！

![dotAndLineAni](http://zdddrszj.github.io/h5case/DotAndLineAni/dotAndLineAni.gif)

首先声明一些 `js` 变量（这些变量不用提前知晓，进入画图逻辑时自然知道需要声明哪些变量）
```javascript
	// 画布上所有点对象数组
	let dotsArr = [],
    canvas = document.getElementById('canvas'),
    ctx = canvas.getContext('2d'),
    width = document.documentElement.offsetWidth || document.body.offsetWidth,
    height = document.documentElement.offsetHeight || document.documentElement.offsetHeight,
    // 画布点的数量
    dotsNum = parseInt(width * height / 8000),
    // 两个点可以连线的间距临界值
    dotsDistance = 100
    // 画布点最大数量
    maxDotsNum = dotsNum * 1.5
    // 多出的点数量
    overDotsNum = 0
    // 设置画布大小（注意：这里不能用style代替）
    canvas.setAttribute('width', width)
    canvas.setAttribute('height', height)
```
html结构
```html
	<canvas id="canvas"></canvas>
```
### 一、画点
```javascript
// 点对象
function Dot() {
    this.canvas
    this.ctx
    // x轴坐标
    this.x
    // y轴坐标
    this.y
}
//点初始化方法
Dot.prototype = {
	init: function (canvas, x, y) {
        this.canvas = canvas
        this.ctx = this.canvas.getContext('2d')
        // 随机x轴坐标
        this.x = x || Math.random() * this.canvas.width
        // 随机y轴坐标
        this.y = y || Math.random() * this.canvas.height
        // 随机半径
        this.r = Math.random() * 4
        // 随机水平速度
        this.vx = Math.random() * 2 - 1
        // 随机垂直速度
        this.vy = Math.random() * 2 - 1
        // 画个实心点
        this.ctx.beginPath()
        this.ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI)
        this.ctx.fillStyle = "rgba(255, 255, 255, .8)"
        this.ctx.fill()
        this.ctx.closePath()
    }
}
```
```javascript
// 随机生成一定数量的点，画到画布上的同时，将其存储在dotsArr数组中，更新画布时用
for (var i = 0; i < dotsNum; i ++) {
    var dot = new Dot()
    dotsArr.push(dot)
    dot.init(canvas)
}
```
### 二、画线及动效
```javascript
var requestAnimationFrame = requestAnimationFrame || webkitRequestAnimationFrame || oRequestAnimationFrame || msRequestAnimationFrame
requestAnimationFrame(updateCanvas)
// 更新画布
function updateCanvas () {
    // 清空画布
    ctx.clearRect(0, 0, canvas.width, canvas.height)
    // 点的个数为 dotsNum ~ dotsNum，即 0 ~ dotsNum 或者 (dotsNum - maxDotsNum) ~ dotsNum
    if (dotsNum > maxDotsNum) {
        overDotsNum = dotsNum - maxDotsNum
    }
    for (let j = overDotsNum; j < dotsNum; j ++) {
    	// 更新点坐标
        dotsArr[j].update()
    }
    // 画线
    for (let m = overDotsNum; m < dotsNum; m ++) {
        for (let n = m + 1; n < dotsNum; n ++) {
            let dx = dotsArr[m].x - dotsArr[n].x
            let dy = dotsArr[m].y - dotsArr[n].y
            // 两个点之间的距离
            let d = Math.sqrt(Math.pow(dx, 2) + Math.pow(dy, 2))
            if (d < dotsDistance) {
                ctx.beginPath()
                ctx.moveTo(dotsArr[m].x, dotsArr[m].y)
                ctx.lineTo(dotsArr[n].x, dotsArr[n].y)
                ctx.strokeStyle = 'rgba(255, 255, 255, ' + (dotsDistance - d) / dotsDistance + ')'
                ctx.strokeWidth = 1
                ctx.stroke()
                ctx.closePath()
            }
        }
    }
    // 重新渲染画布
    requestAnimationFrame(updateCanvas)
}
// 更新点坐标
Dot.prototype.update = function () {
	// 获取此时该点坐标
	this.x = this.x + this.vx
    this.y = this.y + this.vy

    // 点越界后调用init重新随机生成 因为点的个数固定，屏幕实时绘制，所以移出屏幕的点对象不用考虑占内存之类的
    if (this.x < 0 || this.x > this.canvas.width) {
        this.init(this.canvas)
    }
    if (this.y < 0 || this.y > this.canvas.height) {
        this.init(this.canvas);
    }

    this.ctx.beginPath()
    this.ctx.arc(this.x, this.y, this.r, 0, 2 * Math.PI)
    this.ctx.fillStyle = "rgba(255,255,255,.8)"
    this.ctx.fill()
    this.ctx.closePath()
}
```
到这里基本功能已经完成，下边介绍一下鼠标点击和移动事件的动效。
### 三、鼠标点击事件
```javascript
// 事件绑定
document.addEventListener('click', handleClick)
// 回调函数
function handleClick (e) {
    let x = e.pageX
    let y = e.pageY
    for (let j = 0; j < 5; j ++) {
        dotsNum ++
        var dot = new Dot()
        dotsArr.push(dot)
        dot.init(canvas, x, y)
    }
}
```
### 四、鼠标移动事件
```javascript
// 事件绑定
document.addEventListener('mousemove', handleMousemove)
// 回调函数
function handleMousemove (e) {
    let x = e.pageX,
        y = e.pageY
    if ((x > 0 && x < width) && (y > 0 && y < height)) {
        // dot为画布上最后画的最后一个点
        dot.followMouse(x, y)
    }
}
// 鼠标跟随及重新定义点的坐标
Dot.prototype.followMouse = function (tx, ty) {
    this.x = tx
    this.y = ty
    this.ctx.beginPath()
    this.ctx.arc(this.x, this.y, this.r, 0, 2*Math.PI)
    this.ctx.fillStyle = "rgba(255,0,0,.8)"
    this.ctx.fill()
    this.ctx.closePath()
}
```
<g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji><g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji><g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji> 好了，到此就结束啦~

[查看demo](http://zdddrszj.github.io/h5case/DotAndLineAni/dotAndLineAni.html)