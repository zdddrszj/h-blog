---
title: Canvas 多球碰撞和反弹
date: 2017-12-27 14:21:44
categories: [前端]
tags: [html5, canvas] 
---
<!-- toc -->
现实世界中万千万物都有其独自的运行轨迹，例如直线、圆、螺旋和复杂的贝塞尔曲线。今天我们一起来学习一下如何用 `Canvas` 实现最平凡的直线运动。`Canvas` 画图基础知识可参考 [w3school 在线教程](http://www.w3school.com.cn/html5/html_5_canvas.asp) 。
### 一、矢量移动

要想为图像设置动画效果，可以采用每次为对象绘制不同的 `x` 坐标和 `y` 坐标，然后在每一帧中调用显示更新图像的函数即可。初始化起始点代码如下：

``` javascript
var p1 = {x:20,y:20}; //初始点坐标
var ball = {x:p1.x,y:p1.y,radius:10}; //小球初始坐标及半径
```

在两点之间移动很方便，但是很多时候并没有一个要移到哪里去的目标点，只有从哪里开始的起始点。这种情况下，创建一个 `vector` 作为移动对象就非常有用了。

矢量是一个具有数量和方向的物理量。数量就是对象移动的速度 `speed` 的值，方向就是对象移动的角度 `angle` 的值。现在，将对象移动的角度 `angle` 的值（方向）设为 `45°`，在数学上，平滑直线通常代表角度为 `0` ，`45°` 的矢量就意味着向右下方移动。

弧度 `radians` 是度量角度的标准单位，大部分数学计算都需要将角度转换为弧度才能使用。将角度转为弧度，使用标准方程式 `radians = angle * Math.PI/180` 即可。代码如下：

``` javascript
var angle = 45; //角度
var radians = angle * Math.PI/180; //弧度
```

计算对象沿矢量运动时的坐标值，请看下面简略2D坐标图：

![1](https://cloud.githubusercontent.com/assets/9649921/13239696/2de0e6e6-da17-11e5-8d5b-3f131eca04f5.png)

可以看出，余弦通常与 `x` 值有关，正弦通常与 `y` 值有关，可以利用 `sin` 和 `cos` 来计算对象沿矢量的移动。代码如下：

``` javascript
var speed = 5; //速度
var xunits = Math.cos(radians) * speed; //x坐标增量
var yunits = Math.sin(radians) * speed;  //y坐标增量
```

在渲染画布 `drawScreen()` 函数中，将 `ball.x` 和 `ball.y` 分别加上 `xunits` 和 `yunits` ，即可随时更新小球位置坐标点。

`drawScreen()` 函数详细如下：

``` javascript
//渲染画布
function drawScreen(){
    context.fillStyle = '#eee';
    context.fillRect(0,0,theCanvas.width,theCanvas.height);
    context.strokeRect(1,1,theCanvas.width-2,theCanvas.height-2);
    context.fillStyle = 'red';

    ball.x += xunits;
    ball.y += yunits;

    //小球撞墙检测
    if(ball.x+ball.radius > theCanvas.width){
        ball = {x:p1.x,y:p1.y,radius:10};
    }

    context.beginPath();
    context.arc(ball.x,ball.y,ball.radius,0,Math.PI*2,true);
    context.closePath();
    context.fill();
}
```

框架 `javascript` 代码如下：

``` javascript
window.addEventListener('load',eventWindowLoaded,false);
function eventWindowLoaded(){
    canvasApp();
}
//判断浏览器是否支持canvas标签
function canvasSupport(){
    return !!document.createElement('canvas').getContext;
}
//主函数
function canvasApp(){
    if(!canvasSupport()){
        return;
    }

    //初始化   代码省略...

    var theCanvas = document.getElementById('canvasOne');
    var context = theCanvas.getContext('2d');

    //渲染画布   代码省略...

    var timeOut = setInterval(drawScreen,100);
}
```

`html` 代码如下：

``` html
<div style="position:absolute;top:50px;left:50px;">
    <canvas id="canvasOne" width="200" height="200">
        该浏览器不支持canvas。
    </canvas>
</div>
```

最终实现效果如下：

![jdfw](https://cloud.githubusercontent.com/assets/9649921/13216469/e75cbd06-d996-11e5-8264-c437ca18a3ff.gif)

[查看demo](http://yixunfe.github.io/blog/demo/57/demo.html)
### 二、多球撞墙反弹

尽管创建一个有数量、有方向的矢量并让对象沿着它精确移动看起来很有动画感，但是现实中却很少出现类似的运动。大部分时候，人们会希望这个对象能对周围的世界有反应，例如撞上水平或者垂直的墙后能弹回来。

根据第一小节的学习，我们知道一个球如何沿矢量运动，那么多球撞墙反弹需要解决两个问题，分别是多球沿矢量运动和如何反弹。
#### 第一个问题：多球沿矢量运动

为实现多球应用程序，需要创建一个新的对象来控制关于每个反弹球的所有相关信息：`x、y、 radius、speed、angle、radians、xunits、yunits` 。这里假设我们要创建100个这样的随机小球，给予不同的速度，半径和矢量，如果用代码表示，即：

``` javascript
//定义球数量
var numBalls = 100;
//球最大半径 最小半径
var maxSize = 6,minSize = 4;
//定义球最大速度
var maxspeed=maxSize+5;
//球对象数组
var balls = [];
// 当前球 x坐标 y坐标 速度 角度 半径 弧度 x方向增量 y方向增量
var tempBall,tempX,tempY,tempSpeed,tempAngle,tempRadius,tempRadians,tempXunits,tempYunits;

//初始化100个球
for(var i = 0 ; i < numBalls; i ++){
    tempRadius = Math.floor(Math.random()*maxSize)+minSize;
    tempX = tempRadius + Math.floor(Math.random()*(theCanvas.width - tempRadius*2));
    tempY = tempRadius + Math.floor(Math.random()*(theCanvas.height - tempRadius*2));
    tempSpeed = maxspeed - tempRadius;
    tempAngle = Math.floor(Math.random()*360);
    tempRadians = tempAngle * Math.PI/180;
    tempXunits = Math.cos(tempRadians) * tempSpeed;
    tempYunits = Math.sin(tempRadians) * tempSpeed;

    tempBall = {x:tempX,y:tempY,radius:tempRadius,speed:tempSpeed,angle:tempAngle,radians:tempRadians,xunits:tempXunits,yunits:tempYunits};
    balls.push(tempBall);
}
```
#### 第二个问题：如何反弹

为了更好理解反弹动画，先来看一个简单的物理原理。这条原理经常用于光线上，但对于动画2D形状也非常有用，特别是对象撞上水平或垂直的墙反弹的情况。这个原理就是 **反射角原理**，即入射角等于反射角。

入射角是对象撞墙时的角度，反射角是对象从墙面反弹回来的角度。如下图所示：

![2](https://cloud.githubusercontent.com/assets/9649921/13242615/971719e0-da31-11e5-9ddd-888dc8878cf7.png)

由此可知，`反弹角 = 180 - 入射角`。

 `javascript` 代码如下：

``` javascript
// ball.x 代表小球 x 坐标 ball.y 代表小球 y 坐标
if(ball.x+ball.radius > theCanvas.width || ball.x-ball.radius < 0){
    ball.angle = 180 - ball.angle;
    updateBall(ball);
}else if(ball.y+ball.radius > theCanvas.height || ball.y-ball.radius < 0){
    ball.angle = 360 - ball.angle;
    updateBall(ball);
}
```

`updateBall(ball)` 函数详细如下：

``` javascript
function updateBall(ball){
     ball.radians = ball.angle * Math.PI/180;
     ball.xunits = Math.cos(ball.radians)*ball.speed;
     ball.yunits = Math.sin(ball.radians)*ball.speed;
}
```

到这里，主要代码已经全部给出，最终实现的效果如图：

![jdfw](https://cloud.githubusercontent.com/assets/9649921/13243052/18f53678-da36-11e5-8a84-b9bbf2d04e84.gif)

[查看demo](http://yixunfe.github.io/blog/demo/57/demo1.html)

<g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji><g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji><g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji>  好了，今天就到这里吧~

<br/>
**Thanks**

<br/>
<div class="copyright">版权声明：版权归作者所有，任何形式转载请联系博主。</div>