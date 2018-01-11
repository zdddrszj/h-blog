---
title: DOM 中的范围 
date: 2017-05-24 12:18:10
categories: [前端]
tags: [range]
---

<!-- toc -->
为了让开发人员更方便控制页面，`DOM2` 级遍历和范围模块定义了 **范围 （range）** 接口。通过范围可以选择文档中的一个区域，而不必考虑节点的界限（选择在后台完成，对用户是不可见的）。 `Firefox`、`Opera`、`Safari` 和 `Chrome` 都支持 `DOM` 范围。IE 以专有方式实现了自己的范围特性。
# DOM 中的范围

`DOM2` 级在 `Document` 类型中定义了 `createRange()` 方法。在兼容 `DOM` 的浏览器中，使用这个方法属于 `document` 对象。使用 `hasFeature()` 或者直接检测该方法，都可以确定浏览器是否支持范围。

``` javascript
var supportsRange = document.implementation.hasFeature("Range","2.0");
var alsoSupportsRange = (typeof document.createRange == "function");
```

如果浏览器支持范围，那么就可以使用 `createRange()` 来创建 `DOM` 范围，例如：

``` javascript
var range = document.createRange();
```

与节点类似，新创建的范围与创建它的文档关联在一起，不能用于其他文档。
每个范围由一个`Range` 类型的实例表示，这个实例拥有很多属性和方法。下列属性提供了当前范围在文档中的位置信息：

1、`startContainer`：包含范围起点的节点（即选区中第一个节点的父节点）。
2、`startOffset`：范围在 startContainer 中起点的偏移量。
3、`endContainer`：包含范围终点的节点（即选区中最后一个节点的父节点）。
4、`endOffset`：范围在 endContainer 中终点的偏移量。
5、`commonAncestorContainer`：startContainer 和 endContainer 共同的祖先节点。

下面简单介绍一下范围的使用方法：
### 1、用 DOM 范围实现简单选择

要使用范围来选择文档中的一部分，最简单的方式是使用 `selectNode()` 或 `selectNodeContents()`，例如：

``` html
<html>
    <head>
        <meta charset="utf-8" />
    </head>
    <body>
        <p id="p1"><b>Hello</b> world!</p>
        <script>
            var range1 = document.createRange(),
                range2 = document.createRange(),
                p1 = document.getElementById("p1");
            range1.selectNode(p1);
            range2.selectNodeContents(p1);      
        </script>
    </body>
</html>
```

两个范围包含文档中不同部分，如图：
![1](https://cloud.githubusercontent.com/assets/9649921/11467545/c48f7f2a-9783-11e5-8f56-0b63c1ade9b1.png)
在调用 `selectNode()` 时，`startContainer`、`endContainer` 都等于传入节点的父节点，也就是 `document.body`。而 `startOffset` 属性等于给定节点在其父节点的 `childNodes` 集合中的索引（在这个例子中是 `1` ，因为兼容 `DOM` 的浏览器将空格算作一个文本节点），`endOffset` 等于 `startOffset` 加 `1` （因为只选择了一个节点）。
在调用 `selectNodeContents()` 时，`startContainer`、`endContainer` 等于传入的节点，即这个例子中的 `<p>` 元素。而 `startOffset` 属性始终等于 `0` ，因为范围从给定节点的第一个子节点开始。最后，`endOffset` 等于子节点的数量（node.childNodes.length），在这个例子中是 `2` 。例如：

``` javascript
console.log(range1.startContainer); //<body>...</body>
console.log(range2.startContainer); //<p id="p1">...</p>
console.log(range1.endContainer); //<body>...</body>
console.log(range2.endContainer); //<p id="p1">...</p>
console.log(range1.startOffset); //1
console.log(range2.startOffset); //0
console.log(range1.endOffset); //2
console.log(range2.endOffset); //2
```

此外，为了更精细地控制哪些节点包含在范围内，还可以使用下列方法：

1、`setStartBefore(refNode)`：将范围的起点设置在 refNode 之前。
2、`setStartAfter(refNode)`：将范围的起点设置在refNode 之后。
3、`setEndBefore(refNode)`：将范围的终点设置在 refNode 之前。
4、`setEndAfter(refNode)`：将范围的终点设置在 refNode 之后。
### 2、用 DOM 范围实现复杂选择

要创建复杂的范围就得使用 `setStart()` 和 `setEnd()` 方法。这两个方法都接受两个参数：一个参照节点和一个偏移量值。对 `setStart()` 来说，参照节点会变成 `startContainer`，而偏移量值会变   `startOffset` 。对于 `setEnd()` 来说，参照节点会变成 `endContainer` ，而偏移量值会变成 `endOffset`。
可以使用这两个方法来模拟 `selectNode()` 和 `selectNodeContents()` 。例如：

``` javascript
var range1 = document.createRange(),
    range2 = document.createRange(),
    p1 = document.getElementById("p1"),
    p1Index = -1,i,len;
for(i = 0, len = p1.parentNode.childNodes.length; i < len; i++){
    if(p1.parentNode.childNodes[i] == p1){
        p1Index = i;
        break;
    }
}
range1.setStart(p1.parentNode,p1Index);
range1.setEnd(p1.parentNode,p1Index+1);
range2.setStart(p1,0);
range2.setEnd(p1,p1.childNodes.length);
```

假设你只想选择前面 `HTML` 示例代码中从 `hello` 的 `llo` 到 `world!` 的 `o` ，可以这样做：

``` javascript
var p1 = document.getElementById("p1"),
    helloNode = p1.firstChild.firstChild,
    worldNode = p1.lastChild;
var range = document.createRange();
range.setStart(helloNode,2);
range.setEnd(worldNode,3);
```

由于 `helloNode` 和 `worldNode` 都是文本节点，因此它们分别变成了新建范围的 `startContainer` 和 `endContainer` 。此时 `startOffset` 和 `endOffset` 分别用以确定两个节点所包含的文本中的位置，而不是用以确定子节点的位置。如图：
![2](https://cloud.githubusercontent.com/assets/9649921/11468680/5ef0bfd6-978c-11e5-9c91-31f038224c73.png)
### 3、操作 DOM 范围中的内容

对前面的例子而言，范围经过计算知道选取中缺少一个开始的 `<b>` 标签，因此就会在后台动态添加一个该标签，同时还会在前面加入一个表示结束的 `<b>` 标签。于是修改后的 `DOM` 就变成如下所示：
`<p><b>He</b><b>llo</b> world!</p>`。

像这样创建了范围之后，就可以使用各种方法对范围的内容进行操作了。
第一个方法：`deleteContents()`，从文档中删除范围所包含的内容。例如：

``` javascript
range.deleteContents();
```

执行这句代码之后，页面会显示如下 `HTML` 代码：

``` html
<p id="p1"><b>He</b>rld!</p>
```

第二个方法：`extractContents()` ，也会从文档中移除范围选区，但会返回文档片段。例如：

``` javascript
var fragment = range.extractContents();
p1.parentNode.appendChild(fragment);
```

执行这句代码之后，页面会显示如下 `HTML` 代码：

``` html
<p id="p1"><b>He</b>rld!</p><b>llo</b> wo
```

第三个方法：`cloneContents()` ，会创建范围对象的一个副本，然后插入到其他地方。例如：

``` javascript
var fragment = range.cloneContents();
p1.parentNode.appendChild(fragment);
```

执行这句代码之后，页面会显示如下 `HTML` 代码：

``` html
<p id="p1"><b>Hello</b> world!</p><b>llo</b> wo
```
### 4、插入 DOM 范围中的内容

利用范围，可以删除或复制内容，也可以使用 `insertNode()` 方法像范围选区的开始处插入一个节点。假设我们在前面例子中的 `HTML` 前面插入以下 `HTML` 代码：`<span style="color:red;">Inserted text</span>`  ，那么可以使用以下代码：

``` javascript
var span = document.createElement("span");
span.style.color = "red";
span.appendChild(document.createTextNode("Inserted text"));
range.insertNode(span);
```

执行这句代码之后，页面会显示如下 `HTML` 代码：

``` html
<p id="p1"><b>He<span style="color: red;">Inserted text</span>llo</b> world!</p>
```

注意：`<span>` 正好被插入到了 `hello` 中的 `llo` 前面，而该位置就是范围选区的开始位置。还要注意的是，这里并没有添加或删除 `<b>` 元素，使用这种技术可以插入一些帮助提示信息，例如在打开新窗口的链接旁插入一幅图像。

除了像范围内部插入内容之外，还可以围绕范围插入内容，此时就要使用 `surroundContents()` 方法。该方法接受一个参数，即环绕范围内容的节点。例如：

``` javascript
range.selectNode(helloNode);
var span = document.createElement("span");
span.style.backgroundColor = "yellow";  
range.surroundContents(span);
```

执行这句代码之后，会给范围选区加上一个黄色背景。得到的 `HTML` 代码如下所示：

``` html
<p id="p1"><b><span style="background-color: yellow;">Hello</span></b> world!</p>
```

注意：为了插入 `<span>` ，范围必须包含整个 `DOM` 选区（不能仅仅包含选中的 `DOM` 节点）。
### 5、折叠 DOM 范围

所谓 **折叠范围** ，就是指范围中未选择文档的任何部分。使用 `collapse()` 方法来折叠范围，该方法接受一个参数，一个布尔值，表示要折叠到范围的哪一端。参数 `true` 表示要折叠到范围的起点，参数 `false` 表示要折叠到范围的终点。要确定范围已经折叠完毕，可以检查 `collapsed` 属性，如下所示：

``` javascript
range.collapse(true); //折叠到起点
alert(range.collapsed); //输出 true
```

检测某个范围是否处于折叠状态，可以帮助我们确定范围中的两个节点是否紧密相邻。例如，对于下面的 `HTML` 代码：

``` html
<p id="p1">Paragraph 1</p><p id="p2">Paragraph 2</p>
```

我们假设不知其实际构成（比如动态生成的），那么可以像下面这样创建一个范围：

``` javascript
var p1 = document.getElementById('p1'),
    p2 = document.getElementById('p2'),
    range = document.createRange();
range.setStartAfter(p1);
range.setEndBefore(p2);
alert(range.collapsed); //true
```

在这个例子中，创建的范围是折叠的，所以 `p1` 和 `p1` 是相邻的。
### 6、比较 DOM 范围

在有多个范围的情况下，可以使用 `compareBoundaryPoints()` 方法来确定这些范围是否有公共的边界。这个方法接受两个参数：表示比较方式的常量值和要比较的范围。表示比较方式的常量如下：

1、`Range.START_TO_START`：比较第一个范围和第二个范围的起点；
2、`Range.START_TO_END`：比较第一个范围的起点和第二个范围的终点；
3、`Range.END_TO_END`：比较第一个范围和第二个范围的终点；
4、`Range.END_TO_START`：比较第一个范围的终点和第二个范围的起点；

`compareBoundaryPoints()`方法返回值如下：如果第一个范围中的点位于第二个范围中的点之前，返回 `-1` ；如果两个点相等，返回 `0` ；如果第一个范围中的点位于第二个范围中的点之后，返回 `1` 。例如：

``` javascript
var range1 = document.createRange(),
    range2 = document.createRange(),
    p1 = document.getElementById("p1");
range1.selectNodeContents(p1);
range2.selectNodeContents(p1);
range2.setEndBefore(p1.lastChild);
alert(range1.compareBoundaryPoints(Range.START_TO_START,range2)); //0
alert(range1.compareBoundaryPoints(Range.END_TO_END,range2)); //1
```

这个例子中，两个范围的起点相同，第一个范围的终点位于第二个范围的终点后面，如图：
![1](https://cloud.githubusercontent.com/assets/9649921/11490882/d46e1548-9817-11e5-975c-deb496beefce.png)
### 7、复制 DOM 范围

可以使用 `cloneRange()` 方法复制范围，这个方法会创建调用它的范围的一个副本。

``` javascript
var newRange = range.cloneRange();
```

新创建的范围与原来的范围包含相同的属性，而修改它的端点不会影响原来的范围。
### 8、清理 DOM 范围

在使用完范围后，最好是调用 `detach()` 方法，以便从创建范围的文档中分离出该范围。调用 `detach()` 之后，就可以放心地解除对范围的引用，从而让垃圾回收机制回收其内存了。例如：

``` javascript
range.detach(); //从文档中分离
range = null; //解除引用
```

在使用范围的最后在执行这两个步骤是我们推荐的方式，一旦分离范围，就不能在恢复使用了。
# IE8及更早版本中的范围

`IE9` 支持范围，但 `IE8` 及之前版本不支持范围。不过，`IE8` 及早起版本支持一种类似的概念，即 **文本范围（text range）** 。文本范围是 `IE` 专有特性，主要处理文本（不一定是 `DOM` 节点）。通过 `<body>`、
`<button>`、`<input>`和`<textarea>`等几个元素，可以调用 `createTextRange()` 方法来创建文本范围。例如：

``` javascript
var range = document.body.createTextRange();
```

与 `DOM` 范围类似，使用 `IE` 文本范围的方式也有很多种。
### 1、用 IE 范围实现简单的选择

选择页面中某一区域最简单的方式，就是使用范围的 `findText()` 方法。这个方法会找到第一次出现的给定文本，并将范围移过来以环绕该文本。如果没找到文本，这个方法返回 `false` ；否则，返回 `true` 。同样，仍然以下面的 `HTML` 代码为例：

``` html
<p id="p1"><b>Hello</b> world!</p>
```

要选择 `Hello` ，可以使用以下代码：

``` javascript
var range = document.body.createTextRange();
var found = range.findText("Hello");
alert(found); //true
alert(range.text); //"Hello"
```

`IE` 中与 `DOM` 中的 `selectNode()` 方法最接近的方法是 `moveToElementText()` ，这个方法接受一个 `DOM` 元素，并选择该元素的所有文本，包含 `HTML` 标签，例如：

``` javascript
var range = document.body.createTextRange(),
    p1 = document.getElementById("p1");
range.moveToElementText(p1);
```

在文本范围中包含 `HTML` 的情况下，可以使用 `htmlText` 属性取得范围的全部内容，例如：

``` javascript
alert(range.htmlText);
```

`IE` 的范围没有任何属性可以随着范围选区的变化而动态更新。不过，其 `parentElement()` 方法倒是与`DOM` 的 `commonAncestorContainer` 属性类似。例如：

``` javascript
var ancestor = range.parentElement();
```

这样就得到了范围选区的父节点。
### 2、使用 IE 范围实现复杂的选择

在 `IE` 中创建复杂范围的方法，就是以增量向四周移动范围。为此，`IE` 提供了 4 个方法：
`move()`、`moveStart()`、`moveEnd()` 和 `expand()`。
这些方法都接受两个参数：移动单位和移动单位的数量。其中，移动单位是下列一种字符串值：

1、`character`：这个字符地移动；
2、`word`：逐个单词（一系列非空格字符）地移动；
3、`sentence`：逐个句子（一系列以句号、问号或叹号结尾的字符）地移动；
4、`textedit`：移动到当前范围选区的开始或结束位置。

通过 `moveStart()` 方法可以移动到范围的起点，通过 `moveEnd()` 方法可以移动到范围的终点，移动的幅度由单位数量指定，例如：

``` javascript
range.moveStart("word",2); //起点移动 2 个单词
range.moveEnd("character",1); //终点移动 1 个字符
```

使用 `expand()` 方法可以将范围规范化。换句话说，可以将任何部分选择的文本全部选中。例如：

``` javascript
range.expand("word"); //可以将整个单词选中
```

而 `move()` 方法则先会折叠当前范围，然后再将范围移动到指定的单位数量，例如：

``` javascript
range.move("character",5); //移动 5 个字符
```
### 3、操作 IE 范围中的内容

在 `IE` 中操作范围中的内容可以使用 `text` 属性或 `pasteHTML()` 方法。如前所述，通过 `text` 属性可以取得范围中的内容文本；但是，也可以通过这个属性设置范围中的内容文本。例如：

``` javascript
var range = document.body.createTextRange();
range.findText("Hello");
range.text = "Hi";
```

执行以上代码后的 `HTML` 代码如下：

``` html
<p id=p1><b>Hi</b> world!</p>
```

这时，`HTML` 标签保持不变，若要像范围中插入 `HTML` 代码，就得使用 `pasteHTMl()` 方法，例如：

``` javascript
var range = document.body.createTextRange();
range.findText("Hello");
range.pasteHTML("<em>Hi</em>");
```

执行以上代码后的 `HTML` 代码如下：

``` html
<p id=p1><b><em>Hi</em></b> world!</p>
```

注意：在范围中包含 `HTML` 代码时，不应该使用 `pasteHTML()` ，因为这样很容易导致不可预料的结果（很可能是格式不正确的 `HTML`）。
### 4、折叠 IE 范围

`IE` 为范围提供的 `collapse()` 方法与 `DOM` 方法用法一样，可惜，没有对应的 `collapsed` 属性让我们知道范围是否已经折叠完毕。为此，必须使用 `boundingWidth` 属性，该属性返回范围的宽度（以像素为单位）。如果等于 `0` 说明范围已经折叠。例如：

``` javascript
range.collapse(true);
alert(range.boundingWidth); //0
```

此外，还有 `boundingHeight`、`boundingLeft` 和`boundingTop` 等属性，可以提供一些位置信息。
### 5、比较 IE 范围

`IE` 中的 `compareEndPoints()` 方法与 `DOM` 范围的 `compareBoundaryPoints()` 方法类似。这个方法接受两个参数：比较的类型和要比较的范围。比较类型的取值范围是下列几个字符串值：`StartToStart`、
`StartToEnd`、`EndToEnd`、`EndToStart`。 `compareEndPoints()` 方法返回值同 `compareBoundaryPoints()` 方法。例如：

``` javascript
var range1 = document.body.createTextRange(),
    range2 = document.body.createTextRange();
range1.findText("Hello world!");
range2.findText("Hello");
alert(range1.compareEndPoints("StartToStart",range2)); //0
alert(range1.compareEndPoints("EndToEnd",range2)); //1
```

`IE` 中还有两个方法，也用于比较范围：`isEqual()` 方法用于确定两个范围是否相等，`inRange()` 方法用于确定一个范围是否包含另一个范围。例如：

``` javascript
var range1 = document.body.createTextRange(),
    range2 = document.body.createTextRange();
range1.findText("Hello world!");
range2.findText("Hello");
alert("range1.isEqual(range2):"+range1.isEqual(range2)); //false
alert("range1.inRange(range2):"+range1.inRange(range2)); //true
```
### 6、复制 IE 范围

在 `IE` 中使用 `duplicate()` 方法可以复制文本范围，结果会创建原范围的一个副本，例如：

``` javascript
var newRange = range.duplicate();
```

新创建的范围会带有与原范围完全相同的属性。
<br/>
<g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji><g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji><g-emoji alias="yum" fallback-src="https://assets-cdn.github.com/images/icons/emoji/unicode/1f60b.png" ios-version="6.0">😋</g-emoji> 好了，基本知识就是这些，在这里和大家说拜拜啦~

[查看demo](http://yixunfe.github.io/blog/demo/37/demo.html)