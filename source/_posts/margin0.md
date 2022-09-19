---
title: margin:0的作用与利弊
excerpt: 分析一下margin:0的利弊
categories:
- 技术文章
tags:
- css
---

## 作用
{margin: 0; padding: 0}作用是重置浏览器默认样式，对于各浏览器样式统一的话有着简单粗暴的效果。

## 分析
*叫“通配符”用来匹配页面上所有元素。

*{margin:0; padding:0;}代表了网页中所有元素，包括body ,ul, li列表标签 ,p,H标题标签,dd,dt 等……都有默认的margin 或padding值的，加上这句就可以删除浏览器这些默认值，方面后面的设置。

## 利处
使用*{margin: 0; padding: 0}可以简单方便的一次性重置所有HTML网页元素的浏览器样式，代码少，控制量大。

## 弊处
用*效率会低很多(据说)，因为它重置了所有元素的样式，包括不需要重置的样式，例如table，我们不需要去重置table元素的样式，所以还需要再为 table 设置默认样式，反而增加了代码量。

## 解决
我们可以换一种方法解决：

```css
body,
div,
dl,
dt,
dd,
ul,
ol,
li,
h1,
h2,
h3,
h4,
h5,
h6,
pre,
form,
fieldset,
input,
textarea,
p,
blockquote,
th,
td,
img {
    padding: 0;
    margin: 0;
}
```

## 参考链接
[*{margin: 0; padding: 0}的作用与利弊](https://www.imooc.com/article/23764)