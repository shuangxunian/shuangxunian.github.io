---
title: html & css基础题
excerpt: 一些基础，如果感觉答案有问题请联系我
date: 2022-09-30
categories:
- 技术文章
tags:
- css
- html
- 面试
---

## Doctype作用? 严格模式与混杂模式如何区分？它们有何意义?
Doctype声明于文档最前面，告诉浏览器以何种方式来渲染页面，这里有两种模式，严格模式和混杂模式。
- 严格模式的排版和 JS 运作模式是 以该浏览器支持的最高标准运行。
- 混杂模式，向后兼容，模拟老式浏览器，防止浏览器无法兼容页面。

## 分析比较 opacity: 0、visibility: hidden、display: none 优劣和适用场景
**结构：**
- display:none: 会让元素完全从渲染树中消失，渲染的时候不占据任何空间, 不能点击，
- visibility:hidden:不会让元素从渲染树消失，渲染元素继续占据空间，只是内容不可见，不能点击
opacity:0: 不会让元素从渲染树消失，渲染元素继续占据空间，只是内容不可见，可以点击

**继承：**
- display:none：是非继承属性，子孙节点消失由于元素从渲染树消失造成，通过修改子孙节点属性无法显示。
- visibility:hidden：是继承属性，子孙节点消失由于继承了hidden，通过设置visibility: visible;可以让子孙节点显式。

**性能：**
- display:none: 修改元素会造成文档回流,读屏器不会读取display: none元素内容，性能消耗较大
- visibility:hidden: 修改元素只会造成本元素的重绘,性能消耗较少读屏器读取visibility: hidden元素内容
- opacity:0 ： 修改元素会造成重绘，性能消耗较少

联系：它们都能让元素不可见

## 清除浮动的方式有哪些?比较好的是哪一种?
常用的一般为三种.clearfix, clear:both,overflow:hidden;
比较好是 .clearfix,伪元素万金油版本,后两者有局限性.
```css
.clearfix:after {
  visibility: hidden;
  display: block;
  font-size: 0;
  content: " ";
  clear: both;
  height: 0;
}

/*
为毛没有 zoom ,_height 这些,IE6,7这类需要 csshack 不再我们考虑之内了
.clearfix 还有另外一种写法,
*/

.clearfix:before, .clearfix:after {
    content:"";
    display:table;
}
.clearfix:after{
    clear:both;
    overflow:hidden;
}
.clearfix{
    zoom:1;
}

/*
用display:table 是为了避免外边距margin重叠导致的margin塌陷,
内部元素默认会成为 table-cell 单元格的形式
*/
```
clear:both:若是用在同一个容器内相邻元素上,那是贼好的,有时候在容器外就有些问题了, 比如相邻容器的包裹层元素塌陷
overflow:hidden:这种若是用在同个容器内,可以形成 BFC避免浮动造成的元素塌陷

## css sprite 是什么,有什么优缺点
概念：将多个小图片拼接到一个图片中。通过 background-position 和元素尺寸调节需要显示的背景图案。
优点：
1. 减少 HTTP 请求数，极大地提高页面加载速度
2. 增加图片信息重复度，提高压缩比，减少图片大小
3. 更换风格方便，只需在一张或几张图片上修改颜色或样式即可实现

缺点：
1. 图片合并麻烦
2. 维护麻烦，修改一个图片可能需要重新布局整个图片，样式

## link与@import的区别
1. link是 HTML 方式， @import是 CSS 方式
2. link最大限度支持并行下载，@import过多嵌套导致串行下载，出现FOUC
3. link可以通过rel="alternate stylesheet"指定候选样式
4. 浏览器对link支持早于@import，可以使用@import对老浏览器隐藏样式
5. @import必须在样式规则之前，可以在 css 文件中引用其他文件
6. 总体来说：link 优于@import

## display: block;和display: inline;的区别
block元素特点：
1. 处于常规流中时，如果width没有设置，会自动填充满父容器 
2. 可以应用margin/padding 
3. 在没有设置高度的情况下会扩展高度以包含常规流中的子元素 
4. 处于常规流中时布局时在前后元素位置之间（独占一个水平空间） 
5. 忽略vertical-align

inline元素特点：
1. 水平方向上根据direction依次布局
2. 不会在元素前后进行换行
3. 受white-space控制
4. margin/padding在竖直方向上无效，水平方向上有效
5. width/height属性对非替换行内元素无效，宽度由元素内容决定
6. 非替换行内元素的行框高由line-height确定，替换行内元素的行框高由height,margin,padding,border决定
7. 浮动或绝对定位时会转换为block
8. vertical-align属性生效

## display,float,position 的关系
1. 如果display为 none，那么 position 和 float 都不起作用，这种情况下元素不产生框
2. 否则，如果 position 值为 absolute 或者 fixed，框就是绝对定位的，float 的计算值为 none，display 根据下面的表格进行调整。
3. 否则，如果 float 不是 none，框是浮动的，display 根据下表进行调整
4. 否则，如果元素是根元素，display 根据下表进行调整
5. 其他情况下 display 的值为指定值 总结起来：绝对定位、浮动、根元素都需要调整display
