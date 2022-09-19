---
title: 记一次关于滚动条的bug
excerpt: 记一次关于滚动条的bug
categories:
- 技术文章
tags:
- DOM
- 浏览器
- debug
---

## 问题
之前看老男孩前端，弄了一个点击滚动到最顶端。按视频来的他那边可以，我这边不行，一直没弄明白。

## 解决
我之前根本没考虑是浏览器的问题，我以为培训班用的也是Chrome，今天搭项目又需要这个回到顶端，接着重读了一遍代码，发现一个地方

```javascript
var s = document.body.scrollTop;
```

看！body，突然灵光一现，可能是浏览器的问题，于是用IE打开，发现好用了；这样可以直接确定就是浏览器的问题。

Google一查，各浏览器下scrollTop的差异：
- **IE6/7/8：**
对于没有doctype声明的页面里可以使用  document.body.scrollTop 来获取 scrollTop高度 ；
对于有doctype声明的页面则可以使用 document.documentElement.scrollTop；

- **Safari:**
safari 比较特别，有自己获取scrollTop的函数 ： window.pageYOffset ；

- **Firefox:**
火狐等等相对标准些的浏览器就省心多了，直接用 document.documentElement.scrollTop ；

直接使用万能写法：

```javascript
var scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop;
```

啊这终于解决了，还是很开心的。