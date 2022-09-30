---
title: 聊聊前端开发中的长列表
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- js
---

## 前言
前端的业务开发中会遇到一些无法使用分页方式来加载的列表，我们一般把这种列表叫做长列表。在本篇文章中，我们把长列表定义成数据长度大于 1000 条，并且不能使用分页的形式来展示的列表。

本篇文章探讨了以下几个话题：
1. 完整渲染的长列表是否有优化的可能？优化的极限在什么位置？
2. 如果使用非完整的渲染长列表，有哪些方案以及具体的实现思路。

本篇文章的内容和具体框架无关，只是在部分例子中使用了 Vue 来实现。

## 完整渲染长列表
如果长列表不去做任何优化，一次完整渲染出来，到底需要多长时间呢？那么首先要先了解创建所有的 HTMLElement 并添加到 Document 中的时间消耗，因为业务中会混杂一些其他的代码，你的业务的性能不会比这个时间快。对浏览器创建元素的性能有大概的了解，才能知道长列表的优化极限在哪里。

我们可以写一个简单的方法来测试这个性能：
```javascript
var createElements = function(count) {
  var start = new Date();
  
  for (var i = 0; i < count; i++) {
    var element = document.createElement('div');
    element.appendChild(document.createTextNode('' + i));
    document.body.appendChild(element);
  }
  
  setTimeout(function() {
    alert(new Date() - start);
  }, 0);
};
```
我们给一个 Button 绑定了一个 onclick 事件，这个事件调用了 createElements(10000);。 从 Chrome 的 Profile 标签页看到的数据如下：

![](https://api2.mubu.com/v3/document_image/5c7357fe-be3e-464d-8ce7-cf3d5d91452f-3807603.jpg)

Event Click 只执行了 20.20ms，其他时间合计是 450ms，具体如下：
- Event Click: 20.20ms
- Recalculage Style: 16.86ms
- Layout: 410.6ms
- Update Layer Tree: 11.93ms
- Paint: 9.2ms

## 检测渲染时间的方法
你可能注意到了上面的测试代码中的时间计算过程中并没有直接在调用完 API 之后直接计算时间，而是使用了一个 setTimeout，下面会进行一些解释。

最简单的计算一段代码执行的时间可以这么写：
```javascript
var start = Date.now();
​
// ...
​
alert(Date.now() - start);
```

但是对于 DOM 的性能测试这么做是不科学的，因为 DOM 的操作会引起浏览器的 (reflow)，如果浏览器的 reflow 执行的时间远大于代码执行时间，会造成你时间计算完成之后，浏览器仍然在卡顿。统计的时间应该是从『开始创建元素』到『可以进行响应』的时间，所以一个合理的做法是把计算放在 setTimeout(function() {}, 0) 中。setTimeout() 中的 callback 会被推迟到浏览器主线程 reflow 结束后才执行，这个时间和 Chrome Devtools 下的 Profile 的时间基本吻合，可以信任这个时间作为渲染时间。

修改后的代码如下：
```javascript
var start = Date.now();
​
// ...
setTimeout(function() {
  alert(Date.now() - start);
}, 0);
```
如果需要更高的精度，可以使用 performance.now() 来替换 Date.now()，这个 API 可以精确到千分之一毫秒。

## 尝试使用不同的 DOM API
在前几年，优化元素创建性能经常提到的是使用 createDocumentFragment、innerHTML 来替代 createElement，通过 "createElement vs createDocumentFragment" 能找到相当多测测试结果。我们可以做个简单的实验，看看这个结论在 Google Chrome 中是否仍然适用。

我们会分别测试以下 4 种情况：
1. 创建一个空元素，并立即添加到 document 中。
2. 创建一个包含文本的元素，并立即添加到 document 中。
3. 创建一个 DocumentFragment，用来保存列表项，最后再把 DocumentFragment 添加到 document 中。
4. 拼出所有列表项的 HTML，使用元素的 innerHTML 属性赋值。

其中进行第一个测试的原因如下：使用空元素和带文本节点的元素，性能相差有 5 倍左右。
创建空元素的方法如下：
```javascript
var createEmptyElements = function(count) {
  var start = new Date();
  
  for (var i = 0; i < count; i++) {
    var element = document.createElement('div');
    document.body.appendChild(element);
  }
  
  setTimeout(function() {
    alert(new Date() - start);
  }, 0);
};
```
创建带文本元素的方法如下：
```javascript
var createElements = function(count) {
  var start = new Date();
  
  for (var i = 0; i < count; i++) {
    var element = document.createElement('div');
    element.appendChild(document.createTextNode('' + i));
    document.body.appendChild(element);
  }
  
  setTimeout(function() {
    alert(new Date() - start);
  }, 0);
};
```
使用 DocumentFragment 的方法如下：
```javascript
var createElementsWithFragment = function(count) {
  var start = new Date();
  var fragment = document.createDocumentFragment();
  
  for (var i = 0; i < count; i++) {
    var element = document.createElement('div');
    element.appendChild(document.createTextNode('' + i));
    fragment.appendChild(element);
  }
  
  document.body.appendChild(fragment);
  
  setTimeout(function() {
    alert(new Date() - start);
  }, 0);
};
```
使用 innerHTML 的方法如下：
```javascript
var createElementsWithHTML = function(count) {
  var start = new Date();
  var array = [];
  
  for (var i = 0; i < count; i++) {
    array.push('<div>' + i + '</div>');
  }
  
  var element = document.createElement('div');
  element.innerHTML = array.join('');
  document.body.appendChild(element);
  
  setTimeout(function() {
    alert(new Date() - start);
  }, 0);
};
```

## 数据统计
测试代码的计算的时间每次执行都会有一些误差，表格中的数据使用的是进行 10 次测试的平均值：

![](https://api2.mubu.com/v3/document_image/2edf7e13-f763-466b-a2a2-c4f74ee92aea-3807603.jpg)

从结果上来看，只有 innerHTML 会有 10% 的性能优势，createElement 和 createDocumentFragment 性能基本持平。对于现代浏览器来讲，性能瓶颈根本不在调用 DOM API 的阶段，无论使用哪种方式来使用 DOM API 添加元素，对性能的影响都微乎其微。

## 非完整渲染长列表
从上面的测试结果中可以看到，创建 10000 个节点就需要 500ms+，实际业务中的列表每个节点都需要 20 个左右的节点。那么，500ms 也仅能渲染 500 个左右的列表项。

所以完整渲染的长列表基本上很难达到业务上的要求的，非完整渲染的长列表一般有两种方式：
- 懒渲染：这个就是常见的无限滚动的，每次只渲染一部分（比如 10 条），等剩余部分滚动到可见区域，就再渲染另一部分。
- 可视区域渲染：只渲染可见部分，不可见部分不渲染。

## 懒渲染
懒渲染就是大家平常说的无限滚动，指的就是在滚动到页面底部的时候，再去加载剩余的数据。这是一种前后端共同优化的方式，后端一次加载比较少的数据可以节省流量，前端首次渲染更少的数据速度会更快。这种优化要求产品方必须接受这种形式的列表，否则就无法使用这种方式优化。

实现的思路非常简单：监听父元素的 scroll 事件（一般是 window），通过父元素的 scrollTop 判断是否到了页面是否到了页面底部，如果到了页面底部，就加载更多的数据。

本文使用 Vue 实现了一个简单的例子，这个例子中的可滚动区域是在 window 上的，其中的核心代码只有三行：
```javascript
const maxScrollTop = Math.max(document.body.scrollHeight, document.documentElement.scrollHeight) - window.innerHeight;
const currentScrollTop = Math.max(document.documentElement.scrollTop, document.body.scrollTop);
​
if (maxScrollTop - currentScrollTop < 20) {
  //...
}
```
完整的代码：
```vue
<template>
  <div class="lazy-list">
    <div class="lazy-render-list-item" v-for="item in data">{{ item }}</div>
  </div>
</template>
​
<style>
  .lazy-render-list {
    border: 1px solid #666;
  }
​
  .lazy-render-list-item {
    padding: 5px;
    color: #666;
    height: 30px;
    line-height: 30px;
    box-sizing: border-box;
  }
</style>
​
<script>
  export default {
    name: 'lazy-render-list',
​
    data() {
      const count = 40;
      const data = [];
​
      for (let i = 0; i < count; i++) {
        data.push(i);
      }
​
      return {
        count,
        data
      };
    },
​
    mounted() {
      window.onscroll = () => {
        const maxScrollTop = Math.max(document.body.scrollHeight, document.documentElement.scrollHeight) - window.innerHeight;
        const currentScrollTop = Math.max(document.documentElement.scrollTop, document.body.scrollTop);
​
        if (maxScrollTop - currentScrollTop < 20) {
          const count = this.count;
          for (let i = count; i < count + 40; i++) {
            this.data.push(i);
          }
          this.count = count + 40;
        }
      };
    }
  };
</script>
```
如果要应用在生产上，建议使用成熟的类库，可以通过 "框架名 + infinite scroll"来进行搜索。

## 可视区域渲染
可视区域渲染指的是只渲染可视区域的列表项，非可见区域的完全不渲染，在滚动条滚动时动态更新列表项。可视区域渲染适合下面这种场景：
- 每个数据的展现形式的高度需要一致（非必须，但是最小高度需要确定）。
- 产品设计上，一次需要加载的数据量比较大「1000条以上」。
- 产品设计上，滚动条需要挂载在一个固定高度的区域（在 window 上也可以，但是需要整个区域都只显示这个列表）。

本文使用 Vue 实现了一个例子来说明这种类型的列表该如何实现，这个例子做了以下三个设定：
- 列表的高度为 400px。
- 列表中的每个元素的高度是 30px。
- 一次加载 10000 条数据。

完整的代码：
```vue
<template>
  <div class="list-view" @scroll="handleScroll($event)">
    <div class="list-view-phantom" :style="{ height: data.length * 30 + 'px' }"></div>
    <div v-el:content class="list-view-content">
      <div class="list-view-item" v-for="item in visibleData">{{ item.value }}</div>
    </div>
  </div>
</template>
​
<style>
  .list-view {
    height: 400px;
    overflow: auto;
    position: relative;
    border: 1px solid #666;
  }
​
  .list-view-phantom {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    z-index: -1;
  }
​
  .list-view-content {
    left: 0;
    right: 0;
    top: 0;
    position: absolute;
  }
​
  .list-view-item {
    padding: 5px;
    color: #666;
    height: 30px;
    line-height: 30px;
    box-sizing: border-box;
  }
</style>
​
<script>
  export default {
    props: {
      data: {
        type: Array
      },
​
      itemHeight: {
        type: Number,
        default: 30
      }
    },
​
    ready() {
      this.visibleCount = Math.ceil(this.$el.clientHeight / this.itemHeight);
      this.start = 0;
      this.end = this.start + this.visibleCount;
      this.visibleData = this.data.slice(this.start, this.end);
    },
​
    data() {
      return {
        start: 0,
        end: null,
        visibleCount: null,
        visibleData: [],
        scrollTop: 0
      };
    },
​
    methods: {
      handleScroll(event) {
        const scrollTop = this.$el.scrollTop;
        const fixedScrollTop = scrollTop - scrollTop % 30;
        this.$els.content.style.webkitTransform = `translate3d(0, ${fixedScrollTop}px, 0)`;
​
        this.start = Math.floor(scrollTop / 30);
        this.end = this.start + this.visibleCount;
        this.visibleData = this.data.slice(this.start, this.end);
      }
    }
  };
</script>
```
例子代码中的实现细节如下，可以参考这个说明来辅助理解这个例子：
- 使用一个 phantom 元素来撑起整个这个列表，让列表的滚动条出现。
- 列表里面使用变量 visibleData(Array 类型) 记录目前需要显示的所有数据。
- 列表里面使用变量 visibleCount 记录可见区域最多显示多少条数据。
- 列表里面使用变量 start、end 记录可见区域数据的开始和结束索引。
- 在滚动的时候，修改真实显示区域的 transform: translate2d(0, y, 0)。

上面只是一个简单的例子，如果要用在生产上，你可以建议使用 Clusterize 或者 React Virtualized。

你可能会发现无限滚动在移动端很常见，但是可见区域渲染并不常见，这个主要是因为 iOS 上 UIWebView 的 onscroll 事件并不能实时触发。笔者曾尝试过使用 iScroll 来实现类似可视区域渲染，虽然初次渲染慢的问题可以解决，但是会出现滚动时体验不佳的问题（会有白屏时间）。

## 参考链接
[聊聊前端开发中的长列表](https://zhuanlan.zhihu.com/p/26022258)