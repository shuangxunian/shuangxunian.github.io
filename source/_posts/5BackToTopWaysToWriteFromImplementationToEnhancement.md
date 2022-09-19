---
title: 5种回到顶部的写法从实现到增强
excerpt: 做项目遇到的一个要求，收集到了五种解决方案
categories:
- 技术文章
tags:
- js
- css
- 轮子
- 浏览器
- debug
---

## 实现
### 锚点
使用锚点链接是一种简单的返回顶部的功能实现。该实现主要在页面顶部放置一个指定名称的锚点链接，然后在页面下方放置一个返回到该锚点的链接，用户点击该链接即可返回到该锚点所在的顶部位置。

```html
<body style="height:2000px;">
    <div id="topAnchor"></div>
    <a href="#topAnchor" style="position:fixed;right:0;bottom:0">回到顶部</a>
</body>
```

### scrollTop
scrollTop属性表示被隐藏在内容区域上方的像素数。元素未滚动时，scrollTop的值为0，如果元素被垂直滚动了，scrollTop的值大于0，且表示元素上方不可见内容的像素宽度。

由于scrollTop是可写的，可以利用scrollTop来实现回到顶部的功能。

```html
<body style="height:2000px;">
    <button id="test" style="position:fixed;right:0;bottom:0">回到顶部</button>
    <script>
        test.onclick = function(){
            document.body.scrollTop = document.documentElement.scrollTop = 0;
        }
    </script>
</body>
```

### scrollTo()
scrollTo(x,y)方法滚动当前window中显示的文档，让文档中由坐标x和y指定的点位于显示区域的左上角。

设置scrollTo(0,0)可以实现回到顶部的效果。

```html
<body style="height:2000px;">
    <button id="test" style="position:fixed;right:0;bottom:0">回到顶部</button>
    <script>
        test.onclick = function(){
            scrollTo(0,0);
        }
    </script>
</body>
```

### scrollBy()
scrollBy(x,y)方法滚动当前window中显示的文档，x和y指定滚动的相对量。

只要把当前页面的滚动长度作为参数，逆向滚动，则可以实现回到顶部的效果。

```html
<body style="height:2000px;">
    <button id="test" style="position:fixed;right:0;bottom:0">回到顶部</button>
    <script>
        test.onclick = function(){
            var top = document.body.scrollTop || document.documentElement.scrollTop
            scrollBy(0,-top);
        }
    </script>
</body>
```

### scrollIntoView()
Element.scrollIntoView方法滚动当前元素，进入浏览器的可见区域。

该方法可以接受一个布尔值作为参数。如果为true，表示元素的顶部与当前区域的可见部分的顶部对齐（前提是当前区域可滚动）；如果为false，表示元素的底部与当前区域的可见部分的尾部对齐（前提是当前区域可滚动）。如果没有提供该参数，默认为true。

使用该方法的原理与使用锚点的原理类似，在页面最上方设置目标元素，当页面滚动时，目标元素被滚动到页面区域以外，点击回到顶部按钮，使目标元素重新回到原来位置，则达到预期效果。

```html
<body style="height:2000px;">
    <div id="target"></div>
    <button id="test" style="position:fixed;right:0;bottom:0">回到顶部</button>
    <script>
        test.onclick = function(){
            target.scrollIntoView();
        }
    </script>
</body>
```

## 优化
### 显示增强
使用CSS画图，将“回到顶部”变成可视化的图形(如果兼容IE8-浏览器，则用图片代替)。

使用CSS伪元素及伪类hover效果，当鼠标移动到该元素上时，显示回到顶部的文字，移出时不显示。

```html
<style>
.box{
    position:fixed;
    right:10px;
    bottom: 10px;
    height:30px;
    width: 50px;    
    text-align:center;
    padding-top:20px;    
    background-color: lightblue;
    border-radius: 20%;
    overflow: hidden;
}
.box:hover:before{
    top:50%
}
.box:hover .box-in{
    visibility: hidden;
}
.box:before{
    position: absolute;
    top: -50%;
    left: 50%;
    transform: translate(-50%,-50%);
    content:'回到顶部';
    width: 40px;
    color:peru;
    font-weight:bold;

}    
.box-in{
    visibility: visible;
    display:inline-block;
    height:20px;
    width: 20px;
    border: 3px solid black;
    border-color: white transparent transparent white;
    transform:rotate(45deg);
}
</style>

<body style="height:2000px;">
<div id="box" class="box">
    <div class="box-in"></div>
</div>    
</body>
```

### 动画增强
为回到顶部增加动画效果，滚动条以一定的速度回滚到顶部。

动画有两种：一种是CSS动画，需要有样式变化配合transition；一种是javascript动画，使用定时器来实现。

在上面的5种实现中，scrollTop、scrollTo()和scrollBy()方法可以增加动画，且由于无样式变化，只能增加javascript动画。

定时器又有setInterval、setTimeout和requestAnimationFrame这三种可以使用，下面使用性能最好的定时器requestAnimationFrame来实现。

[注意]IE9-浏览器不支持该方法，可以使用setTimeout来兼容。

#### 增加scrollTop的动画效果
使用定时器，将scrollTop的值每次减少50，直到减少到0，则动画完毕

```html
<script>
var timer  = null;
box.onclick = function(){
    cancelAnimationFrame(timer);
    timer = requestAnimationFrame(function fn(){
        var oTop = document.body.scrollTop || document.documentElement.scrollTop;
        if(oTop > 0){
            document.body.scrollTop = document.documentElement.scrollTop = oTop - 50;
            timer = requestAnimationFrame(fn);
        }else{
            cancelAnimationFrame(timer);
        }    
    });
}
</script>
```

#### 时间版运动
但是，上面的代码有一个问题，就是当页面内容较多时，回到顶部的动画效果将持续很长时间。因此，使用时间版的运动更为合适，假设回到顶部的动画效果共运动500ms，则代码如下所示：

```html
<body style="height: 2000px;">
<button id="test" style="position:fixed;right:10px;bottom:10px;">回到顶部</button>
<script>
var timer  = null;
test.onclick = function(){
    cancelAnimationFrame(timer);
    //获取当前毫秒数
    var startTime = +new Date();     
    //获取当前页面的滚动高度
    var b = document.body.scrollTop || document.documentElement.scrollTop;
    var d = 500;
    var c = b;
    timer = requestAnimationFrame(function func(){
        var t = d - Math.max(0,startTime - (+new Date()) + d);
        document.documentElement.scrollTop = document.body.scrollTop = t * (-c) / d + b;
        timer = requestAnimationFrame(func);
        if(t == d){
          cancelAnimationFrame(timer);
        }
    });
}
</script>
</body>
```

#### 增加scrollTo()动画效果
将scrollTo(x,y)中的y参数通过scrollTop值获取，每次减少50，直到减少到0，则动画完毕。

```html
<script>
var timer  = null;
box.onclick = function(){
    cancelAnimationFrame(timer);
    timer = requestAnimationFrame(function fn(){
        var oTop = document.body.scrollTop || document.documentElement.scrollTop;
        if(oTop > 0){
            scrollTo(0,oTop-50);
            timer = requestAnimationFrame(fn);
        }else{
            cancelAnimationFrame(timer);
        }    
    });
}
</script>
```

#### 增加scrollBy()动画效果
将scrollBy(x,y)中的y参数设置为-50，直到scrollTop为0，则回滚停止。

```html
<script>
var timer  = null;
box.onclick = function(){
    cancelAnimationFrame(timer);
    timer = requestAnimationFrame(function fn(){
        var oTop = document.body.scrollTop || document.documentElement.scrollTop;
        if(oTop > 0){
            scrollBy(0,-50);
            timer = requestAnimationFrame(fn);
        }else{
            cancelAnimationFrame(timer);
        }    
    });
}
</script>
```

## 实现
### 实现1
如果项目经理要求不高的话（就像我们的），用下面的就行了，图片标签去阿里矢量库找就行，页面在最上端时隐藏回到顶部标签：

```html
<style>
/*返回顶部*/
.back {
    position: fixed;
    right: 20px;
    bottom: 20px;
    color: red;
}
.hide {
    display: none;
}
</style>
<body onscroll="BodyScroll();">
    <div id="back" class="back hide" onclick="BackTop();"><img src="img/top.png" alt="回到顶部" width="35px"></div>
    <script src="js/jquery-3.5.1.min.js"></script>
    <script src="js/index.js"></script>
</body>
<script>
function BackTop() {
    document.documentElement.scrollTop = window.pageYOffset = document.body.scrollTop = 0;
}
function BodyScroll() {
    var s = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
    var t = document.getElementById('back');
    if (s >= 100) {
        t.classList.remove('hide');
    } else {
        t.classList.add('hide');
    }
}
</script>
```

### 实现2
如果遇到事多的，就用这种以最常用的scrollTop属性实现动画增强效果，时间取500ms，如果觉得500ms的时间不合适，可以根据实际情况进行调整：

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Document</title>
<style>
.box{
    position:fixed;
    right:10px;
    bottom: 10px;
    height:30px;
    width: 50px;    
    text-align:center;
    padding-top:20px;    
    background-color: lightblue;
    border-radius: 20%;
    overflow: hidden;
}
.box:hover:before{
    top:50%
}
.box:hover .box-in{
    visibility: hidden;
}
.box:before{
    position: absolute;
    top: -50%;
    left: 50%;
    transform: translate(-50%,-50%);
    content:'回到顶部';
    width: 40px;
    color:peru;
    font-weight:bold;

}    
.box-in{
    visibility: visible;
    display:inline-block;
    height:20px;
    width: 20px;
    border: 3px solid black;
    border-color: white transparent transparent white;
    transform:rotate(45deg);
}
</style>
</head>
<body style="height:2000px;">
<div id="box" class="box">
    <div class="box-in"></div>
</div>    
<script>
var timer  = null;
box.onclick = function(){
    cancelAnimationFrame(timer);
    //获取当前毫秒数
    var startTime = +new Date();     
    //获取当前页面的滚动高度
    var b = document.body.scrollTop || document.documentElement.scrollTop;
    var d = 500;
    var c = b;
    timer = requestAnimationFrame(function func(){
        var t = d - Math.max(0,startTime - (+new Date()) + d);
        document.documentElement.scrollTop = document.body.scrollTop = t * (-c) / d + b;
        timer = requestAnimationFrame(func);
        if(t == d){
          cancelAnimationFrame(timer);
        }
    });
}
</script>
</body>
</html>
```

## 参考链接
老男孩
[5种回到顶部的写法从实现到增强](https://www.cnblogs.com/xiaohuochai/p/5836179.html)