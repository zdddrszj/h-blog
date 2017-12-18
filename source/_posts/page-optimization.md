---
title: 移动端 H5 页面优化（一）
date: 2017-12-18 15:40:55
categories: [前端]
tags: [html5, performance] 
---

营销代有手段出，各领风骚数百天。要说现在哪些营销方式最能传播，屡屡刷爆朋友圈的 `H5` 页面肯定就是首当其冲的。所谓 `H5` 页面，即代表的是 `Html5` 页面，如今大部分浏览器已经支持`Html5` 页面。`Html5` 有强化 `Web` 网页的表现性能和追加本地数据库这些 `Web` 应用功能的优势。

广义论及的 `Html5` 实际指的是包含`HTML` 、`CSS` 和 `JavaScript` 在内的一套技术组合。今天主要阐述 `H5` 页面众多优化中的几个注意点。
### 页面按需渲染（列表为例）

在原生 `APP` 开发中，如果一个页面很长，`UITableView` 和 `UICollectionView` 都有一个控件池，利用 `Cell复用机制` 来做优化，可以理解为根据当前用户所在屏幕位置进行渲染，其他不渲染或者少渲染，以提高页面性能。但是在 `Hybrid App（混合模式移动应用）` 模式中，对`WebView` 中 `H5` 页面来说，如果一个列表项加载有超过 200 条以上，而且每个项目都有半屏图片时，对于性能劣一点的手机来说，当用户滚动加载更多或者点击项目进入相应详细内容时，可以明显感觉页面迟钝，这 200 条项目渲染起来确实有点压力山大，所以这个优化需要 `H5` 自身来做。

在`WebView` 中`Scroll` 事件不同于在普通浏览器中，会在滚动事件结束时才会被触发，这里我们结合 `setTimeout` 来按需渲染，只需要设置 `visibility: hidden` 样式即可。

变量说明：

``` javascript
var currentTabIndex = 0, // 当前列表索引（用于一个页面多个列表时）
    preTabIndex = 0, // 当前可视列表项索引
    listArrayObject = { // 当前列表对象集合
        '0': null,
        '1': null,
        '2': null,
         ...
  };
```

接口赋值：

``` javascript
// 当前列表索引 
var index = $(this).index(); 

// 把当前列表索引赋值给 currentTabIndex 
currentTabIndex = index; 

// 获取当前列表项
var lis = $('.index_list').eq(index).find('li'); 

// 每页10条，加载的最后页
var collection = lis.slice(lis.length - 10, lis.length); 

 // 对当前列表集合赋值
if (listArrayObject[index]) {
   listArrayObject[index].concat(collection)
} else {
    listArrayObject[index] = collection
}
```

按需渲染：

``` javascript
// 这里利用 requestAnimationFrame() 函数，可以更合理的重新排列动作序列，
// 如果浏览器不支持，只好用 setTimeout() 代替
var rAF = window.requestAnimationFrame  ||
    window.webkitRequestAnimationFrame  ||
    window.mozRequestAnimationFrame     ||
    window.oRequestAnimationFrame       ||
    window.msRequestAnimationFrame      ||
    function (callback) { window.setTimeout(callback, 1000 / 60); };

// 显示当前视窗中的列表项，隐藏超出部分
function showItem (collection, index) {

    // 隐藏之前显示的列表项
    collection.slice(preTabIndex,preTabIndex + 10).find('img').css('visibility', 'hidden');

    // 展示当前列表项 默认显示 10 条
    collection.slice(Math.max(index - 5, 0), Math.min(index + 5, collection.length)).find('img').css('visibility', 'visible');

   // 把当前列表项索引值存储在变量 preTabIndex 中，这样超出屏幕隐藏时就可以根据索引找到上一次显示的项目
   // 而不用遍历所有项目，时间复杂度从 O(n) 降到 O(10)
    preTabIndex = Math.max(index - 5, 0)
}

//监听滚动事件
var isScrollEnd = '';
$(window).on('scroll',function(){
    isScrollEnd = false;
});

//监听当前视口列表项
function isScrollFun(){
    if(!isScrollEnd){
        var winScroTop = $(window).scrollTop(),winHei = $(window).height();

        // 直接从 listArrayObject 集合中获取列表项目，而不是遍历 DOM 树，提高速度
        var indexListLi = listArrayObject[currentTabIndex];

        // 遍历列表项，步长为3，可根据屏幕最多承载项目数适当调整
        if(indexListLi && indexListLi.length > 0){ 
            for(var i = 0; i < indexListLi.length; i += 3){
                var li = indexListLi.eq(i),
                    top = li.offset().top;
                if(top > winScroTop && top < winScroTop+winHei){
                    // 把当前列表集合和屏幕视口当前列表项索引传入函数 showItem() 
                    showItem(indexListLi, i);
                    break
                }
            }
        }
        isScrollEnd = true;
    }
    rAF(isScrollFun);
}

// 调用主函数
isScrollFun();
```
### Touch 事件

在 `Hybrid App` 开发中，若下拉刷新是原生的，那么当用户下拉刷新时：
对于 `Android` ：会触发 `touchstart` 、`touchcancle`
对于 `IOS` ：会触发 `touchstart` 、`touchmove` 、`touchend` 

<br/>
## **Thanks**

<br/>
<div class="copyright">版权声明：版权归作者所有，任何形式转载请联系博主。</div>
