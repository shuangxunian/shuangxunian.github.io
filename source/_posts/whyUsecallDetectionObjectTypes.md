---
title: 为什么用Object.prototype. toString.call(obj)检测对象类型?
excerpt: 如题目~
categories:
- 技术文章
tags:
- js
updated: 2022-10-01
---

## 问题
使用typeof bar === "object"检测”bar”是否为对象有什么缺点？如何避免？

## 解释
这是一个十分常见的问题，用 typeof 是否能准确判断一个对象变量，答案是否定的，null 的结果也是 object，数组的结果也是 object，有时候我们需要的是 "纯粹" 的 object 对象。如何避免呢？比较好的方式是：
```javascript
console.log(Object.prototype.toString.call(obj) === "[object Object]");
```
使用以上方式可以很好的区分各种类型：
（无法区分自定义对象类型，自定义类型可以采用instanceof区分）
```javascript
console.log(Object.prototype.toString.call("jerry"));//[object String]
console.log(Object.prototype.toString.call(12));//[object Number]
console.log(Object.prototype.toString.call(true));//[object Boolean]
console.log(Object.prototype.toString.call(undefined));//[object Undefined]
console.log(Object.prototype.toString.call(null));//[object Null]
console.log(Object.prototype.toString.call({name: "jerry"}));//[object Object]
console.log(Object.prototype.toString.call(function(){}));//[object Function]
console.log(Object.prototype.toString.call([]));//[object Array]
console.log(Object.prototype.toString.call(new Date));//[object Date]
console.log(Object.prototype.toString.call(/\d/));//[object RegExp]
function Person(){};
console.log(Object.prototype.toString.call(new Person));//[object Object]
```
为什么这样就能区分呢？于是我去看了一下toString方法的用法：toString方法返回反映这个对象的字符串。
那为什么不直接用obj.toString()呢？
```javascript
console.log("jerry".toString());//jerry
console.log((1).toString());//1
console.log([1,2].toString());//1,2
console.log(new Date().toString());//Wed Dec 21 2016 20:35:48 GMT+0800 (中国标准时间)
console.log(function(){}.toString());//function (){}
console.log(null.toString());//error
console.log(undefined.toString());//error
```
同样是检测对象obj调用toString方法（关于toString()方法的用法的可以参考toString的详解），obj.toString()的结果和Object.prototype.toString.call(obj)的结果不一样，这是为什么？
这是因为toString为Object的原型方法，而Array 、Function等类型作为Object的实例，都重写了toString方法。不同的对象类型调用toString方法时，根据原型链的知识，调用的是对应的重写之后的toString方法（Function类型返回内容为函数体的字符串，Array类型返回元素组成的字符串.....），而不会去调用Object上原型toString方法（返回对象的具体类型），所以采用obj.toString()不能得到其对象类型，只能将obj转换为字符串类型；因此，在想要得到对象的具体类型时，应该调用Object上原型toString方法。
我们可以验证一下，将数组的toString方法删除，看看会是什么结果：
```javascript
var arr=[1,2,3];
console.log(Array.prototype.hasOwnProperty("toString"));//true
console.log(arr.toString());//1,2,3
delete Array.prototype.toString;//delete操作符可以删除实例属性
console.log(Array.prototype.hasOwnProperty("toString"));//false
console.log(arr.toString());//"[object Array]"
```
删除了Array的toString方法后，同样再采用arr.toString()方法调用时，不再有屏蔽Object原型方法的实例方法，因此沿着原型链，arr最后调用了Object的toString方法，返回了和Object.prototype.toString.call(arr)相同的结果。

## 参考链接
[为什么用Object.prototype.toString.call(obj)检测对象类型?](https://www.cnblogs.com/youhong/p/6209054.html)
