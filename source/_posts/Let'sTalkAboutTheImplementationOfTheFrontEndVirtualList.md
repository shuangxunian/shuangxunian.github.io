---
title: 再谈前端虚拟列表的实现
excerpt: 如题目
categories:
- 技术文章
tags:
- js
---

## 前言
书接上文，在之前的 聊聊前端开发中的长列表 中，笔者对「可视区域渲染」的列表进行了介绍，并写了一个简化的例子来展现如何实现。这种列表一般叫做 Virtual List，在本文中会使用「虚拟列表」来指代。在本文中，笔者会把上篇文章中的简化例子一步步强化成一个相对通用、性能优异的虚拟列表组件，旨在讲清楚虚拟列表的实现思路。阅读本文不需要阅读上一篇文章，但代码是使用 Vue.js 来实现的，所以你需要有 Vue.js 的使用经验。另外，提供了各个步骤的 JSFiddle，如果有不懂的地方，建议通过修改 JSFiddle 在线运行调试。

## 实现思路
在讲解下面的内容之前，先对虚拟列表做一个简单的定义。

因为 DOM 元素的创建和渲染需要的时间成本很高，在大数据的情况下，完整渲染列表所需要的时间不可接收。其中一个解决思路就是在任何情况下只对「可见区域」进行渲染，可以达到极高的初次渲染性能。

虚拟列表指的就是「可视区域渲染」的列表，重要的基本就是两个概念：
- 可滚动区域：假设有 1000 条数据，每个列表项的高度是 30，那么可滚动的区域的高度就是 1000 * 30。当用户改变列表的滚动条的当前滚动值的时候，会造成可见区域的内容的变更。
- 可见区域：比如列表的高度是 300，右侧有纵向滚动条可以滚动，那么视觉可见的区域就是可见区域。

实现虚拟列表就是处理滚动条滚动后的可见区域的变更，其中具体步骤如下：
1. 计算当前可见区域起始数据的 startIndex
2. 计算当前可见区域结束数据的 endIndex
3. 计算当前可见区域的数据，并渲染到页面中
4. 计算 startIndex 对应的数据在整个列表中的偏移位置 startOffset，并设置到列表上

建议参考下图理解一下上面的步骤：

![](https://api2.mubu.com/v3/document_image/4d11d252-aa63-47f6-8baa-fcea6fcc17b0-3807603.jpg)

## 最简单的例子
这个章节会讲述如何把上面步骤变成代码，让这个逻辑在浏览器里真正的运行起来。

为了让这个例子足够简单，做了一个设定：每个列表项的高度都是 30px。在这个约定下，核心 JavaScript 代码不超过 10 行，但是可以完整的实现可见区域的渲染和更新。

我们首先要考虑的是虚拟列表的 HTML、CSS 如何实现，添加了这么几个样式：
- 列表元素（.list-view）使用相对定位
- 使用一个不可见元素（.list-view-phantom）撑起这个列表，让列表的滚动条出现
- 列表的可见元素（.list-view-content）使用绝对定位，left、right、top 设置为 0

```vue
<template>
  <div 
    class="list-view" 
    @scroll="handleScroll">
    <div
      class="list-view-phantom"       
      :style="{
         height: contentHeight
      }">
    </div>
    <div
      ref="content"
      class="list-view-content">
      <div
        class="list-view-item"
        :style="{
          height: itemHeight + 'px'
        }"
        v-for="item in visibleData">
        {{ item.value }}
      </div>
    </div>
  </div>
</template>
​
<style>
.list-view {
  height: 400px;
  overflow: auto;
  position: relative;
  border: 1px solid #aaa;
}

.list-view-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.list-view-content {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
}

.list-view-item {
  padding: 5px;
  color: #666;
  line-height: 30px;
  box-sizing: border-box;
}
</style>
​
<script>
export default {
    name: 'ListView',
    
    props: {
    data: {
        type: Array,
      required: true
    },

    itemHeight: {
      type: Number,
      default: 30
    }
  },
  
  computed: {
    contentHeight() {
        return this.data.length * this.itemHeight + 'px';
    }
  },

  mounted() {
    this.updateVisibleData();
  },

  data() {
    return {
      visibleData: []
    };
  },

  methods: {
    updateVisibleData(scrollTop) {
        scrollTop = scrollTop || 0;
        const visibleCount = Math.ceil(this.$el.clientHeight / this.itemHeight);
      const start = Math.floor(scrollTop / this.itemHeight);
      const end = start + visibleCount;
      this.visibleData = this.data.slice(start, end);
      this.$refs.content.style.webkitTransform = `translate3d(0, ${ start * this.itemHeight }px, 0)`;
    },

    handleScroll() {
      const scrollTop = this.$el.scrollTop;
      this.updateVisibleData(scrollTop);
    }
  }
}
</script>
```

1. 渲染可见区域的数据是 Vue.js 完成的，使用的是 visibleData 这个数组中的元素
2. 其中虚拟列表的更新逻辑是 updateVisibleData 这个方法，笔者对这个方法加了一些注释：
    ```javascript
    updateVisibleData(scrollTop) {
        scrollTop = scrollTop || 0;
        const visibleCount = Math.ceil(this.$el.clientHeight / this.itemHeight); // 取得可见区域的可见列表项数量
        const start = Math.floor(scrollTop / this.itemHeight); // 取得可见区域的起始数据索引
        const end = start + visibleCount; // 取得可见区域的结束数据索引
        this.visibleData = this.data.slice(start, end); // 计算出可见区域对应的数据，让 Vue.js 更新
        this.$refs.content.style.webkitTransform = `translate3d(0, ${ start * this.itemHeight }px, 0)`; // 把可见区域的 top 设置为起始元素在整个列表中的位置（使用 transform 是为了更好的性能）
    }
    ```
3. 为了让虚拟列表可以正确的出现滚动条，使用了 Vue.js 的计算属性 contentHeight 来计算滚动区域的真正高度

## 去掉高度限制
最简单实现中存在每个元素高度相同的限制，如果打破这个限制，代码该如何实现？

例子中使用了 itemHeight 属性来决定列表项的高度，有两个选择可以实现列表项的动态高度：
1. 添加一个数组类型的 prop，每个列表项的高度通过索引来获得
2. 添加一个获取列表项高度的方法，给这个方法传入 item 和 index ，返回对应列表项的高度

很明显第二种方案更灵活一点，所以增加了一个 itemSizeGetter 属性，用来获取每个列表项的高度。

1. 增加 itemSizeGetter 属性，代码如下：
    ```javascript
    itemSizeGetter: {
        type: Function
    }
    ```
2. 因为每一行的高度是不一样的，所以 contentHeight 的算法需要进行更新，更新后的代码如下：
    ```javascript
    contentHeight() {
        const { data, itemSizeGetter } = this;
        let total = 0;
        for (let i = 0, j = data.length; i < j; i++) {
            total += itemSizeGetter.call(null, data[i], i);
        }
        return total;
    }
    ```
3. 上一个例子中，计算可是区域的起始索引和结束索引只需要使用 itemHeight 进行一些简单的计算。在这个例子里面，需要通过 scrollTop 来计算出这个位置的元素索引，所以增加了一个方法叫 findNearestItemIndex，实现代码如下：
    ```javascript
    findNearestItemIndex(scrollTop) {
        const { data, itemSizeGetter } = this;
        let total = 0;
        for (let i = 0, j = data.length; i < j; i++) {
            const size = itemSizeGetter.call(null, data[i], i);
            total += size;
            if (total >= scrollTop || i === j -1) {
            return i;
            }
        }
        return 0;
    }
    ```
4. 同理，某个列表项在列表中的 top 之前也可以通过索引简单的计算出来，现在需要增加一个方法来计算，实现代码如下：
    ```javascript
    getItemSizeAndOffset(index) {
        const { data, itemSizeGetter } = this;
        let total = 0;
        for (let i = 0, j = Math.min(index, data.length - 1); i <= j; i++) {
            const size = itemSizeGetter.call(null, data[i], i);

            if (i === j) {
            return {
                offset: total,
                size
            };
            }
            total += size;
        }

        return {
            offset: 0,
            size: 0
        };
    }
    ```
5. updateVisibleData 方法根据上面的修改做了一个简单的更新，代码如下：
    ```javascript
    updateVisibleData(scrollTop) {
        scrollTop = scrollTop || 0;
        const start = this.findNearestItemIndex(scrollTop);
        const end = this.findNearestItemIndex(scrollTop + this.$el.clientHeight);
        this.visibleData = this.data.slice(start, Math.min(end + 1, this.data.length));
        this.$refs.content.style.webkitTransform = `translate3d(0, ${ this.getItemSizeAndOffset(start).offset }px, 0)`;
    }
    ```

## 缓存计算结果
虽然上个例子实现了列表项的动态高度，但是每个列表项目的尺寸、偏移计算没有任何缓存，在初次渲染、滚动更新时 itemSizeGetter 会被重复调用，性能并不理想。为了优这个性能，需要把尺寸、偏移信息进行一个缓存，在下次时候的时候直接从缓存中取得结果。

在常规情况下，用户的滚动是从顶部开始的，并且是连续的。可以采取一个非常简单的缓存策略，记录最后一次计算尺寸、偏移的 index 。
1. 把这个变量叫做 lastMeasuredIndex，默认值为 -1；存储缓存结果的变量叫做 sizeAndOffsetCahce，类型为对象，实现代码如下：
    ```javascript
    data() {
        return {
            lastMeasuredIndex: -1,
            startIndex: 0,
            sizeAndOffsetCahce: {},
            ...
        };
    }
    ```
2. 缓存列表项高度的计算结果主要是修改 getItemSizeAndOffset 这个方法，增加缓存后的代码如下：
    ```javascript
    getItemSizeAndOffset(index) {
        const { lastMeasuredIndex, sizeAndOffsetCahce, data, itemSizeGetter } = this;
        if (lastMeasuredIndex >= index) {
            return sizeAndOffsetCahce[index];
        }
        let offset = 0;
        if (lastMeasuredIndex >= 0) {
            const lastMeasured = sizeAndOffsetCahce[lastMeasuredIndex];
            if (lastMeasured) {
            offset = lastMeasured.offset + lastMeasured.size;
            }
        }
        for (let i = lastMeasuredIndex + 1; i <= index; i++) {
            const item = data[i];
            const size = itemSizeGetter.call(null, item, i);
            sizeAndOffsetCahce[i] = {
            size,
            offset
            };
            offset += size;
        }
        if (index > lastMeasuredIndex) {
            this.lastMeasuredIndex = index;
        }
        return sizeAndOffsetCahce[index];
    }
    ```
3. findNearestItemIndex 方法中的还在使用 itemSizeGetter 来获取元素大小，我们在这里可以修改成使用 getItemSizeAndOffset 来获取，修改后的代码如下：
    ```javascript
    findNearestItemIndex(scrollTop) {
        const { data, itemSizeGetter } = this;
        let total = 0;
        for (let i = 0, j = data.length; i < j; i++) {
            const size = this.getItemSizeAndOffset(i).size;
            // ...
        }

        return 0;
    }
    ```

如果觉得性能并没有明显的改进，可以把数组的大小改成 20000 或者 50000 试试。

## 优化 contentHeight 的计算
如果你给 itemSizeGetter 加入一行 console.log，你会发现初次渲染的时候 contentHeight 会在第一次把所有列表项的 itemSizeGetter 执行一遍。

为了解决这个问题，需要引入另外一个属性 estimatedItemSize。这个属性的含义是为那些还没计算高度的元素进行一个预估，那么 contentHeight 就等于缓存过的列表项的高度和 + 未缓存过的列表项的数量 * estimatedItemSize。
1. 首先需要为组件增加这个属性，默认值为 30，代码如下：
    ```javascript
    estimatedItemSize: {
        type: Number,
        default: 30
    }
    ```
2. 因为需要得知计算过高度的列表项的高度和，需要增加方法 getLastMeasuredSizeAndOffset，代码如下：
    ```javascript
    getLastMeasuredSizeAndOffset() {
        return this.lastMeasuredIndex >= 0 ? this.sizeAndOffsetCahce[this.lastMeasuredIndex] : { offset: 0, size: 0 };
    }
    ```
3. 根据上面的算法修改后的代码如下：
    ```javascript
    contentHeight() {
        const { data, lastMeasuredIndex, estimatedItemSize } = this;
        let itemCount = data.length;
        if (lastMeasuredIndex >= 0) {
            const lastMeasuredSizeAndOffset = this.getLastMeasuredSizeAndOffset();
            return lastMeasuredSizeAndOffset.offset + lastMeasuredSizeAndOffset.size + (itemCount - 1 - lastMeasuredIndex) * estimatedItemSize;
        } else {
            return itemCount * estimatedItemSize;
        }
    }
    ```

你可以为 itemSizeGetter 属性增加 console.log，来查看 itemSizeGetter 是如何运行的。

## 优化已缓存结果的搜索性能
使用过缓存的虚拟列表实际上还有优化的空间，比如 findNearestItemIndex 的搜索方式是顺序查找的，时间复杂度为 O(N)。实际上列表项的计算结果天然就是一个有序的数组，可以使用二分查找来优化已缓存的结果的搜索性能，把时间复杂度降低到 O(lgN) 。

1. 为组件增加 binarySearch 方法，代码如下：
    ```javascript
    binarySearch(low, high, offset) {
        let index;
        while (low <= high) {
            const middle = Math.floor((low + high) / 2);
            const middleOffset = this.getItemSizeAndOffset(middle).offset;
            if (middleOffset === offset) {
            index = middle;
            break;
            } else if (middleOffset > offset) {
            high = middle - 1;
            } else {
            low = middle + 1;
            }
        }
        if (low > 0) {
            index = low - 1;
        }
        if (typeof index === 'undefined') {
            index = 0;
        }
        return index;
    }
    ```
    这个二分查找和普通的二分查找略有不同，区别在于在任何情况下都不会返回 -1，可以思考下为什么这个逻辑会是这样。
2. 修改 findNearestItemIndex 方法，对于已缓存的结果使用二分查找，代码如下：
    ```javascript
    findNearestItemIndex(scrollTop) {
        const { data, itemSizeGetter } = this;
        const lastMeasuredOffset = this.getLastMeasuredSizeAndOffset().offset;
        if (lastMeasuredOffset > scrollTop) {
            return this.binarySearch(0, this.lastMeasuredIndex, scrollTop);
        } else {
            // ...
        }

        return 0;
    }
    ```

## 优化未缓存结果的搜索性能
未缓存过的结果的搜索依然是顺序搜索的，对于未缓存过的结果的搜索优化有两个思路：
1. 一次查找多条的数量，一个合理的数值是 contentHeight / estimatedSize ，找到超过自己查找结果的 index，然后使用二分查找
2. 按指数数量查找，比如 1、2、4、8、16、32… 的顺序来查找范围，然后使用二分查找

笔者选择了第二种方式，这个搜索算法的名称为 Exponential Search，这个算法的搜索过程可以参考下图：

![](https://api2.mubu.com/v3/document_image/171c72b8-14db-471f-9e12-8b55ff95aaad-3807603.jpg)

1. 增加 exponentialSearch 方法，代码如下：
    ```javascript
    exponentialSearch(scrollTop) {
        let bound = 1;
        const data = this.data;
        const start = this.lastMeasuredIndex >= 0 ? this.lastMeasuredIndex : 0;
        while (start + bound < data.length && this.getItemSizeAndOffset(start + bound).offset < scrollTop) {
            bound = bound * 2;
        }
        return this.binarySearch(start + Math.floor(bound / 2), Math.min(start + bound, data.length), scrollTop);
    }
    ```
    这个算法与标准的 Exponential Search 也略有不同，主要的区别是不会从头进行搜索，会从 lastMeasuredIndex 的位置开始搜索。
2. 修改 findNearestItemIndex 方法，代码如下：
    ```javascript
    findNearestItemIndex(scrollTop) {
        const { data, itemSizeGetter } = this;
        const lastMeasuredOffset = this.getLastMeasuredSizeAndOffset().offset;
        if (lastMeasuredOffset > scrollTop) {
            return this.binarySearch(0, this.lastMeasuredIndex, scrollTop);
        } else {
            return this.exponentialSearch(scrollTop);
        }
    }
    ```

## 参考链接
[再谈前端虚拟列表的实现](https://zhuanlan.zhihu.com/p/34585166)