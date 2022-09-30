---
title: 行内元素、块级元素和行内块级元素的转换
excerpt: display:inline、display:block、display:inline-block三者真实用途及含义是什么
date: 2022-09-30
categories:
- 技术文章
tags:
- html
- css
- 面试
---

## 前言
关于什么是行内元素、块级元素、行内块级元素，[请移步](https://shuangxunian.gitee.io/2020/08/25/%E8%A1%8C%E5%86%85%E5%85%83%E7%B4%A0/)。

## display:block
display:block就是将元素显示为块级元素。

block元素的特点是：
- 总是在新行上开始；
- 高度，行高以及顶和底边距都可控制；
- 宽度是它的容器的100%，除非设定一个宽度；
- &lt;div&gt;, &lt;p&gt;, &lt;h1&gt;, &lt;form&gt;, &lt;ul&gt; 和 &lt;li&gt;是块元素的例子。

## display:inline
display:inline就是将元素显示为内联行内元素。

inline元素的特点是：
- 和其他元素都在一行上；
- 高，行高及顶和底边距不可改变；
- 宽度就是它的文字或图片的宽度，不可改变。
- &lt;span&gt;, &lt;a&gt;, &lt;label&gt;, &lt;input&gt;, &lt;img&gt;, &lt;strong&gt; 和&lt;em&gt;是inline元素的例子。

## display:inline-block
display:inline-block将对象呈递为内联块级对象。

display:inline-block将对象呈递为内联对象，但是对象的内容作为块对象呈递。旁边的内联对象会被呈递在同一行内，允许空格。

inline-block的元素特点：
将对象呈递为内联对象，但是对象的内容作为块对象呈递。旁边的内联对象会被呈递在同一行内，允许空格。(准确地说，应用此特性的元素呈现为内联对象，周围元素保持在同一行，但可以设置宽度和高度地块元素的属性)

## code
举个栗子：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>display:inline、block、inline-block的区别</title>
</head>
<style>
    div,
    span {
        background-color: green;
        margin: 5px;
        border: 1px solid #333;
        padding: 5px;
        height: 52px;
        color: #fff;
    }
    
    .b {
        display: block;
    }
    
    .i {
        display: inline;
    }
    
    div.ib {
        display: inline-block;
    }
    
    div.ib {
        display: inline;
    }
    
    span,
    a.ib {
        display: block;
    }
</style>

<body>
    <div>div display:block</div>
    <div class="i">div display:inline</div>
    <div class="ib">div display:inline-block</div>
    <span>span display:inline</span>
    <span class="b">span display:block</span>
    <span><a class="ib" href="#">a display:block</a></span><br />
</body>

</html>
```

如图：

![](https://api2.mubu.com/v3/document_image/be50f56e-3606-4317-a7e5-1c9ac9c16048-3807603.jpg)

## 参考链接
[display:inline、display:block、display:inline-block三者真实用途及含义是什么？](https://blog.csdn.net/sinat_34719507/article/details/53512509)