---
title: 去除li前面的小黑点,和ulli部分属性方法
excerpt: 如题目
categories:
- 技术文章
tags:
- css
---

## 前言
对于很多人用div来做网站时，总会用到，但在显示效果时前面总是会有一个小黑点，这个令很多人头痛，但又找不到要源，其它我们可以用以下方法来清除。
1. 在CSS中写入代码。找到相关性的CSS，在.li和.ul下写入`list-sytle:none;`当然有的会这样来写`list-style-type:none`, 这种写法特别是在一些CMS中最常见。
2. 在相关的页面找到head部分写入下面的代码
    ```html
    <style type="text/css">
        list-style:none;
    </style>
    ```
3. 在li,ul内加入list-style。如`<ul style="list-style-type:none"><li></li></ul>` 当然这种是很麻烦的了。

最简单的就是第一种了，通过CSS来控制，这个当然会有不错的效果了。
这几种方法都是通过设置`list-style:none`来设置的，有的是会用`list-style-type`，下面我们来看一看它的属性。

## 属性
这些都可以来代替上文中的none，想要什么样的都会有一个相应的对应。
- none不使用项目符号
- disc实心圆，默认值
- circle空心圆
- square实心方块
- decimal阿拉伯数字
- lower-roman小写罗马数字
- upper-roman大写罗马数字
- lower-alpha小写英文字母
- upper-alpha大写英文字母

## eg
1. 运用CSS格式化列表符：
    ```css
    ul li{
    list-style-type:none;
    }
    ```
2. 如果你想将列表符换成图像，则：
    ```css
    ul li{
        list-style-type:none;
        list-style-image: url(images/icon.gif);
    }
    ```
3. 为了左对齐，可以用如下代码：
    ```css
    ul{
        list-style-type:none;
        margin:0px;
    }
    ```
4. 如果想给列表加背景色，可以用如下代码：
    ```css
    ul{
        list-style-type: none;
        margin:0px;
    }
    ul li{
    background:#CCC;
    }
    ```
5. 如果想给列表加MOUSEOVER背景变色效果，可以用如下代码：
    ```css
    ul{ list-style-type: none; margin:0px; }
    ul li a{ display:block; width: 100%; background:#ccc; }
    ul li a:hover{ background:#999; }说明：display:block;这一行必须要加的，这样才能块状显示！
    ```
6. li中的元素水平排列,关键`float:left`：
    ```css
    ul{
        list-style-type:none;
        width:100%;
    }
    ```

## 参考链接
[Html中CSS之去除li前面的小黑点,和ul、LI部分属性方法](https://blog.csdn.net/business122/article/details/7973638)

