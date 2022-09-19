---
title: CSS 让同一行的图片和文字对齐
excerpt: 如题目
categories:
- 技术文章
tags:
- css
---

## 前言
大家在做前端开发的时候那，经常会遇到img标签和文字在同一行。

那么我刚开始的时候那是利用的float浮动布局解决的，定位布局（兼容性需要调整 不划算）下面给大家介绍一些其他的方法：

## 添加CSS属性
添加CSS属性：vertical-align:middle;
```html
<style>
a img{border:none} .testdiv *{ vertical-align:middle; }
</style>
<div class="testdiv">
	<a href="http://www.zc144.com/">
	<img src="http://www.zc144.com/download/Template.jpg" alt="这里是图片"/>
	</a>
	<span>这里是文字,看看文字对齐了没</span>
</div>
```

设置各对象的vertical-align属性， 属性说明 ： 
- baseline－将支持valign特性的对象的内容与基线对齐 
- sub－垂直对齐文本的下标 
- super－垂直对齐文本的上标 
- top－将支持valign特性的对象的内容与对象顶端对齐 
- text-top－将支持valign特性的对象的文本与对象顶端对齐 
- middle－将支持valign特性的对象的内容与对象中部对齐 
- bottom－将支持valign特性的对象的文本与对象底端对齐 
- text-bottom－将支持valign特性的对象的文本与对象顶端对齐 

## div嵌套
div嵌套：将图片和文字分别套上一个div，就可以利用 margin 熟悉任意定位了
```html

<style>
a img{border:none} .testIMG{ float:left; display:inline; margin-top:0; margin-left:5px; } .testTXT{ float:left; display:inline; margin-top:20; margin-left:5px; }
</style>
<div class="testdiv">
	<div class="testIMG">
		<a href="">
		<img src="Template.jpg" alt="这里是图片"/></a>
	</div>
	<div class="testTXT">
		<span>这里是文字,看看文字对齐了没</span>
	</div>
</div>
```

## 把图片作为背景：
如果你的图片只是用来作为小图标放在文字的左侧，那就推荐用这个方法，图片设置成文字的背景，不循环，定位在左侧上下居中，文字向左padding图片的宽度加几个像素。
```html

<style>
 a img{border:none} .testTXT{ height:60px; line-height:60px; padding-left:65px; background:url(http://www.zc144.com/download/Template.jpg) no-repeat left center }
</style>
<div class="testdiv">
	<div class="testTXT">
		<span>这里是文字,看看文字对齐了没</span>
	</div>
</div>
```

## 参考链接
[CSS 让同一行的图片和文字对齐](https://blog.csdn.net/sqc157400661/article/details/72457535)