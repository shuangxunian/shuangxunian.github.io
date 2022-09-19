---
title: 3种方法实现CSS隐藏滚动条并可以滚动内容
excerpt: 如题目
categories:
- 技术文章
tags:
- css
---

## 前言
隐藏滚动条的同时还需要支持滚动，我们经常在前端开发中遇到这种情况，最容易想到的是加一个iscroll插件，但其实现在CSS也可以实现这个功能，我已经在很多地方使用了，下面一起看看这三种方法。

## 方法1：计算滚动条宽度并隐藏起来
把滚动条通过定位把它隐藏了起来。 [演示0] (http://caibaojian.com/demo/2018/3/scroll.html) 下面给一个简化版的代码
```html
<div class="outer-container">
    <div class="inner-container">
    	......
    </div>
</div>
```
```css
.outer-container{
	width: 360px;
	height: 200px;
	position: relative;
	overflow: hidden;
}
.inner-container{
	position: absolute;
	left: 0;
	top: 0;
	right: -17px;
	bottom: 0;
	overflow-x: hidden;
	overflow-y: scroll;
}
```
[演示1](http://caibaojian.com/demo/2018/3/scroll2.html) 这个代码巧妙的向右移动了17个像素，刚好等于滚动条的宽度。这个值是我手动调试得来的。在chrome和IE没发现问题。

## 方法2：使用三个容器包围起来，不需要计算滚动条的宽度
该代码最早是在[Microsoft](https://docs.microsoft.com/zh-cn/archive/blogs/kurlak/hiding-vertical-scrollbars-with-pure-css-in-chrome-ie-6-firefox-opera-and-safari) 博客上看到的，跟我上面的思路差不多，只不过人家里面又加多了一个盒子，将内容限制在盒子里面了。这样子就看不到滚动条同时也可以滚动。 代码如下：
```html
<div class="outer-container">
    <div class="inner-container">
        <div class="content">
            ......
        </div>
    </div>
</div>
```
```css
.element, .outer-container {
  width: 200px;
  height: 200px;
}

.outer-container {
  border: 5px solid purple;
  position: relative;
  overflow: hidden;
}

.inner-container {
  position: absolute;
  left: 0;
  overflow-x: hidden;
  overflow-y: scroll;
}

.inner-container::-webkit-scrollbar {
  display: none;
}
```
[演示2](http://caibaojian.com/demo/2018/3/scroll3.html)

## 方法3：css隐藏滚动条
同时该文章还分享了一种通过CSS隐藏滚动条的方法，不过这个方法不兼容IE，做移动端的可以使用。 那就是自定义滚动条的伪对象选择器 [::-webkit-scrollbar]()，[详情请看之前的文章：CSS3自定义webkit滚动条样式]()
**chrome 和Safari**
```css
.element::-webkit-scrollbar { width: 0 !important }
```
**IE 10+**
```css
.element { -ms-overflow-style: none; }
```
**Firefox**
```css
.element { overflow: -moz-scrollbars-none; }
```
[演示3](http://caibaojian.com/demo/2018/3/scroll4.html)

## 参考链接
[3种方法实现CSS隐藏滚动条并可以滚动内容](http://caibaojian.com/hide-scrollbar.html)