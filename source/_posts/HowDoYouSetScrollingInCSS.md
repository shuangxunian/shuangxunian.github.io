---
title: css中怎么设置滚动？
excerpt: 如题目
categories:
- 技术文章
tags:
- css
---

## 前言
在页面中，常常要将一些内容以滚动的形式来展示，那么我们该如何设置滚动呢？下面我们来看一下css设置网页内容滚动的方法。

## 内容
css可以通过为网页元素设置滚动条样式使网页元素的内容实现滚动。下面我们来看一下css设置滚动条实现滚动的方法。

css通过overflow属性设置滚动条示例：
```html
<html>
    <head>
        <style type="text/css">
            div {
                background-color:#00FFFF;
                width:150px;
                height:150px;
                overflow: scroll;
            }
        </style>
    </head>

    <body>
        <p>如果元素中的内容超出了给定的宽度和高度属性，overflow 属性可以确定是否显示滚动条等行为。</p>

        <div>
            这个属性定义溢出元素内容区的内容会如何处理。如果值为 scroll，不论是否需要，
            用户代理都会提供一种滚动机制。因此，有可能即使元素框中可以放下所有内容也会出现滚动条。默认值是 visible。
        </div>
    </body>
</html>
```

## overflow属性介绍

overflow 属性规定当内容溢出元素框时发生的事情。

说明

这个属性定义溢出元素内容区的内容会如何处理。如果值为 scroll，不论是否需要，用户代理都会提供一种滚动机制。因此，有可能即使元素框中可以放下所有内容也会出现滚动条。

属性值：
- visible 默认值。内容不会被修剪，会呈现在元素框之外。
- hidden 内容会被修剪，并且其余内容是不可见的。
- scroll 内容会被修剪，但是浏览器会显示滚动条以便查看其余的内容。
- auto 如果内容被修剪，则浏览器会显示滚动条以便查看其余的内容。
- inherit 规定应该从父元素继承 overflow 属性的值。

## 参考链接
[css中怎么设置滚动？](https://www.html.cn/qa/css3/13166.html)