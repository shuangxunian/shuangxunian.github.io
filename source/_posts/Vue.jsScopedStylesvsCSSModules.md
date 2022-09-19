---
title: Vue scoped 样式与 CSS Module 对比
excerpt: 如题目
categories:
- 技术文章
tags:
- vue
- css
---

## 前言
在现代化的 Web 开发中，CSS 还远未完美，这一点应该没有什么异议。

现今的项目通常都相当复杂，而 css 样式天生又是全局性的，所以到最后总是极容易地就发生样式冲突——要么是样式相互覆盖，要么就是隐式地级联到了下面那些我们未考虑到的元素。

在减轻 CSS 存在的主要痛点方面，我们普遍采用的解决方案是引入 BEM (Block Element Modifier) 方法学。不过这只能解决我们这个大问题的很小一部分。

我们非常幸运，社区已经开发出了一些解决方案，他们可以帮我们处理这些问题。说不定你已经听说过了 CSS Modules、Styled Components、Glamorous、JSS——这些只是众多流行的工具中的少数几个。如果你对这个话题感兴趣，你可以查看这篇帖文——作者 Indrek Lasn 对 CSS-in-JS 的思想做了非常详尽的讲解。

每个通过 vue-cli 创建的 Vue.js 应用都内置了两个很好的解决方案：Scoped CSS 和 CSS Modules (模块式 CSS)。两种方案各有优缺点，所以下面我们就仔细看下哪种方案在你的案例中更适用。

## Scoped 样式
我们只需要在 style 标签上添加一个 scoped 属性即可启用 scoped 样式：
```html
<template>
  <button class=”button” />
</template>
<style scoped>
  .button {
    color: red;
  }
</style>
```

这样就会使得我们的样式只被应用到这个组件中的元素上。这是借助 PostCSS 实现的，它会将上面的代码转换成下面这样：
```html
<style>
.button[data-v-f61kqi1] {
  color: red;
}
</style>
<button class=”button” data-v-f61kqi1></button>
```

就像你看到的这样，整个过程不需要做什么就可以达到很好的 scoped 样式效果。

现在假设你需要调整一个视图中的某个组件的宽度，那么你可以像你平时那样做的一样：在这个组件上添加一个额外的 class 来设置其样式。
```html
<template>
  <BasePanel class=”pricing-panel”>
    content  
  </BasePanel>
</template>
<style scoped>
  .pricing-panel {
    width: 300px;
    margin-bottom: 30px;
  }
</style>
```

经转换后：
```html
<style>
  .base-panel[data-v-d17eko1] {
    ...
  }
  .pricing-panel[data-v-b52c41] {
    width: 300px;
    margin-bottom: 30px;
  }
</style>
<div class=”base-panel pricing-panel” data-v-d17eko1 data-v-b52c41>
  content
</div>
```

这次还是一样，不需要做什么你就获得了对布局的彻底控制。

不过请注意：这个特性存在一个缺陷，即如果你子组件的根元素上有一个类已经在这个父组件中定义过了，那么这个父组件的样式就会泄露到子组件中。如果想更好地理解这个问题，可以查看这个 CodeSandbox 例子。

还有一些情况是我们需要对我们的子组件的深层结构设置样式——虽然这种做法并不受推荐且应该避免。为了简便起见，我们假设我们的父组件现在要对 BasePanel 的标题设置样式，在 scoped 样式中，这种情况可以使用 >>> 连接符（或者 /deep/ ）实现。

```html
<style scoped>
  .pricing-panel >>> .title {
    font-size: 24px;
  }
</style>
```

经转换后：
```html
.pricing-panel[data-v-b52c41] .title {
  font-size: 24px;
}
```
非常简单，是吧？可是别忘记，我们却因此失去了组件的封装效果。这个组件内的所有的 .title 类的样式都会被这些样式所浸染——即便是孙节点。

## 模块式 CSS
模块式 CSS 的流行源于 React 社区，它获得了社区的迅速的采用。

Vue.js 更甚之，其强大、简便的特性在加上通过 vue-cli 对其开箱即用的支持，将其发展到另一个高度。

现在让我们来看下怎么使用它：
```html
<style module>
  .button {
    color: red
  }
</style>
```

这次我们使用的不是 scoped 属性，而是 module。这等于告诉 vue-template-compiler 和 vue-cli 的 webpack 配置要对这一部分采用哪些相应的 loader，进而生成像下面这样的 CSS：
```css
.ComponentName__button__2Kxy {
  color: red;
}
```

它的特殊之处以及和 scoped 样式不一样的地方就在于所有创建的类可以通过这个组件的 $style 对象获取。因此，要将这个类进行应用，我们需要像下面这样进行 class 绑定：
```html
<template>
  <button :class="$style.button" />
</template>
<style module>
  .button {
    color: red
  }
</style>
```

这段代码将生成下面的 HTML 及相关的样式：
```html
<style>
  .ComponentName__button__2Kxy {
    color: red;
  }
</style>
<button class=”ComponentName__button__2Kxy”></button>
```

它的第一点好处就是，当我们在 HMTL 中查看这个元素时我们可以立刻知道它所属的是哪个组件；第二点好处是，一切都变成显式的了，我们拥有了彻底的控制权——不会再有什么奇怪的现象了。和 scoped 样式那种把普通的标签也加上那些 data 属性的做法不一样，这些普通标签在转换后还是最初的样子。

比较 scoped 样式中的第二个例子，我们来看下我们可以怎么对那个组件设置样式：

```html
<template>
  <BasePanel :class="$style['pricing-panel']">
    content  
  </BasePanel>
</template>
<style module>
  .pricing-panel {
    width: 300px;
    margin-bottom: 30px;
  }
</style>
```

其转换后：
```html
<style>
  .BasePanel__d17eko1 {
    /* some styles */
  }
  .ComponentName__pricing-panel__a81Kj {
    width: 300px;
    margin-bottom: 30px;
  }
</style>
<div class="BasePanel__d17eko1 ComponentName__pricing-panel__a81Kj">
  content
</div>
```

毫无意外，跟我们期望的结果一样。此外，因为所有的 CSS 类可以通过 $style 对象获取到，所以我们可以通过 props 将这些类传递到任何我们希望的深度中，这样，在子组件中的任意位置使用这些类就会变得极其容易：
```html
<template>
  <BasePanel
    title="Lorem ipsum"
    :titleClass="$style.title"
  >
    Content
  </BasePanel>
</template>
```

模块式 CSS 与 JS 有着很好的互操作性 (interoperability)，这一点不只局限于 CSS 类。我们还可以使用 :export 关键字将其他的东西导出到 $style 对象上。

例如，想象一下你有一个图表需要开发 —— 你可以在 CSS 中定义你的色彩变量的同时将其导出，以供你的组件使用：
```html
<template>
  <div>{{ $style.primaryColor }}</div> <!-- #B4DC47 -->
</template>
<style module lang="scss">
  $primary-color: #B4DC47;

  :export {
    primaryColor: $primary-color
  }
</style>
```
对于模块式 CSS的概念，我这里还只是讲到了它的皮毛，它实际要宽泛的多，建议你查看下它完整的规范以了解更多。

## 总结
其实两种方案都非常简单、易用，在某种程度上解决的是同样的问题。 那么你该选择哪种呢？

scoped 样式的使用不需要额外的知识，给人舒适的感觉。它所存在的局限，也正是它的使用简单的原因。它可以用于支持小型到中型的应用。

在更大的应用或更复杂的场景中，这个时候，对于 CSS 的运用，我们就会希望它更加显式，拥有更多的控制权。虽然在模板中大量使用 $style 看起来并不那么“性感”，但却更加安全和灵活，为此我们只需付出微小的代价。还有一个好处就是我们可以用 JS 获取到我们定义的一些变量（如色彩值、样式断点），这样我们就无需手动保持其在多个文件中同步。

## 参考链接
[译 Vue: scoped 样式与 CSS Module 对比](https://juejin.cn/post/6844903673517211655)

