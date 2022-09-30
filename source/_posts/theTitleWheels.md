---
title: 一个浮动的标题
excerpt: 做项目遇到的一个需求
date: 2022-09-30
categories:
- 技术文章
tags:
- js
- css
- 轮子
- 浏览器
- debug
---

## 前言
前天开会，设计项目需求时遇到的一个情况，标题要浮动，但是简单的浮动会导致界面缩小右端看不见，对于此设计的解决方法，如果各位大佬有更好的思路欢迎联系我进行修改添加。

## css部分
当一个页面下滑，标题不动，此时就需要flex，把标题从布局中“浮”起来，如下：

```html
<style>
.headerBackground {
    background: rgb(221, 221, 221);
    width: 1240px;
    height: 60px;
    margin: auto;
    left: 0;
    right: 0;
    position: fixed;
}
</style>
```

ok，完成了，下一步要求是随着页面下滑背景从透明变不透明，字一直不透明，这里我想的是分两层，一层是上边这个作为底，引入**opacity**并将**z-index**设置成一个小于字的值，所以现在是三层，第一层整体页面，第二层背景，第三层logo和文字，改法如下：

```html
<style>
.headerBackground {
    background: rgb(221, 221, 221);
    width: 1240px;
    height: 60px;
    margin: auto;
    left: 0;
    right: 0;
    position: fixed;
    opacity: 0;
    z-index: 10;
}
.header {
    width: 1240px;
    height: 60px;
    margin: auto;
    left: 0;
    right: 0;
    position: fixed;
    display: grid;
    grid: auto/30% 70%;
    z-index: 20;
    justify-items: end;
    align-items: center;
}
</style>
<body>
<div id="header" class="header">
    <img src="" class="headerLeft">
    <div class="headerRight">
        <button>标签1</button>
        <button>标签2</button>
        <button>标签3</button>
        <button>标签4</button>
        <button>标签5</button>
    </div>
</div>
</body>
```

这里用了gird布局，关于grid我以后会专门写一篇文章，现在只需要知道这么用就行。

## js部分
css以上即可，接下来是js的部分，我们现在要随滚动轴越靠下第二层的颜色越深，从要求出发，我们需要拿到y轴坐标和第二层的透明度标签，用y轴坐标除以一个数返回作为**opacity**的返回值，这里除以1000：

```html
<script>
function BodyScroll() {
    var s = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
    $('.headerBackground').css(
        'opacity',
        function() {
            return s / 1000;
        }
    );
}
</script>
```

ok，透明度实现完成，但是现在出现了一个问题，我窗口缩小，拖动横轴，右边的内容永远出不来，这是因为他被“浮”起来了，如果我们想让它根据横轴的变化而变化，我们需要拿到当前窗口x轴坐标，然后拿到logo和文字这一图层的定位坐标，将坐标减去x轴坐标，就能实现按滚动轴移动而移动：

```html
<script>
function BodyScroll() {
    var s = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
    var flexs = window.scrollX;
    $('.headerBackground').css(
        'opacity',
        function() {
            return s / 1000;
        }
    );
    $('.header').css(
        'left',
        function() {
            return -flexs;
        }
    );
}
</script>
```

ok，bug也成功修复~