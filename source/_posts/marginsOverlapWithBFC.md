---
title: 边距重叠与BFC
excerpt: 本文介绍盒模型，JS设置盒模型的宽高，BFC
categories:
- 技术文章
tags:
- css
- 面试
---

## 盒模型
### 基本概念
什么是 CSS 盒模型？相信大部分人都能答出这个问题来，那就是 标准模型 + IE 模型

标准模型：

![](https://api2.mubu.com/v3/document_image/1d4bd78e-362f-4a18-a846-0d2f5b07839e-3807603.jpg)

IE 模型：

![](https://api2.mubu.com/v3/document_image/365e4100-dbea-4012-9243-b19a1acec70f-3807603.jpg)

**很明显：**
- 在 标准盒子模型中，width 和 height 指的是内容区域的宽度和高度。增加内边距、边框和外边距不会影响内容区域的尺寸，但是会增加元素框的总尺寸。
- IE 盒子模型中，width 和 height 指的是content+border+padding

### CSS 如何设置这两种模型
- 标准模型：box-sizing: content-box;
- IE 模型：box-sizing: border-box;

### JS 如何设置盒模型对应的宽和高
- dom.style.width/height : 只能取出内联样式的宽和高 eg: &lt;div id="aa" style="width: 200px"&gt;&lt;/div&gt;
- dom.currentStyle.width/height 获取即时计算的样式，但是只有 IE 支持，要想支持其他浏览器，可以通过下面的方式
- window.getComputedStyle(dom).width: 兼容性更好
- dom.getBoundingClientRect().width/height: 这个较少用，主要是要来计算在页面中的绝对位置

## 边距重叠
什么是边距重叠呢?

边界重叠是指两个或多个盒子(可能相邻也可能嵌套)的相邻边界(其间没有任何非空内容、补白、边框)重合在一起而形成一个单一边界。

### 父子元素的边界重叠

```html
<style>
.parent {
    background: #e7a1c5;
}
.parent .child {
    background: #c8cdf5;
    height: 100px;
    margin-top: 10px;
}
</style>
<section class="parent">
    <article class="child"></article>
</section>
```

以为的效果：

![](https://api2.mubu.com/v3/document_image/5eb8ffc0-01e5-4424-9682-820079038ad4-3807603.jpg)

实际的效果：

![](https://api2.mubu.com/v3/document_image/90c9d35d-94e7-4ccb-bf55-b006ba9e3e29-3807603.jpg)

在这里父元素的高度不是 110px，而是 100px，在这里发生了高度坍塌。

原因是如果块元素的 margin-top 与它的第一个子元素的 margin-top 之间没有 border、padding、inline content、 clearance 来分隔，或者块元素的 margin-bottom 与它的最后一个子元素的 margin-bottom 之间没有 border、padding、inline content、height、min-height、 max-height 分隔，那么外边距会塌陷。子元素多余的外边距会被父元素的外边距截断。

### 兄弟元素的边界重叠

```html
<style>
#margin {
    background: #e7a1c5;
    overflow: hidden;
    width: 300px;
}
#margin > p {
    background: #c8cdf5;
    margin: 20px auto 30px;
}
</style>
<section id="margin">
    <p>1</p>
    <p>2</p>
    <p>3</p>
</section>
```

![](https://api2.mubu.com/v3/document_image/bbac0b7f-6fce-4605-945a-afae0ad53243-3807603.jpg)

可以看到 1 和 2,2 和 3 之间的间距不是 50px，发生了边距重叠是取了它们之间的最大值 30px。

### 空元素的边界重叠
假设有一个空元素，它有外边距，但是没有边框或填充。在这种情况下，上外边距与下外边距就碰到了一起，它们会发生合并：

![](https://api2.mubu.com/v3/document_image/fbc81eb4-71e1-454a-8a62-1de06129fe1e-3807603.jpg)

## BFC
解决上述问题的其中一个办法就是创建 BFC。BFC 的全称为 Block Formatting Context，即块级格式化上下文。
- 处于同一个 BFC 中的元素相互影响，可能会发生 margin collapse；
- BFC 在页面上是一个独立的容器，容器里面的子元素不会影响到外面的元素，反之亦然；
- 计算 BFC 的高度时，考虑 BFC 所包含的所有元素，包括浮动元素也参与计算；
- 浮动盒的区域不会叠加到 BFC 上；

### 防止垂直 margin 重叠
父子元素的边界重叠得解决方案： 在父元素上加上 overflow:hidden;使其成为 BFC。

```css
.parent {
    background: #e7a1c5;
    overflow: hidden;
}
```

![](https://api2.mubu.com/v3/document_image/aa11bbd8-a9a7-4bf4-bd7b-9805fcc2c1f0-3807603.jpg)

兄弟元素的边界重叠，在第二个子元素创建一个 BFC 上下文：

```html
<section id="margin">
    <p>1</p>
    <div style="overflow:hidden;">
        <p>2</p>
    </div>
    <p>3</p>
</section>
```

![](https://api2.mubu.com/v3/document_image/5d84c3b8-a674-4068-8ed7-86f4c4d43b42-3807603.jpg)

### 清除内部浮动

```html
<style>
#float {
    background: #fec68b;
}
#float .float {
    float: left;
}
</style>
<section id="float">
    <div class="float">我是浮动元素</div>
</section>
```

父元素#float 的高度为 0，解决方案为为父元素#float 创建 BFC，这样浮动子元素的高度也会参与到父元素的高度计算：

```css
#float {
    background: #fec68b;
    overflow: hidden; /*这里也可以用float:left*/
}
```

![](https://api2.mubu.com/v3/document_image/55bdd47c-040e-445a-a6ad-ed3d5dc9fc05-3807603.jpg)

### 自适应两栏布局

```html
<style>
#layout {
    background: red;
}
#layout .left {
    float: left;
    width: 100px;
    height: 100px;
    background: pink;
}
#layout .right {
    height: 110px;
    background: #ccc;
}
</style>
<section id="layout">
    <!--左边宽度固定，右边自适应-->
    <div class="left">左</div>
    <div class="right">右</div>
</section>
```

在这里设置右边的高度高于左边，可以看到左边超出的部分跑到右边去了，这是由于由于浮动框不在文档的普通流中，所以文档的普通流中的块框表现得就像浮动框不存在一样导致的。

![](https://api2.mubu.com/v3/document_image/efa9dd1a-d0db-438c-9873-f259eadb650e-3807603.jpg)

解决方案为给右侧元素创建一个 BFC，原理是 BFC 不会与 float 元素发生重叠。

```css
#layout .right {
    height: 110px;
    background: #ccc;
    overflow: auto;
}
```

![](https://api2.mubu.com/v3/document_image/0505ebba-5f78-4d1f-8f88-cf6650b9fa33-3807603.jpg)

## 参考链接
[面试之CSS篇 - 边距重叠与BFC](https://juejin.im/post/6844903794321391623)