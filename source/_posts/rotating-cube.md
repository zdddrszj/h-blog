---
title: 3D 旋转立方体
date: 2017-12-18 17:51:41
categories: [前端]
tags: [CSS3, animation]
---

今天我们一起来学习一下如何用 css3 变换实现小时候玩的小魔方立方体。但是要提前复习一下 3D 变换的相关知识，你可以看一下 [w3cplus博客](http://www.w3cplus.com/content/css3-transform) 或者    [w3school文档](http://www.w3school.com.cn/cssref/pr_transform.asp)，由于目前 webkit 内核对 css3   属性兼容性比较好，为代码简洁，这里我们只考虑 chrome 浏览器，其他浏览器读者可自行加相应前缀来实现。

好了，准备就绪之后那我们就开始进入正题吧。首先，我们来看一道数学题：请想一下6个正方形如何摆放能拼成一个正方体呢？当然，方式有很多，这里只提出一种，如下：

![w](https://cloud.githubusercontent.com/assets/9649921/11388537/62bc684a-9372-11e5-8dc2-1a17cab5d109.png)

正方体 `前后` `左右` `上下` 分别用 `front` `back` `left` `right` `top` `bottom` 来表示。
### 实现方法

我们用自定义标签来实现，html 代码如下：

``` html
<body>
     <cube>
          <front></front>
          <back></back>
          <left></left>
          <right></right>
          <top></top>
          <bottom></bottom>
     </cube>
</body>
```

这里，给背景一个紫色到黑色的径向渐变，样式如下：

``` css
html{
    background:radial-gradient(ellipse at center,#430d6d 0%,#000 100%);
    height:100%;
}
```

设置正方体在浏览器窗口正中间，若想看到变换后的 3D 效果，需要给 `cube` 声明 3D 变换 `transform-style:preserve-3d;` ，同时给 `body` 设置 `视差` ，样式如下：

``` css
body{
    padding:0;
    margin:0;
    height:100%;
    /*设置视差*/
    perspective:1000px;
}
cube{
    width:20em;
    height:20em;
    /*在窗口居中*/
    position:absolute;left:0;top:0;right:0;bottom:0;margin:auto;
    /*声明 3D 变换*/
    transform-style:preserve-3d;
}
```

注意：`cube` 是正方体的父标签，6个子元素构成正方体，我们以父元素为相对坐标，分别对6个正方形相对父元素设置绝对定位。

这时，6个正方形和父元素重合，接下来，分别对其定位到上边图片对应位置，即 `back` `left` `right` `top` `bottom` 分别相对 `front` 进行位移和旋转，这样就可以拼成正方体啦~~~ 样式如下：

``` css
front{
     left:0px;
     top:0px;
     transform:translateZ(10em);
}
back{
     top:0px;
     left:0px;
     transform:translateZ(-10em);
}
left{
     left:-20em;
     top:0px;
     transform: translateZ(10em) rotateY(-90deg);
     transform-origin:right;
}
right{
     right:-20em;
     top:0px;
     transform:translateZ(10em) rotateY(90deg);
     transform-origin:left;
}
top{
     left:0px;
     top:-20em;
     transform:translateZ(10em) rotateX(90deg);
     transform-origin:bottom;
}
bottom{             
     left:0px;
     bottom:-20em;
     transform:translateZ(10em) rotateX(-90deg);
     transform-origin:top;
}
```

注意：看到这里，大家可能不明白为什么上面的 `transform` 都要加一个 `transform:translateZ(10em)` ，这时因为接下来我们要给 `cube` 加旋转动画，`cube` 就会以自身中心为旋转点（对应正方体 `front` 这个面为旋转点），导致 `正方体` 不能以自身中心为旋转点进行旋转，故而我们把构成它的这6个面分别沿 `Z轴` 向正方向位移 `10em` ，就是正方形边长一半，这样 `cube` 中心点和 `正方体` 中心点 `重合`。

到这里我们已经完成一半了，可是我们这个正方体并不是小时候玩的魔方那样，每个面还有 `4x4=16` 个小正方形，这里设置 `8x8=64` 个小正方形（`20em/2.5em=8`）。css3 可以为我们的背景添加渐变，若想复习一下渐变知识，也可以看一下  [w3cplus博客](http://www.w3cplus.com/content/css3-gradient)。样式如下：

``` css
cube{
    background:linear-gradient(rgba(0,0,0,0) 0px,rgba(54,226,248,.5) 0px,
                                        rgba(54,226,248,.5) 3px,rgba(0,0,0,0) 3px),
               linear-gradient(90deg,rgba(0,0,0,0) 0px,rgba(54,226,248,.5) 0px,
                                        rgba(54,226,248,.5) 3px,rgba(0,0,0,0) 3px);
    background-size:2.5em 2.5em,2.5em 2.5em;
    border:2px solid rgba(54,226,248,.5);
    box-shadow:0 0 5em rgba(0,128,0,.4);
}
```

最后一步，为了正方体看上去不是呆板的，可以对 `cube` 加上一个动画，样式如下：

``` css
cube{
    animation:cube 6s linear infinite;
    -webkit-animation:cube 6s linear infinite;
}
@keyframes cube{
    from{transform:translateZ(-10em) rotateX(0deg) rotateY(0deg);}
    to{transform:translateZ(-10em) rotateX(360deg) rotateY(360deg);}
}
@-webkit-keyframes cube{
    from{transform:translateZ(-10em) rotateX(0deg) rotateY(0deg);}
    to{transform:translateZ(-10em) rotateX(360deg) rotateY(360deg);}
}
```

不知道大家对上边动画 `transform:translateZ(-10em)` 是否理解，这是因为我们在努力为 `cube` 和 `正方体` 中心点重合时设置了 `transform:translateZ(10em)`  。

若想在正方体中添加文字，设置字体大小要注意一下，chrome 浏览器默认字体大小`16px` ，正方形宽度 `16px*20em=320px` ，故 `line-height:320px` ；同时给字体设置多重阴影 `text-shadow`，可以达到立体效果，详细如下：

``` html
<div>放飞梦想</div>
```

``` css
div{
    color:#fff;
    font-size:40px;
    font-weight:bold;
    text-align:center;
    line-height:320px;
    text-shadow:0 1px 0 #ccc,0 2px 0 #bbb,0 3px 0 #c9c9c9,0 4px 0 #bbb,0 4px 0 #b9b9b9,0 5px 0 #aaa;
}
```
<g-emoji alias="smile" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f604.png" ios-version="6.0">😄</g-emoji><g-emoji alias="smile" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f604.png" ios-version="6.0">😄</g-emoji><g-emoji alias="smile" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f604.png" ios-version="6.0">😄</g-emoji>  好了，到这里就完工啦！！！

[查看demo](http://yixunfe.github.io/blog/demo/36/demo.html)

<br/>
### **Thanks**

<br/>
<div class="copyright">版权声明：版权归作者所有，任何形式转载请联系博主。</div>
