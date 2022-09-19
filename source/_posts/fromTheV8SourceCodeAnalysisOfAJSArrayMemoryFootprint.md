---
title: 从V8源码分析一个JS数组的内存占用问题
excerpt: 如题目
categories:
- 技术文章
tags:
- js
- 浏览器
---

## eg
先看一段代码
```javascript
const a = new Array(99999);
a[99998] = undefined;
const b = new Array(99999);
b[99999] = undefined;
```

我们通过 node --inspect-brk 来分别运行这两段代码，在代码运行的最开始和结束的时候分别task heap snapshot，分析对应的内存占用信息如下：

![](https://api2.mubu.com/v3/document_image/edb84f6f-f25e-46ad-9bcc-ae55eb462a80-3807603.jpg)

![](https://api2.mubu.com/v3/document_image/e05f4081-3502-4a5d-802d-2b51214690d4-3807603.jpg)

可以发现第二段代码的内存占用明显要小于第一段，那么问题就出现在这个 99999 的越界赋值上面。

## 解释

在[V8代码](https://github.com/v8/v8/blob/master/src/objects/js-array.h#L19)中有很明确的标注，数组有两种模式，快数组和慢数组，在数组初始化时，默认的存储方式为[快数组](https://github.com/v8/v8/blob/master/src/objects/js-objects.h#L317)，其内存占用是连续的，而慢数组会使用HashTable来进行数据存储。另外数组会分为压紧（Packed）的和有洞的（Holey）两种，例如 ['a', 'b', 'c'] 这样的数组长度为3，数组索引0、1、2均有值，那么就认为是Packed；而对于 ['a',,,'d'] 这样的数组，长度为4，但是索引1、2位置并没有进行初始化赋值，那么就认为是Holey。当数组出现了较大空洞的时候，内存明显是被浪费了。

V8中对于大型空洞数组进行了优化，在[V8博客](https://v8.dev/blog/fast-properties#elements-or-array-indexed-properties)中进行说明了这一点，对于非常大的Holey数组来说，FixedArray会造成内存浪费，所以会使用字典来节约内存，也就是会使用慢数组模式。

使用v8-debug分别对最开始的两段代码进行调试：

![](https://api2.mubu.com/v3/document_image/4d7ccb1f-6864-408e-be32-69ed14732fa5-3807603.jpg)

![](https://api2.mubu.com/v3/document_image/7d63f0de-3cb0-4587-a549-b4f9d9806e77-3807603.jpg)

可以很明显的看到，第一个数组为FixedArray，而第二个数组为Dictionary，那么为什么只有第二个数组转换为了字典模式呢？

在V8中JSArray是继承于JSObject的，所以当设置属性的时候，会依次执行 `Object::SetProperty` 、 `Object::AddDataProperty` 、 `JSObject::AddDataElement` 、 `ShouldConvertToSlowElements` ,回到V8代码中，`ShouldConvertToSlowElements`这个方法，它是用来判断是否将一个数组转换为[慢模式（Dictionary）](https://github.com/v8/v8/blob/master/src/objects/js-objects-inl.h#L794)：

![](https://api2.mubu.com/v3/document_image/d4a225d5-d13f-4b50-b3c1-58b9e9252b42-3807603.jpg)

从上面的代码可以看到，当设置 99998 的时候，索引小于当前容量的时候，返回值为false，也就是不进行转换。 而当设置 99999 这个索引的值的时候，因为超出了原来的FixedArray容量，那么就会进行扩容，[扩容的算法](https://github.com/v8/v8/blob/4b9b23521e6fd42373ebbcb20ebe03bf445494f9/src/objects/js-objects.h#L540)为容量 + 容量 /2 + 16，那么原来 99999 的容量就会扩容放大到 15万。

![](https://api2.mubu.com/v3/document_image/a01fc97b-6f49-45a8-b676-24ccd5c36423-3807603.jpg)

然后会执行 GetFastElementsUsage 来获取原来的数组中[非空洞](https://github.com/v8/v8/blob/4b9b23521e6fd42373ebbcb20ebe03bf445494f9/src/objects/js-objects.cc#L4725)的元素数量，乘以 kPreferFastElementsSizeFactor(值为3) 与 kEntrySize (值为2) ，与新的容量长度进行对比，如果小于新的容量长度，那么就转换为慢数组。

最开始的第二段代码中，非空洞元素数量为0，计算后的乘积也为0，因此小于15万的新数组长度，于是数组转换为了慢数组，使用了Dictionary进行数据的存储，从而节省了大量的内存。

（本篇内容来自阿里巴巴淘系技术 洗剑）

## 参考链接
[从V8源码分析一个JS 数组的内存占用问题](https://zhuanlan.zhihu.com/p/342042184)
