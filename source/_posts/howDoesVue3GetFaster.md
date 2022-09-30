---
title: Vue3是如何变快的？
excerpt: 如题目
date: 2022-09-30
categories:
- 技术文章
tags:
- vue
---

## 前言
新版的Vue中最重要的一些特性有：
- 性能得到了显著的提升；
- Composition API；
- 一个组件可以支持多个根元素；
- Tree shaking，当一些功能不用的时候就不会打包到最终的代码里；
- 新的组件：Fragment、Teleport、Suspense；

其中性能提升主要体现在：更新比vue2有1.3～2倍的提升、SSR有2～3倍的提升。

## test
做一个验证，创建一个vue2项目和一个vue3项目, 分别加载有1万条数据点表格，点击shuffle按钮，打乱表格数据，计算更新时间，得到的结果如下：
- Vue2：加载时间410ms，更新时间713ms
- Vue3：加载时间358ms，更新时间533ms

比较两者的时间，更新的提升比较符合官方给出的范围。

## Vue3是如何变快的呢？
- diff算法的优化
- hoistStatic 静态提升
- cacheHandlers 事件侦听器缓存
- ssr渲染

### diff方法优化
例如一个包含div，div内包含三个段落，其中前两个段落是静态固定不变的，而第三个段落的内容绑定的msg属性，当msg改变的时候，Vue会生成新的虚拟DOM然后和旧的进行对比。

vue2中虚拟Dom的所有节点都进行对比。

然而模板中div元素是固定不变的，前两个p元素也是固定不变的，可能变化的只是第3个p元素的文本，对所有元素进行对比没有必要。

vue3新增了静态标记（PatchFlag），在创建虚拟DOM的时候，如果节点会发生变化，就会加上静态标记， 然后对比的时候只比较带有Patchflag的节点。

Patchflag是个枚举，取值为1代表这个元素的文本是动态绑定的，取值为2代表元素的class是动态绑定的。

### 静态提升
vue2中无论元素是否参与更新，每次都会重新创建，然后再渲染。还是以刚才点模板为例，msg每次更新都会创建前两个p节点。

vue3中对于不参与更新点元素，会做静态提升、只会被创建一次，在渲染时直接复用即可。

### 事件侦听器缓存
给元素绑定一个事件会被视为一个动态绑定，前面说过动态绑定会给它添加Patchflag，每次DOM比较点时候都会比较这个节点，但是事件绑定点函数都是同一个函数，所以不用追踪变化，直接缓存起来复用即可。
```html
<div>
    <button @click="onClick">提交</button>
</div>
```
没有缓存事件侦听器：
```javascript
export function render(_ctx, _cache, $props, $setup, $data, $options) {
 return (_openBlock(), _createBlock("div", null, [
 _createVNode("button", { onClick: _ctx.onClick }, "提交", 8 /* PROPS */, ["onClick"])
 ]))
}
```
缓存了事件侦听器：
```javascript
export function render(_ctx, _cache, $props, $setup, $data, $options) {
 return (_openBlock(), _createBlock("div", null, [
 _createVNode("button", {
 onClick: _cache[1] || (_cache[1] = (...args) => (_ctx.onClick(...args)))
 }, "提交")
 ]))
}
```

### ssr渲染
当有大量静态内容的时候，这些内容会被当作纯字符串推进一个buffer里面，即使存在动态的绑定，会通过模板插值嵌入进去。这样会比通过虚拟DOM来渲染的快上很多。

当静态内容大到一定量级的时候，会用_createStaticVNode方法在客户端去生成一个static node，这些静态node，会被直接innerHtml， 就不需要再创建对象，然后根据对象渲染。

## 总结
vue3通过优化diff算法减少了遍历成本，然后通过静态提升以及时间侦听器缓存减少了多次创建的开销，从而组件更新性的能得到了提升。

## 参考链接
[Vue3是如何变快的？](http://www.geeee.top/2020/09/29/vue3/)