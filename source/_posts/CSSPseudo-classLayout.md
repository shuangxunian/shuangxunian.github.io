---
title: 伪类和伪元素
excerpt: 本文介绍什么是伪类、什么是伪元素以及几个常用的css伪类。
categories:
- 技术文章
tags:
- html
- css
- 面试
---

## 前言
css 伪类是用于向某些选择器添加特殊的效果，是动态的，指当前元素所处的状态或者特性。只有一个元素达到一个特定状态时，它可能得到一个伪类的样式；当状态改变时，它又会失去这个样式。

## 伪类与伪元素
### 介绍
伪元素由双冒号和伪元素名称组成。双冒号是在当前规范中引入的，用于区分伪类和伪元素。但是伪类兼容现存样式，浏览器需要同时支持旧的伪类，比如:first-line、:first-letter、:before、:after等。

CSS 伪类用于向某些选择器添加特殊的效果；CSS 伪元素用于将特殊的效果添加到某些选择器。以明确两点，第一两者都与选择器相关，第二就是添加一些“特殊”的效果。这里特殊指的是两者描述了其他 css 无法描述的东西。伪类可以独立于文档的元素来分配样式，且可以分配给任何元素，逻辑上和功能上类类似，但是其是预定义的、不存在于文档树中且表达方式也不同，所以叫伪类。伪元素所控制的内容和一个元素控制的内容一样，但是伪元素不存在于文档树中，不是真正的元素，所以叫伪元素。

**伪类有：**:first-child ，:link:，vistited，:hover:，active:focus，:lang，:not(s)，：root等...

**伪元素有：**:first-line，:first-letter，:before，:after，:placeholder,:selection

**注：**如果你的网站只需要兼容webkit、firefox、opera等浏览器，建议对于伪元素采用双冒号的写法，如果不得不兼容IE浏览器，还是用CSS2的单冒号写法比较安全。如果自己不确定的话可以针对某些需要兼容的属性有两种属性。

### 区别
**伪类**
伪类选择元素基于的是当前元素处于的状态，或者说元素当前所具有的特性，而不是元素的id、class、属性等静态的标志。由于状态是动态变化的，所以一个元素达到一个特定状态时，它可能得到一个伪类的样式；当状态改变时，它又会失去这个样式。由此可以看出，它的功能和class有些类似，但它是基于文档之外的抽象，所以叫伪类。

**伪元素**
与伪类针对特殊状态的元素不同的是，伪元素是对元素中的特定内容进行操作，它所操作的层次比伪类更深了一层，也因此它的动态性比伪类要低得多。实际上，设计伪元素的目的就是去选取诸如元素内容第一个字（母）、第一行，选取某些内容前面或后面这种普通的选择器无法完成的工作。它控制的内容实际上和元素是相同的，但是它本身只是基于元素的抽象，并不存在于文档中，所以叫伪元素。


## 实用的伪类
### ::first-line | 选择文本的第一行
::first-line 伪元素在某块级元素的第一行应用样式。第一行的长度取决于很多因素，包括元素宽度，文档宽度和文本的文字大小。

::first-line 伪元素只能在块容器中,所以,::first-line伪元素只能在一个display值为block, inline-block, table-cell 或者 table-caption中有用。在其他的类型中，::first-line 是不起作用的。

用法如下：

```css
p::first-line {
  color: lightcoral;
}
```

### ::first-letter | 选择这一行的第一字
CSS 伪元素 ::first-letter会选中某块级元素第一行的第一个字母。用法如下：

```css
p::first-letter{
    color: red;
    font-size: 2em;
}
```

### ::selection| 被用户高亮的部分
::selection 伪元素应用于文档中被用户高亮的部分（比如使用鼠标或其他选择设备选中的部分）。

```css
div::selection {
    color: #409EFF;
}
```

### :root | 根元素
:root 伪类匹配文档树的根元素。对于 HTML 来说，:root 表示 <html>元素，除了优先级更高之外，与 html 选择器相同。

在声明全局 CSS 变量时 :root 会很有用：

```css
:root {
    --main-color: hotpink;
    --pane-padding: 5px 42px;
}
```

### :empty | 仅当子项为空时才有作用
:empty 伪类代表没有子元素的元素。子元素只可以是元素节点或文本（包括空格）,注释或处理指令都不会产生影响。

```css
div:empty {
    border: 2px solid orange;
    margin-bottom: 10px;
}
```

### :only-child | 只有一个子元素才有作用
:only-child 匹配没有任何兄弟元素的元素.等效的选择器还可以写成 :first-child:last-child或者:nth-child(1):nth-last-child(1),当然,前者的权重会低一点。

```html
<style>
p:only-child{
    background: #409EFF;
}
</style>
<div>
    <p>第一个没有任何兄弟元素的元素</p>
</div>
<div>
    <p>第二个</p>
    <p>第二个</p>
</div>
```

### :first-of-type | 选择指定类型的第一个子元素
:first-of-type表示一组兄弟元素中其类型的第一个元素。

```css
.innerDiv p:first-of-type {
    color: orangered;
}
```

上面表示将 .innerDiv 内的第一个元素为 p 的颜色设置为橘色。

### :last-of-type | 选择指定类型的最后一个子元素
:last-of-type CSS 伪类 表示了在（它父元素的）子元素列表中，最后一个给定类型的元素。当代码类似Parent tagName:last-of-type的作用区域包含父元素的所有子元素中的最后一个选定元素，也包括子元素的最后一个子元素并以此类推。

```css
.innerDiv p:last-of-type {
    color: orangered;
}
```

上面表示将 .innerDiv 内的的最后一个元素为 p 的颜色设置为橘色。

### :nth-of-type() | 选择指定类型的子元素
:nth-of-type() 这个 CSS 伪类是针对具有一组兄弟节点的标签, 用 n 来筛选出在一组兄弟节点的位置。**注：**从1开始。

```css
.innerDiv p:nth-of-type(1) {
    color: orangered;
}
```

### :nth-last-of-type() | 在列表末尾选择类型的子元素
:nth-last-of-type(an+b) 这个 CSS 伪类 匹配那些在它之后有 an+b-1 个相同类型兄弟节点的元素，其中 n 为正值或零值。它基本上和 :nth-of-type 一样，只是它从结尾处反序计数，而不是从开头处。

```css
.innerDiv p:nth-last-of-type(1) {
    color: orangered;
}
```

这会选择innerDiv元素中包含的类型为p元素的列表中的最后一个子元素。

### :link | 选择一个未访问的超链接
:link伪类选择器是用来选中元素当中的链接。它将会选中所有尚未访问的链接，包括那些已经给定了其他伪类选择器的链接（例如:hover选择器，:active选择器，:visited选择器）。

为了可以正确地渲染链接元素的样式，:link伪类选择器应当放在其他伪类选择器的前面，并且遵循LVHA的先后顺序，即：:link — :visited — :hover — :active。:focus伪类选择器常伴随在:hover伪类选择器左右，需要根据你想要实现的效果确定它们的顺序。

```css
a:link {
    color: orangered;
}
```

### :checked | 选择一个选中的复选框
:checked CSS 伪类选择器表示任何处于选中状态的radio, checkbox  或("select") 元素中的option HTML元素("option")。

```css
input:checked {
  box-shadow: 0 0 0 3px hotpink;
}
```

### :valid | 选择一个有效的元素
:valid CSS 伪类表示内容验证正确的 &lt;input&gt; 或其他 &lt;form&gt; 元素。这能简单地将校验字段展示为一种能让用户辨别出其输入数据的正确性的样式。

```css
input:valid {
  box-shadow: 0 0 0 3px hotpink;
}
```

### :invalid | 选择一个无效的元素
:invalid CSS 伪类 表示任意内容未通过验证的 &lt;input&gt; 或其他 &lt;form&gt; 元素。

```css
input[type="text"]:invalid {
    border-color: red;
}
```

### :lang() | 通过指定的lang值选择一个元素
:lang() CSS 伪类基于元素语言来匹配页面元素。

```css
/* 选取任意的英文（en)段落 */
p:lang(en) {
  quotes: '\201C' '\201D' '\2018' '\2019';
}
```

### :not() | 用来匹配不符合一组选择器的元素
CSS 伪类 :not() 用来匹配不符合一组选择器的元素。由于它的作用是防止特定的元素被选中，它也被称为反选伪类（negation pseudo-class）。

来看一个例子：

```html
<style>
.innerDiv :not(p) {
    color: lightcoral;
}
</style>
<div class="innerDiv">
    <p>Paragraph 1</p>
    <p>Paragraph 2</p>
    <div>Div 1</div>
    <p>Paragraph 3</p>
    <div>Div 2</div>
</div>
```

Div 1 和 Div 2会被选中，p 不会被选中。

## 参考链接
[这 16 个 CSS 伪类，助你提升布局效率！](https://zhuanlan.zhihu.com/p/250062861)
[CSS中一个冒号和两个冒号有什么区别](https://www.cnblogs.com/libin-1/p/6440454.html)