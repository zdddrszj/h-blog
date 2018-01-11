---
title: 3D 旋转心
date: 2017-07-06 11:05:12
categories: [前端]
tags: [CSS3, animation]
---

亲爱的小伙伴，今天在这里 `用 css3 实现 3D 旋转心` 作为小礼物送给大家。

制作之前首先要学习一下 `border-radius` 基础知识，因为今天的这个心就是她的杰作。大家可以参考 [CSS | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) 技术文档。
### 实现方法

大家看看下面这张图：

![qq 20151223112051](https://cloud.githubusercontent.com/assets/9649921/11970598/cc5bf296-a968-11e5-88c6-969b01eacbdf.png)

青色直线上部分的红色弧线是不是像一个一半的 `❤` 呢？没错，只要将外面黄色矩形设置一定的圆角比例，就会变成红色矩形，其中圆角比例我已在图片上标注，同时将矩形左边和下边边框长度设为`0px`，在将矩形沿`Z 轴` 旋转 `45deg` ，我们就会看到下面这个图形：

![1](https://cloud.githubusercontent.com/assets/9649921/11970674/3fd95cda-a96a-11e5-87f5-872123ebf998.png)

如果有36个这样的半心形，每个心形沿 `Y轴` 旋转 `10deg`，是不是就会变成下面这个图形呢？

![2](https://cloud.githubusercontent.com/assets/9649921/11970716/e43be77a-a96a-11e5-976f-32b2c4c3fea6.png)

说到这里，估计大家已经知道大概代码了吧~~~

``` html
<html>
    <head>
        <title>旋转心</title>
        <meta charset="utf-8" />
        <style>
            body{background-color:black;}
            .heart_div{
                width:200px;
                height:320px;
                transform-style:preserve-3d;
                position: relative;
                left: 0;top:0;bottom: 0;right: 0;
                margin:200px auto;
                /*border:1px solid yellow;*/
            }
            .heart_div [class^=heart]{
                width:200px;
                height:320px;
                border:solid red;
                border-width:2px 2px 0 0;
                border-radius:50% 50% 0%/40% 50% 0%;
                position: absolute;
            }
        </style>
    </head>
    <body>  
        <div class="heart_div"></div>
        <script>
            var heart_div = document.getElementsByClassName("heart_div")[0];
            for(var i = 1;i<37;i++){
                /*给heart_div添加36个子元素 且类名为 heart1 heart2 ...*/
                var div = document.createElement("div");
                div.className = "heart"+i;
                div.style.transform="rotateY("+i*10+"deg)  rotateZ(45deg) translateX(60px)";
                heart_div.appendChild(div);
            }           
        </script>
    </body>
</html>
```

主要部分完成了，下面就加个旋转动画吧! 详细如下：

``` css
<style>
    .heart_div{
        width:200px;
        height:320px;
        transform-style:preserve-3d;
        position: relative;
        left: 0;top:0;bottom: 0;right: 0;
        margin:200px auto;
        animation:play 10s linear infinite;
        -webkit-animation:play 10s linear infinite;
        -moz-animation:play 10s linear infinite;
        -o-animation:play 10s linear infinite;
        -ms-animation:play 10s linear infinite;
    }
    @keyframes play{
        to{
            transform:rotateY(360deg) rotateX(360deg) rotateZ(360deg);
        }
    }
    @-webkit-keyframes play{
        to{
            transform:rotateY(360deg) rotateX(360deg) rotateZ(360deg);
        }
    }
    @-moz-keyframes play{
        to{
            transform:rotateY(360deg) rotateX(360deg) rotateZ(360deg);
        }
    }
    @-o-keyframes play{
        to{
            transform:rotateY(360deg) rotateX(360deg) rotateZ(360deg);
        }
    }
    @-ms-keyframes play{
        to{
            transform:rotateY(360deg) rotateX(360deg) rotateZ(360deg);
        }
    }
</style>
```
<p><img class="emoji" alt=":bowtie:" src="https://assets-cdn.github.com/images/icons/emoji/bowtie.png" height="20" width="20" /><img class="emoji" alt=":bowtie:" src="https://assets-cdn.github.com/images/icons/emoji/bowtie.png" height="20" width="20"><img class="emoji" alt=":bowtie:" src="https://assets-cdn.github.com/images/icons/emoji/bowtie.png" height="20" width="20">  好了，到这里就完工啦！！！</p>

[查看demo](http://yixunfe.github.io/blog/demo/43/demo.html)